# 11 Hybrid Environments and Migration

## Border Gateway Protocol (BGP) (Advanced)

- BGP as a system is made up of many self managing systems known as Autonomous Systems(AS)
  - AS could be a large network with lots of routers but for BGP it's a black box
- BGP is a routing protocol - which means its a protocol that controls the flow of data from A to B, B to C and then C to it's destination D.
- BGP allows you to communicate between Autonomous Systems (AS)
- Autonomous System Number(ASN) are unique and are allocated a number(16 bits in length) by IANA 
  - 0 to 65535 
  - from 64512 to65534 are private ASN Number
- BGP operates using TCP at port number179 - it's a reliable communication
- BGP does not peer automatically between AS - it has to be configured manually
- BGP is a path-vector protocol - it exchanges the best path to a destination between peers - the path is called ASPATH
  - BGP doesn't care bout link speed but uses best path available
- iBGP = Internal BGP - Routing within an AS
- eBGP = External BGP - Routing between AS's
- When BGB Peering is configured, the AS share shortest routing paths with each other and BGB route traffic with the shortest path regardless of the connection strength or speed
- But you as network administrator can influence the routing path between AS using **AS Path Prepending**
- With AS Path Prepending you can arrange the routing table in a way that routing takes place according to your design rather than BGP using it's own routing mechanism.
- For example of you know there is a weak internet link between two AS, then you can use AS Path Prepending to use that path only at the last resort and instead use traffic routing using more powerful routing path

## AWS Site-to-Site VPN

- AWS Site-to-Site provides a quick network connection between AWS and something that's not AWS(could be on-premise network, another cloud platform or a data centre)
- It's a logical connection between AWS and on-premise network encrypted using **IPSec** as it runs over the public internet
- It can fully Highly available if you design and implement it correctly
- It's quick to provision - can be setup and running within an hour
- The first thing you need for Site-to-Site VPN connection is a VPC and an on-premise network
- The next thing we need is **Virtual Private Gateway (VGW)** - this is another type of logical gateway network which can be the target on the route tables - it is something you create and associate with VPC and then could be the target of one or more route tables.
- Next we need **Customer Gateway (CGW)** - this could refer to two things:
  - logical piece of configuration within AWS
  - a physical on-premise router which the VPN connects to
- VPN Connection between the VGW and CGW

**Detail Setup Configuration**

- We need IP range of the VPC network that needs to connect to the on -premise network
- We also need the IP range of the on-premise network
- And the IP address of the physical router on the customer premises
- Once we have all this information, we can connect the Virtual Private Gateway (VPG) and attach it to our VPC
- Just like any other gateway object, this VPG can also be the target of the routes in route tables
- The physical router on the customer premises, we'll have public/external IP address
- For this on-premises router we'll create Customer Gateway(CGW) object within AWS
- This Customer Gateway(CGW) is again a logical entity that represents this on-premises Physical router 
- We now need to define the public/external IP on this Customer Gateway(CGW) so it matches the physical router 
- VPG is a highly available gateway object - just like Internet Gateway.
- Behind the scenes VPG has physical Endpoints - these are devices in multiple AZs each with Public IPv4 addresses - this means if one AZ fails the VPG will continue to function using another AZ
- This doesn't meant the whole thing is highly available - need to keep in mind
- Next we need to create VPN connection inside AWS - there are two types of VPNs - **Static** and Dynamic
- Once VPN is created, it needs to be linked to VPG (Virtual Private Gateway) - this means the VPN can use the endpoints that the VPG provides
- You will also need to specify the Customer Gateway(CGW) to use.
- Once you provide both VGW and CGW then two VPN tunnels are created 
  - One Tunnel between each end point between VGW 
  - and another between the on-premise physical router
- These tunnels are encrypted channel and that allows data to flow between the VPC and the Physical router
- As long as one of the tunnels are active the data can continue to flow
- In this example we have two endpoints in VGW that are connected to our customer gateway using two tunnels - this means it is highly available. If one of the endpoints in an AZ fails then another endpoint will continue to function
- Since this is a Static VPN so we have statically provide IP address details to the VPN connection
  - this means we have to tell AWS the IP addresses in use in the on-premise network 
  - and we have to configure the on-premise site so it know the IP range in use in the VPC 
  - This is to allow the traffic to flow via VPC router through VPG, over the tunnels to the on-premise network and back again
