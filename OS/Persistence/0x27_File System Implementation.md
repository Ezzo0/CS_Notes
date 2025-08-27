# Explanation
- In this chapter, we introduce a simple file system implementation, known as **vsfs** (the **Very Simple File System**). This file system is a simplified version of a typical UNIX file system and thus serves to introduce some of the basic on-disk structures, access methods, and various policies that you will find in many file systems today.
##### The Way To Think
- To think about file systems, we usually suggest thinking about two different aspects of them; if you understand both of these aspects, you probably understand how the file system basically works.
- The first is the **data structures** of the file system. In other words, what types of on-[[0x24_Hard Disk Drives|disk]] structures are utilized by the file system to organize its data and metadata?
- The second aspect of a file system is its **access methods**. How does it map the calls made by a process, such as `open()`, `read()`, `write()`, etc., onto its structures? Which structures are read during the execution of a particular system call? Which are written? How efficiently are all of these steps performed?
##### Overall Organization
- The first thing we’ll need to do is divide the disk into **blocks**; simple file systems use just one block size. Let’s choose a commonly-used size of 4 KB.
- Thus, the view of the disk partition is: a series of blocks, each of size 4 KB. The blocks are addressed from 0 to N - 1.
- Let’s now think about what we need to store in these blocks to build a file system. In fact, most of the space in any file system is (and should be) user data. Let’s call the region of the disk we use for user data the **data region**.
	 ![[data region.PNG]]
- The file system has to track information about each file. This information is a key piece of **metadata**, and tracks things like which data blocks (in the data region) comprise a file, the size of the file, its owner and access rights, access and modify times, and other similar kinds of information. 
- To store this information, file systems usually have a structure called an **inode**.
- To accommodate inodes, we’ll need to reserve some space on the disk for them as well. Let’s call this portion of the disk the **inode table**, which simply holds an array of on-disk inodes.
	 ![[inode_table.PNG]]
- We should note here that inodes are typically not that big, for example 128 or 256 bytes.
- One primary component that is still needed, as you might have guessed, is some way to track whether inodes or data blocks are free or allocated. Such **allocation structures** are thus a requisite element in any file system.
- Many allocation-tracking methods are possible We chose a simple and popular structure known as a **bitmap**, one for the data region (the **data bitmap**), and one for the inode table (the **inode bitmap**).
- A bitmap is a simple structure: each bit is used to indicate whether the corresponding object/block is free (0) or in-use (1).
	 ![[bitmap.PNG]]
