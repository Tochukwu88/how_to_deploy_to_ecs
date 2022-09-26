# how_to_deploy_to_ecs
In this exercise, you will use Amazon Elastic Container Registry (Amazon ECR) to store the Docker images you created in the previous exercise. You will use Amazon Elastic Container Service (Amazon ECS) to deploy and manage the containers. Amazon ECS is a highly scalable, high-performance container orchestration service that supports Docker containers and allows you to easily run and scale containerized applications on AWS.

To begin, follow the steps below.

# 1. Create an Amazon ECR repository for timezones-frontend container.
In this section, you will create an Amazon ECR repository for the timezones-frontend container you built in the last exercise. Follow the steps below.

Sign in to the AWS Management Console as the edXOptimizingUser IAM user.
Make sure you are in the Oregon region.
In your AWS Cloud9 environment, run the command below to see the Docker images that you built in the last exercise.
docker images

You should see an output similar to below. You should see the two Docker images, timezones-frontend and timezones-appserver, that you built and tested in the last exercise.
  REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
  timezones-frontend    latest              82d4563ac65f        22 hours ago        104MB
  timezones-appserver   latest              5dcd1185b0b5        23 hours ago        102MB
  python                alpine3.4           8eb1c554687d        5 days ago          90.4MB
  lambci/lambda         python3.6           e0a5dc25cde0        6 weeks ago         1.1GB
  lambci/lambda         nodejs6.10          41f202a579aa        6 weeks ago         1.02GB
  lambci/lambda         python2.7           9e76d0c1940f        6 weeks ago         972MB
  lambci/lambda         nodejs4.3           c9bf81763271        6 weeks ago         937MB
In the console, click Services, then click Elastic Container Service to open the Amazon ECS dashboard.
In the left side navigation menu, click Repositories.
Click Get started.
To create a repository for the timezones-frontend image, type timezones-frontend for Repository name.
Click Next step.
You should see a success message and a list of steps with commands to push the Docker image to the repository. You will run these commands in your AWS Cloud9 terminal.
In your AWS Cloud9 terminal, run the command below to authenticate your Docker client to your registry. The command below will give you a docker login command. You will run the docker login command in the next step.
aws ecr get-login --no-include-email --region us-west-2

You should see a docker login command.
Copy the docker login command.
Run the docker login command copied from the previous step.
You should see a message similar to WARNING! Using --password via the CLI is insecure. Use --password-stdin. Login Succeeded. You may ignore the warning message. You can read more about the warning message here.
You have already built the timezones-frontend image in the last exercise. In the next step, you will tag the timezones-frontend image for the ECS registry destination. To do this we tag the image with the Amazon ECR registry, repository (timezones-frontend) and latest image tag.
On the Build, tag, and push Docker image page in the console, copy the docker tag command mentioned in step 4. Refer to the screenshot below.

In your AWS Cloud9 terminal, run the docker tag command you copied in the previous step.
Run the command below to see the tag applied to the timezones-frontend image.
docker images

Observe that the tag is applied to the timezones-frontend image.
Copy the repository URL for the timezones-frontend image. Refer to the screenshot below. You will use the repository URL in a subsequent section.

To push the timezones-frontend container image to Amazon ECR, copy the docker push command mentioned in step 5 in the console. Refer to the screenshot below.

In your AWS Cloud9 terminal, run the docker push command you copied in the previous step.
Once the image is pushed, return to the console and click Done at the bottom. You should see that the timezones-frontend repository contains the timezones-frontend image tagged as latest.
 # 2. Create an Amazon ECR repository for timezones-appserver container.
In this section, you will create an Amazon ECR repository for the timezones-appserver container you built in the last exercise. Follow the steps below.

In the console, click Services, then click Elastic Container Service to open the Amazon ECS dashboard.
In the left side navigation menu, click Repositories.
Click Create repository.
For Repository name, type timezones-appserver
Click Next step.
You should see a success message and a list of steps with commands to push the Docker image to the repository. You will run these commands in your AWS Cloud9 terminal.
Since you have already authenticated via the docker login command in the last section, you don't have to run the docker login command for the timezones-appserver repository again.
You have already built the timezones-appserver image in the last exercise. In this step, you will tag the timezones-appserver image. On the Build, tag, and push Docker image page in the console, copy the docker tag command mentioned in step 4.
In your Cloud9 terminal, run the docker tag command you copied in the previous step.
Run the command below to see the tag applied to the timezones-appserver image.
docker images

