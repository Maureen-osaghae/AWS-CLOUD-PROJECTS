<h1>Migrate a Web Application to Docker Containers</h1>
<h2>Lab overview and objectives</h2>

In this lab, you will migrate a web application to run on Docker containers. The application is installed directly on the guest operating systems (OSs) of two Amazon Elastic Compute Cloud (Amazon EC2) instances. You will migrate the application to run on Docker containers.

After completing this lab, you should be able to:

<ol>
<li>Create a Dockerfile.</li>
<li>Create a Docker image by using a Dockerfile.</li>
<li>Run a container from a Docker image.</li>
<li>Interact with and administer your containers.</li>
<li>Create an Amazon Elastic Container Registry (Amazon ECR) repository.</li>
<li>Authenticate the Docker client to Amazon ECR.</li>
<li>Push a Docker image to Amazon ECR.</li>
</ol>

</h2>Business Scenario</h2>

The café owners have noticed how popular their gourmet coffee offerings have become. Customers cannot seem to get enough of their cappuccinos and lattes. Meanwhile, the café owners have been challenged to consistently source the highest quality coffee beans. Recently, the owners learned that one of their favourite coffee suppliers wants to sell her company. Frank and Martha jumped at the opportunity to buy the company.
The acquired coffee supplier runs an inventory tracking application on an AWS account. Sofía has been tasked to understand how the application works and then create a plan to integrate the application into the café's existing application infrastructure.
In this lab, I will again play the role of Sofía, and work to migrate the application to run on containers.

The following diagram shows the architecture that was created for me in AWS at the beginning of the lab: 

<img width="416" alt="image" src="https://github.com/user-attachments/assets/44cbcaec-2221-4877-8102-159b780cdb7e" />

By the end of this lab, I have migrated the application and the backend database to run as Docker containers, as shown in the following diagram: 

<img width="421" alt="image" src="https://github.com/user-attachments/assets/765ceecd-d8f1-40db-8711-7f346bf7b3db" />

I will register these two containers in Amazon ECR to make them available to deploy as needed.

Let's get started!

<h2>Preparing the development environment</h2>

<h3>Connect to the VS Code IDE.</h3>

 Copy the LabIDEURL and LabIDE Password
In a new browser tab, paste the value for LabIDEURL to open the VS Code IDE.

On the prompt window Welcome to code-server, enter the value for LabIDEPassword you copied to the editor earlier, choose Submit to open the VS Code IDE.

<img width="785" alt="image" src="https://github.com/user-attachments/assets/64e1e463-6df0-4f80-b1f1-79393f064b09" />

Download and extract the files that you will need for this lab. In the same terminal, run the following command:

    wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-44869/06-lab-containers/code.zip -P /home/ec2-user/environment

The code.zip file is downloaded to the VS Code IDE. The file is listed in the left navigation pane.

Extract the file:
    
    unzip code.zip

<img width="461" alt="image" src="https://github.com/user-attachments/assets/a4acf207-4caa-45eb-b050-1dd339c88c00" />


Run a script that will upgrade the version of Python and the AWS CLI that are installed on the VS Code IDE.

Set permissions on the script so that you can run it, then run it:

    chmod +x ./resources/setup.sh && ./resources/setup.sh

Verify that the script completed without error. Verify the version of AWS CLI installed.
      In the VS Code IDE Bash terminal (at the bottom of the IDE), run the following command:

aws –version
The output should indicate that version 2 is installed. Verify that the SDK for Python is installed. Run the following command:
          
          pip3 show boto3

<h2>Task 2: Analysing the existing application infrastructure</h2>

In this task, you will analyze the current application infrastructure. Open the existing coffee supplier application in a browser tab. Return to the browser tab labeled Your environments, and navigate to the EC2 console. Choose Instances.
Notice that three instances are running.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/430351f3-e843-4cbf-ac04-c85bad8b3969" />

<ol>
<li>One instance is the VS Code IDE that you used in the previous task.</li>
<li>The other two instances (MysqlServerNode and AppServerNode) support the application that you will containerize in this lab.</li>
<li>Choose the AppServerNode instance, and copy the Public IPv4 address value.</li>
<li>Open a new browser tab and navigate to the IP address.</li>
</ol>
 The coffee suppliers website displays.

 <img width="821" alt="image" src="https://github.com/user-attachments/assets/c8028790-ffef-4654-9f53-b206714db299" />

 Note: The page uses http:// instead of https://. Your browser might indicate that the site is not secure, because it does not have a valid SSL/TLS certificate. You can ignore the warning in this development environment. 

