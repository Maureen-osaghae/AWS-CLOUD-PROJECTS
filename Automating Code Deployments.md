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

<img width="764" alt="image" src="https://github.com/user-attachments/assets/9900e225-05f4-45a8-bff4-98f1d9cd9fa1" />

Choose Commit changes.

<img width="958" alt="image" src="https://github.com/user-attachments/assets/b1788417-cc3f-4b8c-9496-3e8ef92bcce2" />

◦ Review commit.

◦ In the left navigation pane, under Repositories, choose Commits.

◦ Choose the link for the commit ID. Only one should be listed.

◦ Review the information about this commit.

 Note: On this page, you see the author of the commit, the commit message, the date of the commit, and the name and contents of the file that was added. 

<img width="768" alt="image" src="https://github.com/user-attachments/assets/6ad9eaeb-ead8-4b9e-9319-d4fe9f1d1659" />

Add a comment to the committed file.

• In the test.html section, hover on the plus sign (+) icon on line 4.

• To the left of the line number, a comment icon appears. Choose the icon.
      
Note: The comment icon is a small box that contains an ellipsis.
    
• In the New comment text box, enter: I must add a better title at some point.
      
The following image shows the comment text box.

<img width="715" alt="image" src="https://github.com/user-attachments/assets/2f2bcb90-9fa2-463b-8d54-9d21300e828f" />

<img width="584" alt="image" src="https://github.com/user-attachments/assets/53457dcc-119e-435c-ada9-0847006f89b4" />

Now that I have created a CodeCommit repository to host and manage code changes, I can use it as a source to automate publishing updates to the café website. 

<h2>Task 3: Creating a pipeline to automate website updates</h2>
So far, you have been running commands from VS Code IDE to update the code in the S3 bucket for the website. In this task, you will configure a CI/CD pipeline by using CodePipeline to automate the website updates. The pipeline will use the code in your CodeCommit repository to deploy changes to the website's S3 bucket.
       
Return to the VS Code IDE.

In the Explorer section, expand Environment, which is located in the upper-left corner. 

Expand the resources folder, and open the file named cafe_website_front_end_pipeline.json

<img width="845" alt="image" src="https://github.com/user-attachments/assets/3ea930d5-ce69-4442-8885-6bd1c53271dd" />

This file defines configuration that will be used to deploy my new pipeline. Review the following code snippets to understand how the pipeline is configured.

The following lines declare the AWS Identity and Access Management (IAM) role that will be associated with the pipeline.

      "pipeline": {
       "roleArn": "arn:aws:iam::<FMI_1>:role/RoleForCodepipeline",

The following code snippet defines the Source that my pipeline will use to create and update my application. In this case, the source is my CodeCommit repository, front_end_website. Note that the pipeline will be configured to use the main branch.
      
      {
         "name": "Source",
         "actions": [
          {
            "inputArtifacts": [],
            "name": "Source",
            "actionTypeId": {
              "category": "Source",
              "owner": "AWS",
              "version": "1",
              "provider": "CodeCommit"
            },
            "outputArtifacts": [
              {
                "name": "MyApp"
              }
            ],
            "configuration": {
              "RepositoryName": "front_end_website",
              "BranchName": "main"
            },
            "runOrder": 1
          }
        ]
      }

The following section of code defines the Deploy stage. The deployment will update code in Amazon S3. The configuration settings define details about the deployment target. In this case, this section defines the S3 bucket name, configures files to be extracted from a .zip file, and sets a caching policy. 
            
            {
                "name": "Deploy",
                
                "actions": [
                    {
                        "inputArtifacts": [
                            {
                                "name": "MyApp"
                            }
                        ],
                        "name": "CafeWebsite",
                        "actionTypeId": {
                            "category": "Deploy",
                            "owner": "AWS",
                            "version": "1",
                            "provider": "S3"
                    },
                        "outputArtifacts": [],
                        "configuration": {
                            "BucketName": "<FMI_2>",
                            "Extract": "true",
                            "CacheControl": "max-age=14"
                        },
                        "runOrder": 1
                    }
                ]
            }

The final section of the code defines the artifactStore. This is the S3 bucket where CodePipeline artifacts will be stored. 
     
      "artifactStore": {
              "type": "S3",
              "location": "codepipeline-us-east-1-<FMI_1>-website"
          },
          "name": "cafe_website_front_end_pipeline",
          "version": 1
      }

<h4>Update the cafe_website_front_end_pipeline.json file:</h4>

Replace the two <FMI_1> placeholders with my AWS account ID.

Note: To find your account ID, run the following command: aws  sts get-caller-identity

Replace the <FMI_2> placeholder with the name of my bucket 

Note: To retrieve a list of the S3 buckets in your account, run the following command: aws  s3 ls
and  save

 To create the pipeline, run the following commands. 

       cd ~/environment/resources
      aws codepipeline create-pipeline --cli-input-json file://cafe_website_front_end_pipeline.json

<img width="862" alt="image" src="https://github.com/user-attachments/assets/21170fbb-3442-4266-9478-cc5b485e4c19" />

Navigate to the CodePipeline console.
      
The Pipelines section lists the cafe_website_front_end_pipeline Pipeline.
        
