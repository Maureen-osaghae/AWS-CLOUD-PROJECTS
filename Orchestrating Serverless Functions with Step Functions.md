<h1>Orchestrating Serverless Functions with Step Functions</h1>
<h2>Lab overview and objectives</h2>
In this lab, I will use AWS Step Functions to coordinate the actions necessary to generate and deliver a report upon request. The report will contain data from a database.

After completing this lab, I was able able to:
    
•  Create an asynchronous state machine by using AWS Step Functions
    
•  Configure an Amazon Simple Notification Service (Amazon SNS) topic to deliver email alerts
      
•  Configure AWS Lambda functions to be invoked from a Step Functions state machine
      
•  Use a parallel state flow object in the design of a Step Functions state machine
      
•  Invoke a state machine to start when a REST API endpoint is invoked
      
•  Generate a presigned URL for an object stored in an Amazon Simple Storage Service (Amazon S3) bucket

<h2>Business Scenario</h2>
Thanks to a recent update, the coffee suppliers' inventory now automatically refreshes on the café website. However, the task is not yet complete. A request was made to enable website users to log in and retrieve a report with the most up-to-date inventory information. Recently, an AWS consultant visited the café and, during a conversation over coffee, learned about the need to build a reporting mechanism. After reviewing the high-level business requirements, the consultant recommended using AWS Step Functions to coordinate the report generation process. Step Functions allow developers to automate business workflows, support parallel execution, and integrate seamlessly with other AWS services.

In this lab, the functionality to generate and deliver the inventory report will be implemented. In the following lab, the design will be enhanced to include authentication, ensuring that only authorized users can request and access reports.

The following diagram shows the architecture of the previous labs that I created in the previous labs. At the START of the lab, the infrastructure was recreated.

<img width="551" height="259" alt="image" src="https://github.com/user-attachments/assets/3e3c6db1-24f0-43d5-9874-c020512e1328" />

By the end of this lab, you will have created the architecture shown in the following diagram.

<img width="689" height="275" alt="image" src="https://github.com/user-attachments/assets/b3fb286c-a88c-4779-adf4-9c2d8a6f8f79" />

Now, lets explain the diagram.

1. A user (Frank) requests a report by using the create_report REST API endpoint. I created this REST API in a previous lab. The request invokes a Step Functions state machine that I will build.

2. The state machine first invokes a Lambda function that looks up the latest coffee supply information. The information is in a MySQL database hosted on Amazon Relational Database Service (Amazon RDS).

3. The state machine then invokes two Lambda functions to run in parallel.

4. One of the functions transforms the JSON data from the database into an HTML report that is nicely formatted for readability. The report is uploaded to an S3 bucket.

5. The other function generates a presigned URL to access the report and then sends the presigned URL to an SNS topic.

6. Frank is subscribed to the SNS topic, so he receives an email with the presigned URL. He can then access the report from Amazon S3. Only he knows the presigned URL, and it will expire after a short period.

In this lab, I will again play the role of Sofía to build the functionality described. Let's get started!

<h2>Task 1: Preparing the development environment</h2>
In this first task, I will configure your Visual Studio Code Integrated Development Environment (VS Code IDE). I will also run the script to re-create the work I completed in previous labs.

Before proceeding to the next step, verify that the AWS CloudFormation stack creation process for the lab has successfully completed.

◦ In a new browser tab, navigate to the CloudFormation console.
        
◦ In the left navigation pane, choose Stacks.
       
◦ For the stack with "ACD_2.0" in the Description column, verify that the Status says CREATE_COMPLETE. 

<img width="959" height="427" alt="image" src="https://github.com/user-attachments/assets/3ef5c665-c72e-4016-b2b2-c2f3107fbf3f" />

Connect to the VS Code IDE.
<img width="854" height="398" alt="image" src="https://github.com/user-attachments/assets/42287666-97c0-4dbf-b8cf-945626b63940" />

Downloaded and extract the files that I need for this lab.
        
◦ In the VS Code IDE bash terminal, run the following command:

    wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-44869/11-lab-step/code.zip -P /home/ec2-user/environment
          
◦ The code.zip file is downloaded to the the VS Code IDE. The file is listed in the left navigation pane.  Extract the file:
         
          unzip code.zip

<img width="505" height="394" alt="image" src="https://github.com/user-attachments/assets/3526041d-9911-4820-ae9a-f5b625d97dae" />

Run a script to the configure VS Code IDE and re-create the work that I completed in earlier labs into this AWS account. Set permissions on the script so that I can run it, and then run it:
          
          chmod +x ./resources/setup.sh && ./resources/setup.sh
       