<ol>
  <li>Choose List of suppliers and then choose Add a new supplier.</li>
  <li>Fill in all of the fields with values. For example:</li>
  <li>Name: Nikki Wolf</li>
  <li>Address: 100 Main Street</li>
  <li>City: Anytown</li>
  <li>State: CA</li>
  <li>Email: nwolf@example.com </li>
  <li>Phone: 4155551212</li>
  <li>Choose Submit.</li>
</ol>

The All suppliers page displays and includes the record that you submitted.

<img width="947" alt="image" src="https://github.com/user-attachments/assets/6bf06088-54ca-4435-9329-0f0fa2a5320b" />

 •  Choose edit and change the record (for example, modify the phone number).
      
 •  To save the change, choose Submit.
      
 •  Notice that the change was saved in the record.

Analyze the web application code.

A copy of the application code that is installed on the AppServerNode EC2 instance is also available in your VS Code IDE. Return to the VS Code IDE browser tab.
In the file browser in the left navigation pane, expand the resources directory, and then expand the codebase_partner directory to see the application code.

<h3>Optional:</h3>
If you are interested to know how the code is configured on the AppServerNode, you can connect to the instance and view the code there as well. 

To do this, in the terminal next to these instructions, run the following commands, to connect to the EC2 instance and see the files that are installed (replace <public-ip-address> with the actual IPv4 address of the AppServerNode instance).

    ssh -i ~/.ssh/labsuser.pem ubuntu@<public-ip-address>
    cd resources/codebase_partner
    ls -l

For this lab, it is not necessary for you to understand the details of how the application was built. However, the following details might be of interest to you:
        
◦  The application was built with Express, which is a framework for building web applications.
          
◦ The application runs on port 80 and is coded in node.js.
          
◦ To install the application directly on the guest OS of the AppServerNode Ubuntu Linux EC2 instance, node.js and the node package manager (npm) were installed. 

Then, the code that you can see in resources/codebase_partner was placed on the server, and the connection to the application's database was configured.

<h2>Task 3: Migrating the application to a Docker container</h2>

In this task, you will migrate an application that is installed directly on the guest OS of an Ubuntu Linux EC2 instance to instead run in a Docker container. The Docker container is portable and could run on any OS that has the Docker engine installed.
For convenience, you will run the container on the same EC2 instance that hosts the VS Code IDE that you are using. You will use this IDE to build the Docker image and launch the Docker container.

The following diagram shows the migration that you will accomplish in this task:

<img width="404" alt="image" src="https://github.com/user-attachments/assets/39af85bd-8876-4ffc-80dc-f9573973ff8e" />

Create a working directory for the node application, and move the source code into the new directory.

In the VS Code IDE, to create and navigate to a directory to store your Docker container code, run the following commands:

    mkdir containers
    cd containers

To create and navigate to a directory named node_app inside of the containers directory, run the following commands:
     
      mkdir node_app
      cd node_app
To move the code base, which you copied earlier, into the new node_app directory, run the following command:
     
      mv ~/environment/resources/codebase_partner 
      ~/environment/containers/node_app

<h3>Create a Dockerfile.</h3>
<span>Note:</span>
A Dockerfile is where you provide instructions to Docker to build an image. A Docker image is a template that has instructions to create a container.
To create a new Dockerfile named Dockerfile in the node_app/codebase_partner directory, run the following command:

    cd ~/environment/containers/node_app/codebase_partner
    touch Dockerfile

In the left navigation pane, browse to and open the empty Dockerfile that you just created.

Copy and paste the following code into the Dockerfile: 

    FROM node:11-alpine
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app
    COPY . .
    RUN npm install
    EXPOSE 3000
    CMD ["npm", "run", "start"]

<img width="610" alt="image" src="https://github.com/user-attachments/assets/b0985c66-042f-4bed-8121-76e85c3d0c80" />

Save the changes.
<h3>Analysis:</h3>
This Dockerfile code specifies that an Alpine Linux distribution with node.js runtime requirements should be used to create the image. The code also specifies that the container should allow network traffic on TCP port 3000. Finally, the code specifies that the application should be run and started when the container launches.

<h3>Build an image from the Dockerfile.</h3>
In the VS Code IDE Bash terminal, run the following command:

    docker build --tag node_app .

