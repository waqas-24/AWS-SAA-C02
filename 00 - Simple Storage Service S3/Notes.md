## S3 Events

- This is feature in S3 that generates notifications when something happens in S3 bucket
- It can send notifications to SNS, SQS and lamda Functions
- you can generate notifications when an object:
  - **Created** (Put, Copy, Post, CompleteMultiPartUpload)
  - **Deleted** (*, Delete, DeleteMarkerCreated)
  - **Restore **(Post(Initiated), Completed) - e.g. restoring objects from S3 Glasier
  - **Replication** (OperationMissedThreshold, OperationReplicatedAfterThreshold, OperationNotTracked, OperationFailedReplication) - E.g tracking replication related activities
- S3 Event Notification is a bit old and can only capture limited events happening in S3. 
- An alternative to s£ events is EventBridge service which supports more types of events and services

## S3 Access Logs

S3 Access logs is simple. When turned on either via console UI or CLI, it captures who is accessing the source bucket and generates logs. It then send generated logs to target bucket

S3 Log Delivery Group handles the logging - it understands the configuration set on the source bucket and delivers the logs to target bucket. 

this is a best effort process - once enabled or changes made to configuration - logs are then usually delivered within few hours

The access to target bucket for S3 Log Delivery Group is done via ACL on target bucket

logs are delivered via new line delimited files and attributes are space delimited. Single target bucket can be used for many source buckets logs

lifecycle and movement of log files should be managed by you it's not part of the feature. 