<h2>Analysis:</h2> The CloudFormation template that ran when I started this lab created resources in the AWS account. The script I just ran also created resources. Between the two of them, the following resources have been created to replicate what I built in previous labs:

◦ An S3 bucket with an associated bucket policy. The bucket contains the café website code.

◦ An Amazon DynamoDB table populated with menu data.

◦ A REST API configured using Amazon API Gateway.

◦ A Lambda function that retrieves data from DynamoDB when invoked.

◦ A Memcached cluster that caches supplier data from Amazon Aurora Serverless for the suppliers application.

◦ A café/node-web-app Docker image, which is stored in the Amazon Elastic Container Registry (Amazon ECR).

◦ An AWS Elastic Beanstalk environment and application that runs an Amazon Elastic Compute Cloud (Amazon EC2) instance named MyEnv. The EC2 instance hosts a Docker container created from the Docker image that is stored in Amazon ECR.
        
◦ An Aurora Serverless database running MySQL on Amazon RDS, which contains the supplierdb database, which stores coffee supplier information.

 Verify the version of AWS CLI installed. In the VS Code IDE Bash terminal (at the bottom of the IDE), run the following command:
          
          aws --version
The output should indicate that version 2 is installed.
          
Verify that the SDK for Python is installed. Run the following command:
          
          pip3 show boto3

I Verify that I can access the café website.
        
◦ Navigate to the Amazon S3 console.

◦ Choose the link for the bucket that has -s3bucket in the name.
        
◦ Open the index.html file.

<img width="777" height="100" alt="image" src="https://github.com/user-attachments/assets/57415e5e-1634-4529-9376-65138b4b5bc9" />

In the Object overview section, open the Object URL in a new browser tab.

<img width="791" height="134" alt="image" src="https://github.com/user-attachments/assets/d40be8fe-1c03-46a7-a9ac-2f9b8b3ee589" />

The café website displays. 

<img width="959" height="451" alt="image" src="https://github.com/user-attachments/assets/9c64702a-2a63-4f92-9c0b-64c7fb5bf24d" />

<h2>Task 2: Configuring an SNS topic and subscribing to it</h2>
In this task, I will configure an SNS topic that can be used to alert Frank when a requested report is available. 
The following diagram shows the solution architecture for this lab, with the part that I will build in tasks 2 and 3 highlighted.

<img width="650" height="284" alt="image" src="https://github.com/user-attachments/assets/3888d14f-8bb0-46a0-8f10-a8409af3a83a" />

Create an SNS topic for email notification.

◦ Navigate to the Amazon SNS console.

<img width="953" height="336" alt="image" src="https://github.com/user-attachments/assets/2c1e8111-3f85-4104-b82b-5c0b0a71a32d" />

In the left navigation pane, choose Topics.

<img width="931" height="232" alt="image" src="https://github.com/user-attachments/assets/29d94af2-dfbf-4143-88fd-4141ca625a16" />

◦ Choose Create topic and configure the following:

▪ Type: Choose Standard.

▪ Name: Enter EmailReport

▪ Expand the Access policy - optional section.
            
▪ Define who can publish messages to the topic: Choose Everyone.
           
▪ Define who can subscribe to this topic: Choose Everyone.

◦ At the bottom of the page, choose Create topic.

<img width="862" height="417" alt="image" src="https://github.com/user-attachments/assets/f1b3d4a7-add7-4272-bdd5-bcd99ca31176" />

Create an email subscription to the SNS topic.

◦ Choose Create subscription and configure the following:

▪ Topic ARN: Notice that the Amazon Resource Number (ARN) of the topic that you just created is already filled in.

▪ Protocol: Choose Email.

<img width="959" height="302" alt="image" src="https://github.com/user-attachments/assets/876c9a87-e9ea-4325-b7b7-2a6253982262" />