- This is partially highly available network with one point of failure - that is the router in the on-premise network. If this physical router fails then the whole VPN connection fails
  - This is still highly available from the AWS side but because we only have on physical router and of that fails then the whole VPN fails - this is why it can only be classed as partially highly available
- To make this VPN connection Fully Highly available, we can add another Physical Router in on-premise network
- Ideally this new physical router should be added in a new building
- Once you have added new Physical Router that means you now have another Customer Gateway
- Now you can create another VPN connection with again two endpoints in the same Virtual Private Gateway and link this with new Customer Gateway(CGW) using another pair of VPN tunnels
- Now this makes it highly available VPN connectivity 

**Static vs Dynamic VPN (BGP)**

- Dynamic VPN uses a protocol called Border Gateway Protocol(BGP). If your physical on-premise  router doesn't support BGP then you can't use dynamic VPNs
- The architecture for both Static and Dynamic is the same - but with Static you have to provide IP addresses of the networks involved both on AWS and on-premises - this makes it simple and it works everywhere using **IPSec** 
- with Dynamic, it uses protocol called BGP. This protocol lets router exchange networking/routing information - so the on-premises and AWS networks can learn the networks themselves - you won't need to provide network details manually at either side of network
- If you like you can still manually provide routing information in the route tables of the VPC or if you like you can make the whole solution fully dynamic using by enabling a feature called **Route Propagation** on the route tables in the VPC
- While the VPN is active, and the route propagation is enabled, any networks these VPNS become aware of, meaning the on-premises networks in this example, are automatically added as dynamically learned routes in the VPC route tables - this means we don't have to statically enter route details. route propagation takes care of it automatically with the help of VPG when the VPN is active

**VPN Considerations**

- Speed Limitations ~ 1.25 Gbps cap for VPN - this is an AWS limit
  - one VPN with two tunnels has max throughput of 1.25 gigabits per second
  - you should also check the speed supported by on-premise network router
  - because VPN is encrypted to there is processing overhead of encrypting and decrypting data 
  - at high speeds this overhead can be significant
  - there is also a cap for the Virtual Private gateway (VPG) as a whole - so all VPNs connecting to VPG can only have 1.25 Gbps
- Latency Considerations - inconsistent as it uses public internet. it also depends on internet connection quality and hops between on-premise and AWS VPN endpoints. each hop adds latency and variability
- Cost - AWS hourly cost - GB out cost, Data cap (on-premises)
- Benefit of VPN - quick to setup. Within hours everything can be configured and up and running as most of this stuff is software configuration and IPSec is supported on pretty much all hardware but if you use dynamic VPN then you will need BGP supported which is much less common
- it can also be used as a backup for Direct Connect (DX) which is direct physical connection to AWS. 
  - you can use DX as primary and if anything goes wrong then as a back you can use VPN
- You can use VPN with Direct Connection(DX) - you can start with VPN and then once DX is provisioned then move to DX or run both together for High Availability
- you can run VPN on top of DX to add a layer of encryption

## Direct Connect (DX)

- DX is a 1 or 10 Gbps Network Port into AWS
- The port is made available to you at a DX Location (1000-Base-LX or 10GBASE-LR) Major Data Centre - these are located Globally
  - 1000-Base-LX for 1 Gbps and 10GBASE-LR for 10 Gbps
  - Both use single mode fibre optic cable 
- once you got the port from AWS you also need a router inside the DX location that will connect with the Port. 
  - **Customer Router** - If you are using your own router then it requires VLANS/BGP
  - **Partner Router** - You can also contact big telecom companies like BT to provide a router at DX location for you. 
- If you are a large corporation then you could also arrange for a physical cable connection from this router in DX to your premises - again you will need this via companies like BT
- The physical cable you need to be extended from the Router in DX to your on-premises location could take weeks/Months for Physical Cable installation - especially if it has to cover a large distance
- On top of this DX connection you can run Virtual Interfaces (VIFs)
  - VIFs are just virtual connections run on top of the physical cable
  - one physical cable can have multiple VIFs
  - Each of these VIFs is a VLAN and a BGP connection b/w your on-premises router and the router in DX location
- VIFs come in two different types 
  - **Private VIFs** - Associated with Virtual Private Gateway and connect to single VPC - you can have as many private VIFs as you like but each connect to one VPC
  - **Public VIFs** - These provide connectivity to AWS public zone services, like S3 and DynamoDB, SNS, SQS, and all of the other public AWS services - NOT the public internet

