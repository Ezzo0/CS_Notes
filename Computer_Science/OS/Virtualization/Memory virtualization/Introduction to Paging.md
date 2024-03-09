# Explanation
- The operating system takes one of two approaches when solving most any space-management problem:
	1. Chop things up into variable-sized pieces, as we saw with [[Segmentation]] in virtual memory.
		- This solution has inherent difficulties. In particular, when dividing a space into different-size chunks, the space itself can become **fragmented**, and thus allocation becomes more challenging over time. 
	2. Chop up space into _fixed-sized_ pieces.
		- This idea is called **paging**.
		- Instead of splitting up a process’s address space into some number of variable-sized logical segments (e.g., code, heap, stack), we divide it into fixed-sized units, each of which is called a **page**.
		- Physical memory is viewed as an array of fixed-sized slots called **page frames**; each of these frames can contain a single virtual-memory page.
			![[paging.png|500]]
- Paging has a number of advantages over our previous approaches:
	1. **flexibility**: for example, make assumptions about the direction the heap and stack grow and how they are used.
	2. **simplicity** of free-space management: For example, when the OS wishes to place 64-byte address space into our eight-page physical memory, it simply finds 4 free pages; perhaps the OS keeps a **[[Free-Space Management#Low-level Mechanisms|free list]]** of all free pages for this, and just grabs the first four free pages off of this list.
- To record where each virtual page of the address space is placed in physical memory, the operating system usually keeps a _per-process_ data structure known as a **page table**.
- The major role of the page table is to store **address translations** for each of the virtual pages of the address space.
- Each process has its page table.
- To translate this virtual address that the process generated, we have to first split it into two components: the **virtual page number (VPN)**, and the **offset** within the page. 
- For example: If page is 16 byte and there are 4 pages, VPN and offset will be:![[16byte_per_page_and_4pages.png|400]] ![[The Address Translation Process.png|500]]
##### Where Are Page Tables Stored?
- Because page tables are so big, we don’t keep any special on-chip hardware in the MMU to store the page table of the currently-running process.
- Instead, we store the page table for each process in _memory_ somewhere.
##### What’s Actually In The Page Table?
- The simplest form is called a **linear page table**, which is just an array.
- The OS indexes the array by the virtual page number (VPN), and looks up the page-table entry (PTE) at that index in order to find the desired physical frame number (PFN).
- The content of PTE is:
	1. **Valid bit**: Indicate whether the particular translation is valid or not.
		- when a program starts running, it will have code and heap at one end of its address space, and the stack at the other. 
		- All the unused space in-between will be marked **invalid**, and if the process tries to access such memory, it will generate a trap to the OS which will likely terminate the process.
		- By simply marking all the unused pages in the address space invalid, we remove the need to allocate physical frames for those pages and save a great deal of memory.
	2. **Protection bits**: Indicate whether the page could be read from, written to, or executed from.
		- Accessing a page in a way not allowed by these bits will generate a trap to the OS. 
	3. **Present bit**: Indicates whether this page is in physical memory or on disk (i.e., it has been **swapped out**).
	4. **Dirty bit**: Indicates whether the page has been modified since it was brought into memory.
	5. **Reference bit** (a.k.a. **accessed bit**): used to track whether a page has been accessed. 
		-  It is useful in determining which pages are popular and thus should be kept in memory; such knowledge is critical during **page replacement**.
	![[PTE.png]]
##### Paging: Also Too Slow
- Paging requires us to perform one extra memory reference in order to first fetch the translation from the page table.
- Extra memory references are costly, and in this case will likely slow down the process by a factor of two or more.
- Without careful design of both hardware and software, page tables will cause the system to run too slowly, as well as take up too much memory.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 18.
- [Lecture 4 - part 1](https://youtu.be/wAx_h3HkIX0)
- [Lecture 4 - part 2](https://youtu.be/7BOXM2XgGO4)