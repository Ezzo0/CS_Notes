# Explanation
- The motivation to develop a new file system known as the log-structured file system was based on the following observations:
	- **System memories are growing:** 
		- As memory gets bigger, more data can be cached in memory. 
		- As more data is cached, disk traffic increasingly consists of writes, as reads are serviced by the cache.
		- Thus, file system performance is largely determined by its write performance.
	- **There is a large gap between random I/O performance and sequential I/O performance**.
	- **Existing file systems perform poorly on many common workloads:**
		- For example, [[0x28_Locality and The Fast File System|FFS]] would perform a large number of writes to create a new file of size one block: one for a new inode, one to update the inode bitmap, one to the directory data block that the file is in, one to the directory inode to update it, one to the new data block that is a part of the new file, and one to the data bitmap to mark the data block as allocated. 
		- Thus, although [[0x28_Locality and The Fast File System|FFS]] places all of these blocks within the same block group, FFS incurs many short seeks and subsequent rotational delays and thus performance falls far short of peak sequential bandwidth.
	- **File systems are not [[0x25_Redundant Arrays of Inexpensive Disks|RAID]]-aware:** 
		- For example, both [[0x25_Redundant Arrays of Inexpensive Disks#RAID Level 4 Saving Space With Parity|RAID-4]] and [[0x25_Redundant Arrays of Inexpensive Disks#RAID Level 5 Rotating Parity|RAID-5]] have the small-write problem where a logical write to a single block causes 4 physical I/Os to take place. 
		- Existing file systems do not try to avoid this worst-case RAID writing behavior.
- When writing to disk, LFS first buffers all updates (including metadata!) in an inmemory **segment**; when the segment is full, it is written to disk in one long, **sequential transfer** to an unused part of the disk. 
- LFS **never overwrites** existing data, but rather **always** writes segments to free locations.
	![[LFS seq writes.PNG|450]]
##### Writing Sequentially And Effectively
- Unfortunately, writing to disk sequentially is not (alone) enough to guarantee efficient writes. For example, imagine if we wrote a single block to address $A$, at time $T$ . We then wait a little while, and write to the disk at address $A + 1$, but at time $T + δ$. 
- In-between the first and second writes, the disk has rotated; when you issue the second write, it will thus wait for most of a rotation before being committed. 
- Thus you can hopefully see that simply writing to disk in sequential order is not enough to achieve peak performance; rather, you must issue a large number of contiguous writes (or one large write) to the drive in order to achieve good write performance.
- To achieve this end, LFS uses an ancient technique known as **write buffering**. Before writing to the disk, LFS keeps track of updates in memory; when it has received a sufficient number of updates, it writes them to disk all at once, thus ensuring efficient use of the disk.
- The large chunk of updates LFS writes at one time is referred to by the name of a **segment**.
	 ![[LFS buffering.PNG|500]]
##### How Much To Buffer?
- Assume that positioning before each write takes roughly $T_{position}$ seconds. Assume further that the disk transfer rate is $R_{peak}$ MB/s. How much should LFS buffer before writing when running on such a disk?
- The way to think about this is that every time you write, you pay a fixed overhead of the positioning cost. Thus, how much do you have to write in order to **amortize** that cost? The more you write, the better, and the closer you get to achieving peak bandwidth.
- To obtain a concrete answer, let’s assume we are writing out $D$ MB. The time to write out this chunk of data ($T_{write}$) is the positioning time $T_{position}$ plus the time to transfer $D$. $$T_{write} = T_{position} + \frac{D}{R_{peak}}$$
- And thus the effective rate of writing ($R_{effective}$), which is just the amount of data written divided by the total time to write it, is: $$R_{effective} = \frac{D}{Twrite} = \frac{D}{T_{position} +\frac{D}{Rpeak}}$$
- In mathematical form, this means that $R_{effective} = F × R_{peak}$ ,  where $0 < F < 1$.
- At this point, we can solve for D: $$D = \frac{F}{1 - F} × R_{peak} × T_{position}$$
##### The Inode Map
- In LFS, inodes are all scattered throughout the disk. To remedy this, the designers of LFS introduced a **level of indirection** between inode numbers and the inodes through a data structure called the **inode map (imap)**.
- The imap is a structure that takes an inode number as input and produces the disk address of the most recent version of the inode.
- Thus, you can imagine it would often be implemented as a simple array, with 4 bytes (a disk pointer) per entry. Any time an inode is written to disk, the imap is updated with its new location.
- Imap could live on a fixed part of the disk, of course. Unfortunately, as it gets updated frequently, this would then require updates to file structures to be followed by writes to the imap, and hence performance would suffer.
- Instead, LFS places chunks of the inode map right next to where it is writing all of the other new information. 
- Thus, when appending a data block to a file k, LFS actually writes the new data block, its inode, and a piece of the inode map all together onto the disk, as follows:
	 ![[LFS imap.PNG|500]]
##### The Checkpoint Region
- The is a problem here. How do we find the inode map, now that pieces of it are also now spread across the disk?
- In the end, there is no magic: the file system must have some fixed and known location on disk to begin a file lookup.
- LFS has just such a fixed place on disk for this, known as the **checkpoint region (CR)**. The checkpoint region contains pointers to the latest pieces of the inode map, and thus the inode map pieces can be found by reading the CR first.
- Note the checkpoint region is only updated periodically (say every 30 seconds or so), and thus performance is not ill-affected.
	 ![[CR.PNG|550]]
##### What About Directories?
- Directory structure is basically identical to classic UNIX file systems, in that a directory is just a collection of (name, inode number) mappings.
- For example, when creating a file on disk, LFS must both write a new inode, some data, as well as the directory data and its inode that refer to this file. 
- Remember that LFS will do so sequentially on the disk (after buffering the updates for some time). Thus, creating a file foo in a directory would lead to the following new structures on disk:
	 ![[LFS directories.PNG|550]]
- The piece of the inode map contains the information for the location of both the directory file $dir$ as well as the newly-created file $f$. 
- Thus, when accessing file $foo$ (with inode number $k$), you would first look in the inode map (usually cached in memory) to find the location of the inode of directory $dir$ (A3).
- You then read the directory inode, which gives you the location of the directory data (A2); reading this data block gives you the name-to-inode-number mapping of ($foo, k$). 
- You then consult the inode map again to find the location of inode number $k$ (A1), and finally read the desired data block at address A0.
- There is one other serious problem in LFS that the inode map solves, known as the **recursive update problem**. The problem arises in any file system that never updates in place (such as LFS), but rather moves updates to new locations on the disk.
- Whenever an inode is updated, its location on disk changes. If we hadn’t been careful, this would have also entailed an update to the directory that points to this file, which then would have mandated a change to the parent of that directory, and so on, all the way up the file system tree.
- LFS cleverly avoids this problem with the inode map. Even though the location of an inode may change, the change is never reflected in the directory itself; rather, the imap structure is updated while the directory holds the same name-to-inode-number mapping. 
- Thus, through indirection, LFS avoids the recursive update problem.
##### A New Problem: Garbage Collection
- You may have noticed another problem with LFS; it repeatedly writes the latest version of a file (including its inode and data) to new locations on disk. 
- This process, while keeping writes efficient, implies that LFS leaves old versions of file structures scattered throughout the disk. We (rather unceremoniously) call these old versions **garbage**.
	 ![[garbage.PNG|550]]
- In the diagram, you can see that both the inode and data block have two versions on disk, one old (the one on the left) and one current and thus **live** (the one on the right).
- As another example, imagine we instead append a block to that original file k. In this case, a new version of the inode is generated, but the old data block is still pointed to by the inode. Thus, it is still live and very much part of the current file system:
	 ![[append to a file in LFS.PNG|550]]
- So what should we do with these older versions of inodes, data blocks, and so forth? One could keep those older versions around and allow users to restore old file versions; such a file system is known as a **versioning file system** because it keeps track of the different versions of a file. However, LFS instead keeps only the latest live version of a file.
- Imagine what would happen if the LFS cleaner simply went through and freed single data blocks, inodes, etc., during cleaning. The result: a file system with some number of free **holes** mixed between allocated space on disk.
- Write performance would drop considerably, as LFS would not be able to find a large contiguous region.
- Instead, the LFS cleaner works on a segment-by-segment basis, thus clearing up large chunks of space for subsequent writing. The basic cleaning process works as follows.
	- Periodically, the LFS cleaner reads in a number of old (partially-used) segments, determines which blocks are live within these segments, and then write out a new set of segments with just the live blocks within them, freeing up the old ones for writing.
##### Determining Block Liveness
- To do so, LFS adds a little extra information to each segment that describes each block. Specifically, LFS includes, for each data block D, its inode number and its offset. 
- This information is recorded in a structure at the head of the segment known as the **segment summary block**.
- For a block $D$ located on disk at address $A$, look in the segment summary block and find its inode number $N$ and offset $T$.
- Next, look in the imap to find where $N$ lives and read $N$ from disk. Finally, using the offset $T$, look in the inode (or some indirect block) to see where the inode thinks the $T$th block of this file is on disk. 
- If it points exactly to disk address $A$, LFS can conclude that the block $D$ is live. If it points anywhere else, LFS can conclude that $D$ is not in use and thus know that this version is no longer needed.
- There are some shortcuts LFS takes to make the process of determining liveness more efficient. For example, when a file is truncated or deleted, LFS increases its **version number** and records the new version number in the imap.
- By also recording the version number in the on-disk segment, LFS can short circuit the longer check described above simply by comparing the on-disk version number with a version number in the imap, thus avoiding extra reads.
##### A Policy Question: Which Blocks To Clean, And When?
- On top of the mechanism described above, LFS must include a set of policies to determine both when to clean and which blocks are worth cleaning. 
- Determining when to clean is easier; either periodically, during idle time, or when you have to because the disk is full.
- Determining which blocks to clean is more challenging, and has been the subject of many research papers. In the original LFS paper, the authors describe an approach which tries to segregate hot and cold segments. 
- A hot segment is one in which the contents are being frequently over-written; thus, for such a segment, the best policy is to wait a long time before cleaning it, as more and more blocks are getting over-written (in new segments) and thus being freed for use.
- A cold segment, in contrast, may have a few dead blocks but the rest of its contents are relatively stable. 
- Thus, the authors conclude that one should clean cold segments sooner and hot segments later, and develop a heuristic that does exactly that.
##### Crash Recovery And The Log
- During normal operation, LFS buffers writes in a segment, and then, writes the segment to disk. LFS organizes these writes in a **log**, i.e., the checkpoint region points to a head and tail segment, and each segment points to the next segment to be written.
- LFS also periodically updates the checkpoint region.
- Crashes could clearly happen during either of these operations (write to a segment, write to the CR). So how does LFS handle crashes during writes to these structures?
	1. **Crash during write to the CR**
		- To ensure that the CR update happens atomically, LFS actually keeps two CRs, one at either end of the disk, and writes to them alternately.
		- LFS also implements a careful protocol when updating the CR with the latest pointers to the inode map and other information. 
		- Specifically, it first writes out a header (with timestamp), then the body of the CR, and then finally one last block (also with a timestamp).
		- If the system crashes during a CR update, LFS can detect this by seeing an inconsistent pair of timestamps. 
		- LFS will always choose to use the most recent CR that has consistent timestamps, and thus consistent update of the CR is achieved.
	2. **Crash during write to a segment**
		- Because LFS writes the CR every 30 seconds or so, the last consistent snapshot of the file system may be quite old. 
		- Thus, upon reboot, LFS can easily recover by simply reading in the checkpoint region, the imap pieces it points to, and subsequent files and directories; however, the last many seconds of updates would be lost.
		- To improve upon this, LFS tries to rebuild many of those segments through a technique known as **roll forward** in the database community.
		- The basic idea is to start with the last checkpoint region, find the end of the log (which is included in the CR), and then use that to read through the next segments and see if there are any valid updates within it.
		- If there are, LFS updates the file system accordingly and thus recovers much of the data and metadata written since the last checkpoint.
# source
- Operating Systems: Three Easy Pieces - Chapter 43.
- [Lecture 14 - part 1](https://youtu.be/59XSFnXQ-9Q).
- [Lecture 14 - part 2](https://youtu.be/6fbm9u7__L0).