The output is similar to the following: 
    
    [+] Building 9.4s (10/10) FINISHED                                                                         docker:default
     => [internal] load build definition from Dockerfile                                                                 0.0s
     => => transferring dockerfile: 225B                                                                                 0.0s
     => [internal] load metadata for docker.io/library/node:11-alpine                                                    0.3s
     => [internal] load .dockerignore                                                                                    0.0s
     => => transferring context: 2B                                                                                      0.0s
     => [1/5] FROM docker.io/library/node:11-alpine@sha256:8bb56bab197299c8ff820f1a55462890caf08f57ffe3b91f5fa6945a4d50  3.4s
     => => resolve docker.io/library/node:11-alpine@sha256:8bb56bab197299c8ff820f1a55462890caf08f57ffe3b91f5fa6945a4d50  0.0s
     => => extracting sha256:e7c96db7181be991f19a9fb6975cdbbd73c65f4a2681348e63a141a2192a5f10                            0.2s
     => => sha256:0119aca4464934fdf40121363b1cf0537e464c07c790df99058ba587b14aaadd 21.66MB / 21.66MB                     0.5s
     => => sha256:40df19605a18a2dc4e3c6c0cb5d0a93fd2f7615e2a92eccd743698c24c23fe84 1.33MB / 1.33MB                       0.1s
     => => sha256:82194b8b4a64bd286671a1464d3fe319832aaa67c4e41c455fdfdd295ae7aa73 279B / 279B                           0.2s
     => => sha256:8bb56bab197299c8ff820f1a55462890caf08f57ffe3b91f5fa6945a4d505932 1.42kB / 1.42kB                       0.0s
     => => sha256:914ff2c2145de019a19c080a9e42b5763c826194110ec8e02c8e92845799fba6 1.16kB / 1.16kB                       0.0s
     => => sha256:f18da2f58c3dabd7d06c02f4a76719ba0c786737aa83d4bfc8f1e5cb0800b96a 5.66kB / 5.66kB                       0.0s
     => => sha256:e7c96db7181be991f19a9fb6975cdbbd73c65f4a2681348e63a141a2192a5f10 2.76MB / 2.76MB                       0.1s
     => => extracting sha256:0119aca4464934fdf40121363b1cf0537e464c07c790df99058ba587b14aaadd                            2.6s
     => => extracting sha256:40df19605a18a2dc4e3c6c0cb5d0a93fd2f7615e2a92eccd743698c24c23fe84                            0.1s
     => => extracting sha256:82194b8b4a64bd286671a1464d3fe319832aaa67c4e41c455fdfdd295ae7aa73                            0.0s
     => [internal] load build context                                                                                    0.0s
     => => transferring context: 1.25MB                                                                                  0.0s
     => [2/5] RUN mkdir -p /usr/src/app                                                                                  0.8s
     => [3/5] WORKDIR /usr/src/app                                                                                       0.0s
     => [4/5] COPY . .                                                                                                   0.1s
     => [5/5] RUN npm install                                                                                            4.3s
     => exporting to image                                                                                               0.3s
     => => exporting layers                                                                                              0.3s
     => => writing image sha256:be21636b8618e57c99de2ffef205bb1567e9f326795d58df9dcc8682c14e4fe0                         0.0s
     => => naming to docker.io/library/node_app   
    

Verify that the Docker image was created. To list the Docker images that your Docker client is aware of, run the following command:
      
      docker images

<img width="715" alt="image" src="https://github.com/user-attachments/assets/ae8295f2-2741-484c-bd02-1c664d2c090a" />

<h3>Create and run a Docker container based on the Docker image.</h3>
      
To create and run a Docker container from the image, run the following command:
      
    docker run -d --name node_app_1 -p 3000:3000 node_app

<h3>Analysis:</h3>
This command launches a container with the name node_app_1, using the node_app image that you created as the template. The -d argument specifies that it should run in the background and print the container ID. The -p specifies to publish container port 3000 to the host (VS Code IDE) port 3000. The terminal returns the container ID. It is a long string of letters and numbers.

 To view the Docker containers that are currently running on the host, run the following command:
     
      docker container ls

  <img width="959" alt="image" src="https://github.com/user-attachments/assets/dd91fd43-d9ca-42a3-abfe-2985df7db2f7" />

Verify that the node application is now running in the container.

Verify that the node application is now running in the container. To check that the container is working on the correct port, run the following command:
      
      curl http://localhost:3000
      
 <img width="939" alt="image" src="https://github.com/user-attachments/assets/59440489-b35c-4e14-81f8-61b658fff0b6" />

 The webpage looks similar to the following:

     <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="/css/bootstrap.min.css">
        <link rel="stylesheet" href="/css/base.css">
        <title>Coffee suppliers</title>
    </head>
    <body>
    
    <div class="container">
        <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
            <img src="/img/espresso.jpg" width="200">
            <div><a class="navbar-brand page-title" href="/supplier">Coffee suppliers</a></div>
            <div class="collapse navbar-collapse" id="navbarSupportedContent">
                <ul class="navbar-nav mr-auto">
                    <li class="nav-item active">
                        <a class="nav-link" href="/">Home</a>
                        <a class="nav-link" href="/suppliers">Suppliers list</a>
                    </li>
                </ul>
            </div>
        </nav>    <div class="container">
            <h1>Welcome</h1>
            <p>Use this app to keep track of your coffee suppliers</p>
            <p><a href="/suppliers">List of suppliers</a></p>
        </div>
    </div>
    <script src="/js/jquery-3.6.0.min.js"></script>
    <script src="/js/bootstrap.min.js"></script>
    </body>

