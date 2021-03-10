## 08 - Serverless & Application Services

## Architecture Evolution - Monolithic, Tiered, Queue, Microservices, Event Driven Architecture

**Solution Example:**

- Lets take an example of YouTube to understand various architecture

- YouTube allows you to upload videos, once uploaded it makes multiple versions of the video 1080p, 480p, 720p etc
- So this means You Tube has upload, processing and storing functionality in addition to website channels playlists etc

**Monolithic**

- With Monolithic architecture you have all the functionalities of the Solution in one place - this means you have one server that handles upload, processing and storage functionalities
- If any of these functionalities fail then the whole solution fails
- **Vertical scaling**: Just like the solution fails together as it's highly coupled architecture it also scales together
- Another key point to note is that you also get billed together - all the resources are always running and always incur charges - even if certain function of the solution isn't needed you need to maintain enough capacity all the time meaning you always have to pay for resources even if they are not consumed
  - Due to this, it's one of the least cost effective architecture

**Tiered Architecture**

- In tiered architecture we break the various functions of the solution into tiers
- these tiers can be on the same servers or different servers

- This tiered architecture is still tightly coupled together as they all need each other to provide the overall functionality but the benefit is that they all can be scaled independently of each other 
  - For example if the processing component of the application requires more CPU capacity then it can be increased in size independently of other components such as Upload or Store and Manage

- We can evolve these tiered architecture even more by adding load balancers in between 
  - This allows the application to scale horizontally and become highly available - you can add multiple instances for any functionality and link them via load balancers. If any of the instances fails, the load balancer will redistribute the traffic to a healthy instance
  - This also means that each tier is not directly connected to just one instance - it has no visibility of how many instances are running for e.g. for processing tier. As soon as any component of the solution requires anything it is provided via the load balancers

**Issues with Tiered Architecture**

Even though we have made tiers and added load balancers where the individual tiers can scale vertically and horizontally this architecture still has issues

The components such as Upload still need and expects processing component to respond

If for whatever reason the processing function fails then the upload feature will stop working or if there is a backlog in in processing component then it can also impact the upload tier

 Another problem with this architecture is that the processing tier has to be doing something and should be available. So this means that we can not scale down to zero when no processing is required as the communication between tiers is synchronous 

**Queue Based Architecture**

- Queue is a system that accepts messages 
- Messages can be received or pulled from the Queue
- In most cases there is ordering in the Queue system - mostly used is called FIFO (First In First Out) architecture but you can also change this
- Queue based architecture decouples our YouTube system
  - When a user uploads a video and it's complete, instead of passing the uploaded file directly to the processing tier, a message is generated
  - This message contains the location of the uploaded file and all other relevant information and then message is sent into the queue
  - Since this is the first message so it's at the front of the Queue
  - This also means that the upload tier now has no visibility of the processing tier nor it expects any message from processing tier - it's job is completed after uploaded file is stored and message is generated
  - With the help of Queue, the Upload and processing tiers are decoupled
  - This means the system has moved from Synchronous to Asynchronous style of communication
  - just like the first video, when more users upload their videos the messages are added to the queue
  - At the other end of the queue we have AutoScaling group that has minimum size of Zero, desired size of 0 and max 1337. This means that when there are no messages in the queue all instances are terminated and we are not billed for anything
  - We link the auto scaling policy with the length of the queue - which means the instances will automatically be provisioned or terminated based on the load in the queue
  - This is a very powerful asynchronous architecture as the components can scale independently and the processing tier uses worker-fleet architecture which means it can scale from 0 to infinity as it's linked to the load in the queue

**Microservice Architecture**

- if you keep breaking up the monolithic architecture into smaller pieces then you will end up with Microservice Architecture which is a collection of microservices
- Microservices perform individual things very well - meaning we can have a microservice for upload function, one microservice for processing and one for storing 
- For more complex applications you can have 100s or even 1000s of these microservices - Some might be different services or they can be copies of lots of same services
- A microservice is a tiny self sufficient application which has it's own logic, own store of data it's own input output components
- There are three types of Microservices
  - Producer - Upload service is the producer service
  - Consumer - processing is the consumer service
  - Both - Data, store and management microservice performs both
