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



