<h3>Analysis: </h3>
This demonstrates that the application is running and available on TCP port 3000. However, you want to interact with this as a website, and you don't yet know if the application is able to connect to the database. 

Adjust the security group of the VS Code IDE instance to allow network traffic on port 3000 from your computer.
       
Note: Because you are using the VS Code IDE instance to run the container, you must open TCP port 3000 for inbound traffic.
<ol>
<li>Return to the AWS Management Console browser tab, and navigate to the EC2 console.</li>
<li>Locate and select the Lab IDE instance.</li>
<li>Choose the Security tab, and choose the security group hyperlink.</li>
<li>Choose the Inbound rules tab, and choose Edit inbound rules.</li>
<li>Choose Add rule and configure the following:</li>
<li>Type: Custom TCP</li>
<li>Port range: 3000</li>
<li>Source: My IP</li>
<ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/c3fb4d45-9a39-4b96-94cc-9a7b328afb81" />

◦ Choose Save rules.

Access the web interface of the application, which is now running in a container.

◦ In the EC2 console, choose Instances and choose the Lab IDE instance.
 
◦ On the Details tab, copy the Public IPv4 address value.

◦ Open a new browser tab. Paste the IP address into the address bar, and add :3000 at the end of the address.
 
The web application loads in the browser. You have seen this page before; however, you previously accessed the application that was running directly on the AppServerNode EC2 instance guest OS. This time you accessed the application running in a container on the VS Code IDE instance's Docker hypervisor.

<img width="803" alt="image" src="https://github.com/user-attachments/assets/6c7605b1-5bd4-4174-ab22-e921c5d83c4c" />

<h3>Summary of how you have used Docker so far</h3>

You just completed a series of steps with Docker. The following diagram summarizes what you have accomplished with Docker so far.

<img width="389" alt="image" src="https://github.com/user-attachments/assets/40b23b2c-4bb0-4ccd-8590-6ad2f3d2f94e" />

You copied the code base into a directory, which acted as your build area [a]. You also created a Dockerfile that provided instructions for how to create a Docker image. That Dockerfile specified a FROM instruction that identified a starter image to use. You then ran the docker build command [b]. Docker read the Dockerfile and requested the starter image from an image repository [c]. The image repository returned the starter image file [d]. The docker build command then finished building the image according to the instructions in the Dockerfile, which resulted in the Docker image [e]. Finally, you ran the docker run command [f] to run a Docker container [g].

<h3>Analyze the database connection issue.</h3>
In the coffee suppliers application, choose List of suppliers. You see an error stating that there was a problem retrieving the list of suppliers.

<img width="819" alt="image" src="https://github.com/user-attachments/assets/b63d63ab-2b63-4528-8d66-4b11ea34503f" />

 <h3>Analysis:</h3>
 This is because the node_app_1 container is having trouble reaching the MySQL database, which is running on the EC2 instance named MysqlServerNode. Return to the VS Code IDE browser tab.
 Open the config.js file in the containers/node_app/codebase_partner/app/config/ directory. The top of the file contains the following code:

 <img width="937" alt="image" src="https://github.com/user-attachments/assets/2748c6d1-1082-4c76-9536-53dfd14346e1" />

 <h3>Analysis:<h3>
The application code checks for an environmental variable to learn how to connect to the MySQL database. As you see in the code, the settings include a hardcoded IP address for the location of the MySQL database host. However, the application code also contains the following logic:

      Object.keys(config).forEach(key => {
      if(process.env[key] === undefined){
        console.log(`[NOTICE] Value for key '${key}' not found in ENV, using default value.  See app/config/config.js`)
      } else {
        config[key] = process.env[key]
      }
    });
    
    module.exports = config; 

This checks for the existence of an environmental variable. If one is found, that value overrides the placeholder (hardcoded) APP_DB_HOST address. When you visited the web application earlier—the version that was directly installed on the EC2 instance guest OS—the node instance passed an environment variable (the IPv4 address of the MysqlServerNode EC2 instance) to the application. However, you did not launch your node_app_1 container with that environment variable. Therefore, the node application defaulted to the hardcoded 3.82.161.206 IP address, which does not match the IP address of the MySQL instance.

