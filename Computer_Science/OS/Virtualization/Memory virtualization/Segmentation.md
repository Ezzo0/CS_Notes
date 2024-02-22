# Explanation
- With the [[Address Translation#Dynamic Relocation (base and bounds)|base and bounds registers]], the OS can easily relocate processes to different parts of physical memory. 
- However, you might have noticed something about these address spaces of ours: there is a big chunk of “free” space right in the middle, between the stack and the heap.
- This type of waste is usually called **internal fragmentation**, as the space inside the allocated unit is not all used (i.e., is fragmented) and thus wasted.
##### Segmentation: Generalized Base/Bounds
- The idea is simple: instead of having just one base and bounds pair in our MMU, why not have a base and bounds pair per logical segment of the address space?
- This idea is called **segmentation**.
- A segment is just a contiguous portion of the address space of a particular length.
- Segmentation allows the OS to place each one of those segments in different parts of physical memory, and thus avoid filling physical memory with unused virtual address space.
	 ![[Placing Segments In Physical Memory.png|350]]
- In this case, a set of three base and bounds register pairs ![[Segment Register Values.png|300]]
- let’s look at an address in the heap, virtual address 4200. If we just add the virtual address 4200 to the base of the heap (34KB), we get a physical address of 39016, which is not the correct physical address.
- What we need to first do is extract the offset into the heap, i.e., which byte(s) in this segment the address refers to. Because the heap starts at virtual address 4KB (4096), the offset of 4200 is actually 4200 minus 4096, or 104. We then take this offset (104) and add it to the base register physical address (34K) to get the desired result: 34920.
	 ![[A Process And Its Address Space.png|400]]
##### Which Segment Are We Referring To?
- One common approach, sometimes referred to as an **explicit** approach, is to chop up the address space into segments based on the top few bits of the virtual address.
- In our example above, we have three segments; thus we need two bits to accomplish our task. If we use the top two bits of our 14-bit virtual address to select the segment, our virtual address looks like this:![[virtual address.png|350]]
- You may also have noticed that when we use the top two bits, and we only have three segments (code, heap, stack), one segment of the address space goes unused. 
- To fully utilize the virtual address space (and avoid an unused segment), some systems put code in the same segment as the heap and thus use only one bit to select which segment to use.
- There are other ways for the hardware to determine which segment a particular address is in.
- In the implicit approach, the hardware determines the segment by noticing how the address was formed.
	- If, for example, the address was generated from the program counter (i.e., it was an instruction fetch), then the address is within the code segment. 
	- If the address is based off of the stack or base pointer, it must be in the stack segment; any other address must be in the heap.
##### What About The Stack?
- The first thing we need is a little extra hardware support. Instead of just base and bounds values, the hardware also needs to know which way the segment grows (a bit, for example, that is set to 1 when the segment grows in the positive direction, and 0 for negative).![[Segment Registers.png]]
##### Support for Sharing
- To support sharing, we need a little extra support from the hardware, in the form of protection bits. 
- Basic support adds a few bits per segment, indicating whether or not a program can read or write a segment, or execute code that lies within the segment. 
- By setting a code segment to read-only, the same code can be shared across multiple processes, without worry of harming isolation.
- In addition to checking whether a virtual address is within bounds, the hardware also has to check whether a particular access is permissible.
- If a user process tries to write to a read-only segment, or execute from a non-executable segment, the hardware should raise an exception.
##### Fine-grained vs. Coarse-grained Segmentation
- Dividing the address space into relatively large, coarse chunks, is known as **coarse-grained**.
- However, some early systems were more flexible and allowed for address spaces to consist of a large number of smaller segments, referred to as **fine-grained**segmentation.
	- Supporting many segments requires even further hardware support, with a **segment table** of some kind stored in memory.
##### OS Support
- When a new address space is created, the OS has to be able to find space in physical memory for its segments. 
- Previously, we assumed that each address space was the same size, and thus physical memory could be thought of as a bunch of slots where processes would fit in. 
- Now, we have a number of segments per process, and each segment might be a different size.
- The general problem that arises is that physical memory quickly becomes full of little holes of free space, making it difficult to allocate new segments, or to grow existing ones. We call this problem **external fragmentation**.
- One solution to this problem would be to **compact** physical memory by rearranging the existing segments.![[Non-compacted and Compacted Memory.png]] ^130c27
- However, **compaction** is expensive, as copying segments is memory-intensive and generally uses a fair amount of processor time.
- Compaction also (ironically) makes requests to grow existing segments hard to serve, and may thus cause further rearrangement to accommodate such requests.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 16.
- [Lecture 3 - part 3](https://youtu.be/0WVoWlOT-kY)