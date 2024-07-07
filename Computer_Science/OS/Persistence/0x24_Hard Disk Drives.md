# Explanation
- The **hard disk drives** have been the main form of persistent data storage in computer systems for decades and much of the development of file system technology is predicated on their behavior. 
- Thus, it is worth understanding the details of a disk’s operation before building the file system software that manages it.
##### The Interface
- The basic interface for all modern drives is straightforward. 
- The drive consists of a large number of sectors (512-byte blocks), each of which can be read or written.
- Thus, we can view the disk as an array of sectors; $0$ to $n − 1$ is thus the **address space** of the drive.
- Multi-sector operations are possible; indeed, many file systems will read or write 4KB at a time (or more). 
- However, when updating the disk, the only guarantee drive manufacturers make is that a single 512-byte write is **atomic**.
##### Basic Geometry
- The components of a modern disk are:
	1. A **platter**, a circular hard surface on which data is stored persistently by inducing magnetic changes to it.
		- Each platter has 2 sides, each of which is called a surface. 
	2. The platters are all bound together around the **spindle**
		 - Spindle is connected to a motor that spins the platters around at a constant (fixed) rate. 
		 - The rate of rotation is often measured in **rotations per minute (RPM)**, and typical modern values are in the 7,200 RPM to 15,000 RPM range.
	3. Data is encoded on each surface in concentric circles of sectors; we call one such concentric circle a **track**.
		- A single surface contains many thousands and thousands of tracks.
	4. To read and write from the surface, the **disk head** allows us to either sense (i.e., read) the magnetic patterns on the disk or to induce a change in (i.e., write) them. 
		- There is one such head per surface of the drive. 
		- The disk head is attached to a single **disk arm**, which moves across the surface to position the head over the desired track.
##### A Simple Disk Drive
- Imagine we now receive a request to read block 0. How should the disk service this request?
- In our simple disk, the disk doesn’t have to do much. In particular, it must just wait for the desired sector to rotate under the disk head.
- This wait is an important enough component of I/O service time, that it has a special name: **rotational delay**.
	 ![[A Single Track Plus A Head.png|500]]
- So far our disk just has a single track, which is not too realistic; modern disks of course have many millions. Let’s thus look at an ever-so-slightly more realistic disk surface, this one with three tracks.
- we now trace what would happen on a request to a distant sector, e.g., a read to sector 11. 
- To service this read, the drive has to first move the disk arm to the correct track (in this case, the outermost one), in a process known as a **seek**.
- Seeks, along with rotations, are one of the most costly disk operations.
	 ![[Three Tracks Plus A Head.png]]
- When sector 11 passes under the disk head, the final phase of I/O will take place, known as the **transfer**, where data is either read from or written to the surface.
- Many drives employ some kind of **track skew** to make sure that sequential reads can be properly serviced even when crossing track boundaries.
- Sectors are often skewed like this because when switching from one track to another, the disk needs time to reposition the head.
- Without such skew, the head would be moved to the next track but the desired next block would have already rotated under the head, and thus the drive would have to wait almost the entire rotational delay to access the next block.
	 ![[Track Skew.png|500]]
- Another reality is that outer tracks tend to have more sectors than inner tracks, which is a result of geometry; there is simply more room out there. 
- These tracks are often referred to as **multi-zoned** disk drives, where the disk is organized into multiple zones, and where a zone is consecutive set of tracks on a surface.
- Each zone has the same number of sectors per track, and outer zones have more sectors than inner zones.
- An important part of any modern disk drive is its **cache**, sometimes called a **track buffer**.
- This cache is just some small amount of memory (usually around 8 or 16 MB) which the drive can use to hold data read from or written to the disk.
	- For example, when reading a sector from the disk, the drive might decide to read in all of the sectors on that track and cache them in its memory; doing so allows the drive to quickly respond to any subsequent requests to the same track.
	- On writes, the drive has a choice: should it acknowledge the write has completed when it has put the data in its memory, or after the write has actually been written to disk? The former is called **write back** caching (or sometimes **immediate reporting**), and the latter **write through**.
