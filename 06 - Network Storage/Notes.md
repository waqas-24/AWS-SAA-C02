## Network Storage

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
  -  MAX IO can scale to higher level of aggregate throughput and operations per seconds but it has trade off of increased latencies - Mostly used for highly parallel applications (Big data, media processing, scientific analysis)
- EFI comes with two throughput  modes - Bursting and Provisioned 
  - Bursting mode works like GP2 volumes work inside EBS (it's like GP2 vs IO)
    - Throughout scales with the size of file system - the more data you store in file system the better performance you get
  - With Provisioned you can specify throughput requirements separately from size of EFS but generally you should pick bursting by default
- EFS comes with two storage classes: 
  - Standard - Default - more expensive then IA
  - Infrequent Access - less expensive - used where you don't need data frequently
- Like in S3 you can use Lifecycle policies to move data between classes

