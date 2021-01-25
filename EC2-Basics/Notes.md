## Elastic Block Store (EBS) Service Architecture
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
  - On top of baseline you also get **3 IOPS per GB per Second**
  - This means 100 GB volume gets you 300 IO credits per second
  - You only get baseline IOPS if size of volume is below **33.33 GiB**
  
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

