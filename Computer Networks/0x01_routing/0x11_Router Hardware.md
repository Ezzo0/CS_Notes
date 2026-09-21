# Explanation
- A router is a specialized computer optimized for performing routing and forwarding tasks.
- **Colocation facilities** or **carrier hotels** are buildings where multiple ISPs install routers to connect to each other.
- These buildings are specially designed to have power and cooling infrastructure, and ISPs can rent space to install routers and connect them to other routers in the same building.
	![carrierHotel|600](https://textbook.cs168.io/assets/routing/2-114-carrier-hotel.png)
#### Router Sizes and Capacities
- There are different ways we can measure the size of a router. We could consider its physical size, the number of physical ports it has, and its bandwidth.
- We can measure a router’s capacity as the **number of physical ports, multiplied by the bandwidth of each physical port**. The speed or bandwidth of a physical port is often called its **line rate**.
- Not all physical ports need to have the same line rate. For example, a modern home router might have 4 physical ports that can send at 100 Mbps, and 1 physical port that can send at 1 Gbps. The total capacity of this router is 1.4 Gbps.
- A modern state-of-the-art router used by ISPs might have a line rate of up to 400 Gbps per physical port. This router contains multiple removable **line cards**, where each line card contains a set of physical ports.
- A modern router might have 8 line cards, with 36 physical ports per line card, for a total of 288 physical ports. 288 physical ports, each with 400 Gbps bandwidth, gives our router a total capacity of 115.2 Tbps.
#### Data, Control, Management Planes
- The hardware and software components of the router can conceptually be split into three planes.
	1. The **data plane:**
		- Mainly responsible for forwarding packets.
		- Used every time a packet arrives and needs to be forwarded.
		- Operates locally, without coordinating with other routers.
	2. The **control plane:**
		- Mainly responsible for communicating with other routers and running routing protocols.
		- Used every time the topology of the network changes (e.g. when links are added or removed).
		- The result of those routing protocols (e.g. the [[0x07_Routing States#Forwarding Tables|forwarding table]]) can then be used by the data plane.
	3. The **management plane:**
		- Used to tell routers what to do, and see what they are doing.
		- Systems and humans interact with the management plane to configure and monitor the router. This is where operators can configure the device functionality.
		- Provides monitoring tools. How much traffic is being carried over each link? Has any physical component of the router failed? This information can be relayed back to the operator.
- Because the data plane and control plane operate at different time scales, and are running different protocols, the hardware and software of a router are optimized for different tasks. In practice, packets arrive much more frequently than the network topology changing.
- Therefore, the data plane is optimized for performing very simple tasks (table lookup and forwarding) very quickly. By contrast, the control plane is optimized for more complex tasks (re-computing paths in the network).
- The data plane and control plane operate in real-time, receiving and processing packets on the order of nanoseconds (data) and seconds (control).
- By contrast, the management plane works on the order of tens to hundreds of seconds. If the operator changes a configuration, the router might spend time performing validation checks and processing the configuration before fully applying the update.
- The **network management system (NMS)** is some piece of software run by the operator to interact with the routers.
- This software computes a network configuration (maybe with the help of manual operator input), and then applies that configuration to the routers. The router publishes some API that the system can use to talk to the router.
#### What’s Inside a Router?
- We defined a router as a computer that performs routing tasks, but in reality, inside the router, there are many smaller computers (e.g. CPUs, specialized chips) that work together to perform routing tasks.
- The physical shelf that makes up an industrial-size router is called a **chassis**. Inside the chassis, we install many **line cards**, and we have several physical ports on each line card. Each physical port can be used for either input or output.
	![router1](https://textbook.cs168.io/assets/routing/2-118-router1.png)
- Every physical port has to be connected to every other physical port in the router (both in the same linecard and other linecards). You might receive a packet through one port, and need to forward it out of a port on a different linecard.
- It would be pretty inefficient to physically wire each port to every other port. Instead, we have a fabric of wires to connect linecards together. Each linecard also has chips to facilitate connections to the fabric.
- Separate from all the linecards, we have a controller card with its own CPU, which talks with other routers to perform routing protocols. After running some algorithm to compute paths, the controller programs the forwarding chips with the correct forwarding table entries.
	![router2](https://textbook.cs168.io/assets/routing/2-119-router2.png)
- Each linecard has its own local CPU to control linecard functions (e.g. populate the forwarding table). The linecard also has hardware for basic processing of packets (e.g. updating its TTL before sending it out).
- The linecard contains one or more chips specifically optimized for forwarding.
	![router3](https://textbook.cs168.io/assets/routing/2-120-router3.png)
- We can also categorize the router components by the different planes.
	- The data plane is supported by forwarding chips on linecards, the fabric connecting linecards, and the fabric chips connecting the linecards to the fabric.
	- The control plane and management plane are supported by the controller card.3
	![router4](https://textbook.cs168.io/assets/routing/2-121-router4.png)
#### Types of Packets
- The most common packet is a **user packet**, containing data from an end host. When the router receives this packet, the forwarding chip first reads the destination field in the header and looks up the appropriate port.
- If that port is on a different linecard, the packet is sent through the fabric to the appropriate linecard. Once the packet reaches the correct linecard, the packet is sent along the appropriate port.
	![userPacket](https://textbook.cs168.io/assets/routing/2-123-user-traffic.png)
- Some packets are **control-plane traffic**, which are destined for the router itself. In particular, when we run routing protocols, advertisements are sent to the router itself.
- When the router receives this packet, the forwarding chip sends the packet up to the controller card. The CPU on the controller card processes the packet accordingly.
- The last type of traffic is **punt traffic**. These are user packets, but they require some **additional special processing**. For example, if we receive a packet with a TTL of 1, the packet has expired, and we shouldn’t forward it.
- We might also need to send an error message back to the sender. When the router receives a punt packet, the forwarding chip “punts” the packet to the controller card for special processing.
	![controlPuntTraffic](https://textbook.cs168.io/assets/routing/2-124-punt-traffic.png)
#### Linecard Functionality
- What specific tasks does a linecard need to do when it receives a packet?
- First, the linecard needs to take the signal (e.g. optical, electrical) and decode this signal into ones and zeros that make up the packet. This is the **PHY** part of the linecard, which handles the [[0x00_Layers of the internet#Layer 1 Physical Layer|physical layer (Layer 1)]] functionality.
- Once we have a sequence of ones and zeros, we have to read those bits and parse them. We might also have to perform other link-layer operations (e.g. if a link is connected to more than 2 machines). The **MAC** part of the linecard handles the [[0x00_Layers of the internet#Layer 2 Link Layer|link layer (Layer 2)]] functionality.
- Now that we have an IP packet, we have to parse the packet. For example, we need to check if the packet is IPv4 or IPv6. Then, we have to read the destination address and perform a lookup for forwarding (or discover that we need to punt the packet).
- We may also need to update various IP header fields. We have to decrease the TTL. Since we updated the header, we also need to update the checksum in the header. We might also need to update other fields like options and fragment.
	![linecardPipeline](https://textbook.cs168.io/assets/routing/2-126-queuing.png)
- In order to make these operations fast, forwarding chips are extremely specialized for the limited tasks that they perform (e.g. reading packet header, table lookup).
- You can’t write a general-purpose program and run it on a forwarding chip. If a packet requires functionality that the forwarding chip can’t support, we can always punt the packet to the general-purpose CPU on the controller card.
- Simple operations, like decrementing the TTL, are easy to implement in hardware. More complex operations, like special options, usually require punting to the controller card.
- In the modern Internet, we avoid special options whenever possible, in order to maximize use of the fast path and avoid punting (if we punted everything, controller cards would be overwhelmed).
#### Efficient Forwarding Table Lookup
- We now know that routers need to perform lookups in forwarding tables at extremely high rates. One major challenge is that our table entries can contain ranges of IP addresses (192.0.1.0/24) in addition to individual IP addresses.
- Also, these ranges could be overlapping (a destination could match multiple ranges). How can we make lookups extremely fast?
- Ideally, for maximum speed, the forwarding table could contain one entry per destination, with no ranges. Then, we just need to take the destination in the packet, and look up an exact match to learn the next hop.
- This is space-inefficient (remember, this is being implemented in hardware). Also, if a route changes, we’d have to update tons of entries in the table. Expanding routes isn’t going to work, so we’ll have to work with ranges.
- Recall that [[0x07_Routing States#Forwarding Tables|forwarding table]] is done using **longest prefix matching**. If multiple ranges match the destination, we pick the most specific range (most prefix bits fixed).
- If none of the ranges match, we pick the [[0x10_Addressing#Default Routes|default route]] ($*.*, 0.0.0.0/32)$, matches all destinations). If there’s no default route, we drop the packet.
- How do we implement longest prefix matching in hardware efficiently?
- First, for readability, we rewrite all the ranges and the destination in binary. Then, we scan the destination bits, one by one.
- We continue checking bit-by-bit, eliminating rows that don’t match, and confirming rows that are full matches. Eventually, we have one or more rows that match, and we pick the match with the longest prefix.
	![longestPrefixMatching](https://textbook.cs168.io/assets/routing/2-128-forwarding2.png)
- If we implemented this naively, then for every bit, we would have to match that bit against every entry in the forwarding table. The asymptotic runtime would scale with the number of entries in the forwarding table.
#### Efficient Lookup with Tries
- Tries are a data structure that efficiently store maps where the keys are strings (in this case, bitstrings). Tries store the key-value pairs by writing out the keys one character (bit) at a time, which enables efficient longest prefix matching.
- For example, this trie stores a map of words to numbers. If we want to find the longest prefix, we read the word one letter at a time.
- This allows us to trace a path down the tree, from the root to a leaf. Along this path, we look for all prefixes in the table (nodes with colors), and pick the longest prefix.
	![trie](https://textbook.cs168.io/assets/routing/2-130-trie2.png)
- We can use a similar approach for our forwarding table. Each layer of the trie represents one of the digits in the IP address. The zeroth layer is the root (empty string), the first layer represents the first bit, the second layer represents the second bit, etc.
- Each node in the trie represents a prefix. For example, the 2-bit prefix $11*$ is at the second layer of the tree, and the 3-bit prefix $100$ is at the third layer of the tree.
- The trie has all possible 3-bit prefixes. If a prefix is in the forwarding table, at the corresponding node, we write the next hop. If the prefix is not in the forwarding table, we don’t write anything in the node (in the picture, colored white).
	![IPTrie](https://textbook.cs168.io/assets/routing/2-131-trie3.png)
- Tracing the path down the tree can be done in **constant time**. We visit one node per bit of the destination address, and the destination address is **always 32 bits** (constant). Even if the forwarding table had millions of entries, we’d still pick out 32 nodes.
- If there’s no overlapping ranges, every valid prefix corresponds to a leaf node. If ranges are overlapping, a non-leaf node could also be a valid prefix.
- As before, we use the destination address to trace a path down the tree. If we fall off the tree, we stop early and pick the longest prefix out of the nodes we visited.
- As a slight optimization, as we walk down the tree, we could keep track of the longest prefix match seen so far. This will always be the most recent match, because the prefixes get longer as we move down the tree.
- If we fall off the tree, we use the longest prefix match (the most recent match we found).
- Note that the **default route would be stored in the root node** (0-length prefix). Our algorithm of walking down the tree ensures that we only use the default route if no other prefixes match.
	![defaultRouteIPTrie](https://textbook.cs168.io/assets/routing/2-133-trie5.png)
# Sources
- [Lecture 8 - Routing 4: Routers](https://www.youtube.com/watch?v=KWU_8UY2cOM).
- [Router Hardware](https://textbook.cs168.io/routing/router.html).