Choose the cafe_website_front_end_pipeline hyperlink and review the pipeline status, as shown in the following image.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/cbf9a631-3118-4bfb-bd5d-6545caef8a14" />

<img width="928" alt="image" src="https://github.com/user-attachments/assets/69869bcb-893d-44f3-b0aa-65ce74329ed9" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/000b67c4-82a6-42f5-9fb4-08cf9371b763" />

The pipeline should deploy successfully. Code was deployed using CodeCommit as the source and the café website S3 bucket as the target. This means the bucket should have been updated with the test.html file. 

Verify the automated deployment.

• Return to the VS Code IDE bash terminal.
    
• To find your Amazon CloudFront distribution domain name, run the following command:

      aws cloudfront list-distributions --query DistributionList.Items[0].DomainName --output text

Update the following URL by replacing <cloudfront_domain> with the value that was returned by the previous command: https://<cloudfront_domain>/test.html
      
The updated URL is similar to https://aaabbb111222.cloudfront.net/test.html.

• Open a new browser tab, and enter the URL that you just created. 

You reach a sample webpage similar to the following:

<img width="619" alt="image" src="https://github.com/user-attachments/assets/820c4668-ec59-4179-b347-2eca47a62e9d" />

Now, when I update the repository, the pipeline will automatically update my website.
Next, I will clone the repository to my VS Code IDE. This will provide the ability to edit the files locally and synchronize with my centralized CodeCommit repository.

<h2>Task 4: Cloning a repository in VS Code IDE</h2>

It's more efficient to edit code in an IDE than it is to use the CodeCommit console. In this task, I will clone a local copy of the repository in my VS Code IDE work environment.
Retrieve the SSH clone URL for my repository.

◦ Navigate to the CodeCommit console.

◦ In the navigation pane, choose Repositories.

◦ In the Clone URL column, choose Clone HTTPS(GRC) to copy the URL to my clipboard.

 Note: URL should look similar to codecommit::us-east-1://front_end_website

Clone your repository using the SSH URL.
    
• Return to the VS Code IDE terminal.
   
• Run the following command to clone the repository to my VS code IDE, replace the <<Clone URL>> with the value I copied.

      cd ..
      git clone <<Clone URL>>

<img width="181" alt="image" src="https://github.com/user-attachments/assets/47eccae2-1342-430f-9940-b28322aa0da0" />

<img width="856" alt="image" src="https://github.com/user-attachments/assets/7263e30b-28a1-47af-9d14-9893542c7302" />

I now have a local clone of my repository that can be edited using my IDE. Next, I will lean how to manage my local copy of the repository and synchronize changes with CodeCommit.

<h2>Task 5: Exploring the Git integration with the VS Code IDE</h2>

I can interact with the repository through the command line, but VS Code provides Git integration in the IDE to make it easier to manage my repository. In this task, I will perform simple repository management operations by using this integration. 

◦ Locate the branch icon, which is located in the lower-left corner of the IDE.

▪ Source Control option is opened.

◦ From the top, choose three dots to the right of SOURCE CONTROL.

◦ From the menu displayed, choose Source Control Repositories

◦ Your repository front_end_website is displayed.
 The IDE 
 is currently set up to communicate with the main branch.

Explore and edit the repository files.

Remember that the repository was cloned to the local folder environment/front_end_website. I can open repository files in my IDE to view and edit them. From the explorer menu, expand the front_end_website folder to reveal the test.html file, which is shown in the following image.

<img width="806" alt="image" src="https://github.com/user-attachments/assets/edbd33b9-3740-46f8-a987-866357dc80f4" />

Open the test.html file and edit the page title on line 4. Replace the current title with the following text: Best test page ever

Save changes.

Commit changes to the main branch

◦ Open the Source Control.

 Note: You don't have to use the IDE integration. You could also use Git from the command line. For example, if you enter the following command, you should find the test.html file listed in the output.

      cd ~/environment/front_end_website
      git status

• On the Commit button, choose more actions which is located to the right.

• Choose Commit & Push, then choose Yes. 
    
• This will push the code and trigger the code pipeline which deploys new version of the file to website.

Review the changes in CodeCommit.

• Navigate to the CodeCommit console.

• Choose the front_end_website repository link.

• In the navigation pane, under Repositories, choose Commits.

• Choose the link for the commit ID with the most recent commit date.

• Go to the test.html section, and notice the highlighted lines, which are shown in the following image.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/29667415-cc51-420f-93d5-1abd9db31475" />

The first highlighted line is preceded by a minus sign (-) and highlighted in red. This shows the contents of the line before the change was made. The second highlighted line is preceded by a plus sign (+) and highlighted in green. This line shows the current content of the line.
Great work! I have confirmed that VS Code IDE is able to push changes to your CodeCommit repository.
       
Now that have have learned to manage your local repository and synchronize it with CodeCommit, it's time to update the repository with the actual café website code.

<h2>Task 6: Pushing the café website code to CodeCommit </h2>

In this task, I will first remove the test.html file. Then, I will update the local repository with the café website code. Finally, verify that my pipeline built the café website on S3. I will also verify that max-age is set to 14 to confirm that the latest changes are being applied and the caching was updated.








    