Observe that the tag is applied to the timezones-appserver image.
Copy the repository URL for the timezones-appserver image. Refer to the screenshot below. You will use the repository URL in a subsequent section.

To push the timezones-appserver container image to Amazon ECR, copy the docker push command mentioned in step 5 in the console.
In your Cloud9 terminal, run the docker push command you copied in the previous step.
Once the image is pushed, return to the console and click Done at the bottom. You should see that the timezones-frontend repository contains the timezones-appserver image tagged as latest.
 # 3. Create an Amazon ECS cluster.
In this section, you will create an Amazon ECS and deploy the containers you pushed to the Amazon ECR repositories in the last section. Follow the steps below.

In the console, click Services, then click Elastic Container Service to open the Amazon ECS dashboard.

In the left side navigation menu, click Clusters.
Click Create Cluster.
Click the EC2 Linux + Networking box.
Click Next step.
For Cluster name, type optimizing-cluster
For EC2 instance type, select t2.micro which is free tier eligible.
If you have a key pair in your account, you can select the key pair in the Key pair drop-down menu. This will enable you to SSH into the cluster instance and help you with debugging if something goes wrong. Note: This is an optional step.
Leave the rest of the defaults and scroll down and click Create. It will take 2-3 minutes for Amazon ECS to create the cluster.
Once the cluster is created, click View cluster to view the cluster details.
# 4. Create an Application Load Balancer.
In this section, you will create an Application Load Balancer and configure it such that the web traffic to the Amazon ECS cluster is routed via the load balancer. Follow the steps below.

In the console, click Services, then click EC2 to open the Amazon EC2 dashboard.
Make sure you are in the Oregon region.
In the left side navigation menu, under LOAD BALANCING, click Load Balancers.
Click Create Load Balancer.
Click Create in the Application Load Balancer box.
For Name, type timezones-alb
Scroll down to VPC.
For VPC, select the VPC with the CIDR range as 10.0.0.0/16. This is the same VPC in which the Amazon ECS cluster is present.
Select both Availability Zones in the VPC.
Click Next: Configure Security Settings.
Click Next: Configure Security Groups.
On the Configure Security Groups page, select the Create a new security group option.
For Security group name, type alb-sg
Click Next: Configure Routing.
On the Configure Routing page, for Name, type default-target
Click Next: Register Targets.
Skip the Register Targets page and click Next: Review.
Click Create. This will provision the load balancer.
Click Close.
# 5. Edit the security group for the Amazon ECS cluster.
In this section, you will modify the Amazon ECS cluster security group rule to allow all TCP traffic from the application load balancer.

In the console, click Services, then click EC2 to open the Amazon EC2 dashboard.
In the left side navigation menu, click Security Groups.
Locate and select the security group for the Amazon ECS cluster. The name of the security group is similar to EC2ContainerService-optimizing-cluster-EcsSecurityGroup-XXXXXXX.
Click the Inbound tab at the bottom.
Click Edit.
For Type, select All TCP.
For Source, delete the existing text, type alb and then select the security group that appears.
Click Save.
You have successfully modified the Amazon ECS cluster security group to allow all TCP traffic from the application load balancer.

 # 6. Create a task definition for the timezones-appserver container.
In this section, you will create a task definition for the timezones-appserver container. Then you will create a task from the task definition and run the task as service inside the Amazon ECS cluster. Follow the steps below.

In the console, click Services, then click Elastic Container Service to open the Amazon ECS dashboard.
In the left side navigation menu, click Task Definitions.
Click Create new Task Definition.
On the Select launch type compatibility page, click the EC2 box.
Click Next step.
For Task Definition Name, type timezones-appserver-task
Scroll down to the Task size section.
For Task memory (MiB), type 256
Click Add container.
For Container name, type timezones-appserver
For Image, type REPLACE_WITH_APPSERVER_REPO_URL:latest
Important: Replace REPLACE_WITH_APPSERVER_REPO_URL above with the timezones-appserver repository URL you copied in a previous section.
For Memory Limits (MiB), keep the default selection to Hard limit and type 256
For Port mappings, in the Host port textbox, type 0 and in the Container port textbox, type 8081
Scroll down to STORAGE AND LOGGING.
For Log configuration, enable Auto-configure CloudWatch Logs
Leave the rest of the settings to default and click Add at the bottom.
Click Create. You should see a success message.
You have successfully created a task definition for the timezones-appserver container.

 # 7. Launch a task for the timezones-appserver container.
