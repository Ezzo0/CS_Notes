# Explanation
- Unlike most data structures of a running program in memory, file system data structures must **persist**, i.e., they must survive over the long haul, stored on devices that retain data despite **power loss** or **system crash**.
- Because of power losses and crashes, updating a persistent data structure can be quite tricky, and leads to a new and interesting problem in file system implementation, known as the **crash-consistency problem**.
- This problem is quite simple to understand. Imagine you have to update two on-disk structures, A and B, in order to complete a particular operation.
- Because the disk only services a single request at a time, one of these requests will reach the disk first (either A or B). If the system crashes or loses power after one write completes, the on-disk structure will be left in an **inconsistent** state.
##### Crash Scenarios
- To understand the problem better, let’s look at some example crash scenarios. Imagine only a single write succeeds; there are thus three possible outcomes, which we list here:
	1. **Just the data block (Db) is written to disk.** 
		- In this case, the data is on disk, but there is no inode that points to it and no bitmap that even says the block is allocated. 
		- Thus, it is as if the write never occurred. This case is not a problem at all, from the perspective of file-system crash consistency.
	2. **Just the updated inode ($I[v2]$) is written to disk.** 
		- In this case, the inode points to the disk address (5) where Db was about to be written, but Db has not yet been written there. 
		- Thus, if we trust that pointer, we will read **garbage** data from the disk.
		- Further, we have a new problem, which we call a **file-system inconsistency**. The on-disk bitmap is telling us that data block 5 has not been allocated, but the inode is saying that it has.
	3. **Just the updated bitmap ($B[v2]$) is written to disk.**
		-  In this case, the bitmap indicates that block 5 is allocated, but there is no inode that points to it. Thus the file system is inconsistent again. 
		- If the problem is left unresolved, this write would result in a space leak, as block 5 would never be used by the file system.
- There are also three more crash scenarios in this attempt to write three blocks to disk. In these cases, two writes succeed and the last one fails:
	1. **The inode ($I[v2]$) and bitmap ($B[v2]$) are written to disk, but not data (Db).** 
		- In this case, the file system metadata is completely consistent: the inode has a pointer to block 5, the bitmap indicates that 5 is in use, and thus everything looks OK from the perspective of the file system’s metadata. But there is one problem: 5 has garbage in it again. 
	2. **The inode ($I[v2]$) and the data block (Db) are written, but not the bitmap ($B[v2]$).** 
		- In this case, we have the inode pointing to the correct data on disk, but again have an inconsistency between the inode and the old version of the bitmap (B1).
	3. **The bitmap ($B[v2]$) and data block (Db) are written, but not the inode ($I[v2]$).**
		- In this case, we again have an inconsistency between the inode and the data bitmap. However, even though the block was written and the bitmap indicates its usage, we have no idea which file it belongs to, as no inode points to the file. 
