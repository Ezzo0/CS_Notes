# Explanation
- The second problem that [[Introduction to Paging|paging]] introduces: page tables are too big and thus consume too much memory.
- Assume a 32-bit address space ($2^{32}$ bytes), with 4KB ($2^{12}$ byte) pages and a 4-byte page-table entry.
- An address space thus has roughly one million virtual pages in it; multiply by the page-table entry size and you see that our page table is 4MB in size.
- We usually have one page table for every process in the system! With a hundred active processes, we will be allocating hundreds of megabytes of memory just for page tables!
##### Simple Solution: Bigger Pages
- The major problem with this approach, however, is that big pages lead to waste within each page, a problem known as **internal fragmentation**. 
- Applications thus end up allocating pages but only using little bits and pieces of each, and memory quickly fills up with these overly-large pages.
##### Hybrid Approach: Paging and Segments
- The hybrid approach is that instead of having a single page table for the entire address space of the process, let's have one per logical segment.
- Remember with segmentation, we had a **[[Segmentation#Segmentation Generalized Base/Bounds|base]]** register that told us where each segment lived in physical memory, and a **[[Segmentation#Segmentation Generalized Base/Bounds|bound]]** or limit register that told us the size of said segment.
- In our hybrid, we still have those structures in the MMU; here, we use the base not to point to the segment itself but rather to hold the physical address of the page table of that segment. 
- The bounds register is used to indicate the end of the page table (i.e., how many valid pages it has).
	 ![[Paging and Segments.png|400]]
- To determine which segment an address refers to, we’ll use the top two bits of the address space.![[hybridVA.png]]
- However, as you might notice, this approach is not without problems.
	1. [[Segmentation]] is not quite as flexible as we would like, as it assumes a certain usage pattern of the address space; if we have a large but sparsely-used heap, for example, we can still end up with a lot of page table waste. 
	2. This hybrid causes external fragmentation to arise again. While most of memory is managed in page-sized units, page tables now can be of arbitrary size (in multiples of [[Introduction to Paging#What’s Actually In The Page Table?|PTE]]s). Thus, finding free space for them in memory is more complicated.
##### Multi-level Page Tables
- Multi-levels page table turns the linear page table into something like a tree.
- The basic idea behind a multi-level page table is:
	- First, chop up the page table into page-sized units. 
	- Then, if an entire page of page-table entries (PTEs) is invalid, don’t allocate that page of the page table at all.
- To track whether a page of the page table is valid (and if valid, where it is in memory), use a new structure, called the **page directory**.
	 ![[Linear and Multi-Level Page Tables.png]]
- The page directory, in a simple two-level table, contains one entry per page of the page table.
- It consists of a number of page directory entries(**PDE**).
- A PDE (minimally) has a **valid bit** and a **page frame number (PFN)**, similar to a [[Introduction to Paging#What’s Actually In The Page Table?|PTE]].
- If the PDE is valid, it means that at least one of the pages of the page table that the entry points to (via the PFN) is valid.
- Multi-level table only allocates page-table space in proportion to the amount of address space you are using; thus it is generally compact and supports sparse address spaces.
- If multi-level table carefully constructed, each portion of the page table fits neatly within a page, making it easier to manage memory; the OS can simply grab the next free page when it needs to allocate or grow a page table.
- There is a cost to multi-level tables; on a [[Faster Translations (TLBs)|TLB]] miss, two loads from memory will be required to get the right translation information from the page table (one for the page directory, and one for the [[Introduction to Paging#What’s Actually In The Page Table?|PTE]] itself), in contrast to just one load with a linear page table.
- VPN is as follows:![[Page Directory VPN.png|600]]
- Note that the page-frame number obtained from the page-directory entry must be left-shifted into place before combining it with the page- table index to form the address of the PTE.
- An even more extreme space savings in the world of page tables is found with **inverted page tables**.
- Instead of having many page tables (one per process of the system), we keep a single page table that has an entry for each physical page of the system. 
- The entry tells us which process is using this page, and which virtual page of that process maps to this physical page.
- Finding the correct entry is now a matter of searching through this data structure. A linear scan would be expensive, and thus a [[Hash Tables|hash table]] is often built over the base structure to speed up lookups.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 20.
- [Lecture 5 - part 1](https://www.youtube.com/watch?v=ggPkFxOTwHY)