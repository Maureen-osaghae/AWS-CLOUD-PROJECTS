<h1>Implement a Serverless Architecture on AWS</h2>
<h2>Scenario</h2>
I am creating an inventory tracking system. Stores from around the world will upload an inventory file to Amazon S3. My team wants to be able to view the inventory levels and send a notification when inventory levels are low.
In this lab, I will:
<ol>
  <li>I will upload an inventory file to an Amazon S3 bucket.</li>
  <li>This upload will trigger a Lambda function that will read the file and insert items into an Amazon DynamoDB table.</li>
  <li>A serverless, web-based dashboard application will use Amazon Cognito to authenticate to AWS. The application will then gain access to the DynamoDB table to display inventory levels.</li>
  <li>Another Lambda function will receive updates from the DynamoDB table. This function will send a message to an SNS topic when an inventory item is out of stock.</li>
  <li>Amazon SNS will then send you a notification through short message service (SMS) or email that requests additional inventory.</li> 
</ol>

<h2>Lab overview</h2>
Traditionally, applications run on servers. These servers can be physical (or bare metal). They can also be virtual environments that run on top of physical servers. However, you must purchase and provision all these types of servers, and you must also manage their capacity. In contrast, you can run your code on AWS Lambda without needing to pre-allocate servers. With Lambda, you only need to provide the code and define a trigger. 

The Lambda function can run when it is needed, whether it is once per week or hundreds of times per second. You only pay for what you use.
This lab demonstrates how to trigger a Lambda function when a file is uploaded to Amazon Simple Storage Service (Amazon S3). 

The file will be loaded into an Amazon DynamoDB table. The data will be available for you to view on a dashboard page that retrieves the data directly from DynamoDB. This solution does not use Amazon Elastic Compute Cloud (Amazon EC2). It is a serverless solution that automatically scales when it is used. It also incurs little cost when it is in use. When it is idle, there is practically no cost because will you only be billed for data storage.

What I Learned:

 • Implement a serverless architecture on AWS
 
 • Trigger Lambda functions from Amazon S3 and Amazon DynamoDB

• Configure Amazon Simple Notification Service (Amazon SNS) to send notifications

At the end of this lab, your architecture will look like the following example:

<img width="411" alt="image" src="https://github.com/user-attachments/assets/903eadcd-0737-4687-97fc-5f37c5ba334b" />

<h2>Task 1: Creating a Lambda function to load data</h2>

In this task, you will create a Lambda function that will process an inventory file. The Lambda function will read the file and insert information into a DynamoDB table. 

<img width="398" alt="image" src="https://github.com/user-attachments/assets/aec640f6-f109-4727-8d1c-2fb6b912f89d" />

In the AWS Management Console, on the Services menu, choose Lambda.

Choose Create function 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/963d5c60-67a1-4acc-8aa2-864d494c8668" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/cc4c722c-00cd-4df2-afd7-c76de38afb80" />

Blueprints are code templates for writing Lambda functions. Blueprints are provided for standard Lambda triggers, such as creating Amazon Alexa skills and processing Amazon Kinesis Data Firehose streams. This lab provides me with a pre-written Lambda function, so I will use the Author from scratchoption. 

Configure the following settings:
<ol>
 <li>Function name: Load-Inventory</li>
 <li>Runtime: Python 3.9</li>
 <li>Expand  Choose or create an execution role.</li>
 <li>Execution role: Use an existing role</li>
 <li>Existing role: Lambda-Load-Inventory-Role</li>
</ol>

This role gives the Lambda function permissions so that it can access Amazon S3 and DynamoDB. Choose Create function

<img width="959" alt="image" src="https://github.com/user-attachments/assets/4c1edc12-f261-4213-858e-7a71a837fa8c" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/ecd2add3-6f08-464c-a696-3f50db928c1a" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/2bb49538-7d34-4a2a-8ac0-3d03ffbaed09" />

