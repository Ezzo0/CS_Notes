# Explanation
- When the UNIX operating system was first introduced, the UNIX wizard himself Ken Thompson wrote the first file system. Let’s call that the “old UNIX file system”, and it was really simple. Basically, its data structures looked like this on the disk:
	 ![[old unix fs.PNG|500]]
- The super block (S) contained information about the entire file system: how big the volume is, how many inodes there are, a pointer to the head of a free list of blocks, and so forth. The inode region of the disk contained all the inodes for the file system. Finally, most of the disk was taken up by data blocks.
##### The Problem: Poor Performance
- The main issue was that the old UNIX file system treated the disk like it was a random-access memory; data was spread all over the place without regard to the fact that the medium holding the data was a disk, and thus had real and expensive positioning costs.
- For example, the data blocks of a file were often very far away from its inode, thus inducing an expensive seek whenever one first read the inode and then the data blocks of a file.
- Worse, the file system would end up getting quite **fragmented**, as the free space was not carefully managed. 
- The free list would end up pointing to a bunch of blocks spread across the disk, and as files got allocated, they would simply take the next free block.
	 ![[fragmentation.PNG|400]]
- This problem is exactly what disk defragmentation tools help with; they reorganize ondisk data to place files contiguously and make free space for one or a few contiguous regions, moving data around and then rewriting inodes and such to reflect the changes.
- One other problem: the original block size was too small (512 bytes). Thus, transferring data from the disk was inherently inefficient. 
- Smaller blocks were good because they minimized **internal fragmentation**, but bad for transfer as each block might require a positioning overhead to reach it.
##### Fast File System (FFS): Disk Awareness Is The Solution
- The idea was to design the file system structures and allocation policies to be “disk aware” and thus improve performance.
- FFS thus ushered in a new era of file system research; by keeping the same _interface_ to the file system but changing the internal implementation, the authors paved the path for new file system construction, work that continues today.
##### Organizing Structure: The Cylinder Group
- The first step was to change the on-disk structures. FFS divides the disk into a number of **cylinder groups**. 
- A single **cylinder** is a set of tracks on different surfaces of a hard drive that are the same distance from the center of the drive.
	 ![[cylinder group.PNG]]
- Note that modern drives do not export enough information for the file system to truly understand whether a particular cylinder is in use, disks export a logical address space of blocks and hide details of their geometry from clients.
- Thus, modern file systems (such as Linux ext2, ext3, and ext4) instead organize the drive into **block groups**, each of which is just a consecutive portion of the disk’s address space.
	 ![[block groups.PNG|470]]
- These groups are the central mechanism that FFS uses to improve performance. Critically, by placing two files within the same group, FFS can ensure that accessing one after the other will not result in long seeks across the disk.
- To use these groups to store files and directories, FFS needs to have the ability to place files and directories into a group, and track all necessary information about them therein.
- To do so, FFS includes all the structures you might expect a file system to have within each group, e.g., space for inodes, data blocks, and some structures to track whether each of those are allocated or free.
	 ![[single block group.PNG|500]]
- FFS keeps a copy of the **super block** (S) in each group for reliability reasons. The super block is needed to mount the file system; by keeping multiple copies, if one copy becomes corrupt, you can still mount and access the file system by using a working replica.
- Within each group, FFS needs to track whether the inodes and data blocks of the group are allocated. A per-group **inode bitmap** (ib) and **data bitmap** (db) serve this role for inodes and data blocks in each group.
- The **inode** and **data** block regions are just like those in the previous very-simple file system (VSFS).
##### Policies: How To Allocate Files and Directories
- With this group structure in place, FFS now has to decide how to place files and directories and associated metadata on disk to improve performance. 
- The basic mantra is simple: **_keep related stuff together_**. To achieve this end, FFS makes use of a few simple placement heuristics.
	- The first is the placement of directories. FFS employs a simple approach: find the cylinder group with a **low number of allocated directories** (to balance directories across groups) and a **high number of free inodes** (to subsequently be able to allocate a bunch of files), and put the directory data and inode in that group.
	- For files, FFS does two things. 
		1. First, it makes sure (in the general case) to allocate the data blocks of a file in the **same group as its inode**, thus preventing long seeks between inode and data (as in the old file system). 
		2. Second, it places all files that are in the same directory in the cylinder group of the directory they are in.
