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

## Application Load Balancer (ALB)

- Elastic Load Balancer refers to family of three products
  - Classic Load Balancer (Legacy Load Balancer)
  - Application Load Balancer
  - Network Load Balancer
- ALB is a 'layer-7' LB - Understands HTTP/S
  - Layer-7 here means OSI layer 7 model
  - ALB is capable of inspecting data that passes through it - it can understand the application layer
  - This application layer can understand HTTP/S protocols
  - it therefore can take actions on things like paths, headers and hosts that are in HTTP/S protocols
- ALL AWS load balancers are highly available and scalable
  - The capacity of ALB balancers increases automatically as you put more load in it
  - it usually runs in multiple AZs
  - if it's configured correctly - it's highly available product
- Load balancers can be internet facing of internal
- Load balancers have a front end that the **Listening** side and it has a back-end that is connected to one or more servers called targets
- it's billed based on hourly rate and LCU rate (Load Balancer Capacity Units)
  - 1 LCU allows 25 new connections per second 
  - 3000 active connections per minute 
  - 1 GB per hour for EC2 instances containers and IP addresses as Targets
  - 0.4 GB per hour for Lambda function as Targets
  - 1000 rule evaluations per seconds
  - If you consume the highest values mentioned above that means you are consuming 1 LCU
  - so you will be billed for the hourly rate for the load balancer and in addition to that you will pay for the number of LCUs you consumed

**Architecture**

- LB has at least one node per AZ 
- Lets say we have two LB in two AZ 
- The LB will have a DNS name which will distribute traffic 50% each LB node in AZ
- if you have much higher network traffic to handle then you can also have multiple LB in each AZ
- The ALB can by default use **Cross Zone Load Balancing** to route traffic across the AZ if you have few instances to serve in one AZ than other
- ALB can also perform health checks - if all instances attached to ALB then they will all continue to receive traffic but if any health check fails then that instance status becomes unhealthy and no more connects are sent to that target
- Since the ALB operates on the layer 7 so it has the capability to thoroughly health check the application 
- ALB can distribute traffic to many target types including:
  - EC2 instances
  - Containers running inside ECS(Elastic Container Service)
  - Lambda Functions
- Targets(listeners) can be combined into a group - each group can have one or many targets(instances that can server the traffic)
- In ALB you can also define rules. These can be:
  - Path rules (HTTP Address)
    - with path rules, we can say divert traffic to group1 when users try to reach animals4life.org/cat and divert to target/listener group 2 in case of animals4life.org/dog
  - host rules (Multiple Domain names)
    - can be used if you have multiple domains on the same Load balancer
    - so we can have dogs.animal4life.org or we might have marketing1337.animals4life.org

**Exam PowerUps!**

- Targets are the listeners => EC2 Instances, Containers or Lambda Functions
- Target Groups => It's just a group with one or more than one Targets/Listeners
- Rules => Targets are addressed using Rules
  - Path Rules: These rules can be Path based rules where traffic is directed based on HTTP path name /dog or /cat
  - Host Rules: Can be used when you have multiple domain names such as dogs.animals4life.org

- ALB supports:
  - EC2
  - ECs
  - EKS
  - Lambda
  - HTTPS
  - HTTP/2
  - Websockets
- ALB can use SNI( Server Name Indication) for multiple SSL Certificates - Host Based rules
- AWS doe not recommend to use CLB (Classic Load Balancer) anymore - they only exists for legacy reasons
  - you can achieve better performance, low cost and more features using ALB compared to CLB

## Launch Configuration and Launch Template - EC2

- LC and LT at high level perform the same task
- They allow EC2 configuration to be defined in advance
- They are documents to define: 
  - AMI , Instance Type, Storage Key Pair
  - Networking & Security Groups
  - Userdata & IAM role
  - Both are not editable but with LT you can have versions
  - LT also provides features like T2/T3 Unlimited, Placement Groups, Capacity Reservations and Elastic Graphics
- AWS recommends to use LT instead of LC. LT has all the features you find in LC and more. 
- LC only has one use - it's used as part of Auto Scaling Groups. One you created it, you can't edit it.
- LT can also be used as part of Auto Scaling Group 
  - it can also be used to launch EC2 instances

## Auto Scaling Group

- It allows EC2 instances to scale automatically based on the configuration provided 
  - you can also do self-healing as part of auto scaling or in isolation 
- It's a great feature which is implemented using Launch Configuration or Launch Template
- you define the configuration in Launch Template(LT) or Launch Configuration(LC) so Auto Scaling Group know what and how to provision EC2 
- It has three key values 
  - Minimum Size
  - Desired Size
  - Maximum Size