- DX By default doesn't provide any built-in encryption - you have to arrange encryption yourself

**Considerations**

- Takes much longer to provision DX vs VPN
- DX Port provisioning is quick but connecting that DX port with Customer/Partner Router via cross-connect takes time - this is a manual work 
- The extension of physical cable from DX location to your on-premises can take weeks/months - depending on the distance and telecom company
- Generally, you can setup VPN first and then replace it later with DX (or use both and leave VPN as  backup)
- DX has much higher bandwidth than site-to-site VPN
  - up to 40 Gbps with aggregation - that's like 4 10 Gbps
  - VPN is limited to 1.25 Gbps per second - both for connection and for all VPN connections connected via Virtual Private Gateway
- DX provides consistently low latency as it doesn't need to use the internet crossing multiple hops and doesn't use your internet bandwidth
- DX does not provide any built-in encryption - it's a completely unencrypted connection - this applies to both Public and Private VIFs
- There is a workaround to setup encryption
  - you can use Virtual Private Gateway that creates endpoints in AWS Public Zone and then create a VPN and use Public VIF running via DX
  - So basically rather than using public internet for VPN, you can use Public VIF
  - This way you will get IPSEC encryption as you are using VPN and you can use both VPC and AWS Public Services for this connection

## Direct Connect (DX) Resilience and HA

- Points of failures
  - Entire DX Location could fail - 
  - DX Router
  - Cross Connect
  - Customer DX Router
  - Extension
  - Customer Premises
  - Customer Router
- You can add resilience by provisioning two ports at DX location rather than one Port and then use two physical cable extension, two routers at DX Location and on premises 
  - But we still have points of failures:
    - DX Location
    - Customer Premises
    - Extension cable from DX to on-premises location - as both cable will be using the same path to your premises and if one fails there is a high chance that the other would also fail
- A better resilient architecture would be to use 2 separate DX location and two separate on-premises location
- To provide extreme levels of resilience,  you should have two separate DX location and 2 separate Customer premises but rather than using single port at each DX location, you should use two separate ports at each DX locations - this means you will end up with 4 ports in total and 4 separate physical cable link going into your premises and 4 separate routers 

## Transit Gateway

- It's Network Transit Hub that allows you to connect VPCs to each other and to on-premises networks via Site-to-Site VPNs and Direct Connect
- it is designed to reduce complexity of networks within AWS
- It's another network Gateway Object - Highly Available and Scalable by default
- The architecture is that you create attachments that allows you to connect two networks. Currently you can attach it to VPC, Site-To-Site VPN and Direct Connect Gateway
- You can use VPC peering to connect to VPCs and then Site to Site VPN with on premises but the problem is that VPC peering is not transitive which means if you have 4 VPCs then you will end up with 6 VPC peering if you want all VPCs to talk to each other and Site-to-Site- VPN connects only 1 VPC at a time so you can imagine if you want to connect all VPCs you will end up with a lot of tunnels
- With Transit Gateway  you can attach it with all the VPCs and it has Transitive network property so all VPCs attached will be able to talk to each other plus you can connect the same Transit Gateway with ON-premises network with just two tunnels via Site to Site VPN and with Direct Connect

**Consideration**

- Supports Transitive Routing - as long as you setup touring tables properly
- Can be used to create global networks via peering multiple Transit Gateway 
- Can share Transit Gateways between different AWS Accounts via AWS RAM(Resource Access Manager)
- Peer Transit Gateway with different Regions - same or Different Accounts
- less complexity vs without Transit Gateway

## Gateway Storage

Extension - Migration - Backups

- it allows you to extend file or block storage into AWS
  - it's like a portal in your on-premises which you can use to store data into AWS
- It also allows you to keep your data locally but also replicate it into AWS asynchronously - helps you recover in case of a disaster
- you can backup your existing data stored in Tape into AWS - so rather than using Tape to store data you can instead use AWS 
  - AWS can also emulate as table architecture to backup storage systems
- It helps you with Migration of existing infrastructure to AWS

**Gateway Storage Modes**

