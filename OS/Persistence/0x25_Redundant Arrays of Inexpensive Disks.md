# Explanation
- The **Redundant Array of Inexpensive Disks**, better known as **RAID**, is a technique to use multiple [[0x24_Hard Disk Drives#A Simple Disk Drive|disks]] in concert to build a faster, bigger, and more reliable disk system.
- Externally, a RAID looks like a disk: a group of blocks one can read or write.
- Internally, the RAID is a complex beast, consisting of multiple disks, memory (both volatile and non-), and one or more processors to manage the system.
- RAIDs offer a number of advantages over a single disk. 
	1. One advantage is **performance**. Using multiple disks in parallel can greatly speed up I/O times. 
	2. Another benefit is **capacity**. Large data sets demand large disks. 
	3. Finally, RAIDs can improve **reliability**; spreading data across multiple disks (without RAID techniques) makes the data vulnerable to the loss of a single disk; with some form of **redundancy**, RAIDs can tolerate the loss of a disk and keep operating as if nothing were wrong.
##### Interface And RAID Internals
- When a file system issues a $logical$ I/O request to the RAID, the RAID internally must calculate which disk (or disks) to access in order to complete the request, and then issue one or more $physical$ I/Os to do so.
- As a simple example, consider a RAID that keeps two copies of each block (each one on a separate disk); when writing to such a **mirrored** RAID system, the RAID will have to perform two physical I/Os for every one logical I/O it is issued.
##### Fault Model
- The first fault model we will assume is quite simple, and has been called the **fail-stop** fault model.
- In this model, a disk can be in exactly one of two states: working or failed. With a working disk, all blocks can be read or written. In contrast, when a disk has failed, we assume it is permanently lost.
- One critical aspect of the fail-stop model is what it assumes about fault detection.
- Specifically, when a disk has failed, we assume that this is easily detected.
##### How To Evaluate A RAID
- We will evaluate each RAID design along three axes. 
	1. The first axis is **capacity**
	2. The second axis of evaluation is **reliability**. How many disk faults can the given design tolerate?
	3. The third axis is **performance**.
##### RAID Level 0: Striping
- The first RAID level is actually not a RAID level at all, in that there is no redundancy.
- However, RAID level 0, or striping as it is better known, serves as an excellent upper-bound on performance and capacity and thus is worth understanding.
- The simplest form of striping will stripe blocks across the disks of the system as follows
	 ![[RAID-0.png|450]]
- The basic idea: spread the blocks of the array across the disks in a round-robin fashion. 
- This approach is designed to extract the most parallelism from the array when requests are made for contiguous chunks of the array.
- We call the blocks in the same row a **stripe**; thus, blocks 0, 1, 2, and 3 are in the same stripe above.
- This arrangement need not be the case. For example, we could arrange the blocks across disks as:
	 ![[Striping.png|500]]
- In this example, we place two 4KB blocks on each disk before moving on to the next disk. Thus, the **chunk size** of this RAID array is 8KB, and a stripe thus consists of 4 chunks or 32KB of data.
- Chunk size mostly affects performance of the array.
	- A small chunk size implies that many files will get striped across many disks, thus increasing the parallelism of reads and writes to a single file.
	- However, the positioning time to access blocks across multiple disks increases, because the positioning time for the entire request is determined by the maximum of the positioning times of the requests across all drives.
	- A big chunk size reduces such intra-file parallelism, and thus relies on multiple concurrent requests to achieve high throughput.
	- However, large chunk sizes reduce positioning time; if, for example, a single file fits within a chunk and thus is placed on a single disk, the positioning time incurred while accessing it will just be the positioning time of a single disk.
##### RAID-0 Analysis:
- From the perspective of capacity, it is perfect: given $N$ disks each of size $B$ blocks, striping delivers $N · B$ blocks of useful capacity.
- From the standpoint of reliability, striping is also perfect, but in the bad way: any disk failure will lead to data loss.
##### Evaluating RAID-0 Performance:
- In analyzing RAID performance, one can consider two different performance metrics:
	1. The first is **single-request latency**.
		- Understanding the latency of a single I/O request to a RAID is useful as it reveals how much parallelism can exist during a single logical I/O operation.
		- The latency of a single-block request should be just about identical to that of a single disk; after all, RAID-0 will simply redirect that request to one of its disks.
	2. The second is **steady-state throughput** of the RAID, i.e., the total bandwidth of many concurrent requests.
		- We will assume, for this discussion, that there are two types of workloads: **sequential** and **random**.
		- We will assume that a disk can transfer data at $S$ MB/s under a sequential workload, and $R$ MB/s when under a random workload. In general, S is much greater than R (i.e., S ≫ R).
		- From the perspective of steady-state sequential throughput, we’d expect to get the full bandwidth of the system. 
		- Thus, throughput equals N (the number of disks) multiplied by S (the sequential bandwidth of a single disk).
		- For a large number of random I/Os, we can again use all of the disks, and thus obtain $N · R$ MB/s.
##### RAID Level 1: Mirroring
- With a mirrored system, we simply make more than one copy of each block in the system; each copy should be placed on a separate disk, of course. By doing so, we can tolerate disk failures.
	 ![[RAID-1.png|450]]
- The arrangement above is a common one and is sometimes called **RAID-10** (or **RAID 1+0, stripe of mirrors**) because it uses mirrored pairs (RAID-1) and then stripes (RAID-0) on top of them (will be used).
- Another common arrangement is **RAID-01** (or **RAID 0+1, mirror of stripes**), which contains two large striping (RAID-0) arrays, and then mirrors (RAID-1) on top of them.
- When reading a block from a mirrored array, the RAID has a choice: it can read either copy.
- When writing a block, though, no such choice exists: the RAID must update both copies of the data, in order to preserve reliability (these writes can take place in parallel).
##### RAID-1 Analysis
- From a capacity standpoint, RAID-1 is expensive; with the mirroring level = 2, we only obtain half of our peak useful capacity.
- With N disks of B blocks, RAID-1 useful capacity is $\frac{(N · B)}{2}$.
- From a reliability standpoint, It can tolerate the failure of any one disk.
- From the perspective of the latency of a single read request, we can see it is the same as the latency on a single disk; all the RAID-1 does is direct the read to one of its copies.
- A write is a little different: it requires two physical writes to complete before it is done. 
- These two writes happen in parallel, and thus the time will be roughly equivalent to the time of a single write.
- However, because the logical write must wait for both physical writes to complete, it suffers the worst-case seek and rotational delay of the two requests, and thus (on average) will be slightly higher than a write to a single disk.
- When writing out to disk sequentially, each logical write must result in two physical writes. Thus, we can conclude that the maximum bandwidth obtained during sequential writing to a mirrored array is ( $\frac{N}{2} · S$), or half the peak bandwidth.
- Unfortunately, we obtain the exact same performance during a sequential read.
	- To see that this is not (necessarily) the case, however, consider therequests a single disk receives (say disk 0). 
	- First, it gets a request for block 0; then, it gets a request for block 4 (skipping block 2). 
	- In fact, each disk receives a request for every other block. While it is rotating over the skipped block, it is not delivering useful bandwidth to the client. 
	- Thus, each disk will only deliver half its peak bandwidth.
- Random reads are the best case for a mirrored RAID. In this case, we can distribute the reads across all the disks, and thus obtain the full possible bandwidth. Thus, for random reads, RAID-1 delivers $N · R$ MB/s.
- Random writes perform as you might expect: $\frac{N}{2} · R$ MB/s. Each logical write must turn into two physical writes, and thus while all the disks will be in use, the client will only perceive this as half the available bandwidth.
##### RAID Level 4: Saving Space With Parity
- Parity-based approaches attempt to use less capacity and thus overcome the huge space penalty paid by mirrored systems.
- For each stripe of data, we have added a single parity block that stores the redundant information for that stripe of blocks.
- To compute parity, we need to use a mathematical function **XOR**.
	 ![[RAID-4.png|500]]
- The number of 1s in any row, including the parity bit, must be an even (not odd) number; that is the **invariant** that the RAID must maintain in order for parity to be correct.
- Imagine the column labeled C2 is lost. To figure out what values must have been in the column, we simply have to read in all the other values in that row (including the XOR’d parity bit) and **reconstruct** the right answer.
- A simple way to compute the reconstructed value: we just XOR the data bits and the parity bits together.
	 ![[xor.png|450]]
- Bitwise XOR is performed across each bit of the data blocks and the result of each bitwise XOR is put into the corresponding bit slot in the parity block.
##### RAID-4 Analysis
- From a capacity standpoint, RAID-4 uses one disk for parity information for every group of disks it is protecting. Thus, our useful capacity for a RAID group is $(N − 1) · B$.
- From a Reliability standpoint, RAID-4 tolerates one disk failure and no more. If more than one disk is lost, there is simply no way to reconstruct the lost data.
- Sequential read performance can utilize all of the disks except for the parity disk, and thus deliver a peak effective bandwidth of $(N − 1) · S$ MB/s.
- When writing a big chunk of data to disk, RAID-4 can perform a simple optimization known as a **full-stripe write**.
	 ![[Full-stripe.png|450]]
- In this case, the RAID can simply calculate the new value of P0 (by performing an XOR across the blocks 0, 1, 2, and 3) and then write all of the blocks (including the parity block) to the five disks above in parallel.
- Once we understand the full-stripe write, the he effective bandwidth of sequential writes is also $(N − 1) · S$ MB/s.
- Even though the parity disk is constantly in use during the operation, the client does not gain performance advantage from it.
- A set of 1-block random reads will be spread across the data disks of the system but not the parity disk. Thus, the effective performance is: $(N − 1) · R$ MB/s.
- Imagine we wish to overwrite block 1 in the example above. 
- We could just go ahead and overwrite it, but that would leave us with a problem: the parity block P0 would no longer accurately reflect the correct parity value of the stripe; in this example, P0 must also be updated. 
- How can we update it both correctly and efficiently?
- There are two methods: 
	1. The first, known as **additive parity**
		- To compute the value of the new parity block, read in all of the other data blocks in the stripe in parallel and XOR those with the new block (1).
		- The problem with this technique is that it scales with the number of disks, and thus in larger RAIDs requires a high number of reads to compute parity.
	2. The **subtractive parity** method
		- Let’s imagine that we wish to overwrite C2 with a new value which we will call $C2_{new}$ .
		- The subtractive method works in three steps. 
			1. First, we read in the old data at C2 and the old parity.
			2. Then, we compare the old data and the new data; if they are the same, then we know the parity bit will also remain the same.
			3. If, however, they are different, then we must flip the old parity bit to the opposite of its current state.
			 ![[subtractive parity.png|400]]
- For this performance analysis, let us assume we are using the subtractive method.
- Imagine there were 2 small writes submitted to the RAID-4 at about the same time, to blocks 4 and 13.
- The data for those disks is on disks 0 and 1, and thus the read and write to data could happen in parallel, which is good. 
- The problem that arises is with the parity disk; both the requests have to read the related parity blocks for 4 and 13, parity blocks 1 and 3.
- The parity disk is a bottleneck under this type of work-load; we sometimes thus call this the **small-write problem** for parity-based RAIDs.
- Because the parity disk has to perform two I/Os (one read, one write) per logical I/O, we can compute the performance of small random writes in RAID-4 by computing the parity disk’s performance on those two I/Os, and thus we achieve $\frac{R}{2}$ MB/s.
- As you now know, a single read (assuming no failure) is just mapped to a single disk, and thus its latency is equivalent to the latency of a single disk request. 
- The latency of a single write requires two reads and then two writes; the reads can happen in parallel, as can the writes, and thus total latency is about twice that of a single disk.
##### RAID Level 5: Rotating Parity
- To address the small-write problem RAID-5 was introduced.
- RAID-5 works almost identically to RAID-4, except that it **rotates** the parity block across drives.
	 ![[RAID-5.png|500]]
##### RAID-5 Analysis
- The effective capacity and failure tolerance of the two levels are identical to RAID-4. So are sequential read and write performance.
- The latency of a single request (whether a read or a write) is also the same as RAID-4.
- Random read performance is a little better, because we can now utilize all disks.
- Random write performance improves noticeably over RAID-4, as it allows for parallelism across requests.
- Random write performance improves noticeably over RAID-4, as it allows for parallelism across requests.
- Imagine a write to block 1 and a write to block 10; this will turn into requests to disk 1 and disk 4 and requests to disk 0 and disk 2. Then our total bandwidth for small writes will be $\frac{N}{4} · R$ MB/s.
- The factor of four loss is due to the fact that each RAID-5 write still generates 4 total I/O operations, which is simply the cost of using parity-based RAID.
##### RAID Comparison: A Summary
  ![[RAID.png]]
# Sources
- Operating Systems: Three Easy Pieces - Chapter 38.
- [Lecture 11 - part 1](https://youtu.be/XF0mKxLrSVs)
- [Lecture 11 - part 2](https://youtu.be/h3WKYo1B19U)
- [Lecture 11 - part 3](https://youtu.be/Mn9g9XWec28)