Establish a terminal connection to the container to observe the settings. To find the container ID, run the following command:
          
          docker ps

<img width="741" alt="image" src="https://github.com/user-attachments/assets/64ed2eea-3d8a-42a4-84b6-284c898fad95" />

To connect your terminal to the container, run the following commands, one at a time. Replace <container-id> with the actual container ID value that you just retrieved: 

    docker exec -ti <container-id> sh
    whoami
    
<img width="782" alt="image" src="https://github.com/user-attachments/assets/78dd1cb7-d86d-48e9-a08e-e0f20bc61a60" />

Your terminal is now connected to the container as the root user.

To observe the environment variables that are present in the node user's environment, run the following commands:
      
      su node
      env
 Notice that the APP_DB_HOST variable is not present.

 <img width="820" alt="image" src="https://github.com/user-attachments/assets/5c7a8448-e6d5-4e7d-a728-e85d886275ae" />

To disconnect from the container, run the following commands:
      
      exit
      exit

The first exit command makes you the root user again. The second command disconnects you from the container. 

Stop and remove the container that has the database connectivity issue. To get the ID of the running container, run the following command:

    docker ps
<img width="717" alt="image" src="https://github.com/user-attachments/assets/2bff32f0-b9f5-4bd1-8121-e0995ffa5a0d" />

Notice the name of the application that is returned in the NAMES column. To stop and remove the container, run the following command:
       
       docker stop node_app_1 && docker rm node_app_1
      
To verify that the application is no longer running, run the following curl command:
      
      curl http://localhost:3000

The output indicates a failure to connect to the application (Connection refused).

 Tip: If you refresh the web application in the browser (the version that is running as a container on the VS Code IDE), you will find that the application no longer loads.

 <h3>Launch a new container. </h3>

This time, you will pass an environment variable to tell the node application the correct location of the database.
   
• Return to the EC2 console, and copy the Public IPv4 address value of the MysqlServerNode EC2 instance.
      
• Return to the VS Code IDE Bash terminal.
      
• To run the application in a container and pass an environment variable to specify the database location, run the following command. Replace <ip-address> with the actual public IPv4 address of the MysqlServerNode EC2 instance

    docker run -d --name node_app_1 -p 3000:3000 -e APP_DB_HOST="3.94.253.183 " node_app
    
 <img width="959" alt="image" src="https://github.com/user-attachments/assets/b3cf35ac-0fa7-42d3-ab32-ed3344a887de" />

 When you pass in the network location of the database as an environment variable, you give the node application the information that it needs to establish network connectivity to the database, as illustrated in the following diagram: 

 <img width="383" alt="image" src="https://github.com/user-attachments/assets/15121aff-05a1-4936-93b9-b42eebaa66b8" />

 Optional: To check the environment variables of the new container, run the docker exec command, which you used previously. The container now has an APP_DB_HOST variable.

 <img width="806" alt="image" src="https://github.com/user-attachments/assets/0e8a6543-c38d-42ef-aa54-92d96f5f4b3c" />

 Verify that the database connection is now working. Try to access the web application again.
           
▪ If you still have the page open, refresh the browser tab. Otherwise, to navigate to the application in a new browser tab, go to http://<LabIDE-public-ip>:3000 (replace <LabIDE-public-ip> with the actual public IPv4 address of your VS Code IDE).
   
◦ The application is working. Choose List of suppliers to go to the http://<LabIDE-public-ip>:3000/suppliers page.
        
◦ The page displays the supplier entry that you created earlier. This indicates that your container is connecting to the MysqlServerNode EC2 instance where that data is stored.
          
Congratulations! You have successfully migrated the node application to a container. Also, the application container (node-app_1) is now able to successfully establish a network connection with the MySQL database, which is still running on an EC2 instance.

<h2>Task 4: Migrating the MySQL database to a Docker container</h2>

In this task, you will work to migrate the MySQL database to a container as well. To accomplish this task, you will dump the latest data that is stored in the database and use that to seed a new MySQL database running in a new Docker container. The following diagram shows the migration that you will accomplish in this task:

<img width="417" alt="image" src="https://github.com/user-attachments/assets/402a1a5f-3dda-4997-903d-6e85686b74f6" />

Create a mysqldump file from the data that is currently in the MySQL database.
    
• Return to the VS Code IDE, and close any file tabs that are open in the text editor.
      
