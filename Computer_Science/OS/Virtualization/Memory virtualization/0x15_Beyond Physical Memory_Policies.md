# Explanation
##### A Simple Policy: FIFO
- Pages are placed in a queue when they enter the system; when a replacement occurs, the page on the tail of the queue (the “first-in” page) is evicted.
- FIFO simply can’t determine the importance of blocks.
- Comparing FIFO to optimal, FIFO does notably worse: a 36.4% hit rate (or 57.1% excluding compulsory misses). 
##### Another Simple Policy: Random
- Random policy picks a random page to replace under memory pressure.
- Random does a little better than FIFO, and a little worse than optimal. As it depend on **luck**, sometimes it does much worse.
##### Using History: LRU
- One type of historical information a page-replacement policy could use is **frequency**.
- If a page has been accessed many times, perhaps it should not be replaced as it clearly has some value.
- A more commonly-used property of a page is its **recency** of access; the more recently a page has been accessed, perhaps the more likely it will be accessed again.
- This family of policies is based on what people refer to as the **principle of locality**, which basically is just an observation about programs and their behavior.
- The **Least-Frequently-Used (LFU)** policy replaces the least-frequently-used page when an eviction must take place.
- The **Least-Recently-Used (LRU)** policy replaces the least-recently-used page.
	 ![[Memory Policies.png|400]]
##### Implementing Historical Algorithms
- To keep track of which pages have been least- and most-recently used, the system has to do some accounting work on every memory reference. 
- Clearly, without great care, such accounting could greatly reduce performance.
- One method that could help speed this up is to add a little bit of hardware support
- For example, a machine could update, on each page access, a time field in memory (for example, this could be in the per-process [[0x11_Introduction to Paging#^6dd76f|page table]], or just in some separate array in memory, with one entry per physical page of the system).
- Thus, when a page is accessed, the time field would be set, by hardware, to the current time. 
- Then, when replacing a page, the OS could simply scan all the time fields in the system to find the least-recently-used page.
- Unfortunately, as the number of pages in a system grows, scanning a huge array of times just to find the absolute least-recently-used page is prohibitively expensive. 
##### Approximating LRU
- It is possible to approximate LRU and still obtain the desired behavior.
- The idea requires some hardware support, in the form of a **use bit** (sometimes called the **reference bit**).
- Whenever a page is referenced (i.e., read or written), the use bit is set by hardware to 1. 
- The hardware never clears the bit, though; that is the responsibility of the OS.
- The **clock algorithm** is one simple approach that the use OS employ the **use bit** to approximate LRU.
- Imagine all the pages of the system arranged in a circular list. 
- A **clock hand** points to some particular page to begin with (it doesn’t really matter which). 
- When a replacement must occur, the OS checks if the currently-pointed to page P has a use bit of 1 or 0. 
- If 1, this implies that page P was recently used and thus is not a good candidate for replacement. 
- Thus, the use bit for P is set to 0 (cleared), and the clock hand is incremented to the next page (P + 1). 
- The algorithm continues until it finds a use bit that is set to 0, implying this page has not been recently used (or, in the worst case, that all pages have been and that we have now searched through the entire set of pages, clearing all the bits).
##### Considering Dirty Pages
- One small modification to the clock algorithm is the additional consideration of whether a page has been modified or not while in memory.
- If a page has been **modified** and is thus **dirty**, it must be written back to disk to evict it, which is expensive. 
- If it has not been modified (and is thus **clean**), the eviction is free; the physical frame can simply be reused for other purposes without additional I/O. 
- Thus, some VM systems prefer to evict clean pages over dirty pages.
- To support this behavior, the hardware should include a **modified bit** (or **dirty bit**).
- This bit is set any time a page is written, and thus can be incorporated into the page-replacement algorithm. 
- The clock algorithm, for example, could be changed to scan for pages that are both unused and clean to evict first; failing to find those, then for unused pages that are dirty, and so forth.
##### Other VM Policies
- Page replacement is not the only policy the VM subsystem employs.
- The OS also has to decide when to bring a page into memory.
- This policy, sometimes called the **page selection** policy.
- For most pages, the OS simply uses **demand paging**, which means the OS brings the page into memory when it is accessed. 
- The OS could guess that a page is about to be used, and thus bring it in ahead of time; this behavior is known as **prefetching**.
- Another policy determines how the OS writes pages out to disk.
- Many systems instead collect a number of pending writes together in memory and write them to disk in one (more efficient) write.
- This behavior is usually called **clustering** or simply **grouping** of writes, and is effective because of the nature of disk drives, which perform a single large write more efficiently than many small ones.
##### Thrashing
- **Thrashing** is known as the system will constantly be paging.
- It happens when memory is simply oversubscribed, and the memory demands of the set of running processes simply exceeds the available physical memory.
- Some earlier operating systems had a fairly sophisticated set of mechanisms to both detect and cope with thrashing when it took place. 
- For example, given a set of processes, a system could decide not to run a subset of processes, with the hope that the reduced set of processes’ working sets (the pages that they are using actively) fit in memory and thus can make progress. 
- This approach, generally known as **admission control**.
- Some current systems take more a draconian approach to memory overload. 
- For example, some versions of Linux run an **out-of-memory killer** when memory is oversubscribed. 
- This daemon chooses a memory- intensive process and kills it, thus reducing memory in a none-too-subtle manner. 
- While successful at reducing memory pressure, this approach can have problems, if, for example, it kills the X server and thus renders any applications requiring the display unusable.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 22.
- [Lecture 5 - part 2](https://www.youtube.com/watch?v=4tPXkN5nRQs)