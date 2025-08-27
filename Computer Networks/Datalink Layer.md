# Explanation
+ The Data Link layer is responsible for communications between end-device network interface cards (hop-to-hop delivery).
+ It allows upper layer protocols to access the physical layer media and encapsulates Layer 3 packets (IPv4 and IPv6) into Layer 2 Frames (physical addressing).
+ It also performs error detection and rejects corrupts frames (for example via CRC).
+ The Data Link Layer consists of two sublayers: 
	1. Logical Link Control (LLC).
		+ The LLC sublayer communicates between the networking software at the upper layers and the device hardware at the lower layers. 
	2. Media Access Control (MAC).
		+ The MAC sublayer is responsible for data encapsulation and media access control. 
+ #### Access Control Methods
	+ **Contention-based access**
		+ All nodes operating in half-duplex, competing for use of the medium. Examples are:
			+ Carrier sense multiple access with collision detection (CSMA/CD) as used on legacy bus-topology Ethernet.
			+ Carrier sense multiple access with collision avoidance (CSMA/CA) as used on Wireless LANs. 
	+ **Controlled access**
		+ Deterministic access where each node has its own time on the medium.
		+ Used on legacy networks such as Token Ring and ARCNET.
- #### Data Link Frame
	- Data is encapsulated by the data link layer with a header and a trailer to form a frame.
	- A data link frame has three parts:
		1. Header
		2. Data
		3. Trailer
	- The fields of the header and trailer vary according to data link layer protocol.
	- The amount of control information carried within the frame varies according to access control information and logical topology.
	 ![[Data Link Frame.png]]
# Sources
- [08- CCNA 200-301-Sem-1(Data Link Layer - Layer 2 in OSI Model) شرح بالعربى](https://www.youtube.com/watch?v=vhaOBteimIc&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=10&t=1s)
 