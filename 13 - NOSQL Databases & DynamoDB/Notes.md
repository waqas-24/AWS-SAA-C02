# 13 NOSQL Databases & DynamoDB

## DynamoDB - Architecture 

- NoSQL - Wide Column - Database as a Service Product - Key/Value & Document -
- Usually used for serverless or web scale traditional applications in AWS
- Public Service - With access to the public endpoints of DynamoDB
  - Can be access via Public Internet 
  - Via VPC with Gateway VPC Endpoint or Internet Gateway
- Capable of handling Key Value or Key Document like DocumentDB model
- No Admin Overhead - You have no self management to worry about like with RDS or Aurora or Aurora Serverless
- Scaling - you have two options 
  - you can either go with Manual / Automatic provisioned performance 
  - On-Demand 
- it's highly resilient service across AZs and optionally global 
- by default data is replicated in multiple nodes so you don't need to explicitly handle it like you do with RDS
- DynamoDB is backed with SSDs so it's really really fast and provides single digit millisecond access to your data
- provides backups, point-in-time recovery and encryption at rest
- Event-Driven integration ... you can do things if data changes
- **Base Entity** in DynamoDB is tables - No limit of number of rows in Table
- Primary Key - Have to be unique
  - Simple Primary Key - it's also called the partition Key - has to be unique
  - Composite Key - Partition and Sort Key - Both keys combined has to be unique
- A row can be maximum of 400 KB in size - includes primary key, attribute values and name of attributes
- for attributes in the table there is no fixed schema to follow - you can have any type of values for each attribute 
- Capacity - means speed not storage
  - provisioned - you set the capacity value per table basis
    - Writes 1 WCU(write capacity Unit) = 1KB per second 
    - Reads 1 RCU(Read Capacity Unit) = 4 KB per second
  - on-Demand capacity
- Backups
  - On-Demand Backups => Full on-demand backup of table retained  until removed
    - this on-demand backup can be restore either same or different regions 
    - you can restore backup with or without indexes
    - you can adjust encryption settings as part of restore
    - main thing is you are responsible for taking and restoring and managing the backups 
  - Point in Time Recovery (PITR)
    - Not enabled by default
    - Continuous record of changes allows replay to any time in the 35 days recovery window
    - you can use this PITR to restore the table with 1 second granularity
- Considerations
  - Any question in Exam related to NoSQL default to DynamoDB 
  - any question related to relational data - generally NOT DynamoDB
  - any question with Key/Value default to DynamoDB
  - Accessed via console, ALI, API **NO SQL like query option**
  - **Billed** based on RCU, WC values you set on the table, Storage required for the table and any additional features
    - you can also buy reserved capacity if you are certain you need long-term capacity on DynamoDB table

## DynamoDB - Operations, Consistency and Performance

- You can select two capacity modes when you create DynamoDB table (With some restrictions you can switch b/w these modes even when you have data in table)
- On-Demand
  - you use this when you have unpredictable work load and you don't want any admin overhead
  - you pay per million read or write units
  - The price you pay using on-demand model could 5 times higher than provisioned
- Provisioned
  - you set the capacity Read and Write per table 
- Every operation consumes at least 1 RCU/WCU
- 1 RCU is 1x 4KB read operation **per second**
  - you can have maximum of 400 KB of data per row in DynamoDB so you will need 100 RCU to read all that data in one operation
  - if you read 1 KB of data you still consume 1 RCU
- 1 WCU is 1x1KB write operation **per second**
- Every table has a RCU and WCU burst pool
  - this gives you 300 Read and write capacity units per table 
  - Try to be careful when you use this pool as other operations might also be using it and if you deplete the pool and have insufficient capacity units set on the table then you will receive provisioned throughput exceeded exception 
  - you will then have to wait for the pool to become available or add more capacity units
- **Query**
  - it is used to return data and you have to provide PK or PK as well as Sort Key to get the data back
  - if your query results in 2 records - one with 2.5K and another 1.5 K that means in total you returned 4K - means you consumed 1 RCU
  - if you query both records separately returning 2.5 KB and 1.5 KB then you would be consuming 2 RCU - it's always a good idea to bring more data and then filter it yourself at application level
  - you can use either Partition Key (PK) or PK and Sort Key(SK) to query the data
