## Domain Name Service (DNS) Fundamentals

It's a discovery service. It's how systems are discovered in public and private networks

It translates human readable information into a language that machines can understand

For example, if you type www.amazon.com in your browser, it will translate it to an IP address(104.98.34.131) to tell you where to go to get www.amazon.com page.

As you can imagine the number of websites in the world, DNS has to maintain a huge database which has to be distributed

With IPv4 there are over 4.3 billion IP addresses and for each IP address if there are more than one web page assigned then you can imagine how massive this database can be.

With IPv6 you have even more IP addresses to handle.

**Architecture**

- When you first try to talk to amazon.com, from your PC/device you first reach out to DNS resolver that could be with your Internet Service Provider(ISP) or it could be in your router. 
- To find the IP address of amazon.com, we need to reach the Zone File that's stored somewhere in the global infrastructure of DNS. This Zone file has the DNS record for www.amazon.com meaning the IP address
- Out of millions of DNS servers around the world, this Zone file is physically hosted in one or two DNS server called Nameserver(NS)
- So DNS provides a core service to allow DNS resolver that's with your ISP or in your router to fin the Zone file for the website you are trying to reach, query the zone file and return the IP address for the web site you are tying to reach
- DNS is distributed across the globe, it's resilient and hugely scalable.

**Key terms to remember**:

- **DNS client** - The device that's trying to find the IP address of the website it's trying to reach. In other words DNS client is any device that's looking for information that DNS has. it's usually a piece of software that's inside the device that you are using- it could be your PC, phone tablet or laptop
- **DNS Resolver** - That's a piece of software that could also be running on your device(DNS Client), or a separate server inside your router or could be a physical server running inside your ISP(Internet Service Provider) - It's this DNS resolver that queries the DNS system for you.
  - So in short the DNS client talks to DNS resolver, the DNS resolver talks to DNS system to find the IP address of the website your trying to reach.
- **Zone** - This is a part of global DNS system for example, amazon.com, netflix .com google.com. These zones live inside the DNS system. Zone is like the classification of the data - meaning what data are we talking about. Zone file is how the data is physically stored for the zone. 
- **ZoneFile** - is where the data stored physically for the zone - so there will be a physical file for amazon.com, netflix.com etc
- **Nameserver** - This is where the zone files are hosted. 

**DNS workflow**

- Find the nameserver that's hosting the zone file you are looking for, query this name server to find the record of the website that is stored in the zonefile on this name server and return the result back to DNS resolver which will return the result back to DNS client. 

**DNS Root** 

DNS is distributed but it has to have a starting point. This starting point is the root.

When you enter a website www.amazon.com. it is read from right to left. Notice the period after com, that's the root. usually when you type www.amazon.com that period is not there/needed but browser adds it for you. 

from this period, your device starts reading the website name and it represents the root of DNS

This DNS root is also known as DNS root zone is the starting point of DNS infrastructure and it's just a database.

DNS infrastructure is like a tree upside down. and this DNS root or DNS root zone is at the top of this DNS tree

This DNS root or DNZ root zone is hosted on 13 special name servers knows as root servers. These 13 root servers are run by 12 different large companies around the world.

These companies only manage root zone servers only, not the database. As you can imagine these are not just 13 servers but they can be a cluster of servers distributed globally but represented by one company. 

Each of these 13 servers (cluster of servers) are assigned letters from a to m. Each of these managed by different companies except Verisign that manages two servers **j** and **a**

These root servers are the entry point to find the website. So in order to reach any website you need to get to one of these servers. 

To find these root servers, you use Root Hints file that's provided by your OS provider. For Windows it's Microsoft and for Linux is OS maintaining group.

This root hints file is just a file that has a list of these 13 root servers. it's just a pointer to the root servers. 

**DNS Workflow**

Step 1 - So the first step is that your DNS Client(Your device) talks to DNS resolver to find the IP address of www.aamzon.com

Step 2 - Using this root hints file, the DNS resolver communicates to one or more of the root servers to find the Root zone and begin the process of finding IP address