- Auto scaling group can provision new instances or terminate existing ones to manage the desired level
- if your desired value is higher than existing EC2 instances then the new EC2 instances will automatically be provisioned to match the desired value, if its lower than actual no. of instances then instances are terminated to match the desired value
- The Desired value can be set manually or it can be a dynamic by using the scaling policies
  - increasing and decreasing desired value can be done using metrics - for example you can use CPU utilisation

**Architecture**

- Auto Scaling Groups runs in a VPC in one or more subnets
- configuration for EC2 instances provided by Launch Templates or Launch Configuration
- on the Auto Scaling Group we specify the minimum value - for Example 1. This means we will always have at least 1 EC2 instance
  - we can also set a desired value - lets say 2 so if the number of running instances is 1 then auto scaling group will try to increase the number of running instances to two
  - we can also set the maximum value - for example we can set it to 4 - this means maximum number of 4 instances could be provisioned but they won't be immediately provisioned as desired is still 2.
- we can set the desired value manually or we can use scaling policies to automatically update the desired value
- Architecturally Auto scaling groups defines where instances are launched
- they are linked to VPCs and subnets within that VPC are configured on the Auto Scaling Group
- Auto Scaling group will try to keep the instances even in all AZs - if your desired value is three then there ASG will try to have 1 instance in each AZ rather than all in 1 AZ
- Scaling Polices are rules that you define to adjust the values in an Auto Scaling Group. Three way you can manage Scaling Policies
  - Manual - no automation. Just manually adjust desired value
  - Scheduled - Perfect for Time based adjustment of desired value - such as Sales pariod
  - Dynamic - you define rules which will trigger a change in desired values 
    - Simple - it's based on metric - CPU above 50% => add 1 instance or CPU below 50% remove 1 Instance. You can also use Memory or Disk I/O as a metric. you can also use Cloud Watch Agent where you need OS level specific metrics e.g. length of SQSQ 
    - Stepped - you can here define more detailed rules. Allows you to react based on how extreme the metric is - for example you can add 1 EC2 instance if CPU is above 50% and 3 instances if CPU is above 80% - It's almost always preferred over simple scaling
    - Target Tracking - it lets you define an Ideal amount of something (metric). for example 40 % CPU in all the instances in the Auto Scaling Group. When you define this then the group will scale as required to stay at that level- For example if you want to maintain a specific response time or number of connections to a particular part of  the infrastructure per instance
- Cooldown Periods - This is a value in seconds and it controls how long to wait at the end of a scaling action before performing another. This helps to avoid paying for minimum cost associated with provisioning EC2 instance. 
  - if you keep adding and removing EC2 instancing it could cost you a lot. 
- **Self Healing** - Auto Scaling Group also monitors health of EC2 instances that they provision
  - if it detects an EC2 instance has failed then it will terminate it and create a brand new instance
  - you can make just one EC2 instance highly available by setting up an Auto Scaling Group with minimum 1 and Desired 1 value. This way if your EC2 instance fails then it will automatically terminate it and try to launch a new EC2 instance in the same AZ. If not in that AZ it can also launch it in another AZ if you have actuated your Auto Scaling Group to multiple Subnets in different AZ 
  - This is a simple and a cheap way to make your EC2 instance Highly available
- Auto Scaling Group are good on their own but are amazing if you use it with Load Balancers
  - you can increase or decrease the Target Group of Load balancer by attaching an Auto Scaling Group
  - his way if your application has more load then Auto Scaling Group can increase the number of targets in your Target Group
  - Another very powerful thing you can do is to use Load Balancer health check in your Auto Scaling group. Load balancer Health Check are much richer and provides more application related checks as it operates at layer 7 that has full access to HTTP/ HTTPS requests. You can use this rich health checks to auto scale your target group for load balancer
- Auto Scaling group are free
- you are only build for the resources created by the Auto Scaling Group
- you should always use cool down period to avoid rapid auto scaling
- try to use more smaller instances as it gives you more granularity
- it's always a good idea to use Application Load Balancer with Auto scaling Group for anything that's client facing as it will abstract away the the listeners from the client as they will be connecting via Load balancer
- Auto Scaling Group defines **WHEN**(e.g. CPU is above a certain amount) and **WHERE**(e.g. AZ ) you want to launch and **Launch Template** or **Launch Configuration** defines **WHAT**(e.g. NEtworking, security group, AMI etc) you want to launch

## Network Load Balancer (NLB)

- It's part of AWS version 2 series of load balancers
- NLB is a layer 4 load balancer - Only understands TCP and UDP
  - things like IP addresses, TPC/UDP protocols and ports
  - NLB comes with pros and cons
  - Cons - Can't understand HTTP/S 
  - Pros - Faster 100ms compared to 400ms for ALB