##### Solution #1: The File System Checker
- Early file systems took a simple approach to crash consistency. Basically, they decided to let inconsistencies happen and then fix them later (when rebooting).
- A classic example of this lazy approach is found in a tool that does this: **fsck**. fsck is a UNIX tool for finding such inconsistencies and repairing them.
- Note that such an approach can’t fix all problems; consider, for example, the case above where the file system looks consistent but the inode points to garbage data. The only real goal is to make sure the file system metadata is internally consistent.
- The tool fsck run before the file system is mounted and made available (fsck assumes that no other file-system activity is on-going while it runs). 
- Once finished, the on-disk file system should be consistent and thus can be made accessible to users.
- Here is a basic summary of what _fsck_ does:
	- **Superblock:** 
		- fsck first checks if the superblock looks reasonable, mostly doing sanity checks such as making sure the file system size is greater than the number of blocks that have been allocated. 
		- Usually the goal of these sanity checks is to find a suspect (corrupt) superblock; in this case, the system (or administrator) may decide to use an alternate copy of the superblock.
	- **Free blocks:**
		- fsck scans the inodes, indirect blocks, double indirect blocks, etc., to build an understanding of which blocks are currently allocated within the file system. It uses this knowledge to produce a correct version of the allocation bitmaps.
		- Thus, if there is any inconsistency between bitmaps and inodes, it is resolved by trusting the information within the inodes. 
		- The same type of check is performed for all the inodes, making sure that all inodes that look like they are in use are marked as such in the inode bitmaps.
	- **Inode state:**
		- Each inode is checked for corruption or other problems. For example, fsck makes sure that each allocated inode has a valid type field. 
		- If there are problems with the inode fields that are not easily fixed, the inode is considered suspect and cleared by fsck; the inode bitmap is correspondingly updated.
	- **Inode links:**
		- fsck also verifies the link count of each allocated inode. To verify the link count, fsck scans through the entire directory tree, starting at the root directory, and builds its own link counts for every file and directory in the file system.
		- If there is a mismatch between the newly-calculated count and that found within an inode, corrective action must be taken, usually by fixing the count within the inode. 
		- If an allocated inode is discovered but no directory refers to it, it is moved to the **lost+found** directory.
	- **Duplicates:**
		- fsck also checks for duplicate pointers, i.e., cases where two different inodes refer to the same block. If one inode is obviously bad, it may be cleared. 
		- Alternately, the pointed-to block could be copied, thus giving each inode its own copy as desired.
	- **Bad blocks:**
		- A check for bad block pointers is also performed while scanning through the list of all pointers. 
		- A pointer is considered “bad” if it obviously points to something outside its valid range, e.g., it has an address that refers to a block greater than the partition size.
		- In this case, fsck can’t do anything too intelligent; it just removes (clears) the pointer from the inode or indirect block.
	- **Directory checks:**
		- fsck does not understand the contents of user files; however, directories hold specifically formatted information created by the file system itself.
		- Thus, fsck performs additional integrity checks on the contents of each directory, making sure that “.” and “..” are the first entries, that each inode referred to in a directory entry is allocated, and ensuring that no directory is linked to more than once in the entire hierarchy.
- Fsck and similar approaches have a bigger and perhaps more fundamental problem: they are **too slow**. With a very large disk volume, scanning the entire disk to find all the allocated blocks and read the entire directory tree may take many minutes or hours. 
- Performance of fsck, as disks grew in capacity and RAIDs grew in popularity, became prohibitive.
##### Solution #2: Journaling (or Write-Ahead Logging)
- The basic idea is as follows. When updating the disk, before overwriting the structures in place, first write down a little note (somewhere else on the disk, in a well-known location) describing what you are about to do.
- Writing this note is the **write ahead** part, and we write it to a structure that we organize as a **log**; hence, write-ahead logging.
- By writing the note to disk, you are guaranteeing that if a crash takes places during the update (overwrite) of the structures you are updating, you can go back and look at the note you made and try again. 
- Thus, you will know exactly what to fix (and how to fix it) after a crash, instead of having to scan the entire disk.
- Most of the on-disk structures are identical to **Linux ext2**, e.g., the disk is divided into block groups, and each block group contains an inode bitmap, data bitmap, inodes, and data blocks.
- The new key structure is the journal itself, which occupies some small amount of space within the partition or on another device. Thus, an ext2 file system (without journaling) looks like this:
	 ![[Linux ext2.PNG]]
- Assuming the journal is placed within the same file system image (though sometimes it is placed on a separate device, or as a file within the file system), a **Linux ext3** file system with a journal looks like this:
	 ![[Linux ext3.PNG]]
###### Data Journaling
- Say we have our canonical update again, where we wish to write the inode ($I[v2]$), bitmap ($B[v2]$), and data block ($Db$) to disk again. Before writing them to their final disk locations, we are now first going to write them to the log (a.k.a. journal).
	 ![[data journaling.PNG]]