- Root Zone is managed by an organisation called IANA (Internet Assigned Number Authority)
- It's their job to manage the content of Root Zone
- its a separate job from managing the root servers which hosts the root zone
- DNS is built on trust. DNS client trust this root zone via root hint file or the DNS resolver server that itself relies on this root hint file
- In DNS the starting point is root zone and that's the only thing we trust at the beginning - when we trust something in DNS system, that thing is an authority. in other words that thing is authoritative
- The root zone can also delicate some responsibilities related to DNS to another authority  or entity. So if Root zone trust this entity that means that is trusted and it also becomes an authority. 
- Root zone is just a database - it's a database of top level domain(TLD)
- The top level domain with country code are called CCTLDs  (Country Code Top Level Domains) and others are called generic gTLDs or just TLDs as they aren't mapped to any country.
- So th eroot zone just contains the database for TLDs and ccTLDs that point to different entities for different root zone. For Example, .com is delegated to Verisign and then Verisign has various nameservers that hold the .com zone. one of the zone within verisign will have amazon.com and that zone will give you the IP address of amazon.com
- So you first go to root zone with amazon.com, that root zone will forward you to the relevent trusted entity. In this case verisign. And then verisign will forward you to exact name server where .com zonefile is stored that contains amazon.com ip address. 

## Route 53 Fundamentals

It allows you to:

- Register a domain
  - To register a domain, Route53 has relationships with all major Top Level Domain(TLDs) companies
  - Lets support you want to register animal4life.org 
  - .org is managed by a company called PIR
  - When you try to register this domain, Route53 first checks the registry if that domain is available using contacting PIR. Assuming it's available.
  - Route53 creates a ZoneFile for your domain. Zone file is just a file that contains DNS information for that domain - eg. IP address
  - Route53 then allocates a name servers for this zone
  - Route 53 manages these  name servers that are distributed globally. For each zone there are 4 name servers. 
  - This zone file is called hosted zone in AWS route 53 terminology and route 53 puts this files to these 4 managed name servers 
  - Route53 also has to communicate with .org registry and it lists these name servers into the zone file for the .org top level domain. 
  - It used Name server records to do this. 
- Host zones and manage Nameservers
  - Zone files in AWS are called hosted zones as they are hosted on 4 name servers
  - you can have public or private hosted zones
  - with public anyone with internet connect can reach to the name server that hosted zone is located
  - but with private it's only available with the VPC it's linked to. 
  - it stores various record set such as name server and others...

It's a global service that is globally resilience. If any region goes down, this service will continue to function. 

##  DNS Record Type

- **Nameserver (NS)**
  - This allows how the delegation occurs in DNS
  - .com is managed by verisign - so as you can imagine with .com you can't have just one server with all the zonefiles in it. So to reach amazon.com there will be multiple namesservers that can help you with amazon.com zone that verisgn handles. Just like AWS has at least 4 nameserves for each zonefile(hostedzone)
  - In other words, NS is type of DNS Record Type that has the list of server names that contains DNS records such as for amazon.com. So these servers host amazon.com zone. Inside this zone we have DNS records such as www which is how you access these DNS records. 
  - **So name servers records are how the delegation works - They guide you where to go next to resolve the DNS record you are looking for.** 

- **A and AAAA Records**
  - It converts Host to IP address
  - A record maps host to an IPv4 address
  - AAAA maps host to an IPv6 address
- **CNAME Records**
  - Creates DNS shortcuts - Host to Host records
  - one server can perform multiple tasks such as ftp, mail, web hosting and others 
  - so three CNAME records are created pointing to the same server
  - CNAMES are used to reduce admin overhead
  - CNAME only points to the Sever name for example amazon.com. It doesn't point to the IP address. Only A or AAAA record type points to IP address

- **MX Records**
  - It is a massively important type of record - especially how the email works
  - MX records are used to find the mail server
  - for example if you want to send email to hi@google.com. You use the same route - first go to root zone, then google.com zone and that's where you will find the MX record.  
  - Example of mx records 
    - MX 10 mail
    - MX 20 mail.orther.domain.
  - MX records have two things, priority and value. The low value in priority field such as 10 has higher priority.
  - you will first go to mail server using A record that has the IP address. 
  - If you just mail in your MX record that just means that you want to go to mail.google.com
  - but if you have a fully qualified domain name such as mail.other.domain then it could mean you want to go elsewhere. for example you might want to use microsoft 365 for your mail services.
  - In other words an MX record is used to find the mail server for a specific domain

