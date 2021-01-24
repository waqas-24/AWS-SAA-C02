
- You are billed GB per month
- It’s an AZ based service
- You can not attach an EBS volume to an EC2 instance not in the same AZ
- One or more than one EBS can be attached to an EC2 instance
- EBS replicates data within AZ to provide resilience
- If the entire AZ fails then the EBS also fails even with resilience
- You can provide more resilience by taking volume snapshot and storing it in S3 which is then replicated within all AZs in that region
- This also allows you to create an EBS volume using this snapshot in another AZ in that region or another region.