- **Scan**
  - it is least efficient but most flexible operation in DynamoDB
  - you are charged for each row that is scanned through Scan operation
  - unlike query where you have to provide PK or PK and SK, here you can pick any attribute and say bring me all rows that have a particular value. 
  - Scan will need to scan through all of your table and bring your all the rows with specified value but this means all the rows scanned will cost you for RCU
  - for example you have 4 rows in total:
    - Row 1 5K 
    - Row 2 4K
    - Row 3 2K
    - Row4 3K 
      - Your scan will consume 5K + 4K + 2K + 3K = 14K = 4RCU
      - but it will only return 5K+ 4K of data meaning only row 1 and 2

**2 Consistency Mode** 

- **Eventually Consistent**
  - Easier to implement and scale 
  - with this model you can get double the amount of data with with the same prove this means 8K will   use 1 RCU rather than 4 but the data might not be consistent

- **Strongly or Immediately Consistent** 
  - in some applications it's essential but it's more costly to achieve and difficult to scale 

- DynamoDB replicates data in multiple AZ - For example if it's replicating in 3 AZ then it will elect on of the AZ as it's leader Node. 
- This leader node is where you write the data 
- If this leader node fails, then another node becomes the leader
- With strongly consistent model, you always read from the leader node
- With eventually consistent mode you could be directed to read from a node that's not fully up to date
- If your application can tolerate some lag to updated data then go with Eventually consistent model and you will be able to save 50% of the money
- but if you have a critical application and you are using DynamoDB to store important health related or other critical data such as stocks then you can't tolerate inconsistently and you will need to select the strong consistent model 

**WCU Calculations**

- if you have 10 items of 2.5K per second to write then
- WCU per item = Round Up(Item Size / 1KB) 
- WCU per item = Round Up (2.5KB / 1KB)  
- WCU per item = Round Up (2.5KB )  
- WCU per item = 3
- Now we have 10 items per second to write so = 10 x WCU per Item
- Now we have 10 items per second to write so = 10 x 3
- Now we have 10 items per second to write so = 30
- Total WCU Required = 30

**RCU Calculations**

- if you have 10 items of 2.5K per second to read then
- RCU per item = Round Up(Item Size / 4KB) 
- RCU per item = Round Up(2.5 KB / 4KB) 
- RCU per item = Round Up(0.625)
- RCU per item = 1
- Now we have 10 items per second to read so = 10 x RCU per Item
- Now we have 10 items per second to read so = 10 x 1
- Total RCU need for 2.5KB 10 times per second = 10
- Since Eventually consistent is 50% of Strongly consistent so Eventually consistent will only need **5RCU** for the same operation.

## DynamoDB - Streams & Lambda Triggers

- With DynamoDB you can enable Streams on your table to use Trigger functionality based on changes on the table 
- Once Stream is enabled it captures Delete, Update and Insert changes on DynamoDB Table for 24 Hour window and it provides 4 types of view
  - Key only - provides you the PK or PK and SK of the record where change occurred
  - Old Image - Provide only the previous state of record
  - New Image - Provides only the new record after change
  - Old and New Image - Provides both new records and old record
- Lambda Function is invoked when stream event occurs -  Stream view data is passed to Lambda function as event 
- Based on this, you can take action via Lambda
- you can use this trigger/event based architecture in Reporting and Analytics scenarios
- good for data aggregation - vote aggregation - use it for messaging or notifications
  - For example if you have a messaging app then stream and lambda can be used to send push notification based on any changes to your table 
  - This way you won't have to poll the database for changes as that uses compute power you can instead use stream and lambda to develop event driven architecture

## DynamoDB Local and Global Secondary Indexes

**Local Secondary Index** - SAME PK but different Sort Key

- Its an alternate view of your base table in DynamoDB where you can have an alternative Sort Key on the table
- LSI must be created when you create the table - you can not add LSI once table has been created
- You can have maximum of 5 LSI created per base table
- They share the same RCU and WCU values that you have allocated to the base table if it's using provisioned capacity
- It allows you to make one of the attributes as Sort Key so you can avoid using Scan operation and instead use Query which is more efficient
- It's up to you which attributes you include in the LSI View - you can have all attributes or Keys_Only or you can pick which attributes to be added
- Indexes are sparse - only items which have a value in the index are added to the view - meaning you those records where your index have no value will be discarded
- Keep in mind that Query operation can only work with 1 Partition Key(PK) and one or a range of Sort Key(SK)

