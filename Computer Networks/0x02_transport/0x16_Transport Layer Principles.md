# Explanation
- Many applications require reliability. For example, when sending a file over the Internet, we want the recipient to receive the same bytes in the same order as what the sender sent.
- However, [[0x00_Layers of the internet#Layer 3 Internet Layer|Layer 3]] only provided unreliable, [[0x00_Layers of the internet#Layer 3 Best-Effort Service Model|best-effort]] packet delivery. [[0x00_Layers of the internet#Layer 3 Packets Abstraction|Packets]] can be lost (dropped), corrupted, and reordered (order of packets sent doesn’t match order of packets received). Packets can be delayed (e.g. a packet could stuck in a queue waiting to cross a link).
- In rare cases, packets can even be duplicated, where the sender sends one packet but the recipient receives multiple copies of that packet. This usually happens if a router along the path encounters an error of some sort. In practice, this error is very rare.
- Reliability is implemented in the operating system for convenience, so that applications don’t need to all re-implement their own reliability.
- We will formalize reliability by defining **at-least-once delivery**. In this model, the destination must receive every packet, without corruption, at least once, but may receive multiple duplicate copies of a packet.
- [[0x00_Layers of the internet#Layer 4 Transport|The transport layer]] will use the best-effort delivery to provide at-least-once delivery. Then, using at-least-once delivery, our protocol can remove duplicates and provide exactly-once delivery to the applications.
- Note that reliable delivery does not guarantee that packets will be sent. A computer not connected to the network cannot send data to the destination, no matter what reliability protocol we use.
- Reliability protocols are allowed to give up and **fail to send a packet**, but the failure must be reported to the application. The protocol cannot falsely claim to have successfully delivered a packet.
#### Bytestream Abstraction
- Implementing reliability at the transport layer means that the application developer no longer needs to think in terms of individual limited-size packets being sent across the network.
- Instead, the developer can think in terms of a **reliable in-order bytestream**. The sender has a stream of bytes with no length limit, and provides this stream to the transport layer.
- Then, the recipient receives the exact same stream of bytes, in the same order, with no bytes lost. You can think of a bytestream as a pipe, where the sender inserts bytes, one by one, into the pipe, and those same bytes appear, one by one, on the recipient’s end of the pipe.
- The sender and recipient don’t need to think about re-sending lost packets or packets arriving out of order, because the transport layer protocol will implement that for the developer.
	![byteStream](https://textbook.cs168.io/assets/transport/3-004-bytestream.png)
#### UDP and Datagrams
- Sometimes, applications don’t need reliability. For example, consider a sensor that reads the water pressure in your home. The sensor sends a reading (small, fixed-size message with the time and water pressure) to the utility company every minute.
- This system might not need packets to arrive in order (e.g. if the readings already include timestamps). The system might not even need reliability, as long as most of the readings arrive at the utility company.
- Applications that don’t need reliability can use **UDP** (User Datagram Protocol) instead of TCP at the transport layer. UDP does not provide reliability guarantees.
- If the application needs a packet to arrive, the application must handle re-sending packets on its own (the transport layer will not re-send packets).
- Messages in UDP are limited to a single packet. If the application wants to send larger messages, the application is responsible for breaking up and reassembling those messages.
- At the transport layer, you can choose to use either UDP and TCP depending on your needs, but you can’t choose to use both. UDP and TCP are the standard transport layer protocols in the modern Internet.
	![tcpUdp|250](https://textbook.cs168.io/assets/transport/3-006-tcp-features.png)
# Sources
- [Lecture 11 - Transport 1: TCP I](https://www.youtube.com/playlist?list=PL0_XloRC3MWuroYx9jW6ZHw4F_tymsGUH).
- [Transport Layer Principles](https://textbook.cs168.io/transport/reliability.html).