- Producers usually produce data or messages - Consumers consumes data or messages and lastly we have another type of service that can do both (consume and produce messages)
- Things that microservices produce or consumes are events
  - Queues can be used to communicate events
  - Larger microservice architecture can get complex pretty quickly - if your services need to exchange data between partner microservices - if we do this with queue architecture then we'll end up with a lot of queues - it can work but it can be very complicated

**Event Driven Architecture**

- **Event Producers**
  - these are just a collection of events producers that could be components of your application that interact with customers or they might be part of your infrastructure such as EC2 or they could be system monitoring components 
  - These are bit of software that produce events in reaction to something - for example if a customer clicks submit, then it could be an event - if a customer is doing something that generates an error that's also an event 
  - Producers are things that produce events
- **Event Consumers**
  - Inverse of event producers is Event Consumers
  - These are pieces of software which is ready and waiting for events to occur
  - if they see an event they care about, they will do something with that event - they will take an action - it could be displaying something for a customer or it might be to retry and upload
- **Consumers and Producers**
  - Components or services within application can be both producers or consumers
  - Sometimes a component might generate an event for example a failed upload and then consume events to force a retry of that upload
- Important thing to remember is that neither the producers or consumers are waiting around for things to occur - They are not constantly consuming any resources
- With producers, events are generated when something happens such as when a user clicks submit button to submit a form or if an error has occurred
- similarly, consumers are not waiting around for those event either. They have those events delivered and when they receive an event they take an action  and stop - this means they are not constantly consuming resources
- **Event Router**
  - It would be a really complex application if every service or components need to be aware of every other components or service or if every component/service required a queue between it and every other components to put events into (you can imagine they would be a huge number of queue)
  - Best practice for Event Driven System is called Event Router
  - it's a highly available, central exchange point for events
  - Event router has what's called **Event Bus** where information flows constantly
  - When items are produces by event producers they are added to the event bus via Event Router
  - Event router then deliver those to event consumers
  - Event driven architecture is unlike the wordpress EC2 instance which is always needed even if it's in use or not
  - Event driven architecture consume recourses as and when needed. So if there are no events to deal with then no resources should be in use
- **Key properties of Event Driven System**
  - No constant running or waiting for things
  - Producers generate events when something happens
    - example: clicks, errors, criteria met, uploads or actions
  - Events are delivered to Consumers that take necessary actions on those events and then the system returns to normal state where no constant resources are consumed- Delivery of events usually happens via **Events Router**
    - Event router decides which consumers to deliver events to
  - A mature and optimised event-driven architecture only consumes resources while handling events (they are like serverless)

##  AWS Lambda

- Lambda is a Function-as-a-Service (FaaS) product 
- this means you define a function(small piece of programming code) such as in Python 3.8 and give it to Lambda. Lambda runs that code and you get billed for the duration of the execution of that function
- Lambda is an example of Event-driven service - You invoke lambda either manually or by an event
- Lambda function  = only accepts piece of code in one language
- Lambda function use a runtime such as Python 3.6 - in order for the code to execute it needs a runtime
- The language you wrote the code in needs to match with the runtime in lambda
- Lambda runs in a runtime environment
- it's like a container for the specific language you wrote your code in
- in comparison with EC2 instance, with lambda you are only billed for the duration of lambda function
  - if you provided a python code to lambda that runs it in 5 seconds then you are only charged for that 5 seconds - there is no charge for creating a lambda function 
- Lambda is a key component of serverless architecture

**Architecture**

- when you are creating lambda function - you are actually creating a container and the most important part of that container is your code = function code
- Try to make your lambda function really small but really good at doing one specific thing
- Lambda function can be written in large range of different languages
  - Java - Node.js - Python etc
