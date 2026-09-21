# Explanation
- Recall that in [[0x17_TCP Design|TCP]], the sender maintains a sliding window of consecutive bytes/packets in flight. The size of the window is determined by [[0x17_TCP Design#Window Size Flow Control|flow control]] (decided by buffer space at recipient) and [[0x20_Congestion Control Design|congestion control]] (rate computed by sender).
- More specifically, in flow control, the recipient sends an advertised [[0x17_TCP Design#Window-Based Algorithms|window]], indicating how many more bytes can be sent without overflowing the recipient’s memory. This advertised window value is sometimes abbreviated **RWND (receiver window)**.
- In congestion control, the sender maintains a value, sometimes abbreviated **CWND (congestion window)**, which denotes the rate the sender can send [[0x18_TCP Implementation#^45f742|packets]] without overloading [[0x04_Links|links]]. This value will be dynamically set and adjusted by the congestion control algorithm.
- The sender’s window is computed as the minimum of CWND and RWND. For this lecture, we’ll assume that RWND is larger than CWND, so the bottleneck is the network, not the recipient’s memory. This is usually, but not always true in practice.
- To detect loss, we maintain a single timer for the left-most packet in the [[0x18_TCP Implementation#Sliding Window|window]]. If the timer expires without that packet being acked, we re-send the left-most packet in the window.
- Also, to detect loss, we count the number of duplicate acks, and re-send the left-most packet if we see 3 duplicate acks. This duplicate ack-based approach is sometimes called **fast retransmit**.
#### Windows and Rates
- How do we adjust the rate for congestion control, and how do we compute the congestion window? It turns out that these two values are directly related, and adjusting the window is achieved by adjusting the rate.
- The window size and the rate of sending data are correlated by the following equation: **Rate X [[0x17_TCP Design#^3ce022|RTT]] = [[0x17_TCP Design#Window-Based Algorithms|window]] size**.
- Intuitively, you can think of window size and rate as the same quantity, expressed in two different “units of measurement.” An increased window size means we’re sending data faster, and vice-versa.
- To see why this equation holds, consider the first RTT. We can send **window size number of [[0x00_Layers of the internet#Layer 3 Packets Abstraction|packets]] during this first RTT**, for a rate of $window Size / RTT$.
#### Event-Driven Updates
- In our conceptual model, our goal is to adjust the **rate/window once per “iteration,”** but we haven’t formalized how to measure each iteration.
- We can roughly define each **iteration as one RTT, but the RTT itself is a dynamically changing value** that we can’t accurately measure.
- In order to update the window size in a more predictable, measurable way, we can consider the various events that the existing TCP implementation responds to, and update the window each time one of these events occurs. These are called **event-driven updates**.
- The three TCP events where we need to update the window size are: **new ack, 3 duplicate acks, and timeout**.
- When we see a new ack (for data that was previously not acknowledged), this is a sign that our data made it through the network without loss.
- In our model, we detect congestion by checking for loss, so a new ack is a sign that the network is not congested. Therefore, when we see a new ack, we can increase the window size (either during [[0x20_Congestion Control Design#Discovering Initial Rate|slow-start discovery]], or [[0x20_Congestion Control Design#Adjustments AIMD Dynamics|AIMD adjustment]]).
- When we see 3 duplicate acks, we mark a packet lost. This is a signal of isolated loss, which indicates mild congestion.
- We lost a packet, but subsequent packets are still being received. To react to this loss, we will decrease the window size (during AIMD adjustment).
- When we encounter a timeout, we mark a packet lost. The fact that we detected the loss after a timeout, not duplicate acks, is a signal of many packets being lost (heavy congestion).
- To see why, consider a window size of 100 packets. If we encounter a timeout, this means we didn’t get an ack for the left-most packet in the window. But it also means that we failed to get 3 duplicate acks for any other packets in the window during the entire duration of the timer.
- A timeout means that very few, if any, packets are being received, and **something bad has happened**. If we discover a timeout, something unexpected has happened (e.g. network changed), and we should no longer trust our current window size.
- To react, we should **go back to the slow-start phase and re-discover a good window size**. This isn’t the only way to react to timeout, but this is what [[0x17_TCP Design|TCP]] decided.
#### Event-Driven Slow Start
- In our conceptual model, we implemented slow start by choosing a slow rate, and increasing the rate exponentially (e.g. doubling on each iteration) until we encounter the first loss. We now need an event-driven way to double the window once per RTT.
- TCP starts with a small window of 1 packet. Recall, we can convert packet to bytes with the [[0x18_TCP Implementation#^45f742|maximum segment size (MSS)]], and then convert bytes to rate by dividing $MSS/RTT$.
- Every time we get an acknowledgement, we will increase the window size by 1 packet. The intuition for what will happen is:
	 ![eventDrivenSS|400](https://textbook.cs168.io/assets/transport/3-071-event-driven-ss.png)
- In general, assuming no loss and no re-ordering, every time we receive an ack, the sliding window allows us to send one more packet, and the increased window allows us to send another packet. Because every ack leads to 2 packets being sent, we get the behavior where the window doubles every RTT.
- For example, within an RTT interval where we receive 16 acks, each ack triggers to packets being sent, for a total of 32 packets. Then, in the next RTT interval, those 32 packets will be acked, triggering 64 packets being sent.
- Eventually, after some time spent doubling the window every RTT (increasing the window by 1 for every ack), we’ll encounter loss. This also means we’ve learned the maximum allowable “safe” rate for sending packets without encountering loss.
- We’ll remember this rate in a new parameter called **SSTHRESH (slow start threshold)**. Specifically, as soon as we encounter packet loss, we’ll set SSTHRESH to half the window size.
- For example, if a window of 16 packets doesn’t cause loss, but a window of 32 packets does cause loss, then we would set SSTHRESH to 16.
	![ssThresh|600](https://textbook.cs168.io/assets/transport/3-072-ssthresh-ss.png)
#### Implementing Additive Increasing
- In our conceptual model, after [[0x20_Congestion Control Design#Discovering Initial Rate|slow start]], we want to slowly (additively) increase the rate when there’s no loss. We need an event-driven way to increase the window by 1 packet for each RTT.
- We don’t have an exact number for the RTT, but we do know that within a single RTT, we expect a window’s worth of packets to be acked.
- For example, with window size 10, we receive 10 acks per RTT. If we increase the window by **1/10 packet per ack**, then across a RTT, the window should increase by 1 packet, as desired.
- Each time we receive an acknowledgement, we will take the current window size CWND and reassign it to **CWND + (1/CWND)**. This increases the window by a fraction of a packet on each ack. After a full window’s worth of packets (i.e. after one RTT), the window increases by 1 packet.
- Formally, TCP measures the window in bytes, not packets, so $(1/CWND)$ is equivalent to $MSS * (MSS/CWND)$ in bytes.
- In (1/CWND), the numerator is 1 packet (total increase in an RTT), and the denominator is CWND measured in packets. Since the denominator is now measured in packets, we also have to measure the numerator in packets: 1 packet = MSS bytes.
- But the fraction $1/CWND$ or $MSS/CWND$ is still a ratio (dimensionless), representing the fraction to be increased on each ack. The total increase we want is 1 packet = MSS bytes, so we have to **multiply this fraction by MSS bytes**.
- As an example, suppose our CWND was 3 packets = 150 bytes (assuming MSS = 50 bytes). In the packet view, we would add 1/3 packets to the window each time, for a total increase of 1 packet.
- In the byte view, we can divide MSS/CWND = 50/150 to get the same 1/3 ratio that we need to step by each time, for a total increase of 1. But we still need to multiply by MSS so that the total increase is MSS instead of 1.
	![eventDrivenAIMD](https://textbook.cs168.io/assets/transport/3-073-event-driven-aimd.png)
- Note that the increase **isn’t perfectly linear, but provides a good enough approximation**. For example, starting with CWND = 4, the first update is 4 + 1/4 = 4.25, and the second increase is 4.25 + 1/4.25 = 4.49. After four updates, the window size would be 4.92 in this approximation (we wanted it to be 5 in the exact model).
#### Implementing Multiplicative Decrease
- If we detect loss from 3 duplicate acks, we divide the window size by 2.
- Recall that if the retransmission timer expires, we interpret the timeout as multiple packets being lost (we didn’t even get duplicate acks).
- We assume that the current window might be way off, and in order to be cautious, we’ll rediscover a good rate from scratch.
- First, we’ll make a note that the current rate is too high, and the best known safe rate is half of our current rate (following the multiplicative decrease principle). To record this safe rate, we’ll set SSTHRESH to half the current window.
- Then, we’ll set the window size back to 1 packet, and repeat the slow start process again.
- Note that when we re-try slow start, we need to be careful not to return to the dangerous rate with timeouts from earlier. Fortunately, we set SSTHRESH to be just below the dangerous rate.
- Therefore, in subsequent slow start re-trys (where SSTHRESH is set), **as soon as our window exceeds SSTHRESH, we should switch from multiplicative to additive increasing**. On the first slow start, SSTHRESH is unset (or infinity).
- To summarize: In slow-start, we increase the window by 1 packet for each ack (results in doubling the rate on each RTT). When in AIMD, we increase the window by a fraction of the window size for each ack (results in increasing by 1 for each window’s worth of data).
- We decrease the window by halving it when receiving 3 duplicate acks, and changing it to 1 on a timeout.
- If we plot rate over time, we see the initial exponential growth (slow start). As soon as we experience loss, we cut the rate in half, and switch to AIMD mode. Now, we increase linearly until we encounter loss, and we halve the rate each time we encounter loss.
	![sawtooth-ssthresh](https://textbook.cs168.io/assets/transport/3-074-sawtooth-ssthresh.png)
#### Fast Recovery: Running Example
- When we encounter an isolated packet loss, the congestion window is halved, as intended. However, this has the unintended side effect of causing the sender to stall for some time before it can continue sending packets.
- To see this in action, let’s consider a running example. We send 10 packets, numbered 101 through 110. The first packet (101) is dropped.
- After the third duplicate ack(101) (generated by receiving 102, 103, and 104), the sender re-sends 101.
- Eventually, the ack for the re-sent 101 arrives. It says ack(111), because packets 102 through 110 were all received earlier, and with the receipt of 101, the next expected byte is 111.
	![fastRecoveryExample|500](https://textbook.cs168.io/assets/transport/3-075-fastrecovery1.png)
#### Fast Recovery: The Problem
- Let’s assume that CWND starts at 10. Packets 101 through 110 are allowed to be in flight. The sender sends 101 through 110, but 101 is dropped.
- The sender sees ack(101), generated from the other side receiving 104. This is the third duplicate ack, so we must decrease CWND to 5.
- The first unacked byte is still 101, and CWND is 5, so packets 101 through 105 are allowed to be in flight. The sender still can’t send anything new.
- We re-send 101 (left-most packet in window) because we saw the third duplicate ack.
	![fastRecoveryWindow](https://textbook.cs168.io/assets/transport/3-076-fastrecovery2.png)
- The sender gets ack(101) 5 times from the other side receiving 105, 106, 107, 108, 109, 110. In every case, 101 is still the first unacked byte, so the window is still 101 through 105, and the sender can’t send anything new.
- What happened here? **Only a single packet was dropped, but as a result, the sender had to completely stop sending for a long time**.
- Eventually, the sender receives ack(111) from the re-sent 101. This causes the window to leap forward and slide to the new first unacked packet, 111. CWND is still 5, so the sender is now able to send 111 through 115.
	![fastRecoveryWindowLeap](https://textbook.cs168.io/assets/transport/3-078-fastrecovery4.png)
- What happened here? We now have a secondary problem. The sender stalled for a long time, but as soon as 101 was acked with ack(111), the window leapt forward all the way to 111-115, and the sender **suddenly has to scramble to send 111-115 all at the same time**.
- The sender stalled for a long time, sending nothing, and then suddenly scrambled to send 111-115 at the same time. Now, the sender has to **wait another full round-trip** for 111-115 to get acked, before it can send 116 and beyond.
	![fastRecoverySenderWait](https://textbook.cs168.io/assets/transport/3-079-fastrecovery5.png)
	![fastRecoveryAnotherWait](https://textbook.cs168.io/assets/transport/3-080-fastrecovery6.png)
- Note that after we get 3 duplicate ack(101) messages, we re-send 101, and we **never re-send 101 again**, even if more duplicate ack(101) messages come in. This is just the TCP rule for re-sending on duplicate acks.
#### Fast Recovery: The Idea
- So, how do we solve this problem? Ideally, we don’t want the sender to stall, and we want the sender to keep sending later packets (111 onwards), even if 101 gets lost.
- Notice that even though the sender can’t deduce exactly which packets arrive, the sender can deduce that later (non-101) packets are getting received.
- When we see ack(101), generated from 102 being received, we don’t actually know that 102 was received, but we know some packet (non-101) got received. Therefore, only 9 packets remain in flight.
- As we keep receiving duplicate ack(101) messages, we can deduce that fewer packets remain in flight:
	- After ack(101) from 102: 9 packets in flight.
	- After ack(101) from 103: 8 packets in flight.
	- After ack(101) from 104: 7 packets in flight.
	- After ack(101) from 105: 6 packets in flight.
	- After ack(101) from 106: 5 packets in flight.
	- After ack(101) from 107: 4 packets in flight.
	- After ack(101) from 108: 3 packets in flight.
	- After ack(101) from 109: 2 packets in flight.
	- After ack(101) from 110: 1 packet in flight.
- Eventually, after we get ack(101) nine times (from 102 through 110 being received), we know that only 1 packet remains in flight, namely 101.
- After the isolated loss, we really want CWND to be 5, which means we want 5 packets in flight at any given time. By the time we get ack(101) from 107, we can deduce that only 4 packets remain in flight. (In reality, they are 101, 108, 109, 110, though the sender doesn’t know that.)
- At this point, we’d like to be **able to send 111, for a total of 5 packets in flight**. But the window won’t let us do that, because the window is still stuck at 101 (first unacked byte) through 105 (CWND bytes later).
- The key idea that will un-stall the sender is: Let’s grant the sender temporary credit for each duplicate ack.
- When a duplicate ack arrives, we can deduce that one fewer packet is in flight, though we don’t know which one. To account for this, we will **_artificially extend the window by 1 packet_**, to allow the sender to send one more packet.
#### Fast Recovery: The Solution
- Let’s take this idea of artificially extending the window for each duplicate ack, and apply it to the example from before.
- As before, the window starts at 101 through 110, and we send out 10 packets.
	![FastRecoverySolution1](https://textbook.cs168.io/assets/transport/3-081-fastrecovery7.png)
- As before, we get ack(101) from 104, the window stays unchanged, and we can’t send anything new. The third duplicate ack means we decrease CWND to 5, so the window is now 101 through 105.
- However, we got 3 acks, so we artificially extend the window by 3 to account for those acks. Thus, CWND is actually set to 5 + 3 = 8. But, If you look at this extended window, 3 of the packets are **acked** (102, 103, 104, though we don’t know it’s these), and the other 5 are in-flight. This achieves our intended **window of 5 packets in-flight**.
- Next, we get ack(101) from 105. This allows us to extend the window again, to 9. Now the window spans 101 through 109, so we still can’t send new packets.
	![fastRecoveryWindowExtended1](https://textbook.cs168.io/assets/transport/3-082-fastrecovery8.png)
- Next, we get ack(101) from 106. We extend the window again to 10, spanning 101 through 110, and we can’t send anything new.
- Next, we get ack(101) from 107. We extend the window again to 11, spanning 101 through 111. We can now send out 111!
- Next, we get ack(101) from 108. We extend the window again to 12, spanning 101 through 112. We can now send out 112!
- Next, we get ack(101) from 109. We extend the window again to 13, spanning 101 through 113. We can now send out 113!
	![fastRecoveryWindowExtended2](https://textbook.cs168.io/assets/transport/3-083-fastrecovery9.png)
- Next, we get ack(101) from 110. We extend the window again to 14, spanning 101 through 114. We can now send out 114!
- Eventually, we get ack(111) from the re-sent 101. At this point, we can reset CWND to its original intended value of 5, so that the window spans 111 through 115. This allows us to send out 115!
	![fastRecovery101ACK](https://textbook.cs168.io/assets/transport/3-084-fastrecovery10.png)
- With this fix, we have solved our problem of the stalled sender. Originally, the sender had to wait for the re-sent 101 to be acked before sending new packets. Now, the sender is now able to keep sending packets before the re-sent 101 is acked.
	![fastRecoveryproblem1Resolved](https://textbook.cs168.io/assets/transport/3-085-fastrecovery11.png)
- We also solved the secondary problem from earlier, where the window leapt forward and we sent a burst of new packets (111 through 115). Now, 111 through 114 were sent out earlier, and when the window leapt forward, we only had to send out 115.
- Without this fix, we had to stall for another round-trip while waiting for the burst of 111 through 115 to be acked.
- Now, because we stayed busy earlier and sent out 111 through 114, they’ll get acked earlier, and we can keep sending 116 and beyond without that entire RTT of stalling.
	![fastRecoveryproblem2Resolved](https://textbook.cs168.io/assets/transport/3-086-fastrecovery12.png)
#### Fast Recovery: Implementation
- When we detect packet loss from duplicate acks, we temporarily enter the **fast recovery** mode, where additional duplicate acks artificially extend the window to prevent stalling.
- Fast recovery mode is triggered when we receive 3 duplicate acks. Instead of just halving CWND, as we did before, we now **set CWND to CWND/2 + 3**, where the window is artificially extended by 3 for the 3 duplicate acks we got. We also set SSTHRESH to CWND/2, so that we remember the new safe rate for later.
- Eventually, when we receive a new, non-duplicate ack, we leave fast recovery mode and set CWND to SSTHRESH.
- Note that while we were artificially extending the window, SSTHRESH always helped us remember the original halved rate that we want to ultimately send at.
#### TCP State Machine
- We are finally ready to put all the pieces together and implement TCP, with congestion control. The sender maintains 5 values:
	- The duplicate ack count helps us detect loss earlier than timeouts. It’s initialized to 0.
	- The timer is used to detect loss. There’s just a single timer.
	- RWND is used for flow control (don’t overwhelm recipient buffer).
	- CWND is used for congestion control. It’s initialized to 1 packet.
	- SSTHRESH helps the congestion control algorithm remember the latest safe rate. It’s initialized to infinity.
- The recipient maintains a buffer of out-of-order packets.
- The sender responds to 3 events: Ack for new data (not previously acked), duplicate ack, and timeout.
- The recipient responds to receiving a packet, by replying with an ack and a RWND value.
- Let’s see how the sender responds to each of the 3 events.
	- When we receive an ack for new data, not previously acked:
		- If in slow-start mode, we increase CWND by 1. This allows the CWND to double each RTT.
		- If we’re in fast-recovery mode, we set CWND to SSTHRESH, so that we leave fast recovery (since we just got a new ack).
		- If we’re in congestion avoidance mode, we add 1/CWND to CWND, so that CWND increases by 1 per RTT (additive increase). We also reset the timer, reset the duplicate ack count, and, if the window allows, send new data.
	- When we receive a duplicate ack, we increment the duplicate ack count.
		- If the count reaches 3, we re-send the left-most packet in the window. This is sometimes called **fast retransmit**.
		- We also enter fast-recovery mode by setting SSTHRESH to CWND/2 (remember last safe rate) and set CWND to $CWND/2 + 3$ (adding 3 to artifically extend the window for duplicate acks).
		- If the count exceeds 3, we stay in fast-recovery mode and artificially extend the CWND by 1 for every subsequent duplicate ack.
	- When the timer expires, we re-send the left-most packet in the window. We also go back to slow-start mode, setting SSTHRESH to CWND/2 (remembering the last safe rate), and resetting CWND back to 1 packet.
	![TCP-stateMachine](https://textbook.cs168.io/assets/transport/3-087-state-machine.png)
#### TCP Congestion Control Variants
- There are several variants of the TCP congestion control algorithm, all implemented in the end host’s operating system.
	- In TCP Tahoe, if we get three duplicate acks, we reset CWND to 1, instead of halving CWND.
	- In TCP Reno, if we get three duplicate acks, we halve CWND. On timeout, we reset CWND to 1.
	- TCP New Reno is the same as Reno, but adds fast recovery. This is what we just implemented.
	- Other variants exist too. In TCP-SACK, we add selective acknowledgments where acks contain more detail (e.g. received all up to 13, and 18 too).
- How can all these different variants co-exist? Why don’t we need a single uniform protocol that everybody speaks? Remember, congestion control is implemented at the end hosts, so the sender can do whatever they want to adjust their rate.
- Ultimately, the network and the other end hosts just see TCP packets being sent at a (hopefully reasonable) rate, and they don’t care how the rate is being computed. The underlying TCP packet format doesn’t change with the different congestion control algorithms.
- Not all protocols are compatible, though. If you use the TCP-SACK variant with selective acknowledgements, and I use TCP Tahoe, we have a problem. You expect selective acks, but I’m only providing cumulative acks.
# Sources
- [Lecture 13 - Transport 3: Congestion Control I](https://www.youtube.com/watch?v=c_iL0b1kXbk).
- [Lecture 14 - Transport 4: Congestion Control II](https://www.youtube.com/watch?v=td_edyGAOFk).
- [Congestion Control Implementation](https://textbook.cs168.io/transport/cc-implementation.html).