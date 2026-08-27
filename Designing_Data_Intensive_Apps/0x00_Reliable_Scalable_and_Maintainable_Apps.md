# Explanation
- If you are designing a data system or service, a lot of tricky questions arise. How do you ensure that the data remains correct and complete, even when things go wrong internally? How do you provide consistently good performance to clients, even when parts of your system are degraded? How do you scale to handle an increase in load? What does a good API for the service look like?
- There are many factors that may influence the design of a data system, we focus on three concerns that are important in most software systems:
	- **Reliability**
		- The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).
	- **Scalability**
		- As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.
	- **Maintainability**
		- Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.
### Reliability
- Everybody has an intuitive idea of what it means for something to be reliable or unreliable. For software, typical expectations include:
	- The application performs the function that the user expected.
	- It can tolerate the user making mistakes or using the software in unexpected ways.
	- Its performance is good enough for the required use case, under the expected load and data volume.
	- The system prevents any unauthorized access and abuse.
- If all those things together mean “*working correctly*,” then we can understand reliability as meaning, roughly, “*continuing to work correctly, even when things go wrong*.”
- The things that can go wrong are called faults, and systems that anticipate faults and can cope with them are called fault-tolerant or resilient.
- Note that a fault is ***not the same as*** a failure. 
	- A **fault** is usually defined as one component of the system deviating from its spec.
	- A **failure** is when the system as a whole stops providing the required service to the user.
### Hardware Faults
- When we think of causes of system failure, hardware faults quickly come to mind. Hard disks crash, RAM becomes faulty, the power grid has a blackout, someone unplugs the wrong network cable.
- Hard disks are reported as having a mean time to failure (MTTF) of about **10 to 50 years**. Thus, on a storage cluster with 10,000 disks, we should expect on average one disk to die per day.
- Our first response is usually to add redundancy to the individual hardware components in order to reduce the failure rate of the system. Disks may be set up in a [[0x25_Redundant Arrays of Inexpensive Disks|RAID]] configuration, servers may have dual power supplies and hot-swappable CPUs, and datacenters may have batteries and diesel generators for backup power.
### Software Errors
- Another class of fault is a systematic error within the system. Such faults are harder to anticipate, and because they are correlated across nodes, they tend to cause many more system failures than uncorrelated hardware faults. Examples include:
	- A software bug that causes every instance of an application server to crash when given a particular bad input. For example, consider the leap second on June 30, 2012, that caused many applications to hang simultaneously due to a bug in the Linux kernel.
	- A runaway process that uses up some shared resource—CPU time, memory, disk space, or network bandwidth.
### Scalability
- Scalability is the term we use to describe a system’s ability to cope with **increased load**.
##### Describing Load
- Load can be described with a few numbers which we call *load parameters*.
- The best choice of parameters depends on the architecture of your system: it may be **requests per second** to a web server, the **ratio of reads to writes** in a database, the number of **simultaneously active users** in a chat room, the **hit rate on a cache**, or something else.
##### Describing Performance
- Once you have described the load on your system, you can investigate what happens when the load increases. You can look at it in two ways:
	- When you increase a load parameter and keep the system resources (CPU, memory, network bandwidth, etc.) unchanged, how is the performance of your system affected?
	- When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?
- Both questions require performance numbers, so let’s look briefly at describing the performance of a system.
- In a batch processing system such as Hadoop, we usually care about throughput—the number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size.
- In online systems, what’s usually more important is the service’s response time—that is, the time between a client sending a request and receiving a response.
- Even if you only make the same request over and over again, you’ll get a slightly different response time on every try. We therefore need to think of response time not as a single number, but as a *distribution of values* that you can measure.
- In Figure 1-4, each gray bar represents a request to a service, and its height shows how long that request took. Most requests are reasonably fast, but there are occasional outliers that take much longer.
	![[percentiles.png]]
- It’s common to see the average response time of a service reported. However, the mean is not a very good metric if you want to know your “typical” response time, because it doesn’t tell you how many users actually experienced that delay.
- Usually it is better to use percentiles. If you take your list of response times and sort it from fastest to slowest, then the median is the halfway point: for example, if your median response time is 200 ms, that means half your requests return in less than 200 ms, and half your requests take longer than that.
- This makes the median a good metric if you want to know how long users typically have to wait: half of user requests are served in less than the median response time, and the other half take longer than the median. The median is also known as the **50th percentile**, and sometimes abbreviated as **p50**.
- Note that the median refers to a single request; if the user makes several requests (over the course of a session, or because several resources are included in a single page), the probability that *at least one* of them is slower than the median is *much greater than* 50%.
- In order to figure out *how bad* your outliers are, you can look at higher percentiles: the 95th, 99th, and 99.9th percentiles are common.
- They are the response time thresholds at which 95%, 99%, or 99.9% of requests are faster than that particular threshold.
- For example, if the 95th percentile response time is 1.5 seconds, that means 95 out of 100 requests take less than 1.5 seconds, and 5 out of 100 requests take 1.5 seconds or more.
### Maintainability
- We should design software in such a way that it will hopefully minimize pain during maintenance, and thus avoid creating legacy software ourselves.
- To this end, we will pay particular attention to three design principles for software systems:
	- **Operability:** Make it easy for operations teams to keep the system running smoothly.
	- **Simplicity:** Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. (Note this is not the same as simplicity of the user interface.)
	- **Evolvability:**
		- Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change.
		- Also known as ***extensibility***, ***modifiability***, or ***plasticity***.
# Sources
- Designing Data-Intensive Applications - Chapter 1.
- [Designing Data Intensive Applications بالعربي - ch1 - Reliable Scalable and Maintainable Apps](https://www.youtube.com/watch?v=Ipj7H_vCi6Q&list=PLTRDUPO2OmIljJwE9XMYE_XEgEIWZDCuQ&index=1&pp=iAQB0gcJCQYLAYcqIYzv).