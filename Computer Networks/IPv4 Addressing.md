# Explanation
- An IPv4 address is a 32-bit hierarchical address that is made up of a network portion and a host portion.
- A subnet mask is used to determine the network and host portions.
 ![[IPv4_portion.png|500]]
- To identify the network and host portions of an IPv4 address, the subnet mask is compared to the IPv4 address bit for bit, from left to right.
- To identify the network address, the host IPv4 address is logically ANDed, bit by bit, with the subnet mask to identify the network address.
	  ![[IPv4_networkAddress.png|500]]
	  ![[IPv4_subnetmask.png|1000]]
- A prefix length is another method used to identify a subnet mask address.
- The prefix length is the number of bits set to 1 in the subnet mask.
 ![[prefixLength.png|500]]
- Unicast Communication:
	 ![[IPv4_unicast.png]]
- Broadcast Communication:
	- Direct Broadcast Address in Example is: 172.16.4.255
	- Indirect Broadcast Address in Example is: 255.255.255.255
	 ![[IPv4_broadcast.png]]
- Multicast Communication:
	- To send a message in multicast group, IPv4 must start with 244. group number. As in the example destination IPv4 is 224.10.10.5 (.10.10.5 is the group number).
	 ![[IPv4_multicast.png]]
- As defined in in RFC 1918, public IPv4 addresses are globally routed between internet service provider (ISP) routers.
- Private addresses are common blocks of addresses used by most organizations to assign IPv4 addresses to internal hosts.
- Private IPv4 addresses are not unique and can be used internally within any network.
- Private addresses are not globally routable.
 ![[privateIP.png|400]]
- Network Address Translation (NAT) translates private IPv4 addresses to public IPv4 addresses.
- NAT is typically enabled on the edge router connecting to the internet.
- Loopback addresses is used to `ping` yourself: 
	- 127.0.0.0 /8 (127.0.0.1 to 127.255.255.254)
	- Commonly identified as only 127.0.0.1
	- Used on a host to test if TCP/IP is operational.
- Link-Local addresses: 
	- Commonly known as the Automatic Private IP Addressing (APIPA) addresses or selfassigned addresses.
	- Used by Windows DHCP clients to self-configure when no DHCP servers are available.
	- 169.254.0.0 /16 (169.254.0.1 to 169.254.255.254).
- RFC 790 (1981) allocated IPv4 addresses in classes:
	- Class A (0.0.0.0/8 to 127.0.0.0/8)
	- Class B (128.0.0.0 /16 – 191.255.0.0 /16)
	- Class C (192.0.0.0 /24 – 223.255.255.0 /24)
	- Class D (224.0.0.0 to 239.0.0.0)
	- Class E (240.0.0.0 – 255.0.0.0)
- Class A, B, and C are addresses for network devices.
- Class D is for multicast.
- Class E is for experiment.
- Classful addressing wasted many IPv4 addresses. 
- Classful address allocation was replaced with classless addressing which ignores the rules of classes (A, B, C).
- The Internet Assigned Numbers Authority (IANA) manages and allocates blocks of IPv4 and IPv6 addresses to five Regional Internet Registries (RIRs).
- RIRs are responsible for allocating IP addresses to ISPs who provide IPv4 address blocks to smaller ISPs and organizations.
- 
# Sources
- [14- CCNA 200-301-Sem-1 (IPv4 Addressing - Subnetting - FLSM - VLSM - Part-1) شرح بالعربى](https://www.youtube.com/watch?v=umYMAWjNdwE&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=15)
- [15- CCNA 200-301-Sem-1 (IPv4 Addressing - Subnetting - FLSM - VLSM - Part-2) شرح بالعربى](https://www.youtube.com/watch?v=Mnw9rg_oUx8&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=16)
- [Module 11: IPV4 Addressing](https://www.youtube.com/watch?v=NS0ho8Hh6aA&t=1250s)