- **TXT Records**
  - allow you to add a piece of text of your choice to a domain
  - This allows DNS to provide additional functionality such as to prove domain ownership.
  - So for example lets say you added a TXT record with text "cats are the best" 
  - Now you want to use Google or microsoft mail services to setup your own domain email. Google or microsoft could query TXT record and if it matches to your text that means we own the domain. 
  - It can also be used to fight spam

**DNS TTL** (Time To Live)

lets say you need to open amazon.com page, you first go to root zone, that gives you the .com zone, and that .com zone give you amzon.com zone and thats where you get the www record for amazon.com

This whole process takes time and by getting the answer back from the final name server is called Authoritative answer. 

This walking down from the root to branches of tree process takes time but is always preferred as it;s always going to be accurate.

But the administrators or amazon.com could tell us how long we can cache that authoritative answer. 

Lets say amazon has set 3600 TTL value, it means the resolver server can store the answer to query for 1 hour. 

If someone else tries to reach amazon.com then they will get non-authoritative answer that's stored in resolver server. but the client won't have go from tree root to branch and the result will be super quick as it is cached as the resolver server. 

it's good to get quick non-authoritative answer from resolver server but imagine if you have changed your mail server and you have high TTL value on your MX record then your email delivery might be delayed

Low ttl value means more queries against name servers high value means less control (non-authoritative answers)

The resolver should obey TTL values but it's not guaranteed. It could ignore TTL value. This configuration is managed locally by the admin of resolver server.

if you are doing any work that involves changing DNS records then it's best to reduce TTL value well in advance - weeks or months. So the records are not cached.

## R53 Public Hosted Zones

