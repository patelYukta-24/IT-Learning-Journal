# Day 82 — Project 6: Deploying a Node.js App on AWS ECS Fargate and AWS ECR

**Part XII: Project 6 (Day 82)**
**Curriculum:** NexusCorp DevOps Transformation — Shubham Londhe

---

## ❖ Project Overview

AWS ECS Fargate allows containers to run without managing servers or clusters. AWS ECR is a Docker container registry that simplifies storing, managing, and deploying Docker container images. This project walks through deploying a Node.js application using these two services, demonstrating a serverless, containerized deployment workflow on AWS.

## ❖ Project Objective

- Gain hands-on experience with AWS ECS Fargate and AWS ECR.
- Understand the workflow of containerized deployment on AWS.
- Explore the benefits of serverless computing with Fargate.
- Learn to build, tag, and push Docker images to AWS ECR.
- Set up a continuous delivery pipeline for a Node.js application.

## ❖ Skills Showcased

- AWS ECS Fargate Service Deployment
- AWS ECR Image Management
- Docker Operations
- AWS CLI Proficiency
- Infrastructure as Code (IaC) Practices

## ❖ Environment

- **AWS services:** ECS (Fargate launch type), ECR, IAM, VPC/Security Groups, CloudWatch Logs
- **CLI tools:** AWS CLI v2, Docker
- **App:** Simple Node.js/Express application with a Dockerfile in the repo

---

## ❖ Tasks / Exercises

### Task 1: Deploy a Node.js Application on AWS ECS Fargate Using an Image Stored in ECR

#### Subtask 1 — Find and clone a Node.js application repository

Selected a small open-source Node.js/Express app that already included a working `Dockerfile`, and cloned it locally:

```bash
git clone https://github.com/<nodejs-app-repo>.git
cd nodejs-app
```

**Dockerfile (from the repo)**
```dockerfile
FROM node:18-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

#### Subtask 2 — Build the Docker image

```bash
docker build -t nodejs-ecs-app:v1 .
docker run -p 3000:3000 nodejs-ecs-app:v1
```

Verified locally at `http://localhost:3000` that the app responded correctly before moving to AWS.

#### Subtask 3 — Set up AWS CLI and authenticate

```bash
aws --version
aws configure
# AWS Access Key ID:     <access_key>
# AWS Secret Access Key: <secret_key>
# Default region name:   ap-south-1
# Default output format: json

aws sts get-caller-identity
```

The `get-caller-identity` call returned the account ID, user ARN, and user ID — confirming successful authentication before proceeding.

#### Subtask 4 — Create an ECR repository and tag the image

```bash
aws ecr create-repository \
  --repository-name nodejs-ecs-app \
  --region ap-south-1
```

This returned the repository URI in the form:
```
<account_id>.dkr.ecr.ap-south-1.amazonaws.com/nodejs-ecs-app
```

Tagged the locally built image to match the ECR repository URI:

```bash
docker tag nodejs-ecs-app:v1 <account_id>.dkr.ecr.ap-south-1.amazonaws.com/nodejs-ecs-app:v1
```

#### Subtask 5 — Push the tagged image to ECR

Authenticated Docker against ECR, then pushed:

```bash
aws ecr get-login-password --region ap-south-1 \
  | docker login --username AWS --password-stdin <account_id>.dkr.ecr.ap-south-1.amazonaws.com

docker push <account_id>.dkr.ecr.ap-south-1.amazonaws.com/nodejs-ecs-app:v1
```

Confirmed the pushed image in the repository:

```bash
aws ecr describe-images \
  --repository-name nodejs-ecs-app \
  --region ap-south-1
```

The output listed the `v1` image tag, digest, and push timestamp.

#### Subtask 6 — Set up an ECS cluster and Task Definition

Created a Fargate-type cluster:

```bash
aws ecs create-cluster --cluster-name nodejs-fargate-cluster
```

Defined a Task Definition (`task-def.json`) referencing the ECR image:

```json
{
  "family": "nodejs-ecs-app-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::<account_id>:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "nodejs-ecs-app",
      "image": "<account_id>.dkr.ecr.ap-south-1.amazonaws.com/nodejs-ecs-app:v1",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/nodejs-ecs-app",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

Created the CloudWatch log group and registered the task definition:

```bash
aws logs create-log-group --log-group-name /ecs/nodejs-ecs-app --region ap-south-1