- When lambda function is invoked it runs your code in a runtime environment which has to match the language that the function is written in
  - For example, if you wrote your code in Python 3.8 then you will need to have Python 3.8 Runtime Environment to run your code
  - Runtime environment is like a little compute container which is provided with resources such as RAM and CPU
  - It's up to you how much memory you allocate - the more memory you allocate the more CPU you get 
  - The bigger the memory & CPU you allocate to your lambda function the more it costs you to run that lambda function every second
- When you create a lambda function you can also configure it to use a an IAM role to run it - this is knows as execution role
- this execution role is linked to the lambda function - meaning it is passed into the runtime environment  like EC2 instance role is passed into an instance
- So when this execution role runs the lambda function it will have all the permissions it is granted using permissions policy
- For example of you grant this execution role to access S3 bucket - you could write lambda function to save something in S3 using this role

- Lambda function can be triggered by an event for example an AWS service or a customer can trigger it via API Gateway or it can also be scheduled. Lambda function can also be executed manually 
- When lambda function is triggered, it is then physically downloaded to an AWS managed compute hardware and it is then executed on that hardware
- Once it's executed then the hardware is terminated
- You are only billed for the duration of that function runs
- A function can never run or only run 1 time or it could run a trillion times
- **Parallel Execution** - A function could just run one copy of itself or many many copies 
- Earlier when we talked about the video upload function - if this can be converted into a lambda function rather than using an EC2 instance then that's one use of Lambda function and it can easily scale
- **Highly Scalable  and Parallel** - every time you invoke the lambda function it uses a new and clean environment as the environment is stateless - it's not a persistent environment
-  You don't get any persistent storage with Lambda function
- By default Lambda function is a public service - this means that they have access to the public internet not resources inside VPC
- By default it can access internet based API services any public web resources
- But it can also be configured in a private VPC which then allows you to access resources attached to that VPC
  - if you use this option and also need to access public internet then you will have to configure your VPC in a way that it can access public internet access
- **Input & Output** - Since you have no persistent storage with lambda function - as it's stateless so you should get your data via public internet or AWS services for input and output
  - With Execution Role's permission policy you can have permissions to access AWS services such as DynamoDB and S3 for input output
- Whichever event trigger lambda function, the lambda function will get some meta data with it. For example if adding an object in S3 triggers a lambda function so it will get some meta data from the event that triggered lambda function
- **Important to remember**: Lambda function can run for less than a second or a min or 5 min or all the way up to 15 min.
- EXAM: Lambda function currently have 15 min execution limit 
  - in exam if they suggest to run a code for under 15 min then default to lambda first
  - it's much more economical compute service then running EC2
  - especially for things that don't need to be running all the time or for small bursty usage
  - if you don't need full access to OC and don't need persistent storage then lambda is ideal
- Currently 15 min execution limit
- new runtime environment every execution - no persistence
- Execution Role to run lambda and access AWS services/products via permissions policy
- Load data from other services either via internet or  AWS services and products 
- Save data to other services either via internet or  AWS services and products 
- Lambda comes under free tier - you get 1,000,000 requests and 400,000 GB per second compute time free per month
  - so you could run 1 GB of allocated Lambda function for 400,000 seconds per month - means you can run your code with 1 GB memory for 400,000 seconds each month for free
  - in most cases you don't use 1 GB ram for lambda function execution so you actually get a lot more seconds to run each month

## CloudWatch Events and EventBridge

- **Cloud Watch** - Monitors metrics and monitoring
- **CloudWatch Logs** - Ingestion and Management of logging data
- **CloudWatch Events** - delivers near real time stream of system events linked to supported AWS services and products (for example if an instance is stopped started or terminated - these generate events)
- **EventsBridge** - is a new service that's replacing CloudWatch Events. All same functionalities that CloudWatch Events has plus additional features
  - It can also handle events from third parties as well as custom application and not just AWS products and services
  - Underlying architecture of both CloudWatch Events and EventBridge is the same but AWS is now encouraging to start using EventBridge 

