# Explanation 
- Network is a collection of devices that are connected together to provide a certain service to the user.
- ##### Network Services
	- Data Sharing
	- Resource Sharing
	- Modern Technology (VOIP, Video conference, Clouding, BYOD, IOT, etc....)
- ##### Network Components
	- End devices (PC, IP Phone, Smart Phone, Printer, etc...)
	- Network Device or Intermediate Devices (Switch, Hub, Router, Bridge, Repeater, wireless access points, AP, Firewall)
		- An intermediary device interconnects end devices.
		- Management of data as it flows through a network including:
			- Regenerate and retransmit data signals.
			- Maintain information about what pathways exist in the network.
			- Notify other devices of errors and communication failures.
	- Medium (Can be wire or wireless)
		 ![[media types and description.png|400]]
- ##### Network Topologies
	- The topology of a network is the arrangement, relationship of the network devices, and the interconnections between them.
	- There are two types of topologies used when describing networks:
		1. Physical topology – illustrate the physical location of intermediary devices and cable installation.
		2. Logical topology – identifies the virtual connections between devices using device interfaces and IP addressing schemes (illustrate devices, ports, and the addressing scheme of the network).
	+ ###### LAN Topologies:
		- **Bus Topology**
			- Co-axial cables were used to connect components in bus topology.
			- End devices are connected to bus by **T connector**.
			 ![[bus-topology-vs-ring-topology1.png]]
			- Terminator is used to drop signal at the two ends of the bus to avoid looping.
			- If a device isn't the destination, it receives and processes the data transmitted by another device.
			- To avoid collision, **CSMA/CD** (Carrier sense multiple access / Collision detection) is used by each device.
			- CSMA/CD is a software on network interface card.
			- If a collision situation occurred , a signal called **Jam signal** will be send on the bus by the devices that caused the collision.
			- After receiving jam signal, All devices will not send any data on the bus and run an algorithm called **Back off Algorithm**.
			- Back off algorithm set a different random delay for each device to ensure collision avoidance.
			- Bus physical topology is **_BUS_**.
			- Bus logical topology is **_BUS_**.
		- **Star Topology**
			- **_HUB_**:
				- If data was sent by a device to hub, hub will send this data to all devices except the one that sent it.
				- HUB star physical topology is **_STAR_**.
				- HUB star logical topology is **_BUS_**.
				- CSMA/CD is also used in HUB star topology.
				- HUB star topology broadcast domain is **_1_**.
				- HUB star topology collision domain is **_1_**.
				 ![[hub_star_topology.png]]
			- **_SW_**:
				- Each device send Data divided into pieces.
				- when data is sent to switch, switch use the first piece of data only to build MAC address-table by send this piece to all devices.
				- MAC-address-table is built by the source devices only.
				- After this step, switch send data to the destination device only if it is in MAC address-table.
				- Information of the device in MAC address-table will be available only if the device send data again in time less than aging-time which is equal to 300 sec by default. 
				- By default, there is no collision on switch.
				- SW star physical topology is **_STAR_**.
				- SW star logical topology is **_BUS_**.
				- SW star topology broadcast domain is **_1_**.
				- SW star topology collision domain is **_No. of interfaces_**.
				 ![[Star_Topology_Switch.png|300]]
		- **Ring Topology**
			- In ring topology, data is sent from one device to the next one until it reach the destination.
			- **_FDDI_**
				 ![[fddi_ring_topology.gif]]
				- FDDI ring physical topology is **_ring_**.
				- FDDI ring logical topology is **_ring_**.
			- **_Token Ring_**
				- Token is sent to each device by order. 
				- If the device want to send data, it put data on the token.
				- If the device is the destination, it take data from token.
				- Token ring physical topology is **_star_**.
				- Token ring logical topology is **_ring_**.
				 ![[token_ring.png]]
			
	- #### WAN Topologies
		+ There are three common physical WAN topologies:
			+ Point-to-point – the simplest and most common WAN topology. Consists of a permanent link between two endpoints.
			+ Hub and spoke – similar to a star topology where a central site interconnects branch sites through point-to-point links.
			+ Mesh – provides high availability but requires every end system to be connected to every other end system.
				+ Nodes connect directly, dynamically and non-hierarchically to as many other nodes as possible and cooperate with one another to efficiently route data to and from clients and achieve high availability.
				- It is also called redundant topology.
				 ![[mesh_topology.jpg|300]]
- #### Common Types of Networks
	- Network infrastructures vary greatly in terms of:
		1. Size of the area covered
		2. Number of users connected
		3. Number and types of services available
		4. Area of responsibility
	- **Two most common types of networks:**
		- Local Area Network (LAN)
		- Wide Area Network (WAN).
	 ![[LAN & WAN.png]]
	- **Intranets and Extranets**
		 ![[Intranet & Extranet.png|900]]
- **Network Architecture**
	- Network Architecture refers to the technologies that support the infrastructure that moves data across the network.
	- There are four basic characteristics that the network architectures need to address to meet user expectations:
		1. Fault Tolerance.
			- A fault tolerant network limits the impact of a failure by limiting the number of affected devices. 
			- Multiple paths are required for fault tolerance.
			- Reliable networks provide redundancy by implementing a packet switched network:
				- Packet switching splits traffic into packets that are routed over a network.
				- Each packet could theoretically take a different path to the destination.
			 ![[Fault Tolerance.png]]
		2. Scalability.
			- A scalable network can expand quickly and easily to support new users and applications without impacting the performance of services to existing users.
		3. Quality of Service (QoS). 
		4. Security.
			- There are two main types of network security that must be addressed:
				1. Network infrastructure security:
					-  Physical security of network devices.
					- Preventing unauthorized access to the devices.
				2. Information Security:
					- Protection of the information or data transmitted over the network .
			- Three goals of network security:
				- Confidentiality – only intended recipients can read the data.
				- Integrity – assurance that the data has not be altered with during transmission.
				- Availability – assurance of timely and reliable access to data for authorized users.
# Sources
- [1- Windows Server 2019 | Network Topologies](https://www.youtube.com/watch?v=LULwpR-UCKE&list=PLped9VG7STA9vZFYwVIIbnSdfu8gKUpm2&index=3)
- [01- CCNA 200-301-Sem-1 (Introduction to Network Today) شرح بالعربى](https://www.youtube.com/watch?v=L_ynA1b8lk4&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=3)
