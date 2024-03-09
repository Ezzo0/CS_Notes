# Explanation
- It was assumed that every address space of every running process fits into memory. We will now relax these big assumptions, and assume that we support many concurrently-running large address spaces.
- To support large address spaces, the OS will need a place to stash away portions of address spaces that currently aren’t in great demand.
- In modern systems, this role is usually served by a **hard disk drive**.
- A contrast is found in older systems that used **memory overlays**, which required programmers to manually move pieces of code or data in and out of memory as they were needed.
##### Swap Space
- Some space on the disk for moving pages back and forth is called **swap space**.
- To read from and write to the swap space, the OS will need to remember the **disk address** of a given page.
##### The Present Bit
- The way the hardware (or the OS, in a software-managed [[Faster Translations (TLBs)|TLB]] approach) determines this that the page is present in physical memory or not is through a new piece of information in each [[Introduction to Paging#What’s Actually In The Page Table?|page-table entry]], known as the **present bit**.
- If the present bit is set to one, it means the page is present in physical memory; if it is set to zero, the page is not in memory but rather on disk somewhere.
- The act of accessing a page that is not in physical memory is commonly referred to as a **page fault**.
- A particular piece of code, known as a **page-fault handler**, runs, and must service the page fault.
##### The Page Fault
- If a page is not present and has been swapped to disk, the OS will need to swap the page into memory in order to service the page fault.
- In many systems, the page table is a natural place to store such information.
- Thus, the OS could use the bits in the [[Introduction to Paging#What’s Actually In The Page Table?|PTE]] normally used for data such as the PFN of the page for a disk address.
- When the OS receives a page fault for a page, it looks in the PTE to find the address, and issues the request to disk to fetch the page into memory.
- When the disk I/O completes, the OS will then: 
	1. update the page table to mark the page as present.
	2. update the PFN field of PTE to record the in-memory location of the newly-fetched page. 
	3. update the TLB with the translation. 
	4. retry the instruction.
- While the I/O is in flight, the process will be in the **blocked** state.
##### What If Memory Is Full?
- If memory is full, the OS might like to first **page out** one or more pages to make room for the new page(s) the OS is about to bring in.
- The process of picking a page to **kick out, or replace** is known as the **page-replacement policy**.
##### When Replacements Really Occur
- It was assumed that the OS waits until memory is entirely full, and only then replaces (evicts) a page to make room for some other page.
- This is a little bit unrealistic, and there are many reasons for the OS to keep a small portion of memory free more proactively.
- To keep a small amount of memory free, most operating systems have some kind of **high watermark** (_HW_) and **low watermark** (_LW_) to help decide when to start evicting pages from memory.
- when the OS notices that there are fewer than _LW_ pages available, a background thread that is responsible for freeing memory runs.
- The thread evicts pages until there are _HW_ pages available. The background thread, sometimes called the **swap daemon** or **page daemon**, then goes to sleep.
- By performing a number of replacements at once, new performance optimizations become possible. 
- For example, many systems will **cluster** or **group** a number of pages and write them out at once to the swap partition, thus increasing the efficiency of the disk.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 21.
- [Lecture 5 - part 2](https://www.youtube.com/watch?v=4tPXkN5nRQs)