- **Tape Gateway (VTL) Mode**
  - VTL = Virtual Tape Library 
  - To backup servers it looks just like a tape drive library and a tape shelf 
  - Behind the scenes it stores virtual tapes on S3 - When archived it moved to Glacier 
  - but your backup server doesn't know the difference - it thinks it's interacting with physical tape infrastructure
  - Uses HTTPS to transfer data from on-premises to AWS S3 for active tapes archived tapes stored in VTS in Glacier 
  - IT Pretends to be a iSCSI tape library , changer and drive
  - Virtual tapes can range from 100 GiB to 5 TiB and total of 1 PB can be configured locally and unlimited number of tables can be archived in Glacier
  - Great for moving from an existing backup systems to AWS 
- **File Mode** 
  - SMB and NFS
  - lets you create File Shares
  - behind the scenes it uses S3 to store files 
  - great way to extend your file storage
  - Uses HTTPS to store the files in S3 where lifecycle polices can be applied to control storage classes
  - SMB Shares can integrate with active directory for file authorization
- **Volume Mode** 
  - Gateway Cache/Stored - iSCSI
  - Block storage backed by S3 and EBS Snapshots
  - **Gateway Stored** 
    - you get 16 TB per volume - 32 Volumes MAX - total 512 TB storage
    - this storage is available locally on premises and are made available via iSCSI for network based servers. These are backed up asynchronously  in AWS and then EBS snapshots are created from the backup 
    - you can use these EBS snapshots to create volume and attach it to EC2 instance. 
  - **Gateway Cached**
    - designed for extensions into AWS, when you have limited capacity and you are looking to decommission your existing infrastructure 
    - in this mode you are not storing data locally but are only caching data that's needed frequently 
    - you still have local volumes that are made available via iSCSI for network based servers
    - Primary data is stored on S3-backed volume(AWS managed Bucket) - snapshots are stored as standard EBS Snapshots
    - storage : 32 TB per volume - 32 Volumes max and 1 PB total capacity
- Use cases
- Tape Gateway - Replace Tape Backup System
- File Mode - If you need to store files and would like to transfer those to S3 
- Volume Mode - Gateway Stored - if you want to keep the storage locally but asynchronously would like to back up to AWS 
- Volume Mode - Cached Stored - extend into AWS but minimise local infrastructure footprint 

## Snowball / Edge / Snowmobile

- Move large amounts of data IN and OUT of AWS
- Physical storage - briefcase or truck

**Snowball**

- order from AWS, Log a job, Device delivered - not instant
- 10 TB to 10 PB Economical range (multiple devices to potentially multiple premises)
- Storage only - NO compute

**Snowball Edge** - Storage and Compute

- larger capacity vs Snowball
- Storage Optimised - 80 TB - 24 vCPU - 32 GiB RAM, 1 TB SSD( to run EC2 instance)
- Compute Optimised - 100 TB + 7.86 GB NVME, 52 vCPU and 208 GiB RAM
- Compute with GPU - as above with a GPU
- EXAM: Ideal for remote sites or where data processing on ingestion is needed

**Snowmobile**

- portable Data centre within a shipping container on a truck
- Special order - not available everywhere - only 1 truck delivered to 1 location
- ideal for single location when 10 PB+ is required
- up to 100PB per snowmobile
- not economical for multi-site(unless huge) or less than 10 PB

## Directory Service

**Simple AD Mode**

	- Open Source(Samba 4)  with limited functionality compared to Microsoft AD
	- 500 users for small
	- 5000 users for large scale
	- can't integrate with MS AD
	- Workspaces can use it for login management

**Microsoft Active Directory Mode**

Complete MS AD Service with HA and managed by AWS

you can create a trust relationship with on-premises MS AD via VPN. if VPN fails it can continue function 

**AD connector Mode**

it's just a proxy to actual AD in on-premises used via VPN

you use it when you don't want to have your own AD inside AWS as you have one on-premises

allows AWS services that need a directory 

if the VPN connection fails then AD won't be available

**When to pick which mode?**

**Simple AD** - The default . When you have simple requirements then this mode should be good to use in AWS

- when you don't need any connectivity with on-premises AD 

**Microsoft AD** - you use this when you have applications in AWS that need MS AD or you need to trust MS AD in on-premises via VPN connection

**AD Connector** -  Use AWS services that need a directory without storing any directory info in the cloud - proxy to your on-premises directory

## DataSync

- allows you to transfer data from and to AWS
- you can set the bandwidth for the data transfer
- DataSync Agent runs on VMWare and communicated with AWS DataSync Endpoint
- You can incremental or schedule data transfer, during/avoiding certain time period
- allows compression, encryption, data validation, automatic recovery from transit errors
- you can sync data with S3 Storage Classes, Elastic File System (EFS) and FSx for Windows Server

