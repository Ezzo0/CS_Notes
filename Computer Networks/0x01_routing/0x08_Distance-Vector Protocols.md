# Explanation
- The prototypical distance-vector protocol is the **Routing Information Protocol (RIP)**, and the D-V protocol we’ll design shares many similarities with RIP.
- To gain some intuition for the routing protocol we’ll study in this section, consider the following network.
- To start out, every router’s forwarding table is empty. Our goal is to fill in the forwarding tables of every router, such that [[0x00_Layers of the internet#Layer 3 Packets Abstraction|packets]] can be routed from anywhere to the destination, A.
- To start, A can tell R1: “I am A.” Now, R1 knows how to forward packets to A. Now that R1 has a path to A, it can tell its neighbors, R2 and R3: “I am R1, and I can reach A.”
	![sketch|500](https://textbook.cs168.io/assets/routing/2-033-sketch2.png)
- Now, R2 and R3 know that they can reach A by forwarding packets to R1. R2 can now tell its neighbors, R4 and R5: “I am R2, and I can reach A.” Similarly, R3 can tell its neighbors, R6 and R7.
- Now, R4 and R5 know that packets for A can be forwarded to R2, and R6 and R7 know that packets for A can be forwarded to R3.
- The process continues: R4, R5, R6, and R7 each tell their neighbors who they are, and that they can reach A. By the end, everybody’s forwarding table is filled in, and we can route packets from anywhere in the network towards A.
	![sketch2|500](https://textbook.cs168.io/assets/routing/2-035-sketch4.png)
- In summary:
	- When you receive an announcement from someone saying they can reach A, you should write down who sent the announcement. Now, you can send messages bound for A through that person.
	- Also, now that you have a way to send messages to A, you should make an announcement to all of your neighbors, so that they can send messages bound for A through you.
- What if there were multiple destinations? We could run this same algorithm repeatedly, once per destination. The forwarding table would then contain multiple entries, one per destination.
#### Rule 1: Bellman-Ford Updates
- What if there are multiple paths to reach A? In this scenario, both R3 and R4 will announce that they can reach A. Should R5 choose to forward packets to R3 or R4?
	![multiPath|400](https://textbook.cs168.io/assets/routing/2-037-multipath1.png)
- Recall that our goal is to find [[0x07_Routing States#Least-Cost Routing|least-cost]] routes through the network. To allow routers to pick the least-cost path out of multiple being advertised, we’ll need to also include costs in the announcements.
	- R3’s announcement now says: “I am R3, and I can reach A with cost 3.”
	- R4’s announcement now says: “I am R4, and I can reach A with cost 2.”
- Now, R5 notices that R4 is offering the shorter path, and decides to forward packets via R4.
- We’ll use the forwarding table to remember the best-known cost to the destination (and the corresponding next-hop). Each entry of the forwarding table now tells us: **the destination, the next-hop for that destination, and the cost** to reach the destination via that next hop.
- Note: Formally, the forwarding table **stores key-value pairs, mapping each destination to a 2-tuple containing the next hop and the distance**. We’ll draw tables with 3 columns for simplicity.
- R5 might not hear about both paths simultaneously, so we’ll need to be more precise about what happens when we hear about a new path. There are three possibilities when we hear about a path:
	1. If the table doesn’t have a path to the destination, accept the path. If I don’t have a way to reach A, I should accept any path offered.
		![r5Table](https://textbook.cs168.io/assets/routing/2-039-multipath3.png)
	2. If the new path (that we hear about) is better than the best-known path (from the forwarding table), we should accept the new path, and replace the old path from the table.
		![newBetterPath](https://textbook.cs168.io/assets/routing/2-040-multipath4.png)
	3. 1. If the new path (that we hear about) is worse than the best-known path (from the forwarding table), we should ignore the new path, and keep using the path in the table.
- How do we know if a new path is better or worse? We have to be careful, because not all link costs are the same. When someone advertises a path, the cost via that path is actually the sum of two numbers: **The link cost from you to the neighbor, plus the cost from the neighbor to the destination (as advertised by the neighbor)**.
- As a concrete example, suppose we hear: “I am R1, and A is 5 away from me.” The cost of this new path is actually 1 (the link cost from us to R1), plus 5 (the cost from R1 to A, from the advertisement), which is 6.
	![cost1|500](https://textbook.cs168.io/assets/routing/2-041-costs1.png)
- Later, we might hear: “I am R2, and A is 3 away from me.” It is incorrect to just look at the cost in the advertisement.
- In this case, the cost of the new path is actually 10 (the link cost from us to R2), plus 3 (the cost from R2 to A, from the advertisement), which is 13. This cost is not better than our best-known cost of 6, so we don’t update the table. Packets still get forwarded to R1.
	![cost2|500](https://textbook.cs168.io/assets/routing/2-042-costs2.png)
#### Rule 1: Distributed Bellman-Ford Algorithm
- Does this operation look familiar? It turns out, this is exactly the relaxation operation from Dijkstra’s shortest paths algorithm!
- **Bellman-Ford** is another shortest paths algorithm that relies on relaxation as the key operation. Bellman-Ford is even simpler than Dijkstra’s: Cycle through all the edges repeatedly, relaxing every edge, until we get all the shortest paths.
- Unfortunately, the common implementation code wouldn’t be very useful for our routing protocol.
	![bellmanFord](https://textbook.cs168.io/assets/routing/2-043-bellman-ford.png)
- Remember, the routing protocol must be **distributed**, because routers don’t have a global view of the network (no central mastermind). Also, the routers are **operating asynchronously**. There’s nobody enforcing the order in which routers perform relaxation operations, or the order in which routers send out announcements.
- Instead, the routing protocol that we’ve been designing is a distributed, asynchronous version of the Bellman-Ford algorithm.
- The protocol is distributed, because we aren’t asking a single computer to run the entire algorithm. Instead, every router is computing its own part of the answer (populating its own forwarding table) without seeing the entire graph.
- The protocol is asynchronous, because the routers can all run the algorithm at the same time, without needing to control the order of operations.
#### Rule 2: Updates From Next-Hop
- Recall one of our routing challenges is that the network topology can change. Suppose that we hear an advertisement from R2, saying that A is 3 away from R2. If there’s nothing in our table, we’ll accept this advertisement and record a cost of 1+3=4..
- Later, we might hear a different advertisement from R2, saying that A is 8 away from R2. From the previous rule, we would reject this, because the advertised cost (1+8=9) is worse than our current cost (4).
	![newAdvertisement](https://textbook.cs168.io/assets/routing/2-053-change2.png)
- However, we have to be careful about rejecting this advertisement. The router making the announcement (R2), was the same as the next hop router we were using.
- R2 is trying to say: “If you’re using me as a next hop, my distance to A is no longer 3, it’s 8.” But we ignored this message because we weren’t thinking about the possibility that paths might change.
- To fix this, we have to modify our update rule. If we hear an announcement from the next-hop router (the router with the best-known path that we were forwarding packets to), we should treat that announcement as an update, and edit our forwarding table.
- We should do this even if the announcement produces a worse path, because the next hop could be telling us that the path cost has changed and gotten worse.
	![change](https://textbook.cs168.io/assets/routing/2-054-change3.png)
- In order to support changing topologies, routers will run the routing protocol indefinitely. Suppose we ran the protocol indefinitely, with no topology changes. Initially, some relaxations will succeed and the forwarding tables will change.
- Eventually, the algorithm will **converge** when we have found all the least-cost routes through the network. At this point, if we continue relaxing the edges, the forwarding tables will not change.
- Every relaxation will be rejected, because the best-known paths to the goal are all the shortest paths, and we’ll never find a better path to replace the current shortest paths. The state of the network at convergence is called **steady state**.
- Later, suppose we change the topology (e.g. maybe a router fails). As we continue running the protocol, some relaxations might succeed again, since we’ve changed the underlying graph.
- After some time, the delivery tree will converge again on the new least-cost routes and stop changing until the next time the topology changes.
#### Rule 3: Resending
- Recall another one of our routing challenges is that [[0x00_Layers of the internet#Layer 3 Packets Abstraction|packets]] can get dropped.
- If R2 and R3 have empty forwarding tables, and R1 is updated with the [[0x07_Routing States#Static Routing|hard-coded route]] to A. Now What will happen if R1 issues an announcement, but the packet is dropped? R2 never hears an announcement, and the protocol fails.
	![droppedPacket](https://textbook.cs168.io/assets/routing/2-055-dropped.png)
- You could try to design a more complicated scheme to ensure reliability (e.g. forcing recipients to send acknowledgements), but let’s use something simple: If you have an announcement to make, re-send that announcement every few seconds. It turns out this simple approach works well with some of our later design choices, and nothing more complicated is necessary.
- Formally, the protocol will define an **advertisement interval**. 30 seconds is a common interval used in practice. If the interval is X seconds, then every advertisement must be re-sent every X seconds.
- As long as we wait long enough and re-send the packet enough times, the link will eventually successfully send the advertisement, as long as the link works some of the time.
- If the link was dropping every single packet, then there’s no way for the advertisement to be sent (and maybe a link with 0% success rate probably shouldn’t be in the graph anyway). Eventually, with enough re-sending, this protocol will still converge.
- Note that re-sending at intervals can work in combination with our rule from earlier, where we sent an announcement any time the forwarding table changes. Announcements sent immediately after a change are called **triggered updates**.
- The protocol would still converge if we only sent announcements at intervals. The table changes, we wait for the interval to expire, and send out the announcement.
- However, adding triggered updates in addition to interval updates is an optimization that can help the protocol converge quicker. As soon as we know the update, we might as well announce it, without waiting for the interval.
- With this new rule, once the network converges, every router will continue to re-send announcements periodically, but none of the announcements will be accepted, because we’re in steady state and everybody already has the shortest-cost routes.
- In the example from earlier, after the network converges, R3 might decide to re-send its announcement, with destination A, next hop R3, and cost via R3 of 3. But R2 will ignore this announcement because its forwarding table has a cheaper route of cost 2 already (the announcement path costs 3 + 1 = 4).
#### Rule 4: Expiring
- Recall our routing challenge from earlier: The network topology can change. In particular, links and routers can fail.
- If a router fails in the network, our route might become invalid. The failed router won’t tell us about the problem (since it’s failed), so we’re stuck with this invalid route.
- To solve this problem, we’ll give every route (i.e. every table entry) a finite **time to live (TTL)**. This is a countdown timer, telling us how much longer we can keep this forwarding entry.
- Periodic updates help us confirm that a route still exists. If we get an advertisement from the next-hop, we can reset (“recharge”) the TTL to its original value.
- If something in the network fails, we’ll stop getting periodic updates. Eventually, the TTL will expire. If the TTL expires, we’ll delete the entry from the table. Intuitively: We aren’t getting updates anymore, so this route is probably no longer valid.
- Here’s an example of the TTL in action. In this example, we are R3. At time t=0, we hear an announcement: “I’m R2, and A is 5 away from me.”
- Our table doesn’t have an entry for A, so we’ll accept this path, and set its TTL to 11. Notice that this TTL is associated with the specific table entry. If we had multiple table entries, they would each have their own TTL.
	![ttl1](https://textbook.cs168.io/assets/routing/2-056-ttl1.png)
- The TTL of 11 tells us that R2 must send us another confirmation of this route in the next 11 seconds. Otherwise, this table entry will be deleted. Time passes. At t=1, the TTL is now 10. At t=2, the TTL is now 9. At t=3, the TTL is now 8. At t=4, the TTL is now 7.
- At t=5, R2 does its periodic re-sending of the announcement: “I’m R2, and A is 5 away from me.” We look in our table and realize that R2 is the current next-hop to A, so we should **accept this advertisement (per Rule 2) and update the table**.
- Because we got a confirmation of this route still existing, the TTL can be reset back to its initial value of 11. We need to get another confirmation of this route from R2 in the next 11 seconds.
- Suppose that a link goes down at t=6, and A is now unreachable. R2 removes its static route to A, and no longer sends any periodic updates.
- At t=16 (11 seconds after the last update at t=5), the TTL in our table entry has decreased all the way to 0, so we’ll delete the entry from our table.
	![ttl2](https://textbook.cs168.io/assets/routing/2-059-ttl4.png)
- Be careful not to confuse the various timers that the router must maintain.
	- The advertisement interval tells the router when to advertise routes to neighbors. This is usually a single timer for the entire table, so the router advertises all the routes in the table whenever the advertisement interval timer expires.
	- In the example above, the advertisement interval timer was 5 seconds, since R2 sent advertisements at t=0 and t=5.
	- By contrast, the TTL tells the router when to delete a table entry. Each table entry has its own independent TTL, counting down for that specific entry.
	- In the example above, the initial TTL was 11 seconds (reset to 11 when we accept an advertisement), and counted down for each table entry.
#### Rule 5: Poisoning Expired Routes
- Waiting for routes to expire is slow. In this example, we are R3. Assume that by t=5, we’ve learned a route to A, via R2, and this route has 11 seconds of TTL remaining.
	![poison1](https://textbook.cs168.io/assets/routing/2-060-poison1.png)
- At t=6, the A-to-R2 link goes down! The table entry is now busted, because if we forwarded packets to R2, they wouldn’t actually reach A. However, we don’t know that this entry is busted yet. We have to wait another 10 seconds for this route to expire.
- Also at t=6, we get a new announcement: “I’m R1, and A is 1 away from me.” We look in our table, and we already have a way to reach A, so we reject this announcement. 
- If only we knew that our existing route is busted, we could accept this new advertisement right now. But instead, we’re doomed to wait another 10 seconds of using this busted path.
	![poison2](https://textbook.cs168.io/assets/routing/2-061-poison2.png)
- Time passes. By t=16 (five seconds later), the busted route TTL finally reaches 0, and we can delete this entry from the table.
- Also at t=16, R1 re-sends its announcement again: “I’m R1, and A is 1 away from me.” Finally, our table doesn’t have a route to A (the busted route just got deleted), so we can accept this announcement.
	![poison4](https://textbook.cs168.io/assets/routing/2-063-poison4.png)
- The key problem here is: When something fails, it’s not being reported, so we’re forced to rely on timeouts to delete busted paths. This is slow. Is there any way we can detect failures earlier? The solution is **poison**: When something fails, if possible, explicitly advertise that a path is busted.
- In English, the new poison announcement that R2 sends would say: “I’m R2, and I no longer have a way to reach A.” In the protocol, we encode this message by advertising a path with **cost infinity**: “I’m R2, and A is infinity away from me.” This infinite-cost path represents a busted path.
- Poisoned paths propagate just like any other path. If we’re forwarding packets to R2, and we get a poison message from R2, we update our forwarding table and replace the cost with infinity (per Rule 2).
- We can also advertise this infinite-cost poison to our neighbors, so they are also alerted of the busted path. This allows an invalid path to propagate through the network, which can be much faster than waiting for the path to time out.
	![poison5](https://textbook.cs168.io/assets/routing/2-064-poison5.png)
- Let’s formalize the rules of poison. Poison originates from one of two sources:
	- One of your routes times out, or
	- You notice a local failure (e.g. one of your links goes down).
- When one of these occurs, you can update the appropriate table entry with cost infinity, reset the TTL, and advertise this poison to your neighbors.
#### Rule 6A: Split Horizon
- Suppose we’re in steady state, and the forwarding tables have the correct shortest routes to A. Announcements are being periodically re-sent, but all announcements are being rejected because we’re in steady state.
- The R1-R2 link goes down, and R2’s entry expires, because R1 stopped sending periodic announcements. R2 now has an empty forwarding table. What happens next?
	![splitHorizon2](https://textbook.cs168.io/assets/routing/2-068-splithorizon2.png)
- Eventually, R3 re-sends its announcement to R2, with destination (A), next hop (R3), and cost via next hop (3).
- R2’s table is empty, so it accepts this announcement and adds destination (A), next hop (R3), and cost via next hop (3 + 1 = 4). We’ve created a routing loop! R2 will forward packets to R3, and R3 will forward packets to R2.
	![splithorizon4](https://textbook.cs168.io/assets/routing/2-070-splithorizon4.png)
- This leads us to a solution called **split horizon**, where we never advertise a route back to the person who gave us that route.
#### Rule 6B: Poison Reverse
- **Poison reverse** is an alternative way to avoid routing loops. We can use either split horizon or poison reverse to solve the problem from earlier (but not both).
- In split horizon, if someone gives me a route, I don’t advertise the route back at them. By contrast, in poison reverse, if someone gives me a route, I explicitly advertise poison back at them. In other words, I explicitly tell them, “Do not forward packets my way”.
	![poisonReverse1](https://textbook.cs168.io/assets/routing/2-071-poisonreverse1.png)
- Let’s see the demo again, but using poison reverse instead of split horizon this time. As before, we reach steady state, then R1-R2 goes down, and R2 loses its table entry.
- If we implemented split horizon, R3 would not advertise its route back to R2 at this point. In the poison reverse approach, R3 explicitly sends an advertisement back to R2: “I’m R3, and A is infinity away from me.”
	![poisonReverse3](https://textbook.cs168.io/assets/routing/2-073-poisonreverse3.png)
- R2 doesn’t have an entry for A (its old one expired), so it accepts this new, poisoned route. Now, R2’s table explicitly says that it cannot reach A via R3. We’ve avoided the routing loop with the help of poison reverse!
- In our model of the network, split horizon and poison reverse will both help avoid routing loops. More generally, poison reverse can help eliminate routing loops sooner if they ever arise.
- For example, suppose we end up with a routing loop somehow, where R2 and R3 are forwarding packets to each other.
- In the split horizon approach, no poison gets sent. R2 got its route from R3, so it won’t send anything to R3. Similarly, R3 got its route from R2, so it won’t send anything to R2. The loop exists until the table entries expire. Until then, packets could get lost in the loop.
	![splitPoison1](https://textbook.cs168.io/assets/routing/2-074-split-and-poison1.png)
- By contrast, if we used the poison reverse approach, R3 explicitly sends poison back to R2. R2 accepts this advertisement (Rule 2, route from its next-hop), and updates its table to invalidate the path via R3. The poison reverse advertisement immediately eliminates the routing loop.
	![splitPoison2](https://textbook.cs168.io/assets/routing/2-075-split-and-poison2.png)
#### Rule 7: Count to Infinity
- Split horizon or poison reverse helped us avoid length-2 loops, where R1 forwards to R2, and R2 forwards to R1. But we can still get routing loops involving 3 or more routers.
	![inf1](https://textbook.cs168.io/assets/routing/2-076-infinity1.png)
- To see why, consider this network. Suppose the tables reach steady-state. R1 and R2 both forward to R3, which forwards to A. The A-R3 link goes down! A is now unreachable. Per Rule 5, R3 updates its table to show infinite cost to A, and sends this poison to both R2 and R1.
- R2 gets the poison advertisement and updates its table (Rule 2, accept from next-hop). Now, both R2 and R3 know that A is unreachable.
- The poison advertisement to R1 is dropped! R1 doesn’t see the poison, so it still thinks it can reach A via R3. (The poison can get re-sent later, but for this demo, all the bad things that are about to happen will happen before the poison gets a chance to be re-sent.)
	![inf2](https://textbook.cs168.io/assets/routing/2-077-infinity2.png)
- Eventually, R1 sends out an advertisement. R1’s path to A is via R3, so by split horizon, it won’t advertise to R3. However, R1 will still advertise to R2. R2 doesn’t have a way to reach A, so it accepts this route. Now, R2 is fooled into thinking it can reach A with cost 3.
	![inf4](https://textbook.cs168.io/assets/routing/2-079-infinity4.png)
- R2 sends out an advertisement about its new route. Split horizon dictates that R2 won’t advertise back to R1, but it will still advertise to R3. R3 doesn’t have a way to reach A, so it accepts this route. Now, R3 is fooled into thinking it can reach A with cost 4.
	![inf5](https://textbook.cs168.io/assets/routing/2-080-infinity5.png)
- Next, R3 sends out an advertisement to R1 (not R2, per split horizon): “I am R3, and A is 4 away from me.” R1 will accept this advertisement (Rule 2, advertisement from next-hop) and update its table. Now, R1 thinks its cost to A is 5.
- Maybe you’re seeing where this is going. R1 advertises to R2 (not R3, per split horizon): “I’m R1, and A is 5 away from me.”
	![inf7](https://textbook.cs168.io/assets/routing/2-082-infinity7.png)
- R2 accepts this advertisement (Rule 2), and thinks it can reach A with cost 6. R2 advertises a cost of 6 to R3, who now thinks it can reach A with cost 7.
	![inf8](https://textbook.cs168.io/assets/routing/2-083-infinity8.png)
- R1, R2, and R3 will keep sending advertisements to each other in a cycle, with progressively higher costs (which will all be accepted by Rule 2). Also, packets for A will get stuck in a forwarding loop between these routers.
- Let’s restate the problem again. The poison didn’t correctly propagate to all hosts, so one of the routers still had a busted path in its table.
- Then, that busted path got advertised in a loop, and Rule 2 caused the costs to keep increasing, with no end in sight.
- Why didn’t split horizon rescue us? Remember, split horizon only stops a router from advertising back to its next-hop. But in this case, the loop is of length 3, and we were never advertising back to the next-hop.
- **Note: Poison reverse wouldn’t rescue us either. If R3 advertises poison back to R2, then R2 would ignore that poison, because R2’s next hop is R1, not R3**.
- This is called the **count-to-infinity** problem, and none of our fixes so far (poison expired routes, split horizon, poison reverse) can solve it.
- To solve this problem, we will enforce a **maximum cost**. In RIP, this value is 15. All costs greater than this maximum (i.e. 16 or above) are considered infinity.
- With this fix, the loop will still exist for some time, but eventually, all the costs will reach 16 (infinity). Let’s watch this in action.
- The costs are increasing with every advertisement. Eventually, R1 advertises to R2: “I’m R1, and A is 14 away from me.” R2 accepts (per Rule 2) and updates its cost to 15.
	![inf11](https://textbook.cs168.io/assets/routing/2-086-infinity11.png)
- R2 advertises to R3: “I’m R2, and A is 15 away from me.” R3 accepts (per Rule 2), but instead of updating its cost to 16, the cost is updated to infinity.
	![inf12](https://textbook.cs168.io/assets/routing/2-087-infinity12.png)
- Next, R3 advertises to R1: “I’m R3, and A is infinity away from me.” R1 accepts (per Rule 2), and now R1 also has a cost of infinity. (Note: This advertisement looks just like poison, though the infinity originated from counting to infinity instead of detecting a failure.)
	![inf13](https://textbook.cs168.io/assets/routing/2-088-infinity13.png)
- Finally, R1 advertises to R2: “I’m R1, and A is infinity away from me.” R2 accepts (per Rule 2), and now all the routers have a cost of infinity.
	![inf14](https://textbook.cs168.io/assets/routing/2-089-infinity14.png)
- We’ve reached steady-state again! Any future advertisements would all be advertising infinite cost, and they won’t change the tables.
- Eventually, the infinite-cost entries would all expire. Or, if another route to A appears, it would replace the infinite-cost entry.
#### Eventful Updates
- There are three occasions where a router might want to send advertisements:
	1. Send advertisements when the table changes.
		- These are called **triggered updates**. The table might change when we accept a new advertisement, or
		- When a new link is added (e.g. new static route), or when a link goes down (e.g. route gets poisoned).
	2. Send advertisements periodically, once every advertisement interval.
	3. Send advertisements when a table entry expires (and gets replaced by poison).
- Note that triggered updates are an **optimization**. Instead of advertising every time the table changes, we could just wait for the next advertisement interval to advertise the changes.
- This protocol would still be correct. However, triggered updates, in addition to the periodic updates, help our protocol converge on correct routes faster, because we propagate new information the instant we learn about it.
# Sources
- [Lecture 5 -Routing 2: Distance-Vector](https://www.youtube.com/watch?v=6gCFWEqusMA).
- [Lecture 6 -Routing 2: Distance-Vector (continued)](https://www.youtube.com/watch?v=glKg5DAXW7o).
- [Distance-Vector Protocols](https://textbook.cs168.io/routing/distance-vector.html).