**Architecture**

- Both CloudWatch Events and EventBridge allows your develop/implement an architecture where CW Events/EventBridge can observe if X happens or if something happens at a certain time (Y) then do Z.
  - **X is Producer** - X here is a supported service that generates events
  - **Y is Time Period** - Y here is a certain time or time periods specified using UNIX Chrome format (allows you to specify one or more times flexibly)
  - **Z Target Service** - Is a supported target service to deliver the event to 
- EventBridge is like CloudWatch events v2 - uses the same underlying APIs and has the same basic architecture - AWS recommends for new deployments to use EventBridge
  - Things created in one are visible in other for now but this could change in future
- **Event Bus** : Both CloudWatch Events and EventBridge use Event Bus to receive and deliver events
  - Both services have a default Event Bus for a single AWS account
  - In CW Events only 1 single Event bus available which is not exposed to UI  
  - In EventBridge you can create additional Event Buses either for your application or third party products and services
- **Rules** - for CW Events and Event Bridge you can create rules that can pattern match with the events occur on the buses or you can also have schedule based rules 
  - **Event Pattern Rule** - When the events matches the rules specified they are then delivered to a target you specified in the rule
  - **Schedule Rule **- you can also have scheduled based rules they are also pattern matching rules but they are match at a certain date and time or ranges of date and times
- Events itself are just JSON structures and the data in the events structure can be used by the targets 

## API Gateway

**What's an API**

- API is a piece of code that's running/hosted on a machine that allows you to communicate to products or services. (including hardware) - API is used to pass commands and receive answers
- For example if you would like to talk to an AWS service such as S3, you can use User Interface, CLI (Command Line Interface) or an Application you wrote. You can use the API to communicate with S3
- In order to communicate you need to be authenticated via username/password or key based authentication and then you need to have the right level of authorisation to carry out the task you want to use API for. 
  - Once you are authorised, you become authenticated identity
  - API is an entity that authorises every single operation that you try to carry out that the API manages
- When you connect to any of the AWS services, behind the scenes you are actually using an endpoint. For EC2 you are using https://ec2.us-east-1.amazonaws.com
- The server that's running the APIs has to be highly available as this is the only way to communicate with any of the AWS services and if the API goes down then you won't be able to communicate with any products/services(including hardware)

**API Gateway** 

- API gateway is an AWS managed service and it provides managed API endpoints - It's a core product to deliver serverless architectures inside AWS
- It allows you to create, Publish, Monitor and Secure APIs.... as a service
- it is billed based on number of API Calls, Data Transfer and additional features such as caching
- it can be used for serverless architecture
- It's also good for architecture evolution - in case you have on-premise system hosting their own API then you can migrate that to AWS API Gateway to move towards serverless architecture
- It can be used for legacy and progressive/modern architecture
- It can also be used to slowly migrate from a monolithic architecture to a modern serverless architecture 
  - API Gateway allows your move to serverless architecture step by step 
  - API Gateway even has the capability of directly connecting to some supported products/services such as DynamoDB
  - This means you don't even need the compute service to talk to DynamoDB
  - You can pull data directly from DynamoDB using API Gateway

## Serverless Architecture

- Serverless isn't one single thing
- Serverless is about how software architecture than hardware 
- With serverless application you tend to manage few, if any servers - this means low overhead
- In serverless architecture you break your application down to smallest possible pieces that do specialised task and then stop - in AWS Lambda is a great product for these specialised functions
- These specialised functions run in stateless and ephemeral environments - this means when you run these functions they use a clean brand new environment
  - If you make/architect your application in a way that these functions are stateless then they can run anywhere - this means you can scale your application easily 
  - Every time these functions are run, then they do something/compute then optionally they store some data persistently or deliver the output somewhere else 
  - The reason Lambda is cheap is because it's scalable, each environment is easy to provision and each environment is the same 
  - Serverless architecture can use Lambda to its advantage 