<h3>Endpoint:</h3> Enter an email address where I can receive emails during this lab. (In the café story, Sofía would enter Frank's email address.)
        
◦ Choose Create subscription.

<img width="770" height="361" alt="image" src="https://github.com/user-attachments/assets/dd85263d-bf58-4a45-800d-0aeac90a00fb" />

Check my email and confirm the subscription.

◦ Check email for a message from AWS Notifications.

◦ In the email body, choose the Confirm subscription link.

◦ A webpage opens and displays a message that the subscription was successfully confirmed.

<img width="484" height="320" alt="image" src="https://github.com/user-attachments/assets/46038544-dec2-4872-9395-71dc5e5c6fc4" />

<img width="818" height="152" alt="image" src="https://github.com/user-attachments/assets/e68aa2f6-fbfe-4bc3-aee5-74d91e17fac8" />

Publish a test message to confirm that messages can be published to the EmailReport SNS topic.

◦ Return to the Amazon SNS console, and return to the EmailReport topic page. Tip: In the breadcrumbs at the top of the page, choose EmailReport.
        
◦ Choose Publish message and configure the following:
        
▪ Subject - optional: Enter Test
            
▪ Message body: Enter Hello! This is a test.
        
◦ At the bottom of the page, choose Publish message.

<img width="959" height="353" alt="image" src="https://github.com/user-attachments/assets/4ea685a5-3f6e-4e12-94b3-5349d3e32b08" />

<img width="765" height="337" alt="image" src="https://github.com/user-attachments/assets/448997e3-bdf0-4b4d-90a1-c01770064055" />

Confirm that I received the message at my email address.
The subject and message body should match the details that you configured in the previous step.

<img width="734" height="332" alt="image" src="https://github.com/user-attachments/assets/8b0a828b-72d5-41ad-a8b2-b7d8abd4e7b3" />

<h2>Task 3: Creating a Step Functions state machine</h2>
In this task, I will create a Step Functions state machine that can send an email notification by using the SNS topic.
The Step Functions state machine will need permissions to access the Lambda service. Start by looking at an AWS Identity and Access Management (IAM) role that has already been created for me for this purpose.

<h3>Review the IAM role for Step Functions</h3>
      
Analyze the details of the IAM role that Step Functions will use.

◦ Navigate to the IAM console.

◦ Review the details of the RoleForStepToCreateAReport IAM role.

▪ In the left navigation pane, choose Roles.
            
▪ In the search box under Roles, search for and choose the RoleForStepToCreateAReport role.

<img width="811" height="227" alt="image" src="https://github.com/user-attachments/assets/9bfdaf06-fcc8-4079-aba7-dd93e31e683d" />

On the Permissions tab, expand the stepPolicyForCreateReport policy, and choose {} JSON. I notice that this role allows all Lambda and log actions on all resources.

<img width="791" height="347" alt="image" src="https://github.com/user-attachments/assets/6eff6455-c077-4ffd-9ab5-f711e1245e42" />

▪ Expand the AWSLambdaRole managed policy, which is also attached to the role. I also notice that this role allows the lambda:InvokeFunction action on all resources. This will allow me to test the function from the Lambda console.

<img width="838" height="350" alt="image" src="https://github.com/user-attachments/assets/cf48cad7-3d81-4a1b-8032-22eb3a1b233f" />

Choose the Trust relationships tab. This role allows the Step Functions service to assume this role (as indicated by the states.amazonaws.com service endpoint).

<img width="749" height="257" alt="image" src="https://github.com/user-attachments/assets/64b424d5-2620-402c-a892-7cef97bd5588" />

<span>Note</span> in this lab environment, AWS cannot grant me permissions to create an IAM role. Therefore, later in this lab, I will observe the details for other IAM roles that have already been created for me. However, in an AWS account where I have more permissions to use the IAM service, I could create the IAM role that I just observed, attach the managed policy, and also create the custom IAM policy and attach it to the role.

<h2>Create a state machine to send an email</h2> 
Begin to create a Step Functions state machine. Navigate to the Step Functions console.

<img width="902" height="291" alt="image" src="https://github.com/user-attachments/assets/e1308d66-990a-4952-b085-9d925ed46165" />

In the left navigation pane, choose State machines.

<img width="887" height="262" alt="image" src="https://github.com/user-attachments/assets/aba5e51b-933e-44ab-a739-7566c841686f" />

◦ Choose Create state machine and configure the following for step 1:
           
▪ Choose a template: Choose Blank.

Choose Select.

<img width="940" height="411" alt="image" src="https://github.com/user-attachments/assets/7055abbc-bbde-488d-8b9f-fb1478336550" />

The Workflow Studio appears for step 2.

<img width="959" height="409" alt="image" src="https://github.com/user-attachments/assets/5da9c402-9a3b-49d1-b501-ed62cba8901f" />

Design the workflow.
    • In the search box under Step 2: Design workflow, enter SNS
    • Drag the Amazon SNS Publish object onto the canvas, to the box labeled Drag first state here.
    • In the SNS Publish pane that displays, configure the following:
        ◦ Topic: Choose the ARN of the EmailReport SNS topic that you created earlier.




























       



   

   



