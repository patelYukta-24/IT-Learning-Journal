# Day 49: Setting up an Application Load Balancer with AWS EC2

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. What is Load Balancing?](#1-what-is-load-balancing)
- [2. Elastic Load Balancing (ELB)](#2-elastic-load-balancing-elb)
- [3. Tasks](#3-tasks)
- [4. Troubleshooting Notes](#4-troubleshooting-notes)
- [5. Key Learnings](#5-key-learnings)
- [6. Resources](#6-resources)

---

## Overview
Day 49 follows on from Launch Templates and multi-instance launches (Day 48) by introducing **Load Balancing** — the piece that actually makes multiple instances useful as a single, resilient application. Today's work is about launching two independent Apache web servers and putting an Application Load Balancer (ALB) in front of them so traffic is distributed automatically and health is monitored continuously.

## Topics Covered
- ❖ What is Load Balancing
- ❖ Elastic Load Balancing (ELB) — ALB, NLB, CLB
- ❖ Tasks — 2× EC2 with Apache via User Data, ALB setup, target group health verification

---

## 1. What is Load Balancing?
Load balancing is the practice of distributing incoming traffic/workload across multiple servers instead of relying on a single one. It matters for two closely related reasons:

- **Reliability** — if one server fails or becomes unhealthy, traffic is automatically routed to the remaining healthy servers instead of the application going down
- **Performance** — no single instance gets overwhelmed; requests are spread out so resource utilization stays even across the fleet

This is what allows an application to scale horizontally (more instances) rather than only vertically (a bigger instance) — and it pairs naturally with the Launch Templates and Auto Scaling Groups from Day 48, since a load balancer is what a scaling group's instances typically sit behind.

---

## 2. Elastic Load Balancing (ELB)
Elastic Load Balancing is AWS's managed service for automatically distributing incoming traffic across multiple EC2 instances (and other targets). AWS offers three types, differing mainly by which OSI layer they operate at:

| Load Balancer | OSI Layer | Best for | Key features |
|---|---|---|---|
| **Application Load Balancer (ALB)** | Layer 7 (HTTP/HTTPS) | Web apps, microservices, advanced routing | Path/host-based routing, supports containers & Lambda targets, WebSocket support |
| **Network Load Balancer (NLB)** | Layer 4 (TCP/UDP) | High-throughput, low-latency, extreme performance | Handles millions of requests/sec, preserves client source IP, static IP support |
| **Classic Load Balancer (CLB)** | Layer 4 (with limited Layer 7) | Legacy applications built on the EC2-Classic network | Basic load balancing only — AWS recommends ALB/NLB for new applications |

### 2.1 Why ALB Fits Today's Task
Since the task is HTTP-based (Apache serving `index.html`), an **Application Load Balancer** is the right fit — it can inspect HTTP requests, route based on content, and reports per-target health checks that are easy to verify from the console.

### 2.2 Core ALB Concepts
| Concept | What it does |
|---|---|
| **Load Balancer** | The entry point that receives client traffic |
| **Listener** | Checks for connection requests using a protocol/port (e.g., HTTP:80) you configure |
| **Target Group** | A set of registered targets (EC2 instances) that the listener routes requests to |
| **Health Check** | Periodic requests (e.g., `GET /`) sent to each target to confirm it's responding correctly before routing live traffic to it |

---

## 3. Tasks

### Task 1 — Launch 2 EC2 Instances (Ubuntu) with Apache via User Data

**Objective:** Launch two Ubuntu EC2 instances, install Apache via User Data, and customize each instance's `index.html` — instance 1 shows your name, instance 2 shows "TrainWithShubham Community is Super Awesome :)".

**User Data script — Instance 1 (`apache-userdata-instance1.sh`):**
```bash
#!/bin/bash
apt-get update -y
apt-get install -y apache2
systemctl enable apache2
systemctl start apache2

cat <<'EOF' > /var/www/html/index.html
<html>
  <head><title>Instance 1</title></head>
  <body>
    <h1>Hello, this is Yukta's Apache Server (Instance 1)</h1>
  </body>
</html>
EOF
```

**User Data script — Instance 2 (`apache-userdata-instance2.sh`):**
```bash
#!/bin/bash
apt-get update -y
apt-get install -y apache2
systemctl enable apache2
systemctl start apache2

cat <<'EOF' > /var/www/html/index.html
<html>
  <head><title>Instance 2</title></head>
  <body>
    <h1>TrainWithShubham Community is Super Awesome :)</h1>
  </body>
</html>
EOF
```

**Step-by-step (console):**
1. EC2 → Launch Instance → AMI: **Ubuntu Server 22.04 LTS** (Free Tier eligible)
2. Instance type: `t2.micro`
3. Key pair: select/create one
4. Security group: allow inbound `22` (SSH) and `80` (HTTP) from your IP (or `0.0.0.0/0` for testing HTTP)
5. Advanced details → User data → paste Instance 1's script
6. Launch → repeat the process for Instance 2 using its own script
7. Once both are running, copy each instance's **Public IPv4 address** from the EC2 console

**Equivalent via AWS CLI:**
```bash
# Instance 1
aws ec2 run-instances \
  --image-id ami-0xxxxxxxxxxxxxxxx \
  --instance-type t2.micro \
  --key-name my-keypair \
  --security-group-ids sg-xxxxxxxx \
  --user-data file://apache-userdata-instance1.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Day49-Apache-Instance1}]'

# Instance 2
aws ec2 run-instances \
  --image-id ami-0xxxxxxxxxxxxxxxx \
  --instance-type t2.micro \
  --key-name my-keypair \
  --security-group-ids sg-xxxxxxxx \
  --user-data file://apache-userdata-instance2.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Day49-Apache-Instance2}]'
```

**Verification:**
1. Paste each instance's public IP into a browser (`http://<public-ip>`)
2. Confirm Instance 1 shows the custom name page, Instance 2 shows the TrainWithShubham message
3. Note: Ubuntu AMIs use `apt`/`apache2`, unlike the `yum`/`httpd` pattern on Amazon Linux — worth remembering since the curriculum has used Amazon Linux 2 in prior days

> Status: reference implementation — pending execution + screenshots of both instances' pages in-browser.

---

### Task 2 — Create an Application Load Balancer and Attach the Instances

**Objective:** Create an ALB via the AWS Console, register both Task 1 instances into a target group, and verify health + load balancing behavior.

**Step-by-step (console):**
1. **Target Group first** (ALB needs one to route to):
   - EC2 → Target Groups → Create target group
   - Target type: **Instances**
   - Protocol/Port: **HTTP : 80**
   - VPC: same VPC as your two instances
   - Health check path: `/` (default)
   - Next → select both Instance 1 and Instance 2 → Include as pending below → Create target group

2. **Create the ALB:**
   - EC2 → Load Balancers → Create Load Balancer → **Application Load Balancer**
   - Name: `day49-web-alb`
   - Scheme: Internet-facing
   - Listeners: HTTP : 80
   - VPC + at least **2 Availability Zones/subnets** (ALB requires ≥2 AZs)
   - Security group: allow inbound `80` from `0.0.0.0/0`
   - Listener → default action → forward to the target group created in step 1
   - Create load balancer

3. **Verify:**
   - EC2 → Target Groups → select your group → **Targets** tab → confirm both instances show **healthy** status
   - Copy the ALB's **DNS name** (EC2 → Load Balancers → your ALB → Description)
   - Paste the DNS name into a browser and refresh several times — you should see the response alternate between Instance 1's and Instance 2's page, confirming traffic is being distributed

**Equivalent via AWS CLI:**
```bash
# Create target group
aws elbv2 create-target-group \
  --name day49-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-xxxxxxxx \
  --health-check-path /

# Register both instances
aws elbv2 register-targets \
  --target-group-arn <target-group-arn> \
  --targets Id=<instance-1-id> Id=<instance-2-id>

# Create the ALB
aws elbv2 create-load-balancer \
  --name day49-web-alb \
  --subnets subnet-aaaaaaa subnet-bbbbbbb \
  --security-groups sg-xxxxxxxx \
  --scheme internet-facing \
  --type application

# Create listener forwarding to the target group
aws elbv2 create-listener \
  --load-balancer-arn <alb-arn> \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=<target-group-arn>
```

**Verification via CLI:**
```bash
# Check target health
aws elbv2 describe-target-health --target-group-arn <target-group-arn>

# Get ALB DNS name
aws elbv2 describe-load-balancers --names day49-web-alb \
  --query "LoadBalancers[0].DNSName" --output text
```
Then repeatedly `curl` the DNS name to observe the response alternating between instances:
```bash
for i in {1..6}; do curl -s http://<alb-dns-name> | grep -i "h1"; done
```

> Status: reference implementation — pending execution + screenshots of target group health status and the alternating browser responses.

---

## 4. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| Target shows "unhealthy" | Security group on instance doesn't allow inbound HTTP from the ALB's security group | Update instance SG to allow port 80 from the ALB's SG (not just your IP) |
| ALB creation fails / greyed out AZ option | Fewer than 2 subnets selected | ALB requires at least 2 AZs — select 2+ subnets in different AZs |
| Browser only ever shows one instance's page | Browser/keep-alive caching the connection to one target | Use `curl` in a loop, or a private/incognito window with cache disabled, to see true round-robin behavior |
| `apache2` not found / install fails | Used a `yum`/Amazon Linux command on an Ubuntu instance | Ubuntu uses `apt-get` and the package name `apache2`, not `httpd` |
| Target group health check fails but page loads fine in browser | Health check path/port misconfigured | Confirm target group health check path is `/` and port matches 80 |

---

## 5. Key Learnings
- Load balancing turns multiple independent servers into one resilient, distributable application — this is the mechanism that makes horizontal scaling (from Day 48's Auto Scaling Group) actually useful in practice.
- An ALB's Target Group is where health checks live — "healthy" targets are what actually receive traffic, so this is the first place to check when something isn't routing as expected.
- Security group rules need to allow traffic **from the ALB to the instance**, not just from your own IP — a common gap when moving from single-instance testing to a load-balanced setup.
- ALB (Layer 7) is the right choice for HTTP applications needing content-based routing; NLB (Layer 4) would be the pick instead for raw TCP/UDP throughput and latency-sensitive workloads.

## 6. Resources
- [Elastic Load Balancing — AWS Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html)
- [Application Load Balancer — AWS Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [Network Load Balancer — AWS Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)
- [Classic Load Balancer — AWS Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html)

---