In this section, you will launch a task as a service from the timezones-appserver-task task definition and run the task inside the Amazon ECS cluster. Follow the steps below.

In the Amazon ECS dashboard, click Clusters in the left side navigation menu.
Click the optimizing-cluster hyperlink.
Under the Services tab, click Create.
For Launch type, select EC2.
Observe that the Task Definition is the timezones-appserver-task you created in the last section.
For Service name, type timezones-appserver-service
For Number of tasks, type 1
Click Next step.
For Load balancing, select Application Load Balancer.
Scroll down and observe that the timezones-alb has been selected as the load balancer.
Observe that the timezones-appserver container and port 8081 have been selected to load balance.
Click Add to load balancer.
For Listener port, select 80:HTTP.
For Target group name, type ecs-timezones-appserver-service
For Path pattern, delete the existing text and type /api*
For Evaluation order, type 10
For Health check path, delete the existing text and type /api/v1.0/get_zones
Click Next step.
Skip the Set Auto Scaling (optional) page and click Next step.
On the Review page, scroll down and click Create Service. This will create the service for the timezones-appserver container in a few seconds.
Click View Service. You should see the task status as RUNNING. You may need to refresh the page to see the status.
# 8. Verify the timezones-appserver container via the load balancer.
In this section, you will verify whether the timezones-appserver service you launched in the Amazon ECS cluster in the last section is running and is accessible via the load balancer DNS name. Follow the steps below.

In the console, click Services, then click EC2 to open the EC2 dashboard.
In the left side navigation menu, under LOAD BALANCING, click Target Groups.
Select the ecs-timezones-appserver-service target group.
Click Targets tab at the bottom.
Observe that the instance in the Registered targets list has a status as Healthy. This means the ECS cluster sitting behind the load balancer is healthy and ready to receive traffic.
In the left side navigation menu, click Load Balancers.
Select the timezones-alb load balancer.
In the Description tab at the bottom, copy the DNS name. The DNS name is similar to timezones-alb-111111111.us-west-2.elb.amazonaws.com.
Open a web browser and paste the DNS name followed by /api/v1.0/get_zones. The URL should look like the following: timezones-alb-111111111.us-west-2.elb.amazonaws.com/api/v1.0/get_zones.
Click ENTER. You should see a JSON output as shown below returned by the get_zones API. This is the response from the timezones-appserver container that is running inside the Amazon ECS cluster.
{
  "hostname": "c1faa9a9951f",
  "zones": {
    "ap-northeast-1": {
      "TZ": "Asia/Tokyo",
      "Title": "Tokyo"
    },
    "ap-northeast-2": {
      "TZ": "Asia/Seoul",
      "Title": "Seoul"
    },
    "ap-south-1": {
      "TZ": "Asia/Kolkata",
      "Title": "Mumbai"
    },
    "ap-southeast-1": {
      "TZ": "Asia/Singapore",
      "Title": "Singapore"
    },
    "ap-southeast-2": {
      "TZ": "Australia/Sydney",
      "Title": "Sydney"
    },
    "ca-central-1": {
      "TZ": "Canada/Eastern",
      "Title": "Montreal"
    },
    "eu-central-1": {
      "TZ": "Europe/Berlin",
      "Title": "Frankfurt"
    },
    "eu-west-1": {
      "TZ": "Europe/Dublin",
      "Title": "Ireland"
    },
    "eu-west-2": {
      "TZ": "Europe/London",
      "Title": "London"
    },
    "eu-west-3": {
      "TZ": "Europe/Paris",
      "Title": "Paris"
    },
    "sa-east-1": {
      "TZ": "America/Sao_Paulo",
      "Title": "S\u00e3o Paulo"
    },
    "us-east-1": {
      "TZ": "US/Eastern",
      "Title": "N. Virginia"
    },
    "us-east-2": {
      "TZ": "US/Eastern",
      "Title": "Ohio"
    },
    "us-joke-1": {
      "TZ": "US/Mountain",
      "Title": "Roswell, NM"
    },
    "us-west-1": {
      "TZ": "US/Pacific",
      "Title": "N. California"
    },
    "us-west-2": {
      "TZ": "US/Pacific",
      "Title": "Oregon"
    }
  }
}
You have successfully verified that the timezones-appserver container is up and running in the Amazon ECS cluster and is being served via the Application Load Balancer.

