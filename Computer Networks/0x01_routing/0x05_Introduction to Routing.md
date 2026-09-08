# Explanation

#### Inter-Domain and Intra-Domain Routing
- One possible strategy for routing is to build a model of the Internet that includes every single machine in the world, and design a single giant routing protocol that will allow us to send packets anywhere in the world. However, this is infeasible in practice because of the scale of the Internet.
- Instead, we’ll take advantage of the fact that the Internet is a network of networks. Each local network implements its own routing protocol that specifies how to send packets within just that local network.
- Then, we can connect up all those local networks and implement a routing protocol across all the local networks, specifying how to send packets between different local networks.
- With the network of networks model, we can let individual local networks choose a routing strategy for packets within their network. Each operator can choose the protocol that works best for them.
- The protocols for routing packets within a local network are called **[[0x06_Model for Intra-Domain Routing|intra-domain]]** routing protocols, or **interior gateway protocols (IGPs)**. Real-world examples include OSPF (Open Shortest Path First) and IS-IS (Intermediate System to Intermediate System).
- By contrast, protocols for routing packets across different networks are called **inter-domain** routing protocols, or **exterior gateway protocols (EGPs)**. In order to support sending packets across different local networks, every network needs to agree to use the same protocol for routing packets between each other.
- If different networks used different inter-domain protocols, there’s no guarantee that the entire Internet could be connected in a consistent way.
- Because every network must agree to use the same inter-domain protocol, there is only one protocol implemented at scale on the Internet, namely **BGP (Border Gateway Protocol)**.
	![intradomain](https://textbook.cs168.io/assets/routing/2-003-intradomain.png)
- This model of interior and exterior gateway protocols is convenient for intuition, but in practice, there is not always a clear distinction between them. For example, BGP is sometimes also used inside a local network, in addition to between different networks.
# Sources
- [Lecture 4 - Routing 1: Principles](https://www.youtube.com/watch?v=aEkZKsOWvzE).
- [Introduction to Routing](https://textbook.cs168.io/routing/intro.html).