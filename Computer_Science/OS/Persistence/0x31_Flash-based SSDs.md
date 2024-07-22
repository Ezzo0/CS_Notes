# Explanation
- Unlike [[0x24_Hard Disk Drives|hard-disk drives]] **solid-state storage** have no mechanical or moving parts; rather, they are simply built out of transistors, much like memory and processors.
- However, unlike typical random-access memory (e.g., DRAM), such a **solid-state storage** device retains information despite power loss, and thus is an ideal candidate for use in persistent storage of data.
##### Storing a Single Bit
- Flash chips are designed to store one or more bits in a single transistor; the level of charge trapped within the transistor is mapped to a binary value.
- In a **single-level cell (SLC)** flash, only a single bit is stored within a transistor (i.e., 1 or 0); with a **multi-level cell (MLC)** flash, two bits are encoded into different levels of charge, e.g., 00, 01, 10, and 11 are represented by low, somewhat low, somewhat high, and high levels. There is even **triple-level cell (TLC)** flash, which encodes 3 bits per cell.
##### From Bits to Banks/Planes
- Flash chips are organized into **banks** or **planes** which consist of a large number of cells.
- A bank is accessed in two different sized units: **blocks** (sometimes called **erase blocks**), which are typically of size 128 KB or 256 KB, and **pages**, which are a few KB in size (e.g., 4KB).
- Within each bank there are a large number of blocks; within each block, there are a large number of pages.
	 ![[A Simple Flash Chip.PNG|450]]
##### Basic Flash Operations
- The **read** command is used to read a page from the flash; **erase** and **program** are used in tandem to write.
	- **Read (a page):** 
		- This operation is typically quite fast, 10s of microseconds or so, regardless of location on the device, and (more or less) regardless of the location of the previous request. 
		- Being able to access any location uniformly quickly means the device is a **random access** device.
	- **Erase (a block):**
		- Before writing to a page within a flash, the nature of the device requires that you first erase the entire block the page lies within.
		- The erase command is quite expensive, taking a few milliseconds to complete.
	- **Program (a page):**
		- Programming a page is less expensive than erasing a block, but more costly than reading a page, usually taking around 100s of microseconds on modern flash chips.
- One way to think about flash chips is that each page has a state associated with it. Pages start in an $INVALID$ state. 
- By erasing the block that a page resides within, you set the state of the page (and all pages within that block) to $ERASED$, which resets the content of each page in the block but also (importantly) makes them programmable. 
- When you program a page, its state changes to $VALID$, meaning its contents have been set and can be read.
- Frequent repetitions of this program/erase cycle can lead to the biggest reliability problem flash chips have: **wear out**.
- One other reliability problem within flash chips is known as **disturbance**. When accessing a particular page within a flash, it is possible that some bits get flipped in neighboring pages; such bit flips are known as **read disturbs** or **program disturbs**, depending on whether the page is being read or programmed, respectively.
##### From Raw Flash to Flash-Based SSDs
- Internally, an SSD consists of some number of flash chips (for persistent storage). 
- An SSD also contains some amount of volatile (i.e., nonpersistent) memory (e.g., SRAM); such memory is useful for caching and buffering of data as well as for mapping tables. 
- Finally, an SSD contains control logic to orchestrate device operation.
	 ![[A Flash-based SSD.PNG|500]]
- One of the essential functions of this control logic is to satisfy client reads and writes, turning them into internal flash operations as need be. The **flash translation layer**, or **FTL**, provides exactly this functionality.
- The FTL takes read and write requests on logical blocks (that comprise the device interface) and turns them into low-level read, erase, and program commands on the underlying physical blocks and physical pages (that comprise the actual flash device).
- Excellent performance can be realized through a combination of techniques. One key will be to utilize multiple flash chips in **parallel**.
- Another performance goal will be to reduce **write amplification**, which is defined as the total write traffic (in bytes) issued to the flash chips by the FTL divided by the total write traffic (in bytes) issued by the client to the SSD.
- High reliability will be achieved through the combination of a few different approaches. One main concern is **wear out**. 
- The FTL should try to spread writes across the blocks of the flash as evenly as possible, ensuring that all of the blocks of the device wear out at roughly the same time; doing so is called **wear leveling**.
- Another reliability concern is program disturbance. To minimize such disturbance, FTLs will commonly program pages within an erased block **in order**, from low page to high page. This sequential-programming approach minimizes disturbance.
##### FTL Organization: A Bad Approach
- The simplest organization of an FTL would be something we call **direct mapped**.
- In this approach, a read to logical page N is mapped directly to a read of physical page N. A write to logical page N is more complicated. 
- The FTL first has to read in the entire block that page N is contained within; it then has to erase the block; finally, the FTL programs the old pages as well as the new one.
- As you can probably guess, the direct-mapped FTL has many problems, both in terms of performance as well as reliability. 
	- **The performance problems come on each write:** 
		- The device has to read in the entire block (costly), erase it (quite costly), and then program it (costly). 
		- The end result is severe write amplification and as a result, terrible write performance, even slower than typical hard drives with their mechanical seeks and rotational delays.
	- **The reliability problems of this approach:**
		- If file system metadata or user file data is repeatedly overwritten, the same block is erased and programmed, over and over, rapidly wearing it out and potentially losing data.
