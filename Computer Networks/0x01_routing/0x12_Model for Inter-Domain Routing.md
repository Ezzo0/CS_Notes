# Explanation
- The notion of a local network can be formalized by defining an **autonomous system (AS)**, which is one or more local network(s) all run by the same operator.
- For example, within a company like Google, there might be a local network for employee computers, and another local network for data centers, but both networks are controlled by the same company.
- The operator can deploy a single [[0x06_Model for Intra-Domain Routing|intra-domain routing]] protocol to send messages between machines on any of those local networks.
- Sometimes, the term **domain** is used to informally refer to an AS, though this term is also used in other protocols, so we will say AS when possible.
- To think about routing packets between autonomous systems, we can abstract away all of the individual routers and hosts within the AS, and treat the AS as a single entity.
- Then, we can draw a graph where each node represents an AS, and edges between two ASes represents a connection between them. This graph is sometimes called the **inter-domain topology** or an **AS graph**.
	![interDomainTopology](https://textbook.cs168.io/assets/routing/2-134-interdomain.png)
#### Types of ASes
- A **stub autonomous system** only exists to provide Internet connectivity to the hosts in its local networks. A stub AS only sends and receives packets on behalf of hosts that are inside the AS, and does not forward packets between different ASes.
- These are analogous to the [[0x06_Model for Intra-Domain Routing#^a761ab|end hosts]] in our intra-domain routing model, which only sent and received their own packets and did not forward other people’s packets.
- Real-life examples of stub ASes include non-Internet companies (e.g. a bank offering connectivity to its employees) or universities (e.g. UC Berkeley offering connectivity to its students and employees).
- By contrast, a **transit autonomous system** forwards packets on behalf of other ASes. A transit AS could carry a packet between two different ASes by receiving and forwarding that packet.
- Transit ASes correspond to real-life companies whose business includes selling Internet connectivity to other organizations. Real-life examples of transit ASes include AT&T and Verizon, which are companies that you can pay to offer you Internet connectivity.
- Note that a transit AS can still **contain end hosts** that send and receive packets of their own. Nevertheless, a transit AS is similar to the [[0x06_Model for Intra-Domain Routing#^75153e|routers]] in our intra-domain routing model, which received other users’ packets and forwarded them on behalf of users.
#### Inter-Domain Topology Is Defined by Business Relationships
- There are two possible ways that a pair of ASes could be related. 
	1. **_A pair of ASes could be involved in a customer-provider relationship._** 
		- In real life, the **customer** is paying for service, and the **provider** is offering connectivity in exchange for money. For example, the local bank AS could be the customer, paying the provider, Verizon, for Internet services.
	2. **_A pair of ASes could also be involved in a peer relationship._**
		- Two peer ASes usually send each other a roughly equal amount of traffic. In real life, two ASes could agree to become peers by signing a legal contract between companies.
		- Usually, the two peers agree to not pay each other for connectivity services, as long as the traffic sent in either direction is roughly equal.
- We can draw these relationships into the AS graph by adding arrows to the graph.
	- A **directed edge** points from the provider to the customer.
	- An **undirected edge** connects two peers. Note that the graph can contain both directed and undirected edges (not all edges need to have arrows).
	 ![asGraph|300](https://textbook.cs168.io/assets/routing/2-137-as-graph.png)
- Note that the direction of the arrow **does not tell us anything** about what direction the packets are being sent. In fact, packets can be sent in both directions even along a directed edge. The customer often pays the provider for the ability to send packets to and receive packets from the rest of the Internet.
- The graph of customer-provider relationships is **acyclic**. The graph does not contain any cycles consisting of directed edges.
- This acyclic property exists because of the real-world implications of having a cycle. In real life, a cycle would mean that A pays B, B pays C, and then C pays A, and it doesn’t make sense for money to flow from somebody back to themselves.
- Also, this cycle would mean that A provides service to C, which provides service to B, which provides service to A. It also doesn’t make sense for somebody to provide connectivity to themselves.
- Note that the acyclic property only applies to customer-provider relationships. It is okay if peering relationships form a cycle.
	![acyclicAsGraph|500](https://textbook.cs168.io/assets/routing/2-138-acyclic.png)
#### Provider Hierarchy and Tier 1 ASes
- A consequence of the graph being acyclic is, we can form a hierarchy of providers. In other words, we can arrange the nodes such that all the arrows point downward.
- The stub ASes are at the bottom, the providers are at the top. Service flows from higher to lower nodes. The lower nodes pay money up to higher nodes.
- At the very top of the hierarchy, there are **Tier 1 autonomous systems**, which have no providers (no incoming edges). Every Tier 1 AS has a peering relationship with every other Tier 1 AS.
	![tier1AS|400](https://textbook.cs168.io/assets/routing/2-139-tier1.png)
- In this hierarchy, starting from any AS, and following the uphill chain of providers, always leads to a Tier 1 AS. This also makes sense in real life. The Tier 1 ASes all peering with each other is why the entire Internet is connected.
- In order to guarantee having a path to every other AS in the graph, every AS must have a path upwards that eventually leads to a Tier 1 AS.
#### Policy-Based Routing
- Recall that in intra-domain routing, our goal was to find paths that are [[0x07_Routing States#Routing State Validity Definition|valid]] and [[0x07_Routing States#Least-Cost Routing|good]]. In inter-domain routing, we still want paths to be valid.
- However, unlike in intra-domain routing, where there was nothing special about one router over another, each autonomous system has its own business goals and relationships with other ASes (e.g. customer, provider, peer). Therefore, we will need to re-define “good” to reflect the real-world business goals and preferences of ASes.
- In order to allow each AS to carry traffic in a way that’s compatible with its real-world goals, our routing protocol will allow each AS to set its own policy. Then, the paths computed by the protocol should properly respect each AS’s policy.
- In theory, ASes can set any sort of policy that they like, although standard conventions do exist (which we’ll discuss next). Here are some examples of policies that an AS could set:
	- “I don’t want to carry AS#2046’s traffic through my network.” (Defining how I will handle traffic from other ASes.)
	- “I prefer if my traffic was carried by AS#10 instead of AS#4.” (Defining how other ASes should handle my traffic.)
	- “Don’t send my traffic through AS#54 unless absolutely necessary.”
- The routing protocol doesn’t care why the AS has these preferences. Perhaps I’m refusing to carry traffic from AS#2046 because it’s a rival company, but the protocol doesn’t need to know that.
- Our least-cost routing protocols so far have no way of supporting these policies. Least-cost was a global minimization problem, where every router was trying to solve the same problem.
- By contrast, in policy-based routing, each AS only cares about its own policy, and there isn’t a global problem that everybody is cooperating to solve.
#### Gao-Rexford Rules for Routing Policies
- Although our routing protocol allows each AS to set any arbitrary policy they like, in practice, most ASes set their policies according to some standard conventions, known as the **Gao-Rexford rules**. These conventions are based in the assumption that real-world organizations like making money, and dislike losing money.
- There are two broad rules that ASes typically follow. First, when an AS has a choice of multiple routes, the AS prefers to forward packets to the **most profitable next hop**. Specifically, the AS prefers a route with a next hop that is a customer.
- If there are no such routes, the AS prefers a route with a next hop that is a peer. The AS will only select a route with a next hop that is a provider if it’s forced to do so, because there are no better routes.
	![mostProfitableHop|600](https://textbook.cs168.io/assets/routing/2-140-gaorexford1.png)
- Second, ASes only carry traffic **if they’re getting paid for it**. There’s no incentive for ASes to perform free labor. This principle dictates the paths that the AS is willing to participate in.
- You can think of this principle as a more restrictive version of announcing paths in the distance-vector protocol. Instead of advertising a route to every neighbor, allowing anybody to forward packets through me, I only advertise routes in which I’m paid to forward packets.
- A consequence of this second principle is: As an AS, the traffic I carry **should come from a customer, or go to a customer**. In other words, for any route going through me, one of my neighbors must be a customer. Let’s go through all the specific cases.
- Routes where both of my neighbors are customers are good, because I am getting paid by the two customers to forward packets.
	![customerNeighbors](https://textbook.cs168.io/assets/routing/2-142-gaorexford3.png)
- Similarly, routes where one of my neighbors is a customer, and one of my neighbors is a peer are good, because even though the peer doesn’t pay me, the customer does.
	![onePeerOneCustomer](https://textbook.cs168.io/assets/routing/2-143-gaorexford4.png)
- Routes where one of my neighbors is a customer, and the other is a provider are good. At first, it might seem like this path is bad, because the customer is paying me, and then I’m paying the provider. Isn’t it possible that I make no money, or lose money from this transaction?
- That may be true, but if we didn’t participate in these routes, we would be a useless AS with no customers. An AS’s job is to give connectivity to its users, and participating in these customer-AS-provider routes unlocks more routes to the rest of the Internet.
	![oneCustomerOneProvider](https://textbook.cs168.io/assets/routing/2-144-gaorexford5.png)
- Routes where both of my neighbors are peers are bad, because neither side is paying me to forward packets. More generally, peers do not provide transit between other peers. Thinking in terms of the hierarchy structure, a path should not stay at a given level for multiple hops.
	![twoPeers](https://textbook.cs168.io/assets/routing/2-145-gaorexford6.png)
- Routes where one of my neighbors is a peer, and the other is a provider are also bad, because again, neither side is paying me to forward packets.
	![oneProviderOnePeer](https://textbook.cs168.io/assets/routing/2-146-gaorexford7.png)
- Similarly, routes where both of my neighbors are providers are bad, because I have to pay both sides to forward the packet, and nobody is paying me to do this.
	![twoProviders](https://textbook.cs168.io/assets/routing/2-147-gaorexford8.png)
#### Routes are Valley-Free
- Paths in the AS graph are always **valley-free**. Thinking in terms of the hierarchy structure, if a path includes a lateral hop via a peering link, the immediate next hop needs to go downhill to a customer. The next hop cannot be lateral again (both neighbors peers), and the next hop cannot be uphill to a provider (peer and provider neighbors).
- Thinking in terms of the hierarchy structure, if a path includes a downward hop from provider to customer, the immediate next hop must continue to go downhill to one of its customers. The next hop cannot be lateral again (neighbors are provider and peer), and the next hop cannot be uphill (neighbors are both providers).
- If a downhill link must be followed by another downhill link, then we can conclude that as soon as you have a downhill link in a path, all subsequent links must also be downhill.
- A valley is a path that goes downhill, and then turns around to start going uphill. Paths cannot contain valleys, because once you start going downhill, you must continue downhill all the way to the destination.
	![singlePeek](https://textbook.cs168.io/assets/routing/2-152-singlepeak.png)
- Paths cannot have lateral moves anywhere except the peak. As soon as you make a lateral move, you must turn around and go back down. You cannot continue traveling laterally or uphill.
#### ASes Want Autonomy and Privacy
- When designing a protocol for computing inter-domain routes, our protocol should respect the autonomy and privacy of each AS.
- ASes want **autonomy**, the freedom to choose their own arbitrary policies, without coordinating with other ASes, or worrying about what policies the protocol allows.
- In practice, the policies usually follow the money-based principles we described, but the protocol shouldn’t force the AS to follow any specific policy.
- ASes also want **privacy**. ASes don’t want to have to explicitly tell others in the network about their preferences and policies.
- For example, an AS shouldn’t need to explicitly tell everybody about whether its neighbors are peers, customers, or providers. This reflects real-world business strategies. As a company, you might not want to reveal information about your customers and providers to your rivals.
- Note that our definition of privacy says that ASes shouldn’t need to _explicitly_ reveal their policies.
- In practice, ASes still need to coordinate with the rest of the network to agree on paths through the network, so some amount of information leakage is inevitable. Reverse-engineering techniques exist to trace the routes packets are taking through the network.
- For example, it’s unavoidable that others on the network can discover what route a packet is taking. However, our protocol shouldn’t force an AS to tell the world “I liked this path more than this other path.” We also shouldn’t force an AS to disclose who their providers, peers, and customers are.
# Sources
- [Lecture 9 - Routing 5: BGP I](https://www.youtube.com/watch?v=MAbRnl5tqTo).
- [Model for Inter-Domain Routing](https://textbook.cs168.io/routing/autonomous-systems.html).