**Global Secondary Index** - Different Partition and Sort Key

- They can be created anytime -
  - unlike LSI that have to be created when you create the table 
- By Default you can have maximum 20 GSI per base table
- GSI let you define alternative PK and SK
- They have their own RCU and WCU allocations - They don't share it with the base table
- Just like LSI, you can select which attributes you want in your GSI view. you can wither have all attributes, Keys only or include attributes of your choice
- Just like LSI, GSI are also sparse, only rows that have values in them are added to the value the rest are discarded
- GSI are **always** eventually consistent, replication between base table and GSI is asynchronous

**GLSI and GSI Considerations**

- Careful with projection (Keys_Only, Include, All)
  - Make sure you include the right attributes in the view as it impacts RCU and WCU 
  - Also if you query needs an attribute that's not part of the view then it will still work but it'll be highly inefficient
  - Plan your indexes/views in advance to make sure right attributes are added
- Use GSI as default as they are more flexible and can be created after you have created your base table
- Only use LSI if you need strong consistency
- Use indexes for alternative access patterns

## DynamoDB - Global Tables

- Provide multi Master cross-region replication
- Create tables in multiple regions and add them to the same global table
- if same data written in two different tables then last Writer wins
- Reads and Writes can occur in any region
- Generally sub-second replication between regions
- Strong consistent reads ONLY in the same region as writes but eventually consistent in other regions  
- Application needs to take into account that the Global tables will be eventual consistent if you are reading from a region just when writes occurred in another region
- **Consistency**: Globally it's always eventual consistency but you can have eventual or strongly consistent within a region 
- Multi-Master - tables can be used for Read and Write
- Use cases
  - Globally Highly Available Application
  - Improve global data performance of an application
  - add Global Disaster Recovery or business continuity to your application 
- Conflict Resolution: Last writer wins
- Sub-Seconds Replication between table replicas

## DynamoDB - Accelerator (DAX)

- DAX provides in-memory cache for DynamoDB to accelerate to data read 
- Using Traditional Cache you have to make a call to the cache to check if the data is there, if the data isn't there then you will need to make another call to the database to get the data and the cache is also updated
- With DAX you can use DAX SDK to make a single call to get the data - DAX is integrated with DynamoDB so the DAX call will return the data from its cache or retrieves it from the DynamoDB table and then caches it
- DAX provides less complexity for app developer with tighter integration
- DAX is implemented in the form of a cluster 
  - you get a Primary Node that Writes and replicas that can provide Read operations
- Nodes are implemented to provide High Availability - if the primary node fails then election is conducted and a new primary is selected
- DAX in an In-Memory cache - this means it provides much faster reads and reduce costs as you are not going directly to DynamoDB time after time for same data
  - With DynamoDB you get data back in single digit millisecond
  - with DAX you get it in micro seconds
- You can scale **UP** and Scale **OUT** meaning you can either add bigger instances or more instances
- Supports write-through - Data is written to DDB table and then DAX
- DAX is accessed via an endpoint. Cache HITS returned microsecond and Misses in milliseconds
- Item cache holds results of (Batch) Getitem. The query cache holds data based on query/scan parameters
- DynamoDB is a public service but DAX is deployed WITHIN a VPC so any application using DAX needs to be deployed inside that VPC
- You are reading same data again and again then consider using DAX
- You are having performance issues during heavy work loads then consider using DAX
- if you want incredibly low response time then consider using DAX
- if you are interested only one specific data type than others then consider using DAX 

**When you DON'T need DAX**

- DAX is not ideal if you want strongly consistent reads - if your app can not Telerate eventual consistency then DAX is not ideal
- If you don't need incredibly low response time then DAX might not be a good solution
- if your app is write-heavy and infrequently uses read operations then DAX is probably not the right solution

**Exam** if you are asked about cache related to DynamoDB then default to DAX 

## Amazon Athena

