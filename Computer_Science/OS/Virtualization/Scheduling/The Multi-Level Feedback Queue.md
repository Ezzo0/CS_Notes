# Explanation
- The fundamental problem MLFQ tries to address is two-fold. 
	1. First, it would like to optimize turnaround time, which, is done by running shorter jobs first; unfortunately, the OS doesn’t generally know how long a job will run for, exactly the knowledge that algorithms like [[Introduction to scheduling#Shortest Job First (SJF)|SJF]] (or [[Introduction to scheduling#Shortest Time-to-Completion First (STCF)|STCF]]) require. 
	2. Second, MLFQ would like to make a system feel responsive to interactive users, and minimize response time; unfortunately, algorithms like [[Introduction to scheduling#Round Robin|Round Robin]] reduce response time but are terrible for turnaround time.
##### MLFQ: Basic Rules
- The MLFQ has a number of distinct queues, each assigned a different **priority level**.
- MLFQ uses priorities to decide which job should run at a given time: a job with higher priority (i.e., a [[Process]] on a higher queue) is chosen to run.
- Of course, more than one job may be on a given queue, and thus have the same priority. In this case, we will just use [[Introduction to scheduling#Round Robin|Round Robin]] scheduling among those jobs.
- The key to MLFQ scheduling therefore lies in how the scheduler sets priorities. 
- Rather than giving a fixed priority to each job, MLFQ varies the priority of a job based on its **observed behavior**. 
	- If, for example, a job repeatedly relinquishes the CPU while waiting for input from the keyboard, MLFQ will keep its priority high, as this is how an interactive process might behave. 
	- If, instead, a job uses the CPU intensively for long periods of time, MLFQ will reduce its priority. In this way, MLFQ will try to learn about processes as they run, and thus use the history of the job to predict its future behavior.
##### How To Change Priority
- The **job's allotment** is the amount of time a job can spend at a given priority level before the scheduler reduces its priority.
- For simplicity, at first, we will assume the allotment is equal to a single time slice.
- Here is our first attempt at a priority-adjustment algorithm:
	- Rule 3: When a job enters the system, it is placed at the highest priority (the topmost queue).
	- Rule 4a: If a job uses up its allotment while running, its priority is reduced (i.e., it moves down one queue).
	- Rule 4b: If a job gives up the CPU before the allotment is up, it stays at the same priority level.
- Unfortunately, the approach we have developed thus far contains serious flaws.
	- There is the problem of **starvation**: if there are “too many” interactive jobs in the system, they will combine to consume all CPU time, and thus long-running jobs will never receive any CPU time (they **starve**). We’d like to make some progress on these jobs even in this scenario.
	- Secondly, a program may change its behavior over time; what was CPU-bound may transition to a phase of interactivity. With our current approach, such a job would be out of luck and not be treated like the other interactive jobs in the system.
##### The Priority Boost
- The simple idea here is to periodically boost the priority of all the jobs in the system by throwing them all in the **topmost queue after some time period S**.
    ![[Priority Boost.png|600]]
##### Better Accounting
- The solution to prevent **gaming of our scheduler** is to perform better accounting of CPU time at each level of the MLFQ.
- **Gaming the scheduler** generally refers to the idea of doing something sneaky to trick the scheduler into giving you more than your fair share of the resource.
- Instead of forgetting how much of its allotment a process used at a given level when it performs I/O, the scheduler should keep track; once a process has used its allotment, it is demoted to the next priority queue. 
- Whether it uses its allotment in one long burst or many small ones should not matter.
- Rules 4a and 4b are reduced to the following single rule:
	- Rule 4: Once a job uses up its time allotment at a given level (regardless of how many times it has given up the CPU), its priority is reduced (i.e., it moves down one queue).
##### Tuning MLFQ And Other Issues
- Most MLFQ variants allow for varying time-slice length across different queues. 
	- The high-priority queues are usually given short time slices; they are comprised of interactive jobs, after all, and thus quickly alternating between them makes sense (e.g., 10 or fewer milliseconds).
	- The low-priority queues, in contrast, contain long-running jobs that are CPU-bound; hence, longer time slices work well (e.g., 100s of ms).
- The Solaris MLFQ implementation —Time-Sharing scheduling class, or TS — is particularly easy to configure. 
- It provides a set of tables that determine exactly how the priority of a process is altered throughout its lifetime, how long each time slice is, and how often to boost the priority of a job.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 8.
- [Lecture 3 - part 1](https://youtu.be/cAiwISFta4g)