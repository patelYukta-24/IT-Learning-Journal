# Day 65 — Scaling with Terraform

## 📌 Overview

Day 65 covers **scaling** infrastructure with Terraform using AWS **Auto Scaling Groups (ASGs)** — automatically adding or removing EC2 instances to match application demand. The exercises walk through creating an ASG via Terraform and then testing scale-out/scale-in behavior by adjusting desired capacity in the AWS Console.

---

## Understanding Scaling

Scaling is the process of adding or removing resources to match the changing demands of an application. As an application grows, more resources are needed to handle increased load; as load decreases, extra resources can be removed to save costs.

Terraform makes scaling straightforward through its declarative model — the desired number of resources is defined in configuration, and Terraform (or, for ASGs, AWS itself based on the ASG's settings) creates or destroys resources to match.

---

## Tasks / Exercises

### 1. Create an Auto Scaling Group

Auto Scaling Groups automatically add or remove EC2 instances based on current demand, keeping the running instance count between a defined minimum and maximum, targeting a desired capacity.

#### Prerequisite: Launch Template

An ASG needs a template describing what each instance should look like:

**`main.tf`**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

resource "aws_launch_template" "day65_lt" {
  name_prefix   = "day65-lt-"
  image_id      = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  vpc_security_group_ids = [aws_security_group.day65_sg.id]

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "day65-asg-instance"
    }
  }
}

resource "aws_security_group" "day65_sg" {
  name        = "day65-allow-http-ssh"
  description = "Allow HTTP and SSH for ASG instances"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**`variables.tf`**
```hcl
variable "ami_id" {
  description = "AMI ID to use for the launch template"
  type        = string
  default     = "ami-0c2af51e265bd5e0e"  # replace with a current AMI for your region
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "key_name" {
  description = "Existing EC2 key pair name"
  type        = string
  default     = "day65-keypair"
}

variable "subnet_ids" {
  description = "List of subnet IDs for the ASG to launch instances into"
  type        = list(string)
  default     = []  # populate with your VPC's subnet IDs
}
```

#### Auto Scaling Group resource

```hcl
resource "aws_autoscaling_group" "day65_asg" {
  name                = "day65-asg"
  desired_capacity    = 1
  min_size            = 1
  max_size            = 3
  vpc_zone_identifier = var.subnet_ids

  launch_template {
    id      = aws_launch_template.day65_lt.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "day65-asg-instance"
    propagate_at_launch = true
  }
}
```

Run:
```bash
terraform init
terraform validate
terraform plan
terraform apply
```

This creates an ASG with `min_size = 1`, `max_size = 3`, starting at `desired_capacity = 1` — one instance running initially, with room to scale up to three.

---

### 2. Test Scaling

Manual scale-out and scale-in test, performed via the AWS Console:

1. Opened the **AWS Management Console → EC2 → Auto Scaling Groups**
2. Selected the `day65-asg` group created above and clicked **Edit**
3. Increased **Desired Capacity** from `1` to `3` and clicked **Save**
4. Waited a few minutes for the new instances to launch
5. Went to **EC2 → Instances** and confirmed **3 instances** were now running, tagged `day65-asg-instance`
6. Returned to the Auto Scaling Group, decreased **Desired Capacity** back to `1`, and saved
7. Waited a few minutes for the extra instances to be terminated
8. Went back to **EC2 → Instances** and confirmed only **1 instance** remained running — the ASG had automatically terminated the other two

> Since the console edit changes the ASG's desired capacity outside of Terraform, running `terraform plan` afterward would show drift (Terraform's recorded `desired_capacity = 1` vs. the console's temporary `3`). Running `terraform apply` again reconciles it back to what's declared in `main.tf`, or the `.tf` file's `desired_capacity` can be updated to match if the change should be kept.

Tear down when finished:
```bash
terraform destroy
```

---

## 🧠 Key Learnings

- An Auto Scaling Group needs a **launch template** (or launch configuration) describing what each instance looks like, plus `min_size`, `max_size`, and `desired_capacity` to define its scaling bounds
- Scaling can be triggered manually (as in this exercise, via the console) or automatically via scaling policies tied to metrics like CPU utilization — this exercise covered the manual case as a first step
- Changes made directly in the AWS Console (like editing desired capacity) cause **state drift** from what's declared in Terraform — `terraform plan` will surface this, and `apply` re-asserts whatever the `.tf` files declare
- `min_size`/`max_size` set the safe operating bounds so a manual or automatic capacity change can't accidentally scale infrastructure (and cost) out of control
- Terminating an ASG's instances is handled by AWS itself based on capacity settings — not by manually stopping EC2 instances one by one

## ⚠️ Challenges Faced

- Understanding that editing Desired Capacity in the console creates drift from the Terraform-managed state, and that a follow-up `terraform apply` will reset it unless the `.tf` file is updated to match
- Making sure the launch template's security group allowed the traffic needed to actually verify the scaled instances were reachable

## 🔗 Resources

- Terraform `aws_autoscaling_group` resource documentation
- Terraform `aws_launch_template` resource documentation
- AWS documentation — Auto Scaling Groups

---