- It's serverless interactive querying service
- With Athena you can do ad-hoc query on structured, semi-structured or unstructured data stored in S3
- The source data in S3 is never changed when using Athena
- Athena can read various data file types such as XML, JSON, CSV/TSV, AVRO, PARQUET, ORC, Apache, Cloudtrail and VPC FlowLogs
- You use schema-on-read to define table like structure in advance and then you run queries on those tables via SQL-Like queries on data without transforming the original data
- Output of data can be sent to visualising tools and Athena queries can be run in an event-driven, fully serverless way.
- Billed based on data consumed during query
- **EXAM**: If you see a question where data is being stored in S3 and data is structured, semi-structured or unstructured and you need to perform ad-hoc queries where you are only charged for performing those queries then default to Athena 

## ElastiCache

- In-memory database ... High performance
- you can use Redis or Memcached as a service 
- Used for Read heavy workloads
- Reduces database reads
- it can be used to store session data (to make your application Stateless)
- It requires you to change your application code - you can just use it to plug it into your application

## Redshift Architecture

- Petabyte-scale Column Based Data Warehouse
- OLAP(online Analytical Processing) - Column Based not OLTP (row/transactions) - inserts, updates and deletes
  - OLAP is designed for complex queries to analyse aggregated historical data from OLTP systems 
  - so OLTP systems put their data in OLAP systems - so RDS might put data into Redshift  for more detailed long-terms analysis and trending 
  - Redshift stores data in columns
  - in OLTP you store data by rows - this means if a customer has placed an order then you would put order number, product, customer name and address all in one row
  - and if you want to look into just one field such as product then you would have to read all records
  - This is not the case with column based OLAP database such as Redshift - it makes reporting style queries much easier and more efficient to process
- RDS is provisioned just like RDS and it comes with advance features
- **Redshift Spectrum**
  - it allows querying data on S3 without leading it in Redshift in advance
  - You still need Redshift Cluster in place to use this but you won't have to go through time consuming exercise to load data into redshift before you query
- **Federated Query**
  - allows you to query remote databases using Federated Query
- Integrates with AWS tools such as Quicksight for visualisation
- it also has SQL-like interface that you can connect via JDBC/ODBC
- It's is server based product - not like Athena that's serverless
  - Athena is best suited for ad-hoc queries as you don't need to provision anything but with Redshift you still have to provision Redshift Cluster that takes time so for ad-hoc queries for data stored in S3 default to Athena
- Redshift uses Cluster Architecture which is a private network
- One AZ in a VPC - Network cost/performance - not High Available by design
- All Redshift Clusters have a **Leader Node** and that's the node that you use to Query, planning and aggregation
- This Leader node develops execution plans to carry out complex queries
- **Compute Node** - run queries assigned to them by the leader Node - They also store the data loaded into the system
  - A Compute Node is partitioned into slices - each slice is allocated a portion of Node's memory and disk space
  - The Leader Node manages the distribution of the data to the slices and portions of the workload for any queries 
  - Slices then work in parallel to complete the operation
  - Compute Node can have 2 4 16 or 32 Slices - depends on the resource capacity of that node
- Redshift is a VPV service so all parts can be managed including VPC security, IAM Permissions, KMS at rest encryption, CW Monitoring
- **EXAM Redshift Enhanced VPC Routing** - VPC Networking
  - By default it uses public routes when communicating with external services or any services like S3
  - When you enable Enhanced VPC Routing , the traffic is routed based on your VPC networking configuration 
  - This means it can be controlled by security groups, Network Access Control List, custom DNS and it will require the use of VPC gateways to reach external services
  - so if you want advance networking controls then you need to enable Enhanced VPC routing

**Backups**

- Automatic snapshots to S3 (ever 8 hours or 5 GB of data) with configurable retention period - default 1 day but can be configured up to 35 days - you get the backup capacity = to the storage capacity of your cluster for free included in the price of cluster
- Manual snapshots to S3 managed by you - they are retained indefinitely until removed
- You can also configure to copy snapshots to another AWS region for global resilience and you can spin up another cluster in another region in an event of DR or in another AZ

**Getting Data into Redshift**

- You can copy data via S3
- You can copy data via DynamoDB
- You can use DMS( Database Migration Service) to copy data from SQL based systems such as RDS
- You can also use Kinesis Firehose to stream data into Redshift