<h1>How to Automate Your Code Deployments Using AWS CI/CD Tools</h1>

<h3>Business Scenario</h3>
Now that the café website is in production, Frank wants a reliable process in place to track code changes and update the site when changes are made. He asked Sofía to find a way to centralize the website code and add version control. He has also asked if it's possible to automatically update the website instead of manually running scripts and uploading files when changes are made.
Mateo, a café regular and AWS consultant who specializes in automating repeatable processes, was chatting with Sofía about his work. He mentioned using CodeCommit to collaborate on projects with other developers. Sofía shared that she was researching CodeCommit to centralize the café website's code and asked Mateo for suggestions about automating updates to the site. Mateo suggested using CodePipeline because it easily integrates with both CodeCommit and Amazon Simple Storage Service (Amazon S3).
When I start this lab, many services are already deployed for me. In this lab, I focus on the services that are represented in the following diagram. As in past labs, the VS Code IDE is used as the development environment. The developer runs commands from VS Code IDE to update the code in the S3 bucket that hosts the website. The second bucket in the diagram will be used to set up a continuous integration and continuous delivery (CI/CD) pipeline.

<img width="445" alt="image" src="https://github.com/user-attachments/assets/1fea581c-4065-4fd1-a420-e720df7085ed" />

By the end of this lab, I have created the architecture in the following diagram including a CodeCommit repository to store the website code. A CodePipeline pipeline to automate updates to the website when changes are pushed to the CodeCommit repository.
Artifacts that CodePipeline uses to deploy and update the website application will be hosted on an S3 bucket that is separate from the bucket that hosts the café website.

<img width="434" alt="image" src="https://github.com/user-attachments/assets/00160c32-5a55-4294-8112-7b2c9cd04199" />

<h2>Lab overview and objectives</h2>
In this lab, I will create an AWS CodeCommit repository and an AWS CodePipeline pipeline. I will configure the pipeline to automatically apply updates to the café business website as changes are saved to the repository.

What I Learned:

• Create a new CodeCommit repository
      
• Clone and update a CodeCommit repository
      
• Create a pipeline by using CodePipeline

<h2>Task 1: Preparing the development environment</h2>
Before I can start working on this lab, I must import some files and install some packages in the Visual Studio Code Integrated Development Environment (VS Code IDE) that was prepared for me. 

Connect to the VS Code IDE using the provided login details
▪ LabIDEURL
              
▪ LabIDEPassword

▪ In a new browser tab, paste the value for LabIDEURL to open the VS Code IDE.
              
▪ On the prompt window Welcome to code-server, enter the value for LabIDEPassword . Choose
              
▪  Submit to open the VS Code IDE.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/7d2a2fb0-049a-407f-8467-59e801cdc8f3" />

 I verify that the AWS CloudFormation stack creation process for the lab has successfully completed.
 
◦ In a new browser tab, navigate to the CloudFormation console.

• In the left navigation pane, choose Stacks.

• For all three stacks, verify that the Status says CREATE_COMPLETE. 

⚠️ If any stack doesn't yet have that status, wait until it does. It might take about 12 minutes to complete from the time that you chose Start Lab.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/cf2ef1c6-7797-4027-93ae-7d3314e3c261" />

Download and extract the files. In the VS Code IDE bash terminal (at the bottom of the IDE), run the following command:

    wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-44869/13-lab-ci-cd/code.zip -P /home/ec2-user/environment
      
<img width="959" alt="image" src="https://github.com/user-attachments/assets/0ecb6e67-4da1-47ca-af5c-dd5a4a1ca22f" />

Notice that a code.zip file was downloaded to the VS Code IDE. The file is listed in the Environment window. Extract the file:

    unzip code.zip

<img width="593" alt="image" src="https://github.com/user-attachments/assets/d985ec22-67d4-4cf4-809f-dada76eacc50" />

Run a script that upgrades the version of the AWS CLI installed on the VS Code IDE. The script also re-creates the work that I completed in earlier labs into this AWS account.

Note: This script deploys the architecture that I created across the previous labs. This includes populating and configuring the S3 bucket that the café website uses. The script creates the Amazon DynamoDB table and populates it with data. The script also re-creates the REST API that I created in the Amazon API Gateway lab and deploys the containerized coffee suppliers AWS Elastic Beanstalk application.

Set permissions on the script so that you can run it, and then run it: 

    chmod +x ./resources/setup.sh && ./resources/setup.sh

Note:

When prompted for an IP address, enter the IPv4 address that the internet uses to contact your computer. You can find your IPv4 address at https://whatismyipaddress.com.

The IPv4 address that you set is the one that will be used in the bucket policy. Only requests that originate from this IPv4 address will be allowed to load the website pages. Do not set it to 0.0.0.0 because the S3 bucket's block public access settings will prevent access.
When prompted for an email address, enter one that you have access to as you complete the lab.

Verify the version of AWS CLI installed. In the VS Code IDE Bash terminal, run the following command:

    aws –version

The output should indicate that version 2 is installed.
          
 Verify that the SDK for Python is installed. Run the following command:
 
     pip3 show boto3

<img width="779" alt="image" src="https://github.com/user-attachments/assets/d43abbca-5e93-4e62-a25c-3358803a3ba4" />

<h2>Task 2: Creating a CodeCommit repository</h2>

When a developer manages code in their local working environment, it is difficult to track and manage changes. This approach also limits the ability for multiple developers to collaborate on the same codebase. 
AWS CodeCommit is a secure, highly scalable, managed source control service that hosts private Git repositories. CodeCommit makes it easy for teams to securely collaborate on code with contributions encrypted in transit and at rest. 
In this task, I will create a CodeCommit repository to host the café website code. 
Create a CodeCommit repository to host the codebase.

 • In the search bar at the top, search and select CodeCommit to open the AWS CodeCommit service console. 

 • From the navigation pane, choose Create repository.

Configure the following settings:

◦ Repository name: Enter front_end_website

◦ Description: Enter Repository for the cafe website front end

◦ Keep the rest of the default settings.

• Choose Create.

<img width="955" alt="image" src="https://github.com/user-attachments/assets/02e81ada-cafc-40c5-96b0-4730c6aed7a2" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/3d98c7b7-1b53-4a4c-86d5-b63505f1adda" />

Create a test file.
• In the front_end_website section, choose Create file.

• Copy and paste the following code into the front_end_website text box:
    
    <!DOCTYPE html>
    <html>
        <head>
        <title>Test page</title>
        </head>
        <body>
            <h1>
               This is a sample HTML page.
            </h1>
        </body>
    </html>

In the Commit changes to main section, configure the following settings:
       
◦ File name: Enter test.html
       
◦ Author name: Enter your name.

◦ Email address: Enter the same email address that you used in Task 1.

◦ Commit message: Enter This is my first commit. 
