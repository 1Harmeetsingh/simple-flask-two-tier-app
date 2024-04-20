Flask App with MySQL Docker Setup

This is a simple Flask app that interacts with a MySQL database. The app allows users to submit messages, which are then stored in the database and displayed on the frontend.

Prerequisites
Before you begin, make sure you have the following installed:

1. Docker
2. docker-compose
3. Git (optional, for cloning the repository)
4. Jenkins
5. AWS account to deploy the application on ec2 instance

Deploying application without CI/CD pipeline:
Setup
1. Clone this repository (if you haven't already):
   git clone https://github.com/your-username/your-repo-name.git

2. Navigate to the project directory:
   cd your-repo-name

3. Create a .env file in the project directory to store your MySQL environment variables:
   touch .env

4. Open the .env file and add your MySQL configuration:
   MYSQL_HOST=mysql
   MYSQL_USER=your_username
   MYSQL_PASSWORD=your_password
   MYSQL_DB=your_database

Ways to build and run the application:
1. We can run the docker containers using docker but for this we have to run two seperate containers
   one for flaskapp and another for mysql and we have to integrate them using a same network.

2. Second option is we can use docker-compose for running multiple containers just by writing a simple
   docker-compose.yml file in which configuration for multiple containers can be defined.   

To run this two-tier application using docker-compose:
Install docker-compose using "suco apt install docker-compose"

1. Start the containers using Docker Compose:
   docker-compose up --build

2. Access the Flask app in your web browser:
   Frontend: http://localhost
   Backend: http://localhost:5000

Cleaning Up

1. To stop and remove the Docker containers, press Ctrl+C in the terminal where the containers are running, or use the following command:
 docker-compose down


To run this two-tier application using without docker-compose

1. First create a docker image from Dockerfile
   docker build -t flaskapp .

2. Now, make sure that you have created a network using following command
   docker network create twotier

Attach both the containers in the same network, so that they can communicate with each other

i) MySQL container:

   docker run -d --name mysql -v mysql-data:/var/lib/mysql -v ./message.sql:/docker-entrypoint-initdb.d/message.sql --network=twotier -e MYSQL_DATABASE=mydb
   -e MYSQL_USER=root -e MYSQL_PASSWORD=admin -e MYSQL_ROOT_PASSWORD="admin" -p 3306:3306 mysql:5.7

ii) Backend container:

    docker run -d --name flaskapp -v mysql-data:/var/lib/mysql -v ./message.sql:/docker-entrypoint-initdb.d/message.sql --network=twotier -e 
    MYSQL_HOST=mysql -e MYSQL_USER=root -e MYSQL_PASSWORD=admin -e MYSQL_DB=mydb -p 5000:5000 flaskapp:latest


Building and deploying application using jenkins CI/CD pipelines:

1. Install jenkins on ec2 server:
   Pre-Requisites
       Java(jdk)
   a). First install java using command "sudo apt install openjdk-11-jre"

   b). Now run the command:
     curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
     sudo apt-get update
     sudo apt-get install jenkins
   
 3. Check the status of jenkins by running  command on terminal "sudo systemctl status jenkins"

 4. **Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080
   in the inbound traffic rules as show below.

   EC2 > Instances > Click on
   In the bottom tabs -> Click on Security
   Security groups
   Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed All traffic).


   Login to Jenkins using the below URL:
   http://:8080 [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: 
If you are not interested in allowing All Traffic to your EC2 instance 
a). Delete the inbound traffic rule for your instance
b). Edit the inbound traffic rule to only allow custom TCP port 8080

After you login to Jenkins, - Run the command on terminal to copy the Jenkins Admin Password 
- sudo cat /var/lib/jenkins/secrets/initialAdminPassword - Enter the Administrator password

5. Install plugins:
   After logging into jenkins navigate to manage jenkins-->plugins-->available plugins
   a). Install Docker pipeline plugin
   b). Install Git plugin
   And then simply restart jenkins using command "sudo systemctl restart jenkins"

6. Store credentials:
   Navigate to manage jenkins-->credentials-->systems-->global credentials-->add credentials
   a). add credentials for github
   b). add credentials for dockerhub 
   
7. Create a job:
   Navigate to Dashboard-->create new job-->
   a). Give name and select pipeline option and save
   b). Then give description of pipeline and select:

    Source Code Management (SCM):
       Choose the type of version control system you are using (e.g., Git or Subversion).
       Enter the repository URL.
       Specify credentials if needed (SSH keys or username/password).
       Choose the branch you want to build (e.g., “*/main” for the main branch).
       Optionally, you can specify a local subdirectory for the workspace.
 

 9. Configure webhooks to automatically deploy application on every commit made to the github repo:
    a). Go to your GitHub repository and click on ‘Settings’.
        Click on Webhooks and then click on ‘Add webhook’.
        In the ‘Payload URL’ field, paste your Jenkins environment URL. At the end of this URL add /github-webhook/.
        In the ‘Content type’ select: ‘application/json’ and leave the ‘Secret’ field empty.
        Save the webhook configuration.
    
    b). Go to build trigger section in Jenkins job configuration and select GitHub hook trigger for GITScm polling.


 10. Click on build now option to trigure pipeline or you can also trigure it by commiting any change
     in code in github repository   

  11. Finally access the application on http://public_ip/5000
      For this you have to allow inbound traffic for port 5000 in security section of ec2 instance.

Notes

1. Make sure to replace placeholders (e.g., your_username, your_password, your_database) with your actual MySQL configuration.

2. This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.

3. Be cautious when executing SQL queries directly. Validate and sanitize user inputs to prevent vulnerabilities like SQL injection.

4. If you encounter issues, check Docker logs and error messages for troubleshooting.