- The transaction begin ($TxB$) tells us about this update, including information about the pending update to the file system (e.g., the final addresses of the blocks $I[v2]$, $B[v2]$, and $Db$), and some kind of **transaction identifier ($TID$)**.
- The middle three blocks just contain the exact contents of the blocks themselves; this is known as **physical logging**.
- The final block ($TxE$) is a marker of the end of this transaction, and will also contain the $TID$.
- Once this transaction is safely on disk, we are ready to overwrite the old structures in the file system; this process is called **checkpointing**.
- Things get a little trickier when a crash occurs during the writes to the journal. Here, we are trying to write the set of blocks in the transaction to disk. 
- One simple way to do this would be to issue each one at a time, waiting for each to complete, and then issuing the next.
- However, this is slow. Ideally, we’d like to issue all five block writes at once, as this would turn five writes into a single sequential write and thus be faster.
- However, this is unsafe, for the following reason: given such a big write, the disk internally may perform scheduling and complete small pieces of the big write in any order. 
- Thus, the disk internally may 
	1. Write $TxB$, $I[v2]$, $B[v2]$, and $TxE$. 
	2. And only later Write $Db$. 
- Unfortunately, if the disk loses power between (1) and (2), this is what ends up on disk:
	 ![[powerloss in jounal write.PNG]]
- Why is this a problem? Well, the transaction looks like a valid transaction (it has a begin and an end with matching sequence numbers). Further, the file system can’t look at that fourth block and know it is wrong; after all, it is arbitrary user data.
- Thus, if the system now reboots and runs recovery, it will replay this transaction, and ignorantly copy the contents of the garbage block to the location where $Db$ is supposed to live.
- To avoid this problem, the file system issues the transactional write in two steps. First, it writes all blocks except the $TxE$ block to the journal, issuing these writes all at once.
	 ![[without TxE.PNG]]
- When those writes complete, the file system issues the write of the $TxE$ block.
- An important aspect of this process is the atomicity guarantee provided by the disk. It turns out that the disk guarantees that any 512-byte write will either happen or not (and never be half-written); thus, to make sure the write of $TxE$ is atomic, one should make it a single 512-byte block.
- Thus, our current protocol to update the file system, with each of its three phases labeled:
	1. **Journal write**: Write the contents of the transaction (including $TxB$, metadata, and data) to the log; wait for these writes to complete.
	2. **Journal commit**: Write the transaction commit block (containing $TxE$) to the log; wait for write to complete; transaction is said to be committed.
	3. **Checkpoint**: Write the contents of the update (metadata and data) to their final on-disk locations.
###### Recovery
- A crash may happen at any time during this sequence of updates. If the crash happens before the transaction is written safely to the log (i.e., before Step 2 above completes), then our job is easy: the pending update is simply skipped.
- If the crash happens after the transaction has committed to the log, but before the checkpoint is complete, the file system can recover the update as follows.
	- When the system boots, the file system recovery process will scan the log and look for transactions that have committed to the disk. 
	- These transactions are thus replayed (in order), with the file system again attempting to write out the blocks in the transaction to their final on-disk locations.
	- This form of logging is one of the simplest forms there is, and is called **redo logging**.
###### Batching Log Updates
- Imagine we create two files in a row, called file1 and file2, in the same directory. To create one file, one has to update a number of on-disk structures, minimally including: the inode bitmap (to allocate a new inode), the newly-created inode of the file, the data block of the parent directory containing the new directory entry, and the parent directory inode (which now has a new modification time).
- With journaling, we logically commit all of this information to the journal for each of our two file creations; because the files are in the same directory, and assuming they even have inodes within the same inode block, this means that if we’re not careful, we’ll end up writing these same blocks over and over.
- To remedy this problem, some file systems do not commit each update to disk one at a time (e.g., Linux ext3); rather, one can buffer all updates into a global transaction.
- In our example above, when the two files are created, the file system just marks the in-memory inode bitmap, inodes of the files, directory data, and directory inode as dirty, and adds them to the list of blocks that form the current transaction.
- When it is finally time to write these blocks to disk (say, after a timeout of 5 seconds), this single global transaction is committed containing all of the updates described above. Thus, by buffering updates, a file system can avoid excessive write traffic to disk in many cases.
###### Making The Log Finite
- The file system buffers updates in memory for some time; when it is finally time to write to disk, the file system first carefully writes out the details of the transaction to the journal; after the transaction is complete, the file system checkpoints those blocks to their final locations on disk.
- However, the log is of a finite size. If we keep adding transactions to it, it will soon fill. What do you think happens then?
- Two problems arise when the log becomes full. 
	- The first is simpler, but less critical: the larger the log, the longer recovery will take, as the recovery process must replay all the transactions within the log (in order) to recover.
	- The second is more of an issue: when the log is full (or nearly full), no further transactions can be committed to the disk, thus making the file system useless.
