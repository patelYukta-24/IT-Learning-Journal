# Day 48: AWS EC2 Automation

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. Automation in EC2](#1-automation-in-ec2)
- [2. Launch Templates in AWS EC2](#2-launch-templates-in-aws-ec2)
- [3. Instance Types](#3-instance-types)
- [4. AMI (Amazon Machine Image)](#4-ami-amazon-machine-image)
- [5. Tasks](#5-tasks)
- [6. Troubleshooting Notes](#6-troubleshooting-notes)
- [7. Key Learnings](#7-key-learnings)
- [8. Resources](#8-resources)

---

## Overview
Day 48 builds directly on Day 47's IAM and User Data foundations by moving into **repeatable EC2 automation** — Launch Templates, Instance Types, AMIs, and (as a stretch goal) Auto Scaling Groups. The core idea: once you know how to configure an instance correctly once, you shouldn't have to re-type that configuration every time you launch a new one.

## Topics Covered
- ❖ Automation in EC2
- ❖ Launch Templates
- ❖ Instance Types
- ❖ AMI (Amazon Machine Image)
- ❖ Tasks — Launch Template with Jenkins/Docker, multi-instance launch, Auto Scaling Group (stretch)

---

## 1. Automation in EC2
EC2 (Elastic Compute Cloud) provides secure, reliable, and cost-effective compute infrastructure. What makes it powerful for DevOps workflows specifically is that almost every part of launching and configuring an instance can be **automated and made repeatable**:
- **User Data** (Day 47) automates what happens *inside* the instance after boot
- **Launch Templates** (this day) automate *how* the instance itself is configured and launched
- **Auto Scaling Groups** automate *how many* instances exist, scaling up/down based on demand

Together these turn "manually click through the Launch Instance wizard every time" into a defined, versioned, reusable configuration — a first step toward Infrastructure as Code thinking, even before tools like Terraform enter the picture.

---

## 2. Launch Templates in AWS EC2
A **Launch Template** stores the configuration needed to launch an instance — AMI ID, instance type, key pair, security groups, User Data script, storage, network settings, and more — so it doesn't need to be re-entered every time.

### 2.1 Why Use a Launch Template
- **Consistency** — every instance launched from the same template is configured identically, avoiding "it works on my instance" drift
- **Speed** — launching a new instance (or several) becomes a few clicks/one CLI command
- **Versioning** — templates support multiple versions, so you can update the configuration (e.g., a newer AMI) without losing the old version
- **Required by Auto Scaling Groups** — ASGs need either a Launch Template or a (now-legacy) Launch Configuration to know what to launch

### 2.2 Launch Template vs. Launch Configuration
| | Launch Template | Launch Configuration (legacy) |
|---|---|---|
| Versioning | Yes — multiple versions | No |
| Instance types (mixed) | Supports multiple types/purchase options (On-Demand + Spot) | Single instance type only |
| AWS recommendation | ✅ Current best practice | ⚠️ Being phased out |

---

## 3. Instance Types
EC2 offers many instance types, each optimized for a different balance of CPU, memory, storage, and network performance. Choosing the right type/size lets you match cost to actual workload needs instead of over- or under-provisioning.

### 3.1 Instance Type Families (high level)
| Family | Optimized for | Example use case |
|---|---|---|
| **T (T2/T3/T4g)** | Burstable, low-cost general purpose | Free Tier learning, dev/test, low-traffic web apps |
| **M (M5/M6i)** | Balanced compute/memory | General-purpose application servers |
| **C (C5/C6i)** | Compute-optimized | CPU-heavy workloads, batch processing, CI runners |
| **R (R5/R6i)** | Memory-optimized | In-memory caches, large databases |
| **I (I3)** | Storage-optimized (high IOPS) | NoSQL databases, data warehousing |
| **G/P (G5/P4)** | GPU-accelerated | ML training/inference, graphics rendering |

### 3.2 Reading an Instance Type Name
`t2.micro` breaks down as:
- `t` = family (burstable, general purpose)
- `2` = generation
- `micro` = size (within that family — e.g., nano, micro, small, medium, large...)

For this curriculum's Free Tier work, `t2.micro` (or `t3.micro` depending on region/account) is the go-to: enough resources for Jenkins/Docker experimentation, and Free Tier eligible for 750 hrs/month.

---

## 4. AMI (Amazon Machine Image)
An **AMI** is a pre-configured template used to launch an instance — it packages the OS, any pre-installed software, and launch permissions.

### 4.1 Types of AMIs
- **AWS-provided/quick-start AMIs** — e.g., Amazon Linux 2, Ubuntu, Windows Server — maintained and patched by AWS
- **AWS Marketplace AMIs** — pre-built by third parties (e.g., a pre-configured Jenkins or database AMI)
- **Custom/Community AMIs** — created from your own configured instance (via a snapshot) — this is how you'd "bake" Jenkins + Docker directly into an image instead of installing them via User Data every time

### 4.2 Why AMIs Matter for Automation
When you need multiple identical instances, you launch them all from one AMI instead of manually configuring each one — combined with a Launch Template, this means "launch 5 identical, pre-configured servers" becomes a single action.

---

## 5. Tasks

### Task 1 — Launch Template: Amazon Linux 2 + t2.micro + Jenkins/Docker User Data

**Objective:** Create a Launch Template using Amazon Linux 2 AMI, `t2.micro` instance type, and the Jenkins/Docker installation script (from Day 39/47's User Data).

**Reused User Data script (`jenkins-docker-userdata.sh`):**
```bash
#!/bin/bash
set -e

# --- System update ---
sudo yum update -y

# --- Install Docker ---
sudo yum install -y docker
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user

# --- Install Jenkins (requires Java) ---
sudo wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install -y fontconfig java-17-openjdk jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

**Step-by-step (console):**
1. EC2 → Launch Templates → Create launch template
2. **Name:** `jenkins-docker-template`
3. **AMI:** Amazon Linux 2 (search "Amazon Linux 2 AMI" and pick the latest HVM/x86_64 or arm64 version)
4. **Instance type:** `t2.micro`
5. **Key pair:** select or create one (needed if you want SSH access later)
6. **Security group:** allow inbound `22` (SSH) and `8080` (Jenkins) from your IP
7. **Advanced details → User data:** paste the script above
8. Create template

**Equivalent via AWS CLI:**
```bash
aws ec2 create-launch-template \
  --launch-template-name jenkins-docker-template \
  --version-description "Amazon Linux 2, t2.micro, Jenkins+Docker" \
  --launch-template-data '{
    "ImageId": "ami-0c101f26f147fa7fd",
    "InstanceType": "t2.micro",
    "KeyName": "my-keypair",
    "SecurityGroupIds": ["sg-xxxxxxxx"],
    "UserData": "'"$(base64 -w0 jenkins-docker-userdata.sh)"'"
  }'
```
> Note: `ImageId` should be the current Amazon Linux 2 AMI ID for your region — verify with `aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2"`.

> Status: reference implementation — pending execution + console screenshot of the created template.

---

### Task 2 — Launch 3 Instances from the Launch Template

**Objective:** Use the Launch Template to launch 3 identical instances in one action, and locate the "number of instances" option.

**Where to find it (console):**
1. EC2 → Launch Templates → select `jenkins-docker-template` → **Actions → Launch instance from template**
2. On the launch screen, under **Number of instances**, change the value from `1` to `3`
   > This is the field the task is pointing at — it's directly on the "Launch an instance from template" page, right below the template summary.
3. Review settings (they're pre-filled from the template) → Launch instance

**Equivalent via AWS CLI:**
```bash
aws ec2 run-instances \
  --launch-template LaunchTemplateName=jenkins-docker-template,Version='$Latest' \
  --min-count 3 \
  --max-count 3 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Day48-JenkinsDocker}]'
```
`--min-count` / `--max-count` set to `3` is the CLI equivalent of the console's "Number of instances" field.

**Verification:**
```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=Day48-JenkinsDocker" \
  --query "Reservations[].Instances[].[InstanceId,State.Name,PublicIpAddress]" \
  --output table
```
> Status: reference implementation — pending execution + screenshot showing 3 running instances.

---

### Task 3 (Stretch Goal) — Auto Scaling Group

**Objective:** Go one step further and create an Auto Scaling Group (ASG) using the same Launch Template.

**What an ASG adds on top of a Launch Template:**
- Maintains a **desired number of instances** automatically — replaces any that fail health checks
- Can **scale out/in** based on demand (CPU utilization, request count, or a schedule)
- Spreads instances across multiple Availability Zones for resilience

**Step-by-step (console):**
1. EC2 → Auto Scaling Groups → Create Auto Scaling group
2. Name: `jenkins-docker-asg`
3. Launch template: select `jenkins-docker-template`
4. VPC + at least 2 subnets across different AZs (for high availability)
5. (Optional) Attach to a Load Balancer target group if fronting Jenkins with an ALB
6. **Group size:** Desired = `2`, Minimum = `1`, Maximum = `3`
7. **Scaling policy:** Target tracking → e.g., keep average CPU at 50%
8. Review and create

**Equivalent via AWS CLI:**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name jenkins-docker-asg \
  --launch-template LaunchTemplateName=jenkins-docker-template,Version='$Latest' \
  --min-size 1 \
  --max-size 3 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-aaaaaaa,subnet-bbbbbbb"

aws autoscaling put-scaling-policy \
  --auto-scaling-group-name jenkins-docker-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }'
```

**Verification:**
```bash
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names jenkins-docker-asg \
  --query "AutoScalingGroups[].[AutoScalingGroupName,DesiredCapacity,MinSize,MaxSize]" \
  --output table
```
> Status: stretch goal — reference implementation only, pending hands-on execution and screenshots.

---

## 6. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| Launch template launch fails silently | AMI ID doesn't exist in the selected region | Re-verify AMI ID per-region with `describe-images` |
| Only 1 instance launches despite requesting 3 | Used console default without changing "Number of instances", or CLI `--count` used instead of `--min-count`/`--max-count` | Explicitly set the field/flags as shown above |
| User Data didn't run on templated instances | Base64 encoding issue when passing via CLI `--launch-template-data` | Use `base64 -w0` (no line wraps) when embedding the script |
| ASG keeps terminating/relaunching instances | Health check failing (e.g., Jenkins not up within health check grace period) | Increase `--health-check-grace-period`, verify app comes up correctly |
| ASG instances not spread across AZs | Only one subnet provided | Pass at least 2 subnets in different AZs via `--vpc-zone-identifier` |

---

## 7. Key Learnings
- Launch Templates decouple "what to launch" from "how many/when" — this separation is exactly what makes Auto Scaling Groups possible.
- The "Number of instances" field (console) / `--min-count`/`--max-count` (CLI) is how you launch a fleet in a single action instead of repeating the process manually.
- AMIs let you "bake in" configuration (like Jenkins/Docker) permanently, as an alternative or complement to running a User Data script on every boot — worth comparing trade-offs (image build/maintenance time vs. per-boot install time).
- Auto Scaling Groups are the natural next step after Launch Templates — they turn a fixed fleet into an elastic one that reacts to real demand.

## 8. Resources
- [Automation in EC2 — AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instances-and-amis.html)
- [Launch Templates — AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html)
- [Instance Types — AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html)
- [Amazon Machine Images (AMI) — AWS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [Auto Scaling Groups — AWS Documentation](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html)

---