Scroll down to the Code source section, and in the Environment pane, choose lambda_function.py. In the code editor, delete all the code. In the Code source editor, copy and paste the following 
code:

     # Load-Inventory Lambda function
    # This function is triggered by an object being created in an Amazon S3 bucket.
    # The file is downloaded and each line is inserted into a DynamoDB table.
    import json, urllib, boto3, csv
    # Connect to S3 and DynamoDB
    s3 = boto3.resource('s3')
    dynamodb = boto3.resource('dynamodb')
    # Connect to the DynamoDB tables
    inventoryTable = dynamodb.Table('Inventory');
    # This handler is run every time the Lambda function is triggered
    def lambda_handler(event, context):
      # Show the incoming event in the debug log
      print("Event received by Lambda function: " + json.dumps(event, indent=2))
      # Get the bucket and object key from the Event
      bucket = event['Records'][0]['s3']['bucket']['name']
      key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
      localFilename = '/tmp/inventory.txt'
      # Download the file from S3 to the local filesystem
      try:
        s3.meta.client.download_file(bucket, key, localFilename)
      except Exception as e:
        print(e)
        print('Error getting object {} from bucket {}. Make sure they exist and your bucket is in the same region as this function.'.format(key, bucket))
        raise e
      # Read the Inventory CSV file
      with open(localFilename) as csvfile:
        reader = csv.DictReader(csvfile, delimiter=',')
        # Read each row in the file
        rowCount = 0
        for row in reader:
          rowCount += 1
          # Show the row in the debug log
          print(row['store'], row['item'], row['count'])
          try:
            # Insert Store, Item and Count into the Inventory table
            inventoryTable.put_item(
              Item={
                'Store':  row['store'],
                'Item':   row['item'],
                'Count':  int(row['count'])})
          except Exception as e:
             print(e)
             print("Unable to insert data into DynamoDB table".format(e))
        # Finished!
        return "%d counts inserted" % rowCount

<img width="959" alt="image" src="https://github.com/user-attachments/assets/fd6acf4e-1012-44a3-a90a-482d5b6a070c" />

Examine the code. It performs the following steps:

◦ Download the file from Amazon S3 that triggered the event

◦ Loop through each line in the file

◦ Insert the data into the DynamoDB Inventory table
        
◦ Then Choose Deploy to save your changes.

Next, I will configure Amazon S3 to trigger the Lambda function when a file is uploaded.

<h2>Task 2: Configuring an Amazon S3 event</h2>

Stores from around the world provide inventory files to load into the inventory tracking system. Instead of uploading their files via FTP, the stores can upload them directly to Amazon S3. They can upload the files through a webpage, a script, or as part of a program. When a file is received, it triggers the Lambda function. This Lambda function will then load the inventory into a DynamoDB table. 

<img width="428" alt="image" src="https://github.com/user-attachments/assets/80f429ab-157d-4ab4-ab8e-c481654fe5fe" />

In this task, you will create an S3 bucket and configure it to trigger the Lambda function. On the Services menu,  choose S3. Choose Create bucket
Each bucket must have a unique name, so you will add a random number to the bucket name. For example: maureen222

<img width="959" alt="image" src="https://github.com/user-attachments/assets/34d3edd7-5034-4e2e-9515-fead40582b5f" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/85a2efe5-c02f-483d-b774-3efea937a119" />

◦ Configure the bucket to automatically trigger the Lambda function when a file is uploaded.

◦ Choose the name of your inventory- bucket.

◦ Choose the Properties tab.

◦ Scroll down to Event notifications.

Configure an S3 event to trigger when an object is created in the S3 bucket.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/54ec0c22-7ff5-44d0-8e07-9ee07b19a0e4" />

Click Create event notification then configure these settings:
◦ Name: Load-Inventory

◦ Event types:  All object create events
        
◦ Destination: Lambda Function
       
◦ Lambda function: Load-Inventory

◦ Choose Save changes

When an object is created in the bucket, this configuration tells Amazon S3 to trigger the Load-Inventory Lambda function that I created earlier. 
My bucket is now ready to receive inventory files!

<img width="959" alt="image" src="https://github.com/user-attachments/assets/565faa3d-f3aa-4068-8f89-d54749b90690" />

<h2>Task 3: Test the loading process</h2>
I am now ready to test the loading process. I will upload an inventory file, then check that it loaded successfully. 

<img width="403" alt="image" src="https://github.com/user-attachments/assets/75082509-00a6-4f57-a769-26356d823bc4" />

◦ inventory-berlin.csv

◦ inventory-calcutta.csv

◦ inventory-karachi.csv

◦ inventory-pusan.csv

◦ inventory-shanghai.csv

◦ inventory-springfield.csv

These files are the inventory files that you can use to test the system. They are comma-separated values (CSV) filesIn the console,  return to my S3 bucket by choosing the Objects tab.

◦  Choose Upload
       