- To address these problems, journaling file systems treat the log as a circular data structure, re-using it over and over; this is why the journal is sometimes referred to as a **circular log**.
- To do so, the file system must take action some time after a checkpoint. Specifically, once a transaction has been checkpointed, the file system should free the space it was occupying within the journal, allowing the log space to be reused. 
- There are many ways to achieve this end; for example, you could simply mark the oldest and newest non-checkpointed transactions in the log in a **journal superblock**; all other space is free.
	 ![[journal superblock.PNG]]
- In the journal superblock, the journaling system records enough information to know which transactions have not yet been checkpointed, and thus reduces recovery time as well as enables re-use of the log in a circular fashion. 
- And thus we add another step to our basic protocol:
	1. **Journal write**: Write the contents of the transaction (containing $TxB$ and the contents of the update) to the log; wait for these writes to complete.
	2. **Journal commit**: Write the transaction commit block (containing $TxE$) to the log; wait for the write to complete; the transaction is now committed.
	3. **Checkpoint**: Write the contents of the update to their final locations within the file system.
	4. **Free**: Some time later, mark the transaction free in the journal by updating the journal superblock.
- But there is still a problem: we are writing each data block to the disk twice, which is a heavy cost to pay, especially for something as rare as a system crash.
###### Metadata Journaling
- Because of the high cost of writing every data block to disk twice, people have tried a few different things in order to speed up performance. For example, the mode of journaling we described above is often called **data journaling** (as in Linux ext3), as it journals all user data (in addition to the metadata of the file system). 
- A simpler (and more common) form of journaling is sometimes called **ordered journaling** (or just **metadata journaling**), and it is nearly the same, except that user data is not written to the journal.
- The data block Db, previously written to the log, would instead be written to the file system proper, avoiding the extra write; given that most I/O traffic to the disk is data, not writing data twice substantially reduces the I/O load of journaling.
- The modification does raise an interesting question, though: when should we write data blocks to disk?
- The update consists of three blocks: $I[v2]$, $B[v2]$, and $Db$. The first two are both metadata and will be logged and then checkpointed; the latter will only be written once to the file system.
- As it turns out, the ordering of the data write does matter for metadata only journaling. For example, what if we write $Db$ to disk after the transaction (containing $I[v2]$ and $B[v2]$) completes?
- Unfortunately, this approach has a problem: the file system is consistent but $I[v2]$ may end up pointing to garbage data. Specifically, consider the case where $I[v2]$ and $B[v2]$ are written but $Db$ did not make it to disk.
- The file system will then try to recover. Because Db is not in the log, the file system will replay writes to $I[v2]$ and $B[v2]$, and produce a consistent file system (from the perspective of file-system metadata). However, $I[v2]$ will be pointing to garbage data, i.e., at whatever was in the slot where $Db$ was headed.
- To ensure this situation does not arise, some file systems (e.g., Linux ext3) write data blocks (of regular files) to the disk first, before related metadata is written to disk. Specifically, the protocol is as follows:
	1. **Data write**: Write data to final location; wait for completion.
	2. **Journal metadata write**: Write the begin block and metadata to the log; wait for writes to complete.
	3. **Journal commit**: Write the transaction commit block (containing $TxE$) to the log; wait for the write to complete; the transaction (including data) is now committed.
	4. **Checkpoint metadata**: Write the contents of the metadata update to their final locations within the file system.
	5. **Free**: Later, mark the transaction free in journal superblock.
