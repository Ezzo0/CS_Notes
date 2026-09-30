# Explanation
- In the previous sections, we developed an algorithm for congestion control. This algorithm told us how to adjust rate in response to congestion, but it didn’t actually tell us what that rate is.
- We want a simple equation that gives us throughput as a function of a path’s RTT and loss rate. This equation can allow operators and customers to estimate the rate of a TCP connection.
- To simplify our model, we’ll make a few assumptions: There is a single TCP connection. We’ll ignore the slow-start phase. We’ll assume the RTT is some fixed number.
- When the [[0x17_TCP Design#Window-Based Algorithms|window]] size reaches the maximum bottleneck bandwidth $Wmax$ (some constant), we’ll assume we get exactly one packet loss. Since we only lose one packet, our loss will be detected by duplicate acks (no timeouts).
#### Throughput in Terms of Window Size
- In this simplified model, we detect loss when the window size reaches $Wmax$, and our window size changes to $\frac{1}{2}Wmax$ as a result.
- Then, for each subsequent RTT, our window size will increase by $\frac{1}{2}Wmax + 1$, then $\frac{1}{2}Wmax + 2$, then $\frac{1}{2}Wmax + 3$, etc. Eventually, the window size will reach $Wmax$ again and be halved, and this process will repeat.
- Starting at $\frac{1}{2}Wmax$ and reaching $Wmax$ takes $\frac{1}{2}Wmax×RTTs$ (adding 1 per iteration, and each iteration is one RTT). This also tells us that there are $\frac{1}{2}Wmax×RTTs$ between each loss.
	![timebetweenLosses](https://textbook.cs168.io/assets/transport/3-088-equation1.png)
- Within each RTT, the average window size is $\frac{3}{4}Wmax$ (right in between $\frac{1}{2}Wmax$ and $Wmax$).
- This window size is measured in packets (since we were adding 1 packet per iteration). Each packet can contain **_MSS_** bytes ([[0x18_TCP Implementation#^45f742|maximum segment size]]), so the average window size in bytes is $\frac{3}{4}Wmax×MSS$.
- The window size tells us how much data we can send in each RTT. Thus, to compute the rate, we divide window size (data) by RTT (time) to get an average rate of $\frac{3}{4}Wmax× \frac{MSS}{RTT}$.
#### Throughput in Terms of Loss Rate
- Our equation for throughput so far is: $\frac{3}{4}Wmax×\frac{MSS}{RTT}$. But our goal is to express throughput in terms of RTT and loss rate (denoted $p$). So, we now need to express $Wmax$ in terms of the loss rate $p$.
- From earlier, we deduced that a packet is lost once every $\frac{1}{2}Wmax × RTTs$. This was the time it took after a drop to climb back up to $Wmax$ and encounter another drop.
- So, to determine the loss rate, we just need to figure out how many packets are sent in $\frac{1}{2}Wmax × RTTs$.
	![lossRate](https://textbook.cs168.io/assets/transport/3-089-equation2.png)
- Graphically, the number of packets sent is the area of this shape (rate times time), or equivalently, the area under the curve (the curve shows rate, and we want integral of rate).
- We know from earlier that the average window size is $\frac{3}{4}Wmax$, so this is the number of packets sent per RTT. Therefore, across $\frac{1}{2}Wmax × RTTs$, we expect to send $(\frac{1}{2}Wmax) × \frac{3}{4}Wmax = \frac{3}{8}W^2max$ packets.
- Now that we know the number of packets sent between losses, we know that the **loss rate is one lost packet, divided by the number of packets sent between losses**. (For example, if we send 100 packets between losses, the loss rate is roughly 1/100).
- Therefore, our loss rate is $p = \frac{1}{(\frac{3}{8}W^2max)} = \frac{8}{3W^2max}$. Now, we have a relation between $Wmax$ and $p$, so we just need to do algebra to isolate $Wmax$ in terms of $p$.
	![[wmax_p_isolation.png]]
- Now, we can do some more algebra to take our throughput equation from earlier and replace $Wmax$ with $p$:
	![[throughput_intermsof_p.png]]
#### Implications of Equation
- Throughput is inversely proportional to the square root of the loss rate. Intuitively, if the loss rate is higher, then the throughput is lower. This makes sense, because losing more packets means that the window size gets halved more often.
- Throughput is inversely proportional to RTT. Intuitively, if the RTT is lower, then the throughput is higher. This makes sense, because the window size increases every time we receive an ack, and a lower RTT means we get more acks more often.
- This relationship between RTT and throughput can be a problem if we have multiple connections with different RTTs.
- The connection with the lower RTT is going to be receiving acks more quickly, which means this connection also increases its window size and sends packets faster.
- In this case, it turns out the lower-RTT connection gets twice as much bandwidth as the higher-RTT connection.
- Fundamentally, TCP is **unfair when RTTs are heterogeneous** (not the same). A shorter RTT improves propagation time, but it also helps TCP ramp up its rate faster. We accept this as a feature of TCP, and there’s nothing we do about this in practice.
#### Rate-Based Congestion Control
- Our congestion control protocol results in choppy throughput. As seen in the graph, the rate repeatedly swings between $W/2$ and $W$. Some applications don’t like the constantly-changing rate, and would prefer to send data at a steady rate (e.g. streaming applications).
- One possible solution for these applications is **equation-based** or **rate-based congestion control**, which abandons the rules for dynamically adjusting the rate, and instead simply follows the equation.
- To send data at a smooth rate, you can measure RTT and loss rate, plug them into the throughput equation, and constantly send at the calculated rate.
- This solution also maintains fairness (doesn’t hog bandwidth), because the equation ensures that we consume no more bandwidth than TCP would in a similar setting.
# Sources
- [Lecture 14 - Transport 4: Congestion Control II](https://www.youtube.com/watch?v=td_edyGAOFk).
- [TCP Throughput Model](https://textbook.cs168.io/transport/throughput-model.html).