# Explanation
- So far, our model of [[0x12_Model for Inter-Domain Routing|inter-domain routing]] has treated an entire AS as a single entity, importing and exporting paths. However, in reality, the AS contains many [[0x06_Model for Intra-Domain Routing#Routers and Hosts|routers]] (and hosts) connected by [[0x04_Links|links]].
- In order to actually implement [[0x13_Border Gateway Protocol (BGP)|BGP]], we need all the routers inside the AS to work cooperatively to act as a single node.
- Within an AS, we will classify all routers into two types. **Border routers** have at least one link to a router in a different AS. **Interior routers** only have links to other routers within the same AS.
	![borders|500](https://textbook.cs168.io/assets/routing/2-167-borders.png)
- Only the border routers need to advertise routes to other ASes. Sometimes, we call the routers advertising BGP routes **BGP speakers**.
- The BGP speakers need to understand the semantics and syntax of the BGP protocol (how to read and create a BGP announcement, what to do when receiving an announcement, and so on).
#### External and Internal BGP Sessions
- A **BGP session** consists of two routers exchanging information between each other.
- An **external BGP (eBGP) session** is between two routers from different ASes. eBGP sessions can be used to exchange announcements between different ASes and learn about routes to other ASes. Only border routers participate in eBGP sessions (since eBGP requires talking to a different AS).
- By contrast, an **internal BGP (iBGP) session** is between two routers in the same AS (**_not necessarily directly connected by a link_**). More specifically, if a border router learns about a new route, it can use iBGP to distribute that new route to the other routers in the AS.
- This allows all the routers in the AS to coordinate and act together as one entity. Both border and internal routers participate in iBGP sessions.
- eBGP and iBGP sessions are different from **interior gateway protocols (IGP)**. These are the [[0x06_Model for Intra-Domain Routing|intra-domain routing]] protocols (e.g. [[0x08_Distance-Vector Protocols|distance-vector]], [[0x09_Link-State Protocols|link-state]]) that are deployed within an AS to route packets inside the AS.
	![bgp](https://textbook.cs168.io/assets/routing/2-171-bgp4.png)
- eBGP, iBGP, and IGP work together to establish routes from any one router to any other router in the Internet (even if the routers are in different ASes).
	- First, each AS runs IGP to learn [[0x07_Routing States#Least-Cost Routing|least-cost]] paths between any two routers inside the same AS.
	- Next, the ASes run eBGP, advertising routes to each other to learn about routes to other ASes.
	- Finally, the ASes run iBGP, so that a router that has learned about an external route can distribute that route to all the other routers in the same AS.
	- Using the iBGP results, we can figure out which border router is on that external route. Then, we can use IGP to forward the packet to the correct border router (who will then forward the packet to the next AS).
	![bgpInOrder](https://textbook.cs168.io/assets/routing/2-176-bgp9.png)
- The border router who advertises a route to an external destination is sometimes called the **egress router** for that destination.
- This is the router who can help your packet exit the local network and move to other networks closer to the destination. In the example above, G is the egress router for destination Z.
- A consequence of these protocols is that every router has **two forwarding tables**.
	- One is a table mapping all **_internal destinations_** (same AS) to a next hop, populated with information from IGP.
	- The other is a table mapping all **_external destinations_** to an egress router (who knows a route to the external destination), populated with information from eBGP.
	- Note that in the eBGP table, the egress router is not necessarily a next hop. The egress router might be several local hops away, but we use IGP to reach that egress router.
	![egressRouter](https://textbook.cs168.io/assets/routing/2-178-bgp11.png)
- We’ve seen how eBGP ([[0x13_Border Gateway Protocol (BGP)#Modification Path-Vector Protocol|path-vector]], advertising routes) and IGP (distance-vector or link-state) are implemented as algorithms. How is iBGP implemented?
- When a border router installs a new route to a destination, it has to inform the other routers in the AS. One simple solution is to have the border router **directly** tell every other router in the AS.
- This solution is relatively simple, though it requires every border router to have an iBGP session with every other router. In a network with B border routers and N routers total, this protocol would require $B.N$ iBGP connections, and might **_scale poorly_** as local networks get larger.
#### Multiple Links Between ASes: Hot Potato Routing
- In practice, it can be useful to have multiple links between large ASes. For example, Verizon and AT&T are very large ASes with infrastructure across the entire United States.
- Suppose there was only one link between the two ASes on the west coast. If a Verizon router in the east coast and an AT&T router in the east coast wanted to communicate, the packet would have to travel across the country on Verizon’s network, traverse the link into AT&T’s network, and then travel back across the country to the destination.
- Multiple links between two ASes also means that there can be multiple paths between two routers that pass through the same ASes. But, if there are two routes, which route does the importing AS prefer?
	![multiLink|500](https://textbook.cs168.io/assets/routing/2-181-multilink2.png)
- Bandwidth costs money, so I would prefer if this traffic traveled as far as possible on infrastructure owned and paid for by other people, and traveled as little as possible on my own infrastructure. Therefore, the orange path is preferred.
	![multiLinkChoice|500](https://textbook.cs168.io/assets/routing/2-182-multilink3.png)
- More formally, the importing AS receives two announcements: one from the west router, and one from the east router.
- Using iBGP, every router inside the AS sees both announcements. One says, the egress router is the west router, and the other says, the egress router is the east router. Every router has to decide which announcement to import.
- Let’s focus on router E. Using IGP, this router can figure out the distance to the west egress router (F), and the distance to the east egress router (I).
- Since the west egress router (F) is closer, routing packets via the west egress router (F) will use up less of this AS’s bandwidth.
- Therefore, this router will import the path via the west egress router (F). Another router, like one closer to the east egress router (I), might decide to import a different path.
	![multiLinkCalc|550](https://textbook.cs168.io/assets/routing/2-185-multilink6.png)
- This strategy of selecting the nearest egress router is sometimes called **hot potato routing**. We want the packet to leave our AS as soon as possible, and start traveling over somebody else’s links as soon as possible.
#### Multiple Links Between Routers: MED
- What if a router is equally close to both possible egress routers? In order to tiebreak, the exporting AS can announce a preference for one route over the other.
- Which route does the exporting AS prefer? Again, since bandwidth costs money, the exporting AS prefers the pink path, which uses less of its bandwidth.
- In the announcement of the pink path, the exporting AS can additionally say “I prefer if you used this path,” and in the announcement of the orange path, the exporting AS can additionally say “I prefer if you avoided this path.”
	![med|500](https://textbook.cs168.io/assets/routing/2-186-med1.png)
- Now, the router that is equally close to both egress routers can see this extra information in the iBGP announcement.
	![med2](https://textbook.cs168.io/assets/routing/2-187-med2.png)
- Using this extra information, the router can select the egress router on the pink path, since the exporting AS preferred this path.
	![med3](https://textbook.cs168.io/assets/routing/2-188-med3.png)
	![med4](https://textbook.cs168.io/assets/routing/2-189-med4.png)
- This additional information in the exporting announcement is called the **Multi-Exit Discriminator (MED)**. From the perspective of the exporter, it indicates my preferred router for entering my network. From the perspective of the importer, it indicates the other AS’s preferred router for exiting my network and entering the other AS’s network.
- Another way to interpret the MED is, the **distance to the destination**, via this router. The exporter can say, “the west coast router is 3 hops away from the destination,” and “the east coast router is 12 hops away from the destination.”
- Lower MED numbers are preferred, since the exporter wants to use as little of its own bandwidth as possible. The exporter would rather use 3 of its own links, instead of 12 of its own links.
#### Import Policy Priority
- Our more detailed model, where two ASes can be connected with multiple links, means that we now have additional import policy rules, in addition to the [[0x12_Model for Inter-Domain Routing#Gao-Rexford Rules for Routing Policies|Gao-Rexford rules]].
- When you receive multiple announcements for the same destination, select a path based on these tiebreaking rules, in this order:
	1. Use the **Gao-Rexford rules**. Select the path advertised by a customer, over the path advertised by a peer, over the path advertised by a provider.
	2. If multiple paths have the same Gao-Rexford priority (e.g. two paths from customers), select the **shorter path** (the path passing through fewer ASes).
	3. If multiple paths have the same length, select the path with the **closer egress router** (using IGP to find distance to each egress router).
	4. If multiple paths have the same distance to egress router, select the path with the **lower MED** (where MED is included in the advertisement).
	5. If multiple paths have the same MED, **tiebreak arbitrarily** (e.g. pick the router with the lower IP address).
- Notice that closest egress router (hot potato routing) and MED are often contradictory. Every AS prefers to minimize their own bandwidth usage, and wants the packet to be carried on other ASes’ bandwidth.
	![med6](https://textbook.cs168.io/assets/routing/2-191-med6.png)
- One consequence of this contradiction is that paths through the Internet are often **asymmetric**. If two hosts are sending packets back and forth, the path in one direction might be different from the path in the other direction.
- In practice, sometimes ASes will try and implement more clever strategies to trick other ASes into carrying the packet further. Or, an AS with better bandwidth might agree to carry your traffic further for you, if you pay a premium fee.
- Fundamentally, BGP allows this behavior because every AS is granted the autonomy to set their own [[0x12_Model for Inter-Domain Routing#Policy-Based Routing|policy]] (here, that policy is hot potato routing).
#### BGP Message Types and Route Attributes
- There are four different BGP message types.
	- **Open messages** can be used to start a session between two routers to communicate with each other.
	- **KeepAlive messages** can be used to confirm that a session is still open, even if messages haven’t been sent recently.
	- **Notification messages** can be used to process errors. 
	- **Update message**. can be used to announce new routes, change existing routes, or delete routes that are no longer active. We won’t describe these first three message types in any further detail. We’ll focus on the fourth.
- The Update message contains a **destination**, represented as an IP prefix. The message also contains **route attributes**, which can be used to encode any useful information corresponding to that IP prefix.
- The route attributes are a set of **name-value pairs**, where the name indicates the type of attribute, and the value indicates the value of that attribute.
- Some attributes are local to an AS, and are only exchanged in iBGP messages. Other attributes are global, and can be sent in eBGP advertisements.
- There are many BGP attributes, but we’ll focus on three important ones, which are used to encode the different tiebreakers for importing paths.
- The **LOCAL PREFERENCE** attribute encodes the Gao-Rexford import rules (top priority tiebreaker) inside a specific AS. An AS can assign a higher value to more preferred routes (e.g. from customers), and a lower value to less preferred routes (e.g. from providers).
- This attribute is local, and only carried in iBGP messages. This attribute is not sent to other ASes in eBGP announcements, because other ASes don’t need to know about this AS’s preferences.
	![localPreferenceAttr](https://textbook.cs168.io/assets/routing/2-194-attribute1.png)
- The local preference numbers are arbitrary, and only their relative ranking is important. In the example above, the numbers could have been 300 and 100 instead of 3000 and 1000, and the behavior would be the same. The local preference numbers are often set manually by operators.
- The **ASPATH** attribute contains a list of ASes along the route being advertised (in reverse order). This attribute is global, and can be sent in eBGP announcements.
	![ASPathAttr](https://textbook.cs168.io/assets/routing/2-195-attribute2.png)
- The `ASPATH` is the second priority tiebreaker when importing paths. If two announcements have the same local preference (e.g. both are from customers), then we’ll select the shorter path. `ASPATH` tells us the length of each path, measured by the number of ASes the path goes through.
- The **MED** attribute encodes the preferences of the exporting AS. This attribute represents the distance from the exporting router to the destination (lower numbers are preferred).
	![MEDAtrr](https://textbook.cs168.io/assets/routing/2-196-attribute3.png)
#### Issues with BGP
- BGP has no built-in security guarantees. A malicious AS could lie and advertise a route to a destination, even if the AS cannot reach that destination.
- A malicious AS could also advertise a very cheap route to a destination, even if that cheap route doesn’t actually exist. This could encourage other ASes to route packets through the malicious AS, where the attacker could delete or modify packets passing through the malicious AS.
- These attacks are called **prefix hijacking**. There is active research on using cryptography to secure BGP, though such protocols are not widely deployed.
- BGP prioritizes policy over least-cost when selecting paths. Also, because BGP measures path length in terms of the number of ASes, the path length can be misleading (e.g. one AS could contain 2 routers or 200 routers along the path being advertised).
- This can lead to issues where packets don’t always take least-cost paths, and it’s difficult to reason about performance on the Internet. Some might classify these as issues, though they may be more of an intentional design trade-off.
- The designers of BGP made a conscious design choice to prioritize policy and hide the internal topology of an AS, at the expense of performance.
- BGP requires certain assumptions (everybody is following the Gao-Rexford rules, AS graph forms a hierarchy, no provider-customer cycles) in order to guarantee reachability and convergence.
- If these assumptions don’t hold (e.g. an AS chooses its own policy that violates Gao-Rexford), BGP can produce unstable behavior, where routes never converge, or cycles and dead-ends appear.
# Sources
- [Lecture 10 - Routing 6: BGP II](https://www.youtube.com/playlist?list=PL0_XloRC3MWtXZPKRiEhVJmR54yme-mru).
- [BGP Implementation and Issues](https://textbook.cs168.io/routing/bgp-implementation.html).