- Finally, note that forcing the data write to complete (Step 1) before issuing writes to the journal (Step 2) is not required for correctness, as indicated in the protocol above. Specifically, it would be fine to concurrently issue writes to data, the transaction-begin block, and journaled metadata; the only real requirement is that Steps 1 and 2 complete before the issuing of the journal commit block (Step 3).
###### Tricky Case: Block Reuse
- Let’s say you have a directory called `foo`. The user adds an entry to `foo` (say by creating a file), and thus the contents of `foo` (because directories are considered metadata) are written to the log. 
- Assume the location of the `foo` directory data is block 1000. The log thus contains something like this:
	 ![[journal log.PNG]]
- At this point, the user deletes everything in the directory and the directory itself, freeing up block 1000 for reuse.
- Finally, the user creates a new file (say `bar`), which ends up reusing the same block (1000) that used to belong to `foo`.
- The inode of `bar` is committed to disk, as is its data; note, however, because metadata journaling is in use, only the inode of `bar` is committed to the journal; the newly-written data in block 1000 in the file `bar` is not journaled.
- Now assume a crash occurs and all of this information is still in the log. During replay, the recovery process simply replays everything in the log, including the write of directory data in block 1000. 
- The replay thus overwrites the user data of current file bar with old directory contents! Clearly this is not a correct recovery action, and certainly it will be a surprise to the user when reading the file bar.
- There are a number of solutions to this problem. One could, for example, never reuse blocks until the delete of said blocks is checkpointed out of the journal.
- What Linux ext3 does instead is to add a new type of record to the journal, known as a **revoke** record.
- In the case above, deleting the directory would cause a revoke record to be written to the journal. When replaying the journal, the system first scans for such revoke records; any such revoked data is never replayed, thus avoiding the problem mentioned above.
##### Solution #3: Other Approaches
- We’ve thus far described two options in keeping file system metadata consistent: a lazy approach based on fsck, and a more active approach known as journaling.
- However, these are not the only two approaches. One such approach, known as **Soft Updates**.
- This approach carefully orders all writes to the file system to ensure that the on-disk structures are never left in an inconsistent state. 
- For example, by writing a pointed-to data block to disk before the inode that points to it, we can ensure that the inode never points to garbage; similar rules can be derived for all the structures of the file system.
- Implementing Soft Updates can be a challenge, however; whereas the journaling layer described above can be implemented with relatively little knowledge of the exact file system structures, Soft Updates requires intricate knowledge of each file system data structure and thus adds a fair amount of complexity to the system.
- Another approach is known as **copy-on-write** (**COW**), and is used in a number of popular file systems. 
- This technique never overwrites files or directories in place; rather, it places new updates to previously unused locations on disk. 
- After a number of updates are completed, COW file systems flip the root structure of the file system to include pointers to the newly updated structures.
- Another approach is one we just developed here at Wisconsin. In this technique, entitled **backpointer-based consistency** (or **BBC**), no ordering is enforced between writes. 
- To achieve consistency, an additional **back pointer** is added to every block in the system; for example, each data block has a reference to the inode to which it belongs. 
- When accessing a file, the file system can determine if the file is consistent by checking if the forward pointer (e.g., the address in the inode or direct block) points to a block that refers back to it.
- If so, everything must have safely reached disk and thus the file is consistent; if not, the file is inconsistent, and an error is returned. By adding back pointers to the file system, a new form of lazy crash consistency can be attained.
- We also have explored techniques to reduce the number of times a journal protocol has to wait for disk writes to complete. Entitled **optimistic crash consistency**, this new approach issues as many writes to disk as possible by using a generalized form of the **transaction checksum**, and includes a few other techniques to detect inconsistencies should they arise.
# source
- Operating Systems: Three Easy Pieces - Chapter 42.
- [Lecture 13 - part 1]().
- [Lecture 13 - part 2]().