- The FFS policy heuristics are not based on extensive studies of filesystem traffic or anything particularly nuanced; rather, they are based on good old-fashioned **common sense**.
##### The Large-File Exception
- In FFS, there is one important exception to the general policy of file placement, and it arises for large files. Without a different rule, a large file would entirely fill the block group it is first placed within (and maybe others).
- Filling a block group in this manner is undesirable, as it prevents subsequent “related” files from being placed within this block group, and thus may hurt file-access locality.
- Thus, for large files, FFS does the following. 
	- After some number of blocks are allocated into the first block group (e.g., 12 blocks, or the number of direct pointers available within an inode), FFS places the next “large” chunk of the file (e.g., those pointed to by the first indirect block) in another block group (perhaps chosen for its low utilization).
	- Then, the next chunk of the file is placed in yet another different block group, and so on.
- Here is the depiction of FFS without the large-file exception:
	 ![[FFS without exception.PNG]]
- As you can see in the picture, /a fills up most of the data blocks in Group 0, whereas other groups remain empty. If some other files are now created in the root directory (/), there is not much room for their data in the group.
- With the large-file exception (here set to five blocks in each chunk), FFS instead spreads the file spread across groups, and the resulting utilization within any one group is not too high:
	 ![[FFS exception.PNG]]
- Note that spreading blocks of a file across the disk will hurt performance, particularly in the relatively common case of sequential file access. But the problem can be addressed by choosing **chunk size** carefully.
	- If the chunk size is large enough, the file system will spend most of its time transferring data from disk and just a (relatively) little time seeking between chunks of the block. This process of reducing an overhead by doing more work per overhead paid is called **amortization** and is a common technique in computer systems.
- In order to spread large files across groups, FFS took a simple approach, based on the structure of the inode itself.
	- The first twelve direct blocks were placed in the same group as the inode; each subsequent indirect block, and all the blocks it pointed to, was placed in a different group.
##### A Few Other Things About FFS
- FFS introduced a few other innovations too. In particular, the designers were extremely worried about accommodating small files; as it turned out, many files were 2KB or so in size back then, and using 4KB blocks, while good for transferring data, was not so good for space efficiency. 
- This **internal fragmentation** could thus lead to roughly half the disk being wasted for a typical file system.
- The solution the FFS designers hit upon was simple and solved the problem. They decided to introduce **sub-blocks**, which were 512-byte little blocks that the file system could allocate to files. 
- Thus, if you created a small file (say 1KB in size), it would occupy two sub-blocks and thus not waste an entire 4KB block.
- As the file grew, the file system will continue allocating 512-byte blocks to it until it acquires a full 4KB of data. At that point, FFS will find a 4KB block, copy the sub-blocks into it, and free the sub-blocks for future use.
- You might observe that this process is inefficient, requiring a lot of extra work for the file system (in particular, a lot of extra I/O to perform the copy). 
- Thus, FFS generally avoided this pessimal behavior by modifying the libc library; the library would buffer writes and then issue them in 4KB chunks to the file system, thus avoiding the sub-block specialization entirely in most cases.
- A second neat thing that FFS introduced was a disk layout that was optimized for performance. In those times (before SCSI and other more modern device interfaces), disks were much less sophisticated and required the host CPU to control their operation in a more hands-on way. 
- A problem arose in FFS when a file was placed on consecutive sectors of the disk. In particular, the problem arose during sequential reads. 
- FFS would first issue a read to block 0; by the time the read was complete, and FFS issued a read to block 1, it was too late: block 1 had rotated under the head and now the read to block 1 would incur a full rotation.
	 ![[FFS placement.PNG]]
- FFS solved this problem with a different layout, as you can see on the right in Figure 41.3. By skipping over every other block (in the example), FFS has enough time to request the next block before it went past the disk head.
- In fact, FFS was smart enough to figure out for a particular disk how many blocks it should skip in doing layout in order to avoid the extra rotations; this technique was called **parameterization**.
- This scheme isn’t so great after all. In fact, you will only get 50% of peak bandwidth with this type of layout, because you have to go around each track twice just to read each block once. 
- Fortunately, modern disks are much smarter: they internally read the entire track in and buffer it in an internal disk cache (often called a **track buffer**). Then, on subsequent reads to the track, the disk will just return the desired data from its cache.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 41.
- [Lecture 13 - part 3](https://youtu.be/wwvMNItRyl8).