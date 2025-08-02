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



       



   

   



