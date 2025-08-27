# Explanation
- Ethernet operates in the [[Datalink Layer]] and the [[Physical Layer]].
- The 802 LAN/MAN standards, including Ethernet, use two separate sublayers of the data link layer to operate:
	- LLC Sublayer: (IEEE 802.2) Places information in the frame to identify which [[Network Layer]] protocol is used for the frame.
	- MAC Sublayer: (IEEE 802.3, 802.11, or 802.15) Responsible for data encapsulation and media access control, and provides [[Datalink Layer]] addressing.
		- IEEE 802.3 data encapsulation includes the following:
			1. Ethernet frame - This is the internal structure of the Ethernet frame.
			2. Ethernet Addressing - The Ethernet frame includes both a source and destination MAC address to deliver the Ethernet frame from Ethernet NIC to Ethernet NIC on the same LAN.
			3. Ethernet Error detection - The Ethernet frame includes a frame check sequence (FCS) trailer used for error detection.
			 ![[network,datalink,physical layers diagram.png|500]]
- The minimum Ethernet frame size is 64 bytes and the maximum is 1518 bytes (The preamble field is not included when describing the size of the frame).
- Any frame less than 64 bytes in length is considered a “collision fragment” or “runt frame” and is automatically discarded. 
- Frames with more than 1500 bytes of data are considered “jumbo” or “baby giant frames”.
- If the size of a transmitted frame is less than the minimum, or greater than the maximum, the receiving device drops the frame. 
- Dropped frames are likely to be the result of collisions or other unwanted signals. They are considered invalid. 
- Jumbo frames are usually supported by most Fast Ethernet and Gigabit Ethernet switches and NICs.
	 ![[Ethernet_frame.png|500]]
- MSS is expressing the size of segment.
- MTU is expressing the size of packet.
- MRU is expressing the size of the frame.
- An Ethernet MAC address consists of a 48-bit binary value, expressed using 12 hexadecimal values.
- All MAC addresses must be unique to the Ethernet device or Ethernet interface. 
- To ensure this, all vendors that sell Ethernet devices must register with the IEEE to obtain a unique 6 hexadecimal (i.e., 24-bit or 3-byte) code called the organizationally unique identifier (OUI).
- An Ethernet MAC address consists of a 6 hexadecimal vendor OUI code followed by a 6 hexadecimal vendor-assigned value.
 ![[OUI_vendor.png|500]]
- When a device is forwarding a message to an Ethernet network, the Ethernet header include a Source MAC address and a Destination MAC address.
- When a NIC receives an Ethernet frame, it examines the destination MAC address to see if it matches the physical MAC address that is stored in RAM and if there is no match, the device discards the frame. If there is a match, it passes the frame up the OSI layers, where the de-encapsulation process takes place.
- In Ethernet, different MAC addresses are used for Layer 2 unicast, broadcast, and multicast communications.
	1. Unicast communication:
		- A unicast MAC address is the unique address that is used when a frame is sent from a single transmitting device to a single destination device. 
		- The process that a source host uses to determine the destination MAC address associated with an IPv4 address is known as Address Resolution Protocol (ARP). The process that a source host uses to determine the destination MAC address associated with an IPv6 address is known as Neighbor Discovery (ND).
		 ![[unicast_ethernetFrame.png|500]]
	2. Broadcast communication:
		- It has a destination MAC address of FF-FF-FFFF-FF-FF in hexadecimal (48 ones in binary).
		- It is flooded out all Ethernet switch ports except the incoming port and not forwarded by a router.
		- If the encapsulated data is an IPv4 broadcast packet, this means the packet contains a destination IPv4 address that has all ones (1s) in the host portion. This numbering in the address means that all hosts on that local network (broadcast domain) will receive and process the packet.
		 ![[Broadcast_ethernetFrame.png|500]]
	3. Multicast communication
		- An Ethernet multicast frame is received and processed by a group of devices that belong to the same multicast group.
		- There is a destination MAC address of 01-00-5E when the encapsulated data is an IPv4 multicast packet and a destination MAC address of 33-33 when the encapsulated data is an IPv6 multicast packet.
		- There are other reserved multicast destination MAC addresses for when the encapsulated data is not IP, such as Spanning Tree Protocol (STP).
		- Because multicast addresses represent a group of addresses (sometimes called a host group), they can only be used as the destination of a packet. The source will always be a unicast address.
		 ![[Multicast_ehternetFrame.png|500]]
- Switches use one of the following forwarding methods for switching data between network ports:
	- Store-and-forward switching: This frame forwarding method receives the entire frame and computes the CRC. If the CRC is valid, the switch looks up the destination address, which determines the outgoing interface. Then the frame is forwarded out of the correct port.
	- Cut-through switching: This frame forwarding method forwards the frame before it is entirely received. At a minimum, the destination address of the frame must be read before the frame can be forwarded.
		- The switch does not perform any error checking on the frame.
		- There are two variants of cut-through switching:
			1. Fast-forward switching - Offers the lowest level of latency by immediately forwarding a packet after reading the destination address. 
			2. Fragment-free switching - A compromise between the high latency and high integrity of store-and-forward switching and the low latency and reduced integrity of fast-forward switching, 
				- The switch stores and performs an error check on the first 64 bytes of the frame before forwarding. 
				- Because most network errors and collisions occur during the first 64 bytes, this ensures that a collision has not occurred before forwarding the frame.
- A big advantage of store-and-forward switching is that it determines if a frame has errors before propagating the frame. When an error is detected in a frame, the switch discards the frame.
- Discarding frames with errors reduces the amount of bandwidth consumed by corrupt data.
- An Ethernet switch may use a buffering technique to store frames before forwarding them or when the destination port is busy because of congestion.
![[Memory Buffering.png]]
- There are two types of duplex settings used for communications on an Ethernet network:
	- Full-duplex - Both ends of the connection can send and receive simultaneously.
	- Half-duplex - Only one end of the connection can send at a time.
- Autonegotiation is an optional function found on most Ethernet switches and NICs. It enables two devices to automatically negotiate the best speed and duplex capabilities.
- Gigabit Ethernet ports only operate in full-duplex.
# Sources
- [09- CCNA 200-301-Sem-1 (Ethernet Protocol - Ethernet Switching) شرح بالعربى](https://www.youtube.com/watch?v=8DyRPjpC4_Y&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=10&t=1s)
