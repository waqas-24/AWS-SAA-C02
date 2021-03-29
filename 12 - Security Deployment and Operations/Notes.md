# 12 Security, Deployment and Operations

## AWS Secrets Manager

- this often gets confused with SSM Parameter Store
- With SSM Parameter Store you can create secure strings to save passwords 

**When should you use Parameter Store and AWS Secrets Manager?**

- Secrets Manager does share some functionality with Parameter Store
  - This means for certain cases you can pick either Secrets Manager or SSM Parameter Store
- As name suggests, Secrets Manager specially designed for secrets (passwords, API keys)
- you should default to Secrets Manager if you see passwords or API keys in exam
- Secrets Manager is usable via Console, CLU, API or SDK (Integration)
  - Architecturally it is designed to be integrated inside your applications
- it supports automatic rotations of secrets - this uses lambda to rotate secrets periodically
- secrets rotations is integrated with some AWS products such as RDS
- this means that when password in Secrets Manager is refreshed and any authentication build on that such as RDS is is also refreshed - so it's synchronised with Secrets Manager
- Exam - if you see rotating secrets or rotating secrets with RDS then default to secrets manager 
- Keywords for Exam: Secrets, rotation, RDS product, integration with product such as RDS
- SSM Keywords: hierarchical configuration information, configuration for the CloudWatch Agent 

## AWS Shield and Wed Application Firewall (WAF)

**AWS Shield** - Layer 3 and 4 protection - DDoS

- AWS Shield provides protection with DDoS to AWS resources 
- Shield Standard - free with Route53 and CloudFront
- Protection against Layer 3 and Layer 4 DDos Attacks
- **Shield Advanced** - $3,000 p/m
  - EC2, ELB, CloudFront, Global Accelerator & R53
  - You also get DDoS Response Team 24/7 365 and Financial Insurance against any increase in AWS Costs

**AWS WAF** - Layer 7 HTTP/S Firewall/Filtering

- provides Layer 7 (HTTP/s) Firewall/Filtering
- Protects against complex Layer 7 attacks/exploits
  - Such as SQL Injections, Cross-Site Scripting, Geo Blocks, Rate Awareness
- Key component is - Web Access Control List (WEBACL) integrated with ALB, API Gateway and Cloud Front
- Rules are added to a WEBACL and evaluated when traffic arrives

Keywords - Global Perimeter Protection- Shield Layer 3 and 4 DDoS Protection and WAF Layer 7 Rule based HTTP/S Firewall

## CloudHSM

- It is similar to KMS as it creates, manages and secures cryptographic materials or keys
- KMS - Key Management Service - for encryptions/decryption with other AWS products 
- Even though your Keys in KMS are isolated but it's a managed services used by other AWS users
- While permissions in AWS are always strict but AWS do have a certain level of access to the KMS product
- AWS Manages the hardware and software for KMS product
- Behind the scenes KMS uses HSM (Hardware Security Module)
  - these are industry-standard pieces of hardware that are designed to manage keys and perform cryptographic operations
- **EXAM:** With CloudHSM you can have your own "Single Tenant" HSM hosted in AWS Cloud
  - AWS maintains the hardware but they have no access to the stored keys and management of the keys
  - If you lose access to HSM then there is no easy way to recover data - you can provision it again but recovery of data is not guaranteed
- Federal Information Processing Standard Publication 140-2
  - you can check the capability of compliance level of your HSM product with this standard 
- **EXAM:** CloudHSM is FIPS 140-2 Level 3 compliant - KMS is largely Level 2 but some areas are level 3
- **EXAM:** KMS is fully intergrated and used via AWS APIs whereas CloudHSM is not integrated that well by design.  With CloudHSM you can use Industry standard APIs - PKCS#11, Java Cryptography Extensions (JCE), Microsoft CryptoNG(CNG) libraries
- KMS can use CloudHSM as a custom key store - there a level of integration between KMS and Cloud HSM to get the benefits of CloudHSM and integration with AWS 

**Cloud HSM vs KMS**

- No native integration with CloudHSM and AWS products such as S3
  - For example you can't use CloudHSM with S3 Server Side Encryption (SSE)
  - Instead you can perform client side encryption using CloudHSM and then upload the object to S3
- CloudHSM can be used to offload the SSL/TLS Processing for Web Servers
- You can use CloudHSM to enable Transparent Data Encryption (TDE) for Oracle Databases
- Cloud HSM can also be used to protect Private Keys for an Issuing Certificate Authority (CA) 
- **EXAM:** Cloud HSM ideal for requirements where you need to use Hardware Security Module using industry standard APIs which doesn't need much integration with AWS products then you need to use CloudHSM
- **EXAM:** For anything that does require native integration with AWS then CloudHSM isn't suitable

## AWS Config

- it allows you to store configuration changes on resources to S3 config bucket over time
- **EXAM:** it can not prevent you from making configuration changes - it only logs changes
- If you add a port number in Security Group then it AWS config will record the previous configuration, the new configuration as well as time stamp, resource configuration is applied to, the related resources and who made the change
- it is a great way to audit changes and for compliance with standards
- it's a regional service - supports cross-region and account aggregation
- The changes can generate SNS notifications and near-Realtime events via EventBridge and Lambda
- With AWS config you can also have Config Rules that can be your own or AWS managed and this is used by Lambda
- With these config rules you can class your resources as compliant or non-compliant. 
- Lambda function evaluates the config changes based on config rules and it can then notify other products such as SNS to take action. 
- For example Lambda can either send stream of changes or compliant or non-compliant notifications to SNS that can then be forwarded to humans or other product or services
- You can also integrate AWS config with Event Bridge that can invoke another lambda function to take action 
- You can also use SSM(Systems Manager) to take action on the changes but with Lambda it's a bit more flexible for Account level thing as opposed to instance configurations via SSM

## AWS Macie

- It's a Data Security and Data Privacy service that discovers, Monitor and Protect data stored in S3 buckets
- Sensitive data could be PII (PersonalIy Identifiable Information) PHI (Personal Health Information) or Finance data 
- It's centrally managed either via AWS Organisation or one Macie account can invite another AWS account
- You Enable AWS Macie and link it to S3 buckets and then create a discovery job using AWS Managed data identifiers or Custom data identifiers.
  - Managed Data Identifiers = Built-in - ML/Patterns - Managed by AWS
    - it contains a growing list of common sensitive data types 
    - it contains credentials, finance 0 credit cards, health data, personal identifiers (passports, driving licenses etc)
  - Customer Data Identifiers - Propriety - Regex Based - these are created by you
    - you define Regex - it's a pattern to match in data e.g. [A-Z]-\d{8}
    - in addition to regex you can also use Maximum Match Distance and Ignore Words to refine pattern matching
- This discovery job is then scheduled and produces findings
  - Macie produces two types of findings
    - Policy Findings - when policies or settings for an S3 is changes that reduces S3 security and its objects - like it changing public access settings or disabling S3 encryption
    - Sensitive data findings - this findings are based on the pattern matching defined above. 
- then findings can be viewed in AWS console or these can be sent to as an event to EventBridge which can then invoke lambda function to take action on the findings 

