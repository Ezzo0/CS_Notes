# Explanation
- Just like at [[0x00_Layers of the internet#Layer 3 Internet Layer|Layer 3]], we could introduce switches that forward packets through a topology, toward their final destination. But, just like at Layer 3, this introduces the routing problem, where the switches need to decide where to forward [[0x27_Ethernet#Ethernet Packet Structure|packets]].
#### Forwarding with Flooding
- The most naive approach to forwarding is to flood every packet you receive. When a switch receives a packet, it sends the packet out of every port.
- This naive approach has two major problems:
	1. It wastes bandwidth. Copies of the packet get unnecessarily sent toward switches and hosts that don’t need that packet.
	2. Flooding can cause packets to loop and overwhelm the network.
#### Learning Switches
- Let’s start with the first problem: Flooding packets wastes bandwidth. To solve this problem, we’d like to populate the forwarding tables for switches, so that they are able to forward packets directly toward their destination, instead of flooding copies of the packets in all directions.
- We could run a routing algorithm to populate the forwarding tables, but an even simpler approach is use **learning switches**.
- Suppose you are router R2. You don’t have any information about the full network topology, and your forwarding table is empty. You have ports to the north, south, east, and west.
- You see a packet coming from the port to your west. The packet says: “From A, To B.” From this packet, you can deduce that A must be to your west. You can now add an entry to your forwarding table: Packets for A should be forwarded to the west.
- This is the key idea behind learning switches. When you receive an incoming packet, you get a clue about where the _sender_ is. You can use that information to populate the forwarding entry for the _sender_.
	![learningSwitch](https://textbook.cs168.io/assets/end-to-end/5-012-learning-1.png)
- As you receive more incoming packets, you are able to start filling in your forwarding table with more entries.
- If you receive a packet whose destination is not in your forwarding table, you can still forward the packet by flooding it out of all ports (except the incoming port).
	![flooding](https://textbook.cs168.io/assets/end-to-end/5-013-learning-2.png)
	![floodingFromDirect|600](https://textbook.cs168.io/assets/end-to-end/5-016-learning-5.png)
- One last feature we need to add: When a forwarding table entry is installed, we assign it a TTL. If the TTL expires, the entry is deleted. This allows routes that are busted (e.g. because a link, host, or switch went down) to expire.
#### STP Motivation: Loops
- Learning switches solved the first problem of flooding, but they do not solve the problem of loops.
- To see why, consider this topology with loops. Suppose all switches are learning switches, and all forwarding tables start empty. A tries to send a packet to B, and forwards the packet to R1.
	- R1 has no entry for B, so it floods the packet to R2 (and R3).
	- R2 has no entry for B, so it floods the packet to R4.
	- R4 has no entry for B, so it floods the packet to R3.
	- R3 has no entry for B, so it floods the packet to R1.
	- R1 has no entry for B, so it floods the packet to R2, and the cycle continues.
	![loop|500](https://textbook.cs168.io/assets/end-to-end/5-021-loop.png)
- At the same time, a copy of the packet is also traveling in a loop in the other direction: R1 flooded to R3 initially, which then flooded to R4, which then flooded to R2, which then flooded to R1, which then floods to R3, continuing the cycle.
- During this entire process, the switches install forwarding entries for A, but they never get any entries for B, so the infinite loop is never resolved.
- This problem is sometimes called a **broadcast storm**, since the network is getting overwhelmed with broadcast traffic.
- How do we solve this problem? Ideally, we’d like to “delete” redundant links, so that the topology has no loops. Then, the learning switch approach will work just fine, with no broadcast storms.
	![loopFixed|500](https://textbook.cs168.io/assets/end-to-end/5-022-loop-fixed.png)
#### STP: Electing a Root
- The **Spanning Tree Protocol (STP)** helps us disable links, so that the resulting topology has no loops. This will help us avoid broadcast storms.
- How does STP decide which links to disable? Let’s start by solving this problem with a global view of the network. Then, we’ll think about how switches exchange messages to achieve this, without a global view of the network.
- The first step in STP is to elect a **root switch**, as follows:
	- Each switch is assigned an ID, consisting of a priority value (manually set by the network operator), and the MAC address of the switch.
	- When comparing two switches, the switch with the lower priority has the lower ID. If the priorities are tied, then the switch with the lower MAC address has the lower ID.
	- The root switch is the switch with the lowest ID.
	![rootElection|250](https://textbook.cs168.io/assets/end-to-end/5-024-stp-root-election.png)
- If the network operator wants to pick a specific root, they can do so by manually setting the priorities of various switches.
- Or, the operator could leave all the switch priorities at their default value, which would cause the switch with the lowest MAC address to be elected as the root.
#### STP: Port States
- Now that we have a root switch, we will classify every port on every switch into one of three states:
	1. **Designated Port:** These are ports pointing away from the root (i.e. they lead somewhere further from the root).
	2. **Root Port:** There are one or more ports pointing toward the root (i.e. they lead somewhere closer to the root). Of these ports, the one along the least-cost path to the root is the root port.
	3. **Blocked Port:** All ports pointing toward the root, that are not the root port (best way to reach the root), are blocked ports.
	 ![portTypes](https://textbook.cs168.io/assets/end-to-end/5-025-stp-port-types.png)
- Here are some examples of the port states in action. Assume that IDs are ordered according to the router labels. This means that R1 has the lowest ID, so it is elected as the root switch.
	![portTypesExample|200](https://textbook.cs168.io/assets/end-to-end/5-026-stp-port-types-example.png)
- Sometimes, we have a tie, and there are two best ways to reach the root. For example, at R4, both the port to R2 and the port to R3 point toward the root, and both of them provide a cost-2 path to the root.
- In case of a tie, we will say that the **next-hop with the lower ID** is the better path to the root. This makes the port to R2 the root port, and the port to R3 a blocked port.
	![tie1|300](https://textbook.cs168.io/assets/end-to-end/5-027-stp-tie-1.png)
- Sometimes, we’ll have a link that leads somewhere equally-far from the root. For example, R4 is distance 2 from the root, and it has a link to R5, which is also distance 2 from the root.
- Again, we’ll use router IDs as a tiebreaker. If the link leads to a higher-ID router, we’ll say that the link points away from the root. If the link leads to a lower-ID router, we’ll say that the link points toward the root.
- In this example, R4’s right-facing port points away from the root (leads to somewhere same-distance, but higher-ID), so it is a designated port.
- On the other hand, R5’s left-facing port points toward the root (leads to somewhere same-distance, but lower-ID), so it is either a root port or a blocked port.
	![tie2](https://textbook.cs168.io/assets/end-to-end/5-028-stp-tie-2.png)
#### STP: Disabling Links
- Now that every port has been assigned a state, we are ready to remove loops from the network topology.
- To remove loops, each switch simply needs to pretend like its blocked ports don’t exist. In other words, do not send any user data out of that port, and do not receive any user data from that port.
- If we stop sending user data along blocked ports, then any link with a blocked port will end up being disabled.
- Why does this work? Let’s think about it from the perspective of a specific switch. Your root port is the best way for you to reach the root. Your blocked ports also point toward the root, but they are not the best path to the root.
- This means that the blocked port actually creates a redundant (but worse) path to the root, so we should disable that link.
#### STP: Designated Ports
- So far, we’ve been drawing networks where every link connects two machines, but remember that sometimes we can have links that connect multiple computers.
- Suppose that a link connecting two switches also has lots of hosts connected to it. If these hosts want to send or receive data, they will send data to the designated port, but not the blocked port.
- This ensures that their data takes exactly one path to the destination. If the data was sent to both the designated port and blocked port, the data could take two paths to the destination, creating a loop.
	![designatedPorts|300](https://textbook.cs168.io/assets/end-to-end/5-031-stp-designated-ports.png)
- With this in mind, another equivalent interpretation of a designated port is: Hosts on a link should send data toward the designated port to reach the root (or anywhere else on the spanning tree).
- From the switch’s perspective, the designated port points away from the root. From the hosts’ perspective, sending to the designated port takes them closer to the root (or anywhere else on the [[Minimum Spanning Trees#^58b2b0|spanning tree]]).
#### STP: BPDU Exchanges
- Our protocol so far assumes global knowledge of the network. But, In order for switches to learn the information they need to label their ports, the switches exchange messages called **Bridge Protocol Data Units (BPDUs)**.
- These are pretty much the same thing as the control-plane routing messages we exchanged in other routing protocols, but with a fancy name.
- When the protocol begins, every switch thinks that the root is itself, and the cost to the root (itself) is 0.
- As the protocol runs, every switch keeps track of what it thinks the root is, and the best-known path to that root (and the cost of that path).
	![bpduStart|500](https://textbook.cs168.io/assets/end-to-end/5-032-bpdu-start.png)
- When you send a BPDU, you include two pieces of information: Who you think the root is, and how far away you are from the root.
- When you receive a BPDU, you check if it has any “better” information. The BPDU could be better for two reasons:
	1. The root in the BPDU has a lower ID. This means that you have discovered a better root. You should abandon your current root and cost, and instead adopt the new root and the path to the new root.
	2. The root in the BPDU is the same, but the BPDU is offering a better path to the root. You should adopt the new path to the root.
	![bpduAdvertisements](https://textbook.cs168.io/assets/end-to-end/5-033-bpdu-advertisements.png)
- Costs to root are computed just like we did in the [[0x08_Distance-Vector Protocols|distance-vector protocol]].
- When you update your state (who you think the root is, or your best-known cost to root), you should send a BPDU to your neighbors to inform them about your new state.
- Once the protocol converges, the state gives each switch enough information to label all of its ports. You know the best path to the root, so you can label the corresponding port as the root port.
- Your neighbors have also all told you how far away they are from the root. If a neighbor says they are further, then you can label the corresponding port as a designated port.
- If a neighbor says they are closer (but they are not on your best path to the root), then you can label the corresponding port as a blocked port.
- BPDUs are exchanged regularly, so that if the network topology changes, the protocol can adapt and find a [[Minimum Spanning Trees#^58b2b0|spanning tree]] (i.e. disable links) for the new topology.
# Sources
- [Lecture 18 - End-to-End 1: Ethernet, STP](https://www.youtube.com/watch?v=efWtZ8bzBOM).
- [Lecture 19 - End-to-End 2: ARP, DHCP, NAT, TLS](https://www.youtube.com/watch?v=AgoxhLdLQSQ).
- [Layer 2 Routing (STP)](https://textbook.cs168.io/end-to-end/l2-routing.html).