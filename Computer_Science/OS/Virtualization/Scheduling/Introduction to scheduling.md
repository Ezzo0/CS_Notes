# Explanation
##### Workload Assumptions
- Before getting into the range of possible policies, let us first make a number of simplifying assumptions about the [[Process]]es running in the system, sometimes collectively called the **workload**.
	1. Each job runs for the same amount of time.
	2. All jobs arrive at the same time.
	3. Once started, each job runs to completion.
	4. All jobs only use the CPU (i.e., they perform no I/O).
	5. The run-time of each job is known.
##### Scheduling Metrics
- A metric is just something that we use to measure something.
- Scheduling metrics enable us to compare different scheduling policies.
- The turnaround time, which is a performance metric, of a job is defined as the time at which the job completes minus the time at which the job arrived in the system. $$T_{turnaround} = T_{completion} − T_{arrival}$$
- Performance and fairness are often at odds in scheduling; a scheduler, for example, may optimize performance but at the cost of preventing a few jobs from running, thus decreasing fairness.
##### First In, First Out (FIFO)
- Imagine three jobs arrive in the system, A, B, and C, at roughly the same time (T arrival = 0). Because FIFO has to put some job first, let’s assume that while they all arrived simultaneously, A arrived just a hair before B which arrived just a hair before C. Assume also that each job runs for 10 seconds.
- Thus, the average turnaround time for the three jobs is simply $$\frac{10+20+30}{3} = 20$$
	 ![[FIFO Simple Example.png|400]]
- Now let’s relax assumption 1 and assume three jobs (A, B, and C), but this time A runs for 100 seconds while B and C run for 10 each.
- Job A runs first for the full 100 seconds before B or C even get a chance to run. Thus, the average turnaround time for the system is high: a painful 110 seconds.
	 ![[Why FIFO Is Not That Great.png|400]]
- This problem is generally referred to as the **convoy effect**, where a number of relatively-short potential consumers of a resource get queued behind a heavyweight resource consumer.
##### Shortest Job First (SJF)
- SJF runs the shortest job first, then the next shortest, and so on.
- SJF reduces average turnaround from 110 seconds to 50, more than a factor of two improvement.
	 ![[SJF Simple Example.png|400]]
- Let’s relax assumption 2, and now assume A arrives at t = 0 and needs to run for 100 seconds, whereas B and C arrive at t = 10 and each need to run for 10 seconds. With pure SJF, even though B and C arrived shortly after A, they still are forced to wait until A has completed, and thus suffer the same convoy problem.
- Average turnaround time for these three jobs is 103.33 seconds.
	 ![[SJF With Late Arrivals From B and C.png|400]]
##### Shortest Time-to-Completion First (STCF)
- By relaxing assumption 3, the scheduler can **preempt** job A and decide to run another job, perhaps continuing A later. SJF by our definition is a **non-preemptive scheduler**, and thus suffers from the problems described above.
- Any time a new job enters the system, the STCF scheduler determines which of the remaining jobs (including the new job) has the least time left, and schedules that one.
- The result is a much-improved average turnaround time: 50 seconds.
	 ![[STCF Simple Example.png|400]]
##### A New Metric: Response Time
- Response time is defined as the time from when the job arrives in a system to the first time it is scheduled. $$T_{response} = T_{firstrun} − T_{arrival}$$
- For example, if we had the schedule from Figure 7.5 (with A arriving at time 0, and B and C at time 10), the response time of each job is as follows: 0 for job A, 0 for B, and 10 for C (average: 3.33).
- If three jobs arrive at the same time, for example, the third job has to wait for the previous two jobs to run in their entirety before being scheduled just once.
##### Round Robin
- The basic idea is simple: instead of running jobs to completion, RR **runs a job for a time slice** (sometimes called a **scheduling quantum**) and then switches to the next job in the run queue. For this reason, RR is sometimes called **time-slicing**.
- Note that the length of a time slice must be a multiple of the timer-interrupt period.
- RR with a time-slice of 1 second the The average response time of RR is: 1.
	 ![[Round Robin.png|500]]
- The shorter it is, the better the performance of RR under the response-time metric. However, making the time slice too short is problematic: suddenly the cost of context switching will dominate overall performance.
- More generally, any policy (such as RR) that is fair, i.e., that evenly divides the CPU among active processes on a small time scale, will perform poorly on metrics such as turnaround time. 
- Indeed, this is an inherent trade-off: if you are willing to be unfair, you can run shorter jobs to completion, but at the cost of response time; if you instead value fairness, response time is lowered, but at the cost of turnaround time.
##### Incorporating I/O
- Let's  relax assumption 4 and assume we have two jobs, A and B, which each need 50 ms of CPU time. However, there is one obvious difference: A runs for 10 ms and then issues an I/O request (take 10 ms), whereas B simply uses the CPU for 50 ms and performs no I/O.
- The STCF scheduler runs A first, then B after. 
	 ![[IO.png|500]]
- A common approach is to treat each 10-ms sub-job of A as an independent job. 
- Thus, when the system starts, its choice is whether to schedule a 10-ms A or a 50-ms B. With STCF, the choice is clear: choose the shorter one, in this case A. 
- Then, when the first sub-job of A has completed, only B is left, and it begins running. Then a new sub-job of A is submitted, and it preempts B and runs for 10 ms.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 7.
- [Lecture 2 - part 2](https://youtu.be/Q09UgVfragU)