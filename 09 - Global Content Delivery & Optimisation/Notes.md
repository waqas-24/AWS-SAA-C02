## 09 Global Content Delivery and Optimisation

## CloudFront Architecture

- CloudFront(CF) is a global object cache service also known as Content Delivery Network (CDN)
- Instead of users directly connecting to S3 bucket every time they need to access an object, they can have their content cached closer to their location 
  - customers think they are accessing objects from original location but instead they are connecting to CloudFront(local infrastructure)
  - if content is available in local cache then it's delivered to user otherwise a request to original content server is made, object is stored in local cache and then delivered to user
- This provides lower latency and higher throughput
- This also helps in reducing the load on the content server
- This can handle both static and dynamic content
- CloudFront Terms
  - **Origin** - The source location of your content - Usually S3 bucket or App load Balancer or in theory anything that's available on the internet  as long as it's accessible by CloudFront
  - **Distribution** - The **configuration** unit of CloudFront. you create this distribution and then it gets deployed out to the network - all configs are set here
  - **Edge Locations** - Local infrastructure much smaller than regions which hosts cache of your data (third party data centres)
    - there are over 200 edge locations
    - around 90% storage with some associated compute services in these edge locations - this means these location can not be used to provision EC2 instance
  - **Regional Edge Cache** - Larger version of edge location - This provides another layer of caching
    - Items here are accessed less frequently but it still provides performance benefits by holding these items in cache
    - it provides second layer of caching - Items are first checked in edge location, if not found then Reginal Edge Location is checked , if not found then Origin Fetch is called where the item is fetched from the original source location

**Architecture**

- Items are uploaded to S3 - this is the Originial Content
- You create a Configuration Distribution which you link the S3 bucket to 
- in this Configuration Distribution you also specify the endpoint mapping with CloudFront
- so you have the globally distributed edge locations - one thing that is create with this globally distributed edge location is domain name such as http://d32767657623abcdef34.cloudfront.net
- you can then use Configure Distribution to use this cloud front domain name of cloudfront with your application such as http://animal4life.org
- once you have configured your distribution then you can deploy it to the cloudfront network 
- this will push your distribution configuration to all the chosen edge locations
- this means the edge locations can now be used by your customers/users
- Regional Edge Cache provides second layer of caching - Items are first checked in edge location, if not found then Reginal Edge Location is checked , if not found then Origin Fetch is called where the item is fetched from the original source location - the item is first saved in the regional cache, then edge location and then it's sent to your customer
- Once the item is in regional edge cache then any request for that item from edge location can be served by the regional cache  
- Regional cache were not initially part of the Cloud Front - they were added later
- The less your items have to travel the better performance your user/customers will get
- **IMPORTANT**: CloudFront integrates with HTTP and ACM(AWS Certificate Manager) - this means you can have HTTPS capability with CloudFront for statically hosted website running in S3
- **IMPORTANT**: It is only for downloading content not uploading. If you try to use PUT it will go straight to the origin.

**How Caching works in CloudFront?**

- CloudFront Cache works with full URL string - and if you use Query String Parameters such as:
  - http://animal4life.orge/whiskers.jpg?language=en
  - http://animal4life.orge/whiskers.jpg?language=es
- This means that whiskers is saved in local edge location twice - one with en and another with es Query String Parameter
- So you have to be mindful how you setup your application and be careful with Query String Parameters as you might not get optimised benefits of caching 
- If your application doesn't use Query String Parameters then don't worry about them 
- If you use Query String parameters then you can Forward these query string parameters to Origin - 
- So if your are not using query string parameter you can set Forward To Origin to No and you will get full benefit of caching
- If you are using query string parameter then you can set Forward To Origin to Yes to ALL or selected ones 
- So for example if you think the query string parameter order_id in this link http://animals4life.org/whiskers.jpg?order_id=001 has no impact on the content which you want to cache then you can set Forward To Origin to No
- **IMPORTANT**: But the language query string parameter might impact the page - so you have to be careful what you want to cache what you don't want to cache - and what is forwarded to Origin and what isn't

## AWS Certificate Manager (ACM)

