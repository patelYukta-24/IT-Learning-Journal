# Day 62 — Terraform with AWS

## 📌 Overview

Day 62 moves Terraform from local/Docker resources to real cloud infrastructure by provisioning an **AWS EC2 instance**. It covers the two prerequisites — the **AWS CLI** and an **AWS IAM user** with access keys — configuring the AWS provider in Terraform, and writing the resource block to launch an EC2 instance.

Provisioning on AWS is quite straightforward with Terraform: define the provider, define the resource, and let Terraform handle the API calls.

---

## Prerequisites

### AWS CLI installed

The AWS Command Line Interface (AWS CLI) is a unified tool to manage AWS services. With a single tool to download and configure, multiple AWS services can be controlled from the command line and automated through scripts.

**Install AWS CLI (Linux/Ubuntu):**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install
```

Verify:
```bash
aws --version
```

### AWS IAM user

AWS Identity and Access Management (IAM) is a web service that helps securely control access to AWS resources — controlling who is authenticated (signed in) and authorized (has permissions) to use them.

To connect an AWS account to Terraform, an IAM user's **Access Key ID** and **Secret Access Key** need to be available to the machine running Terraform.

**Steps taken:**
1. In the AWS Console → IAM → Users → created a new IAM user (e.g. `terraform-user`)
2. Attached the `AmazonEC2FullAccess` policy (scoped permissions appropriate for this exercise)
3. Generated an **Access Key ID** and **Secret Access Key** for programmatic access
4. Configured the credentials locally:
   ```bash
   aws configure
   ```
   Prompted for:
   - AWS Access Key ID
   - AWS Secret Access Key
   - Default region (e.g. `ap-south-1`)
   - Default output format (e.g. `json`)

   Credentials are stored at `~/.aws/credentials`, which Terraform's AWS provider reads by default — no need to hardcode keys in `.tf` files.

> ⚠️ Access keys are sensitive credentials. They're never committed to the repo or hardcoded in `.tf` files — they're kept in `~/.aws/credentials` (or environment variables), which is excluded via `.gitignore`.

---

## Install Required Providers

**`main.tf`** — Terraform block declaring the AWS provider:
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
```

---

## Tasks / Exercises

### 1. Provision an AWS EC2 instance using Terraform

**`variables.tf`**
```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
  default     = "ami-0c2af51e265bd5e0e"  # Ubuntu 22.04 LTS, ap-south-1
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "key_name" {
  description = "Name of an existing EC2 key pair"
  type        = string
  default     = "day62-keypair"
}

variable "instance_name" {
  description = "Name tag for the EC2 instance"
  type        = string
  default     = "day62-terraform-ec2"
}
```

**`main.tf`** — EC2 resource block:
```hcl
resource "aws_instance" "day62_ec2" {
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  tags = {
    Name = var.instance_name
  }
}
```

**`outputs.tf`**
```hcl
output "instance_id" {
  value = aws_instance.day62_ec2.id
}

output "public_ip" {
  value = aws_instance.day62_ec2.public_ip
}
```

Run it:
```bash
terraform init
terraform validate
terraform fmt
terraform plan
terraform apply
```

Verify:
```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=day62-terraform-ec2" \
  --query "Reservations[].Instances[].State.Name"
```

Or check the EC2 console — the instance appears with the `Name` tag set via Terraform.

Tear down when done:
```bash
terraform destroy
```

---

## 🧠 Key Learnings

- The AWS provider authenticates using credentials from `~/.aws/credentials` (set via `aws configure`), so `.tf` files never need to contain secrets
- An IAM user with scoped permissions (rather than root account credentials) is the correct way to give Terraform access to an AWS account
- `terraform plan` before `apply` is especially important against real cloud infrastructure — it's the last check before something billable gets created
- EC2 provisioning follows the same pattern learned with Docker on Day 60: provider block → resource block → `init` → `plan` → `apply`, just against a different provider
- `terraform destroy` is essential for cloud resources specifically to avoid ongoing charges after the exercise is done

## ⚠️ Challenges Faced

- Making sure the IAM user had the right policy attached (`AmazonEC2FullAccess`) before Terraform could successfully create the instance
- Choosing a valid, region-matching AMI ID — an AMI ID from the wrong region causes `terraform apply` to fail

## 🔗 Resources

- AWS CLI installation guide
- AWS IAM documentation
- Terraform AWS Provider documentation (`hashicorp/aws`)

---
