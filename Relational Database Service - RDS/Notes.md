## Database Refresher

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