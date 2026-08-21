# Day 63 — Working with Terraform Resources

## 📌 Overview

Day 63 goes deeper into **Terraform resources**, building on Day 62's EC2 provisioning by adding a **security group** so the instance can actually be reached over the network. This day combines two resources — `aws_security_group` and `aws_instance` — into a single working configuration that provisions network access rules alongside the EC2 instance itself.

---

## Understanding Terraform Resources

A resource in Terraform represents a component of infrastructure — a physical server, a virtual machine, a DNS record, an S3 bucket, and so on. Resources have **attributes** that define their properties and behavior, such as the size and location of a virtual machine or the domain name of a DNS record.

When defining a resource, three things are specified:
1. The **resource type** (e.g. `aws_instance`, `aws_security_group`)
2. A **unique name** for the resource (used to reference it elsewhere in the config)
3. The **attributes** that configure the resource

Terraform uses the `resource` block to define resources in configuration:
```hcl
resource "<TYPE>" "<NAME>" {
  attribute_1 = value
  attribute_2 = value
}
```

---

## Tasks / Exercises

### 1. Create a security group

To allow traffic to the EC2 instance (SSH access and HTTP web traffic), a security group is created first.

**`main.tf`** — security group resource:
```hcl
resource "aws_security_group" "day63_sg" {
  name        = "day63-allow-ssh-http"
  description = "Allow SSH and HTTP inbound traffic"

  ingress {
    description = "Allow SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "day63-allow-ssh-http"
  }
}
```

Run:
```bash
terraform init
terraform apply
```

> Note: `0.0.0.0/0` on SSH (port 22) is used here for lab/demo purposes. In a real environment, SSH ingress should be restricted to a known IP range rather than the entire internet.

---

### 2. Create an EC2 instance (using the security group)

With the security group created, an EC2 instance is provisioned and attached to it.

**`variables.tf`**
```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance (replace with a current AMI for your region)"
  type        = string
  default     = "ami-0c2af51e265bd5e0e"  # Ubuntu 22.04 LTS, ap-south-1 — replace as needed
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "key_name" {
  description = "Name of an existing EC2 key pair (replace with your own key pair name)"
  type        = string
  default     = "day63-keypair"
}
```

**`main.tf`** — EC2 instance resource, referencing the security group:
```hcl
resource "aws_instance" "day63_ec2" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  key_name               = var.key_name
  vpc_security_group_ids = [aws_security_group.day63_sg.id]

  tags = {
    Name = "day63-terraform-ec2"
  }
}
```

**`outputs.tf`**
```hcl
output "instance_public_ip" {
  value = aws_instance.day63_ec2.public_ip
}

output "security_group_id" {
  value = aws_security_group.day63_sg.id
}
```

> **NOTE:** Replace `ami_id` and `key_name` with values valid for your own AWS account and region — AMI IDs are region-specific, and the key pair must already exist in your account. Available AMIs can be found in the AWS documentation or the EC2 console's "Launch Instance" AMI catalog.

Run:
```bash
terraform apply
```

Verify SSH access using the security group's allowed port:
```bash
ssh -i day63-keypair.pem ubuntu@<instance_public_ip>
```

Verify HTTP reachability (once a web server like Nginx is installed on the instance):
```bash
curl http://<instance_public_ip>
```

Tear down when finished:
```bash
terraform destroy
```

---

## 🧠 Key Learnings

- Resources in Terraform can reference each other by their `type.name` identifier — here, `aws_instance` references `aws_security_group.day63_sg.id`, and Terraform automatically creates the security group before the instance that depends on it
- A security group's `ingress`/`egress` blocks define the actual network rules — without one, an EC2 instance is unreachable even if it's running
- AMI IDs and key pair names are environment-specific and must be replaced per AWS account/region — hardcoded defaults from a tutorial will not work as-is
- Restricting SSH ingress to `0.0.0.0/0` is a common lab shortcut but a real security risk outside of a learning environment

## ⚠️ Challenges Faced

- Making sure the security group was fully created and referenced correctly before the EC2 instance's `apply` ran (Terraform handled the ordering automatically via the reference, but it needed to be understood why)
- Finding a valid, current AMI ID for the target region — AMI IDs change over time and differ by region

## 🔗 Resources

- Terraform `aws_security_group` resource documentation
- Terraform `aws_instance` resource documentation
- AWS documentation — finding available AMIs

---