- There is one block left in the design of the on-disk structure of our very simple file system. We reserve this for the **superblock**.
- The superblock contains information about this particular file system, including, for example, how many inodes and data blocks are in the file system, where the inode table begins (block 3), and so forth. It will likely also include a magic number of some kind to identify the file system type.
##### File Organization: The Inode
- Each inode is implicitly referred to by a number (called the **i-number**), which we’ve earlier called the **low-level name** of the file.
- Recall that disks are not byte addressable, but rather consist of a large number of addressable sectors, usually 512 bytes. 
- Thus, to fetch the block of inodes that contains inode 32, the file system would issue a read to sector $\frac{20×1024}{512}$ , or 40, to fetch the desired inode block. 
- More generally, the sector address sector of the inode block can be calculated as follows: `blk = (inumber * sizeof(inode_t)) / blockSize;
  `sector = ((blk * blockSize) + inodeStartAddr) / sectorSize;`
- Inside each inode is virtually all of the information you need about a file: its **type** (e.g., regular file, directory, etc.), its **size**, the number of blocks allocated to it, **protection information**, some **time information**, including when the file was created, modified, or last accessed, as well as information about where its data blocks reside on disk.
- One of the most important decisions in the design of the inode is how it refers to where data blocks are. One simple approach would be to have one or more **direct pointers** inside the inode; each pointer refers to one disk block that belongs to the file.
- Such an approach is limited: for example, if you want to have a file that is really big (e.g., bigger than the block size multiplied by the number of direct pointers in the inode), you are out of luck.
##### The Multi-Level Index
- To support bigger files, file system designers have had to introduce different structures within inodes. One common idea is to have a special pointer known as an **indirect pointer**
- Instead of pointing to a block that contains user data, it points to a block that contains more pointers, each of which point to user data.
- Thus, an inode may have some fixed number of direct pointers (e.g., 12), and a single indirect pointer. If a file grows large enough, an indirect block is allocated (from the data-block region of the disk), and the inode’s slot for an indirect pointer is set to point to it.
- Not surprisingly, in such an approach, you might want to support even larger files. To do so, just add another pointer to the inode: the **double indirect pointer**. 
- This pointer refers to a block that contains pointers to indirect blocks, each of which contain pointers to data blocks. 
- A double indirect block thus adds the possibility to grow files with an additional 1024· You may want even more, though, and we bet you know where this is headed: the **triple indirect pointer**. Overall, this imbalanced tree is referred to as the **multi-level index** approach to pointing to file blocks.
- Another simpler approach in designing inodes is to use a **linked list**. Thus, inside an inode, instead of having multiple pointers, you just need one, to point to the first block of the file. 
- To handle larger files, add another pointer at the end of that data block, and so on, and thus you can support large files. 
- As you might have guessed, linked file allocation performs poorly for some workloads; think about reading the last block of a file, for example, or just doing random access.
- Thus, to make linked allocation work better, some systems will keep an in-memory table of link information, instead of storing the next pointers with the data blocks themselves.
- The table is indexed by the address of a data block D; the content of an entry is simply D’s next pointer, i.e., the address of the next block in a file which follows D. 
- A null-value could be there too (indicating an end-of-file), or some other marker to indicate that a particular block is free. 
- Having such a table of next pointers makes it so that a linked allocation scheme can effectively do random file accesses, simply by first scanning through the (in memory) table to find the desired block, and then accessing (on disk) it directly.
##### Directory Organization
- For each file or directory in a given directory, there is a string and a number in the data block(s) of the directory. For each string, there may also be a length (assuming variable-sized names).
	 ![[dir.PNG]]
- In this example, each entry has an inode number, record length (the total bytes for the name plus any left over space), string length (the actual length of the name), and finally the name of the entry.
- Note that each directory has two extra entries, . “dot” and .. “dot-dot”; the dot directory is just the current directory, whereas dot-dot is the parent directory.
- Deleting a file (e.g., calling `unlink()`) can leave an empty space in the middle of the directory, and hence there should be some way to mark that as well (e.g., with a reserved inode number such as zero).
- Often, file systems treat directories as a special type of file. Thus, a directory has an inode, somewhere in the inode table (with the type field of the inode marked as **directory** instead of **regular file**).
- The directory has data blocks pointed to by the inode (and perhaps, indirect blocks); these data blocks live in the data block region of our simple file system.
##### Free Space Management
- Free space management is important for all file systems. In vsfs, we have two simple bitmaps for this task.
- For example, when we create a file, we will have to allocate an inode for that file. The file system will thus search through the bitmap for an inode that is free, and allocate it to the file; the file system will have to mark the inode as used (with a 1) and eventually update the on-disk bitmap with the correct information. A similar set of activities take place when a data block is allocated.
- Some other considerations might also come into play when allocating data blocks for a new file. For example, some Linux file systems, such as ext2 and ext3, will look for a sequence of blocks (say 8) that are free when a new file is created and needs data blocks; by finding such a sequence of free blocks, and then allocating them to the newly-created file, the file system guarantees that a portion of the file will be contiguous on the disk, thus improving performance. Such a **pre-allocation** policy is thus a commonly-used heuristic when allocating space for data blocks.
##### Access Paths: Reading and Writing
- **_Reading A File From Disk_**
	- When you issue an `open("/foo/bar", O_RDONLY)` call, the file system first needs to find the inode for the file bar, to obtain some basic information about the file (permissions information, file size, etc.). 
	- To do so, the file system must be able to find the inode, but all it has right now is the full pathname. The file system must **traverse** the pathname and thus locate the desired inode.
	- All traversals begin at the root of the file system, in the **root directory** which is simply called `/`. Thus, the first thing the FS will read from disk is the inode of the root directory. But where is this inode? 
	- To find an inode, we must know its i-number. Usually, we find the i-number of a file or directory in its parent directory; the root has no parent (by definition).
	- Thus, the root inode number must be “well known”; the FS must know what it is when the file system is mounted. In most UNIX file systems, the root inode number is 2. Thus, to begin the process, the FS reads in the block that contains inode number 2 (the first inode block).
	- Once the inode is read in, the FS can look inside of it to find pointers to data blocks, which contain the contents of the root directory. The FS will thus use these on-disk pointers to read through the directory.
	- By reading in one or more directory data blocks, it will find the entry for `foo`; once found, the FS will also have found the inode number of `foo` which it will need next.
	- The next step is to recursively traverse the pathname until the desired inode is found.
	- The final step of `open()` is to read `bar`’s inode into memory; the FS then does a final permissions check, allocates a file descriptor for this process in the per-process open-file table, and returns it to the user.
	- Once open, the program can then issue a `read()` system call to read from the file. The first read (at offset 0 unless `lseek()` has been called) will thus read in the first block of the file, consulting the inode to find the location of such a block; it may also update the inode with a new lastaccessed time. 
	- The read will further update the in-memory open file table for this file descriptor, updating the file offset such that the next read will read the second file block, etc.
	 ![[File Read Timeline.PNG]]
- **_Writing A File To Disk_**
	- Writing to a file is a similar process. First, the file must be opened (as above). Then, the application can issue `write()` calls to update the file with new contents. Finally, the file is closed.
	- Unlike reading, writing to the file may also **allocate** a block (unless the block is being overwritten, for example). When writing out a new file, each write not only has to write data to disk but has to first decide which block to allocate to the file and thus update other structures of the disk accordingly (e.g., the data bitmap and inode).
	- Thus, each write to a file logically generates five I/Os: 
		1. One to read the data bitmap (which is then updated to mark the newly-allocated block as used). 
		2. One to write the bitmap (to reflect its new state to disk).
		3. Two more to read and then write the inode (which is updated with the new block’s location). 
		4. One to write the actual block itself.
	- The amount of write traffic is even worse when one considers a simple and common operation such as file creation. To create a file, the file system must not only allocate an inode, but also allocate space within the directory containing the new file. 
	- The total amount of I/O traffic to do so is quite high: 
		1. One read to the inode bitmap (to find a free inode). 
		2. One write to the inode bitmap (to mark it allocated). 
		3. One write to the new inode itself (to initialize it). 
		4. One to the data of the directory (to link the high-level name of the file to its inode number). 
		5. One read and write to the directory inode to update it. 
		6. If the directory needs to grow to accommodate the new entry, additional I/Os (i.e., to the data bitmap, and the new directory block) will be needed too. 
	 ![[File Creation Timeline.PNG]]
##### Caching and Buffering
- To remedy what would clearly be a huge performance problem, most file systems aggressively use system memory (DRAM) to cache important blocks.
- Without caching, every file open would require at least two reads for every level in the directory hierarchy (one to read the inode of the directory in question, and at least one to read its data). 
- With a long pathname (e.g., /1/2/3/ ... /100/file.txt), the file system would literally perform hundreds of reads just to open the file!
- Early file systems thus introduced a **fixed-size** cache to hold popular blocks. As in our discussion of virtual memory, strategies such as **[[0x15_Beyond Physical Memory_Policies#Using History LRU|LRU]]** and different variants would decide which blocks to keep in cache. This fixed-size cache would usually be allocated at boot time to be roughly 10% of total memory.
- This **static partitioning** of memory, however, can be wasteful; what if the file system doesn’t need 10% of memory at a given point in time? 
- With the fixed-size approach described above, unused pages in the file cache cannot be re-purposed for some other use, and thus go to waste.
- Modern systems, in contrast, employ a **dynamic partitioning** approach. Specifically, many modern operating systems integrate virtual memory pages and file system pages into a **unified page cache**.
- In this way, memory can be allocated more flexibly across virtual memory and file system, depending on which needs more memory at a given time.
- Whereas read I/O can be avoided altogether with a sufficiently large cache, write traffic has to go to disk in order to become persistent.
- Thus, a cache does not serve as the same kind of filter on write traffic that it does for reads. That said, **write buffering** (as it is sometimes called) certainly has a number of performance benefits.
	- First, by delaying writes, the file system can batch some updates into a smaller set of I/Os; for example, if an inode bitmap is updated when one file is created and then updated moments later as another file is created, the file system saves an I/O by delaying the write after the first update.
	- Second, by buffering a number of writes in memory, the system can then **schedule** the subsequent I/Os and thus increase performance.
	- Finally, some writes are avoided altogether by delaying them; for example, if an application creates a file and then deletes it, delaying the writes to reflect the file creation to disk **avoids** them entirely.
- To avoid unexpected data loss due to write buffering, they simply force writes to disk, by calling `fsync()`, by using **direct I/O** interfaces that work around the cache, or by using the **raw disk** interface and avoiding the file system altogether.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 40.
- [Lecture 12 - part 1](https://youtu.be/QMjJlCqUYW4).
- [Lecture 12 - part 2](https://youtu.be/87vv7nVdTDA).
- [Lecture 12 - part 3](https://youtu.be/5n0AdNuBObU).