**TASK** - What is copied from where to which location

**AGENT** - Software used to read write to on-premises data stores via NFS or SMB

**LOCATION** - From/to NFS/SMB to/from S3, EFS, FSx, S3 

**When to use DataSync**

Data Transfer electronically - IN and OUT AWS

Supports Schedule

Bandwidth Throttling

Automatic Retries

compression

cope with huge scale transfers involving multiple AWS products such as S3, EFS or FSx windows server

and traditional file transfer protocols like NFS and SMB

## FSx for Windows Servers

- FSx for Windows is a fully managed single or Multi-AZ(within a VPC) native windows file server - it can be integrated with AD

  - Can integrate with on-premises AD as well as AWS managed AD 

- it's like EFS but that's dedicated to Linux based instances only

- FSx for Windows file server is designed for integration with Windows environments

- it uses Elastic Network interfaces inside the VPC

- if you use single AZ mode - it uses replication within that AZ for resilience

- can perform on-demand and Scheduled Backups - includes both client and AWS side backups

- Accessible via VPC, Peering VPC, VPN, Direct Connect

- EXAM: Look for windows related keywords like native windows file system, Active directory or Directory Service integration. Understand the difference between EFS and FSx as they are both network shared file systems

- once setup it will be shared using normal double back slash shared location for example 

  ```
  \\fs-xxx123.animal4life.org\catpics
  ```

  once setup, workspaces can also use this location

- it's a native windows file system that supports:
  - de-duplication(sub file)
  - Distributed File System (DFS)
  - KMS at-rest encryption
  - enforced encryption in-transit
- The shares are accessed via SMB protocol which is standard in Windows environments and it also supports file level versioning (Volume Shadow Copies)
- FSx is highly performant: 
  - 8 MBps to 2 GBps 
  - 100,000s IOPS
  - <1 ms latency

**Key features**

- VSS - User-Driven restores 
- Native windows file system accessible over SMB 
  - EFS uses NFS protocol only uses Linux instances
- provides Windows Permission Model
- it supports DFS (Distributed File System) - scale-out file share structure
- Managed - no file server administration required
- can integrates with AWS Directory Service and your own AD directory

## FSx for Lustre

- designed for High Performance computing workloads 
- FSx for Windows is a windows native file system, accessed via SMB. it is used for windows native environments 
- Managed Lustre - Designed for High Performance Computing - Linux clients supports (POSIX file system permissions)
- USE CASES: 
  - Machine Learning
  - Big Data
  - Financial Modelling
- Scale to 100 GB/s throughput and sub millisecond latency
- Deployments in two modes
  - Scratch - Highly optimised for short terms - no replication but super fast
  - Persistent - longer term, HA (**in one AZ**), self healing
- Accessible: VPC, VPN or Direct Connect from on-premises location - you need to have high bandwidth to fully use lustre
- It can use S3 as a repository to store file - when the file is first access it gets lazy loaded from S3 into the file system. Once you make changes to data, it can then be exported back to S3 at any point using **hsm_archive**
- Lustre store different components of the data in the file system,
  - MetaData - File names, timestamps, permissions
    - These are stored in Metadata Targets (MDTs)
    - Lustre File system has one of MDTs
  - Data
    - The data is split over multiple storage object targets (OST) (each one of OST = 1.17 TiB in size)
    - by splitting the data between these OST, the level of performance for Lustre is achieved
- Performance provided by the file system is the baseline performance based on size of the file system
- Size Minimum 1.2 TiB then increments of 2.4 TiB
- Scratch - Base  200 MB/s per TiB of storage
- Persistent offers 50 MB/s. 100 MB/s and 200 MB/s per TiB of storage
- Both Types - Burst up to 1,300 MB/s per TiB (Credit System)
  - you earn credit when you are using product below baseline performance  and you used these credits when you need to use Lustre above baseline performance
- Once deployed you used Dist throughput and IOPS for read and write but for frequently accessed data, you use in-memory cache which is dependent on network throughput and IOPS

**Key Points**

- Scratch is designed for pure performance
- Short terms or temp workloads
- No HA or replication 
- if you use larger file system for scratch that means more servers, more disks and more chance of failure
- Persistent mode has replication but within on AZ only
- it auto heals when hardware fails
- You can backup to S3 with both manual and automatic mode(0 to 35 day retention)