- R53 hosted zone is same as Zone file - it's just a database to keep DNS related records such as for animal4life.org
- There are two types of Hosted Zones in r53 - Public and Private
- Public one is exposed to internet and anyone can access it and the private one is not exposed to public internet. It's linked to a VPC inside AWS and it's not visible to internet. 
- R53 is a globally resilient service  - if a region fails then this service will continue to function. 
- Hosted Zone is created automatically if you register a domain using R53 service 
- You can also buy/create a domain elsewhere and then use R53 to host your zone file. 
- There is a monthly fee you have to pay for using R53 to host your zone file (in AWS it's called hosted Zone) and also you pay for the number of time that hosted file is queried/requested. 
- Hosted zone, whether public or private, host different types of DNS records - for example, NS, A, AAAA, TXT etc 
- If a hosted zone is being used via NS record that means your hosted zone is authoritative for that domain. 
- When you register a domain, name server record for that domain are entered in Top Level Domain zone for example for .org it will be PIC(Public Interest Registry) and these point to your name servers in AWS so your name servers and the zone they host becomes authoritative for that domain
- With public R53 hosted zone it's available via public internet as well as VPCs via r54 resolver
- when you buy domain via r53 it saved hosted zone in 4 r54 Name servers for that domain
- to integrate with public DNS system you change the NS records for that domain to point to 4 name servers in r53
- Inside public hosted zone, you create Resource Records like MX, TXT WWW records.
- You can also buy domain via godaddy  or any other company and then create public hosted zone in r 53 that gives you public name server and then using go daddy interface you can point your domain to these name servers.
- The 4 name servers assigned for each hosted hones are accessible via public internet as well as VPC via 553 resolver. 

**Using VPC**

- if you have enabled DNS resolver in your VPC then it's straight forward- you just type the domain name and it goes straight to DNS resolver that points to r53 hosted zone

**Via Public Internet**

- From your PC you go to your ISP resolver then root servers then .org TLD that contains NS records that point to NS servers in R53
- There is a monthly cost for hosting this public hosted zone as well as you will have to pay for the number of times this hosted zone is queried

## R53 Private Hosted Zones

- Private hosted zone is just like public except its only available inside a VPC in AWS
- you can associate private hosted zone in AWS using UI Console, CLI or API
- you can also associate Private Hosted zone in different AWS accounts using CLI or API only
- SPLIT-VIEW: you can have public and private hosted zone of the same name. This means when someone from the pub internet tries to access a web site they will be presented with publicly hosted zone but if someone tries access the same name inside AWS, they will be directed to private hosted zone. 

## CNAME vs R53 Alias

- CNAME maps a name to another name
- For example, you can have a CNAME  www.google.com maps to google.com
- It's a way to create an alternative name in DNS.
- The problem is you can't use CNAME for Naked domain name. Named domain is also called apex of a domain
- it's a domain name without www at the start
- so you can't have google.com pointing to something else
- Alias om AWS fix this problem. With alias you can point to something using apex of a domain/naked domain
- you can still continue to use CNMAE for www records
- ALIAS records map a NAME to an AWS resource
- it can be used for naked/apex as well as normal records
- for www.google.com you can use either CNAME or ALIAS
- for naked/apex domains you have to use ALIAS records if you want to point to an AWS resources
- There is no charge for ALIAS requests pointing at AWS resources
- EXAM: Remember to use ALIAS by default for pointing naked/apex domain for AWS resources
- ALIAS is a subtype - you can have an A record ALIAS meaning an IP address or CNAME record ALIS meaning a name. such as ELB load balancer name
- ALIAS is an AWS naming convention - it's only usable in Route 53 and outside in normal DNS it's not usable
- you need an ALIAS record for AWS services such as API Gateway, CloudFront, Elastic Beanstalk, ELB, Global Accelerator and S3

## Route 53 Simple Routing

One of the simple routing policies available in Route 53 is called Simple Routing

Lets say we have a hosted zone called animal4life.org and with simple routing you can have one record per name 

For example WWW which is an A record type. As you know A record returns an IP address. With Simple routing you can have one or more than one IP address assigned to one WWW records. All IP addresses are returned by the query in random order

Simple routing does not support health check - meaning it doesn't check the IP address it's pointing is operational or not but simple routing is fairly easy to setup and manage. 

all other forms of routing policies in route 53 offer some form of health check on the pointed location. 

## R53 Health Checks

Health checks are different from records - they are a setup and configured separately

Health Checkers are used to check health of something and they are distributed globally

 Health checkers are not limited to AWS - you can check anything over the public internet - you only need an IP address and Check occur every 30 seconds by default but at an additional cost you can increase it to 10 sec

You can do the following checks:

- TCP
  - this needs to be successfult within 10 seconds
- HTTP/HTTPS 
  - More useful for Web application then a simple TCP check
  - Route 53 should establish TCP connection with endpoint within 4 seconds and the end point should respond with HTTP status code in the  200 or 300 range within 2 seconds after connecting
- HTTP/HTTPS String Matching check (Most accurate health check)
  - Same as above but once route 53 receive the status code it must also receive the response body within 2 seconds. Route 53 checks the response body for strings that you specified. 
  - the string should be in the first 5120 bytes of the response body else the target server fails the health check

Based on the results your target servers can be declared Healthy or Unhealthy

**Checks Types**

- Endpoint - simply checking a server

- CloudWatch Alarm - it would reach to CloudWatch alarms configured separately and can contain some in OS or in-app tests if CloudWatch agent is used

- Check of Checks (Calculated) - you can setup application wide health check with individual components


## Failover Routing

With Failover Routing if the primary address isn't working then you will be redirected to the secondary address potentially hosted by a different machine.

We can use "out of band" failure/maintenance page for our website in case the primary address is not working

The health check generally occurs on the primary resource but if it fails then the secondary records of the same address sis returned

You use this to route traffic to a resource when it's healthy but if it's unhealthy then you want ot route to a difference address for the same resource

## Multi Value Routing

it's a bit of a mixture between fail over and simple routing

With Multi Value Routing you can create many records with the same name - For example you can create many WWW A records that map to different IP addresses

Up to 8  healthy records are returned to the DNS client when queried - If you assign more than 8 IP addresses than 8 randomly selected records are returned

Any records with failed health check won't be returned

This is not an alternative to a load balancer that handles session based data but to be able to return multiple healthy IP addresses to server the request improves availability of application 

**Simple Routing vs Failover vs Multi Value Routing**

- Simple routing only links one resources such as a web server and doesn't not support health checks
- Failover is used if your main application is down so you can redirect user to another page(possibly S3 bucket Status Website) 
- Multi Value is used when you have many resources that can server the request and they all can be health checked

## Weighted Routing

This is like a simple form of load balancing or when you want to test new version of Software

lets say you have 3 WWW A records and with weighted routing you can assign weight against each record

so lets say you weight records like this:

WWW 40 A 1.2.3.3

WWW 40 A 1.2.3.4

WWW 20 A 1.2.3.5

the top record will be returned 40 % of the time

Same is true for the middle record but the last record will be returned only 20 % of the time

If you set a weight to 0 that means that record is never returned but if you set all records to 0 then all of them are returned

If a record becomes unhealthy then the process of selection is repeated until a healthy record is selected

Unhealthy record doesn't cause any changed in the weightage

## Latency-Based Routing

it should be used when you what high performance and good user experience

with latency-based routing you can assign regions such as us-east-1

Latency-based routing supports one record with the same name in each AWS region

So AWS maintains a database with users source, destination location and latency

So when a user requests a record for example, from AUS then the record that's tagged with ap-southeast-2 is returned since that would provide the lowest latency based on the AWS latency database

Latency based routing can also be combined with health checks. If a record is unhealthy then the next record with lowest latency will be returned

## Geolocation Routing

With geolocation routing instead of latency the user's geo location and resources locations are used to influence address resolution

It's like a lookup where you create a record and tag it with state(in case of US), country, continent or with default

So if user location match with the tagged record, the record is delivered. if it doesn't match then the record is not delivered at all. This means the user is delivered location based resource. 

IP checks verify location of the user (depending on the DNS system, it could be resolver or the user directly)

so once you have the user location then state check is done - if a record is found with the same state then then the record is returned and the process stops. If not then the country is checked, if not found then the continent  is checked. If not of the records mast with the user locating then the default record is returned. 

**EXAM**: Remember with geo location, nearest record to user is not returned but those that match to user location is returned. This means you can do regional restrictions, language specific content or load balancing across regional endpoint.

## Geoproximity Routing

it provides records that are as close to your customer as possible

With latency based routing you the customer is provided record that is close to their region bases on estimated latency but with Geoproximity it provides records by calculating distance from customer and has some other benefits

with geoproximity, you provide the region name in case of AWS resource but it f it's an external resource you provide longitude and latitude coordinates - you also define a bias

lets say we have three resources, one in US, another in UK and a third one in Australia. Lets say someone from KSA needs to access these resources. Since there isn't any resource in Saudia arabia and the closest location from Saudia is UK so the record in UK will be delivered. 

you can change this decision by adding a bias. With bias you can define which geo locations should be served by which resource. So you can move the traffic coming from KSA to Australia instead. 

This means, rather than using actual physical distance between the user and resource, you can introduce a bias to influence the traffic to a certain location/resource. With plus you can increase the area for a resource to serve - so you can increase the bias for UK to server the whole of Europe and for Australia to serve the whole of Asia.

## R53 Interoperability

R53 provides two functions:

1- Register a domain for you 

- AWS has relationships with key TLD companies to register domain
- you pay AWS to register a domain main for you  - AWS contacts the TLD company that manages DNS for the TLD. For example for .org PIR manages the DNS.

2- Host Domain for you

- AWS can also host the domain for you by storing hosted zones in 4 server
- If you register domain via R53 and also want to host the domain then R53 will send NS records for 4 servers to TLD 

if you like you can also register your domain via thirst party such as godaddy and then host your domain in R53 by creating a public hosted zone and you will need to provide the TLD NS records via godaddy.com

if you like you can also just register domain via r53 but host your domain via 3rd party such as godaddy- this is rarely happen but this is something you can also do. In this case you can use r53 to provide the hosted domain details to TLD

EXAM: Key thing to remember is that r53 provides two separate roles. One is to register a domain on your behalf - Second to host the domain in r53. If you want  you can do either one of them and use 3rd party to do the second bit.



​	



