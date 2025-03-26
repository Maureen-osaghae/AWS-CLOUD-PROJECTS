<h1>Design and configure a high available 3-tier Client Architecture on AWS</h1>

Designing a 3-Tier Client Architecture on AWS teach me a variety of valuable skills, both in terms of technical knowledge and cloud architecture principles. Here’s an outline of what I learned during the design process:

✅ Cloud Architecture Design — Structuring systems for scalability, security, and cost optimization. AWS Service Familiarity — Hands-on experience with core AWS service
(EC2, RDS, VPC, etc.).

✅ Database Tier Design: Gain insight into designing the database layer of the architecture, including choosing between SQL (RDS/Aurora) and NoSQL (DynamoDB) databases.

✅ Networking & Security — Best practices for VPC, subnets, security groups, and IAM roles.

While an infrastructure can, in theory, have any number of tiers, by far the most common pattern is the 3-tier architecture. In 3-Tier architecture, An application server is a component-based product that resides in the middle tier of a server-centric architecture. It provides middleware services for security and state maintenance and also provides data access and persistence. The application servers also contain the business logic. The web servers are located at the top of the diagram. The web servers are responsible for the presentation logic. They are often accompanied by load balancers. Load balancers are responsible for efficiently distributing incoming network traffic across a group of back-end servers. The bottom of this diagram includes the database servers. This tier is responsible for the database logic.

<h2>How can business benefit?</h2>
Designing an AWS 3-Tier Client Architecture provides several business benefits, especially in terms of scalability, security, cost optimization, and high availability. Here’s why businesses choose this model: 
<h2>Scalability & Performance</h2>
<ol>
<li>Auto Scaling: Each tier can scale independently based on demand (e.g., AWS Auto Scaling, ALB).</li>
<li>Elastic Load Balancing (ELB): Distributes traffic across multiple instances, improving responsiveness.</li>
<li>Database Scaling: Amazon RDS and Aurora offer read replicas and caching with ElastiCache.</li>
</ol>
✅ Business Impact: Handles increasing workloads without affecting user experience.

<h2>Security & Compliance</h2>
<ol>
<li>Isolation of Tiers: Using AWS VPC, security groups, and subnets, each layer is protected separately.</li>
<li>Least Privilege Access: The database tier is not directly exposed to the internet, reducing attack risks.</li>
<li>AWS IAM & Security Policies: Fine-grained access control ensures that only authorized services/users access data.</li>
</ol>
✅ Business Impact: Protects sensitive data, reduces security risks, and ensures regulatory compliance (e.g., GDPR, HIPAA).

<h2>Cost Optimization</h2>
<ol>
<li>Efficient Resource Utilization: Separate tiers allow businesses to optimize computing power based on workloads.</li>
<li>Managed Services: Using AWS Lambda (serverless compute) and RDS (managed database) reduces operational overhead.</li>
</ol>
✅ Business Impact: Lowers infrastructure costs while maintaining performance.

This project describes:
<ol>
<li>Public and private subnets</li>
<li>Components in each layer of the 3-tier infrastructure</li>
<li>Simplified diagrams of a common 3-tier infrastructure</li>
<o/l>

<h2>Public and Private Layers</h2>

The 3-tier design separates an infrastructure into three layers: one public and two private layers. 

Public layer: This presentation tier is where the application’s endpoint lives, and where a user is presented with information. 
Private layer: logic tier, which contains code to translate user actions to a functionality.
Private later: data tier, which hosts the databases.
Anything in the public layer is publicly accessible, but all other content and code in the private layers is only accessible from inside the network.

For example, a user’s browser sends a request to the public tier by clicking a link on a webpage, and the request gets forwarded to the other tiers as needed to present the requested webpage or file to the user. In this tiered design, the goal is for the public layer to act as a shield to the private layers. 