##### A Log-Structured FTL
- For these reasons, most FTLs today are **[[0x30_Log-structured File Systems|log structured]]**. Upon a write to logical block N, the device appends the write to the next free spot in the currently-beingwritten-to block; we call this style of writing **logging**.
- To allow for subsequent reads of block N, the device keeps a **mapping table**; this table stores the physical address of each logical block in the system.
- The **logical block addresses** are used by the client of the SSD (e.g., a file system) to remember where information is located.
- Internally, the device must transform these block writes into the erase and program operations supported by the raw hardware, and somehow record, for each logical block address, which **physical page** of the SSD stores its data.
- When the FTL writes logical block 100 to physical page 0, it records this fact in an **in-memory mapping table**.
	 ![[invalid SSD.PNG]]![[erased SSD.PNG]]![[writes to log structured SSD.PNG]]
- The FTL can now spread writes across all pages, performing what is called **wear leveling** and increasing the lifetime of the device.
- Unfortunately, this basic approach to log structuring has some downsides. 
	- The first is that overwrites of logical blocks lead to something we call **garbage**. 
	- The device has to periodically perform **[[0x30_Log-structured File Systems#A New Problem Garbage Collection|garbage collection]]**; excessive garbage collection drives up write amplification and lowers performance. 
	- The second is high cost of in-memory mapping tables; the larger the device, the more memory such tables need.
##### Garbage Collection
- Let’s assume that blocks `100` and `101` are written to again, with contents $c1$ and $c2$. 
- The writes are written to the next free pages, and the mapping table is updated accordingly. 
- Note that the device must have first erased block 1 to make such programming possible.
	 ![[rewriting in SSD.PNG]]
- The process of finding garbage blocks (also called **dead blocks**) and reclaiming them for future use is called **garbage collection**.
- The basic process is simple: 
	- Find a block that contains one or more garbage pages, read in the live (non-garbage) pages from that block. 
	- Write out those live pages to the log, and reclaim the entire block for use in writing.
- To do so, the device will:
	- Read live data (pages 2 and 3) from block 0.
	- Write live data to end of the log.
	- Erase block 0 (freeing it for later usage).
- For the garbage collector to function, there must be enough information within each block to enable the SSD to determine whether each page is live or dead. 
- One natural way to achieve this end is to store, at some location within each block, information about which logical blocks are stored within each page. 
- The device can then use the mapping table to determine whether each page within the block holds live data or not.
	 ![[after collecting garbage.PNG]]
##### Mapping Table Size
- With a large 1-TB SSD, for example, a single 4-byte entry per 4-KB page results in 1 GB of memory needed by the device, just for these mappings! Thus, this **page-level** FTL scheme is impractical.
###### Block-Based Mapping
- One approach to reduce the costs of mapping is to only keep a pointer per block of the device, instead of per page, reducing the amount of mapping information by a factor of $$\frac{Size_{block}}{Size_{page}}$$
- This **block-level** FTL is akin to having bigger page sizes in a virtual memory system; in that case, you use fewer bits for the VPN and have a larger offset in each virtual address.
- Unfortunately, using a block-based mapping inside a log-based FTL does not work very well for performance reasons. 
- The biggest problem arises when a **small write** occurs. In this case, the FTL must read a large amount of live data from the old block and copy it into a new one (along with the data from the small write). 
- This data copying increases write amplification greatly and thus decreases performance.
- The address mapping is slightly different than previous. Specifically, we think of the logical address space of the device as being chopped into chunks that are the size of the physical blocks within the flash. 
- Thus, the logical block address consists of two portions: a chunk number and an offset.
- Because we are assuming four logical blocks fit within each physical block, the offset portion of the logical addresses requires 2 bits; the remaining (most significant) bits form the chunk number.
- Logical blocks 2000, 2001, 2002, and 2003 all have the same chunk number (500), and have different offsets (0, 1, 2, and 3, respectively). Thus, with a block-level mapping, the FTL records that chunk 500 maps to block 1 (starting at physical page 4).
- In a block-based FTL, reading is easy. First, the FTL extracts the chunk number from the logical block address presented by the client, by taking the topmost bits out of the address. 
- Then, the FTL looks up the chunknumber to physical-page mapping in the table. 
- Finally, the FTL computes the address of the desired flash page by adding the offset from the logical address to the physical address of the block.
	 ![[Block-Based Mapping.PNG]]
- But what if the client writes to logical block 2002 (with contents `c’`)? 
- In this case, the FTL must read in 2000, 2001, and 2003, and then write out all four logical blocks in a new location, updating the mapping table accordingly. 
- Block 1 can then be erased and reused.
	 ![[Block-Based Mapping rewrite.PNG]]
- As you can see from this example, while block level mappings greatly reduce the amount of memory needed for translations, they cause significant performance problems when writes are smaller than the physical block size of the device. 
- As real physical blocks can be 256KB or larger, such writes are likely to happen quite often. Thus, a better solution is needed.
###### Hybrid Mapping
- With this approach, the FTL keeps a few blocks erased and directs all writes to them; these are called **log blocks**.
- Because the FTL wants to be able to write any page to any location within the log block without all the copying required by a pure block-based mapping, it keeps **per-page** mappings for these log blocks.
- The FTL thus logically has two types of mapping table in its memory: 
	- A small set of per-page mappings in what we’ll call the $log table$, and 
	- A larger set of per-block mappings in the $data table$.
- When looking for a particular logical block, the FTL will first consult the log table; if the logical block’s location is not found there, the FTL will then consult the data table to find its location and then access the requested data.
- The key to the hybrid mapping strategy is keeping the number of log blocks small. To keep the number of log blocks small, the FTL has to periodically examine log blocks (which have a pointer per page) and **switch** them into blocks that can be pointed to by only a single block pointer. 
- This switch is accomplished by one of three main techniques, based on the contents of the block.
- Let’s say the FTL had previously written out logical pages 1000, 1001, 1002, and 1003, and placed them in physical block 2.
	 ![[Hybrid Mapping data table.PNG]]
- Now assume that the client overwrites each of these blocks, in the exact same order, in one of the currently available log blocks, say physical block 0. 
	 ![[Hybrid Mapping rewrite.PNG]]
- Because these blocks have been written exactly in the same manner as before, the FTL can perform what is known as a **switch merge**.
- In this case, the log block (0) now becomes the storage location for blocks 0, 1, 2, and 3, and is pointed to by a single block pointer; the old block (2) is now erased and used as a log block. 
- In this best case, all the per-page pointers required replaced by a single block pointer.
	 ![[Hybrid Mapping final.PNG]]
- This switch merge is the best case for a hybrid FTL. Unfortunately, sometimes the FTL is not so lucky. 
- Imagine the case where we have the same initial conditions but then the client overwrites logical blocks 1000 and 1001.
- To reunite the other pages of this physical block, and thus be able to refer to them by only a single block pointer, the FTL performs what is called a **partial merge**.
- In this operation, logical blocks 1002 and 1003 are read from physical block 2, and then appended to the log. 
- The resulting state of the SSD is the same as the switch merge above; however, in this case, the FTL had to perform extra I/O to achieve its goals, thus increasing write amplification.
- The final case encountered by the FTL known as a **full merge**, and requires even more work. 
- In this case, the FTL must pull together pages from many other blocks to perform cleaning. For example, imagine that logical blocks 0, 4, 8, and 12 are written to log block $A$. 
- To switch this log block into a block-mapped page, the FTL must first create a data block containing logical blocks 0, 1, 2, and 3, and thus the FTL must read 1, 2, and 3 from elsewhere and then write out 0, 1, 2, and 3 together. 
- Next, the merge must do the same for logical block 4, finding 5, 6, and 7 and reconciling them into a single physical block. 
- The same must be done for logical blocks 8 and 12, and then (finally), the log block $A$ can be freed.
- Frequent full merges, as is not surprising, can seriously harm performance and thus should be avoided when at all possible.
###### Page Mapping Plus Caching
- Given the complexity of the hybrid approach, others have suggested simpler ways to reduce the memory load of page-mapped FTLs. 
- Probably the simplest is just to cache only the active parts of the FTL in memory, thus reducing the amount of memory needed.
- For example, if a given workload only accesses a small set of pages, the translations of those pages will be stored in the in-memory FTL, and performance will be excellent without high memory cost.
- Of course, the approach can also perform poorly. If memory cannot contain the **working set** of necessary translations, each access will minimally require an extra flash read to first bring in the missing mapping before being able to access the data itself.
- Even worse, to make room for the new mapping, the FTL might have to **evict** an old mapping, and if that mapping is **dirty** (i.e., not yet written to the flash persistently), an extra write will also be incurred.
- However, in many cases, the workload will display locality, and this caching approach will both reduce memory overheads and keep performance high.
# source
- Operating Systems: Three Easy Pieces - Chapter 44.
- [Lecture 14 - part 3](https://youtu.be/vvttbstRdj8).