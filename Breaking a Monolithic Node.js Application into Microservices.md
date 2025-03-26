<h1>Breaking a Monolithic Node.js Application into Microservices</h1>
<h2>Overview</h2>

Traditional monolithic architectures are hard to scale. As an application's code base grows, it becomes complex to update and maintain. Introducing new features, languages, frameworks, and technologies becomes challenging, which limits innovation and new ideas.

Within a microservices architecture, each application component runs as its own service and communicates with other services through a well-defined API. 
Microservices are built around business capabilities, and each service performs a single function. Microservices can be written by using different frameworks and programming languages, and you can deploy them as individual services, as a single service, or as a group of services.

In this lab, I migrated a monolithic application that runs in a standard Node.js server to a containerized Docker environment. I then refactor the application into microservices and deploy it to a containerized environment orchestrated by Amazon Elastic Container Service (Amazon ECS). 

The Node.js application implements the functions of a message board where users can create topic threads and post messages on each thread. The following diagram illustrates the evolution of the application's design as it moves from monolithic to microservices based.

<img width="710" alt="image" src="https://github.com/user-attachments/assets/b75a0c02-e6b3-4978-a10b-1a0adee3853b" />

The diagram highlights the following differences between the monolithic approach and the microservices design:
<ol>
<li>In a monolithic design, all of the functions of the Node.js application are packaged and run as a single service. </li>
<li>If one function fails, the entire application fails. Likewise, if one application function experiences a spike in demand, all functions in the service must be scaled together.</li>
<li>In a microservices architecture, each function of the Node.js application runs as a separate service. The services can scale and be updated independently of each other.</li>
</ol>

<h3>What I learned:</h3>
Throughout the lab, I:

✅Migrated a monolithic Node.js application to run in a Docker container.
      
✅Refactor a Node.js application from a monolithic design to a microservices architecture.
      
✅Deploy a containerized Node.js microservices application to Amazon ECS.

<h2>Task 1: Preparing the development environment</h2>

An AWS Cloud9 environment has been created for me as part of the creation process for the lab environment. AWS Cloud9 is a cloud-based integrated development environment (IDE) that you can use to write, run, and debug code with just a browser. It comes pre-packaged with essential tools for popular programming languages and provides access to the AWS Command Line Interface (AWS CLI) in a terminal session tab. 

AWS Cloud9 environment has access to the all of the Amazon Web Services (AWS) resources authorized for the user ID with which you signed in to the AWS Management Console.
To set up my development environment, I open the AWS Cloud9 IDE, and download and extract the required lab files.

<img width="875" alt="image" src="https://github.com/user-attachments/assets/30df8b68-a6f5-49c2-aa77-652d44ac8762" />

Next, download and extract the required lab files. In the bottom pane of the IDE, in the terminal tab labeled bash - "ip-nnn-nnn-nnn-nnn", run the following command:

    curl -s https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-2-91555/16-lab-mod13-guided-1-ECS/s3/lab-files-ms-node-js.tar.gz | tar -zxv

This command retrieves a compressed archive file containing the lab files and extracts its contents in the AWS Cloud9 ~/environment folder. The downloaded and extracted files are visible in the left pane. 

<img width="365" alt="image" src="https://github.com/user-attachments/assets/119f9cc2-6875-4ba7-bdbb-d5676066e38c" />

In the left pane, you should see the following folders:
       
✅ 1-no-container: This folder contains the files related to the monolithic implementation of the application intended to run directly on a Node.js server.
       
✅ 2-containerized-monolith: This folder contains the files related to the monolithic implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.

✅ 3-containerized-microservices: This folder contains the files related to the microservices implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.

You use the AWS Cloud9 IDE tab frequently throughout this lab, so keep this tab open.

 In the left pane, you should see the following folders:
       
✅ 1-no-container: This folder contains the files related to the monolithic implementation of the application intended to run directly on a Node.js server.
       
✅ 2-containerized-monolith: This folder contains the files related to the monolithic implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.
       
✅ 3-containerized-microservices: This folder contains the files related to the microservices implementation of the application intended to run in a Docker-containerized environment orchestrated by Amazon ECS.
       
  You use the AWS Cloud9 IDE tab frequently throughout this lab, so keep this tab open.

  <h2>Task 2: Running the application on a basic Node.js server</h2>
The base Node.js application is a monolithic service that has been designed to run directly on a server without a container. 

In this task, you deploy the application to the Node.js server that is installed on the instance running your AWS Cloud9 environment. You then test the application by using the command line terminal to invoke its Representational State Transfer (RESTful) API methods. The following diagram illustrates the deployment architecture and request flow.

<img width="668" alt="image" src="https://github.com/user-attachments/assets/994c9b80-d80e-40dd-8373-68976a66ebbb" />

The monolithic implementation of the application uses the Node.js cluster functionality to spawn one worker process for each CPU core. The processes share a single port and are invoked in a round-robin fashion by the load balancer built into Node.js. This feature increases scalability on servers that have multiple CPU cores.
In this task, you do the following:

 • Install the Node.js modules required by the application.
      
 • Review the application design and code.
      
