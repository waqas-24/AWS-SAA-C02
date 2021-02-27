# AWS-SAA-C02

These are my personal notes prepared for AWS Solutions Architect Associate(SAA) exam. All these notes are prepared using Adrian Cantrill's (SAA-C02) course and from [alozano-77](https://github.com/alozano-77)'s notes for the same SAA-C02 course. Learning Aids from [aws-sa-associate-saac02](https://github.com/acantril/aws-sa-associate-saac02). I have tried to make these notes as detailed as possible but there may be errors, so please purchase Adrian 's course to get the original and most up to date content https://learn.cantrill.io.

**Table of Contents**

[01 Elastic Compute Cloud - Basics](#01-Elastic-Compute-Cloud---Basics)

[02 Introduction to Containers](#02-Introduction-to-Containers)

[03 Advanced EC2](#03-Advanced-EC2)

[04 Domain Name Service (DNS) Fundamentals](#04-Domain-Name-Service-DNS-Fundamentals)

[05 Relational Database Service (RDS)](#05-Database-Refresher)

[06 Network Storage](#06-Network-Storage)

[07 High Availability and Scaling](#07-High-Availability-and-Scaling)

## 01 Elastic Compute Cloud - Basics

## Elastic Block Store EBS Service Architecture 


- EBS provides block ***storage*** in the form of volume – these volumes can be encrypted via KMS

- Block storage is block addressable storage mechanism that uses block IDs to read/write data

- When you attach a volume to EC2 instance, they just see a block storage that can be used to create a file system on top of it

- You generally create an EBS volume and attach it to one EC2 instance using storage network

- These volumes can also be detached and reattached to other EC2 instance.

- EBS are not linked to EC2 instance lifecycle - If EC2 instance moves host, then EBS follows it. If EC2 is stopped or suspended EBS is maintained. It is a persistent storage until you delete it.  

- Cluster setting - Some storage types allow multi attach – allowing volumes to attached to multiple EC2 instances

- You can provision EBS volume based on different physical storage types(SSD,HDD) and performance profile

- You are billed GB per month

- It’s an AZ based service

- You can not attach an EBS volume to an EC2 instance not in the same AZ

- One or more than one EBS can be attached to an EC2 instance

- EBS replicates data within AZ to provide resilience

- If the entire AZ fails then the EBS also fails even with resilience

- You can provide more resilience by taking volume snapshot and storing it in S3 which is then replicated within all AZs in that region

- This also allows you to create an EBS volume using this snapshot in another AZ in that region or another region. 

  ### EBS Volume Types - General Purpose

- EBS Two general purpose Volume types - **GP2** and **GP3** - both are based on SSDs

- GP2 is the default general purpose SSD-based storage provided by EBS while GP3 is fairly new and might appear in exam

- When you create GP2 it can be as small as **1 TB** or as big as **16 TB** and it is created with an IO allocation. 

  #### GP2 IOPS

- IO is one Input/output operation and in one IO you can read or write **16KB** data in one second

  - This means 1 IO credit gives you 16 KB read/write
  - If you are transferring 160 KB file in one second, that means you need 10 IO in that one second 
  - You get 5.4 million IOPS credits when you provision a GP2 volume
  - You also get baseline 100 IOPS credits per second regardless of the size of volume
  - On top of baseline you also get **3 IOPS per GB per Second** when you provision above 33.33 GB
  - This means 100 GB volume gets you total 300 IO credits per second
  - You only get baseline 100 IOPS if size of volume is below **33.33 GiB**

  #### Consuming IOPS

- **Burst rate **by default GP2 can burst up to 3000 IO of 16 KB per second

- These IOPS architecture is for volumes that are up to one TB in size

- For volumes larger than *one TB* will have equal to or above burst rate of 3000 IOPS

  - They will always achieve this 3000 burst rate as their baseline - they don't use credit system
  - The maximum IOPS for GP2 is ***16000 IOPS*** 
  - Any volume above 5.33 TB in size achieve this 16000 IOPS maximum rate constantly

#### GP2 is good for

- Boot volumes
- low latency interactive apps
- dev and test
- Note - You can also use an elastic feature to chance from GP2 to other type of volumes

### GP3 Volume

- GP3 is also SSD based but doe not use Credit architecture with something much simpler
- Every GP3 volume regardless of size, starts with standard 3000 IOPS 16KB operations per second
- it can transfer 125 MB per second as standard
- it can range from 1 to 16 TB
- right now it's 30% cheaper than GP2
- For extra money you can get 16,000 IOPS or 1,000 MiB/s
- Even with extra money for more performance, it will be cheaper than GP2.
- GP2 only offers maximum 250 MB per second whereas with GP2 you can get up to **1,000 MB per second**. 
- GP3 is simpler to understand and potentially will replace GP2. Though GP2 is still the default. 
- GP2 is used for:
  - virtual desktops 
  - medium sized single instance database such as MSSQL Server or oracle DB
  - low-latency interactive apps
  - dev & test
  - boot volumes

## Provisioned IOPS - io1 and io2

- There are three types of Provisioned IOPS SSD - Two in general release io1 and io2. one is in preview called io2 Block Express

- They all offer different performance with different prices but the common factor is that the IOPS can be configured independent of the volume size and they are designed for super high performance situations 

- These are for low latency and consistent low latency are important characteristics
- you pay for the size of the volume and the provisioned IOPS that you need

- with io1 and io2 you can achieve **64,000 IOPS per volume** - that's 4 times the maximum for GP2 and GP3

- with io1 and io2 you can achieve 1,000 MB/s of throughput - same as GP3

- with io2 block express you can get up to 256,000 IOPS per volume and 4,000 MB/s

- Size

  - io1 and io2 - 4 TB to 16 TB
  - io2 block express - up to 64 TB

- You can configure IOPS regardless of volume size but these constrains apply

  - io1 50 IOPS/GB - MAX
  - 102 500 IOPS/GB - MAX
  - block express 1000 IOPS/GB - MAX

- EC2 per Instance performance

  - In addition to above IOPS constrains, there is a level of maximum performance you can achieve between EC2 and EBS
  - the performance of instance depends on the type and size of the instance
  - with **io1** - you can achieve maximum 260,000 IOPS and 7,500 MB/s per instance
  - this means you need just over 4 volumes performing at their maximum performance to achieve this maximum performance per instance
  - with **io2** - the maximum you can achieve is 160,000 IOPS and 4,750 MB/s per instance
  - with **io2 Block Express** you can maximise instance with 260,000 IOPS and 7500 MB/s
- GP2 and GP3 per instance max is 260,000 IOPS and 7,000 MB/s  

#### Provisioned IOPS SSD use case

- High performance

- latency sensitive workloads

- I/O-intensive NoSQL

- Relational Database

- when you have smaller volumes but super high performance

  ## HDD Based EBS Volumes

- There are two key HDD based volume types - **st1 (Throughput optimised) sc1 (Cold HDD )**

- HDD based are good with sequential data read/write 

- it's good for throughout and economy than IOPS or extreme levels of performance

- **st1 Stats**

  - Size 125GB to 16 TB  
  - IOPS  500 IOPS 1 MB =  Max 500 MB/s

- **st1 Performance** 

  - 40 MB/s/TB Base
  - 250MB/s/TB Burst

- st1 is designed when cost is a concern but you need frequent access storage for throughput intensive sequential workloads

- **st1 used for**

  - Big data
  - data warehouse
  - log processing

- sc1 is designed for infrequent access and geared for maximum economy as it is the lowest cost EBS HDD volume 

- **sc1 Stats**

  - Size 125GB to 16 TB  
  - IOPS  250 IOPS 1 MB =  Max 250 MB/s

- **sc1 Performance** 

  - 12 MB/s/TB Base
  - 80 MB/s/TB Burst

- **sc1 used for**

  - Colder data requiring fewer scans per day

## Instance Store Volumes - Architecture

- Instance Store volume provide Block Storage - this means raw volume that can be attached to an instance which will then be formatted using the OS of the instance to be usable
- They are just like EBS volume but local instead of over the network volumes discussed above
- They are physically attached to one EC2 host
- Each EC2 host has its own Instance Store volume and these volumes are isolated to that particular EC2 host
- Instances on that EC2 host can access those volumes
- since they are locally attached, they provide the highest level of storage performance
- They are included in instance price - different instance types come with different instance store volumes
- **EXAM** - Unlike EBS volumes, you have to attach these volumes at launch time
- Depending on the instance type, you are going to be allocated certain number of instance store volumes. You can chose to use them or not but you can't adjust this later
- **Risk** - These instance store volumes are temporary - if your instance is stopped and started again then it would be transferred to another EC2 host where it will again be connected to a new ephemeral instance store volume but the  data will be lost as this is a new EC2 host.
- Another risk is that if any instance store volume fails the data will be lost. These volumes are only temporary - they should not be used where persistence is required
- The size and number of volumes vary depending on the size and type of instance 
- Some instance type don't support instance store volumes
- The key benefit of instance store is performance
  - D3 = 4.6 GB/s throughput
  - i3 = 16 GB/s of sequential throughput
  - They have more IOPS and throughput vs EBS

### Exam Powerup

- Instance store are local to EC2 host 
- you only add it at instance launch time
- Data is lost on instance move, resize or hardware failure
- high performance
- The price for instance store is included in instance
- This is a **Temporary** storage only 

## Choosing between the EC2 Instance Store and EBS

#### When to use EBS

- Highly available and Reliable storage
- Persistent storage that's independent of the ES2 instance
- **Cluster** - multi-attach feature in io1 to allow your volume to be attached to multiple instances
- Region Resilient backups - can't do this with with Instance store volume in an automated way.
- **Performance Caps EBS** 
  - If you require up to 60,000 IOPS and 1,000 MiB/s per **volume**
  - OR
  - 80,000 IOPS and 2,375 MB/s per **instance** 

#### When to use Instance Store

- Value - included in instance cost. Generally the larger the size of an instance, generally come with more volumes or better performing volumes 
- If you don't care about storage being permanent or highly available, then instance store represent great value as they come for free with the prince of an instance
- **Performance**
  - More than 80,000 IOPS and 2,375 MB/s
- Temp Storage volume
  - Caching
  - areas used to store temporary manipulation of data
- Stateless services - where server holds nothing of value 
  - like web server or other application server that don't themselves store any accounts or any sessions and you just need access to temporary space 
- rigid link storage and instance 
  - When you need data to be removed as soon as instance is gone
- Key words to remember Instance store volumes
  - Temporary
  - performance
  - movement between instances
  - failover

## Snapshot, Restore & Fast Snapshot Restore (FSR)

- Snapshots are an efficient way to backup your EBS volume to S3
  - This is to protect your volume from Availability Zone issues or local storage system failure
- Snapshots are also useful for migrating volume data from one AZ to another using S3
- EBS volumes have AZ resilience only - means if the entire AZ fails your EBS volumes will be impacted 
- Since snapshots are stored in S3 to your volume data becomes region resilient as S3 replicates data between available AZs in that region
- The first time snapshots only copy full data stored on EBS volume
  - if you have 40GB EBS volume and you are occupying only 10 GB then - the first snapshot will only copy 10 GB data
  - When snapshot copy the data, your EBS performance won't be impacted during this initial snapshot - but it takes time in the background 
- Future snapshots are incremental, consumes less space and quicker to perform
- EBS volumes can be created or restored from snapshots
- Snapshots can also be copied to another AWS region
  - You can use this for global DR processes or as a migration tool 

#### Key things to remember about snapshots

- When you create a new EBS volume without using snapshot - the performance is available immediately
- If you restore volume via snapshot, it restores lazily
  - if you restore snapshot via S3 it takes some time, but if you start requesting data immediately while the restore is in process, it will immediately fetch data from S3 which achieves lower level of performance compared to reading from EBS directly
- Rapid initialisation
  - You can force a read of every block of the volume and this is done in the operating system using tools such as DD on Linux
  - This forces EBS to pull all the snapshot data from S3 into that volume 
  - This is something you would do immediately when you restore the volume before moving that volume to production use
- To avoid the admin overhead of using OS toolset, you can use Fast Snapshot Restore(FSR) which does the same thing and instantly restores a volume
- You can create 50 FSR per region - For each FSR, you have to pick the snapshot and the AZ you want to do restore to
- FSR costs extra - can get really expensive if you have a lot of snapshots
- You can save this money by forcing a read of every block manually using DD in Linux or other tool in the OS
- But to save the admin overhead, then go for FSR

**Snapshot Consumption and Billing**

- You are billed GB-month for snapshots
- 20 GB stored for half a month, represents 10 GB-month.
- Remember snapshots only copy used data on the volume NOT allocated storage on the Volume. 
- The data is incrementally stored which means doing a snapshot every 5 minutes will not necessarily increase the charge as opposed to doing one every hour.

## EBS Encryption

Unless the OS of EC2 instance is doing disk encryption, by default there is no encryption applied to EBS volumes

EBS encryption can be turned on and it is applied to the volumes and snapshots at rest

When you first create an EBS volume, EBS uses KMS and a Customer Master Key - This key could be the default AWS managed Customer Master Key called (AWS/EBS)  or it could be Customer Managed Key (CMK)  

The CMK creates an encrypted Data Encryption key (DEK) that is stored with the volume on the physical disk. 

This `DEK` can only be decrypted using KMS - assuming the entity trying to decrypt it has the permission to use KMS

When EC2 instance is launched and you pick encryption, the KMS is requested to decrypt `DEK` and the decrypted key is stored on EC2 host in memory while it's been used. 

At all other times it's stored in encrypted form on the EBS volume

Every time the data is read or written to the volume, this decrypted `DEK` is used and at all times the data in the volume remains encrypted at rest 

This means the data at rest is stored as **Ciphertext**

If the EC2 instance is moved from this EC2 host, the key is then discarded, leaving the encrypted version on the physical storage of the volume. 

If the instance needs to use this encrypted volume again, the `DEK` will needs to be decrypted by KMS and loaded into EC2 host

If a snapshot is made of an encrypted EBS volume, the same data encryption key is used for that snapshot - meaning the snapshot is also encrypted.

Anything volumes created from this snapshot uses the same `DEK` which means they are also encrypted

Every time you create a new EBS volume from scratch, it creates a its own unique `DEK` data encryption key

It doesn't cost anything to use - one of those things you should use by default 

#### Exam PowerUp!

AWS accounts can be set to encrypt EBS volumes by default.

It will use the default CMK unless a different one is chosen which is the customer managed CMK

As you know CMK itself can only encrypt 4 KB data, so it generates one `DEK` (Data Encryption Key) per volume that is then used to encrypt that volume or any future snapshots you take of that volume

Can't change a volume to NOT be encrypted. 

You could mount an unencrypted volume and copy things over but you can't change the original volume encryption status.

The volume is encrypted using AES256 - this occurs between the EC2 host and EBS system. the OS does not see any encryption and there is no performance loss.

If an exam question does not use AES256, or it suggests you need an OS to encrypt or hold the keys, then you need to perform full disk encryption at the operating system level. 

You can perform full disk encryption on an unencrypted or encrypted EBS volume.

Encryption at an EBS volume level is not related to full disc encryption inside the OS level - they both are different things.

From performance perspective, EBS volume encryption is very efficient - you don't have to worry about the keys, it doesn't cost you anything and there is no performance loss for using it.

OS level full disc encryption has a CPU performance penalty for encryption and decryption 



## Network Interfaces, Instance IPs and DNS 

An EC2 instance always starts with at least one Elastic Network Interface (ENI) 

This is your Primary ENI but you can add one or more secondary ENIs to your instance which can be in separate subnets but they all have to be in the same AZ as your instance.

When you launch an instance with Security Groups, they are on the network interface and not the instance.

#### Elastic Network Interface

Has these properties

- MAC address - Hardware address of interface visible in OS
- Primary IPv4 private address
  - From the range of the subnet the ENI is within.
  - 10.16.0.10 will be static and not change for the lifetime of the instance
  - Given a DNS name that is associated with this private address
    - ip-10-16-0-10.ec2.internal
    - only resolvable inside the VPC and always points to private IP address
- 0 or more secondary private IP addresses
- 0 or 1 public IPv4 address given two ways
  - Instance must manually be set to receive an public IPv4 address
  - Default settings into a subnet which automatically allocates an IPv4
  - This is a dynamic IP that is not fixed
  - If you stop an instance the address is deallocated.
  - When you start up again, it is given a brand new IPv4 address
  - Restarting the instance will not change the IP address
  - Changing between EC2 hosts will change the address
  - They are allocated a public DNS name.
  - Public DNS name will resolve to the primary private IPv4 address of the instance
  - Outside of the VPC, the DNS will resolve to the public IP address.
  - Allows one single DNS name for an instance, and allows traffic to resolve to an internal address inside the VPC and the public will resolve to a public IP address.
- 1 elastic IP per private IPv4 address
  - Elastic IP addresses are public IPv4 address
  - These are different than normal public IPv4 address.
  - you can have one Public IPv4 are per instance but you can have one public elastic IPv4 per private IP address on the network interface card
  - Elastic IP addresses are allocated to your AWS account 
  - Can associate elastic IP with a private IP either on the primary interface or on the secondary interface.
  - **EXAM** - If you associate elastic public IPv4 with primary interface, the normal(non-elastic) IPv4 address will be lost and the elastic IPv4 becomes instance's new public IPv4.  If you remove elastic-IP from the instance, it will gain a new IPv4 address but you won't get the initial public IP which you removed by  associating elastic IP to that network interface.
- 0 or more IPv6 address on the interface
  - These are by default public addresses
- Security groups
  - applied to network interfaces
  - will impact all IP addresses on that interface
  - if you need different IP addresses impacted by different security groups, then you need to make multiple interfaces and apply different security groups to those interfaces
- Source / destination checks
  - if traffic is on the interface, it will be discarded if it is not from going to or coming from one of the IP addresses

Secondary interfaces function in all the same ways as primary interfaces except that you can detach them  and move them to other EC2 instances.

#### Exam Power Ups

Legacy software is licensed using a mac address. If you provision a secondary ENI to a specific license, you can move around the license to different EC2 instances.

Multi homed (subnets) management and data - so an instance with an ENI in two different subnets. You can use one for Data and one for management.

Different security groups are attached to different interfaces - that's why you might want to have multiple network interfaces rather than multiple IPs on primary ENI.

The OS doesn't see the IPv4 public address. this is provided by NAT which is performed by Internet Gateway. 

You always configure the private IPv4 private address on the interface.

Never configure an OS with a public IPv4 address.

IPv4 Public IPs are Dynamic, starting and stopping will kill it

Inside VPC, public DNS for a given instance will resolve to the primary private IP address. If you have instance to instance communication within the VPC, it will never leave the VPC. It does not need to touch the internet gateway.

Outside VPC, the public DNS is resolved to public IPv4 address on the instance

## Amazon Machine Images (AMI)

- AMI's can be used to launch EC2 instance.

- Images of EC2.

- When you launch an EC2 instance, you are using an Amazon or community provided AMI.
- you can also launch EC2 instance using Marketplace (can include commercial software) provided AMIs
  - Will charge you for the instance cost and an extra cost for the AMI
- Regional, unique ID
  - ami-`random set of chars`
  - you will have different AMI IDs for the same distribution of an OS in different regions
- Controls permissions
  - Default only your account can use it
  - Can be set to be public - so everybody can access it
  - Can have specific AWS accounts on the AMI
- Can create an AMI from an existing EC2 instance to capture the current config

#### AMI Lifecycle

**Launch**

EBS volumes are attached to EC2 devices using block IDs

- BOOT /dev/xvda
- DATA /dev/xvdf

**Configure**

- Can customize the instance from application installation to attaching specific  volume sizes to meet your needs

**Create Image**

- Once it has been customized, an AMI can be created from that
- AMI contains:
  - Permissions: who can use it
  - EBS snapshots are created from attached volumes when you create AMI from EC2 instance
    - Block device mapping links the snapshot IDs and a device ID for each snapshot.
    - this means the block device mapping will have the snapshot and device ID listed
    - so when launching a new instance using the AMI, we'll have the exact same volumes attached the the new instance

**Launch**

When launching an instance, the snapshots are used to create new EBS volumes in the availability zone of the EC2 instance and contain the same block device mapping.

#### Exam Powerups

AMI can only be used in one region 

AMI Baking: creating an AMI after configuring an EC2 instance with your desired settings.

An AMI cannot be edited. If you need to update an AMI, launch an instance with it, make changes to the configuration and then then make new AMI 

Can be copied between regions. 

Remember permissions by default are your account only but you can add a specific AWS account to use it or you can make your AMI public

Billing is for the storage capacity for the EBS snapshots that AMI references

## Instance Billing Model

There are three main pricing models for EC2 instances

#### On-Demand Instances

**KEY: Most expensive among the three billing model - highly available**

- Hourly rate based on OS, size, options, etc - When instance is in running state

- Billed in seconds (60s min) or hourly

  - Depends on the OS

- Default pricing model

- No long-term commitments or upfront payments

- Great starting point for new apps or apps with uncertain requirements

- Short-term, spiky, or unpredictable workloads which **can't tolerate any disruption**.

- Exam tip, something that can't tolerate disruption or uncertain usage, can't commit or pay upfront then go with on-demand billing model for EC2

  

#### Spot Instances

**KEY:** (**Risk of termination - up to 90% off on-demand**)

Up to 90% off on-demand pricing 

spot pricing is how AWS sell their spare EC2 capacity

price you pay is based on the amount of spare capacity of that particular instance type in a particular region and AZ

AWS set a spot price which changes depending on their spare capacity levels

You can set a maximum hourly rate you are willing to pay for certain type and size of EC2 instance in a certain AZ in a certain region. 

If the max price you set is above the spot price, then you pay only that spot price for the duration you consume that instance - this could be much lower than on-Demand price

As the spot price increases, you'll keep paying the max spot price you set until the price goes above your max spot price. At that time, the instance will be terminated. 

it's great for:

-  any scenario that can tolerate the risk of termination and can continue later
-  apps that have flexible start and end times
-  apps that only make sense at low cost
-  you should not use it for single, primary web server - email or file server. 
-  Cheapest method to get access to EC2 only if your app can handle sudden termination



#### Reserved Instance

**KEY:** (**Requires commitment -up to 90% off on-demand**)

Up to 75% off on-demand in exchange for a commitment. 

You're buying capacity in advance for 1 or 3 years. You will pay regardless of instance running or not. 

Flexibility on how to pay

- All up front - Whole one or three years
- Partial upfront
- No upfront

Best discounts are for 3 years all up front - most expensive is no upfront. 

Reserved in region, or AZ with capacity reservation  -  you can also specify AZ which is known as zonal reservation. This locks you to AZ in that region and reserves capacity and it makes you priority over on-demand if there is any capacity issue in that AZ.

Can perform scheduled reservation when you can commit to specific time windows. Like certain hours every day when have high demand or data processing at certain times. You pay reduced rate in that time window

Best used for:

- If you have a known stead state usage, email usage, domain server. Cheapest option with no tolerance for disruption.
- if you want best price with reserved capacity in an AZ for a business critical apps

you can mix match any of three pricing models depending on your needs. 



**Dedicated hosts** - This is a technical architecture difference, that carry some other cost - there will be a dedicated lesson on this elsewhere but this is not an instance Billing Model like the other three. 



## Instance Status Checks and AutoRecovery

Every instance has two high level per instance status checks

#### System Status Checks

Failure of this check could indicate software or hardware problems of the EC2 service or the host.

Performs two separate tests - if either one of them fails this means you have a problem

First is the **system status** second is **Instance status**

Failure of system status check means:

-  loss of system power
-  loss of network connectivity
-  host software issues
-  host hardware issues

Failure of instance status:

- corrupt file system
- incorrect instance networking
- OS kernel issues

to fix it , you can either try to manually restart instance or use auto recovery that moves the instance to a new host with same configuration 

#### Create Status Check Alarm

This feature has four options

- Recover this instance: this uses the auto recovery feature of EC2.  can be a number of steps depending on the failure. Could migrate to a new host. Won't be able to protect you if entire AZ fails as EC2 is an AZ service. requires space capacity in that AZ. you also need modern type of EC2 instance to use this feature. only works with EBS volumes attached - doesn't work with instance store volume. 

- Stop this instance

- Terminate this instance: useful in a cluster

- Reboot this instance

  

## Horizontal and Vertical Scaling

#### Vertical Scaling

As customer load increases, the server may need to grow to handle more data. 

When you are doing vertical scaling with EC2, you are actually resizing EC2 instance. Because of this, there is a downtime - Each resize requires downtime (Disruption)

Even though AWS always updating their hardware but there is also an upper cap on EC2 instance. (Instance Size)

Often times vertical scaling can only occur during planned outages. 

Larger instances also carry a $ premium compared to smaller instances. 

There is an upper cap on performance - instance size. No application modification is needed. 

Works for all applications, even monoliths. If your app can work on one instance3 it can also work on vertically scaled instance. No app modification required

#### Horizontal Scaling

Instead of increasing the size of one instance, in horizontal scaling you add  more instances

as the load is increased, you add more capacity in terms of number of instances

As the customer load increases, this adds additional capacity.

Instead of one running one copy of an application, you can have multiple copies running on each compute instances. this means that they all need to work together. 

This requires a load balancer. Load balancer is an appliance that sits between customers and your instances. When customers attempt to access the system, the incoming load is distributed across all of the instances running your app.

Each instance gets fair amount of load and for every mouse click/interaction by customer could be on same instance or randomised across any available instances. 

Sessions are everything with horizontal scaling. With horizontal scaling you can shift between instances equally. This requires either application support or **off-host sessions**. This means the instances are **stateless and the app stores session data elsewhere. 

it requires thought and design so your application can support it but if your application does support it then you get all of the benefits:

- No disruption while scaling up or down.
- No real limits to scaling.
- Uses smaller instances so you pay less, allows for better granularity.

## Instance Metadata

Instance Metadata is an EC2 service that is provided to instances

it's data about the instances that can be used to configure or manage a running instance 

it's accessible inside ALL instances

The IP address 169.254.169.254 to access instance meta data. 

**Memorize**  ==>> http://169.254.169.254/latest/meta-data/

Meta data allows anything that's running on the instance to query it for information about that instance and that info is divided into categories

- for example hostname, events, security group etc all information about the environment. 

- networking related information is a big use case for meta data
- authentication information
  - instances can be used to gain access on other AWS services by assuming the roles 
- user-data
- **NO AUTHENTICATION or ENCRYPTED**
  - Anyone who can gain access to the instance will be able to get access to meta data
  - Can be restricted by local firewall by blocking access to 169.254.169.254 IP but that an extra per instance admin overhead. 

## 02 Introduction to Containers

Virtualisation Problems

- If you run a virtual machine with 4 GB ram and 40 GB disk, the OS can consume 60-70% of the disk and little available memory.

- Imagine we have 6 VMs running 6 OS and 6 Apps on top of OS
- With virtualisation you have a copy of individual OS
- Duplication - In most cases many of the OS are the same 
- With all these OS, they consume a lot of resources
- Every operation on these VMs such as restarts, start, stop have to be manipulate the entire OS
- what if we were able to run all six apps in separate, isolated, protected environments?
- With containers, we can. this way we'll just need one OS saving a lot of resources and space. 

#### Containerization

We still have the host hardware instead of virtualisation we have an OS running on this hardware

Running on top of it is a container engine - Docker

Containers in some way is similar to a VM as it provides and isolated environment which an app can run within

Where VMs run whole isolated OS on top of hypervisor, a container runs as a process within the host OS. 

It's isolated from all of the other processes, but it can use the host OS for a lot of things like  networking and file IO.

This means if we want to run 6 isolated apps, we can run them all using just one OS and one hardware as we won't need 6 separate VMs. This is a big benefit with containers.

Containers are much lighters than the VMs - which means we can run a lot of containers instead of virtualisations

#### Containers Architecture

A container is a running copy of a Docker image

Docker images are special as they are made up of multiple independent layers

Docker images are stacks of these layers and not a single monolithic disk image

Docker images are created initially using a Dockerfile. For example you can have a dockerfile that will create an image with a webserver in it ready to run. 

Each line in a dockerfile is processed one by one and each line creates a new filesystem layer inside the docker image that it creates

Docker images are created from scratch or using a base image.

In docker file, if the top line says `From centos:7` that means we are using centos7 as base image.

now this base image is a minimal file system containing just enough to run an isolated copy of centos7- all this is a super thin image of a disk  - it just has the basic minimal centos7 base distribution

now the second layer is to update and install the webserver apache - this means in our docker image we now have two layers. first centos7 and second layer is apache webserver

It's important to remember that the file system layer that make up a Docker image are normally Read Only.  So every change you make is layered on top as another layer

Docker container is just the running copy of docker image, except it has an additional READ/WRITE file system layer.

file system layers that make up the docker image are by default Read Only. They never change after they are created so special read write layer is added which allows containers to run. 

Anything that happens in container for example if log files are generated, or if an app generates or read data, that's all stored in read/write layer of the container. 

Each layer is differential so it stores only the changes made against it versus the layers below. Together all layers stacked up, they make up the what container sees  as a file system.

Cool thing about all this is that we can use this image to make another container. Let's call it container 2. It uses the same three base layers. 

If you have lots of containers with very similar base structures, they will share the parts that overlap.

The other layers are reused between containers.

Base layers, the operating systems, they are generally made available by the operating system vendors via **container registry**

#### Container Registry

Registry or hub of container images.

Dockerfile can create a container image, then you can upload that container image to a public or private repository called Docker Hub.

In the public repository other people and base OS vendors can also upload docker images such as CentOS.

from there, these container images can be deployed to docker hosts which are just servers running a container engine. 

Docker hosts can run many containers based one or more images.

A single image can be used to generate containers in many different docker hosts. 

each container create using container image are completely unique and isolated because of the read write layer  that a container gets the solo use of. 

you can use Docker hub to download container images or upload your own. 

Private registries (repositories) can require authentication but public ones are generally open to the world. 

#### Container Key Concepts

Dockerfiles are used to build docker images

Docker images are these multi layered file system images which are used to run containers. 

containers are portable and always run as expected. Anywhere there is a compatible host, it will run exactly as you intended. Portability and consistency are two main benefits of containers. 

Containers are super lightweight, use the host OS for the heavy lifting, but otherwise are isolated. Layers used within images are shared when possible. and Images can be based off other images. 

layers are read only so an image is basically a collection of layers grouped together which can be shared and reused. 

Containers only run what's needed.  Meaning the application and whatever application itself needs. 

Containers run as a process in the host OS so they don't need to be full OS and contains use very little memory and they are super fast to start and stop but provide same level of isolation as VMs

since containers are isolated so anything that's running in them, need to be **exposed** to the outside world. So containers can expose ports such as TCP port 80 which is used for HTTP 

Complex application stacks can have multi containers - for example you can have one container for database and one for application and these can work together. 

## Elastic Container Service (ECS) Concepts

ECS allows you to use containers running on infrastructure that is fully or partially managed by AWS -it takes away much of admin overhead

ECS uses clusters which run in one of two modes:

- EC2 mode - uses EC2 instances as container hosts
- Fargate mode - serverless way of running docker container

ECS accepts containers and instructions you provide and then it decides where and how to run those containers

It's a managed container based compute service.

ECS allows you to create a cluster. Clusters are where containers run from.

You provide ECS with a container image, and it runs it the form of container in a cluster based on how you want it to run. 

**ECS Architecture** 

- First you need a way to of telling ECS about container images you want to run and how you want them to be run
- Container images will be located on a container registry. Like Docker Hub, AWS provides a registry called ECR (elastic container registry)
- **Container definition** tells ECS where the container image is and which port your container uses. it has enough information to about a single container you want to define. 
- **Task definition** represents a self-contained application. it could have one container defined inside or many. A simple application might use a single container or it could use multiple containers(web app and a database container)
  - Task definition store the resources used by the task such as CPU, memory, networking mode, compatibility  mode(EC2 or Fargate) etc 
  - **EXAM:** One of the important thing Task definition stores is the task role. A task role is an IAM role that a task can assume and can talk to AWS products/services. This allows the task to gain temporary credentials to complete its task. This is the best practice way to give containers the access needed for other AWS resources.
  - When you create Task definition, you actually create a container definition with it but from architecture point of view, they are both different things. This is usually confused by the fact that usually tasks contain one container but they are both different. 

Tasks in ECS doesn't scale on its own, and it isn't by itself  highly available. 

ECS service - Service Definition. This defines how many copies of the task is allowed to run to load balance. It can add capacity and resilience as we can have multiple independent copies of tasks running and you can deploy a load balancer to distribute incoming traffic to all of the tasks inside a service. You can use a service definition to provide scalability and high availability.

It's the asks or services that gets deployed to an ECS cluster - applies equally whether it's EC2 or Fargate based. 

**Container definition** This defines image and the ports that will be used. - it points to a container image that's stored on a container registry and which ports are exposed from that container

**Task definition** it is applied to the app as a whole. it can be a single container or multiple container definition. it also includes task role which is an IAM role that is assumed. The temporary credentials allow access to AWS products and services. it's where you also define the resources that the task will be consuming. 

**Task role** IAM role that's assumed by anything that's inside the task. meaning it can be used by any containers running as part of the task. 

**Service** defines how many copies of a task you want to run for scaling and high availability.

## ECS Cluster Mode

ECS Cluster manages

- Scheduling and Orchestration
- Cluster manager
- Placement engine

#### EC2 mode

ECS Management Component - This handle high level tasks like scheduling, Orchestration, cluster management and placement engine (where to run containers). This exists in EC2 and Fargate mode. 

ECS cluster is created within a VPC. It benefits from the multiple AZs that are within that VPC.

EC2 mode, EC2 instances are used to run containers and you specify an initial size which controls the number of container instances which is handled by **auto scaling group**

ECS using EC2 mode is not a serverless solution, you need to worry about capacity for your cluster and whether the containers are running or not you have to pay for these instances. 

The container instances are not delivered as a managed service, they are managed as normal EC2 instances.

ECS uses container registry, these are where your container images are stored. 

Tasks and Services use images on Container Registries and via tasks and service inside ECS, container images are deployed onto container hosts in the form of containers. 

ECS will handle number of tasks it deployed or services if used but at a cluster level you need to manage  the capacity of the cluster because the container instances are not delivered as a managed service. 

This is good because you can use spot pricing or prepaid EC2 servers.

#### Fargate mode

Removes more of the management overhead from ECS, as there is no need to manage EC2 instances

The key difference from EC2 mode is where the containers are hosted. 

There is a **fargate shared infrastructure** managed by AWS which allows all customers to access from the same pool of resources.

Fargate deployment still uses a cluster with a VPC where AZs are specified.

For ECS tasks, they are injected into the VPC. Each task is given an elastic network interface which has an IP address within the VPC. They then run like any other VPC resource.

They can be accessed from within that VPC and from the public internet, if the VPC is configured that way. 

Important to remember: Tasks and Services running inside from the shared infrastructure platform and then they are injected into your VPC. They are given Network interfaces inside a VPC and it's using these network interfaces in that VPC that you can access them. 

You only pay for the container resources you use.

#### EC2 vs ECS(EC2) vs Fargate

If you are already using containers, use **ECS**

Why containers?

- Containers make sense when you want to isolate an application and your app has low usage levels, application that all use the same OS. Applications where you don't need the overhead of virtualisation. 

**EC2 mode** is good for a large workload and business is price conscious. This allows for spot pricing and reserved pricing.

EC2 mode will be cheaper if you can minimise the admin overhead of managing them meaning scaling, sizing as well as correcting any faults. 

Large workload but overhead conscious, meaning if you don't like to manage overhead then use **Fargate** mode 

Small or burst style workloads **Fargate** makes sense as you only pay for the container capacity that you use. 

Batch or periodic workloads **Fargate** - you pay for what you consume.  **EC2** mode mean paying for container instances even when you don't use them

## 03 Advanced EC2

## Bootstrapping EC2 using User Data

**EC2 Automation**

- It allows you to run certain script once the instance is launched
- For example if you would like to install a software package or configure something you can mention that in to the script and once the instance is launched the script will run to perform configuration or package installation
- this way you will get pre-configured EC2 instance rather than a default instance based on an AMI

**User Data**

- It is where you define what needs to be run once launched the instance. User data is accessed via meta data IP http://169.254.169.254/latest/user-data
- Anything that's in user data is executed **only once** when the instance is initially launched/provisioned
- If you update User Data and restart your instance the user data will not run - it runs only once when you launch the instance
- EC2 does not validate what's in User Data. Whatever script you have in User Data will be executed at the initial launch of the 

**Architecture**

- first the AMI is used to launch instance
- EC2 then passes user data that contains the script to run via meta data IP http://169.254.169.254/latest/user-data
- Instance then runs the user data - if the script is run successfully with no errors then instance will show as running state but even if there were any errors the instance will show as in running state but it might not be configured as you wish. 
- If you wrote something that could cause some irreparable damage to instance such as deleting the instance volume, then of course then instance won't be running properly and it might not be in running state

Since user data is stored in the same meta data IP, that means that it's not secure. Anyone who has access to AWS account can see this user data - try to avoid putting any user credentials or passwords

there is a limit of 16KB in size for the user data - anything larger than this needs to be downloaded.

You can update the user data when instance is at stopped state but it will only be executed at initial launch state. So you can update user data and create a new instance and it will then be executed.

**Booth time to Service Time**

From amazon AMI to instance running state  should only take few minutes. 

Post launch time is the time that takes you to manually configure package installation and other settings once the instance is launched - since this is done manually to the time to complete this setup could take from minutes to hours

To speed up this process you can either use bootstrapping which is running a script once the instance is launched 

or you can also so AMI backing where you create your own pre-configured image so you won't even need to do bootstrapping. 

## Enhanced Bootstrapping with CFN-INIT

- CFN-INIT is another more powerful way to do bootstrapping when you launch your EC2 instance

- The user data option for bootstrapping runs commands in a set procedure but with CFN-INIT configuration system you can launch your EC2 instance in your desired state. 

- With CFN-INIT you tell which apache version you want and CFN-INIT will check if the apache server is already installed, if not then it will install the version you need. It's very powerful as it can manage users, groups, packages., files and other important things.

- To use CFN-INIT you specify the type of desired state you want your EC2 instance and CFN-INIT will make sure it provides it to you

- you define CFN-ININT in cloud formation template under metadata, under AWS::CloudFormation::Init: which holds the configuration details. 

- As soon as the EC2 instance is launched, the user data is executed that contains variables that points to the correct stack to use for cfn-init. With these variables, the CFN-init can communicate with cloudformation to understand the desired state you want your EC2 instance to be in.

- Good thing about this is that it also works with stack updates - This means that you can specify in the meta data to watch for updates on certain component of EC2 instance and if any update is found then it can be execited again to keep your instance in your desired state. Unlike user-data only option which can only be executed once

#### Creation Policy and Signal

Creation policy works along with the Signals - In creation policy you can define time out value and Signal will tell you if the cfn-init command ran successfully or not. Signal can send ok or error message back to cloud formation but if it doesn't provide any information for more than the time out value in creationpolicy, then error will be assumed and stack status will not be set to Create Complete state. If Signal send OK/success message then the stack status will change to Create Complete status. 

## EC2 Instance Roles & Profile

- If you have an app that needs access any AWS service or product, best practice is to use IAM roles. 
- IAM roles assumes a role on your behalf and temporary credentials are generated for that role to execute a task. 
- Just like an app, EC2 itself can assume a role and anything that's running inside that EC2 instance can carry out a task assuming that role the right set of permissions.

**Architecture** 

IAM role is generated that has a permission policy attached to it. Who ever assumes this role, gets temporary credentials generated that has the permissions that the permissions policy defined.

To deliver these IAM role credentials to EC2 instance, InstanceProfile is used that's attached to the EC2 instance and this InstanceProfile delivers credentials via meta-data.

When an instance role(IAM role) is created via AWS console, instance profile is also created with the same name but if cloudformation template is used then both Instance role and Instance profiles have to be created separately. 

An important thing to remember is that these credentials in instance meta-data are always reviews. the STS(Secure Token Service) talks to EC2 service to renew credentials before they expire.  

So your app inside EC2 instance will always have valid credentials every time it fetches via instance meta-data for the IAM role to carry out tasks. 

**Credentials Precedence**

https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-precedence

## SSM Parameter Store

It's an AWS service that allows you to store important data such as database password, license codes or an app password or configuration settings.

Rather than using User data which is accessible to anyone who has access to EC2 instance, or storing passwords or other important information in a persistent disk, it's better to use this service.

It can store strings, documents, and secrets in the form or parameters. You have parameter name and its value and the value is where you store important data. 

Many AWS services have strong integration with SSM Parameter Store. You can use it via Cloudformation or EC2 instance CLI tool

You can store three different types of parameters:

- String
- StringList
- SecureString

Parameter stores also supports hierarchical structure and versioning. You can save parameter as plain text (DB String) or as ciphertext using KMS. 

SSM is very flexible in terms of permissions, you can either create a tree branch and give access to one team so they can store all the passwords they need or you can give access to a branch tree for a specific application

Any change they occurs to any parameter can trigger events 

You can also store parameter as public so every can have access to that(Latest Amazon Linux AMI name)

Access:

- Via IAM User (long term credentials)
- Via IAM Role (Temporary credentials)
- if parameters are encrypted then access to KMS is also needed

## System and Application Logging on EC2

Although CloudWatch and CloudWatch Logs are great way to monitor EC2 instance metrics and logs from the outside but sometimes you need to create logs based on what's happening inside the EC2 instance

To see what's happening inside EC2 instance CloudWatch Agent can be used that can send OS-Visible data to CloudWatch or CloudWatch Logs

You will need to setup some configuration and permissions for CloudWatch Agent 

We can install CloudWatch Agent manually, setup configuration and start logging or we can automate this via Cloud formation. We can also use Parameter store to store CloudWatch agent configuration and we'll also need to have a role that has permissions to CloudWatch Logs that can be attached to EC2 instance

The metrics and logs we want to capture in instance are all stored via log groups:

- one log group for each log file
- Within each log group there will be a stream for each instance

##     EC2 Placement Groups

When you provision an EC2 instance in an AZ, it's up to AWS to decide which EC2 host should give you the EC2 instance you have no say in this. But with Placement Groups you can influence this decision. You can tell AWS if you want two EC2 instance to be close to each other or be away in that AZ

There are three types of Placement Groups for EC2:

1. Cluster - to make sure physical hardware of EC2 instances are close together - belong to 1 AZ
   - Highest level of performance inside EC2 - super low latency - max number of packets
   - Best practice: You first create the group and provision same size instances at the same time to avoid capacity issues. 
   - We'll have same rack - sometimes potentially same EC2 host
   - 10 GBPs single stream (you usually get 5 GBPs)
   - Little to no resilience - since they are physically close - if the hardware fails - they could all go down
   - EXAM 
     - ONE AZ ONLY
     - Can span VPC peers with some performance hit
     - Not all instance are supported in cluster placement group
     - Best practice - use same instance type but this is not necessary
     - Best practice to launch all instances at the same time to potentially avoid capacity issues but not necessary
     - Cluster group placement you can achieve 10 Gbps single stream performance
     - Use Case: Performance, fast speeds, low latency
2. Spread -  keeps your EC2 instances to be spread in AZ (inverse of cluster) - ensuring instances using different hardware
   - Use Case: Max amount of availability and resilience
   - can span multiple AZ - Instance in this group are placed in isolated hardware in each AZ
   - each have their own electricity and networking - meaning if networking or electricity fails then the failure could be isolated
   - EXAM:
     - High availability and resilience - each instance runs from a different hardware rack
     - 7 instances per AZ is a hard limit 
     - This is not supported for dedicated instances or hosts
     - Use Case: critical instances that needed to be kept separated to ensure faults are isolated
     - managed by default by AWS 

3. Partition - designed for distributed computing - Ensuring groups of instances spread apart - each group has different hardware
   - You define how many partitions you need per AZ - Max you can have is 7 partition per AZ
   - unlike Spread Placement group where you can have max 7 instances per AZ, here you can have as many instances as you like per **partition**.
   - You can either allow AWS to decide which instance goes to which partition or you can decide yourself
   - each partition is physically isolated with other and has it's own power and networking.
   - if the hardware of one partition fails then only that partition is impacted
   - you can see which instances is in which partition and you can use this info for topology-aware applications, HDFS, HBASE and Cassandra. These apps can use this info to optimise their replications 
   - Use Case: Large scale systems where you have groups of instances 
   - **EXAM**:
     - 7 partitions per AZ
     - instances can be planked in specific partition or you can let AWS decide(auto placed) 
     - not supported for dedicated hosts
     - USE Case: for apps that need to be topology aware HDFS, HBASE and Cassandra

## Dedicated Hosts

You can have dedicated EC2 host to yourself

you pay for the EC2 host and don't have to pay for the instances inside that EC2 host

You can get it on-Demand or reserve it for 1 to 3 years period

The hardware for dedicated host has sockets and cores. If you are running a licensed software that based on number sockets or cores - you can use the dedicated EC2 host sockets and cores for the license and then you can create as many instances as you like within that EC2 host. 

some enterprise software are sold based on socket/number of cores. So if you have a small EC2 instance in an EC2 host, you will be paying for the physical socket/cores on that host even though you are not consuming all that capacity as you are only running a small EC2 instance on that host.

You can only get specific family of instances with dedicated hosts for example if you have 1 Socket with 16 Cores, you can get:

16 x medium instances

8 x Large instances

4 x xlarge instances

2 x 2xlarge instances

1x 4 large instance

With the new Nitro based dedicated hosts, you have more flexibility. If you have 2 Sockets with 48 Cores:

1 host can have:

1  x 12 xl  

1 x 4xl

4 x 2xl

and another one can have

4 x 4 xl

4 x 2 xl

**Limitations**

These are not supported in Dedicated Hosts:

- AMI Limits - RHEL, SUSE Linux, Windows AMIs
- Amazon RDS
- Placement Groups

Dedicated hosts can be shared with AWS account using RAM(Resource Access Manager) product

**Use Case**

You use dedicated hosts mostly for software licensing that use number of sockets/cores for licensing

## Enhanced Networking and EBS Optimised

**Enhanced Networking**

- It uses SR-IOV Network interface card. It's a special NIC that is aware of virtualisation. One physical SR-IOV can have many logical NIC that attached to EC2 instance
- Without SR-IOV the instance communicated to the physical NIC that's attached to the EC2 Host where the EC2 host has to translate which instance is trying to communicate to the NIC. This is handled via software and it consumes CPU cycles
- With SR-IOV logical NIC no translation is required which in return you get:
  - higher I/O 
  - lower host CPU usage 
  - More Bandwidth
  - Higher Packets-per-seconds(PPS)
  - consistent lower latency
- It is available by default and  does not cost you extra - pretty much available for most modern EC2 instance types

**EBS Optimised**

- Most Instances come with Optimised EBS by default - disabling it might not cause any difference as the hardware now comes with this feature built in
- for some older hardware it is also supported but enabling it will cost extra
- EBS optimised means dedicated storage for EBS
- historically the network was shared between data and EBS. this caused lower performance for both data and EBS - since EBS is just a block storage connected to instance via network
- EBS optimised means that at network level some optimisation has occurred and a dedicated storage capacity is provided to EBS - meaning faster speeds possible with EBS and data transfer over the network 

## 04 Domain Name Service (DNS) Fundamentals

It's a discovery service. It's how systems are discovered in public and private networks

It translates human readable information into a language that machines can understand

For example, if you type www.amazon.com in your browser, it will translate it to an IP address(104.98.34.131) to tell you where to go to get www.amazon.com page.

As you can imagine the number of websites in the world, DNS has to maintain a huge database which has to be distributed

With IPv4 there are over 4.3 billion IP addresses and for each IP address if there are more than one web page assigned then you can imagine how massive this database can be.

With IPv6 you have even more IP addresses to handle.

**Architecture**

- When you first try to talk to amazon.com, from your PC/device you first reach out to DNS resolver that could be with your Internet Service Provider(ISP) or it could be in your router. 
- To find the IP address of amazon.com, we need to reach the Zone File that's stored somewhere in the global infrastructure of DNS. This Zone file has the DNS record for www.amazon.com meaning the IP address
- Out of millions of DNS servers around the world, this Zone file is physically hosted in one or two DNS server called Nameserver(NS)
- So DNS provides a core service to allow DNS resolver that's with your ISP or in your router to fin the Zone file for the website you are trying to reach, query the zone file and return the IP address for the web site you are tying to reach
- DNS is distributed across the globe, it's resilient and hugely scalable.

**Key terms to remember**:

- **DNS client** - The device that's trying to find the IP address of the website it's trying to reach. In other words DNS client is any device that's looking for information that DNS has. it's usually a piece of software that's inside the device that you are using- it could be your PC, phone tablet or laptop
- **DNS Resolver** - That's a piece of software that could also be running on your device(DNS Client), or a separate server inside your router or could be a physical server running inside your ISP(Internet Service Provider) - It's this DNS resolver that queries the DNS system for you.
  - So in short the DNS client talks to DNS resolver, the DNS resolver talks to DNS system to find the IP address of the website your trying to reach.
- **Zone** - This is a part of global DNS system for example, amazon.com, netflix .com google.com. These zones live inside the DNS system. Zone is like the classification of the data - meaning what data are we talking about. Zone file is how the data is physically stored for the zone. 
- **ZoneFile** - is where the data stored physically for the zone - so there will be a physical file for amazon.com, netflix.com etc
- **Nameserver** - This is where the zone files are hosted. 

**DNS workflow**

- Find the nameserver that's hosting the zone file you are looking for, query this name server to find the record of the website that is stored in the zonefile on this name server and return the result back to DNS resolver which will return the result back to DNS client. 

**DNS Root** 

DNS is distributed but it has to have a starting point. This starting point is the root.

When you enter a website www.amazon.com. it is read from right to left. Notice the period after com, that's the root. usually when you type www.amazon.com that period is not there/needed but browser adds it for you. 

from this period, your device starts reading the website name and it represents the root of DNS

This DNS root is also known as DNS root zone is the starting point of DNS infrastructure and it's just a database.

DNS infrastructure is like a tree upside down. and this DNS root or DNS root zone is at the top of this DNS tree

This DNS root or DNZ root zone is hosted on 13 special name servers knows as root servers. These 13 root servers are run by 12 different large companies around the world.

These companies only manage root zone servers only, not the database. As you can imagine these are not just 13 servers but they can be a cluster of servers distributed globally but represented by one company. 

Each of these 13 servers (cluster of servers) are assigned letters from a to m. Each of these managed by different companies except Verisign that manages two servers **j** and **a**

These root servers are the entry point to find the website. So in order to reach any website you need to get to one of these servers. 

To find these root servers, you use Root Hints file that's provided by your OS provider. For Windows it's Microsoft and for Linux is OS maintaining group.

This root hints file is just a file that has a list of these 13 root servers. it's just a pointer to the root servers. 

**DNS Workflow**

Step 1 - So the first step is that your DNS Client(Your device) talks to DNS resolver to find the IP address of www.aamzon.com

Step 2 - Using this root hints file, the DNS resolver communicates to one or more of the root servers to find the Root zone and begin the process of finding IP address

- Root Zone is managed by an organisation called IANA (Internet Assigned Number Authority)
- It's their job to manage the content of Root Zone
- its a separate job from managing the root servers which hosts the root zone
- DNS is built on trust. DNS client trust this root zone via root hint file or the DNS resolver server that itself relies on this root hint file
- In DNS the starting point is root zone and that's the only thing we trust at the beginning - when we trust something in DNS system, that thing is an authority. in other words that thing is authoritative
- The root zone can also delicate some responsibilities related to DNS to another authority  or entity. So if Root zone trust this entity that means that is trusted and it also becomes an authority. 
- Root zone is just a database - it's a database of top level domain(TLD)
- The top level domain with country code are called CCTLDs  (Country Code Top Level Domains) and others are called generic gTLDs or just TLDs as they aren't mapped to any country.
- So th eroot zone just contains the database for TLDs and ccTLDs that point to different entities for different root zone. For Example, .com is delegated to Verisign and then Verisign has various nameservers that hold the .com zone. one of the zone within verisign will have amazon.com and that zone will give you the IP address of amazon.com
- So you first go to root zone with amazon.com, that root zone will forward you to the relevent trusted entity. In this case verisign. And then verisign will forward you to exact name server where .com zonefile is stored that contains amazon.com ip address. 

## Route 53 Fundamentals

It allows you to:

- Register a domain
  - To register a domain, Route53 has relationships with all major Top Level Domain(TLDs) companies
  - Lets support you want to register animal4life.org 
  - .org is managed by a company called PIR
  - When you try to register this domain, Route53 first checks the registry if that domain is available using contacting PIR. Assuming it's available.
  - Route53 creates a ZoneFile for your domain. Zone file is just a file that contains DNS information for that domain - eg. IP address
  - Route53 then allocates a name servers for this zone
  - Route 53 manages these  name servers that are distributed globally. For each zone there are 4 name servers. 
  - This zone file is called hosted zone in AWS route 53 terminology and route 53 puts this files to these 4 managed name servers 
  - Route53 also has to communicate with .org registry and it lists these name servers into the zone file for the .org top level domain. 
  - It used Name server records to do this. 
- Host zones and manage Nameservers
  - Zone files in AWS are called hosted zones as they are hosted on 4 name servers
  - you can have public or private hosted zones
  - with public anyone with internet connect can reach to the name server that hosted zone is located
  - but with private it's only available with the VPC it's linked to. 
  - it stores various record set such as name server and others...

It's a global service that is globally resilience. If any region goes down, this service will continue to function. 

##  DNS Record Type

- **Nameserver (NS)**
  - This allows how the delegation occurs in DNS
  - .com is managed by verisign - so as you can imagine with .com you can't have just one server with all the zonefiles in it. So to reach amazon.com there will be multiple namesservers that can help you with amazon.com zone that verisgn handles. Just like AWS has at least 4 nameserves for each zonefile(hostedzone)
  - In other words, NS is type of DNS Record Type that has the list of server names that contains DNS records such as for amazon.com. So these servers host amazon.com zone. Inside this zone we have DNS records such as www which is how you access these DNS records. 
  - **So name servers records are how the delegation works - They guide you where to go next to resolve the DNS record you are looking for.** 

- **A and AAAA Records**
  - It converts Host to IP address
  - A record maps host to an IPv4 address
  - AAAA maps host to an IPv6 address
- **CNAME Records**
  - Creates DNS shortcuts - Host to Host records
  - one server can perform multiple tasks such as ftp, mail, web hosting and others 
  - so three CNAME records are created pointing to the same server
  - CNAMES are used to reduce admin overhead
  - CNAME only points to the Sever name for example amazon.com. It doesn't point to the IP address. Only A or AAAA record type points to IP address

- **MX Records**
  - It is a massively important type of record - especially how the email works
  - MX records are used to find the mail server
  - for example if you want to send email to hi@google.com. You use the same route - first go to root zone, then google.com zone and that's where you will find the MX record.  
  - Example of mx records 
    - MX 10 mail
    - MX 20 mail.orther.domain.
  - MX records have two things, priority and value. The low value in priority field such as 10 has higher priority.
  - you will first go to mail server using A record that has the IP address. 
  - If you just mail in your MX record that just means that you want to go to mail.google.com
  - but if you have a fully qualified domain name such as mail.other.domain then it could mean you want to go elsewhere. for example you might want to use microsoft 365 for your mail services.
  - In other words an MX record is used to find the mail server for a specific domain

- **TXT Records**
  - allow you to add a piece of text of your choice to a domain
  - This allows DNS to provide additional functionality such as to prove domain ownership.
  - So for example lets say you added a TXT record with text "cats are the best" 
  - Now you want to use Google or microsoft mail services to setup your own domain email. Google or microsoft could query TXT record and if it matches to your text that means we own the domain. 
  - It can also be used to fight spam

**DNS TTL** (Time To Live)

lets say you need to open amazon.com page, you first go to root zone, that gives you the .com zone, and that .com zone give you amzon.com zone and thats where you get the www record for amazon.com

This whole process takes time and by getting the answer back from the final name server is called Authoritative answer. 

This walking down from the root to branches of tree process takes time but is always preferred as it;s always going to be accurate.

But the administrators or amazon.com could tell us how long we can cache that authoritative answer. 

Lets say amazon has set 3600 TTL value, it means the resolver server can store the answer to query for 1 hour. 

If someone else tries to reach amazon.com then they will get non-authoritative answer that's stored in resolver server. but the client won't have go from tree root to branch and the result will be super quick as it is cached as the resolver server. 

it's good to get quick non-authoritative answer from resolver server but imagine if you have changed your mail server and you have high TTL value on your MX record then your email delivery might be delayed

Low ttl value means more queries against name servers high value means less control (non-authoritative answers)

The resolver should obey TTL values but it's not guaranteed. It could ignore TTL value. This configuration is managed locally by the admin of resolver server.

if you are doing any work that involves changing DNS records then it's best to reduce TTL value well in advance - weeks or months. So the records are not cached.

## R53 Public Hosted Zones

- R53 hosted zone is same as Zone file - it's just a database to keep DNS related records such as for animal4life.org
- There are two types of Hosted Zones in r53 - Public and Private
- Public one is exposed to internet and anyone can access it and the private one is not exposed to public internet. It's linked to a VPC inside AWS and it's not visible to internet. 
- R53 is a globally resilient service  - if a region fails then this service will continue to function. 
- Hosted Zone is created automatically if you register a domain using R53 service 
- You can also buy/create a domain elsewhere and then use R53 to host your zone file. 
- There is a monthly fee you have to pay for using R53 to host your zone file (in AWS it's called hosted Zone) and also you pay for the number of time that hosted file is queried/requested. 
- Hosted zone, whether public or private, host different types of DNS records - for example, NS, A, AAAA, TXT etc 
- If a hosted zone is being used via NS record that means your hosted zone is authoritative for that domain. 
- When you register a domain, name server record for that domain are entered in Top Level Domain zone for example for .org it will be PIC(Public Interest Registry) and these point to your name servers in AWS so your name servers and the zone they host becomes authoritative for that domain
- With public R53 hosted zone it's available via public internet as well as VPCs via r54 resolver
- when you buy domain via r53 it saved hosted zone in 4 r54 Name servers for that domain
- to integrate with public DNS system you change the NS records for that domain to point to 4 name servers in r53
- Inside public hosted zone, you create Resource Records like MX, TXT WWW records.
- You can also buy domain via godaddy  or any other company and then create public hosted zone in r 53 that gives you public name server and then using go daddy interface you can point your domain to these name servers.
- The 4 name servers assigned for each hosted hones are accessible via public internet as well as VPC via 553 resolver. 

**Using VPC**

- if you have enabled DNS resolver in your VPC then it's straight forward- you just type the domain name and it goes straight to DNS resolver that points to r53 hosted zone

**Via Public Internet**

- From your PC you go to your ISP resolver then root servers then .org TLD that contains NS records that point to NS servers in R53
- There is a monthly cost for hosting this public hosted zone as well as you will have to pay for the number of times this hosted zone is queried

## R53 Private Hosted Zones

- Private hosted zone is just like public except its only available inside a VPC in AWS
- you can associate private hosted zone in AWS using UI Console, CLI or API
- you can also associate Private Hosted zone in different AWS accounts using CLI or API only
- SPLIT-VIEW: you can have public and private hosted zone of the same name. This means when someone from the pub internet tries to access a web site they will be presented with publicly hosted zone but if someone tries access the same name inside AWS, they will be directed to private hosted zone. 

## CNAME vs R53 Alias

- CNAME maps a name to another name
- For example, you can have a CNAME  www.google.com maps to google.com
- It's a way to create an alternative name in DNS.
- The problem is you can't use CNAME for Naked domain name. Named domain is also called apex of a domain
- it's a domain name without www at the start
- so you can't have google.com pointing to something else
- Alias om AWS fix this problem. With alias you can point to something using apex of a domain/naked domain
- you can still continue to use CNMAE for www records
- ALIAS records map a NAME to an AWS resource
- it can be used for naked/apex as well as normal records
- for www.google.com you can use either CNAME or ALIAS
- for naked/apex domains you have to use ALIAS records if you want to point to an AWS resources
- There is no charge for ALIAS requests pointing at AWS resources
- EXAM: Remember to use ALIAS by default for pointing naked/apex domain for AWS resources
- ALIAS is a subtype - you can have an A record ALIAS meaning an IP address or CNAME record ALIS meaning a name. such as ELB load balancer name
- ALIAS is an AWS naming convention - it's only usable in Route 53 and outside in normal DNS it's not usable
- you need an ALIAS record for AWS services such as API Gateway, CloudFront, Elastic Beanstalk, ELB, Global Accelerator and S3

## Route 53 Simple Routing

One of the simple routing policies available in Route 53 is called Simple Routing

Lets say we have a hosted zone called animal4life.org and with simple routing you can have one record per name 

For example WWW which is an A record type. As you know A record returns an IP address. With Simple routing you can have one or more than one IP address assigned to one WWW records. All IP addresses are returned by the query in random order

Simple routing does not support health check - meaning it doesn't check the IP address it's pointing is operational or not but simple routing is fairly easy to setup and manage. 

all other forms of routing policies in route 53 offer some form of health check on the pointed location. 

## R53 Health Checks

Health checks are different from records - they are a setup and configured separately

Health Checkers are used to check health of something and they are distributed globally

 Health checkers are not limited to AWS - you can check anything over the public internet - you only need an IP address and Check occur every 30 seconds by default but at an additional cost you can increase it to 10 sec

You can do the following checks:

- TCP
  - this needs to be successfult within 10 seconds
- HTTP/HTTPS 
  - More useful for Web application then a simple TCP check
  - Route 53 should establish TCP connection with endpoint within 4 seconds and the end point should respond with HTTP status code in the  200 or 300 range within 2 seconds after connecting
- HTTP/HTTPS String Matching check (Most accurate health check)
  - Same as above but once route 53 receive the status code it must also receive the response body within 2 seconds. Route 53 checks the response body for strings that you specified. 
  - the string should be in the first 5120 bytes of the response body else the target server fails the health check

Based on the results your target servers can be declared Healthy or Unhealthy

**Checks Types**

- Endpoint - simply checking a server

- CloudWatch Alarm - it would reach to CloudWatch alarms configured separately and can contain some in OS or in-app tests if CloudWatch agent is used

- Check of Checks (Calculated) - you can setup application wide health check with individual components


## Failover Routing

With Failover Routing if the primary address isn't working then you will be redirected to the secondary address potentially hosted by a different machine.

We can use "out of band" failure/maintenance page for our website in case the primary address is not working

The health check generally occurs on the primary resource but if it fails then the secondary records of the same address sis returned

You use this to route traffic to a resource when it's healthy but if it's unhealthy then you want ot route to a difference address for the same resource

## Multi Value Routing

it's a bit of a mixture between fail over and simple routing

With Multi Value Routing you can create many records with the same name - For example you can create many WWW A records that map to different IP addresses

Up to 8  healthy records are returned to the DNS client when queried - If you assign more than 8 IP addresses than 8 randomly selected records are returned

Any records with failed health check won't be returned

This is not an alternative to a load balancer that handles session based data but to be able to return multiple healthy IP addresses to server the request improves availability of application 

**Simple Routing vs Failover vs Multi Value Routing**

- Simple routing only links one resources such as a web server and doesn't not support health checks
- Failover is used if your main application is down so you can redirect user to another page(possibly S3 bucket Status Website) 
- Multi Value is used when you have many resources that can server the request and they all can be health checked

## Weighted Routing

This is like a simple form of load balancing or when you want to test new version of Software

lets say you have 3 WWW A records and with weighted routing you can assign weight against each record

so lets say you weight records like this:

WWW 40 A 1.2.3.3

WWW 40 A 1.2.3.4

WWW 20 A 1.2.3.5

the top record will be returned 40 % of the time

Same is true for the middle record but the last record will be returned only 20 % of the time

If you set a weight to 0 that means that record is never returned but if you set all records to 0 then all of them are returned

If a record becomes unhealthy then the process of selection is repeated until a healthy record is selected

Unhealthy record doesn't cause any changed in the weightage

## Latency-Based Routing

it should be used when you what high performance and good user experience

with latency-based routing you can assign regions such as us-east-1

Latency-based routing supports one record with the same name in each AWS region

So AWS maintains a database with users source, destination location and latency

So when a user requests a record for example, from AUS then the record that's tagged with ap-southeast-2 is returned since that would provide the lowest latency based on the AWS latency database

Latency based routing can also be combined with health checks. If a record is unhealthy then the next record with lowest latency will be returned

## Geolocation Routing

With geolocation routing instead of latency the user's geo location and resources locations are used to influence address resolution

It's like a lookup where you create a record and tag it with state(in case of US), country, continent or with default

So if user location match with the tagged record, the record is delivered. if it doesn't match then the record is not delivered at all. This means the user is delivered location based resource. 

IP checks verify location of the user (depending on the DNS system, it could be resolver or the user directly)

so once you have the user location then state check is done - if a record is found with the same state then then the record is returned and the process stops. If not then the country is checked, if not found then the continent  is checked. If not of the records mast with the user locating then the default record is returned. 

**EXAM**: Remember with geo location, nearest record to user is not returned but those that match to user location is returned. This means you can do regional restrictions, language specific content or load balancing across regional endpoint.

## Geoproximity Routing

it provides records that are as close to your customer as possible

With latency based routing you the customer is provided record that is close to their region bases on estimated latency but with Geoproximity it provides records by calculating distance from customer and has some other benefits

with geoproximity, you provide the region name in case of AWS resource but it f it's an external resource you provide longitude and latitude coordinates - you also define a bias

lets say we have three resources, one in US, another in UK and a third one in Australia. Lets say someone from KSA needs to access these resources. Since there isn't any resource in Saudia arabia and the closest location from Saudia is UK so the record in UK will be delivered. 

you can change this decision by adding a bias. With bias you can define which geo locations should be served by which resource. So you can move the traffic coming from KSA to Australia instead. 

This means, rather than using actual physical distance between the user and resource, you can introduce a bias to influence the traffic to a certain location/resource. With plus you can increase the area for a resource to serve - so you can increase the bias for UK to server the whole of Europe and for Australia to serve the whole of Asia.

## R53 Interoperability

R53 provides two functions:

1- Register a domain for you 

- AWS has relationships with key TLD companies to register domain
- you pay AWS to register a domain main for you  - AWS contacts the TLD company that manages DNS for the TLD. For example for .org PIR manages the DNS.

2- Host Domain for you

- AWS can also host the domain for you by storing hosted zones in 4 server
- If you register domain via R53 and also want to host the domain then R53 will send NS records for 4 servers to TLD 

if you like you can also register your domain via thirst party such as godaddy and then host your domain in R53 by creating a public hosted zone and you will need to provide the TLD NS records via godaddy.com

if you like you can also just register domain via r53 but host your domain via 3rd party such as godaddy- this is rarely happen but this is something you can also do. In this case you can use r53 to provide the hosted domain details to TLD

EXAM: Key thing to remember is that r53 provides two separate roles. One is to register a domain on your behalf - Second to host the domain in r53. If you want  you can do either one of them and use 3rd party to do the second bit.

## 05 Database Refresher

Databases are systems that store and manage data

Key differences in database systems is how the data is stored, managed physically on disk and in memory and how the data is retrieved and presented to the user

There are two key categories of databases 

- Relational (SQL)
- Non-Relational(NoSQL)

Relational Database Systems

- Structured Query Language (SQL) is used to store, update and retrieve data. It's a language that most relational database management system (RDBMS)  use
- Relational databases have a rigid structure that they follow. It's called Schema. For example a table in a database will need to have a schema before any data is stored in it. It's fixed. 
- Schema defines the name of things such as column names, the type of things you can store such as String or integer, valid values your table/columns can accept and where to store the data
- In Relational database systems you also have a fixed relationships defined between tables and that's also defined in advance 

NoSQL

- it's not a single thing - it's something that doesn't fit in relational database systems
- it's like an alternative database model
- it's not as rigid as relational database - it's has a much more relaxed form of schema - Either they have weak or no schemas at all
- Relationships between tables are also handled differently 

### No-SQL

**Key-Value Databases**

it just has a key and it's value

For example you have a key with time stamp and value could be the temperature recorded at that time stamp.

it has no schema and no structure - it's usually very scalable as the sections of the Key-Value pair could be distributed in multiple servers and of course since it's just key and value, it's super fast

EXAM: it is also used for in-memory caching 

### Wide Colum Store(Dynamo DB)

- It's like the key value pair defined above but with this you can have one or more keys per record. This is the only rigid part of this model. If you have defined two keys then you have to have two keys for each record. They have to be unique to that table
- The first key is called the partition key and then you can have another key if you like
- Wide Column store offers groupings of items called tables
- another important thing to remember is wide column store can have many attributed attached ot it and not every record has to follow those attributes 
- you can also have a records with no attributes at all - There is no fixed structure/schema for the attributes
- They only thing that's rigid in this model is the keys have to be unique per record. Doesn't matter if you are using a single key or a composite one but they have to be unique per record
- good for large/web scale applications

### Document Database

- Jason or XML - the structure of documents could also be different in the same database
- it's like an extension of key value store - each document is accessed via key that's unique for that document
- for each key you will have a document
- it's good for order, collection or contacts database
- Generally good for when you interact with data as a document
- it's good for nested items/deep attributes
- good for catalogues, user profiles and other content management systems where each document is unique but changes over time
- Each document has a unique ID and the database has access to the contents inside the document
- provides flexible indexing - you can run powerful queries and extract data stored deep inside a document

### Column Database (Redshift)

#### Row based databased

Row based databases are good when you have to access the whole row of data - you you would like to update delete or add a new row

they are also called Online Transaction Processing (OLTP) databases

goof for performing transactions

#### Column Based database

it stores data in columns

it's really good for reporting

Columns are stored together - perfect for querying/reporting if you are only interested in just one part of the data such as product or size

Generally they are suited very well for reporting and analytics

### Graph

relationships between things are defined and stored in a database along with the data

the relationship is not calculated each and every time you run a query

perfect for Social media and HR systems

it stores data in the form of nodes for example person, company or city

nodes are just like key value pair for example person node can have a value of Greg or Natalie. 

These nodes are then linked with other nodes using edges. Edges can have name and direction 

so you can have Greg working for a company called xyz_corp so the person node with greg will have a relationship with company node with value xyz_corp and direction will be from greg to company

Edge can also have a key value pair - so in case of Greg we can store the start date of when Greg started working for that company

with SQL when you retrieve the data for company xyz_corp the relationships of the company is executed at the time of query execution

but with graph database the relationships are stored with the data so it's but faster and efficient to retrieve 

Use Case: Social Media - Complex relationships

## ACID vs BASE

ACID and BASE are mechanisms to access database.

**CAP Theorem**

- Consistency
  - The queried data from database will always be consistent and most recent otherwise you will get an error
- Availability
  - it means that even if the data is not consistent you will always get an answer from the database 
  - You won't get an error even if certain portion of your database is going through a failure but without the guarantee that the data will be consistent/most up to date
- Partition Tolerance(resilience)
  - This means that the database system can be made of multiple network partitions
  - the system will continue to function even if there are errors in some of nodes 

The CAP theorem says that any database system is able to deliver any of the two of these factors

ACID = Consistency

BASE = Availability

ACID = Atomic Consistent Isolated Durable

EXAM: ASID = RDS databases = Relational Database Systems - ACID limits the ability of a database to scale

- ATOMIC = either all or no part of transaction successful
- CONSISTENT = When a transaction happens, the database moves from one state to another - nothing in between
- ISOLATED = multiple transaction occurring at one time do not interfere. multiple transaction are executed in a way as if they were executed sequentially
- DURABLE = Once transactions are executed successfully, they become durable. Power outage  or crashes don't hard the successful transaction

Most relational databases use ACID based transaction models - these are rigid databases used mostly by financial system and limit scalability 

BASE = NoSQL Models = Basically Available - Soft State - Eventually Consistent

- Basically Available -Read/Write are usually available but without consistent guarantee. Rather than focusing on consistency they tend to spread/replicate data to multiple nodes. It tries to be best with consistency but there are no guarantees
- Soft State - The database itself isn't guaranteed to be consistent so developers/applications have to somehow figure out consistency on their own
- Eventual  Consistency - If we wait long enough the reads from the system will be consistent

It just means that when you are reading via BASE modelled database, you will not necessarily get the most up to date write. You will eventually get the write you want want but it might take some time and is not guaranteed. So you as a developer have you have an awareness of this

DynamoDB is NoSQL but Dynamo DB also offers ACID based model called DynamoDB Transactions

## Databases on EC2

it's usually considered a bad practice to run a database in an EC2 instance 

Architecturally speaking you might have just one big EC2 instance and run Database, application and Webserver on the same instance.

Or you might setup your database in a separate EC2 instance in one AZ and another instance with Application and Webserver in another AZ. Now that you have your system spread into two AZ bear in mind that there is a cost associated when transferring data between AZs

**Why you might want to do this...**

- You might need OS level access on the Database EC2 instance (OS level access is rarely required)
- advanced database tuning/optimisation - this can only be done via Database root level access - often not required but advance managed DB systems do provide many of these tuning parameters
- it's usually vendor that requests this level of access but not what your business might need
- you might want to run a database version that AWS doesn't provide
- specific OS/Database combination that AWS doesn't provide
- some sort of architecture that AWS doesn't provide (resilience/replication)
- Sometimes you just want your own EC2 instance without much justification

**Why you shouldn't do this...**

- Admin Overhead - managing EC2 and DBHost
- Backup / DR Management 
- If you use EC2 you are bound to the AZ EC2 is in - if that AZ fails your database fails 
- Features - some of AWS DB products are amazing
- EC2 is ON or OFF - no serverless, no easy scaling
- Replication - skills, setup time, monitoring and effectiveness
- Performance... AWS managed DB are highly optimised and features rich to support high level performance

## Relational Database Service (RDS)

it's a Database as a service (DaaS) product. To be more precise you can call it DatabaseServer-as-a-service

What you get is a managed database instance and within it, you can have one or many databases

AWS handles all the admin overhead, so you don't have to worry about patches, physical hardware, or server OS or the Database system itself

With RDS you can have pretty much all the popular database you can think of such as:

- MySQL
- MariaDB
- PostgreSQL
- Oracle
- Microsoft SQL Server

you can pick whichever database instance you need. They all come with their own features and limitations. So you should be aware of those. No need to understand that in detail for the exam.

You can also have amazon's own Aurora database system with RDS but it's soo different then those mentioned above so we'll discuss it separately

**Architecture**

- DB instance
  - When you provision the database instance you get one database with it
  - but you can create more databases within this instance later on
  - These will be called user created databases
- Once you have created a database, you access it using database host-name, a **CNAME**
  - this is the only wat to connect to database - you can't use IP address to connect to database 
- You can use the same tools you would use to access the RDS database like you would use for same database self managed in an EC2 instance. For example sql developer or DBeaver
- comes in various types and sized(CPU and Memory) but with prefix db
  - db.m5 - general purpose
  - db.r5 - memory optimised
  - db.t3 - burst instance type
- Comes with single or multi-AZ (two instances work together to provide failover support)
- when you provision rds instance you also allocate the dedicated storage to that instance
  - the storage is just a block storage - it's essentially EBS storage which is also located in the same AZ as the instance 
  - The dedicated EBS storage could be:
    - io1 - high end performance - lots of IOPS and consistent low latency
    - gp2 - usually the default
    - Magnetic - not usually the default but it's available if your setup is historically used magnetic type storage
- Billing 
  - RDS instance hourly rate for the compute the instance on (CPU/Memory)
  - GB/month - billed for the amount of storage allocated for RDS - just like EC2 EBS volume if you allocate 50GB then you will be charged for 50 GB per month.
- Security 
  - Access to RDS instance is controlled by the security group that's attached to RDS instance

## RDS High-Availability (Multi AZ)

RDS supports MultiAZ to provide resilience 

**Architecture**

- You can have your primary RDS instance in AZ-A and attach an EBS storage with it in the that AZ. 
- Then, you can enable synchronous Replication and have another RDS instance on standby in AZ-B and attach an EBS with it in that AZ.
- you can only access the primary RDS using CNAME. the standby RDS is just there to receive updates from the primary RDS
- When you write data in your primary RDS, the moment it commits that write in its attached EBS volume, the commands to write the same data is issues to the standby RDS and it then also commits to the write to its attached EBS storage
- This means there is hardly any lag between Primary and standby. This type of replication is called synchronous replication

**Failover**

- if for any reason the primary RDS fails then immediately the traffic is redirected to standby RDS. This is done by RDS itself by changing the CNAME.
- users might get interrupted but it only lasts for 60 to 120 seconds
- This means Multi AZ RDS only provide high availability not fault tolerance 

**Exam PowerUp!**

- RDS Multi AZ is not available in free-tier - it costs extra for standby replica
- It generally costs two times the price of single RDS
- Standby replica can not be directly accessed - ALL RDS access is via single database instance CNAME
- Standby is only accessed in case of primary RDS fails. Standby does not provide any profmormance benefits or load balancing etc. it's only to improve availability not performance
- Failover takes about 60 to 120 seconds - it's highly available not fault tolerant
- RDS Multi AZ is same region only - meaning another subnet in another AZ in the same VPC
- Backups can be taken from the standby so no performance impact on the primary RDS
- if you have only one instance of RDS and you are also taking backup from it then it can cause availability and performance related issues 
- failure can happen due to full AZ outage so RDS will failover to standby replica in another AZ not affected by the outage
- if primary instance fails the AWS will detect it and initiate failover 
- You can also do manual failover to test performance issues
- if you change the type of RDS instance(for example you want to get a larger RDS instance)  it will failover as it will try to minimise disruption

##  RDS Automatic Backup, RDS Snapshots and Restore 

### RPO(Recovery Point Objective) 

- Time between a failure and the last backup
- the lower the value costlier the solution
- if the failure occurred at 11 am and your last successful backup was t 3 am that means it's an eight hour RPO. 
- if your backup failed then the RPO could be much more longer than 8 hours
- Lowering RPO means more frequent backups or some form of replication

### RTO(Recovery Time Objective)

- RTO is the time between a failure and the time system is returned to service
- this is influenced by process, staff, tech and documentation
- generally lower values cost more
- so if your failure occurred at 11am and you restore your system at 5 pm that means your RTO is 6 hours

### RDS Backups

**Manual Snapshots**

- Snapshots are manual and you have to run them against an RDS instance
- these are again stored in S3 managed by AWS
- the first snapshot is full copy and then incremental onward
- for each snapshot there is a brief interruption to the flow of data b/w the compute resource and storage 
- if you are using single AZ, this can impact your application but in multi-AZ setup no interruption occurs as the replica RDS is used 
- the initial full copy of the snapshot will take time but incremental will be much quicker
- Manual snapshots don't expire - since you take these snapshots manually so you have to delete them. They will never expire until you delete them
- This means even if you delete your RDS instance, the manual snapshots will remain available in your AWS account 
- When you delete an RDS instance, it will offer you to make one final snapshot for you - Snapshots are taken of the RDS instance storage - that means that the snapshots will contain all the databases inside the RDS instance
- you can run manual snapshot one per week, per month, per hour - it's up to you as they are fully manual
- the more regular snapshots are taken to lower the RPO value will be

**Automatic Backups**

- Automatic backups are just like manual snapshots but they are taken automatically. 
- you can define how often you want these automatic backups taken
- just like with manual snapshots, the first back is full copy and then onwards it's incremental
- you have to define when you want to take these backups - if you have single AZ setup then it's best to chose a time when your business isn't using database actively - such as 3 am in the morning
- but whatever window you chose, it impacts RPO - RPO is a time between a failure and last successful backup 
- In addition to these automated backups, RDS also writes transaction logs every 5 minutes to S3
- transaction logs contain actual data that's changing inside the database - meaning if you ran an update command to update a record the transaction logs will contain this operation with the data. 
- this means  you can you get your database restored within that would include these transaction logs meaning your RPO could theoretically be 5 min  
- The way it works is that the database backup is restored and then the transaction logs are applied on top of the restored backup - it's very powerful and provides very low RPO value
- automatic backups aren't retained indefinitely, they are automatically cleaned up
- You can set the retention period from 0 to 35 days - 0 means automated backups are disabled
- 35 days means you can restore your database at any point in those 35 days - using both the snapshots and transaction logs
- when you deleted the database you can chose to retain automated backups - but they still expire based on the retention period
- only way you keep backups indefinitely is by creating a final backup/snapshot when you delete the RDS instance and it's only you who can delete it. All other automated backups have retention period applied. 
- storing data in S3 means it's region resilient - as it's replicated across AZs in that region

### Exam PowerUps!

- when you perform a restore the RDS creates a new RDS instance and this new RDS instance will have a new endpoint address(CNAME)
- You will need to update your application to use this new CNAME as it will be different from the original RDS instance
- When you are restoring via manual snapshot it's fixed at a single point in time. Meaning it will be restored to the same time when the snapshot was successfully taken. 
- This means it influences the RPO value
- With automated backup the RPO can be minimised to 5 min only. it's up to you to chose the restore point
- When backup is restored from automated backup - the backup is first restored and then transaction logs are applied/replayed to bring DB to desired point in time. 
- Restoring backups is not a fast process - think of RTO - it can take a very long time if you have a very large database

IMPORTANT: With replication you can potentially replicate if you have corrupt data but with snapshots you can go back to a time before the corruption happened 

## RDS Read-Replicas

- copy data only when the primary RDS has already written to the disk - unlike Synchronous replica when write occurs at the same time as primary RDS. This type of replica is called asynchronous replica 
- Because of this there is always a lag with read replica - depends on how many time write is happening on primary instance and network
- you can have read replica within same region or a different region in AWS

**Why Read Replica?**

- Performance Improvements

  - you can have 5 direct read-replicas per DB instance
  - each provides additional instance of read performance
  - it is a way to scale out read capacity for a database
  - you can use read-replicas for your application when you need to read data and use the primary RDS for write only operations
  - so use Multi AZ to provide availability benefits and to remove issues with backups impacting performance and use read replicas to scale out your reads
  - you can also configure read-replicas of read-replicas but then lag starts to become a problem
  - it can also provide global performance improvements 
  - Remember: it can not be used for write operations

- Availability Improvements

  - snapshots and backups are great to improve RPO but RTO remains a problem
  - Read Replica offer near 0 RPO
  - if your Primary RDS fails then you can promote your Read Replica to become a new primary instance - this offers a really low RTO value 
  - You can generally promote read-replica to be a primary instance within minutes
  - Remember it only works on the failure of Primary RDS 
  - if due to malware or data is corrupted then read replica will so replicate that corruption so you can't rely on read replica for RTO and you will have to go back to snapshots and backups. 
  - read replicas are read only until they are promoted
  - once promoted it is not reversible. you have to delete it and create new read replica 
  - it provides global availability improvements and global resilience

- EXAM

  - Read Replica = asynchronous
  - you can have only 5 read-replicas
  - read replicas only provide read scaling for RDS instances - Read Replicas are how you scale read loads on your database- can not be used to write scale
  - you do not use multi-AZ for read scaling
  - Multi-AZ is only for availability 

## RDS Data Security

- **Transit** - SSL/TLS (in transit) is available for RDS - it can also be set to mandatory per user basis

- **Rest** - RDS support EBS volume encryption using KMS
- the encryption at rest is handled by Host/EBS - the database engine just writes unencrypted data to storage - the data is encrypted by the host of the RDS instance is running on
- KMS is used for encryption so you can use Customer Managed Key(CMK) or AWS managed keys.
- Then these CMKs are used to generate data encryption keys(DEK) which are then used to perform encryption 
- Using this method, Storage, Logs, Snapshots and replicas are all encrypted using the same customer master key.
- Encryption can not be removed once added - All these features are available in RDS as standard
- In addition to KMS, EBS-based encryption, MS SQL and Oracle also support TDE (Transparent Data Encryption)
- This TDE is supported and handled within the DB engine - meaning data is encrypted and decrypted by db engine and not by the host that instance is running on(RDS)
- RDS Oracle also supports integration with CloudHSM - with this the encryption is even more secure as it provides even stronger key controls as Cloud HSM is managed by you with no key exposure to AWS
- For demanding regulatory industry this CloudHSM support valuable 

**IAM Authentication for RDS**

- Usually logins to RDS are managed by local database users
- You can configure RDS to allow IAM user authentication against a database 
- First we can create a local DB user account and link it to use AWS Authentication Token to access RDS
- We can then either have a IAM user or instance role  and attached a policy to them
- this attached policy with wither role or IAM user contains a mapping to RDS local DB user
- This allows either the role or IAM user to run **generate-db-auth-token** operation that works with RDS and IAM
- Based on the policies attached to IAM user or role the token is generated with 15 min validity using generate-db-auth-token operation. This token can then be used to login as the database user in RDS without requiring a password. 
- REMEMBER: This is only authentication NOT authorisation. authorisation is managed by DB engine and are given to local DB users. 

## Aurora Architecture

- its different than other RDS DB engines
- It uses cluster as its core - A cluster is based on a single primary instance and 0 or more replicas
- Replicas in Aurora can be used for reads during normal operations
  - this means unlike RDS standby replica that only gets activated during failover - Aurora replica is used for reads during normal operations. So aurora automatically provides both Multi-AZ and RDS Read Replicas
- Second key difference is storage - it doesn't use local storage instead uses cluster volume
  - This storage is shared and available to all compute instances within a cluster
  - this means faster provisioning, improved availability and better performance
  - The replication happens at storage level - no extra resources needed by the instances during replication process 

- By default write is only performed by primary instance and replicas and primary can perform read operations 
- Because Aurora replicates data in at least 3 AZs - losing data due to failure of storage disk is greatly minimised
- Aurora also automatically repairs any disk damage and uses replica to recreate the data in damaged section of storage
- With Aurora you can have upto15 replicas and anyone can be failover target

**Things to remember**

- All shared storage in Aurora is SSD based - meaning high IOPS and low latency - you don't get the option to use magnetic storage 
- Billing for the storage is different then normal RDS engine - With Aurora you don't have to allocate the storage that the cluster uses
  - Storage is billed based on what you consumed
  - This is great as you don't have to pay for storage that remains empty in normal RDS instance
  - High Water Mark - billed for the most storage you consumed
  - for example if you consumed 50 GB you will be billed for 50 GB but if you move down to 40 GB then you will continue to be billed for 50 GB. 
  - To get rid of paying for extra 10 GB you can create a new aurora cluster and migrate data from old to new cluster
  - Since the storage is for cluster not instances so replicas can be added or removed without the need for storage provisioning 

- Access to Aurora - Like RDS, Aurora uses endpoints - these are DNS addresses that are used to connect the cluster. 
  - Unlike RDS, with Aurora you have multiple endpoints
  - You have Cluster Endpoint  and Reader Endpoint
  - Cluster Endpoint always point to the Primary instance - can be used for Read and Write operations
  - The Reader Endpoint will point to primary instance if that's all you have otherwise it will load balance with all available replicas - this can only be used for read operations
  - This makes Read scaling so much easier vs RDS
  - In addition to above  you can also have custom end points 
  - and also you can you have primary and replicas their own unique endpoints 
  - this means there is a greater flexibility with endpoints compared to normal RDS

**Cost**

- no free-tier options
- this is because aurora doesn't support Micro Instances
- Other than RDS singleAZ(micro) Aurora offers better value
- Compute - hourly charged, per second - 10 min minimum
- Storage - GB-Month consumed(remember high water mark) also add IO cost per request. 
- 100% DB size in backups are included
  - if your DB consuming 100 GB then you are given 100 GB of storage for backups as part of what you pay for that cluster

**Restore, Clone and Backtrack**

- Backups work the same way as RDS
- Restore will create a brand new cluster 
- Backtrack can be used which will take your database back to a previous point in time
  - It needs to be enabled per cluster basis and you can adjust the window that backtrack will work for
  - lets say you know a corruption happened in database at some point 
  - you can roll back to a previous point in time in the same cluster with Backtrack rather than creating a brand new cluster
- You can also make fast clones
  - it references with original storage and only stores any differences between those two.
  - it only stores changes between the source data and the clone 

## Aurora Serverless

- it provides a version of Aurora database product where you don't have to worry about provisioning database instance of a certain size or worry about managing those database instances
- it's just another step closer to database as a service

**Architecture**

- you still create an Aurora cluster but Aurora serverless uses the concept of ACUs(Aurora Capacity Units)
- Capacity Units = Certain amount of Compute  and memory 
- you can set Minimum and Maximum ACU values and Aurora Serverless will scale between those values
- Cluster is adjusted based on load
- it can even go down to 0 or paused - meaning you are only billed for the storage cluster used
- billing is based on per-seconds resources consumed
- provides same level of resilience as Aurora Provisioned - (6 copies across AZs)
- easier to manage - less complex and easily scalable. Increases ACUs seamlessly as and when needed with no disruption to client connections
- most cost effective - you only pay for database resources you consume on per second basis. Unlike Aurora provisioned you are charged for resources whether you are consuming or not. 

**Architecture**

- Cluster still exists but in the form of serverless cluster
- instead of having provisioned servers, we have ACUs
- ACUs are allocated when needed rapidly - These ACUS are stateless which are shared across many AWS customers and they have no local storage
- These ACUs are made available from warm pool of Capacity Units managed by AWS
- once they are allocated to serverless Aurora cluster, they get access to cluster storage
- If your serverless require more than the max ACUS specified, then bigger ACU will be allocated to your serverless and those small ones that are no longer needed will be de-allocated
- **Proxy Fleet** - This is a shared proxy fleet that manages connections to Aurora serverless on your behalf
  - if a user interacts with the cluster via an application, it actually goes via this proxy fleet
  - any of the proxy fleet instances can be used - they will establish a connection between application and Aurora Capacity Units
  - Because the client application never connects directly to ACU the scalability can be fluid
  - Client application only connects via instance in proxy fleet
  - This proxy fleet is managed by AWS and you don't have to worry about it
  - all you have to do is select minimum and maximum ACU
- You are only billed for the amount of ACUs you are using at a particular point in time and the cluster storage

**Use Cases**

- Infrequently used applications - blog posts
- New applications where you are unsure about the levels of load and size of DB going to be on application
- variable workload - if you have an application where you get traffic at certain time of the week or month then again serverless is a good option as it can scale based on demand. and you can save money if you provisioned static capacity such as aurora provisioned
- unpredictable workloads - you can set maximum fairly high when you don't know the workload capacity then you can monitor the workload 
- Development and test databases - very cost effective . it can be configured to pause(meaning scale to zero) the cluster when no development is taking place - you are then only billed for storage
- Multi-tenant applications - where you are billing a user set amount of money per month per license. if your incoming load  is aligned with incoming revenue then serverless is perfect

##     Aurora Global Database

- With aurora you can have 1 Read/Write and 15 read only replicas
- Global database allows you to create global level of replication using Aurora from a Master Region to five secondary AWS regions
- replication in global aurora database occurs 1 or less second between regions
  - it's only one way replication - from primary to secondary - it's not bidirectional
  - replication has no impact on DB performance as it occurs at storage layer
  - Secondary regions can have 16 replicas
  - all of them can be promoted to Read/Write - in case of any disaster situation
  - Currently there are MAX 5 secondary regions
- great for cross region Disaster Recovery and business continuity
- so if one of the regions goes down then you can promote secondary region for read and write operations while the primary region recovers
- With Aurora Global database, you can keep your RTO and RPO values really low in case of failover
- great for global read scaling - low latency performance improvement for international customers

## Multi-master writes

- Default aurora mode is Single Master means you can have 1 read/write node and 0 to 15 read only replicas
- In Aurora Cluster Endpoint is used for Read and Write and then we have read endpoints that are used for load balanced reads
- Failover takes time and you need to promote a replica to perform Read/Write operations
- in multi-master mode all instances are capable of both Read/Write operations - so failover doesn't take time in multi master cluster as all instances are capable of read/write
- There is no cluster endpoint to use - Application can connect to one or more instances within cluster and there is no concept of load balancing 
- No load-balanced endpoint in multi master 
- when a write request is received by the application the node sends the message to all storage to commit the write - Other nodes have to agree - in case there is a conflict with other operation then an error is generated but in case of no conflict, the write is committed to storage
- one the data is written in storage it is also replicated to multi-node instances cache - this means that any reads from those instances will be consistent with the data stored in shared storage
- in an event of failure, with Single master it takes time to be redirected to functioning ready only node that gets promoted to read and write. This transition isn't quick and takes time. quicker than normal RDS but it takes time
- in an event of failure with multi master, if you have two nodes then both of them can read and write. If you have designed your application in a way that you can connect to both nodes at the same time and if one fails then your application can continue to function with little to no disruption. 
  - it's almost close to fault tolerant system 

**Benefits**

- better and much faster availability
- failover events can be performed inside application and it doesn't need to disrupt traffic 
- it can be used to implement fault tolerance but the application will need to manage load balance manually across instances - it's not something that's handled by cluster

##     Database Migration Service (DMS)

- it's a managed database migration service 
- it runs using a replication instance  - this replication instance runs one or more replication tasks 
- this replication tasks is where all the configurations are defined for migration of databases
  - one key configuration parameter you need to define is your source and destination endpoints
  - these source and target endpoints should point to physical source and target databases
- One restriction this service has is that at least one endpoint must be on AWS

**Architecture**

- you need a source and destination database - one of them must be in AWS
- databases can use range of database engines
- in between these two databases (source, dest) we have the Database migration service (DMS)
- it uses an EC2 instance that runs migration software and has the ability to communicate with DMS service
- inside this instance you can define replication tasks - you can have one or more replication tasks - these tasks define migration configuration
- most important thing to remember is the source and destination endpoints (these endpoints contains the connection details of source and destination databases)
  - so replication instance and tasks can access both physical databases
- DMS provides three types of jobs
  - Full Load - one off migration of all data - if you have the option to have an outage while this migration happens then this is a good option to use 
  - Full load + CDC (change data capture ) - it does full data load transfer but while this transfer is happening it also captures all the changes to the database. so once the full data load is migrated then those changes captured can be applied. Eventually  you will get to a point when source and target are in sync 
    - No downtime migration with no data loss
  - CDC Only - allows you to use an external tool to migrate data from source to destination(sometimes vendor specific tool might be faster) and then you can use DMS to capture just the changes while migration is happening and it can then be applied later. 
- DMS doesn't natively provide any schema conversion but there is a tool called Schema Conversion Tool(SCT)  which can assist with schema conversions with migration

**Use Case**

- great tool to migrate database within AWS
- you can also migrate database from AWS to on-premise
- you can also migrate database from another cloud provider to AWS - as long as one endpoint is in AWS you can use DMS.
- EXAM - DMS provides little to no downtime migration

## 06 Network Storage

### Elastic File System (EFS) Architecture

- Your images or other content that you store on EC2 instance for your web application are gone the moment you lose access to your EC2 instance 
- EFS allows you to have a network storage that can be connected to multiple EC2 instances at once - this means if you lose access to EC2 instance your data is saved in your database and your content such as imaged or other media is saved in you EFS
- EFS is AWS implementation of fairly common shared storage standard called NFSv4
- With EFS you create a base file system that can be mounted within EC2 instances in Linux
- Linux uses tree structure for its file system 
  - Devices can be mounted in folders in hierarchy
  - NFS can also be mounted to a folder in Linux
- Interesting thing about EFS is that is can be mounted to multiple EC2 instances so data can be shared
- EBS is block storage - means you have to setup a file system of your choice on top of EBS
- EFS is a file storage which can be mounted to Linux as if they are connected directly on EC2 instance
- EFS is a private service - isolated to VPC it's provisioned into
- Access to EFS is via mount targets which are things inside vpc
- It can be accessed from on-premises -  VPN or DX (direct connect)(these hybrid networks are explained later in the course) - this just means that EFS can also be accessible outside VPC using hybrid network products

**Architecture**

- It runs inside VPC
- inside EFS you first create file system - these use POSIX permissions (see this link:https://en.wikipedia.org/wiki/File_system_permissions )  - its a standard for permissions that all Linux based OS understands
- EFS is then made available via Mount targets that run from inside subnets of VPC
  - These mount targets have IP addresses taken from the IP address available from the subnets they are inside
  - to ensure high availability put these mount targets in multiple availability zones
  - it is these mount targets that EC2 instances connect to
  - Using VPN and Direct connect the on-premise network can connect to these mount targets and access EFS file systems

**EXAM**

- it's only for Linux instances
- EFS offers two modes General Purpose and MAx I/O Performance Modes
  - General Purpose is good for latency sensitive use cases (web Servers, content management systems, home directories or general file serving)
  - General purpose is the default and is used 99.9% of the time
  - MAX IO can scale to higher level of aggregate throughput and operations per seconds but it has trade off of increased latencies - Mostly used for highly parallel applications (Big data, media processing, scientific analysis)
- EFI comes with two throughput  modes - Bursting and Provisioned 
  - Bursting mode works like GP2 volumes work inside EBS (it's like GP2 vs IO)
    - Throughout scales with the size of file system - the more data you store in file system the better performance you get
  - With Provisioned you can specify throughput requirements separately from size of EFS but generally you should pick bursting by default
- EFS comes with two storage classes: 
  - Standard - Default - more expensive then IA
  - Infrequent Access - less expensive - used where you don't need data frequently
- Like in S3 you can use Lifecycle policies to move data between classes



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





​	





