##### The Nautilus DevOps team is tasked with deploying a containerized application using Amazon's container services. They need to create a private Amazon Elastic Container Registry (ECR) to store their Docker images and use Amazon Elastic Container Service (ECS) to deploy the application. The process involves building a Docker image from a given Dockerfile, pushing it to the ECR, and then setting up an ECS cluster to run the application.

###### Create a Private ECR Repository:

Create a private ECR repository named nautilus-ecr to store Docker images.
Build and Push Docker Image:

Use the Dockerfile located at /root/pyapp on the aws-client host.
Build a Docker image using this Dockerfile.
Tag the image with latest tag.
Push the Docker image to the nautilus-ecr repository.

###### Create and Configure ECS cluster:

Create an ECS cluster named nautilus-cluster using the Fargate launch type.
 ###### Create an ECS Task Definition:

Define a task named nautilus-taskdefinition using the Docker image from the nautilus-ecr ECR repository.
Specify necessary CPU and memory resources.
Deploy the Application Using ECS Service:

###### Create a service named nautilus-service on the nautilus-cluster to run the task.
Ensure the service runs at least one task.



###### Solution 

Step 1: Create a Private ECR Repository
AWS Console Method
Login to the AWS Management Console.
Navigate to ECR → Repositories → Create repository.
Configure:
Repository name: datacenter-ecr
Visibility: Private
Leave other defaults.
Click Create repository.

aws ecr create-repository \
    --repository-name datacenter-ecr \
    --region us-east-1

Step 2: Build and Push Docker Image
Step 2a: Build the Docker Image
Assuming your Dockerfile is located at /root/pyapp:
cd /root/pyapp
docker build -t datacenter-app:latest .
Step 2b: Authenticate Docker to ECR
Get the login command from AWS CLI:
aws ecr get-login-password --region us-east-1 | \
docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com
Replace <aws_account_id> with your AWS account ID.
Step 2c: Tag the Image for ECR
docker tag datacenter-app:latest <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
Step 2d: Push the Image to ECR
docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
After this, your image is now stored in datacenter-ecr.

Step 3: Create and Configure ECS Cluster
AWS Console Method
Navigate to ECS → Clusters → Create Cluster.
Choose Networking only (Fargate) → Next.
Cluster name: datacenter-cluster
Leave other defaults → Create.
AWS CLI Method
aws ecs create-cluster --cluster-name datacenter-cluster

Step 4: Create ECS Task Definition
Step 4a: Console Method
Navigate to Task Definitions → Create new Task Definition → Fargate.
Task Definition Name: datacenter-taskdefinition
Task Role: Leave default (or create IAM role if needed for permissions).
Network Mode: awsvpc
CPU: 256
Memory: 512
Container Definition:
Container name: datacenter-container
Image: <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
Port mappings: 80 (if exposing HTTP)
Click Create.
Step 4b: CLI Example
aws ecs register-task-definition \
    --family datacenter-taskdefinition \
    --requires-compatibilities FARGATE \
    --network-mode awsvpc \
    --cpu "256" \
    --memory "512" \
    --container-definitions '[{
        "name":"datacenter-container",
        "image":"<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest",
        "essential":true,
        "portMappings":[{"containerPort":80,"protocol":"tcp"}]
    }]'

Step 5: Deploy the Application Using ECS Service
Step 5a: Console Method
Navigate to Clusters → Select datacenter-cluster → Create Service.
Launch Type: Fargate
Service name: datacenter-service
Task Definition: datacenter-taskdefinition
Number of tasks: 1
Configure network: choose VPC and subnets, enable public IP if needed.
Security group: allow port 80 (for HTTP).
Click Create Service → Create.
Step 5b: CLI Example
aws ecs create-service \
    --cluster datacenter-cluster \
    --service-name datacenter-service \
    --task-definition datacenter-taskdefinition \
    --desired-count 1 \
    --launch-type FARGATE \
    --network-configuration 'awsvpcConfiguration={
        subnets=["subnet-xxxxxxx"],
        securityGroups=["sg-xxxxxxx"],
        assignPublicIp="ENABLED"
    }'
Replace subnet-xxxxxxx and sg-xxxxxxx with your subnet IDs and security group IDs.
Step 6: Verify Deployment
Go to Clusters → datacenter-cluster → Services → datacenter-service.
Ensure at least 1 task is running.
Check the task logs (CloudWatch) to verify the container is running properly.
Access the service using the public IP of the Fargate task (if port 80 is exposed) to confirm it’s serving your application.