![image](https://github.com/user-attachments/assets/42789565-e021-4357-9582-1262f4050601)

<h2>Detailed Breakdown of Public and Private Subnets</h2>
Understanding the roles and configurations of public and private subnets is crucial for effectively implementing the 3-tier network architecture. The main feature that makes a subnet "public" or "private" is how instances in that subnet access the internet. 

<h3>Public Subnets</h3>
Gateways to the Internet Instances/resources in a public subnet have access to and are accessible from the internet.
<h3>Internet Gateway (IGW)</h3>
Public subnets allow its instances to access the internet via an Internet Gateway. This allows resources like Application Load Balancers (ALBs) to be directly accessible from the internet, enabling users to interact with your application.
<h3>NAT Gateway</h3>
Although instances within a public subnet can access the internet, they can also route traffic to private subnets via a NAT Gateway. This configuration is particularly useful for hybrid setups where some resources need to access the internet for updates or external API calls without exposing your entire infrastructure. 
<h3>Private Subnets </h3>
Securing the Core Infrastructure
A private subnet allows its instances to access the internet via Amazon's managed NAT service (NAT Gateway), which is managed by AWS and scales out as needed. Access to the internet (when permitted) is only one-way, originating from within the subnet.

<h3>Managed NAT Service</h3> 
Instances within private subnets are protected from direct internet exposure. When these instances need to access the internet (e.g., to download updates or communicate with external APIs), they do so through a NAT Gateway located in the public subnet. This ensures that outbound traffic is secure and controlled.
<h3>Security Groups</h3>
To further enhance security, AWS Security Groups are used to tightly control the inbound and outbound traffic. Security Groups act as virtual firewalls for your instances.

Lets get started;
<h2>Task 1: Create Your VPC</h2>
Create a VPC and Subnets as well as routing and security groups

• Go to “Your VPCs” from the VPC service on the AWS management console and click on
the orange “Create VPC” button.
![image](https://github.com/user-attachments/assets/6699dcb8-68e3-46ab-af68-daba236f0741)

Only create a VPC here and give it a name. You are free to make your own name or follow along with the one put here.

Give it a 10.0.0.0/16 CIDR block and leave everything else as default. 

Click create.
![image](https://github.com/user-attachments/assets/a89edf77-7aae-4ea2-be68-4564983aa09e)

![image](https://github.com/user-attachments/assets/44a29347-b842-4f6f-ba0d-76cbecfaf972)

 To create your subnets go to Subnets on the left hand side of the VPC service and click on it.
 
 ![image](https://github.com/user-attachments/assets/2bd375e9-2c1d-4068-9d4b-af31e72a3d57)

 ● Add your VPC ID to where it asks.

Assign it a name letting you know that it is your first public subnet

● Put it in any availability zone and give it a CIDR of 10.0.0.1.0/24

![image](https://github.com/user-attachments/assets/8d678b3b-131d-43cd-8ff2-815012671952)

![image](https://github.com/user-attachments/assets/11e2268f-268e-4745-992a-4070885c5dea)

Add a second subnet and name it Private Subnet 1 or something to let you know it is your first private subnet

● Put it in the same availability zone as the first subnet you made and give it a CIDR of 10.0.2.0/24

![image](https://github.com/user-attachments/assets/6e375fe1-215d-43c0-9c4c-02fe4a7e60ee)

Add a third subnet and assign a name letting you know it is the second private subnet you will be making

● Put it in the same availability zone as your first public subnet and give it a CIDR of 10.0.3.0/24

![image](https://github.com/user-attachments/assets/8c8fe5a6-0573-4ae4-94cf-ccd696edf170)

Add a fourth and final subnet and give it a name letting you know it is the third private subnet

● Put it in a different availability zone from the rest of your subnets and give it a CIDR of 10.0.4.0/24

![image](https://github.com/user-attachments/assets/39abf25b-ca79-41af-9616-6284ff6e7926)

![image](https://github.com/user-attachments/assets/01b3e963-1f99-43a5-bf1b-18fb990c246c)

Now create an internet gateway and attach it to the VPC by going to Internet Gateways on the left hand side and clicking “Create Internet Gateway”

● Name it something similar to what is below and then click “Create Internet Gateway”

![image](https://github.com/user-attachments/assets/45b9dfef-6586-43d6-a2e7-8eb8c2d4ad2e)

![image](https://github.com/user-attachments/assets/f34ed00e-9647-4269-8b93-9efb5fbe128b)

Once it is created, attach it to your VPC by clicking “Attach to a VPC” on the top of the screen

● Click the drop down and select your VPC that you mad.

![image](https://github.com/user-attachments/assets/44122e96-c32c-4511-bc5c-10718b2de3ab)

![image](https://github.com/user-attachments/assets/7f10a4b6-09a5-406f-95a9-fc43570a1ef4)

![image](https://github.com/user-attachments/assets/48544291-171c-4ba4-8c26-72f525f6b5ff)

Allocate an Elastic IP address by going to Elastic IPs on the left hand side and click “Allocate Elastic IP address”

![image](https://github.com/user-attachments/assets/373f668f-cc7f-4242-8dbe-e0c919417023)

Everything should be good as default but make sure that you are in the same region you have been creating everything in and then press “Allocate”. You can also add a name tag if you wish but it isn’t necessary.

![image](https://github.com/user-attachments/assets/fe6585b5-31f5-4952-960c-38ab0ca3591b)

Create a NAT Gateway by clicking on Nat Gateways on the left hand side and then clicking “Create NAT Gateway”

![image](https://github.com/user-attachments/assets/8eba56b9-c3de-4233-baaa-5608c6795800)

Give it a name similar to the one below and assign it to a public subnet

● Click the drop down for Elastic IPs and click the one you created previously.

● Click “Create NAT gateway”

![image](https://github.com/user-attachments/assets/af003d69-0451-403e-938a-6844db14b62a)

![image](https://github.com/user-attachments/assets/4a7f7eec-6d44-4da5-b412-2adb8f4f5b5a)

Create Route Tables by first heading to “Route Tables” on the left hand side.

![image](https://github.com/user-attachments/assets/96e5a85c-724d-4d90-a529-d9bf53caf9f8)

Click “Create route table”

● Give it a name letting you know this is the public route table for this project.

![image](https://github.com/user-attachments/assets/7621d61f-1a77-460d-96ac-fad5d7049339)

● Assign your VPC to it and click “Create route table”

![image](https://github.com/user-attachments/assets/4105dbf3-7e40-4eb9-8269-1940821aced2)

Create a second route table naming it something to let you know that this is the private route table for your project and assign your VPC to it.

![image](https://github.com/user-attachments/assets/c4b14b00-0948-4d8b-a9bf-056a220ea240)

![image](https://github.com/user-attachments/assets/f7f20aae-f517-4ff4-94d4-ccd9317e3911)

Now associate your subnets with their respective route table

● Click on the public route table and click on “Subnet association” next to “Details”

![image](https://github.com/user-attachments/assets/11dbf9f5-9008-4b87-8030-37d9a67fa974)

![image](https://github.com/user-attachments/assets/9034b3ce-2859-42d3-9010-e25b05409d62)

Click on your public subnet and then click “Save associations”. Now add a route to our public route table to get access to the internet gateway. Click on “Routes” next to “Details” and click “Edit routes” Add a new route having a destination of anywhere and a target of your internet gateway and click “Save changes”.

![image](https://github.com/user-attachments/assets/50a14389-e7ad-4a48-a94e-66d062bb6c44)

![image](https://github.com/user-attachments/assets/d834860e-e934-407d-95fe-342fe1762030)

Do the same thing for your private route table by clicking on it and going to its subnet associations and editing them. Click on all three of your private subnets and save the associations.

![image](https://github.com/user-attachments/assets/578d3496-371f-476a-aaac-cc9e9ea4556d)

Go to edit the routes of the private table Add a route to the private table that has a destination of anywhere and a target of your Nat gateway that you created earlier.

![image](https://github.com/user-attachments/assets/61c60c75-f714-4890-b196-70d03618d380)

![image](https://github.com/user-attachments/assets/ce66a0a9-fdc2-49aa-b465-f5bc543fad1f)

 Create Your Security Groups</h2>

Now to create our security groups (One for our bastion host, web server, app server, and our database). we will head to Security Groups on the left and click “Create security group”.

![image](https://github.com/user-attachments/assets/72d3dcc4-b184-42d6-8a85-e7dda3b64877)

Give it a name and description letting you know it is for a bastion host Assign your VPC to it.

![image](https://github.com/user-attachments/assets/9b895c77-25c2-4ce7-a296-fa62b59a3871)

Give it three inbound rules, one for SSH using your IP and one for HTTP using 0.0.0.0/0 as well as https using 0.0.0.0/0

![image](https://github.com/user-attachments/assets/8feaf179-739a-40c1-9d40-ce58165f4d0d)

![image](https://github.com/user-attachments/assets/db6f817a-aa95-4a15-baf9-39f29501ae3d)

Create another security group Give it a name and description letting you know it is for a Web server Assign your VPC to it.

![image](https://github.com/user-attachments/assets/77d024fe-d297-41bc-a313-2c803a50cd0b)

Give it the same inbound rules as the Bastion Host security group.

![image](https://github.com/user-attachments/assets/81d46a23-e26d-4b47-bd1b-bbb49635902d)

![image](https://github.com/user-attachments/assets/a0940976-0764-4b7d-ad63-8a9a191b08ed)

Create another security group Give it a name and description letting you know it is for an app server.

![image](https://github.com/user-attachments/assets/4ef35213-727f-4cd9-8a33-e2d8396eebc2)

Assign your VPC to it Give it an inbound rule for All ICMP -IPv4 with a source of your web server SG and another inbound rule for SSH with a source of your bastion host SG.

![image](https://github.com/user-attachments/assets/d9cddbb4-65c5-4528-96a9-8d2b1c54656d)

![image](https://github.com/user-attachments/assets/206474c2-a3cb-4c8e-b4be-7b7281408811)

Create one final security group. Give it a name and description letting you know it is for a database server. Assign your VPC to it.

![image](https://github.com/user-attachments/assets/4e1fb31c-f76b-4fa3-88d1-a245fdc2b3c5)

Give it two inbound rules both for MYSQL/Aurora and give one of them a source of your app server SG and the other one a source of your bastion host SG.

![image](https://github.com/user-attachments/assets/8ce57db1-fe57-471d-b9a8-29744acaf012)

![image](https://github.com/user-attachments/assets/d0de1447-a85c-4298-9bf0-027c38f98dc7)

Go back to your bastion host inbound rules and add one more for MYSQL/Aurora and a source of your database SG.

![image](https://github.com/user-attachments/assets/9721e0d7-5d50-41a7-a9c9-5813afb6f07d)

Go back to your web server inbound rules and add one more for All ICMP — IPv4 and a source of your app server SG.

![image](https://github.com/user-attachments/assets/30043831-7514-4d2e-b5c3-839c7d9849db)

Go back to your app server inbound rules and add one more for MYSQL/Aurora and a source of your database SG and then an HTTP and HTTPS rule both with a source of 0.0.0.0/0.

![image](https://github.com/user-attachments/assets/00d25e3f-5599-446d-95b2-b43c3d38b096)

<h2>Tast 2. Create Servers</h2>

In this task, you launch an EC2 instance into the new VPC. You configure the instance to act as a web server. On the AWS Management Console, in the Search bar, enter and choose EC2 to go to the EC2 Management Console.

![image](https://github.com/user-attachments/assets/4868fb19-b48a-4d12-a55f-9665d66f38d3)

<h3>Create Bastion Host Server</h3>

![image](https://github.com/user-attachments/assets/1c61820d-c05e-46c4-bc63-c3a227e8d2ce)

Select Amazon Linux 2 AMI

![image](https://github.com/user-attachments/assets/4b7c2b1c-3e07-47c8-a12e-56bb05bccf06)

Instance type: Select t2.micro

![image](https://github.com/user-attachments/assets/2ba06ba2-5b25-4b16-bce9-09e7ac3f5227)

Put in your VPC and Public Subnet and enable auto assign public IP. Select an existing group and select your Bastion Host SG.

![image](https://github.com/user-attachments/assets/903da5d8-7781-45db-ac8c-b1962417e9f2)

Storage leave default. Add a name tag to let you know this is the Bastion Host Launch and choose an existing keypair. This can be downloaded from the lab page.

![image](https://github.com/user-attachments/assets/a1c6fc63-b9f5-46b9-9872-7f2411af66ae)

![image](https://github.com/user-attachments/assets/0551db5c-ec13-494e-8d89-d51cfaa4a09c)

To create the Web Server follow the same steps until you get to Step 3

![image](https://github.com/user-attachments/assets/38febb78-4e42-4abb-afe1-821fdff34ede)

Follow along like previously and change your network, and enable auto assign public ip.

![image](https://github.com/user-attachments/assets/22d46f2d-4c5a-4c1c-a8ff-d2228f227e1f)

Then go to user data and type this into it to set up the web server

    #!/bin/bash
    sudo yum update -y
    sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
    Sudo yum install -y httpd
    sudo systemctl start httpd
    sudo systemctl enable httpd

![image](https://github.com/user-attachments/assets/d64a8608-cca2-41ad-9c36-679293221f61)

Just like before launch and use the existing keypair

Follow the same steps once again to create the app server until you get to step 3

![image](https://github.com/user-attachments/assets/0cf4df1e-dbb4-409f-a3f6-0b4a6fc5f150)

Put in your VPC and then choose Private Subnet 1 for the subnet and leave auto assign public IP disabled.

![image](https://github.com/user-attachments/assets/045ae724-10c9-4df7-b20a-3fc651971deb)
Then go into metadata and type this out to set up a database server on our app server
     
    #!/bin/bash
    sudo yum install -y mariadb-server
    Sudo service mariadb start
    
![image](https://github.com/user-attachments/assets/ebe8df10-576f-4294-aa56-7ef2d8ec3ee3)

Give it a name letting you know it is the App Server

● Select an existing security group and select the app server SG

● Just like before launch and use the existing keypair

![image](https://github.com/user-attachments/assets/ecf09241-5602-43b4-ab88-44bc3767d2e5)

<h2>Task 3: Create a Database </h2>
Create a DB subnet group by first heading to the Amazon RDS service page on the AWS management console.

![image](https://github.com/user-attachments/assets/8361ada4-0a23-46b4-acf0-ada5b5b21620)

Click on Subnet Groups on the left hand side and the click on “Create DB subnet group”.

Give it a name and description letting you know what it is and then assign your VPC to it. Put in the availability zones you used for your subnets.

![image](https://github.com/user-attachments/assets/58b2104c-b078-4978-96a0-b46da04ba6cf)

Select subnets 2 and 3

![image](https://github.com/user-attachments/assets/fb4d2966-7b5c-46e8-aa2a-c4bda361d9eb)

![image](https://github.com/user-attachments/assets/716b0bec-6329-4da6-aad6-0fb67caf1207)

Go to Databases on the left hand side and click on “Create Database”.

![image](https://github.com/user-attachments/assets/54530f90-775e-418d-8cda-ee63c87628ad)

Click on Standard create and MariaDB for the engine type.

![image](https://github.com/user-attachments/assets/56c1a5f2-1f91-4003-82b8-6c206279d816)

Make sure you click on dev/test here.

![image](https://github.com/user-attachments/assets/906a0754-adc6-480c-a7ff-0f01f2a07a4b)

Give it an identifier you can easily identify it with.

● Give it a master username or leave it as default admin. For the purpose of this project, I will be using root

● Give it a password that you write down somewhere else to make sure you have the correct one. For the purpose of this project, I will be using Re:Start!9

![image](https://github.com/user-attachments/assets/499af228-6ac5-4cf1-a63c-bbc4873acb23)

Everything between this and the last step is left default.
Assign your VPC
✅ Make sure your subnet group is listed under the subnet group section

✅ Public access is No

✅ Choose existing VPC security groups

✅ Remove the default security group and add your database security group

✅ Select your first availability zone as well

![image](https://github.com/user-attachments/assets/01ba94c7-d320-41f5-b4fc-a049fd399ef4)

![image](https://github.com/user-attachments/assets/84f60099-fa89-4932-825b-6f881881259e)

Scroll down to Additional configuration on the bottom and give it an initial database name and save it in the same spot as your password since it will be used later.

Disable automated backups and encryption since they are not needed (These are normally best practice to leave enabled but the database will spin up faster with those checked off as they are not needed).

![image](https://github.com/user-attachments/assets/6b7a6700-ed95-4da1-ab98-2e30233868b8)

Scroll down all the way to the bottom and create your database

![image](https://github.com/user-attachments/assets/64c9e681-d45d-4181-808e-93df21e4f1e7)

![image](https://github.com/user-attachments/assets/dfa3be72-6a09-4fec-9c4b-4cc152a159fa)

<h2>Task 4: Test connections</h2>

● SSH into your Bastion Host after downloading both the .pem and .ppk files from the lab environment.

This is for windows only as I only have a windows machine to work on, sorry Mac and Linux users. Go into your powershell and type this command out to download the keys.

    pscp -scp -P 22 -i "C:\Users\YourUsername\Downloads\labsuser.ppk" "C:\Users\YourUsername\Downloads\labsuser.pem" ec2-user@BASTION-HOST-PUBLIC-IP:/home/ec2-user/

![image](https://github.com/user-attachments/assets/0da5109d-491c-404c-8a99-b429e97d3992)

After downloading the keys to my Bastion host, SSH into my bastion host Instance.

![image](https://github.com/user-attachments/assets/029ec55b-4f33-4836-9dec-6daa1f72be7c)

Change file permissions for the file you just downloaded to your bastion host by typing chmod 400 labsuser.pem

Then run the following command ssh -i labsuser.pem ec2-user@APP-SERVER-PRIVATE-IP to SSH into my APP Server instance.

Replace APP-SERVER-PRIVATE-IP with the private IP of your app server instance.

![image](https://github.com/user-attachments/assets/cedd6a0d-b456-4194-8eae-a72fab63ba90)

Use ls to see that you are now SSH into a APP Server instance since there is no more labuser.pem key.

<h2>Ping the Web Server Private IP</h2>

Check connectivity from the app server to the Web Server.

Use ping and the private IP address of your web server to ping the web server and see it connect. ping WEB-SERVER-PRIVATE-IP

![image](https://github.com/user-attachments/assets/6bfeca4c-48be-47ff-9733-63bbdac4ead7)

Test out connecting to the database by typing out

mariadb — user=root — password=’Re:Start!9' — host=databse-end-point

<h2>Challenge:</h2>

I Found it difficult to connect to my mariadb database, then I had to troubleshoot.

first I install mariadb client: sudo dnf install -y mariadb105

Then Verify installation with mariadb --version
Try connecting to my database again.

Replace database-server-endpoint with the database server endpoint

Type show databases; to see your database from the app server

![image](https://github.com/user-attachments/assets/fb540a59-55a4-466a-8c20-11e8e8bba310)

<h2>Conclusion:</h2>
Why Businesses Choose AWS 3-Tier Architecture

✔ Scalability — Handle traffic spikes seamlessly.

✔ Security — Reduce attack risks with proper isolation.

✔ Cost Savings — Optimize resources.

✔ High Availability — Minimize downtime with AWS failover features.

✔ Future-Ready — Supports microservices, cloud-native development.




















































































