aws ecs register-task-definition \
  --cli-input-json file://task-def.json \
  --region ap-south-1
```

#### Subtask 7 — Run the Task Definition on Fargate

Ran the task as an ECS Service (for restart/self-healing behavior) inside a default VPC subnet, with a security group allowing inbound traffic on port 3000:

```bash
aws ec2 create-security-group \
  --group-name nodejs-ecs-sg \
  --description "Allow inbound 3000 for Node.js ECS app" \
  --vpc-id <vpc_id>

aws ec2 authorize-security-group-ingress \
  --group-id <sg_id> \
  --protocol tcp --port 3000 --cidr 0.0.0.0/0

aws ecs create-service \
  --cluster nodejs-fargate-cluster \
  --service-name nodejs-ecs-service \
  --task-definition nodejs-ecs-app-task \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[<subnet_id>],securityGroups=[<sg_id>],assignPublicIp=ENABLED}"
```

Checked service and task status:

```bash
aws ecs describe-services \
  --cluster nodejs-fargate-cluster \
  --services nodejs-ecs-service

aws ecs list-tasks --cluster nodejs-fargate-cluster
```

Once the task reached `RUNNING`, retrieved its public IP via the attached ENI:

```bash
aws ecs describe-tasks --cluster nodejs-fargate-cluster --tasks <task_id>
# noted the eni-id from the network interface attachment

aws ec2 describe-network-interfaces --network-interface-ids <eni_id>
# read the public IP from the result
```

Accessed the app in a browser at `http://<public_ip>:3000` and confirmed the Node.js application responded correctly, running fully serverlessly on Fargate.

---

## ❖ Validation Checklist

- [x] `aws sts get-caller-identity` confirms CLI authentication
- [x] ECR repository created and visible in the AWS Console / `aws ecr describe-repositories`
- [x] Image tag `v1` present in ECR (`aws ecr describe-images`)
- [x] Task Definition registered (`aws ecs describe-task-definition --task-definition nodejs-ecs-app-task`)
- [x] ECS Service shows `desiredCount` == `runningCount`
- [x] Application reachable over HTTP via the task's public IP on port 3000
- [x] CloudWatch Logs group `/ecs/nodejs-ecs-app` receiving container stdout/stderr

---

## ❖ Deliverables

- [x] **GitHub repository link** for the Node.js application (with included Dockerfile)
- [x] **Dockerfile** used to build the image
- [x] **AWS CLI setup and authentication** — logs from `aws configure` / `aws sts get-caller-identity`
- [x] **ECR tag and push evidence** — `docker tag`, `docker push`, and `aws ecr describe-images` output
- [x] **ECS Task Definition configuration** — `task-def.json`, family `nodejs-ecs-app-task`, Fargate launch type, 256 CPU / 512 MB memory
- [x] **Evidence of the running application** — public endpoint `http://<public_ip>:3000` confirmed reachable in browser

---

## ❖ Notes / Learnings

- Fargate removes the need to provision or patch EC2 instances for the cluster — the Task Definition's `cpu`/`memory` values are the only capacity planning required at this level.
- `awsvpc` network mode gives each task its own ENI and private/public IP, which is why the running task's IP had to be looked up through the ENI rather than the cluster or service itself.
- Running as an ECS **Service** (rather than a one-off `run-task`) was deliberate — it gives ECS a desired count to reconcile against, so a failed task is automatically replaced, mirroring the self-healing behavior explored with Kubernetes on Day 81.
- The security group step is easy to miss and is the most common reason the public IP is unreachable after a task reports `RUNNING`.
- This was completed as a **self-study lab exercise** against a personal AWS account within the free-tier/low-cost range (single Fargate task, minimal CPU/memory) — documented here as practice toward the ECS/ECR skill areas of the curriculum, not a production deployment.

---

## ❖ Cleanup

To avoid ongoing charges after validation:

```bash
aws ecs update-service --cluster nodejs-fargate-cluster --service nodejs-ecs-service --desired-count 0
aws ecs delete-service --cluster nodejs-fargate-cluster --service nodejs-ecs-service
aws ecs delete-cluster --cluster nodejs-fargate-cluster
aws ecr delete-repository --repository-name nodejs-ecs-app --force
```

## ❖ References

- AWS ECS documentation (Fargate launch type)
- AWS ECR documentation
- AWS CLI v2 documentation
