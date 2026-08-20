# Day 52: All About ECS

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. What is ECS?](#1-what-is-ecs)
- [2. Difference between EKS and ECS](#2-difference-between-eks-and-ecs)
- [3. Task](#3-task)
- [4. Troubleshooting Notes](#4-troubleshooting-notes)
- [5. Key Learnings](#5-key-learnings)
- [6. Resources](#6-resources)

---

## Overview
Day 52 moves from raw EC2 automation into **container orchestration on AWS** — specifically ECS (Elastic Container Service). The task is to deploy an Nginx container on ECS, which involves understanding a new set of building blocks: clusters, task definitions, services, and launch types (Fargate vs. EC2).

## Topics Covered
- ❖ What is ECS
- ❖ EKS vs. ECS
- ❖ Task — deploy Nginx on ECS

---

## 1. What is ECS?
Amazon ECS (Elastic Container Service) is a fully managed container orchestration service that runs and manages Docker containers on a cluster, without requiring manual management of the underlying orchestration layer. It supports two launch types:

| Launch Type | Description |
|---|---|
| **Fargate** | Serverless — AWS manages the underlying compute; you only define the container and its resource needs | 
| **EC2** | You provide and manage the EC2 instances that form the cluster; more control, more operational overhead |

ECS integrates natively with other AWS services — Elastic Load Balancing, Auto Scaling, and VPC — making it straightforward to build scalable, highly available containerized applications entirely within the AWS ecosystem. It also supports Docker Compose-style workflows, easing adoption for teams already using Docker locally.

### 1.1 Core ECS Concepts
| Concept | Description |
|---|---|
| **Cluster** | A logical grouping of tasks/services — the "environment" containers run in |
| **Task Definition** | A blueprint (JSON) describing one or more containers — image, CPU/memory, ports, environment variables |
| **Task** | A running instance of a task definition |
| **Service** | Ensures a specified number of tasks are running and healthy at all times, and can integrate with a load balancer |
| **Container Registry** | Where container images are pulled from — typically Amazon ECR, though public registries like Docker Hub also work |

---

## 2. Difference between EKS and ECS
Both EKS (Elastic Kubernetes Service) and ECS are AWS container orchestration platforms, but they differ significantly:

| Dimension | ECS | EKS |
|---|---|---|
| **Architecture** | Centralized control plane managing scheduling on EC2 (or Fargate) | Distributed Kubernetes control plane across multiple nodes |
| **Kubernetes support** | No — ECS has its own proprietary orchestration engine | Yes — fully managed native Kubernetes |
| **Scaling** | Manual configuration of scaling policies for tasks/services | More automatic, Kubernetes-native scaling behavior |
| **Flexibility** | More restrictive/opinionated, AWS-specific | Highly customizable, standard Kubernetes API and ecosystem |
| **Community/Ecosystem** | Smaller, AWS-driven | Large, active open-source Kubernetes community |
| **Learning curve** | Simpler to get started with if you're AWS-native | Steeper, but skills transfer across any Kubernetes environment (not just AWS) |

**Practical takeaway:** ECS is a good fit when staying fully within the AWS ecosystem is fine and simplicity is preferred. EKS makes more sense when Kubernetes portability (multi-cloud, on-prem, hybrid) or the broader Kubernetes tooling ecosystem matters.

---

## 3. Task

### Task — Set up ECS with Nginx

**Objective:** Deploy an Nginx container on ECS and confirm it's reachable.

This walkthrough uses the **Fargate** launch type (serverless — no EC2 instances to manage), which is the simplest path for a first ECS deployment.

**Step-by-step (console):**

1. **Create a Cluster**
   - ECS → Clusters → Create cluster
   - Cluster name: `day52-nginx-cluster`
   - Infrastructure: **AWS Fargate (serverless)**
   - Create

2. **Create a Task Definition**
   - ECS → Task definitions → Create new task definition
   - Family name: `nginx-task`
   - Launch type: **Fargate**
   - Task size: 0.5 vCPU / 1 GB memory (sufficient for a basic Nginx container)
   - Container details:
     - Container name: `nginx-container`
     - Image URI: `nginx:latest` (pulled from Docker Hub)
     - Container port: `80`, Protocol: TCP
   - Create

3. **Create a Service (runs the task and keeps it alive)**
   - Inside the cluster → Services tab → Create
   - Launch type: **Fargate**
   - Task definition: `nginx-task` (latest revision)
   - Service name: `nginx-service`
   - Desired tasks: `1`
   - Networking: select a VPC + public subnets
   - **Security group:** allow inbound `80` (HTTP) from your IP
   - **Auto-assign public IP: Enabled** (needed to reach the container directly, since there's no load balancer in this minimal setup)
   - Create

4. **Find the running task's public IP**
   - Cluster → Tasks tab → select the running task → **Networking** section → copy the **Public IP**

5. **Verify**
   - Browse to `http://<public-ip>` — the default Nginx welcome page ("Welcome to nginx!") should load

**Equivalent via AWS CLI:**
```bash
# Create cluster
aws ecs create-cluster --cluster-name day52-nginx-cluster

# Register task definition (nginx-task-def.json below)
aws ecs register-task-definition --cli-input-json file://nginx-task-def.json

# Create service
aws ecs create-service \
  --cluster day52-nginx-cluster \
  --service-name nginx-service \
  --task-definition nginx-task \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxxxxxx],securityGroups=[sg-xxxxxxxx],assignPublicIp=ENABLED}"
```

**`nginx-task-def.json`:**
```json
{
  "family": "nginx-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "nginx-container",
      "image": "nginx:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "essential": true
    }
  ]
}
```

**Verification via CLI:**
```bash
# Get the running task's ARN
aws ecs list-tasks --cluster day52-nginx-cluster --service-name nginx-service

# Describe the task to find its ENI / public IP
aws ecs describe-tasks --cluster day52-nginx-cluster --tasks <task-arn>
```
The public IP can be resolved from the task's attached Elastic Network Interface (ENI) details in the `describe-tasks` output.

> Status: reference implementation — pending execution + screenshot of the Nginx welcome page loading in-browser.

---

## 4. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| Task stuck in `PROVISIONING` / never reaches `RUNNING` | Subnet has no route to the internet, or image can't be pulled | Ensure the subnet is public (has an Internet Gateway route) and `assignPublicIp=ENABLED` |
| Can't reach Nginx in browser | Security group missing inbound port 80 | Add inbound rule for TCP 80 from your IP on the service's security group |
| Task fails immediately, `STOPPED` with exit code | Container port mismatch, or task lacks enough CPU/memory | Confirm `containerPort: 80` matches Nginx's default, and task size is sufficient |
| No public IP shown on the task | `assignPublicIp` was left disabled, or subnet is private | Re-create the service with `assignPublicIp=ENABLED` and a public subnet |
| `register-task-definition` fails | Malformed JSON, or `requiresCompatibilities` mismatched with launch type | Validate JSON syntax; confirm `FARGATE` is listed if using the Fargate launch type |

---

## 5. Key Learnings
- ECS's Fargate launch type removes the need to manage EC2 instances entirely — a meaningfully lower operational overhead than the EC2 launch type, at the cost of some control.
- A Task Definition is just a blueprint; a Service is what actually keeps the defined number of tasks running and can integrate with a load balancer for production use.
- ECS and EKS solve the same broad problem (container orchestration) with different trade-offs: ECS trades flexibility for simplicity and AWS-native integration; EKS trades a steeper learning curve for full Kubernetes compatibility and portability.
- For anything beyond a single-container demo, fronting the ECS service with an Application Load Balancer (rather than relying on a task's public IP directly) would be the more realistic next step — worth revisiting once comfortable with this basic setup.

## 6. Resources
- [Amazon ECS — Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [Amazon ECS on Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/what-is-fargate.html)
- [Amazon EKS — User Guide](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [ECS Task Definitions Reference](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)

---
