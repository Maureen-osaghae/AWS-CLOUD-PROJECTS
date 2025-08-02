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



   

   



