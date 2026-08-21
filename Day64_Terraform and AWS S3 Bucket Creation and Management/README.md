# Day 64 — Terraform and AWS S3 Bucket Creation and Management

## 📌 Overview

Day 64 covers provisioning and managing an **Amazon S3 bucket** with Terraform — creating the bucket, configuring public read access, attaching a bucket policy for read-only IAM access, and enabling versioning.

> **Note:** The source task list is headed "Create a security group," but the actual bullets are all S3-related (public read access, bucket policy, versioning) — this looks like a leftover heading carried over from Day 63 in the curriculum material. This README addresses the S3 tasks as written in the bullets.

---

## AWS S3 Bucket

Amazon S3 (Simple Storage Service) is an object storage service offering industry-leading scalability, data availability, security, and performance. It supports a wide range of use cases — storing and retrieving arbitrary data, hosting static websites, backups, data lakes, and more.

---

## Tasks / Exercises

### Prerequisite: Provider setup

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
```

### 1. Create an S3 bucket using Terraform

**`variables.tf`**
```hcl
variable "bucket_name" {
  description = "Globally unique name for the S3 bucket"
  type        = string
  default     = "day64-yukta-devops-bucket"  # must be globally unique — change before applying
}
```

**`main.tf`**
```hcl
resource "aws_s3_bucket" "day64_bucket" {
  bucket = var.bucket_name

  tags = {
    Name = var.bucket_name
    Day  = "64"
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

> S3 bucket names must be **globally unique** across all of AWS, not just within an account — `var.bucket_name` should be changed to something unique before applying.

---

### 2. Configure the bucket to allow public read access

Modern AWS accounts block public bucket access by default via **Block Public Access** settings, so those settings must be explicitly disabled before a public policy can take effect.

```hcl
resource "aws_s3_bucket_public_access_block" "day64_public_access" {
  bucket = aws_s3_bucket.day64_bucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_policy" "day64_public_read" {
  bucket = aws_s3_bucket.day64_bucket.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "PublicReadGetObject"
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource  = "${aws_s3_bucket.day64_bucket.arn}/*"
      }
    ]
  })

  depends_on = [aws_s3_bucket_public_access_block.day64_public_access]
}
```

> ⚠️ Public read access exposes every object in the bucket to anyone on the internet. This is appropriate for something like static website hosting, but should never be used for buckets holding sensitive or private data.

---

### 3. Create an S3 bucket policy for read-only access to a specific IAM user/role

Rather than opening the bucket to the public, this policy scopes read-only access to one specific IAM user.

**`variables.tf`** (addition)
```hcl
variable "readonly_iam_user_arn" {
  description = "ARN of the IAM user granted read-only access to the bucket"
  type        = string
  default     = "arn:aws:iam::123456789012:user/readonly-user"  # replace with actual ARN
}
```

**`main.tf`**
```hcl
resource "aws_s3_bucket_policy" "day64_readonly_user" {
  bucket = aws_s3_bucket.day64_bucket.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "ReadOnlyForSpecificUser"
        Effect    = "Allow"
        Principal = {
          AWS = var.readonly_iam_user_arn
        }
        Action = [
          "s3:GetObject",
          "s3:ListBucket"
        ]
        Resource = [
          aws_s3_bucket.day64_bucket.arn,
          "${aws_s3_bucket.day64_bucket.arn}/*"
        ]
      }
    ]
  })
}
```

> Note: A bucket can only have **one** resource-based bucket policy at a time. The public-read policy (Task 2) and the scoped IAM-user policy (Task 3) are alternative approaches for different use cases — in practice, only one `aws_s3_bucket_policy` would be applied per bucket, or both statements would be combined into a single policy document.

---

### 4. Enable versioning on the S3 bucket

Versioning keeps multiple variants of an object in the same bucket, protecting against accidental overwrites and deletions.

```hcl
resource "aws_s3_bucket_versioning" "day64_versioning" {
  bucket = aws_s3_bucket.day64_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

Run:
```bash
terraform apply
```

Verify:
```bash
aws s3api get-bucket-versioning --bucket <bucket_name>
```

---

## 🧠 Key Learnings

- S3 bucket configuration in modern Terraform AWS provider versions is split across multiple resources (`aws_s3_bucket`, `aws_s3_bucket_policy`, `aws_s3_bucket_versioning`, `aws_s3_bucket_public_access_block`) rather than one monolithic resource — this replaced older inline arguments from provider v3 and earlier
- Public access to an S3 bucket requires **two** things to align: the Block Public Access settings must permit it, *and* a bucket policy must explicitly grant it
- Bucket names are globally unique across all AWS accounts, not just within one account — a name clash will fail `apply` with a clear error
- IAM-scoped policies (Task 3) are the safer, more realistic pattern for real-world access control compared to fully public buckets (Task 2)
- Versioning is a simple but powerful safeguard against accidental data loss, at the cost of additional storage for retained versions

## ⚠️ Challenges Faced

- Realizing that `block_public_acls`/`restrict_public_buckets` must be explicitly disabled before a public-read bucket policy will actually take effect — otherwise AWS silently blocks the access
- Understanding that only one bucket policy document can be attached at a time, so public-read and IAM-scoped read-only access needed to be treated as separate, mutually exclusive scenarios rather than both applied simultaneously

## 🔗 Resources

- Terraform `aws_s3_bucket` resource documentation
- Terraform `aws_s3_bucket_policy` resource documentation
- Terraform `aws_s3_bucket_versioning` resource documentation
- AWS S3 documentation — Block Public Access

---
