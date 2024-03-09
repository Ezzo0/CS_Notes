# Explanation
- Managing free space is easy when the space you are managing is divided into fixed-sized units. 
- In such a case, you just keep a list of these fixed-sized units; when a client requests one of them, return the first entry.
- Free-space management becomes more difficult is when the free space you are managing consists of variable-sized units. 
- This arises in a user-level memory-allocation library (as in `malloc()` and `free()`) and in an OS managing physical memory when using **[[Segmentation]]** to implement virtual memory.
##### Assumptions
- We’ll also assume that once memory is handed out to a client, it cannot be relocated to another location in memory. Thus, no **[[Segmentation#^130c27|compaction]]** of free space is possible
- We’ll assume that the allocator manages a contiguous region of bytes (single fixed size throughout its life).
##### Low-level Mechanisms
- The space that memory allocation library manages is known historically as the heap, and the generic data structure used to manage free space in the heap is some kind of **free list**. 
###### Splitting and Coalescing
- This structure contains references to all of the free chunks of space in the managed region of memory.
	 ![[heap.png|300]] ![[free_list.png|300]]

- Assume we have a request for just a single byte of memory. In this case, the allocator will perform an action known as **splitting**. 
- It will find a free chunk of memory that can satisfy the request and split it into two.
	 ![[free_list2.png|400]]
- A corollary mechanism found in many allocators is known as **coalescing** of free space.
- If an application calls `free(10)` and is added to our free list, the problem will be that while the entire heap is now free, it is seemingly divided into three chunks of 10 bytes each. 
- Thus, if a user requests 20 bytes, a simple list traversal will not find such a free chunk, and return failure. ![[free_heap_list.png|500]]
- What allocators do in order to avoid this problem is coalesce free space when a chunk of memory is freed. ![[freeheap.png|500]]
###### Tracking The Size Of Allocated Regions
- Most allocators store a little bit of extra information in a header block which is kept in memory, usually just before the handed-out chunk of memory.![[An Allocated Region Plus Header.png|500]]
- The header minimally contains the size of the allocated region; it may also contain additional pointers to speed up deallocation, a magic number to provide additional integrity checking, and other information. Let’s assume a simple header which contains the size of the region and a magic number, like this:
	```c
	typedef struct {
		int size;
		int magic;
	} header_t;
	```
	 ![[Specific Contents Of The Header.png|400]]
- When the user calls `free(ptr)`, the library then uses simple pointer arithmetic to figure out where the header begins:
	```c
		void free(void *ptr) {
			header_t *hptr = (header_t *) ptr - 1;
			...
	```
- Note that: the size of the free region is the size of the header plus the size of the space allocated to the user. 
- Thus, when a user requests N bytes of memory, the library does not search for a free chunk of size N; rather, it searches for a free chunk of size N plus the size of the header.
###### Embedding A Free List
- In a more typical list, when allocating a new node, you would just call `malloc()` when you need space for the node. Unfortunately, within the memory-allocation library, you can’t do this! Instead, you need to build the list inside the free space itself.
- Assume we have a 4096-byte chunk of memory to manage. To manage this as a free list, we first have to initialize said list; initially, the list should have one entry, of size 4096 (minus the header size). Here is the description of a node of the list:
	```c
	typedef struct __node_t {
		int size;
		struct __node_t *next;
	} node_t;
	```
- We are assuming that the heap is built within some free space acquired via a call to the system call `mmap();` this is not the only way to build such a heap but serves us well in this example. Here is the code:
	```c
	// mmap() returns a pointer to a chunk of free space
	node_t *head = mmap(NULL, 4096, PROT_READ|PROT_WRITE,
						MAP_ANON|MAP_PRIVATE, -1, 0);
	head->size = 4096 - sizeof(node_t);
	head->next = NULL;
	```
	 ![[A Heap With One Free Chunk.png|500]]
##### Basic Strategies
- The **best fit** strategy is quite simple: first, search through the free list and find chunks of free memory that are as big or bigger than the requested size. Then, return the one that is the smallest in that group of candidates.
- The intuition behind best fit is simple: by returning a block that is close to what the user asks, best fit tries to reduce wasted space. 
- However, there is a cost; naive implementations pay a heavy performance penalty when performing an exhaustive search for the correct free block.
- The **worst fit** approach is the opposite of best fit; find the largest chunk, return the requested amount, and keep the remaining (large) chunk on the free list.
- Most studies show that it performs badly, leading to excess fragmentation while still having high overheads.
- The **first fit** method simply finds the first block that is big enough and returns the requested amount to the user.
- First fit has the advantage of speed — no exhaustive search of all the free spaces are necessary — but sometimes pollutes the beginning of the free list with small objects.
- Thus, how the allocator manages the free list’s order becomes an issue. One approach is to use **address-based ordering**; by keeping the list ordered by the address of the free space, coalescing becomes easier, and fragmentation tends to be reduced.
- Instead of always beginning the first-fit search at the beginning of the list, the **next fit** algorithm keeps an extra pointer to the location within the list where one was looking last. 
- The idea is to spread the searches for free space throughout the list more uniformly, thus avoiding splintering of the beginning of the list. 
- The performance of such an approach is quite similar to first fit, as an exhaustive search is once again avoided.
##### Other Approaches
###### Segregated Lists
- The basic idea is simple: if a particular application has one (or a few) popular-sized request that it makes, keep a separate list just to manage objects of that size; all other requests are forwarded to a more general memory allocator.
- The benefits of such an approach are obvious. By having a chunk of memory dedicated for one particular size of requests, fragmentation is much less of a concern; moreover, allocation and free requests can be served quite quickly when they are of the right size, as no complicated search of a list is required.
- This approach introduces new complications into a system as well. For example, how much memory should one dedicate to the pool of memory that serves specialized requests of a given size, as opposed to the general pool?
###### Buddy Allocation
- In such a system, free memory is first conceptually thought of as one big space of size $2^N$ .
- When a request for memory is made, the search for free space recursively divides free space by two until a block that is big enough to accommodate the request is found. 
	 ![[Buddy-managed.png|500]]
- This scheme can suffer from **internal fragmentation**, as you are only allowed to give out power-of-two-sized blocks.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 17.
- [Lecture 3 - part 3](https://youtu.be/0WVoWlOT-kY)