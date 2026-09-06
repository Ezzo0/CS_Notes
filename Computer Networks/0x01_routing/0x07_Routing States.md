# Explanation
- So far, we’ve defined the routing problem as this: When a router receives a packet, how does the router know where to forward the packet such that it will eventually arrive at the final destination?
- Once we find an algorithm (a routing protocol) to solve this problem, we can apply that algorithm to generate an answer, which we’ll call a **routing state**.
- You can think of a routing state as a _set of rules_ that each **router uses to forward packets** it receives. What does a routing state look like, and how can we check if a given routing state is valid or good?
- To start, we could consider some bad strategies for generating routing states. One possible routing strategy is: The router forwards the [[0x00_Layers of the internet#Layer 3 Packets Abstraction|packet]] to a randomly-selected neighbor.
- Intuitively, we can already see that routing states generated this way probably won’t be valid. If we use this strategy, we can’t be sure that packets will reach their final destination.
- Another possible bad strategy is: The router forwards a copy of the packet to every single one of its neighbors.
- Intuitively, this might be valid, in the sense that copies of the packet will eventually spread across the entire network and probably reach the destination. However, this strategy is inefficient, because it **wastes a lot of bandwidth** forwarding the packet to routers that were not needed to send the packet to its final destination.
#### Forwarding Tables
- In our model of the network, each router has some number of outgoing links connecting it to adjacent routers and hosts.
- When the router receives a packet, with its final destination in the metadata, the router needs to decide which of the adjacent routers or hosts the packet should be forwarded to.
- The next intermediate router that the packet will be forwarded to is called the **next hop**.
- For example, consider this network. If R2 receives a packet whose final destination is B, the natural corresponding next hop would be R3.
- The possible choices of next hop are R1, R3, and R4 (the three routers adjacent to R2), and R3 is the next hop that sends the packet closer to B.
- If R2 instead receives a packet whose final destination is A, then the natural corresponding next hop would be R1 instead.
- For each possible final destination, we can write down the corresponding next hop to forward the packet closer to that destination. The result is called a **forwarding table**.
	![forwardingTable|600](https://textbook.cs168.io/assets/routing/2-015-forwarding-table.png)
- By writing down the forwarding table for each intermediate router, we now have a full routing state for the network. In other words, given a [[0x00_Layers of the internet#Layer 3 Packets Abstraction|packet]] with some final destination, we know exactly how each router will forward that packet.
- In the physical world, instead of mapping destinations to next hops, routers will often map destinations to **physical ports**, where each physical port corresponds to a link. In the graph model, we would now be mapping each destination to an edge, instead of mapping each destination to a neighboring node.
	![portMapping|400](https://textbook.cs168.io/assets/routing/2-016-ports.png)
- This is a subtle distinction, and it reflects the fact that the router doesn’t really care about the identity of the neighboring router. The only decision the router needs to make is to send the packet along one of the wires, regardless of who the wire is connected to.
- In these notes, we’ll draw forwarding tables as mapping destinations to next hops (instead of physical ports), for simplicity.
#### Destination-Based Forwarding
- A consequence of using a forwarding table is that given a packet, the decision of where to forward the packet depends only on the destination field of the packet.
- In other words, if a router receives many different packets, all with the same destination, they will all be routed to the same next hop (assuming the forwarding table stays _unchanged_).
- Since each destination is only mapped to a single next hop, there’s no way for two packets with the same destination to be forwarded to different routers. This approach is called **destination-based forwarding** or **destination-based routing**.
#### Routing vs. Forwarding
- Now that we’ve introduced the idea of a forwarding table, we need to make a distinction between the process of creating the forwarding table, and the process of using the forwarding table.
- **Routing** is the process of routers communicating with each other to determine how to populate their forwarding tables.
- **Forwarding** is the process of receiving a packet, looking up its appropriate next hop in the table, and sending the packet to the appropriate neighbor.
#### Routing State Validity is Global
- Recall that a routing state consists of a forwarding table for each router, which collectively tells us how packets will travel through the network. Given a routing state, how can we tell if the routing state is correct or incorrect?
- First, we need to formally define **routing state validity** to determine whether a routing state is valid (though this term may not be widely used outside CS 168 at UC Berkeley). The main requirement for validity is: the routing state needs to produce forwarding decisions that ensure that packets actually reach their destination.
- Note that validity must be evaluated in the global context, not a local context. Looking at local routing state, such as a single router’s forwarding table, cannot tell us whether a routing state is valid.
- For example, in a router R2’s local forwarding table, we might see that the next hop for destination A is router R3, but we have no way to decide if this is valid. Will forwarding packets to R3 help packets reach destination A? There’s no way to tell from just the forwarding table.
- Instead, we need to consider the global routing state, which consists of the collection of all the forwarding tables in all of the routers.
	![globalValidity](https://textbook.cs168.io/assets/routing/2-019-validity-global.png)
#### Routing State Validity Definition
- A global routing state is valid if and only if, for any destination, packets do not get stuck in dead ends or loops.
- A **dead end** occurs if a packet arrives at a router, but the router doesn’t know how to forward the packet to its destination, so the packet is not forwarded. This might occur if the router’s forwarding table doesn’t contain an entry for the packet’s destination.
- A **loop** occurs if a packet is sent in a cycle around the same of nodes. Note that because we’re using destination-based forwarding, where the next hop only depends on the destination, once a packet enters a loop, it will be trapped in the loop forever.
- When the packet arrives at the router the first time, or the 10th time, or the 500th time, it will be forwarded the exact same way (since the final destination is the same). Since this applies to every router on the loop, the packet will be stuck in the loop forever.
	![loop](https://textbook.cs168.io/assets/routing/2-021-loop.png)
#### Directed Delivery Trees
- Now that we have a formal definition for routing state validity, we can ask: given a global routing state, how can we check if it’s valid?
- To simplify the problem, let’s start by considering only a single destination end host, ignoring all other end hosts. In each router, we can look up this destination to get the corresponding next hop, which tells us how each router will forward packets meant for this destination.
- We can represent the next hop at each router (for this single destination) as an arrow, which shows us all the possible paths that this packet might take to reach the single destination.
	![deliveryTree](https://textbook.cs168.io/assets/routing/2-022-delivery-tree.png)
- In the resulting graph, each node will only have one outgoing arrow. This reflects our assumption that in each router’s forwarding table, there is only one next hop corresponding to a destination.
- Notice that in the resulting graph, once two paths meet, they never split. In other words, even if there are multiple incoming arrows (paths) to a node, since there is only one outgoing arrow, those paths will now converge into a single path.
- This reflects our destination-based forwarding approach, because each router only uses the final destination to decide how to forward a packet. The router does not care how the packet arrived at the router in the first place.
	![noDiverging](https://textbook.cs168.io/assets/routing/2-023-no-diverging.png)
- The arrows we’ve drawn form a set of paths that a packet can take to reach the single destination. This set of paths is called a **directed delivery tree**.
- In graph terms, the arrows in a valid delivery tree must form an **oriented spanning tree**, rooted at the destination.
- Recall that a [[Minimum Spanning Trees#^58b2b0|spanning tree]] is a set of edges in the graph that touch every node and form a tree. We want the delivery tree to be a tree, since there should be no cycles (packets can’t travel in loops).
- We want the delivery tree to be spanning (touch every node), because we want to be able to reach the destination from everywhere. The delivery tree is oriented because the edges have arrows, which tells us which direction to forward the packet.
- All edges in a valid delivery tree should point toward the destination. In other words, starting from any node, following the arrows should always result in reaching the destination.
#### Least-Cost Routing
- Now that we have a definition of what makes a routing state valid (routes have no loops and dead ends), we can additionally define what makes a routing state good.
- It’s possible that a network has multiple valid routing states, and we want some metric that can help us determine whether one route is better than another.
- **Least-cost routing** is a common approach for measuring whether a route is good. In least-cost routing, we assign a numeric cost to every link, and look for routes that minimize the cost.
- In other words, we want routes that result in packets traveling along the lowest-cost paths to their destinations.
	![cost|400](https://textbook.cs168.io/assets/routing/2-029-costs.png)
- There are many different costs we could consider assigning to links. The cost could depend on the price of building the link, the propagation delay, the physical distance of the link, the unreliability, the bandwidth, among other factors.
- For example, we could assign costs based on the quality of the link (bandwidth and propagation delay), such that the lowest-cost path prefers higher-quality links.
- If we assign a cost of 1 to every link, then the least-cost path is the path that travels along the fewest links. We sometimes call this minimizing the **hop count**. 
- The operator of a network can decide how to assign costs to each link. The operator might manually assign costs.
- Or, the operator could have the network automatically configure the costs, although this may not work with some metrics that can’t be automatically measured (e.g. the network has no idea about the financial cost to build the link).
#### Static Routing
- One possible way to generate routes is to have the network operator manually populate the forwarding table. This is known as **static routing**.
- Static routing by itself isn’t practical (e.g. not scalable, prone to human error), but even with a routing protocol implemented, some routes still need to be manually created by operators.
- You can think of these manual routes as the “trivial” or “base case” routes, from which the routing protocol generates more complex routes.
- If we’re directly connected to another machine that we want to route packets to, we can manually configure a route to forward packets to that other machine. These routes are called **direct routes** or **connected routes**.
- For example, your home router is connected to your personal computer with a link, so your home router can add an entry in the forwarding table corresponding to your computer. This entry is added by telling the router about the connection, and is not added from running any routing protocol.
	![staticRoutes](https://textbook.cs168.io/assets/routing/2-031-static.png)
- It is also possible to use static routing to hard-code entries for destinations in the forwarding table, even if we aren’t directly connected to that destination.
- This can be useful if there’s a route that never changes, and we want that route to always stay in our forwarding table, regardless of what the routing protocol is doing.
# Sources
- [Lecture 4 - Routing 1: Principles](https://www.youtube.com/watch?v=aEkZKsOWvzE).
- [Routing States](https://textbook.cs168.io/routing/solutions.html).