- Another key thing about serverless architecture is that generally everything is event-driven. This means nothing is running until it's required
- Serverless architectures should use FaaS  (funtion as a service) where possible such as Lambda - it's billed based on execution duration and functions only run when an event happens  
- The cost of running serverless architecture should be close to 0 or 0 when nothing is happening. You components of your architecture should only run when some event happens. 
- It should used **Managed Services** and shouldn't reinvent the wheel. Examples are S3 and DynamoDB and thirst party identity provides such as Google, Twitter, facebook or even corporate identities such as Active Directory(AD) instead of building your own
- Elastic Transcode - converts/manipulate media files in other ways is another example of managed services

**Serverless Architecture Example** 

- Use S3 for static website which will run a JavaScript code to upload a video
- before the video is uploaded third party identifier such as google is used
- AWS Cognito swaps credentials to allow the user to upload video to original S3 bucket
- as soon as video is uploaded, an event is generated that is passed to lambda function
- lambda function then sends the original video to Elastic Transcoder that will create multiple versions of the video and store it in transcode Bucket
- Dynamo DB and Transcode bucket is then used by another lambda function to make the new video available back to the user
- This is a good example of serverless and event driven architecture where we were not using any dedicated server that's running 24 7  to help with video upload and processing
- Only those components are run when needed and when no components are running our cost is close to 0.

## Simple Notification Service (SNS)

- Highly Available - Durable Pub Sub Messaging service 
- Public AWS Service - you need network connectivity with public Endpoint in AWS - The benefit of this is it's accessible from anywhere via internet
- It coordinates sending and delivery of messages that are up to **256KB** in size
- Base Entity for SNS is called SNS Topics - this is where the permissions and configurations are defined
- A **Publisher** sends messages into a **TOPIC**
- A **Subscribers** received the all the messages that were published/sent to the **TOPIC**
- Subscribers come in different forms such as:
  - HTTP(S), emails(JSON), SQS (Queue), Mobile Push notifications, SMS Messages, and Lambda

- SNS is used across AWS for notifications - for example CloudWatch ans CloudFormation

**Architecture**

- It's in AWS Public zone - meaning users via internet can access it if they have the permissions

- For products/services inside a VPC can also access it if configured VPC correctly
- Within SNS, you can create Topics. Various producers such as an API in internet, CloudWatch om AWS, or EC2, ASG or CloudFormation Stack can generate messages into this topic
- The topic has subscribers - things can be producers and subscribers at the same time - for example APIs
- Any subscriber will receive all of the messaged sent to the topic by publishers
  - Although you can apply a filter to certain subscribers so they can receive only relevant messages
- **Fanout** - You can also have multiple SQS Queues subscribed to a topic in SNS where you can then use that message received to do different tasks - for example one SQS queue will create 1080p video and another 720p video
- SNS offers - Delivery Status (Including HTTP(s), Lambda, SQS)
- Delivery Retries  - Reliable Delivery
- HA and Scalable (Region resilience)
- Capable of Server Side Encryption (SSE)
- Cross-Account via Topic Policy - (Topic Policy = Resource Policy)

## Step Function

- Lambda functions are great but they have a limitation of 15 min
- this means that if you have an application that need to process something that could take longer than 15 min then Lambda isn't going to work
- You can chain lambda functions together where you invoke another lambda function at the end of first lambda function and then another after that. But this can get really message with complex applications
- Also these lambda functions are run in stateless runtime Environments - you can not maintain state between these lambda functions as each lambda function run in a stateless environment - so if you have to store data during the lambda function you will have to do it elsewhere
- **Step Function** lets you create what is known as State Machines 
  - State machines are long running serverless workflows - they have start and End and in between they have states.
  - States can be decision points or they can be tasks that performs actions on behalf of state machine. Using these state machines you can develop complex workflows that integrate with AWS services
