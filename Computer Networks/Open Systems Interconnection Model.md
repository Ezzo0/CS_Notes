# Explanation
- OSI is a reference model, which means that it consists of number of layers.
- Reference model was invented for:
	- Standardization.
	- Better understanding of data transfer.
- Layers is defined as actions on data.
- In each layer there are some protocols.
- OSI model consists of 7 layers:
	- OSI layers have interactions between each other, this interaction is called adjacent layer interaction.
	- OSI layers have interactions between same layer in source and destination, this interaction is called same layer interaction.
- OSI Layers:
	1. **[[Physical Layer]]**
	2. **[[Datalink Layer]]**
		-  Datalink layer is divided into 2 sublayers:
			1. LLC (logical link control) 
				-  This sub-layer will negotiate with upper layer to get information about used CLNS(IP).
			2. Physical addressing (MAC (Media Access Control))
				- Every segment of data will have MAC of source and MAC of destination.
				- MAC address is 48bit.
		- Datalink layer is responsible for hop-to-hop delivery.
	3. **[[Network Layer]]**
		- Network layer do
			1. Logical addressing
				- Every segment of data will have CLNS (IP) of source and CLNS (IP) of destination.
			2. End-to-End delivery.
	4. **Transport Layer**
		 - There are two protocols inside Transport layer
			1. **_TCP (Transmission control protocol)_** 
				- TCP is reliable protocol.
			2. **_UDP (User datagram protocol)_**
				- UDP is unreliable protocol.
		-  Transport layer do
			1. Segmentation: Split message into chunks.
			2. Sequencing: Give a number to each chunk.
			3. Session addressing (Source port & Destination port).
			4. Assembling chunks in receiving state.
			5. Flow control (windowing, buffering, congestion avoidance)
				-  The destination and source negotiate about buffer section in destination.
				- After negotiation the source send segments till the the destination's buffer is full.
				- After that, destination process the sent segments, empty buffer, and send acknowledge.
			6.  Error correction
	5. **Session Layer**
		-  Session layer do:
			1. Creates logical session.
			2. Terminate logical session.
			3. Defines communication mode (management of logical session):
				-  There are 3 types of communication modes:
					1. Single
						- Source always send data and never receive data.
					2. Half duplex
						- Data is sent and received on the same carrier on different time. 
					3. Full Duplex
						- Data is sent and received on different carriers.
	6. **Presentation Layer**
		- Presentation layer do:
			1. Compression / decompression
				- Source will compress the data and destination will decompress it.
			2. Encoding / decoding 
				- Source will Encode data from ASCII code to machine code and destination will decode it. 
			3. Defines data format
				- If data is presented like this: 01010101, then source will provide some numbers before data to state its format like the following: 110101010111.
	7. **Application Layer**
		- It is the first layer that interact with the user.
		- It is responsible for communication between user and network.
		![[osi_model_7_layers.png]]
- #### TCP/IP model
	- TCP/IP  model has 4 layers until 2010, now it has 5 layers.
		![[TCP-IP_OSI.png|500]]
- #### Encapsulation
	- Data is encapsulated by different headers in different layers.
	 ![[data encapsulation.png]]
	- Segment header        ==> TCP or UDP header.
	- Packet header            ==> IP header.
	- Frame header            ==> Ethernet header.
	- Frame trailer              ==> FCS (frame check sequence) trailer (used to detect error).
	- **In OSI model**:
		- data                    ==> PDU (protocol data unit).
		- Segment header ==> L4PDU (session address which is source port & destination port).
		- Packet header     ==> L3PDU (Source IP & destination IP).
		- Frame header     ==> L2PDU (Source MAC & destination MAC).
- #### Devices
	 ![[devices_with_layer.png]]
# Sources
- [2- Windows Server 2019 - OSI Model](https://www.youtube.com/watch?v=PUahuNxkQoo&list=PLped9VG7STA9vZFYwVIIbnSdfu8gKUpm2&index=3)
- [04- CCNA 200-301-Sem-1 (TCP/IP Model, OSI Model, and Protocols - Part-1) شرح بالعربى](https://www.youtube.com/watch?v=cuAuHsx0bcc&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=6&t=346s)
- [05- CCNA 200-301-Sem-1 (TCP/IP Model, OSI Model, and Protocols - Part-2) شرح بالعربى](https://www.youtube.com/watch?v=8pu0nh9BZnU&list=PLyDiLBk6tDH48IAmjAfH8OhrIeTSf23id&index=7&t=1s)