##### I/O Time: Doing The Math
- we can now represent I/O time as the sum of three major components: $$T_{I/O}=T_{seek} + T_{rotation} + T_{transfer}$$
- Note that the rate of I/O ($R_{I/O}$ ), which is often more easily used for comparison between drives (as we will do below), is easily computed from the time. Simply divide the size of the transfer by the time it took: $$R_{I/O}=\frac{Size_{Transfer}}{T_{I/O}}$$
##### Disk Scheduling
- Because of the high cost of I/O, the OS has historically played a role in deciding the order of I/Os issued to the disk. 
- More specifically, given a set of I/O requests, the disk scheduler examines the requests and decides which one to schedule next.
- Unlike job scheduling, where the length of each job is usually unknown, with disk scheduling, we can make a good guess at how long a “job” (i.e., disk request) will take.
- By estimating the seek and possible rotational delay of a request, the disk scheduler can know how long each request will take, and thus (greedily) pick the one that will take the least time to service first. 
- Thus, the disk scheduler will try to follow the principle of [[0x03_Introduction to scheduling#Shortest Job First (SJF)|SJF]] in its operation.
##### SSTF (SSF): Shortest Seek Time First (Shortest Seek First)
- SSTF orders the queue of I/O requests by track, picking requests on the nearest track to complete first.
- For example, assuming the current position of the head is over the inner track, and we have requests for sectors 21 (middle track) and 2 (outer track), we would then issue the request to 21 first, wait for it to complete, and then issue the request to 2.
	 ![[SSTF.png|500]]
- However, SSTF is not a panacea, for the following reasons. 
	1. First, the drive geometry is not available to the host OS; rather, it sees an array of blocks. 
		-  Fortunately, this problem is rather easily fixed. Instead of SSTF, an OS can simply implement **nearest-block-first (NBF)**, which schedules the request with the nearest block address next.
	2. The second problem is: **starvation**.
		- Imagine in our example above if there were a steady stream of requests to the inner track, where the head currently is positioned. 
		- Requests to any other tracks would then be ignored completely by a pure SSTF approach. 
##### Elevator (a.k.a. SCAN or C-SCAN)
- How can we implement SSTF-like scheduling but avoid starvation?
- The algorithm, originally called **SCAN**, simply moves back and forth across the disk servicing requests in order across the tracks. 
- Let’s call a single pass across the disk (from outer to inner tracks, or inner to outer) a sweep. 
- Thus, if a request comes for a block on a track that has already been serviced on this sweep of the disk, it is not handled immediately, but rather queued until the next sweep (in the other direction).
- SCAN has a number of variants, all of which do about the same thing.
	- **F-SCAN**, which freezes the queue to be serviced when it is doing a sweep; this action places requests that come in during the sweep into a queue to be serviced later.
		- Doing so avoids starvation of far-away requests, by delaying the servicing of late-arriving (but nearer by) requests.
	- **C-SCAN** is another common variant, short for Circular SCAN. Instead of sweeping in both directions across the disk, the algorithm only sweeps from outer-to-inner, and then resets at the outer track to begin again.
		- Doing so is a bit more fair to inner and outer tracks, as pure back- and-forth SCAN favors the middle tracks, i.e., after servicing the outer track, SCAN passes through the middle twice before coming back to the outer track again.
- For reasons that should now be clear, the SCAN algorithms are sometimes referred to as the elevator algorithm, because it behaves like an elevator which is either going up or down and not just servicing requests to floors based on which floor is closer.
- Unfortunately, SCAN and its cousins do not represent the best scheduling technology. In particular, they ignore rotation. Also, SCAN (or SSTF even) does not actually adhere as closely to the principle of SJF as they could.
##### SPTF (SATF): Shortest Positioning (Access) Time First
- How can we implement an algorithm that more closely approximates SJF by taking both seek and rotation into account?
- If, in our example, seek time is much higher than rotational delay, then SSTF (and variants) are just fine. 
- However, imagine if seek is quite a bit faster than rotation. 
- Then, in our example, it would make more sense to seek further to service request 8 on the outer track than it would to perform the shorter seek to the middle track to service 16, which has to rotate all the way around before passing under the disk head.
	 ![[sstf_rotation.png|500]]
- On modern drives, as we saw above, both seek and rotation are roughly equivalent (depending, of course, on the exact requests), and thus SPTF is useful and improves performance.
##### Other Scheduling Issues
- An important related task performed by disk schedulers is **I/O merging**. For example, imagine a series of requests to read blocks 33, then 8, then 34, as in Figure 37.8. In this case, the scheduler should **merge** the requests for blocks 33 and 34 into a single two-block request; any reordering that the scheduler does is performed upon the merged requests.
- One final problem that modern schedulers address is this: how long should the system wait before issuing an I/O to disk?
- One might naively think that the disk, once it has even a single I/O, should immediately issue the request to the drive; this approach is called  **work-conserving**, as the disk will never be idle if there are requests to serve. 
- However, research on anticipatory disk scheduling has shown that sometimes it is better to wait for a bit, in what is called a **non-work-conserving** approach.
- By waiting, a new and “better” request may arrive at the disk, and thus overall efficiency is increased.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 37.
- [Lecture 10 - part 2](https://youtu.be/15dJR01z82k)
- [Lecture 10 - part 3](https://youtu.be/yErUVST4Fv0)