9. Create a Task Definition for the timezones-frontend container.
In this section, you will create a task definition for the timezones-frontend container. Then you will create a task from the task definition and run the task as service inside the ECS cluster. Follow the steps below.

In the console, click Services, then click Elastic Container Service to open the Amazon ECS dashboard.
In the left side navigation menu, click Task Definitions.
Click Create new Task Definition.
On the Select launch type compatibility page, click the EC2 box.
Click Next step.
For Task Definition Name, type timezones-frontend-task
Scroll down to the Task size section.
For Task memory (MiB), type 256
Click Add container.
For Container name, type timezones-frontend
For Image, type REPLACE_WITH_FRONTEND_REPO_URL:latest
Important: Replace REPLACE_WITH_FRONTEND_REPO_URL above with the timezones-frontend repository URL you copied in a previous section.
For Memory Limits (MiB), keep the default selection to Hard limit and type 256
For Port mappings, in the Host port textbox, type 0 and in the Container port textbox, type 8080
Scroll down to ENVIRONMENT.
You will add an environment variable called APP_SERVER, just like you did in the previous exercise, when you set up the docker containers and ran the application locally on the Cloud9 instance. The APP_SERVER environment variable will point to the DNS name of the application load balancer which serves traffic to the timezones-appserver container.
For Env Variables, in the Key textbox, type APP_SERVER and in the Value textbox, type http://REPLACE_WITH_DNS_NAME
Important: Replace REPLACE_WITH_DNS_NAME above with the DNS name of the timezones-alb load balancer you copied in a previous section.
Scroll down to STORAGE AND LOGGING.
For Log configuration, enable Auto-configure CloudWatch Logs
Leave the rest of the settings to default and click Add at the bottom.
Click Create. You should see a success message.
You have successfully created a task definition for the timezones-frontend container.

10. Launch and verify the task for the timezones-frontend container.
In this section, you will launch a task as a service from the timezones-frontend-task task definition and run the task inside the Amazon ECS cluster. Follow the steps below.

In the Amazon ECS dashboard, click Clusters in the left side navigation menu.
Click the optimizing-cluster hyperlink.
Under the Services tab, click Create.
For Launch type, select EC2.
For Task Definition, select the timezones-frontend-task you created in the last section.
For Service name, type timezones-frontend-service
For Number of tasks, type 1
Click Next step.
For Load balancing, select Application Load Balancer.
Scroll down and observe that the timezones-alb has been selected as the load balancer.
Observe that the timezones-frontend container and port 8080 have been selected to load balance.
Click Add to load balancer.
For Listener port, select 80:HTTP.
For Target group name, select default-target
Click Next step.
Skip the Set Auto Scaling (optional) page and click Next step.
On the Review page, scroll down and click Create Service. This will create the service for the timezones-frontend container in a few seconds.
Click View Service. You should see the task status as RUNNING. You may need to refresh the page to view the status.
To verify the timezones-frontend task, paste the DNS name of the timezones-alb load balancer in a browser and click ENTER. You should see the application load. You may need to wait for a few minutes for the load balancer to serve the application.
You have successfully launched the timezones-frontend and timezones-appserver containers inside an Amazon ECS cluster and served the traffic via an application load balancer.

11. Scale the ECS cluster down.
In this section, you will scale the Amazon ECS cluster instances down to 0 to stay under the free tier limits. Follow the steps below.

In the Amazon ECS dashboard, click Clusters in the left side navigation menu.
Click the optimizing-cluster hyperlink.
Click the ECS Instances tab in the bottom pane.
Click Scale ECS Instances.
For Desired number of instances, type 0
Click Scale. This will scale the Amazon ECS cluster instances down to 0. In a subsequent exercise, you will scale the cluster back up again to observe Amazon ECS scaling.
