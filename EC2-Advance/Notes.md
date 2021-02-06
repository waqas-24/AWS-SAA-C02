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