- EXAM - if it talks about latency and doesn't need HTTP/HTTPS then you should default to Network Load Balancer
- You can easily use NLB to load balance TCP at port 80 which is the HTTP but the NLB won't understand HTTP it will just balance the incoming load for TCP at 80 with the target groups(listeners).
  - even though it can't understand HTTP but it can balance the load amazingly fast
  - NLBs provides highest level of performance compared to other load balancers
  - it can deliver millions of requests per second
- EXAM: if performance is the priority then go with NLBs
- NLBs are the only load balancers that can be provided with static IP address
  - when NLBs are deployed in an AZ it gets a network interface card
  - this network interface gets an static IP address
  - you can also assign elastic ip address which makes them excellent for white listing on firewalls
- EXAM: While listening, IPs or static IP addresses in relation to load balancing then go with Network Load balancers
- NLBs can also handle SSL passthrough
- EXAM: it can load balance applications other than HTTP/S - as it doesn't care about anything that's above layer 4 - for example things like FTP  or things that aren't HTTP/HTTPS

## SSL Offload & Session Stickiness

**SSL Offload** 

- There are three ways a load balancer can handle secure connections:
  - **Bridging**
    - it's a default mode of an Application Load Balancer (ALB)
    - One or more clients makes one or more connections to a load balancer
    - The Load balancer is configured to listen HTTPS connections
    - this means the client and the Load balancer has SSL connection
    - The connection is decrypted at load balancer - this is also called terminating the connection at load balancer
    - This means that the Load Balancer needs an SSL certificate which should matches with the domain name that the application uses
    - This could also mean that the AWS has some level of access to that certificate
    - If you want to be highly sensitive of your security then you might have a problem with bridge mode, as AWS might  have access to your certificate
    - in other words if you are really sensitive in terms of where the certificate is stored then you might have a problem with this mode
    - once the connection is terminated at the ALB, the ALBE will then make seconds connection with the target compute resource such as EC2 instances
    - The termination of incoming client connection at ALB allows ALB to decrypt HTTPS which is just a secure wrapper of HTTP. This way the ALB can see what's inside HTTP and can take actions if needed/configured.
    - The ALB then makes another **secure** connection with the target compute resource(EC2) which means that EC2 should also have the certificate to decrypt the connection from ALB and take action. the EC2 should also have to have a matching certificate that should match with the domain name.
    - PROS: ALB can see unencrypted HTTP connection and can take action if needed/configures
    - CONS: the connections will need to be decrypted at ALB then encrypted again at ALB and then decrypted again at compute resources. All these operations take time and could mean slow response-
      - another con is that the certificate needs to be stored on ALB and that's a risk
      - EC2 instances also need a copy of that certificate which is an admin overhead
  - **Pass-through**
    - with this mode the NLB just passes through the secure connection to backend compute resource - it doesn't decrypt any connection
    - This method means your backend compute resource will need to have a copy of SSL certificate which should match the domain name
    - this mode has no exposure of the certificates to AWS like in bridging
    - The con is that the NLB has no visibility of the HTTP traffic so can't take any actions based on the content of HTTP traffic
    - the NLB here is configured to listen using TCP 
    - you can also use Cloud HSM appliance to make this architecture even more secure
  - **Offload**
    - in this mode the clients connect to ELB via HTTPS connection which is then decrypted at ELB
    - this means the ELB(elastic Load Balancer) will need the certificate to decrypt the traffic
    - ELB then passes the decrypted connection straight to the target compute resource without encrypting it again
    - this means from ELB to the target compute resource the connection is HTTP and it's not secure
    - From client perspective the connection remains secure but from ELB to target compute resource the connection is not secure - which mean no cryptographic operation is required at the target compute resource 
- Connection Stickiness
  - With no stickiness, client connections are distributed across all those backend instances that are in service
  - **No sticky Architecture** - if the application itself is able to manage the session/state of the user (probably using dynamodb) even if the connection is landing on multiple stateless backend server then that's okay otherwise the user might get logged off if the connection lands on new server
  - With Elastic Load balancers you can use Session Stickiness and in ALB it is enabled on a target group
    - With this feature enabled, when user creates a connection with the ALB, a cookie is created by AWS ALB for that user - This cookie can have a duration of 1 second to 7 days. This is something that you define
    -  Once the cookie is created the connections are sent to the same backend instance 
      - The ALB will continue to send connection for the user to same backend instance unless that instance fails  then the user will be moved to another instance or if the the cookie expires 
    - the problem with this architecture of session stickiness is that if a user continue to use a load of load via one backend server then it could imbalance the load as the same instance will also continue to get new connections
    - Ideally it's best to have the session stored on a separate server such as DynamoDb
      - this way the session/state is in an external server and then the backend instances will become stateless again and load can then be balances easily via load balancer