- Allows you to provision, manage and deploy public and private certificates for use mainly with AWS supported services - things like CloudFront, Application load balancers  NOT EC2. 
- this is capable of handling pretty complicated stuff but for associate level exam you only need to know the public certificate part(HTTTPS)
- HTTP - simple and insecure - No server identity authentication or transport encryption
- HTTPS - SSL/TLS Layer of encryption added to HTTP
  - Data is encrypted in-transit
  - **Prove identity** - also allows servers to prove its identities - Using SSL/TLS service can be authenticated using digital certificates
  - **trusted authority** - these certificates can be signed digitally by trusted authority knows as CAs that are trusted by your web browser
    - since your browser trusts the CA you trust the certificate which it signs - meaning you can trust the site 
  - if netflix.com has a signed certificate from a CA then it's almost certain to be actually netflix.com
  - **DNS names and the certificate are tied together** - a website picks a DNS name, it generates a certificate or one is generated for it - this certificate is signed by an authority trusted by your browser and the site uses this signed certificate to prove its identity
- With ACM you can deploy certificated to Application load balancer as well as cloud front using distribution configuration. All the edge locations will also be accessed via HTTPS but access from edge to origin you have to have a valid certificate - it is not self signed. 

## Securing CloudFront and S3 using Origin Access Identity(OAI)

- With the help of Origin Access Identity(OAI) we can prevent users to directly connecting/fetching data from S3 Bucket
- We can create Origin Access Identity (OAI) associate it with CloudFront Distribution - this means that all the edge locations linked in the distribution also get this OAI
- Now we can adjust or add the Bucket policy at our S3 bucket to add an explicit Allow for OAI and we can remove any other explicit allows on this bucket policy which leaves the implicit Deny
- This means any access to S3 bucket will come from edge locations that has OAI
- So any user if tries to directly access outside from the Edge location will be blocked via implicit Deny and will not be able to access S3. 
- You can use one OAI with many buckets or CloudFront Distribution but best practice is to use one single OAI per CloudFront Distribution and use it to many S3 buckets - this helps manage OAI and permissions easily
- CloudFront Distribution can also be used for private distributions - making it private only affects the distribution at edge locations so it's very important to use OAI to control the access to S3 buckets. 
- If you have a private distribution and someone gets hold of your S3bucket URL they can potentially access the bucket directly if no OAI is in place

## Lambda@Edge

- Lambda@Edge allows you to run lambda functions at edge locations
- this allows you to adjust data between the Viewer & Origin(such as S3)
- Currently Python and Node.js is supported with Lambda@Edge
- These only run in AWS public Space - so no VPC resources access
- Layers are not supported with Lambda@Edge
- Function duration limits are different than normal Lambda Functions
- Lambda@edge Functions can be used:
  - **Viewer Request** - After the request is received from the viewer (128 MB RAM with 5 seconds)
  - **Origin Request** - Before the viewer request is forwarded to the origin (Normal Lambda MB RAM with 30 seconds)
  - **Origin Response** - After the origin response is received from the Origin (Normal Lambda MB RAM with 30 seconds)
  - **Viewer Response** - Before the response is delivered to the viewer (128 MB RAM with 5 seconds)

**Use Cases** 

- A/B Testing - Viewer Request
  - Which object to return to viewer depending on some logic
- Migration Between s3 Origins - Origin Request
  - you can slowly migrate to new s3 bucket using
- Different Objects Based on Device - Origin Request
  - different type/quality content depend on device type
- Content By Country - Origin Request
  - Filter content based on country

## Global Accelerator

- It brings AWS infrastructure closer to customer location - for example of you have your infrastructure provisioned in US and you want to access it from Australia then with Global Accelerator you will connect via edge location using anycast IPs and then you will use amazon network(backbone) to reach infrastructure in US rather than using public internet infrastructure
- with Public internet infrastructure you will have to go through a lot more hops than AWS infrastructure
- Connections enter at edge locations
- You reach one or more AWS locations such as in US as well as London (it will get to your endpoint as quickly as possible)
- it is a network product and unlike CloudFront that is used for content delivery/caching (HTTP/HTTPS) this uses (TCP/UDP)