`• Run the application.

<h3>Task 2.1: Installing the required Node.js modules</h3>
The message board application uses two modules from the Node.js koa framework in its implementation: koa and koa-router. 

Koa.js is a widely used Node.js web application framework that facilitates the building of asynchronous server-side JavaScript applications.

       In the AWS Cloud9 IDE terminal, to install the koa and koa-router modules, enter the following commands:
       cd ~/environment/1-no-container
       npm install koa
       npm install koa-router

<img width="785" alt="image" src="https://github.com/user-attachments/assets/016f2104-604d-43ca-8dec-e93f32b1be1c" />

The modules are downloaded and installed in the 1-no-container/node_modules folder of the AWS Cloud9 ~/environment folder. 

<h2>Task 2.2: Reviewing the application design and code</h2>
The components that implement the monolithic message board application are contained in the 1-no-container folder. Review them to gain an understanding of the application design and code.
In the AWS Cloud9 IDE terminal, in the left pane, expand the 1-no-container folder. The components of the application consist of the following:
         
<h4>node_modules: </h4>
This folder was created when you installed the required JavaScript modules in the previous sub-task and contains their source code.

<img width="167" alt="image" src="https://github.com/user-attachments/assets/e2bbc0e1-33d8-41f6-baf0-09c88c41a0c7" />

✅ db.json: This item is a JSON object that simulates the message board database. It contains attributes representing users, threads, and posts with corresponding sample values.
    
            {
      "users": [
        {
          "id": 1,
          "username": "marceline",
          "name": "Marceline Singer",
          "bio": "Cyclist, musician"
        },
        {
          "id": 2,
          "username": "finn",
          "name": "Finn Alberts",
          "bio": "Adventurer and hero, defender of good"
        },
        {
          "id": 3,
          "username": "pb",
          "name": "Paul Barium",
          "bio": "Scientist, cake lover"
        },
        {
          "id": 4,
          "username": "jake",
          "name": "Jake Storm",
          "bio": "Soccer fan, sky diver"
        }
      ],
    
      "threads": [
        {
          "id": 1,
          "title": "Did you see the Brazil game?",
          "createdBy": 4
        },
        {
          "id": 2,
          "title": "New French bakery opening in the neighborhood tomorrow",
          "createdBy": 3
        },
        {
          "id": 3,
          "title": "In search of a new guitar",
          "createdBy": 1
        }
      ],
    
      "posts": [
        {
          "thread": 1,
          "text": "That last goal was awesome!",
          "user": 4
        },
        {
          "thread": 1,
          "text": "Yes, the way the ball swerved... What talent!",
          "user": 2
        },
        {
          "thread": 2,
          "text": "I have to try their tarts!",
          "user": 3
        },
        {
          "thread": 2,
          "text": "I'm planning to stop by in the morning to try their croissants.",
          "user": 2
        },
        {
          "thread": 2,
          "text": "I could go for a chocolate eclair!",
          "user": 1
        },
        {
          "thread": 3,
          "text": "I need a new acoustic guitar at a good price.",
          "user": 1
        }
      ]
    }

✅ index.js: This item is the JavaScript program that is the application's entry point.

      const cluster = require('cluster');
      const http = require('http');
      const numCPUs = require('os').cpus().length;
      
      if (cluster.isMaster) {
        console.log(`Leader ${process.pid} is running`);
      
        // Fork workers.
        for (let i = 0; i < numCPUs; i++) {
          cluster.fork();
        }
      
        cluster.on('exit', (worker, code, signal) => {
          console.log(`worker ${worker.process.pid} died`);
        });
      } else {
        require('./server.js');
      
        console.log(`Worker ${process.pid} started`);
      }

✅ package-lock.json: This item is a JSON object that was automatically generated when you installed the required JavaScript modules in the node_modules folder. It is used by the npm installation utility to keep track of modifications made to the folder. 
<img width="959" alt="image" src="https://github.com/user-attachments/assets/57979e9d-c0a5-4c6d-abf3-054023a8979b" />

✅ package.json: This item is a JSON object describing the application, its entry point, and its dependencies.

<img width="482" alt="image" src="https://github.com/user-attachments/assets/e510bf90-9d6a-4e81-8ac2-e0ddb2051bf1" />

✅ server.js: This item is the JavaScript program that defines the application's RESTful API methods and implements their respective handlers.

            const app = require('koa')();
      const router = require('koa-router')();
      const db = require('./db.json');
      
      // Log requests
      app.use(function *(next){
        const start = new Date;
        yield next;
        const ms = new Date - start;
        console.log('%s %s - %s', this.method, this.url, ms);
      });
      
      router.get('/api/users', function *(next) {
        this.body = db.users;
      });
      
      router.get('/api/users/:userId', function *(next) {
        const id = parseInt(this.params.userId);
        this.body = db.users.find((user) => user.id == id);
      });
      
      router.get('/api/threads', function *() {
        this.body = db.threads;
      });
      
      router.get('/api/threads/:threadId', function *() {
        const id = parseInt(this.params.threadId);
        this.body = db.threads.find((thread) => thread.id == id);
      });
      
      router.get('/api/posts/in-thread/:threadId', function *() {
        const id = parseInt(this.params.threadId);
        this.body = db.posts.filter((post) => post.thread == id);
      });
      
      router.get('/api/posts/by-user/:userId', function *() {
        const id = parseInt(this.params.userId);
        this.body = db.posts.filter((post) => post.user == id);
      });
      
      router.get('/api/', function *() {
        this.body = "API ready to receive requests";
      });
      
      router.get('/', function *() {
        this.body = "Ready to receive requests";
      });
      
      app.use(router.routes());
      app.use(router.allowedMethods());
      
      app.listen(3000);
      
      Next, you examine the package.json object. 

<img width="624" alt="image" src="https://github.com/user-attachments/assets/f26aaa96-491f-4184-a54f-6b4fad3c9b7d" />

Next, you examine the package.json object. In the left pane, double-click package.json to open it in an editor tab. Notice the following attributes of the JSON object:

✅ Lines 2–5: The dependencies attribute defines the JavaScript module dependencies for the application. Note that the koa and koa-router modules that you installed in the previous sub-task are listed here.
         
✅ Lines 6–8: The scripts attribute indicates that the index.js program is the entry point to the application.

<img width="482" alt="image" src="https://github.com/user-attachments/assets/0b3124a7-42da-461b-82b8-322f59b8a838" />

Next, you examine the db.json object. In the left pane, double-click db.json to open it in an editor tab. Notice the following attributes of the JSON object: 

Lines 2–27: These lines define a users attribute that represents the registered users of the message board. The attribute value is a list of four sample users with the following names: Marcerline Singer, Finn Alberts, Paul Barium, and Jake Storm.

<img width="482" alt="image" src="https://github.com/user-attachments/assets/8b729a27-bd6c-4797-87ea-9fc0dd6e3cdc" />

Lines 29–45: These lines define a threads attribute that represents the current active threads on the message board. The attribute value is a list of three sample threads with the following titles:
<ol>
<li>Did you see the Brazil game?</li>
<li>New French bakery opening in the neighborhood tomorrow</li>
<li>In search of a new guitar</li>
</ol>

<img width="547" alt="image" src="https://github.com/user-attachments/assets/5556fb57-232f-4464-8d1e-9d6273b3b7fe" />

Lines 47–78: These lines define a posts attribute that represents the posted messages on the active threads. The attribute value is a list of six sample message posts.

<img width="626" alt="image" src="https://github.com/user-attachments/assets/cc8cbb20-6b9c-40b8-9b1d-a0c840232a08" />

Next, you review the code for the index.js object. In the left pane, double-click index.js to open it in an editor tab. Notice the following attributes:
Lines 1–3: These lines import the JavaScript modules that the program requires, specifically, cluster, http, and OS.

<img width="461" alt="image" src="https://github.com/user-attachments/assets/549ff463-41c1-4304-9f8e-082588d53345" />

Line 3: This line uses the os JavaScript module to check the number of CPU cores available on the server.

<img width="635" alt="image" src="https://github.com/user-attachments/assets/d1e3d1c1-6cee-4ba6-a63d-81e0752b1373" />

Lines 5 through 15: These lines are run the first time that the program is invoked (when the application is started). They create one leader thread for the cluster and one worker thread for each CPU core available on the server.

<img width="561" alt="image" src="https://github.com/user-attachments/assets/2e68127f-f951-4c6c-a270-9060503e2534" />

Lines 16–19: These lines handle each request to the application by invoking the server.js program in the current worker thread.

<img width="504" alt="image" src="https://github.com/user-attachments/assets/61fc1e42-e195-41df-89e4-5b876b32ffcb" />

Lastly, you review the code for the server.js object. In the left pane, double-click server.js to open it in an editor tab. Use the comments provided in the code to facilitate your understanding of the logic. In particular, notice the following: 
       
Line 3: This line imports the JSON object (db.json) that simulates the database.

<img width="670" alt="image" src="https://github.com/user-attachments/assets/5b137244-d2d8-47a6-afa6-c09dce1b4f1c" />

Lines 6–11: These lines define a generator function that gets run for every request. Its purpose is to print a line containing the HTTP method, resource path URL, and elapsed time for each request that is processed.

<img width="577" alt="image" src="https://github.com/user-attachments/assets/059bc13f-e680-4eaf-b885-6683ae694f74" />

Lines 13–47: These lines define the application's RESTful API methods and their implementation. Specifically, the application can respond to the following RESTful calls:

<img width="553" alt="image" src="https://github.com/user-attachments/assets/4313986a-c95c-4c0b-88de-6f176023f9dd" />

<ol>
<li>GET /api/users returns the collection of users in the database.</li>
<li>GET /api/users/:userId returns the information for the user identified by :userId. </li>
<li>GET /api/threads returns the collection of threads in the database.</li>
<li>GET /api/threads/:threadId returns the information for the thread identified by :threadId.</li>
<li>GET /api/posts/in-thread/:threadId returns the collection of post messages for the thread identified by :threadId.</li>
<li>GET /api/posts/by-user/:userId returns the collection of post messages for the user identified by :userId.</li>
<li>GET /api/ returns the message "API ready to receive requests."</li>
<li>GET / returns the message "Ready to receive requests."</li>
</ol>

Line 52: This line defines the port number on which the application is listening for requests.

<h2>Task 2.3: Running the application</h2>
In this task, you start the Node.js server and run the message board application. You then test some of its RESTful API methods. In the terminal tab, to start the Node.js server and the application, enter the following command:
       
       npm start

<img width="446" alt="image" src="https://github.com/user-attachments/assets/0d4bea4c-1294-447e-8236-a651db16f88e" />

The server is started, and the application's entry point, index.js, is started. The first time it is invoked, index.js creates two cluster threads—Leader and Worker—to process requests.
Next, you need to keep the current terminal session active and open a second terminal tab to test the application's RESTful API.

In the bottom pane, choose (+), and choose New Terminal to open a new terminal tab. You now have two terminals in which you can enter commands. In the right terminal tab, to retrieve the /api/users resource, enter the following command:

      curl localhost:3000/api/users
      
The RESTful invocation returns a JSON object containing the list of users in the message board database. 

<img width="955" alt="image" src="https://github.com/user-attachments/assets/b4bfaa9e-1332-4942-8054-5227de1b0677" />

Choose the left terminal tab. You see the message that is output from server.js indicating that it has processed a GET method request on the resource identified by the path /api/users. The request took 4 milliseconds to process. 

<img width="700" alt="image" src="https://github.com/user-attachments/assets/6d58cda0-82ab-4899-9193-8310d13301b3" />

Similarly, you retrieve information about threads, posts etc and observe the output and the messages in left terminal tab. 

Retrieve information about 4th user: In the right terminal tab, enter the following command:

      curl localhost:3000/api/users/4

<img width="627" alt="image" src="https://github.com/user-attachments/assets/9692a524-bde4-4cd4-89ce-5b8e590309ed" />

Retrieve threads: In the right terminal tab, enter the following command: 

      curl localhost:3000/api/threads

<img width="797" alt="image" src="https://github.com/user-attachments/assets/bdce18ec-e695-4fb5-8fdb-67c71f949729" />

Retrieve thread 1 : In the right terminal tab, enter the following command:

      curl localhost:3000/api/posts/in-thread/1
<img width="805" alt="image" src="https://github.com/user-attachments/assets/0417fbbf-5513-4ff8-9320-c46ff1f2146b" />

Now you stop the Node.js server. In the left terminal tab, press Ctrl+C to shut down the server process. You have validated that the application responds properly to RESTful GET requests and can now proceed to containerize it.

<img width="554" alt="image" src="https://github.com/user-attachments/assets/d72321a3-7bd1-4da7-bdba-08cd8d794904" />

<h2>Task 3: Containerizing the monolith for Amazon ECS</h2>

Containers wrap application code in a unit of deployment that captures a snapshot of the code and its dependencies. They can help ensure that applications deploy quickly, reliably, and consistently regardless of the deployment environment. In this task, you build a container image for the monolithic message board application and push it to Amazon Elastic Container Registry (Amazon ECR). These steps are in preparation for deploying the application to Amazon ECS.

Specifically, you perform the following steps:

✅ Prepare the application for Docker containerization.

✅ Provision a repository.
      
✅ Build and push the Docker image to the repository.

<h3>Task 3.1: Preparing the application for Docker containerization</h3>

To put the message board application into a Docker container, you need to make the following changes to the application:
    
•  Remove the use of the Node.js cluster feature, and convert the application to a single process design. 
      
With Docker containers, the goal is to run a single process for each container rather than a cluster of processes.

•  Create a Docker file for the application. This file is a build script that contains instructions on how to build a container image for the application.

The 2-containerized-monolith folder of your AWS Cloud9 environment includes a container-ready version of the application. In this task, you review the files so that you understand the changes that were made to prepare the application for containerization. In the left pane, expand the 2-containerized-monolith folder, and double-click package.json to open it in an editor tab. 

<img width="482" alt="image" src="https://github.com/user-attachments/assets/4db89db2-2e8f-48bd-894d-8c268c59211f" />

Notice in line 7 that the entry point into the application has been changed from index.js to server.js. The index.js file is no longer present in the application folder because it contains the initialization logic for the Node.js cluster feature, which you are no longer using. 

<img width="551" alt="image" src="https://github.com/user-attachments/assets/fb43bc24-3cb3-4e26-b102-78e189b59822" />

In the left pane, in the 2-containerized-monolith folder, double-click server.js to open it in an editor tab. The only difference between this object and the non-containerized version is the addition of line 54, which prints the message "Worker started" when the application is first started.

<img width="663" alt="image" src="https://github.com/user-attachments/assets/e8b14c14-2f1d-4372-a9d5-432dfabd14ad" />

In the left pane, in the 2-containerized-monolith folder, double-click Dockerfile to open it in an editor tab. This file contains the instructions on how to build the container image for the application.

<img width="594" alt="image" src="https://github.com/user-attachments/assets/06bf8847-97de-4924-a665-3dce9c83da41" />

Understand the following:
<ol>
<li>Line 1: The base image on which the container image is to be built is alpine-node, which is a Node.js image.
<li>Line 3: This line sets the working directory of the filesystem on the image to /srv.</li>
<li>Line 4: This line adds the contents of the 2-containerized-monolith folder (the application folder) to the current working directory of the file system of the image (set in the previous line).</li>
<li>Line 5: This line invokes the npm install command to install all of the application's library dependencies declared in the package.json file.</li>
<li>Line 7: This line informs Docker that the container listens on port 3000 at runtime.</li>
<li>Line 8: This line asks Docker to run the node server.js command, which starts the application when the image is started.</li>
</ol>

Now that you understand how the container image for the application will be built, you look at where to put the image once it is built.

<h3>Task 3.2: Provisioning the repository</h3>

Docker container images are intended to be stored in a repository for sharing, version control, and convenient management purposes. Developers can use the Amazon ECR service to store, manage, and deploy Docker container images. In addition, Amazon ECR is integrated with Amazon ECS, allowing Amazon ECS to pull container images directly for production deployments.
In this sub-task, you create a repository in Amazon ECR to house the container image for the message board application.

On the AWS Management Console, in the search box, enter and choose Elastic Container Registry. In the Create a repository section, choose Get Started.  If you are shown the options menu under the Repositories menu, choose hyperlink Private repositores

For the Repository name enter mb-repo
       
Note: Ensure that under Visibility settings, Private is chosen.

Choose Create repository.
       
A message is displayed at the top of the page indicating that the repository was successfully created.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/5740fe18-b7be-4284-a758-ed8cf896789b" />

<h2>Task 3.3: Building and pushing the Docker image</h2>

You are now ready to build the container image for the application and push it to the Amazon ECR repository that you just created.
One of the useful features of the Amazon ECR console is that it provides ready-to-use command templates to build and push an image to the new repository that you just created. 

You use these provided AWS CLI commands in the next steps. In the message window at the top of the page, choose View push commands. A pop-up window titled Push commands for mb-repo opens. This window lists four AWS CLI commands that are customized for the mb-repo and purposely built to do the following:

<ol>
<li>Authenticate your Docker client to your Amazon ECR registry.</li>
<li>Build your Docker image.</li>
<li>Tag your Docker image.</li>
<li>Push you Docker image to the repository.</li>
</ol>

Notice also that the pop-up window offers two versions of the commands: macOS/Linux and Windows. Make sure that the macOS/Linux tab is selected because you are going to run these commands in your AWS Cloud9 environment.

<img width="542" alt="image" src="https://github.com/user-attachments/assets/4b98efab-7d5a-4d1a-8eef-14eec2835529" />

First, you copy and run the command to log your Docker client in to your registry. In the pop-up window, for the first command, choose the Copy icon to copy it to the clipboard. The command looks like the following:
            
            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 371458831472.dkr.ecr.us-east-1.amazonaws.com

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the left terminal tab, and press Enter to run the command. If the command completes successfully, it returns the message "Login Succeeded." You can ignore the displayed warnings.

<img width="828" alt="image" src="https://github.com/user-attachments/assets/e3e3b7e4-bad3-47f8-bd38-8bff65aadb85" />

Next, you build the Docker image for your application. Note: When a specific terminal tab is not mentioned in an instruction step, use the left terminal tab.
In the terminal tab, to change the directory to the 2-containerized-monolith folder, enter the following command:

    cd ~/environment/2-containerized-monolith

Switch to the Elastic Container Registry browser tab. In the pop-up window, for the second command, choose the Copy icon to copy it to the clipboard. The command looks like the following: 
       
       docker build -t mb-repo .

Note: Make sure to include the period at the end of the command. Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the terminal tab, and press Enter to run the command: The build command produces a lot of output as it runs the instructions contained in the application's Docker file. When it is finished, you see the message.. 

<img width="683" alt="image" src="https://github.com/user-attachments/assets/0fc9ed44-e6b9-4f06-be93-be668c7f1abf" />

Next, you tag the image with the repository URI so that it can be pushed to the repository. Switch to the Elastic Container Registry browser tab. In the pop-up window, for the third command, choose the Copy icon to copy it to the clipboard. The command looks like the following:
       
      docker tag mb-repo:latest 371458831472.dkr.ecr.us-east-1.amazonaws.com/mb-repo:latest

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the terminal tab, and press Enter to run the command: If the command completed successfully, it does not return anything

<img width="688" alt="image" src="https://github.com/user-attachments/assets/fade1ab7-9dbf-42f5-b3bf-545665e5375d" />

Finally, you push the container image to the application's repository. Switch to the Elastic Container Registry browser tab. In the pop-up window, for the fourth command, choose the Copy icon to copy it to the clipboard. The command looks like the following:

            docker push 371458831472.dkr.ecr.us-east-1.amazonaws.com/mb-repo:latest

<img width="734" alt="image" src="https://github.com/user-attachments/assets/24ab3678-f77f-47e4-bc03-8080fb4db36b" />

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Paste the copied command into the terminal tab, and press Enter to run the command: The command outputs several messages as each layer of the image is pushed to the repository. Next, you verify that the image was successfully uploaded. Next, you verify that the image was successfully uploaded. Switch to the Elastic Container Registry browser tab. In the Push commands for mb-repo pop-up window, choose Close. In the Private registry > Repositories list, choose mb-repo. In the images list, you see the container image that you pushed, which is identified by the latest tag.

<img width="850" alt="image" src="https://github.com/user-attachments/assets/ed6e0a55-8706-46f7-b055-b474ee17b9a0" />

Next, you record the image URI. In the Images list, next to Copy URI, choose the Copy icon. Paste the value in a text editor. You use it in a subsequent step.
       
Great job! You have successfully created a container image for the message board application and pushed it to an Amazon ECR repository.

<h2>Task 4: Deploying the monolith to Amazon ECS</h2>
In this task, you deploy the containerized monolithic application to an Amazon ECS runtime environment. Specifically, you use Amazon ECS to create a managed cluster of Amazon Elastic Compute Cloud (Amazon EC2) instances on which to deploy your application container image. The cluster is configured as the target group of an Application Load Balancer to provide failover and scalability. The following diagram shows the deployment architecture of the containerized monolithic application. It also displays the resources that you create in this task.


<img width="699" alt="image" src="https://github.com/user-attachments/assets/cfcb7917-02cc-4c0f-9151-002d858faee6" />


In this task, you perform the following steps:
<ol>
<li>Create an Amazon ECS cluster.</li>
<li>Create a task definition for the application container image.</li>
<li>Deploy the monolithic application as an Amazon ECS service.</li>
<li>Test the containerized monolithic application.</li>
</ol>

<h2>Task 4.1: Creating an Amazon ECS cluster</h2>
An Amazon ECS cluster is a logical grouping of EC2 instances on which you can run tasks or services representing your containerized application. In this sub-task, you create an Amazon ECS cluster by using the Amazon ECS console. The console's cluster creation wizard facilitates the creation of all of the infrastructure components needed to create the Amazon ECS cluster environment, including the virtual private cloud (VPC), subnets, security groups, internet gateway, and AWS Identity and Access Management (IAM) roles. On the AWS Management Console, in the search box, enter and select Elastic Container Service 

<img width="935" alt="image" src="https://github.com/user-attachments/assets/d799e780-7495-49ca-9a4b-a5ae0bcc343a" />

 From the left menu, choose Clusters. Choose Create cluster, and configure the following options:
<ol>
<li>In the Cluster configuration section, for the Cluster name, enter mb-ecs-cluster</li>
<li>In the Infrastructure section, select Amazon EC2 instances, and configure the following options:</li>
<li>Leave the Auto Scaling group (ASG) settings at defaults.</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/83fb75ab-2d23-4617-b3e9-f605fa615d4d" />

 ▪  For Operating system/Architecture, choose Amazon Linux 2023.
              
 ▪  For EC2 instance type, choose t2.medium.

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/4c50c076-8350-4c08-9348-1dffb9fe890c" />

 ▪  For Desired capacity, for Minimum and Maximum, enter 2

 <img width="949" alt="image" src="https://github.com/user-attachments/assets/3441a52a-cf19-4486-b00b-59701f1c7ae4" />

 ◦ In the Network settings for Amazon EC2 instances, configure the following options:
          
 ▪  VPC: choose Lab VPC.
              
 ▪  Subnets: Reconfirm that, all public subnets chosen.

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/c4254260-c6ee-4a45-ba6c-9b250150082c" />

▪ Security group: Choose Use an existing security group.
              
▪ Security group name dropdown list, select the security group that has ECSSG in the name.
              
▪ Clear the default security group.         

 <img width="953" alt="image" src="https://github.com/user-attachments/assets/0530040b-3041-469a-be53-7254431e1c58" />

 Choose Create.

 It takes a few minutes for the cluster to be created. Choose the cluster you created, e.g. mb-ecs-cluster. The details page for mb-ecs-cluster is displayed. Notice that the Status shows a value of Active.

 <img width="919" alt="image" src="https://github.com/user-attachments/assets/2165fee0-401b-4344-8b06-07e46bcc95d4" />

 <img width="953" alt="image" src="https://github.com/user-attachments/assets/5a872262-1792-4374-a9e0-9106a4862fec" />

 Choose the Infrastructure tab.

 The Container instances pane shows that two EC2 instances for the cluster were created.

 <img width="900" alt="image" src="https://github.com/user-attachments/assets/74dadb88-7dff-42c0-b526-8917a2707766" />

 <h3>Task 4.2: Creating a task definition for the application container image</h3>
A task definition is a list of configuration settings for how to run a Docker container on Amazon ECS. The following are examples of the information that a task definition provides to Amazon ECS:
<ol>
<li>Which container image to run</li>
<li>How much CPU and memory the container needs</li>
<li>On which ports the container listens to traffic</li>
</ol>

In this sub-task, you create a task definition for the container image of the message board application. On the Amazon ECS console, from the left menu, choose Task definitions.
Choose Create new task definition, and configure the following options:

• In the Task definition configuration section, for Task definition family, enter mb-task
    
• In the Infrastructure requirements, select Amazon EC2 instances, and clear AWS Fargate.

<img width="825" alt="image" src="https://github.com/user-attachments/assets/816ea704-a961-4f76-be8c-e840d9cb64b7" />

◦  For the Task size, choose CPU: .5vCPU, Memory: 1GB

◦  For Task execution role, choose Create new role.

<img width="839" alt="image" src="https://github.com/user-attachments/assets/f497ef33-365c-4ff8-b769-aeee74fff3e4" />

 ◦ In the Container - 1 section, configure the following options:
            
▪ For Container details, for Name, enter mb-container
           
▪ For Image URI, paste the URI of the users container image that you copied to a text editor earlier.
           
▪ For Port mappings, for Container port, enter 3000. This option specifies the port on which the container receives requests.

<img width="873" alt="image" src="https://github.com/user-attachments/assets/dd347024-4d94-4c8f-a8a4-627d42ff1a1d" />

Choose create
A message is displayed at the top that says, "Task definition successfully created." You now have a task definition that tells Amazon ECS how to deploy your application container across the cluster.

<img width="842" alt="image" src="https://github.com/user-attachments/assets/a1ddd904-8e82-48a0-98a5-b1fbdf26fc52" />

<h3>Task 4.3: Creating and deploying the service</h3>
All of the required Amazon ECS infrastructure components are created, and you can now deploy the containerized monolithic application to the cluster as an Amazon ECS service.
You can use Amazon ECS to run and maintain a specified number of instances of a task definition simultaneously in an Amazon ECS cluster. If one of the tasks fails or stops for any reason, the Amazon ECS service scheduler launches another instance of the task definition to replace it and maintains the desired number of tasks specified in the service.
In this sub-task, you use the Amazon ECS console to create an Amazon ECS service for the message board application's task definition. On the same screen where you created mb-task, choose Deploy, then choose Create service. In the Environment section, configure the following options:

• For Compute options, choose Launch type.
    
• For Launch type, choose EC2.

<img width="781" alt="image" src="https://github.com/user-attachments/assets/c454e194-c1c1-4988-a0cb-15aa28e706fd" />

For Service name choose mb-ecs-service

<img width="818" alt="image" src="https://github.com/user-attachments/assets/11919ab4-dea3-4308-8677-b73f768107d4" />

• Expand the Networking section, and configure the following options:
      
◦ For Security group, choose Use an existing security group.
          
◦ From the Security group name dropdown list, select the security group that has ECSSG in the name.
          
◦ Clear the default security group.

 <img width="791" alt="image" src="https://github.com/user-attachments/assets/80853e5e-8d66-42c0-b956-7df5350d43f7" />

• Expand the Load balancing - optional section, and configure the following options:
      
◦ For Load balancer type, choose Application Load Balancer.
          
◦ For Application Load Balancer, choose Create a new load balancer.

◦ For Load balancer name, enter mb-load-balancer

<img width="781" alt="image" src="https://github.com/user-attachments/assets/d7cf8856-9c8f-4c00-ae41-bb9f0873744e" />

 For Target group name, enter mb-target

 <img width="788" alt="image" src="https://github.com/user-attachments/assets/e6fdf6d1-ef50-4a09-8f17-76fc542b7013" />

Choose Create.

Wait for a few minutes for the service to create all the components. From the Services list, choose the service that you just created, mb-ecs-service. It should show a Status of Active with one Task that's Running.

<img width="835" alt="image" src="https://github.com/user-attachments/assets/bd75eaaf-b8bd-4626-8081-1d12436e9d72" />

In the Status section, choose View load balancer. Copy the DNS name for the load balancer, and paste it into a new browser tab.

<img width="781" alt="image" src="https://github.com/user-attachments/assets/3251ede3-ba5e-4ae8-b120-49bc338d7026" />

The browser page should display a message that says, "Ready to receive requests."

<img width="694" alt="image" src="https://github.com/user-attachments/assets/38fe649d-54e8-49e7-aad8-3f9f60fc9f45" />

You have successfully deployed the containerized monolith as an Amazon ECS service into the cluster. Also paste the DNS name into a text editor. You use it later in this lab.

<h3>Task 4.4: Testing the containerized monolith</h3>
You can now validate your deployment by testing the RESTful API methods of the message board application from a web browser.
You need the DNS name of the load balancer that you used in the previous steps. Open a new browser tab, paste the DNS name into the address field, and press Enter. Enter the following addresses in the browser tab, and examine the results. For each address, replace DNS name with the DNS name from the previous steps.

• DNS name/api
      
• DNS name/api/users


<img width="949" alt="image" src="https://github.com/user-attachments/assets/bdec7754-6b04-4688-bad7-e1b2a7f5782a" />

• DNS name/api/threads

<img width="959" alt="image" src="https://github.com/user-attachments/assets/910a2bd9-7998-4d05-a148-57bb13b5a418" />

DNS name/api/posts/in-thread/2 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/b6a22d2e-892f-468f-9a61-505f8e61823b" />

The returned results are similar to what you observed while running the application locally in the AWS Cloud9 IDE.

Now that you have containerized the monolith application, you can refactor it.

<h2>Task 5: Refactoring the monolith Application</h2>
In this task, you break the containerized monolithic message board application into several interconnected services and microservices and push each service's image to an Amazon ECR repository. Each microservice performs a single business capability of the application and can be scaled independently of the other microservices. Specifically, the application is divided into the following microservices, which represent the top-level classes of objects that the application's RESTful API serves:
<ol>
<li>Users microservice: A service for all user-related REST paths (/api/users/*)</li>
<li>Threads microservice: A service for all thread-related REST paths (/api/threads/*)</li>
<li>Posts microservice: A service for all post-related REST paths (/api/posts/*)</li>
</ol>
      
To expedite the refactoring task, a microservices version of the application is provided to you in the 3-containerized-microservices folder of your AWS Cloud9 environment.
In this task, you perform the following steps:

• Review the refactored microservices application.

• Provision an Amazon ECR repository for each microservice.

• Build and push the images for each microservice.

<h3>Task 5.1: Reviewing the refactored microservices application</h3>
Take a few minutes to review the files and understand the changes that were made in order to refactor the application into microservices.

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. In the left pane, expand the 3-containerized-microservices folder. Notice that there are now three separate sub-folders: posts, threads, and users. These folders represent the three application microservices. Each sub-folder contains the implementation files for the corresponding microservice.

Expand the posts, threads, and users sub-folders.

<img width="602" alt="image" src="https://github.com/user-attachments/assets/314b245a-0ff8-43ce-885f-8f04bb7b5e60" />

Notice that each sub-folder contains a copy of the same application files as those of the containerized monolith application. In fact, the db.json, Dockerfile, and package.json files in each sub-folder are identical to their containerized monolith counterparts. The only file that changed as a result of the re-factoring is server.js. In the posts sub-folder, double-click server.js to open it in an editor tab. The difference from the containerized monolith version is that the program defines only the RESTful API methods and implementation related to the posts resource paths (lines 13–21).

<img width="499" alt="image" src="https://github.com/user-attachments/assets/ae2b778d-5a60-47cd-8fa9-c580cd209f50" />

In the threads sub-folder, double-click server.js to open it in an editor tab. The difference from the containerized monolith version is that the program defines only the RESTful API methods and implementation related to the threads resource paths (lines 13–20).

<img width="554" alt="image" src="https://github.com/user-attachments/assets/8e495788-90e0-4fd3-b7b7-75746080b8ec" />

In the users sub-folder, double-click server.js to open it in an editor tab. The difference from the containerized monolith version is that the program defines only the RESTful API methods and implementation related to the users resource paths (lines 13–20).

<img width="513" alt="image" src="https://github.com/user-attachments/assets/eb77b932-4b3c-453c-9b54-ede3b5da48bb" />

In summary, the only change required to refactor the application was to split the RESTful API method handlers in the monolithic version of server.js into three separate server.js files. Each separate server.js file contains a relevant subset of the API method handlers. 

<h3>Task 5.2: Provisioning an ECR repository for each microservice</h3>
Similarly to what you did for the containerized monolith version, you create an Amazon ECR repository for each of the application's microservices.
In this task, you create a repository for the users, threads, and posts microservice container images.

On the AWS Management Console, in the search box, enter and choose Elastic Container Registry. From the menu on the left, choose Repositories, then choose Create repository.
       
For Repository name, enter mb-users-repo
       
Note: Ensure that under Visibility settings, Private is chosen. Choose Create repository. 

A message is displayed at the top of the page indicating that the repository was successfully created. Repeat the steps in this sub-task to create a repository named mb-threads-repo for the threads microservice container image. Repeat the steps in this sub-task to create a repository named mb-posts-repo for the posts microservice container image.
When you have created the repositories for all three microservices, the Private registry > Repositories list looks like the following:

<img width="849" alt="image" src="https://github.com/user-attachments/assets/71ee81ee-6b95-482b-84d8-8a9a40c15cef" />

<h3>Task 5.3: Building and pushing the images for each microservice</h3>
Next, you build each microservice container image and push it to its corresponding repository. In this sub-task, you use the ready-to-use commands provided by the Amazon ECR console to facilitate the task.

<h4>Task 5.3.1: Building and pushing the image for the users microservice</h4>
Switch to the Cloud9-IDE - AWS Cloud9 browser tab. In the terminal tab, to change directory to the 3-containerized-microservices/users folder, enter the following command:

            cd ~/environment/3-containerized-microservices/users

Switch to the Elastic Container Registry browser tab. From the Private repositories list, choose mb-users-repo. At the top of the page, choose View push commands. A pop-up window titled Push commands for mb-users-repo opens. Next, you build the Docker image for the microservice.

<img width="541" alt="image" src="https://github.com/user-attachments/assets/21b70111-df5f-4c18-814c-457e8e71d6c1" />

 In the pop-up window, for the second command, choose the Copy icon to copy it to the clipboard. The command looks like the following:

      docker build -t mb-users-repo .

Note: Make sure to include the period at the end of the command. Run the copied command in the AWS Cloud9 terminal. When the command is finished, you see the messages similar to "Building 5.1s (9/9) FINISHED."

<img width="560" alt="image" src="https://github.com/user-attachments/assets/834da199-f471-4c0f-b978-962a0048bcf6" />

Next, you tag the image with the repository URI so that it can be pushed to the repository. Switch to the Elastic Container Registry browser tab. In the pop-up window, for the third command, choose the Copy icon to copy it to the clipboard. The command looks like the following:

      docker tag mb-users-repo:latest 371458831472.dkr.ecr.us-east-1.amazonaws.com/mb-users-repo:latest

Run the copied command in the AWS Cloud9 terminal. The command returns to the prompt. Finally, you push the container image to the microservice's repository.
Switch to the Elastic Container Registry browser tab. In the pop-up window, for the fourth command, choose the Copy icon to copy it to the clipboard. The command looks like the following:

      docker push 371458831472.dkr.ecr.us-east-1.amazonaws.com/mb-users-repo:latest

<img width="725" alt="image" src="https://github.com/user-attachments/assets/41e222b1-027b-4297-8085-791c9c4d2472" />

Switch to the Cloud9-IDE - AWS Cloud9 browser tab. Run the copied command in the AWS Cloud9 terminal. The command outputs several messages as each layer of the image is pushed to the repository. Next, you verify that the image was successfully uploaded.

Switch to the Elastic Container Registry browser tab. In the Push commands for mb-users-repo pop-up box, choose Close. Choose Refresh. In the Images list, you see the container image that you pushed, which is identified by the latest tag. Next, you record the image URI. 

<img width="827" alt="image" src="https://github.com/user-attachments/assets/6a16c597-c701-476f-90fe-a3fb89c4a7f2" />

In the Images list, choose Copy URI for the Image URI of the latest version of the image. Paste the value into a text editor, and label it as the users image URI. You use it in a later step.
Next, you build and push the container image for the threads microservice.

<h4>Task 5.3.2: Building and pushing the image for the threads microservice</h4>
In this sub-task, you use the following information and repeat the previous the steps in the previous task to build and push the image for the threads microservice.
       On the Cloud9-IDE - AWS Cloud9 tab, to change the directory to the 3-containerized-microservices/threads folder, enter the following command:
Switch to the Elastic Container Registry browser tab.
       In the left navigation, choose Repositories, and choose mb-threads-repo.
       At the top of the page, choose View push commands. 
       A pop-up window titled Push commands for mb-threads-repo opens.
       Repeat the steps from the previous task to do the following:
          Build the Docker image for the microservice.

<img width="574" alt="image" src="https://github.com/user-attachments/assets/b60fbc85-fca0-47d2-86ab-0f40f46aef28" />

Tag the image with the repository URI so that it can be pushed to the repository. 

Push the container image to the microservice's repository.

<img width="763" alt="image" src="https://github.com/user-attachments/assets/d2506ce1-e86d-40c2-aa90-0f1588b83118" />

       After you have completed these steps for the threads microservice, you verify that the image was successfully uploaded.
In the Push commands for mb-threads-repo pop-up box, choose Close.
       Choose Refresh. 
       In the Images list, you see the container image that you pushed, which is identified by the latest tag.

<img width="857" alt="image" src="https://github.com/user-attachments/assets/211ef7b4-9e12-4f23-9a51-8ae88c09e34d" />

Next, you record the image URI. In the Images list, choose Copy URI for the Image URI of the latest version of the image. Paste the value into a text editor, and label it as the threads image URI. You use it in a later step. Next, you build and push the container image for the posts microservice.

<h4>Task 5.3.3: Building and pushing the image for the posts microservice</h4>
In this sub-task, you use the following information and repeat the previous the steps in the previous task to build and push the image for the posts microservice. On the Cloud9-IDE - AWS Cloud9 tab, to change the directory to the 3-containerized-microservices/posts folder, enter the following command:

      cd ~/environment/3-containerized-microservices/posts

Switch to the Elastic Container Registry browser tab.

In the left navigation, choose Repositories, and choose mb-posts-repo.
At the top of the page, choose View push commands. A pop-up window titled Push commands for mb-posts-repo opens. Repeat the steps from the previous task to do the following:

◦ Build the Docker image for the microservice.

<img width="578" alt="image" src="https://github.com/user-attachments/assets/87d1a0c2-d27b-4c43-96f9-c230508ebd21" />

◦ Tag the image with the repository URI so that it can be pushed to the repository. 
        
◦ Push the container image to the microservice's repository.

<img width="680" alt="image" src="https://github.com/user-attachments/assets/57bad816-182c-4fc7-acb1-71291e89c4f5" />

After you have completed these steps for the posts microservice, you verify that the image was successfully uploaded. Switch to the Elastic Container Registry browser tab.
 In the Push commands for mb-posts-repo pop-up box, choose Close. Choose Refresh. In the Images list, you see the container image that you pushed, which is identified by the latest tag. Next, you record the image URI. 

 <img width="948" alt="image" src="https://github.com/user-attachments/assets/e75fbce7-3c0e-44ca-b329-210e8ab15d7f" />

In the Images list, choose Copy URI for the Image URI of the latest version of the image. Paste the value into a text editor, and label it as the posts image URI. You use it in a later step.
You have successfully built container images for the microservices in your application and pushed them to Amazon ECR.

 <h3>Task 6: Deploying the containerized microservices</h3>

In this task, you deploy the containerized microservices message board application to the same ECS cluster that you used for the containerized monolith. You also use the same Application Load Balancer that you used in previous tasks, but you configure it to route requests to different target groups (one for each microservice container) based on the request URI path.
The following diagram shows the deployment architecture of the containerized microservices application. It also displays the resources that you create in this task.

<img width="694" alt="image" src="https://github.com/user-attachments/assets/0ee79a26-f677-4f21-a4a3-9ca6582d7c69" />

<h3>In this task, you perform the following steps:</h3>

• Create a task definition for each microservice.
      
• Deploy the microservices as Amazon ECS services.
      
• Validate the deployment.

 <h4>Task 6.1: Creating a task definition for each microservice</h4>
Because the microservices in the application are intended to run independently of each other, they require their own task definition. In this sub-task, you create three task definitions that run the container image of each individual microservice. 

<h4>Task 6.1.1: Creating a task definition for the users microservice</h4>

On the AWS Management Console, in the search box, enter and select Elastic Container Service In the left navigation pane, choose Task definitions
Choose Create new task definition, and configure the following options:

• In the Task definition configuration section, for Task definition family, enter mb-users-task
      
• In the Infrastructure requirements, select Amazon EC2 instances, and clear AWS Fargate.
<img width="544" alt="image" src="https://github.com/user-attachments/assets/0fe0a97d-fc20-4127-9bde-4a0f6893b11f" />
For the Task size, choose CPU: .5 vCPU, Memory: 1GB
<img width="770" alt="image" src="https://github.com/user-attachments/assets/8b91d49d-6f0f-4cfa-9181-a4561d22a5cc" />

<h5>In the Container - 1 section, configure the following options:</h5>
<ol>
<li>For Container details, for Name, enter mb-users-container</li>
<li>For Image URI, paste the URI of the users container image that you copied to a text editor earlier.</li>
<li>For Port mappings, for Container port, enter 3000. This option specifies the port on which the container receives requests.</li>
</ol>

<img width="781" alt="image" src="https://github.com/user-attachments/assets/c9a325b4-e0b6-440b-b748-d8568c42456e" />
Choose Create.
A message is displayed at the top that says, "Task definition successfully created."
<img width="744" alt="image" src="https://github.com/user-attachments/assets/5850597e-7c05-4a69-a27c-189d4bfb5fa3" />
<h5>Task 6.1.2: Creating a task definition for the posts microservice</h5>
In the left navigation pane, choose Task definitions. Choose Create new task definition, and configure the following options: In the Task definition configuration section, for Task definition family, enter mb-posts-task

<img width="772" alt="image" src="https://github.com/user-attachments/assets/30801689-4330-46ff-95fd-39ada8a80648" />
<ol>
<li>For the Task size, choose CPU: .5 vCPU, Memory: 1GB</li>
<li>For Container - 1, configure the following options:</li>
<li>For Container details, for Name, enter mb-posts-container</li>
<li>For Image URI, paste the URI of the posts container image that you copied to a text editor earlier.</li>
</ol>
<img width="747" alt="image" src="https://github.com/user-attachments/assets/55bc20ed-42e9-4592-b472-4fbea9bd3383" />
For Port mappings, for Container port, enter 3000. This option specifies the port on which the container receives requests.
<img width="754" alt="image" src="https://github.com/user-attachments/assets/7a61d211-ab75-454d-8bfe-f39047462a52" />
Choose Create.
A message is displayed at the top that says, "Task definition successfully created."
<img width="738" alt="image" src="https://github.com/user-attachments/assets/2dfb7bd3-d73e-436f-bab1-098a3cb46ebc" />

<h5>Task 6.1.3: Creating a task definition for the threads microservice</h5> 

In the left navigation pane, choose Task definitions. Choose Create new task definition, and configure the following options: In the Task definition configuration section, for Task definition family, enter mb-threads-task
<img width="770" alt="image" src="https://github.com/user-attachments/assets/b5ee7680-9f9a-4519-816f-4e191d5df65a" />
<ol>
<li>For the Task size, choose CPU: .5 vCPU, Memory: 1GB</li>
<li>For Container - 1, configure the following options:</li>
<li>For Container details, for Name, enter mb-threads-container</li>
<li>For Image URI, paste the URI of the threads container image that you copied to a text editor earlier.</li>
<li>For Port mappings, for Container port, enter 3000. This option specifies the port on which the container receives requests.</li>
</ol>
<img width="755" alt="image" src="https://github.com/user-attachments/assets/b4a510dd-0df5-403b-a24b-292f6f2c5467" />
Choose Create. A message is displayed at the top that says, "Task definition successfully created."
<img width="772" alt="image" src="https://github.com/user-attachments/assets/c9d361f9-baad-4c33-970f-710c59c16fa5" />
<img width="782" alt="image" src="https://github.com/user-attachments/assets/8c1bcc78-1cdf-458a-a9ad-92a885dc79b4" />

<h4>Task 6.2: Creating and deploying the services</h4>
All of the required Amazon ECS infrastructure components are created, and you can now deploy the containerized monolithic application to the cluster as an Amazon ECS service.
You can use Amazon ECS to run and maintain a specified number of instances of a task definition simultaneously in an Amazon ECS cluster. If one of the tasks fails or stops for any reason, the Amazon ECS service scheduler launches another instance of the task definition to replace it and maintains the desired number of tasks specified in the service.
In this sub-task, you use the Amazon ECS console to create an Amazon ECS service for the message board application's task definition.
In the left navigation pane, choose Clusters, and choose your mb-ecs-cluster cluster. On the Services tab, choose Create, and configure the following options:
In the Environment section, configure the following options:

◦ For Compute options, choose Launch type.
       
◦ For Launch type, choose EC2.

<img width="783" alt="image" src="https://github.com/user-attachments/assets/9770d0da-a9e9-4dea-96ad-455e36af0c1b" />

In the Deployment configuration section, configure the following options: 
◦ For Application type, choose Service.

◦ For Family, chose mb-users-task. 

◦ For Service name, enter mb-users-service

<img width="748" alt="image" src="https://github.com/user-attachments/assets/e702304c-2472-4554-a0cb-ada9f5597e35" />

Expand the Networking section, and configure the following options:
◦ For Security group, choose Use an existing security group.

◦ From the Security group name dropdown list, select the security group that has ECSSG in the name.

◦ Clear the default security group.
<img width="769" alt="image" src="https://github.com/user-attachments/assets/ae51c077-891b-42e6-8e40-77cc2179709d" />

 Expand the Load balancing - optional section, and configure the following options:
 
• For Load balancer type, choose Application Load Balancer.
    
• For Application Load Balancer, choose Use an existing load balancer.
    
 • For Load balancer, choose mb-load-balancer.

 <img width="766" alt="image" src="https://github.com/user-attachments/assets/ff901887-7d1c-4ce5-86f9-fd1440a64fa4" />

 For Listener, choose Use an existing listener, and then choose 80:HTTP from the dropdown list.

 <img width="748" alt="image" src="https://github.com/user-attachments/assets/f538aca0-5fe7-4b71-84d8-98faf4f7dbf6" />

• For Target group, choose Create new target group.

• For Target group name, enter mb-users-target

• For Path pattern, enter /api/users*

• For Evaluation order, enter 1

<img width="746" alt="image" src="https://github.com/user-attachments/assets/8d7e23f2-eb27-4a6c-ba42-1a54c3b11002" />

Choose Create.

Wait a few minutes for the service to create all the components.

<img width="781" alt="image" src="https://github.com/user-attachments/assets/3e288d3b-9932-495a-b07c-351d344b0a78" />

Return to your mb-ecs-cluster cluster. On the Services tab, choose Create, and configure the following options:

<ol>
<li>In the Environment section, configure the following options:</li>
<li>For Compute options, choose Launch type.</li>
<li>For Launch type, choose EC2.</li>
<li>In the Deployment configuration section, configure the following options: </li>
<li>For Application type, choose Service.</li>
<li>For Family, choose mb-posts-task.</li>
<li>For Service Name, enter mb-posts-service</li>
</ol>

<img width="799" alt="image" src="https://github.com/user-attachments/assets/a66082d7-9a6b-487a-8476-1ac5ccee2e2d" />

Expand the Networking section, and configure the following options:

◦ For Security group, choose Use an existing security group.

◦ From the Security group name dropdown list, select the security group that has ECSSG in the name.

◦ Clear the default security group.
   
 Expand the Load balancing section, and configure the following options:

◦ For Load balancer type, choose Application Load Balancer.

◦ For Application Load Balancer, choose Use an existing load balancer.

◦ For Load balancer, choose mb-load-balancer.

◦ For Listener, choose Use an existing listener, and then choose 80:HTTP from the dropdown list.

◦ For Target group, choose Create new target group.

◦ For Target group name, enter mb-posts-target

◦ For Path pattern, enter /api/posts*

◦ For Evaluation order, enter 2

<img width="763" alt="image" src="https://github.com/user-attachments/assets/0ec5f541-2276-4093-81b5-eafad316a5b6" />

Choose Create

<img width="834" alt="image" src="https://github.com/user-attachments/assets/e5ff44a9-2b52-4dfe-9354-4f1463db6c30" />

Wait a few minutes for the service to create all the components.
Return to your mb-ecs-cluster cluster. On the Services tab, choose Create, and configure the following options:

In the Environment section, configure the following options:
       
◦ For Compute options, choose Launch type.

◦ For Launch type, choose EC2.

<img width="782" alt="image" src="https://github.com/user-attachments/assets/b56293c6-83be-4b10-95c9-b1531673fbe1" />

 In the Deployment configuration section, configure the following options: 

◦ For Application type, choose Service.

◦ For Family, choose mb-threads-task.

◦ For Service Name, enter mb-threads-service

<img width="838" alt="image" src="https://github.com/user-attachments/assets/bf984cb1-b246-4e31-8353-6a1099fc4293" />

Expand the Networking section, and configure the following options:

◦ For Security group, choose Use an existing security group.

◦ From the Security group name dropdown list, select the security group that has ECSSG in the name.

Clear the default security group.

Expand the Load balancing section, and configure the following options:

◦ For Load balancer type, choose Application Load Balancer.

◦ For Application Load Balancer, choose Use an existing load balancer.

◦ For Load balancer, choose mb-load-balancer.

<img width="809" alt="image" src="https://github.com/user-attachments/assets/ba99f931-98f6-49cd-ab13-a0d3bb430e8d" />

For Listener, choose Use an existing listener, and then choose 80:HTTP from the dropdown list.
        
◦ For Target group, choose Create new target group.
        
◦ For Target group name, enter mb-threads-target
        
◦ For Path pattern, enter /api/threads*
        
◦ For Evaluation order, enter 3

<img width="889" alt="image" src="https://github.com/user-attachments/assets/65bcc3a9-cec5-49ea-869d-223bc3388423" />

Choose Create.
       
Wait a few minutes for the service to create all the components.

<img width="839" alt="image" src="https://github.com/user-attachments/assets/082e1e54-cfc7-4752-ac9d-53820b28586e" />

Return to the Clusters menu on the ECS console, and choose mb-ecs-cluster. On the Services menu, choose any service you created, go to view load balancer. For mb-load-balancer, on the Listeners and rules tab, choose HTTP:80. Under the Listener rules notice the rules similar to the following:

<img width="844" alt="image" src="https://github.com/user-attachments/assets/60882b69-3d4e-4d81-ac18-9f7334a175dd" />

Notice that the requests are forwarded to the respective groups, and a default rule forwards the request to the mb-target that was created before refactoring. You modify this rule provide a different "Invalid request~.." message in case the requests do not match the desired URL path. 

Choose the Default rule, then choose Actions and Edit rule.
       
Under the Routing actions, choose Forward to target groups.
       
For Target group choose mb-users-target.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/0e391606-7aa1-4f56-ad06-610f3ac7a887" />

Choose Save changes.

Notice the new rules as per following, all requests are routed to mb-users-target if request parameters are not specified. 

<img width="828" alt="image" src="https://github.com/user-attachments/assets/96350b2f-bb89-4495-a344-3f04ccb3e677" />

Return to the Clusters menu on the ECS console, and choose mb-ecs-cluster.

On the Services tab, select mb-ecs-service, and choose Update.
       
On the Update mb-ecs-service page, for Desired tasks, enter 0
      
Choose Update.

<img width="803" alt="image" src="https://github.com/user-attachments/assets/28771a87-2b84-478c-947e-eff4cdddce07" />

The service should not be running any tasks now. The original monolithic container has been stopped, and all requests will be serviced by three services individually.
On the Amazon ECS browser tab, return to your mb-ecs-cluster cluster. Notice the services and tasks that are running.
<img width="757" alt="image" src="https://github.com/user-attachments/assets/75ec6c65-af63-47b1-a44f-25610190b9a0" />

<h3>Task 6.3: Validating the deployment</h3>
You can now test the RESTful API methods of the message board application from a web browser and validate that the microservices-based implementation works correctly.
Before testing, shut down the Amazon ECS service for the containerized monolith version of the application. Make sure that it will no longer serve any requests. Open a new browser tab. In the address field, enter the load balancer DNS name that you pasted into a text editor earlier, and press Enter. A page is returned with a message that says, "Ready to receive requests."

<img width="537" alt="image" src="https://github.com/user-attachments/assets/b9cbd61b-850e-4e59-aea5-13ee1329a778" />

This is the message that the application returns when no resource path is included in the GET request. Recall from the listener rules configuration that this type of request is sent for processing to the users microservice. In the browser address bar, to the end of the URL, add /api and press Enter.
       
The message "API ready to receive requests" is returned by the application, specifically the users microservice.

<img width="562" alt="image" src="https://github.com/user-attachments/assets/adc47145-0fca-4dae-938b-8d54694b06e5" />

Next, you test additional URLs. In the browser address bar, enter the following addresses in the browser tab, and examine the results. For each address, replace DNS name wit the DNS name that you copied to a text editor earlier. DNS name/api/users 

Expected output: List of users 

<img width="944" alt="image" src="https://github.com/user-attachments/assets/a7e253cc-5655-4b2f-b7da-54b8f0fde0fc" />

DNS name/api/users/2 

Expected output: Details of user 2

<img width="554" alt="image" src="https://github.com/user-attachments/assets/1146842e-4664-40ad-ac77-6670e893fbfa" />

DNS name/api/threads 
Expected output: List of threads

<img width="959" alt="image" src="https://github.com/user-attachments/assets/85ce8bb9-dea6-4b63-bf9d-b6612e04ab13" />

• DNS name/api/posts/in-thread/2

Expected output: Details of thread 2 

<img width="575" alt="image" src="https://github.com/user-attachments/assets/e4ffd71a-8925-4e1f-9fd3-8f3d24e2dcd5" />

 • DNS name/xxx
      Expected output: Not found (invalid request)
<img width="533" alt="image" src="https://github.com/user-attachments/assets/3b6b8f7b-f17e-443f-a77e-03bc8bc6b806" />

<h2>Conclusion</h2>
Congratulations! You now have successfully done the following:

• Migrated a monolithic Node.js application to run in a Docker container
    
• Refactored a Node.js application from a monolithic design to a microservices architecture
    
• Deployed a containerized Node.js microservices application to Amazon ECS

© 2023, Amazon Web Services, Inc. or its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited. 























































































 


 






























































