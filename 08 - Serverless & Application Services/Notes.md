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