• Choose File > New File and then paste the following code into the new file:

    mysqldump -P 3306 -h  <mysql-host-ip-address> -u nodeapp -p --databases COFFEE > ../../my_sql.sql

Next, go to the EC2 console and copy the Public IPv4 address value of the MysqlServerNode instance.

• Return to the text file in VS Code IDE and replace <mysql-host-ip-address> in the code with the IP address that you copied.
    
• In the terminal, to ensure that you are in the correct directory, run the following command:
      
      cd /home/ec2-user/environment/containers/node_app/codebase_partner

Finally, copy the command that you created in the text editor into the terminal and run the command. Your command will look similar to the following example, but your IP address will be different:

      mysqldump -P 3306 -h 100.27.45.2 -u nodeapp -p --databases COFFEE > ../../my_sql.s

• The mysqldump utility prompts you for a password. Enter the following password: coffee 

If successful, the terminal does not show any output. However, in the left navigation panel, notice that the  file now appears in the containers directory.

Tip: To see the new file, you might need to choose the settings icon in the upper-right corner of the file tree panel and then choose Refresh File Tree.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/05f03333-bb73-4e03-af65-6879702d9009" />

Open the mysqldump file and observe the contents.

◦ Open the my_sql.sql file in the VS Code IDE editor
          
◦ Scroll through the contents of the file.

    /*!999999\- enable the sandbox mode */ 
    -- MariaDB dump 10.19  Distrib 10.5.25-MariaDB, for Linux (x86_64)
    --
    -- Host: 3.94.253.183    Database: COFFEE
    -- ------------------------------------------------------
    -- Server version 8.0.41-0ubuntu0.20.04.1
    
    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
    /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
    /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
    /*!40101 SET NAMES utf8mb4 */;
    /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
    /*!40103 SET TIME_ZONE='+00:00' */;
    /*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
    /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
    /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
    /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
    
    --
    -- Current Database: `COFFEE`
    --
    
    CREATE DATABASE /*!32312 IF NOT EXISTS*/ `COFFEE` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
    
    USE `COFFEE`;
    
    --
    -- Table structure for table `suppliers`
    --
    
    DROP TABLE IF EXISTS `suppliers`;
    /*!40101 SET @saved_cs_client     = @@character_set_client */;
    /*!40101 SET character_set_client = utf8 */;
    CREATE TABLE `suppliers` (
      `id` int NOT NULL AUTO_INCREMENT,
      `name` varchar(255) NOT NULL,
      `address` varchar(255) NOT NULL,
      `city` varchar(255) NOT NULL,
      `state` varchar(255) NOT NULL,
      `email` varchar(255) NOT NULL,
      `phone` varchar(100) NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
    /*!40101 SET character_set_client = @saved_cs_client */;
    
    --
    -- Dumping data for table `suppliers`
    --
    
    LOCK TABLES `suppliers` WRITE;
    /*!40000 ALTER TABLE `suppliers` DISABLE KEYS */;
    INSERT INTO `suppliers` VALUES (1,'Nikki Wolf','100 Main Street','Anytown','CA','nwolf@example.com','4155558212');
    /*!40000 ALTER TABLE `suppliers` ENABLE KEYS */;
    UNLOCK TABLES;
    /*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
    
    /*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
    /*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
    /*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
    /*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
    /*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
    /*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
    /*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
    
    -- Dump completed on 2025-02-04 19:20:24

 <ol>
<li>Notice that it will create a database named COFFEE and a table named suppliers.</li>
<li>Also, because you added a record using the application web interface earlier in this lab, the</li> script inserts that record into the suppliers table.</li>
<li>Make a small change to one of the values in the file.</li>
<li>Locate the line that starts with INSERT INTO. It will appear around line 51.</li>
<li>Modify the address that you entered. For example, if the address has a street named Main change it to Container. Note: This change will help you later in the lab when you want to confirm that you are connected to the new database running on a container, and not the old database.</li>
<li>Choose File > Save to the change.</li>
</ol>

In the terminal, to create a directory to store your mysql container code and navigate into the directory, run the following commands:
      
       cd /home/ec2-user/environment/containers
       mkdir mysql
       cd mysql
       
<h3>Create a Dockerfile.</h3>
To create a new Dockerfile, run the following command:
      
    • touch Dockerfile

To move the sqldump file into the new mysql directory, run the following command: 

    mv ../my_sql.sql .

Open the empty Dockerfile (in containers/mysql/) and then copy and paste the following code into the file: 

    FROM mysql:8.0.23
    COPY ./my_sql.sql /
    EXPOSE 3306

<img width="583" alt="image" src="https://github.com/user-attachments/assets/7d61b44e-3702-4f39-916e-3a9cbf1dec02" />

Save the changes. 

<h3>Analysis: </h3>

This Dockerfile code specifies that a new Docker image should be created by starting with an existing mysql Docker image. Then, the sqldump file, which you created in a previous step, is copied to the container. The code also specifies that the container should allow network traffic on TCP port 3306, which is the standard MySQL port for network communication. 

Attempt to free up some disk space on the VS Code IDE instance by removing unneeded files. Run the following command:
    
      docker rmi -f $(docker image ls -a -q)

Note: You can safely ignore any Error response from daemon messages that display. Finally, run the following command:

      sudo docker image prune -f && sudo docker container prune -f
     
 Note: In some cases, the total reclaimed space might be zero; however, you might see that some unneeded containers were deleted.

 <img width="781" alt="image" src="https://github.com/user-attachments/assets/120ffebd-33bf-4b5a-af6f-5a2dba94f58a" />

 To build an image from the Dockerfile, run the following command: 
 
    docker build --tag mysql_server .

<img width="694" alt="image" src="https://github.com/user-attachments/assets/84ae5cbf-241d-4718-8350-b4b9ccaa23b8" />

Verify that the Docker image was created.

• To list the Docker images that your Docker client is aware of, run the following command:
     
      docker images
• The output is similar to the following:

<img width="758" alt="image" src="https://github.com/user-attachments/assets/0e9fb02f-a5aa-4289-b105-784c3ead28f8" />

Notice the mysql_server line item, which was created only a few minutes ago 
Create and run a Docker container based on the Docker image. To create and run a Docker container from the image, run the following command:

    docker run --name mysql_1 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=rootpw -d mysql_server

<img width="913" alt="image" src="https://github.com/user-attachments/assets/616a519f-5b2b-43c6-879d-91a69501d980" />

Analysis: This command launches a container with the name mysql_1, using the mysql_server image that you created as the template. The -e parameter passes an environment variable.
   
 • The terminal returns the container ID for the container.
    
• To view the Docker containers that are currently running on the host, run the following command:

     docker container ls

<img width="868" alt="image" src="https://github.com/user-attachments/assets/096a75e3-db9f-4096-a49a-d084883218ac" />

Two containers are now running. One hosts the node application, and the other hosts the MySQL database.  Note the CONTAINER ID  (a83ce12870c0) of the mysql_1, you will need this later.
docker inspect  a83ce12870c0

Import the data into the MySQL database and define a database user.  Run the following command:
      
⚠️ Note that space is not included between -p and rootpw in the command.

    
    sed -i '1d' my_sql.sql
    docker exec -i mysql_1 mysql -u root -prootpw < my_sql.sql

Ignore the warning about using a password on the command line interface being insecure.
    
To create a database user for the node application to use, run the following command:

    docker exec -i mysql_1 mysql -u root  -prootpw -e "CREATE USER 'nodeapp' IDENTIFIED WITH mysql_native_password BY 'coffee'; GRANT all privileges on *.* to 'nodeapp'@'%';"

 <h2>Task 5: Testing the MySQL container with the node application</h2>

 Recall that in a previous task you connected to the node application running in the container, but it was connected to the MySQL database that was running on the MysqlServerNode EC2 instance.
In this task, you will update the node application running in the container to point to the MySQL database running in the container. The following diagram shows the migration that you will accomplish in this task:

<img width="437" alt="image" src="https://github.com/user-attachments/assets/e5061306-42b9-4b98-8682-69f83fbf5f0e" />

To stop and remove the node application server container, run the following command:
      
       docker stop node_app_1 && docker rm node_app_1

Discover the network connectivity information. To find the IPv4 address of the mysql_1 container on the network, run the following command:
     
      docker inspect <CONTAINER ID>
      
The following example output shows only a portion of the output:

<img width="227" alt="image" src="https://github.com/user-attachments/assets/4b4b4a8e-8cae-45ba-9c7b-caff9fee8f82" />

<h3>Start a new node application Docker container</h3>
      In this step, you run the Docker command to start a new container from the node_app Docker image. However, this time you pass the APP_DB_HOST environment variable to the container environment. Run the following command. Replace <ip-address> with the actual IPv4 address value that you just discovered. You do not need to surround the IP address in quotes.

    docker run -d --name node_app_1 -p 3000:3000 -e APP_DB_HOST=172.17.0.3 node_app

The container starts as it did previously. 

<img width="751" alt="image" src="https://github.com/user-attachments/assets/383a28c3-b579-43c3-96e2-8e4303eecce6" />

To verify that both containers are running again, run the following command:
      
       docker ps
  <img width="683" alt="image" src="https://github.com/user-attachments/assets/0418273f-a4f6-48a9-93ca-5f299d84e968" />

  <h4>Test the application.</h4>
  
  ◦  Open the web application that is running as a container on the VS Code IDE. The address for the web application is: http://<LabIDE-public-ip-address>:3000
       
  ◦  In the coffee suppliers application, choose List of suppliers to verify that the database is connected. If you see the supplier entry that you created previously, and the entry contains the change that you made to the street name (for example, Container Street), then that is confirmation that you have successfully connected the node_app running in the container to the database running in the other container.

<img width="931" alt="image" src="https://github.com/user-attachments/assets/358f6987-bf73-4ae0-8972-dee49e8635bd" />

One of the important benefits of running an application on containers is the portability and scalability that doing so provides. Sofía has now proven that she can launch functional containers from each of the two Docker images that she created, she is ready to move this solution into production.

<h2>Task 6: Adding the Docker images to Amazon ECR</h2>

In this final task in the lab, you will add the Docker images that you created to an Amazon Elastic Container Registry (Amazon ECR) repository.  Authorize your Docker client to connect to the Amazon ECR service.
<ol>
<li>Discover your AWS account ID.</li>
<li>Copy the My Account value from the menu. This is your AWS account ID.</li>
<li>Next, return to the VS Code IDE Bash terminal.</li>
<li>To authorize your VS Code IDE Docker client, run the following command. Replace <account-id> with the actual account ID that you just found:</li>
</ol>

    aws ecr get-login-password \
    --region us-east-1 | docker login --username AWS \
    --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

 <img width="629" alt="image" src="https://github.com/user-attachments/assets/afdb598c-54c7-4bdb-a413-5dc4f62ed23a" />

 To create the repository, run the following command:

    aws ecr create-repository --repository-name node-app

The response data is in JSON format and includes a repositoryArn value. This is the URI that you would use to reference your image for future deployments.
The response also includes a registryId, which you will use in a moment.

<img width="525" alt="image" src="https://github.com/user-attachments/assets/74867b81-6779-44f9-8543-92f107933ca9" />

<h3>Tag the Docker image.</h3>
In this step, you will tag the image with your unique registryId value to make it easier to manage and keep track of this image.

Run the following command. Replace <registry-id> with your actual registry ID number.

    docker tag node_app:latest <registry-id>.dkr.ecr.us-east-1.amazonaws.com/node-app:latest

To verify that the tag was applied, run the following command:
     
      docker images
      
This time, notice that the latest tag was applied and the image name includes the remote repository name where you intend to store it. The following image provides an example of the output:

<img width="896" alt="image" src="https://github.com/user-attachments/assets/9124672e-3c7c-4430-8280-d1d9abefaf2c" />

Push the Docker image to the Amazon ECR repository.  To push your image to Amazon ECR, run the following command. Replace <registry-id> with your actual registry ID number:
      
      docker push <registry-id>.dkr.ecr.us-east-1.amazonaws.com/node-app:latest
      
The output is similar to the following:

<img width="672" alt="image" src="https://github.com/user-attachments/assets/367d0e40-7965-4d52-b18a-7204f476dbdc" />

To confirm that the node-app image is now stored in Amazon ECR, run the following aws ecr list-images command: 

    aws ecr list-images --repository-name node-app

 <img width="636" alt="image" src="https://github.com/user-attachments/assets/4bcecb84-c3c2-4b49-acd2-35a5de55f56e" />

 <h2>Update from the café</h2>

Sofía is satisfied that she has made progress. She successfully containerized the web application from the coffee supplier company that the café owners purchased. The application had previously been installed directly on the guest OS of an EC2 instance. 

Now the application is containerized, and she has more flexibility to deploy the application and scale it cost effectively.
She also successfully containerized the backend database that the application uses. Finally, she was able to successfully register the Docker image in Amazon ECR to launch the coffee supplier application in the future.
In the next lab, Sofía will use the Docker image that she just stored in Amazon ECR to deploy the coffee supplier application using AWS Elastic Beanstalk. Although Sofía also containerized the MySQL database, she has decided not to push that image to Amazon ECR. She spoke with one of the AWS consultants who came in for coffee today, and the consultant convinced her that it makes more sense to use the Amazon Relational Database Service (Amazon RDS) service to host the coffee supplier database, instead of hosting it on a container. The next lab will discuss the reasons for this, but for now Sofía is quite content that she was able to containerize the coffee supplier application. She looks forward to deploying it in the next lab!


© 2024 Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited. 

















      


























