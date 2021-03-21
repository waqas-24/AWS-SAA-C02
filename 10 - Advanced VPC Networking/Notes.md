# 10 Advanced VPC Networking

## VPC Flow Logs

- VPC flow Logs only captures packet metadata - they DO NOT capture packet content
  - To capture packet content  you need some sort of packet sniffer software you can install on EC2 instance
  - VPC flow logs can capture source and destination IP address, source and destination ports, packet size and externally visibly meta data
- To use VPC Flow Logs it needs to be applied at one of these places
  - Applied to VPC - All interfaces in that VPCs
  - Applied to subnet - will include all interfaces in that subnet
  - applied to a network interface - will capture all the packet meta data on that interface
- VPC Flow Logs aren't real time - there is always a delay
- VPC Flow Logs can use S3 or CloudWatch Logs as destinations
- If you apply VPC Flow Logs on VPC then every single interfaces inside subnets of the VPC will be captures - so there is one way inheritance - From VPC to Subnets and From Subnets to interfaces
- On each interface we have security group that is stateful - this means that if it allows certain traffic to go in then it also allows for that traffic to go out. This means Security Groups understand state.
- Network ACL on the other hand is state less. It evaluates traffic as initiation and response. This means if a certain traffic is accepted by Security group then it's not guaranteed that it will also be allowed to go out of that interface as Network ACL could reject it. 
- ICMP has protocol value 1 , TCP is 6 and UDP is 17 
- In VPC Flow Logs Source IP always comes first and then Target IP
- VPC Flow Logs does not capture every type of traffic. Traffic from instance meta data (169.254.169.254, DHCP, Amazon DNS server & Amazon Windows License are not recorded. 

## Egress-Only Internet Gateway

- It allows traffic to be allowed only from inside a VPC to outside world
- Just like NAT Gateway that allows private IP addresses to initiate a connection to public internet and get a response - for example, to update software
- Egress-Only Internet Gateway performs the same functionality but for IPv6. NAT Gateway only works for IPV4.
- If you have an EC2 instance that has IPv6, you can't use NAT Gateway to reject any connection initiation from public internet to your IPV6 instance inside VPC.
- Normal Internet Gateway will allow all IPV6 traffic to go out and come in
- Egress-Only Internet Gateway only allows traffic to be initiated from IPv6 enabled instances inside VPC and get a response back but it does not allow any device from internet to initiate a connection to ipv6 enabled EC2 instance.

## Gateway Endpoints (VPC Logical Objects)

- In order to access public services such as S3 or DynamoDB you usually need an Internet Gateway and public IPv6 or IPv4 inside VPC OR you need to implement one or more NAT gateways(which allows instances with private IP addresses to access public services)
- but what if you don't want your private VPC to have any sort of internet connection but still want to use public services such as S3 or DynamoDB
- you can use VPC Endpoints, a gateway that creates a tunnel for your private subnet to be able to communicate with public services like S3 DynamoDB 
  - in other words VPC Endpoints provides private access to S3 and DynamoDB
  - it allows any resources in private only VPC to access S3 or DynamoDB
  - it is created per service and per region
- When you add VPC/Gateway Endpoint to a subnet then a Prefix List is added to the route Table for those subnets called Gateway Endpoint. This prefix list uses Gateway Endpoint as a target
- Gateway Endpoint is like a normal entry in the route table but it's a object/Logical entity that represent services such as S3 or DynamoDB
- so prefix list is added to the route table and is used as the destination and the target is the gateway endpoint. 
- So the traffic that leaves the subnet goes to services like S3/DynamoDB via Gateway Endpoint rather than Internet Gateway
- Gateway Endpoint is not linked to any subnet or AZ itself - it's highly available across all AZ in the region by default 
- Like internet Gateway that's associated with a VPC, with Gateway Endpoint you just specify which subnets it needs to be linked and AWS will automatically update the routing table for that subnet with prefix list
- so it's configured on your behalf automatically once you specify the subnets it needs to be used with - Gateway Endpoint is a VPC gateway object
- With gateway Endpoints you can also configure endpoint policies which can control what can be accessed via gateway endpoint. For example if you can allow a particular subset of S3 buckets
- it's a regional service - so you can access buckets defines in UA from Australian region - both the service and Gateway endpoints have to be in the same region
- Use Case: if you have a private VPC and want to allow access to public service such as S3 in a super secure way without allowing any internet access to your private network then you can use Gateway Endpoint - You can make it even more secure by allowing only specific S3 bucket access via endpoint policies
- You can also set your S3 bucket to private only by allowing access only from a gateway endpoint - you can do this by editing the bucket policy to allow access via only gateway endpoint
  - If you allow only Gateway Endpoint access then Implicit deny will apply meaning no access will be  Interface Endpoints granted to anyone other than Gateway endpoint
- Limitation: only accessible from inside that specific VPC - they are logical gateway objects 

## Interface Endpoints (VPC Objects)

- Just like Gateway Endpoints, Interface Endpoints also provide private access to AWS Public Services
- They provide access to all AWS services except S3 and DynamoDB - For S3 and DynamoDB use Gateway Endpoints 
- **Key difference from Gateway Endpoint**: They are not highly available by default . They are added to specific subnets  - as you know one subnet is in one AZ  - so if that AZ fails then the interface endpoint also fails
- For HA, add one endpoint to each subnet per AZ 
  - IF you use two AZ then go with two endpoints for HA or if you use 3 AZ then you will need 3 interface endpoints
- Network Access controlled via Security Groups from networking perspective - this is something you can't do with Gateway Endpoints 
- Interface also comes with **Endpoint Policies** - so you can define what can be done with that endpoint 
- They only support TCP protocol and IPv4 Only
- behind the scenes Interface Endpoints uses **PrivateLink** which is a product that allows external services to be injected into your VPC (either from AWS or third parties)
- With Gateway Endpoint you use prefix list inside the routing table of your subnets but with interface Endpoints DNS service is used
- When you setup an interface endpoint for a specific service, the endpoint provides a new service endpoint DNS
- for example vpc-123-xyz.sns.useast-1.vpce.amazonfaws.com 
  - this names resolves to the private IP of the interface endpoint
  - if you update your application to use this endpoint specific DNS name, then you can use it to directly access the service via the interface endpoint 
- Interface Endpoints are given a number of DNS names:
  - Endpoint Regional DNS
  - Endpoint Zonal DNS
- Applications can use these endpoints or they can use **PrivateDNS** that overrides the default DNS for services

**EXAM**

- **Gateway Endpoints** work using prefix list and route tables - they never require changes to your application
  - They application thinks they are directly connecting to S3 or DynamoDB but they are in fact using Gateway Endpoint
  - This allows Private IP and private subnets to use public service without allows them internet access
  - Since they are VPC objects - they are highly available by design
- **Interface Endpoints** uses DNA and a private IP address for the interface endpoint 
  - you have the option to use endpoint specific DNS names or you can enable Private DNS which overrides the default which means you don't need to modify your application to access services using the interface endpoints 
  - they don't using routing - they use DNS
  - Because they use normal VPC interface endpoints they are not by default Highly Available 
  - you can make it HA if you are using multiple AZ then you can put these interface endpoints in all those AZs



## VPC Peering

- it allows you to create a direct encrypted link between two and only two VPCs
- it can done between same or different regions or same or different AWS accounts
- optional - When you enable VPC Peering, you can enable Public Hostnames to be resolved to private IPs
  - this means you can locate services whether they are in Peered VPCs or not
  - if this option is enabled and you attempt to resolve a public DNS hostname of an EC2 instance it will resolve to the private IP address of that EC2 instance
- If VPCs in the same regions then security Groups (SG)  can reference peer Security Groups ID
  - In different regions, you can still utilize security groups but you will need to reference IP addresses or IP ranges
- VPC Peering does NOT support transitive peering
  - if you have VPC A peered to VPC B and VPCB peered to VPC C then that does not mean that there is a connection between A and C
  - if you want to VPC A B and C to talk to each other then you will need **3 peers** - 
    - 1 Between VPC A and VPC B
    - 1 Between VPC B and VPC C
    - 1 Between VPC C and VPC A
- When you are creating a VPC peer, what you are actually doing is creating a logical VPC Object inside both of the VPCs - this means you need to configure routing  and SGs and NACLs can filter
- Once you have the logical gateway object inside each VPC the next step is to configure routing
- you need to configure routing tables within VPC and associate these with subnets
- these routing tables have the remote VPC CIDR and as the target the VPC Peering Connection or the gateway object that's created  when we create the VPC peering connection
  - this means that VPC router in VPC A will be able to send traffic destined for IP range of toward the VPC Peering Logical Gateway Object
  - this configuration will be needed at all subnets at both sides of all peering connections assuming we want to allow completely open communications 
- **EXAM** - the VPC CIDRs CANNOT overlap if you want to create VPC peering connections. VPC peering will only work if you have completely isolated VPCs and CIDRs are not overlapping
- communication is encrypted and transits over the AWS global network when using cross region peering rather than internet. 