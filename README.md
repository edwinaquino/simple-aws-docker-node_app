# Simple AWS Docker Demo App

> Use this app to deploy a Docker container from your local machine into AWS 
Amazon Docker Container Registry (ECR). This is a very simple Nodejs hello world example app. You can use this app to build and push it to a docker container as well as to an AWS ECR.

## App Info

This is a "Hello World" Node/Express web application you can use for building and running in a Docker container as well as other container cloud services like AWS and Azure.

![1-simple-aws-docker-node_app](https://user-images.githubusercontent.com/30946443/106987248-e3e5c500-6721-11eb-8913-90dc328caa4f.png)

## Requirements
* Docker
* NodeJs
* AWS Account
* AWS CLI

## Getting Started
To get started follow these steps below. In this example, we are going to create an app called "simple-aws-docker-node_app"

1. Clone this repository<br>
```$ git clone https://github.com/edwinaquino/simple-aws-docker-node_app.git```
2. Change directory into the app.<br>
```$ cd simple-aws-docker-node_app```<br>
3. Send command: Build the nodejs application, this will create the node_modules folder.<br>
```$ npm install```
4. Build the docker container in your local machine.<br>
```$ docker build -t simple-aws-docker-node .```
5. Run the app in your local machine container.<br>
```$ docker run -p 49160:8080 -d simple-aws-docker-node```

6. open: http://localhost:49160/

7. Create an Elastic Container Registry [Example: us-west-1 Region]

https://us-west-1.console.aws.amazon.com/ecr/repositories?region=us-west-1

* Click create repository
* Visibility setting = Public
* Give Repository name = simple-aws-docker-node
* Click Create Repository

NOTE: You can obtain the following push commands by going to the repository you just created and click "View push commands"

8. Now push the docker container we just built to AWS.

Retrieve an authentication token and authenticate your Docker client to your registry.
```$ aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin [YOUR_ACCOUNT_NUMBER].dkr.ecr.us-west-1.amazonaws.com```

Build your Docker image using the following command
NOTE: We already did this command in step 4 above.
```$ docker build -t simple-aws-docker-node .```

After the build completes, tag your image so you can push the image to this repository:
```$ docker tag simple-aws-docker-node:latest [YOUR_ACCOUNT_NUMBER].dkr.ecr.us-west-1.amazonaws.com/simple-aws-docker-node:latest```

Run the following command to push this image to your newly created AWS repository:
```$ docker push [YOUR_ACCOUNT_NUMBER].dkr.ecr.us-west-1.amazonaws.com/simple-aws-docker-node:latest```

After the push completed, go to your Repositories in you ECR, click on the repository we just created and you will see an image with a tag "Latest", Click where it says: Copy URI and put it someone in your text editor.

Example URI: public.ecr.aws/xxxnnnn/simple-aws-docker-node:latest

Now to got ECS in your AWS account to create a cluster

https://us-west-1.console.aws.amazon.com/ecs/home?region=us-west-1#/clusters/create/new

Select "EC2 Linux + Networking" and click next step

* Cluster Name: simple-aws-docker-node-app
* Provisioning Model = On-Demand Instance
* EC2 instance type*: t3a.micro
* NOTE: The t3a.micro is within the free tier
* Number of instances*: 1
* EC2 Ami Id*: Amazon Linux 2 AMI
* Root EBS Volume Size (GiB) 30
* Key pair: none

Networking
* VPC: vpc-xxxxx
* Subnets: [SELECT THE FIRST CHOICE]
* Auto assign public IP: Enabled
* Security group: [Select the default]

Container instance IAM role
You are giving permission to Elastic Container Service to create and use ecsInstanceRole. Leave as default.

Tags
Leave as is

Click Create Button

Your cluster has created resourced (hardware) so you can put your container within.
After the cluster has been created. click "View Cluster"

Go to the "ECS Instance" tab to view your instance that was created.

After cluster, instance and stack have been created, go to this link:

https://us-west-1.console.aws.amazon.com/ecs/home?region=us-west-1#/taskDefinitions

Then click on the Task Definition on the left menu

click on "Create new Task Definition"

Select EC2 and click next step
* Task Definition Name* : simple-aws-docker-node-task
* Task Role: none
* Network Mode: Default
* Task memory (MiB): 500
* Task CPU (unit): 1 vCPU

Click on "Add container" button under Container Definitions

*Container name*: simple-aws-docker-node-container
Image*: REMEMBER I said to copy and paste the Repository URI to your text editor, we are going to need it on this field.

Example URI: public.ecr.aws/xxxnnnn/simple-aws-docker-node:latest

NOTE: you can get the image in the repository page of ECR:
https://us-west-1.console.aws.amazon.com/ecr/repositories?region=us-west-1


* Port mappings:
* Host: 49160
* Container Port: 8080
* NOTE: these ports were declared in step 4 

Click the Add button

Now Click the Create button

You will see a message: "Created Task Definition successfully"

Next: lets deploy a task by clicking to the "Cluster" in the left menu
click on the cluster link:  simple-aws-docker-node-app >

* Click on the Tasks tab
* Click on the "Run new Task" button
* Launch type: EC2
* Task Definition: simple-aws-docker-node-task
* revision: x (latest)
* Cluster: simple-aws-docker-node-app
* Number of tasks: 1
* Placement Templates: AZ Balanced Spread
* Click on Run Task

You should get a message "Created tasks successfully"

Go to the EC2 dashboard on the running instances button.

Click on the Instance ID link

Look on Public IPv4 DNS. Open the URL in a new browser window.

Might look something like this: 
https://ec2-xx-xx-xx-xx.us-west-1.compute.amazonaws.com

WHAT? You don't see your app? No worries, the reason why you are not able to see the app is because we need to expose the application port, we provided in step 4.

Go to the Security Groups on the left menu
Click on the security group ID
Under the inbound rules, click on the Edit inbound rules button
Be sure to add the following rule:
* Type: Custom TCP
* Protocol: TCP
* Port Range: 49160
* Source: Custom
* 0.0.0.0/0

Save Rules
![2-edit-inbound-rules](https://user-images.githubusercontent.com/30946443/106987289-fb24b280-6721-11eb-9849-43702e2b325a.png)

Now try again: 

http://ec2-54-67-89-121.us-west-1.compute.amazonaws.com:49160/

NOTE: be sure not to use https

![1-simple-aws-docker-node_app](https://user-images.githubusercontent.com/30946443/106987248-e3e5c500-6721-11eb-8913-90dc328caa4f.png)


Hope that works for you.

### Author

Edwin Aquino

### Version

1.0

### License

This project is licensed under the Apache License 2.0