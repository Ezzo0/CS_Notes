# Explanation
+ Physical Layer Standards address three functional areas:
	1. Encoding:
		- Encoding converts the stream of bits into a format recognizable by the next device in the network path. 
		- Examples of encoding methods include Manchester form:
		 ![[manchester_encoding.png|400]]
	2. Signaling:
		- The signaling method is how the bit values, “1” and “0” are represented on the physical medium. 
		- The method of signaling will vary based on the type of medium being used. 
	3. Physical Components:
		+ The Physical Components are the hardware devices, media, and other connectors that transmit the signals that represent the bits.
		+ Hardware components like NICs, interfaces and connectors, cable materials, and cable designs are all specified in standards associated with the physical layer.
+ Bandwidth is the capacity at which a medium can carry data.
+ Physical media properties, current technologies, and the laws of physics play a role in determining available bandwidth.
+ Latency is the amount of time, including delays, for data to travel from one given point to another.
+ Throughput is the measure of the transfer of bits across the media over a given period of time.
+ Goodput is the measure of usable data transferred over a given period of time. 
	+ Goodput = Throughput - traffic overhead.
+ #### Copper Cabling
	+ It is inexpensive, easy to install, and has low resistance to electrical current flow. 
	+ **Limitations:**
		+ Attenuation – the longer the electrical signals have to travel, the weaker they get.
		+ The electrical signal is susceptible to interference from two sources, which can distort and corrupt the data signals (Electromagnetic Interference (EMI) and Radio Frequency Interference (RFI) and Crosstalk).
	+ **Mitigation:**
		+ Strict adherence to cable length limits will mitigate attenuation.
		+ Some kinds of copper cable mitigate EMI and RFI by using metallic shielding and grounding.
		+ Some kinds of copper cable mitigate crosstalk by twisting opposing circuit pair wires together.
	- **Types of Copper Cabling**
		- ![[Unshielded Twisted Pair (UTP).png]]
		- ![[Properties of UTP Cabling.png]]
		- ![[Straight-through and Crossover UTP Cables.png]]
		- ![[Shielded Twisted Pair (STP).png]]
		- ![[Coaxial Cable.png]]
- #### Fiber-Optic Cabling
	- Not as common as UTP because of the expense involved.
	- Transmits data over longer distances at higher bandwidth than any other networking media.
	- Less susceptible to attenuation, and completely immune to EMI/RFI.
	- Made of flexible, extremely thin strands of very pure glass.
	- Uses a laser or LED to encode bits as pulses of light.
	- ![[Types of Fiber Media.png]]
- #### Wireless Media
	- It carries electromagnetic signals representing binary digits using radio or microwave frequencies.
	- **Some of the limitations of wireless:**
		- Coverage area.
		- Interference - Wireless is susceptible to interference and can be disrupted by many common devices.
		- Security - Wireless communication coverage requires no access to a physical strand of media, so anyone can gain access to the transmission.
		- Shared medium - WLANs operate in half-duplex, which means only one device can send or receive at a time. Many users accessing the WLAN simultaneously results in reduced bandwidth for each user.
# Sources
+ [06- CCNA 200-301-Sem-1 (Physical Layer - Layer 1 in OSI Model) شرح بالعربى](https://www.youtube.com/watch?v=_WCBYOAs0ac&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=8&t=4s)
 