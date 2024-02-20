# Explanation
- Proportional-share is based around a simple concept: instead of optimizing for turnaround or response time, a scheduler might instead try to guarantee that each job obtain a **certain percentage of CPU time**.
##### Basic Concept: Tickets Represent Your Share
- Tickets are used to represent the share of a resource that a [[Process]] or user should receive.
- The **percent of tickets** that a process has represents its **share** of the system resource.
- Lottery scheduling achieves this probabilistically (but not deterministically) by holding a lottery every so often (say, every time slice).
- The scheduler must know how many total tickets there are.
- The scheduler then picks a winning ticket.
- The winning ticket simply determines whether A or B runs. The scheduler then loads the state of that winning process and runs it.
##### Ticket Mechanisms
- Ticket currency allows a user with a set of tickets to allocate tickets among their own jobs in whatever currency they would like; the system then automatically converts said currency into the correct global value.
- Assume users A and B have each been given 100 tickets. User A is running two jobs, A1 and A2, and gives them each 500 tickets (out of 1000 total) in A’s currency. User B is running only 1 job and gives it 10 tickets (out of 10 total).
    ![[ticket currency.png|700]]
- With ticket transfers, a process can temporarily hand off its tickets to another process. This ability is especially useful in a client/server setting, where a client process sends a message to a server asking it to do some work on the client’s behalf.
- To speed up the work, the client can pass the tickets to the server and thus try to maximize the performance of the server while the server is handling the client’s request. When finished, the server then transfers the tickets back to the client and all is as before.
- With ticket inflation, a process can temporarily raise or lower the number of tickets it owns.
- if any one process knows it needs more CPU time, it can boost its ticket value as a way to reflect that need to the system, all without communicating with any other processes.
##### Implementation
- All you need is a good random number generator to pick the winning ticket, a data structure to track the processes of the system (e.g., a list), and the total number of tickets.
- Let’s assume we keep the processes in a list. Here is an example comprised of three processes, A, B, and C, each with some number of tickets.![[three processes.png]]
```c
// counter: used to track if we’ve found the winner yet
int counter = 0;

// winner: call some random number generator to
// get a value >= 0 and <= (totaltickets - 1)
int winner = getrandom(0, totaltickets);

// current: use this to walk through the list of jobs
node_t *current = head;
while (current) {
	counter = counter + current->tickets;
	if (counter > winner)
		break; // found the winner
	current = current->next;
}
// ’current’ is the winner: schedule it...
```
- To make this process most efficient, it might generally be best to organize the list in sorted order, from the highest number of tickets to the lowest.
##### Stride Scheduling
- While randomness gets us a simple (and approximately correct) scheduler, it occasionally will not deliver the exact right proportions, especially over short time scales. For this reason, stride scheduling was invented, a deterministic fair-share scheduler.
- Each job in the system has a stride, which is inverse in proportion to the number of tickets it has.
- Stride of each process can be computed by dividing some large number by the number of tickets each process has been assigned.
- Every time a process runs, we will increment a counter for it (called its pass value) by its stride to track its global progress.
- A pseudo-code implementation:
```c
curr = remove_min(queue);         //pick client with min pass
schedule(curr);                   //run for quantum
curr->pass += curr->stride;       //update pass using stride
insert(queue, curr);              //return curr to queue
```
- Lottery scheduling has one property that stride scheduling does not: no global state.
- Imagine a new job enters in the middle of our stride scheduling; what should its pass value be? Should it be set to 0? 
	- If so, it will monopolize the CPU. 
	- With lottery scheduling, there is no global state per process; we simply add a new process with whatever tickets it has, update the single global variable to track how many total tickets we have, and go from there. 
	- In this way, lottery makes it much easier to incorporate new processes in a sensible manner.
##### The Linux Completely Fair Scheduler (CFS)
- Whereas most schedulers are based around the concept of a fixed time slice, CFS operates a bit differently. 
- Its goal is simple: to fairly divide a CPU evenly among all competing processes.
- It does so through a simple counting-based technique known as **virtual runtime (vruntime)**.
- CFS manages the scheduling decision tension through various control parameters. The first is **sched_latency**. 
- CFS uses this value to determine how long one process should run before considering a switch (effectively determining its time slice but in a dynamic fashion). 
- A typical sched_latency value is **48 (milliseconds)**; CFS divides this value by the number (n) of processes running on the CPU to determine the time slice for a process.![[CFS Simple Example.png|600]]
- If there are too many processes running that will lead to too small of a time slice, and thus too many context switches.
- To address this issue, CFS adds another parameter, **min_granularity**, which is usually set to a value like **6 ms**. CFS will never set the time slice of a process to less than this value.
- CFS utilizes a periodic timer interrupt, which means it can only make decisions at fixed time intervals.
	- This interrupt goes off frequently (e.g., every 1 ms), giving CFS a chance to wake up and determine: 
		- if the current job has reached the end of its run. 
		- If a job has a time slice that is not a perfect multiple of the timer interrupt interval.
	- CFS tracks vruntime precisely, which means that over the long haul, it will eventually approximate ideal sharing of the CPU.
- CFS also enables controls over process priority, enabling users or administrators to give some processes a higher share of the CPU.
- It does this not with tickets, but through a classic U NIX mechanism known as the **nice** level of a process.
- The nice parameter can be set anywhere from **-20 to +19** for a process, with a default of **0**. 
	- Positive nice values imply lower priority. 
	- Negative values imply higher priority.
- CFS maps the nice value of each process to a weight, as shown here:
```c
static const int prio_to_weight[40] = {
/* -20 */ 88761, 71755, 56483, 46273, 36291,
/* -15 */ 29154, 23254, 18705, 14949, 11916,
/* -10 */  9548,  7620,  6100,  4904,  3906,
/*  -5 */  3121,  2501,  1991,  1586,  1277,
/*   0 */  1024,   820,   655,   526,   423,
/*   5 */   335,   272,   215,   172,   137,
/*  10 */   110,    87,    70,    56,    45,
/*  15 */    36,    29,    23,    18,    15,
};
```
- These weights allow us to compute the effective time slice of each process.
- The formula used to do so is as follows, assuming n processes: $$time-slice_k = \frac{weight_k}{\sum{weight_i}} . sched-latency$$
- The way CFS calculates **_vruntime_** is:$$vruntime_i = vruntime_i + \frac{weight_0} {weight_i} . runtime_i$$
- When the scheduler has to find the next job to run, it should do so as quickly as possible. 
- Simple data structures like lists don’t scale: modern systems sometimes are comprised of 1000s of processes, and thus searching through a long-list every so many milliseconds is wasteful.
- CFS addresses this by keeping processes in a [[Red-black BSTs]].
- CFS does not keep all processes in this structure; rather, only running processes are kept therein. 
- If a process goes to sleep (say, waiting on an I/O to complete, or for a network packet to arrive), it is removed from the tree and kept track of elsewhere.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 9.