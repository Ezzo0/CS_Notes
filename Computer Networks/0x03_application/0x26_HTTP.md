# Explanation
- HTTP runs over [[0x17_TCP Design|TCP]]. Two people who want to send data over HTTP will first start a TCP connection. Then, they can use its [[0x16_Transport Layer Principles#Bytestream Abstraction|bytestream]] abstraction to reliably exchange arbitrary-length data.
- Hosts running HTTP don’t have to worry about packets being reordered, dropped, and so on.
- HTTP is a **client-server** protocol. When forming an HTTP connection, the server must listen for connection requests on the well-known, constant port number 80.(HTTPS, a more recent secure version, uses port 443).
- The client can choose any random ephemeral port number to start the connection, and the server can send replies to that port number.
- HTTP is a **request-response** protocol. For each request that the client sends, the server sends exactly one corresponding response.
#### HTTP Requests
- The HTTP request message is formatted in human-readable plaintext, which means you can type raw HTTP requests into the terminal. The request contains three parts: **_method, URL, version, and optional content_**.
- The message ends with a newline (technically, CRLF). The version number specifies what version of HTTP you’re using, e.g. HTTP/0.9, HTTP/1.0, HTTP/1.1, etc.
- The requested URL identifies a **resource on the server**. You can think of the URL as the filepath of what you’re trying to retrieve from the remote server.
- For example, in the URL http://cs168.io/assets/lectures/lecture1.pdf, we’re trying to retrieve a file in the assets/lectures folder, named lecture1.pdf, on the cs168.io remote server. (Servers aren’t required to work this way, but it’s a useful intuition to have.)
- The method identifies what **action the user wants to perform**. Initially, HTTP only had one method, GET, which allows a client to retrieve a specific page (indicated by the URL) from the server.
- Later, HTTP was extended to add other methods. Notably, the POST method was added, which allows the client to supply information to the server as well.
- Some less-used methods exist as well. HEAD retrieves the headers (metadata) of the response, but not the actual content of the response.
- Other methods like PUT, CONNECT, DELETE, OPTIONS, PATCH, and TRACE extend HTTP into a protocol that lets the user interact with content on the server.
- The user can now make changes to the content, as opposed to the original design, where the user could only retrieve content. These extra methods make HTTP very flexible for all sorts of different application.
- Note that with other methods like POST, we still have to provide a URL to indicate how to interpret the data we’re sending.
- For GET requests, the content of the request is usually empty, since we’re asking for a page from the server and not sending any of our own information. By contrast, for a POST request, the content of the request contains the data we want to send to the server.
#### HTTP Responses
- Each HTTP request corresponds to one HTTP response. The response contains four parts: **version, status code, optional message, and content**.
- The content is where the server would put, for example, the page that the user requested in a GET request.
- The status code is a number that allows the server to indicate the result of the client’s request. Each status code has a corresponding human-readable message.
- The status codes are classified into various categories according to numeric values:
	- **100 = Informational responses.**
	- **200 = Successful responses.**
	- **300 = Redirection messages.**
		- These allow the server to tell the client that they should go look for the resource (specified by the URL) somewhere else.
		- Two common ones are 301 Moved Permanently and 302 Found (a weird name for moved temporarily).
		- Sometimes, the status code itself doesn’t provide enough context (as seen with these redirects).
		- Therefore, the response will also contain additional information about where the resource has moved (e.g. another URL).
		- The use of more specific status codes allows the client to determine its future behavior based on the code.
		- For example, 301 Moved Permanently tells the client to stop looking in the original location, while 302 Found (moved temporarily) might tell the client to come back and check again later.
	- **400 = An error attributable to client action.**
		- 401 Unauthorized says that the client is not allowed to access this content, but if they authenticate their identity (e.g. log in), then they might be able to access the content.
		- 403 Forbidden says that the client is authenticated, and the server knows their identity, but they’re still not allowed to access the content.
	- **500 = An error attributable to server action.**
		- 500 Internal Server Error and 503 Service Unavailable are common.
		- There’s not much the client can do about these errors, except maybe try again later.
#### HTTP Headers
- If the client has additional information they’d like to send to the server, they can include additional metadata called **headers**.
- In HTTP/1.1, no headers are mandatory, so it’s legal to not include any (though the server/client might expect a header and error).
- For example, the **Location header** can be used in HTTP 300 responses to indicate where the resource has moved.
- Sometimes, the header information is optional. For example, the **User-Agent** header in the request lets the client tell the server about the client browser or program.
- This could allow the request to be processed differently depending on the header field.
- Other times, header information is more critical. For example, **Content-Type** tells us whether the payload is an HTML page, image, video, etc. This tells the browser how to display the HTTP response.
- If a server is hosting multiple websites, the **Host** header can be used in requests to specify what website to request.
- Some headers are relevant in requests. These allow the client to pass information to the server.
- For example, the **Accept header** lets the client tell the server which content type the client is expecting (e.g. HTML for human-readable pages, JSON for machine-parsable data).
- Other headers are relevant in responses. For example, **Content-Encoding** tells us how the bits of the response should be interpreted (e.g. Unicode/ASCII for human-readable text, or gzip for a compressed file). The **Date header** tells us when the server generated the response.
#### Speeding Up HTTP with Pipelining
- Loading a single page in your web browser can require several HTTP requests. When you make a request for a YouTube video, your browser has to make separate requests for the video itself, the HTML with the other text on the webpage, thumbnails of related videos, and so on. Many of these requests probably go to the same server.
- Recall that HTTP runs over TCP. In the naive case, every separate request would require starting a new TCP connection with a 3-way handshake.
- After the request, we close the connection and then immediately re-do a handshake for the next request.
	![seperatedTcpConnection](https://textbook.cs168.io/assets/applications/4-16-no-pipeline.png)
- HTTP 1.1 optimized this by allowing multiple HTTP requests and responses to be pipelined over the same connection. Now, we no longer need a separate TCP connection (with a separate handshake) for every request.
- One downside to this optimization is, the server now has to keep **more simultaneous open connections**. The server needs to have some way to time out connections.
- If the server gets overloaded with open connections, the client might get an error like 503 Service Unavailable. Attackers could exploit this in a denial-of-service attack.
	![combinedConnections](https://textbook.cs168.io/assets/applications/4-17-pipeline.png)
#### Speeding Up HTTP with Caching: Types
- Another strategy for speeding up HTTP is caching responses to avoid making duplicate requests for the same data. If we don’t cache, every request must reach the server.
	![noCache](https://textbook.cs168.io/assets/applications/4-18-nocache.png)
- There are three types of HTTP caches:
	- **Private caches**:
		- Associated with a specific end client connecting to the server (e.g. the cache in your own browser).
		- Now, if the same user requests the same resource a second time, they can fetch the resource from their local cache. However, private caches are not shared between users.
		![privateCache](https://textbook.cs168.io/assets/applications/4-19-privatecache.png)
	- **Proxy caches**: 
		- Do exist in the network (not on the end host).
		- Controlled by the network operator, not the application provider.
		- These caches can be shared between lots of users, so a user requesting a resource for the first time might get the data from the proxy cache instead of the origin server.
		- One problem with proxy caches is, the clients need some way to be redirected to the proxy cache. The application isn’t running the proxy cache, so the origin server doesn’t necessarily know about the proxy cache.
		- The network operator needs some way to control the end client to inform them about the proxy cache.
		- One common approach is lying in [[0x25_DNS|DNS]] responses, which is possible if the network operator controls both the proxy cache and the [[0x25_DNS#Stub Resolvers and Recursive Resolvers|recursive resolver]].
		- When the client makes a request to the origin server, it has to look up the origin server’s IP address. The recursive resolver can lie and say, “The IP address of the origin server is, 1.2.3.4 (proxy cache’s IP address).” Now, requests to the origin server go to the proxy cache instead, who can serve cached responses.
		- Or, if the requested resource isn’t in the proxy cache, the proxy cache can make a request to the origin server, and then the cache can serve the request back to the user.
		- Another problem with proxy caches is, the application isn’t managing the proxy cache. The origin server has to trust that the proxy cache is doing the right thing (e.g. respecting cache expiry dates, serving the correct data).
		![proxyCache](https://textbook.cs168.io/assets/applications/4-20-proxycache.png)
	- **Managed caches**:
		- Do exist in the network.
		- Controlled by the application provider.
		- Deployed separately, and are not the original server that generated the content.
		- Because applications control both the origin server and the cache, they can redirect users to the caches themselves.
		- For example, if you request a YouTube video page from the origin server, the reply might contain the HTML (video title, comments).
		- The HTML might then include links to specifically fetch the video and images from the proxy caches (e.g. load from static.youtube.com instead of youtube.com).
		![managedCaches](https://textbook.cs168.io/assets/applications/4-21-managedcache.png)
#### Speeding Up HTTP with Caching: Implementation
- To implement caching, we’ll need to use headers to carry some metadata about caching (e.g. how long to cache the data).
- The original legacy caching functionality in HTTP/1.0 used the **Expires header**, which just specified how long the data can be cached.
- In HTTP/1.1, a more sophisticated **Cache-Control header** was introduced. To support compatibility, some web servers will return data with both headers.
- HTTP/1.0 clients won’t understand the newer Cache-Control header and will ignore it. HTTP/1.1 clients will prioritize the newer Cache-Control header over the older Expires header.
- The Cache-Control header specifies what types of caches can cache the data, and how long the data can be cached.
- For example, if the resource is dynamic and changes per user, but stays the same across time for a specific user, then the server could reply with: `Cache-Control: private, max-age:86400`.
- This says that this content should only be stored in a user’s local cache (not in shared proxy/managed caches), and can be stored for one day (86400 seconds).
- Some data cannot be cached (e.g. dynamic content that changes frequently). In this case, the server can set `Cache-Control: no-store` to indicate that the client and proxy cannot cache the content.
- The Cache-Control header is optional, so there’s no guarantee that the client will read or respect the header. You can think of this header as a request from the server cache something. This is especially a concern for proxy caches, which are not operated by the application provider.
- By contrast, a private cache is run by the client (i.e. their browser) and breaking rules only affects the client themselves.
- A managed cache is run by the same application provider, so they can enforce that rules from the origin server are obeyed by the managed caches.
- This header can be used for more complex policies as well. For example, the server might say, you can cache this data, but before you use the cached data, please make an HTTP HEAD request to re-request the header and re-validate the data. If the header indicates that the data has changed, invalidate the cache.
#### Content Delivery Networks (CDNs)
- Deploying managed caches across the network leads us to the idea of **content delivery networks (CDNs)**, which are sets of servers in the network serving content (e.g. HTTP resources).
- For good performance, we try to put CDNs close to end users. Here, close means geographically close, but also close from a network perspective (fewer hops).
- CDNs allow providers to scale their server infrastructure more easily. With a single origin server, we’d have to scale that server by making it incredibly powerful and giving it incredibly high bandwidth. By contrast, with CDNs, we can scale just by adding more small servers throughout the Internet.
- CDNs also provide better redundancy for providers. If a single origin server goes down, the service might become unavailable. By contrast, with CDNs, if one server goes down, users can still be redirected to other servers.
#### CDN Deployment
- The client’s request is forwarded through WAN routers (owned by the ISP) until it reaches a peering location. Then, the request goes to a peering location in the application provider’s network.
- The request goes through the application’s WAN networks until it reaches a datacenter network, where the origin server lives.
- If we don’t deploy any CDN, every request has to reach the origin server. This has the maximum latency, resulting in the lowest performance.
- Also, this requires the most bandwidth to be traversed, which means we have to build more bandwidth. Finally, this requires the origin server to scale to handle every request.
	![noCDN](https://textbook.cs168.io/assets/applications/4-21-cdn1.png)
- A better option would be to deploy some CDN servers at the edge of the application provider network. Now, the amount of bandwidth sent over the application provider’s network is much lower.
- The origin server sends the video to the CDN once, and the CDN can serve that video to many users. The application network no longer needs to scale its WAN network.
	![edgeCDN](https://textbook.cs168.io/assets/applications/4-22-cdn2.png)
- We can do even better and push caching deeper into the network. Now, the application is deploying servers inside the ISP’s network.
- Why would an ISP agree to let the application deploy a CDN in their network? It turns out this is mutually beneficial for everybody. The ISP’s customers will get better performance because they can use this new closer CDN.
- Also, the heavy traffic between users and the CDN is now all contained within the ISP’s network. This means that ISP needs less bandwidth in the peering connection between the ISP and the application (since the content is sent only once across that peering connection).
	![peerCDN](https://textbook.cs168.io/assets/applications/4-23-cdn3.png)
- We could try to go even further, but eventually, we encounter cost-benefit trade-offs. In the most extreme case, we could deploy a CDN in everybody’s home, but the cost probably outweighs the benefit.
- In particular, CDNs work best when multiple users are using it. The collective cache is larger, and one deployment can reach many users.
	![wanCDN](https://textbook.cs168.io/assets/applications/4-24-cdn4.png)
#### Directing Clients to Caches
- In a CDN, many different servers throughout the Internet are providing the same content. How does the client know which server to contact?
- Some of the tricks from DNS can also apply to CDNs. We could use [[0x25_DNS#Root Server Availability with Anycast|anycast]], where multiple servers advertise the same IP prefix. This allows the routing algorithm to find the best path to any one of the servers.
	![anycast|500](https://textbook.cs168.io/assets/applications/4-25-anycast1.png)
- One problem with anycast is with long-running connections. Suppose the client has an ongoing TCP connection with one of the servers. During the connection, some intermediate [[0x04_Links|link]] in the network fails.
- Since all the servers have the same IP address, from an intermediate router’s perspective, forwarding to any of the servers is valid. The intermediate router may now start forwarding packets to a different server (with the same IP address).
- However, the TCP connection was with the original server, and this new server has no way to continue the original connection.
- Note that this problem didn’t apply when we used anycast in DNS, because DNS connections are very short (usually just one UDP packet).
- We could also use [[0x25_DNS#DNS for Load Balancing|DNS to load-balance]]. Unlike in anycast, the servers now have different IP addresses, though they still all have the same domain. When the client queries for the domain-to-IP mapping, the DNS name server can provide a different IP address depending on the client’s location.
- This DNS-based approach doesn’t have the same problem with long-lived connections that anycast did, because the servers now have different addresses. The router won’t suddenly start forwarding packets to a different server.
- One problem with the DNS-based approach is lack of granularity. As an extreme example, suppose everybody in Comcast’s ISP used the same recursive resolver. This means that everybody sends their DNS queries to the resolver, who then makes the query to the application name server.
- The application name server can only see that the DNS request came from Comcast, and has to give a single IP address back to Comcast. Now, every user in Comcast’s network is using the same server, even if the users are all over the world.
	![dnsLoadBalance](https://textbook.cs168.io/assets/applications/4-27-dns-loadbalance.png)
- A more robust approach than anycast or DNS is application-level mapping. When the origin server receives an HTTP request, the links in the response can point to different servers (e.g. static1.google.com or static2.google.com, two servers in different places), depending on where the request came from.
- Or, the origin server can reply with an HTTP 300-level status code to redirect the user to the appropriate server.
- This application-level approach doesn’t have the granularity problem of DNS, because the application can see the client’s address in the HTTP request. This also doesn’t have the anycast problem, since different servers can have different IP addresses.
- However, just like in DNS load-balancing, the application still needs some way to guess the closest server to the client (where close might be geographic or based on network topology).
- One benefit of application-level mapping is additional granularity depending on the content. For example, popular videos can be deployed to lots of servers, allowing every client to get the video from a nearby server.
- By contrast, unpopular videos that are rarely accessed can be deployed to fewer servers, and require users to go further for the content.
# Sources
- [Lecture 17 - Applications 2: HTTP](https://www.youtube.com/watch?v=WDkTLnYGhA8).
- [HTTP](https://textbook.cs168.io/applications/http.html).