- State machines are like a workflow - these are the base entity in the step function- it starts then it maintains a state and then it ends
- States in the state machines are things that occur and are maintained  for a maximum duration of 1 year
- State Machines in step functions come in two workflow types
  - Standard Workflow - 1 year execution limit
  - Express Workflow - up to five minutes (High volume event processing workloads, such as IoT, streaming data, mobile application backend)
- It can be started via API Gateway or IoT Rules, EventBridge, lambda etc or even manually.
- Usually used for Backend processing 
- you can also export a template of state machines which are saved in JSON format called ASL(Amazon State Language)
- IAM role is used for permissions - State machines assumes the role while running and it gets credentials to interact with any AWS services
- **States Types** - These states control the flow of things through a State Machine
  - Succeed & Fails
  - Wait - state just waits until a certain time or certain date and time reaches and then acts
  - Choice - allows you to make a choice depending on an input - for example you might want the state machine to react differently depending on stock levels of an item in an order
  - Parallel - allows you to create parallel branches within a state machine 
  - Map - Accept list of things - such as list of items ordered by a user. For each item in the list Map state performs an action or set of actions on that particular item
  - TASK (**Action** - Lambda, Batch, DynamoDB, ECS< SNS, SQS, Glue, SageMaker, EMR, Step Functions)

## Simple Queue Service

- Public - Highly Available - Fully Managed service that provides managed queues in standard and FIFO mode
- Highly Available - Highly Resilient and Performant (within a region)
- FIFO Queues - Guarantees an order - First In First Out
- Standard - First In First out is best effort - this means there is no guarantee on the order of messages in Queues 
- Messages to the queue could be up to 265KB in size - for large size data transfer use links
- **Architecture**
  - Clients can send messages to the queue
  - Other clients can poll the queue( process of checking any messages on the queue)
  - When client poll and receives messages - those messages aren't deleted - They are made hidden for a period of time (**Visibility timeout**)
  - **Visibility Timeout**  - is the amount of time the message is hidden when it's received - if it's not explicitly deleted then it will reappear in the queue to be processed again. 
  - Once the client has taken it's time to process the message workload then it can explicitly delete the message from the queue  - that message will then be gone forever 
  - This architecture is to ensure if a client is unable to process the message another client can receive the message when it reappears so make the application fault tolerance, resilient and highly available
  - **Dead-Letter Queue** - This is the queue where problem message can be moved to 
    - for example if you received same message 5 or 6 times but it's not getting processed then those messaged could be moved to Dead-Letter queue so that can be handled separately
    - Or if you received some corrupt messages - those too can be moved to Dead-Letter Queue
- Auto Scaling Groups can be scaled for the length of the queue
- Lambda functions can also be invoked based on the length of the queue

**Example** 

We have have an architecture of Auto Scaling Group with Web Pool and another Auto Scaling Group with Worker Pool

In between Web and Worker pool we have SQS(Simple Queue Service)

The ASG web Pool scales depending on the CPU utilisation of EC2 instances in Web Pool

The ASG worker pool scales depending on the length of messages in the SQS Queue. 

User can use the web pool to upload a video in S3 - SQS can generate a message in the queue with the link of the video uploaded in S3. This queue is polled by the EC2 instance(s) in worker pool

They process the video file and save it in another S3 bucket called Transcode and delete the message in the queue

In this way the Worker pool can scale to the length of messages in the Queue but it can also scale back to zero if there are no messages in the queue

Once the video is saved in the Transcode bucket then using Web Pool it can be made available back to the user

**Advanced Example**

We can also make this architecture a bit more advance by using **SNS and SQS Fanout** architecture

ins this architecture, when the user uploads the video, the message with the link to the video can be added to SNS topic rather than SQS Queue. And rather than one SQS queue we can have three separate SQS queues that can be subscribed to SNS topic. Each queue can be dedicated to 480p, 720p and 1080p workloads respectively. 

