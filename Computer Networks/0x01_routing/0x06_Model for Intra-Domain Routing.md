# Explanation
- The Internet as a set of machines, connected together with a set of links, where each link connects two of the machines on the network.
- We can represent the network topology as a graph, where each node represents a machine, and each edge between two nodes represents a link between two machines.
- Historically, sometimes links could connect more than two machines, but in modern networks, links essentially always connect exactly two machines.
#### Full Mesh Network Topology
- Suppose we have two machines, A and B. If the two machines want to exchange messages, we could add a link between them.
- But what if we had five machines instead of two? One possible approach is to create a link between every pair of machines, such that every machine is connected to every other machine. This is sometimes called a **full mesh topology**.
	![fullMesh|200](https://textbook.cs168.io/assets/routing/2-005-mesh.png)
- What are some drawbacks of this approach? This approach doesn’t scale well. If we tried to scale this to the size of the modern Internet, we’d need a wire connecting every pair of computers in the world.
- When a new computer joins the network, we’d have create new links between that computer and every other computer in the world.
- Although it can’t scale to the entire Internet, there are still some benefits to a a full mesh topology in smaller settings. In particular, having links between every pair of machines gives us a lot of bandwidth on the network.
- Every machine has a dedicated link to all other machines, and each pair of machines can use the full bandwidth on their dedicated link.
- In general, there is no guarantee that each machine has a direct link to all other links. In other words, there is no guarantee that the underlying graph is fully-connected.
#### Single-Link Network Topology
- In addition to the full mesh topology, there are other ways in which we can deploy links to connect up multiple machines. For example, we could use a single link to connect up all five machines:
	![singleLink|300](https://textbook.cs168.io/assets/routing/2-006-single-link.png)
- This approach would scale better than the full mesh topology. For example, if a new computer joined the network, instead of creating five new links between the new computer and the five existing computers, we can just extend the existing wire to the new computer.
- However, this approach is more limited in the amount of bandwidth available to the machines. In particular, there is only a single link, and all five machines need to share the bandwidth on this link.
#### Routers and Hosts
- **End hosts** are machines connecting to the Internet to send and receive data. Examples of end hosts include applications on your own personal computer, such as your web browser.
- Web servers, such as a Google web server receiving Google search queries and sending back search results, are also end hosts. These machines send outgoing packets to other destinations, and could be the final destination for incoming packets.
- However, these machines usually do not receive and forward intermediate [[0x00_Layers of the internet#Layer 3 Packets Abstraction|packets]] (i.e. packets with some different final destination).
- **Routers**, by contrast, are machines connected to the Internet responsible for receiving and forwarding intermediate packets closer to their final destination. For example, consider the router installed in your home network, or routers living in a data center building somewhere.
- These machines usually do not create and send new packets of their own, and they usually are not the final destination for packets.
- Depending on the network design, routers could be legal destinations, but in this unit, we’ll ignore routers as destinations. However, do note that **routers potentially can be sources and send new packets of their own**.
- Note that end hosts generally do not participate in routing protocols, since they don’t forward intermediate packets. Instead, end hosts are often connected to a single router with a single link.
- By default, the end host sends all outgoing messages to the router, which will figure out how to send the packet to its final destination. This strategy of sending everything to the router is sometimes called the **default route** of the end host.
	![network](https://textbook.cs168.io/assets/routing/2-007-host-router.png)
#### Network Topologies with Routers
- Now that we have routers in addition to end hosts, we can create more complicated network topologies like this:
	![routerTopology1|250](https://textbook.cs168.io/assets/routing/2-009-router-topology.png)
- This topology lets us combine the benefits of the full-mesh and single-link topologies. In particular, this topology uses fewer links than the full mesh topology from earlier. Also, this topology has more bandwidth than the single link topology from earlier.
- This topology is also more robust to failure. If a link goes down, the packet can take a different path through the network and still reach its destination.
	![linkDown](https://textbook.cs168.io/assets/routing/2-010-different-path.png)
#### Routing Protocols are Distributed
- If the network changes, perhaps we could solve the routing problem by updating our graph and then computing paths through the new graph.
- Another problem that makes routing difficult is that routers don’t inherently have a global, birds-eye view of the entire network. For example, if a link somewhere else in the network fails, there’s no way for all routers to automatically know this.
- We will have to somehow propagate that information about the new network topology to the routers as part of our routing protocol.
	![nonBirdEye](https://textbook.cs168.io/assets/routing/2-012-non-global.png)
- This leads to routing protocols often being distributed protocols. Instead of a single central mastermind computing all the answers, each router must compute its own part of the answer (possibly without full knowledge of the network topology).
- Collectively, the answers computed by each router must form a global answer to the routing problem that allows packets to reach their end destination.
- The distributed nature of routing protocols also means that we have to account for individual routers failing. If there was a single computer that was solving the problem, and that computer crashed and forgot the answer, we could simply make the computer re-compute the entire answer from scratch.
- However, in a distributed protocol, if one router crashes and forgets its part of the answer, our protocol will need a way to help this one router recover from failure and re-learn its part of the answer.
# Sources
- [Lecture 4 - Routing 1: Principles](https://www.youtube.com/watch?v=aEkZKsOWvzE).
- [Model for Intra-Domain Routing](https://textbook.cs168.io/routing/model.html).