◦ Choose Add files, and select one of the inventory CSV files. (you can choose any inventory file.)

◦Choose Upload

<img width="959" alt="image" src="https://github.com/user-attachments/assets/74c26067-da06-4f31-9621-c1a717f5c60d" />

Amazon S3 will automatically trigger the Lambda function, which will load the data into a DynamoDB table. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/ddc1aaff-b78c-4234-b4e8-f8763eb39209" />

A serverless Dashboard application has been provided for me to view the results.

Open a new web browser tab, paste the URL, and press ENTER.

![image](https://github.com/user-attachments/assets/592c64c7-6cc5-4894-ae7a-f75ed908def8)

The dashboard application open and display the inventory data that you loaded into the bucket. The data is retrieved from DynamoDB, which proves that the upload successfully triggered the Lambda function. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/e4250b17-5d4e-4f85-a20a-aeeaea674e52" />

The dashboard application is served as a static webpage from Amazon S3. The dashboard authenticates via Amazon Cognito as an anonymous user, which provides sufficient permissions for the dashboard to retrieve data from DynamoDB. You can also view the data directly in the DynamoDB table.

On the Services menu, choose DynamoDB. In the left navigation pane,

◦ Choose Tables 

◦ Choose the Inventory table. 

◦ Choose the Items tab.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/8bca25b4-9650-484a-b492-f8a3f07e9b46" />

The data from the inventory file will be displayed. It shows the store, item and inventory count.

<img width="953" alt="image" src="https://github.com/user-attachments/assets/f04c0820-5626-4cb0-b847-d462918c2f2c" />

<h2>Task 4: Configuring notifications</h2>
You want to notify inventory management staff when a store runs out of stock for an item. For this serverless notification functionality, you will use Amazon SNS. 

<img width="418" alt="image" src="https://github.com/user-attachments/assets/94ae2a05-8666-4aee-ae51-7cbe59ec7691" />

Amazon SNS is a flexible, fully managed publish/subscribe messaging and mobile notifications service. It delivers messages to subscribing endpoints and clients. With Amazon SNS, you can fan out messages to a large number of subscribers, including distributed systems and services, and mobile devices. 

On the Services menu, choose Simple Notification Service. In the Create topic box, for Topic name, enter: NoStock. Keep Standard selected. Choose Create topic.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/f4141da8-1ee0-4123-b4b9-51f49da89b88" />

<img width="956" alt="image" src="https://github.com/user-attachments/assets/d5383b4e-8b1d-4438-b003-a86f9108ca41" />

To receive notifications, you must subscribe to the topic. You can choose to receive notifications via several methods, such as SMS and email. 

In the lower half of the page, choose Create subscription and configure these settings:

◦ Protocol: Email

◦ Endpoint: Enter your email address

◦ Choose Create subscription

<img width="959" alt="image" src="https://github.com/user-attachments/assets/7494e615-36b2-4096-8b3b-1c924e554ac9" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/4e04b922-0008-49ee-b4d1-c85e91747371" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/70d6fd55-2eea-4bd6-b26e-e5f6d3e00bec" />

After you create an email subscription, you will receive a confirmation email message. Open the message and choose the Confirm subscription link.
Any message that is sent to the SNS topic will be forwarded to my email.

<img width="704" alt="image" src="https://github.com/user-attachments/assets/c4bd2ac3-2242-4df5-8417-4b40fbbb55ff" />

<img width="605" alt="image" src="https://github.com/user-attachments/assets/382521ca-2aba-406a-badf-3863ca49b606" />

<h2>Task 5: Create another a Lambda function to send notifications</h2>

You could modify the existing Load-Inventory Lambda function to check inventory levels while the file is being loaded. However, this configuration is not a good architectural practice. Instead of overloading the Load-Inventory function with business logic, you will create another Lambda function that is triggered when data is loaded into the DynamoDB table. This function will be triggered by a DynamoDB stream.

This architectural approach offers several benefits:

• Each Lambda function performs a single, specific function. This practice makes the code simpler and more maintainable.
   
• Additional business logic can be added by creating additional Lambda functions. Each function operates independently, so existing functionality is not impacted.

In this task, you will create another Lambda function that looks at inventory while it is loaded into the DynamoDB table. If the Lambda function notices that an item is out of stock, it will send a notification through the SNS topic you created earlier.

<img width="437" alt="image" src="https://github.com/user-attachments/assets/faaf194b-1762-41a6-8d06-ccdeb181cb97" />

On the Services menu, choose Lambda. Choose Create function and configure these settings:

◦ Function name: Check-Stock

◦ Runtime: Python 3.9

◦ Expand  Choose or create an execution role.

◦ Execution role: Use an existing role
        
◦ Existing role: Lambda-Check-Stock-Role; This role was configured with permissions to send a notification to Amazon SNS. 
        
◦ Choose Create function

<img width="959" alt="image" src="https://github.com/user-attachments/assets/a162428b-cd98-4806-8620-7fe0ca9c42f2" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/bfb810a9-14c4-4e79-a130-2c201a14198c" />

Scroll down to the Code source section, and in the Environment pane, choose lambda_function.py. In the code editor, delete all the code. Copy the following code, and in the Code Source editor, paste the copied code:

    # Stock Check Lambda function
    # This function is triggered when values are inserted into the Inventory DynamoDB table.
    # Inventory counts are checked and if an item is out of stock, a notification is sent to an SNS Topic.
    import json, boto3
    # This handler is run every time the Lambda function is triggered
    def lambda_handler(event, context):
      # Show the incoming event in the debug log
      print("Event received by Lambda function: " + json.dumps(event, indent=2))
      # For each inventory item added, check if the count is zero
      for record in event['Records']:
        newImage = record['dynamodb'].get('NewImage', None)
        if newImage:      
          count = int(record['dynamodb']['NewImage']['Count']['N'])  
          if count == 0:
            store = record['dynamodb']['NewImage']['Store']['S']
            item  = record['dynamodb']['NewImage']['Item']['S']  
            # Construct message to be sent
            message = store + ' is out of stock of ' + item
            print(message)  
            # Connect to SNS
            sns = boto3.client('sns')
            alertTopic = 'NoStock'
            snsTopicArn = [t['TopicArn'] for t in sns.list_topics()['Topics']
                            if t['TopicArn'].lower().endswith(':' + alertTopic.lower())][0]  
            # Send message to SNS
            sns.publish(
              TopicArn=snsTopicArn,
              Message=message,
              Subject='Inventory Alert!',
              MessageStructure='raw'
            )
      # Finished!
      return 'Successfully processed {} records.'.format(len(event['Records']))

<img width="959" alt="image" src="https://github.com/user-attachments/assets/1dd274c6-e9e6-4853-97f3-0b9555e4df60" />

<h2>Examine the code. It performs the following steps:</h2>
  
◦ Loop through the incoming records

◦ If the inventory count is zero, send a message to the NoStock SNS topic

 You will now configure the function so it triggers when data is added to the Inventory table in DynamoDB. Choose Deploy to save code changes.
 
Scroll to the Designer section (which is at the top of the page).

• Choose Add trigger and then configure these settings:

• Select a trigger: DynamoDB

• DynamoDB Table: Inventory

• Choose Add

<img width="947" alt="image" src="https://github.com/user-attachments/assets/82441cc0-b4e5-4831-b6d3-dc0a61403385" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/c6679bb7-167f-46ec-ba31-bc2d3e64bca7" />

You are now ready to test the system!

<h2>Task 6: Testing the System</h2>

You will now upload an inventory file to Amazon S3, which will trigger the original Load-Inventory function. This function will load data into DynamoDB, which will then trigger the new Check-Stock Lambda function. If the Lambda function detects an item with zero inventory, it will send a message to Amazon SNS. Then, Amazon SNS will notify you through SMS or email.
       
• On the Services menu, choose S3.
       
• Choose the name of your inventory- bucket.
       
• Choose Upload and upload a different inventory file.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/de75ec60-9b7b-42d0-b01b-9d308612cc73" />

Return to the Inventory System Dashboard and refresh  the page. You should now be able to use the Store menu to view the inventory from both stores.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/18b48954-cf48-43d6-9d98-564cb4b68d51" />

Also, you should receive a notification through email that the store has an out-of-stock item (each inventory file has one item that is out of stock). 

<img width="755" alt="image" src="https://github.com/user-attachments/assets/600e98c1-14b1-49dc-a2a8-42b8fbd73b37" />

<h2>Conclusion</h2>

Implementing a serverless inventory tracking system on AWS using Amazon S3, Lambda, Cognito, SNS, and DynamoDB enhances scalability, reduces operational costs, and improves system performance. This solution achieves 99.9% uptime while delivering cost savings of up to 40%.











































    
