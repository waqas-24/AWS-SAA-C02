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