These three individual queues are polled by three separate auto scaling groups. One for 480p, 720p and 1080p.

Each ASG will pick up the message from SQS, process the file, store it to S3 bucket and then delete the message from SQS. The ASG Web pool will pick up the processed files from the bucket and make it available to the user. 

This is called SNS and SQS Fanout Architecture

**Simple Queue Service (SQS)**

- Standard = Multi Lane Highway - at least once delivery - no guarantees the order
- FIFO = Single Lane = at least once delivery - guarantees order
- FIFO (Don't offer exceptional level Performance) 3,000 messages per seconds with batching or up to 300 messages per seconds without batching
- Standard Queue - Performance is amazing as it's like multi lane highway so more lanes can be added and can scale to near infinite levels - linear scaling
- Billed based on 'requests'
- 1 request = 1 to 10 messages up to 64 KB total
- Short(immediate) vs Long(waitTimeSeconds - can be up to 20 seconds) Polling
  - Long Polling - this is what you should use 
    - up to 10 messages or 64 KB = 1 request 
  - Short - Immediate
    - consumes 1 request even if there are messages on the queue or not 
    - can receive 1 or more messages
    - 
- Messages can live in SQS for up to 14 days to it supports encryption at rest using KMS and in-transit
- Access to Queue is based on identities and for external accounts use Queue Policy
- Queue Policy - External accounts - it's just like the resources policy on S3

## Kinesis & Kinesis Firehose

- It's a scalable streaming service
- the base component of this service is Kinesis Stream
- Producers can send data to Kinesis stream and consumers can use that tream to access the data
- Streams can scale from low to near infinity in terms of data rates
- It's a public service and Highly available 
- Streams only store 24 hour moving window - so data that enters stream is only available for 24 hours period - after 24 hours period the data is gone
- Multiple consumers can access data from stream - they can access the live data as well as go back up to 23 hours period

**Architecture**

**Producers**

- These can be EC2 instances, Mobile devices, IoT Devices, on Premise servers

**Consumers**

- things like, On Premise Servers, Lambda functions, EC2 instances

In between you have Kinesis Stream - that reads data from producers and make it available to consumers

- Kinesis uses Shard Architecture
  - Stream starts with single shard but as more performance is required, more shards can be added to the stream
  - Each shard provides its own performance - 1 MB Ingestion and 2 MB Consumption
  - the more shards you add to the stream, more expensive the stream becomes and you gain more performance
  - Another factor for cost of the stream is the window of the data stored in stream. By Default you get 24 hours rolling window but it can be increased to 7 days for additional cost.
  - The way data is stored in Kinesis stream is by Kinesis Data Record (1MB Max Size)

**Kinesis Firehouse** 

- This is another related product to Kinesis Stream
- It allows you to take the streaming data from Kinesis stream and store it in another AWS product such as S3
- So if you have a lot of IoT sensors producing lots of data which you would like to go through later then you can use Kinesis Firehose to save it in S3 and store data for longer term

**EXAM** **SQS vs Kinesis**

- **SQS**
- If the question is about about worker pools and decoupling of application? or does it mention asynchronous communication? ===> chose SQS 
- SQS usually has 1 thing or group of things sending messages to the queue and 1 consumption group - example could be web tier inside Auto Scaling group
- SQS is usually used for decoupling and asynchronous communication(where sender and receiver don't need to be aware of each other and don't care about the functional state of each other)
- SQS has no concept of persistence of messages - no window. Messages in queue are temporary. Once received and processed they are deleted forever. there is no concept of time window in SQS queue
- **Kinesis**

- If question is about ingestion of data or ingestion of data at scale - meaning large number of devices producing data or large throughput? ==> chose Kinesis
- designed for huge scale ingestion
- it can have multiple consumers(each consumers might be consuming stream at different rates) - it has a rolling window - users can move back and forth in time in this rolling window
- it is used for Data Ingestion, Analytics, Monitoring, App Clicks or mobile click streams