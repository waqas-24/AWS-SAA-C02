## 07 High Availability and Scaling



## Load Balancing Fundamentals

#### Applications without Load Balancer

- If we have just one web server serving all the user base, then it's not ideal. If that server fails then no user will be able get access resulting in no service
- If we have lets say 3 servers instead, then we can use Route53 Simple of Multi-value routing policies to distribute our load to multiple servers
  - This is better then single server but still not great. If one of the server fails then the traffic will still route to the same server - if we use health checks then that server will be removed from routing traffic to that specific location but again this isn't good as adding and removing servers to DNS is slow and clunky as there is TTL and caching issue with DNS
  - DNA is great when you are distributing traffic globally at a high level across multi locations but not great to handle traffic at application level (local resilience and scaling)

#### With Load Balancers

- The user connects to the load Balancer that is also called the listener. 
  - The listener generally use HTTP or HTTPS 80 or 443 ports to listen to the traffic if it's web traffic
- The listener can be connected to one or many servers.
- The user has no idea which server he/she is connecting to - Servers are now an abstract to the user
- As long as one of the servers are running the listener will continue to function
- Listener or the load balancer also health checks the server., if any of the server fails the load balancer will stop sending requests to that application
- Load balancer only fails when all the servers connect to it fail

**Exam PowerUp!**

- Clients/User connect to the Load balancer
  - Specifically they connect to the Listener of the Load Balancer(LB)
- The LB connects on your behalf to 1 or more targets(servers)
- When you have a load balancer you generally have at least two servers connected to it
- With LB, you generally have two connections rather than one. One with client and LB(listener) and another from LB to the server(backend)
- it's the LB job to abstract away client from the individual servers or number of servers or size of server or the performance of servers
- LB are used to achieve High availability, fault tolerance and scaling





