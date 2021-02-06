# AWS-SAA-C02

These are my personal notes prepared for AWS Solutions Architect Associate(SAA) exam. All these notes are prepared using Adrian Cantrill's (SAA-C02) course and from [alozano-77](https://github.com/alozano-77)'s notes for the same SAA-C02 course. Learning Aids from [aws-sa-associate-saac02](https://github.com/acantril/aws-sa-associate-saac02). I have tried to make these notes as detailed as possible but there may be errors, so please purchase Adrian 's course to get the original and most up to date content https://learn.cantrill.io.

**Table of Contents**

[1 Elastic Compute Cloud - Basics](#1-Elastic-Block-Store-EBS-Service-Architecture)

[2 Introduction to Containers](#2-Introduction-to-Containers)

[3 Advanced EC2](#3-Bootstrapping-EC2-using-User-Data)

## 1 Elastic Block Store EBS Service Architecture 


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

## 2 Introduction to Containers

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

## 3 Bootstrapping EC2 using User Data

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





