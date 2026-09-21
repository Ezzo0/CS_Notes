# Explanation
- Recall that if many packets arrive at a router at the same time (e.g. bursty traffic), and the router needs to send both [[0x00_Layers of the internet#Layer 3 Packets Abstraction|packets]] over the same [[0x04_Links|link]], then the router will send one packet and put the other packets in a queue (to be sent later).
- More generally, if the input rate of packets exceeds the output rate that the link can sustain, the router will be unable to keep up with the pace of incoming packets.
- This router is **congested**, and needs to keep packets in a queue while they wait their turn to be sent. The queue can cause packets to be delayed. If the queue itself gets too full and packets are still incoming, then packets can get dropped.
- This graph shows the performance of a queueing system with bursty arrivals. The dotted line represents the link’s capacity (maximum load). As we increase the load, packets get more delayed.
	![congestionGraph|400](https://textbook.cs168.io/assets/transport/3-052-congestion2.png)
- Notice that the graph starts sloping upwards even before we reach the dotted line. This means that the queueing is already delaying packets, even if nothing is dropped.
- By the time we reach maximum utilization and start losing packets, we are already incurring very large packet delays from the queue.
#### Why is Congestion Control Hard?
- To get a sense of why congestion control is a difficult problem, consider the following network graph. At what rate should host A send traffic?
- It depends on the destination, so A can’t just come up with one fixed rate for all destinations. For example, if A is communicating with C, then A could send packets at 10 Gbps.
- What if A is communicating with F instead? The bottleneck link (least capacity) along this path is 2Gbps, so A should probably send packets at 2 Gbps.
- What if A is communicating with E? It depends on what path the traffic is taking between A and E.
- If the traffic is taking the bottom path through R3, then A could send packets at 10 Gbps. But if the traffic is taking the top path through R2, then A can now only send packets at 1 Gbps.
	![sendingRateLimit|600](https://textbook.cs168.io/assets/transport/3-055-congestion5.png)
- One takeaway so far is that our congestion control algorithm will need to somehow learn about the bandwidths and bottlenecks along the path that the packet is taking.
- Also, recall that the network graph changes over time as new links are added or links go down. This means that it’s not enough to learn about paths a single time. Our algorithm will need to be adaptive to changes in network topology.
- So far, we’ve assumed that A is the only host sending traffic on the network, and A can use the full capacity of every link. But what if other connections are also using bandwidth?
- In this example, A and F have a connection, and B and E have a connection. The two connections seem like they should be totally separate (different senders, different recipients), but in fact, their paths share a link in the network.
- If we want the two connections to share the capacity on this link fairly, maybe A and B should each send at 1 Gbps.
	![sendingRateLimit2|600](https://textbook.cs168.io/assets/transport/3-057-congestion7.png)
- What if a new connection starts between G and D? First, notice that the G-D and B-E connections are sharing a link. This means that these two connections have to slow their rate down to 0.5 Gbps.
- Now, if we look back at the 2 Gbps link that A-F and B-E had in common, B-E is only using 0.5 Gbps on this link. This means that A could increase its rate to 1.5 Gbps.
- What happened here? The G-D connection was created, and its path has no links in common with the A-F connection. And yet, this seemingly unrelated connection caused the A-F connection’s rate to increase.
- Connections can indirectly affect other connections, even if those two connections don’t share any links in common!
- In summary: When the sender is trying to determine a rate for sending packets, it has to consider:
	- The destination.
	- The path to that destination.
	- The connections sharing links along that path.
	- The connections sharing links with those connections (indirect competition), and so on.
- Congestion control is a hard problem because all the connections in the network are dependent on each other to determine their optimal sending rate.
#### Goals for a Good Congestion Control Algorithm
- From a resource allocation perspective, there are three goals we want out of a good congestion control algorithm.
	- We’d like the resource allocation to be efficient.
	- Links should not be overloaded, and there should be minimal packet delay and loss.
	- Also, links should be utilized as much as possible.
- We want a solution that achieves a good trade-off between these goals. It would be possible to optimize one goal at the expense of the others, but that leads to bad solutions.
- For example, we could ensure maximal link utilization by having everyone send packets extremely quickly (bad solution, causes congestion). Or, we could ensure minimal packet loss by making everybody send packets extremely slowly (bad solution, not utilizing capacity).
- From a more practical systems perspective, the solution we come up with needs to be scalable and decentralized.
- All modern congestion control algorithms are based on dynamic adjustment. Hosts dynamically learn the current level of congestion, and adjust their sending rate accordingly.
- In practice, dynamic adjustment is a practical solution because it can be easily generalized. This approach doesn’t assume any business model (needed for pricing), and doesn’t assume anything about users knowing the bandwidth they need ahead of time (needed for [[0x03_Designing Resource Sharing#^d1c424|reservations]]).
- Dynamic adjustment does require good citizenship. TCP needs everybody on the network to work together to share the resources fairly. For example, when a new connection starts using links, other connections need to slow down and share the bandwidth.
- Within the dynamic adjustment approach, there are two broad classes of solutions. In **host-based** congestion control algorithms, the sender is monitoring the performance and adjusting its rate accordingly.
- These algorithms are implemented entirely at the sender, and there is no special support from routers. The modification to TCP is a host-based algorithm, and is widely deployed today.
- In **router-assisted** congested control algorithms, routers will explicitly send information about congestion back to the sender, to help the sender adjust its rate.
- Congestion happens at routers, so routers are in a good position to offer information about congestion. Router-assisted algorithms have been deployed in recent years, especially in datacenters.
- Some router-assisted algorithms send very little information, e.g. a single bit indicating congestion, while other algorithms send more detailed information, e.g. the exact rate the sender should use.
- Note that in both cases, routers are signaling congestion back to the sender. In router-assisted algorithms, the router is explicitly sending a message about its level of congestion.
- By contrast, in host-based algorithms, the sender does not receive explicit feedback from the routers. Instead, the sender uses implicit clues from the router (e.g. packets getting dropped or delayed) to deduce that the router is congested. ^630a26
# Sources
- [Lecture 12 - Transport 2: TCP II](https://www.youtube.com/watch?v=t-qicTlEh5o).
- [Congestion Control Principles](https://textbook.cs168.io/transport/cc-principles.html).