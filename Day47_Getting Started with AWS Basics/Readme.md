# Day 47: Getting Started with AWS Basics

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. AWS (Amazon Web Services)](#1-aws-amazon-web-services)
- [2. IAM (Identity and Access Management)](#2-iam-identity-and-access-management)
- [3. IAM Tasks](#3-iam-tasks)
- [4. User Data in AWS](#4-user-data-in-aws)
- [5. User Data Tasks](#5-user-data-tasks)
- [6. Troubleshooting Notes](#6-troubleshooting-notes)
- [7. Key Learnings](#7-key-learnings)
- [8. Resources](#8-resources)

---

## Overview
Day 47 opens the AWS module of the NexusCorp DevOps Transformation curriculum. The goal is to build a working foundation in two areas that underpin almost everything else on AWS:

1. **IAM** — how identity, access, and permissions are structured and secured
2. **EC2 User Data** — how instance configuration can be automated at boot time instead of done by hand

Everything below is documented as self-study: explanations in my own words, full command references I can execute and verify on a Free Tier account, and clearly marked placeholders for screenshots once each task is actually run.

## Topics Covered
- ❖ AWS — free tier and account setup
- ❖ IAM — users, groups, roles, policies (concepts + hands-on)
- ❖ EC2 User Data — shell scripts vs. cloud-init, automated bootstrapping

---

## 1. AWS (Amazon Web Services)
AWS is one of the largest cloud providers, offering on-demand infrastructure (compute, storage, networking, databases, and more) on a pay-as-you-go basis. Its **Free Tier** is specifically useful for students and career-changers because it provides:
- 750 hours/month of `t2.micro`/`t3.micro` EC2 usage (12 months from signup)
- 5 GB of S3 storage
- Various "always free" limits on services like Lambda and DynamoDB

**Account setup checklist:**
1. Sign up at [aws.amazon.com/free](https://aws.amazon.com/free/) with a valid card (used only for identity verification within Free Tier limits)
2. Enable **MFA on the root account** immediately — root should never be used for daily work
3. Set a **Billing Alarm** via AWS Budgets so you're notified before any charges apply
4. Create an **IAM Administrator user/group** and stop using root for anything beyond account-level tasks

---

## 2. IAM (Identity and Access Management)
IAM is AWS's global (not region-scoped) service for controlling **authentication** (who can sign in) and **authorization** (what they're allowed to do) across your account.

### 2.1 Core Building Blocks

| Concept | What it is | When to use it |
|---|---|---|
| **User** | A permanent identity for one person or application, with its own credentials (password for console, access keys for CLI/API) | Individual humans or apps needing standing access |
| **Group** | A named collection of users; policies attached to the group apply to every member | Managing permissions for a team, instead of user-by-user |
| **Role** | A temporary identity with no long-term credentials, meant to be *assumed* — by a user, another AWS account, or an AWS service | EC2 instances, Lambda functions, cross-account access, federated users |
| **Policy** | A JSON document listing allowed/denied actions on specific resources | Attached to users, groups, or roles to actually grant permissions |

### 2.2 Policy Types
- **AWS Managed Policies** — pre-built by AWS (e.g., `AmazonEC2FullAccess`, `ReadOnlyAccess`) — fast to use, but broader than strictly needed
- **Customer Managed Policies** — JSON policies you write and manage yourself — used for least-privilege access
- **Inline Policies** — a policy embedded directly in a single user/group/role — not reusable, generally avoided in favor of managed policies

### 2.3 Users vs. Groups vs. Roles — the key distinction
Users and Groups represent **identity** — "who is this." Roles represent **temporary, assumable trust** — "who can act as this, for how long, and to do what." A Role has no password or long-term access key; instead, whoever/whatever assumes it gets short-lived, auto-rotating credentials via AWS STS (Security Token Service). This is why roles — not embedded access keys — are the AWS-recommended way to give an EC2 instance permission to talk to other AWS services.

### 2.4 IAM Trust Boundary (conceptual)
```
Root Account
   │
   ├── IAM Users ──► attached to ──► IAM Groups ──► attached to ──► Policies
   │
   └── IAM Roles ──► assumed by ──► EC2 / Lambda / other AWS accounts / federated identities
                          │
                          └──► granted temporary credentials (STS) ──► Policies
```

---

## 3. IAM Tasks

### Task 1 — IAM User with EC2 Access → Launch Instance → Install Jenkins + Docker via Shell Script

**Objective:** Create an IAM user with EC2 permissions, use that identity to launch a Linux instance, and install Jenkins + Docker via a single shell script (User Data or manual SSH).

**Step-by-step (console):**
1. IAM → Users → Create user → name: `devops-ec2-admin`
2. Enable **console access** (custom password, require reset on first login)
3. Permissions → Attach policy directly → `AmazonEC2FullAccess`
4. (Optional, more secure) Instead of the full-access managed policy, attach a scoped custom policy — see §2.2
5. Sign out of root/admin, sign back in as `devops-ec2-admin` using the account's IAM sign-in URL
6. EC2 → Launch Instance → Amazon Linux 2023 → `t2.micro` (Free Tier eligible)
7. Create/select a key pair (`.pem`) — needed for SSH if not using User Data
8. Security Group: allow inbound `22` (SSH) from *My IP*, and `8080` (Jenkins) from *My IP* or `0.0.0.0/0` for testing
9. Paste the install script into **Advanced details → User data** (see script below), or SSH in after launch and run it manually

**Equivalent via AWS CLI** (once the IAM user has CLI access configured via `aws configure`):
```bash
aws ec2 run-instances \
  --image-id ami-0c101f26f147fa7fd \
  --instance-type t2.micro \
  --key-name my-keypair \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --user-data file://install-jenkins-docker.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Day47-Jenkins-Docker}]'
```

**Install script (`install-jenkins-docker.sh`):**
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

echo "Docker version: $(docker --version)"
echo "Jenkins status: $(systemctl is-active jenkins)"
```

**Verification:**
```bash
ssh -i my-keypair.pem ec2-user@<public-ip>
docker --version
sudo systemctl status jenkins
```
> Status: reference implementation — pending execution on a live Free Tier instance; screenshots (console + terminal output) to be attached once run.

---

### Task 2 — DevOps Team of Avengers: 3 IAM Users + Group + Policy

**Objective:** Create 3 IAM users, group them under a shared DevOps group, and attach a policy at the group level.

**Step-by-step (console):**
1. IAM → User groups → Create group → name: `Avengers-DevOps`
2. IAM → Users → Create user (repeat 3×): `ironman`, `captain-america`, `thor`
   - Enable console access for each, require password reset on first login
   - Add each user to the `Avengers-DevOps` group during creation (or after, via group membership)
3. IAM → Policies → Create policy → paste JSON below → name: `AvengersDevOpsPolicy`
4. IAM → User groups → `Avengers-DevOps` → Permissions → Attach policy → select `AvengersDevOpsPolicy`

**Custom least-privilege policy (`avengers-devops-policy.json`):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2LimitedAccess",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RunInstances",
        "ec2:TerminateInstances"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyRegionOutsideAllowed",
      "Effect": "Deny",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": "ap-south-1"
        }
      }
    }
  ]
}
```

**Equivalent via AWS CLI:**
```bash
# Create group
aws iam create-group --group-name Avengers-DevOps

# Create users
for user in ironman captain-america thor; do
  aws iam create-user --user-name "$user"
  aws iam add-user-to-group --user-name "$user" --group-name Avengers-DevOps
done

# Create and attach policy
aws iam create-policy --policy-name AvengersDevOpsPolicy \
  --policy-document file://avengers-devops-policy.json

aws iam attach-group-policy --group-name Avengers-DevOps \
  --policy-arn arn:aws:iam::<account-id>:policy/AvengersDevOpsPolicy
```

**Verification:**
```bash
aws iam get-group --group-name Avengers-DevOps
aws iam list-attached-group-policies --group-name Avengers-DevOps
```
> Status: reference implementation — pending execution + console screenshots.

---

## 4. User Data in AWS
User Data is a mechanism to pass a script or configuration directive to an EC2 instance, which is executed automatically **once, on first boot**, by the `cloud-init` service preinstalled on most AMIs. It removes the need to manually SSH in and configure every instance by hand.

### 4.1 Types of User Data
| Type | Format | Use case |
|---|---|---|
| **Shell script** | Starts with `#!/bin/bash` | Imperative, step-by-step commands — most common for simple bootstrapping |
| **Cloud-init directives** | YAML, starts with `#cloud-config` | Declarative configuration — users, packages, files, in a structured format |

### 4.2 How User Data Can Be Passed
- **Plain text** — pasted directly into the "User data" field in the Launch Instance wizard
- **File** — referenced via `--user-data file://script.sh` in the CLI (useful for longer scripts)
- **Base64-encoded text** — required format when calling the EC2 API directly (the console/CLI handle this encoding automatically)

### 4.3 Important Characteristics
- User Data only runs **on first boot** by default (not on every reboot)
- Logs of User Data execution can be checked at `/var/log/cloud-init-output.log`
- Runs as **root**, so no `sudo` prefix is technically required inside the script, though it doesn't hurt
- Has a size limit of **16 KB** (raw, before Base64 encoding)

---

## 5. User Data Tasks

### Task 1 — EC2 Instance with Jenkins Pre-Installed via User Data

**Objective:** Launch an instance with Jenkins auto-installed via User Data, then confirm by browsing to its public IP.

**User Data script (`jenkins-userdata.sh`):**
```bash
#!/bin/bash
sudo yum update -y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install -y fontconfig java-17-openjdk jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

**Launch via CLI:**
```bash
aws ec2 run-instances \
  --image-id ami-0c101f26f147fa7fd \
  --instance-type t2.micro \
  --key-name my-keypair \
  --security-group-ids sg-xxxxxxxx \
  --user-data file://jenkins-userdata.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Day47-Jenkins-UserData}]'
```

**Verification steps:**
1. Open inbound port `8080` on the instance's Security Group
2. Wait 2–3 minutes for User Data to finish running
3. SSH in and check the log: `sudo cat /var/log/cloud-init-output.log`
4. Browse to `http://<public-ip>:8080` — Jenkins unlock screen should load
5. Retrieve the initial admin password: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

> Screenshots (User Data field in console + Jenkins unlock page in browser) — pending, to be attached after execution on a Free Tier instance.

---

### Task 2 — IAM Roles: Explanation + Creating Three Roles

**IAM Users, Groups, and Roles — explained in my own words:**

- **IAM User** — A long-term identity for one person or app. Has its own login credentials. Best for humans who need direct, ongoing AWS access (e.g., a developer signing into the console daily).
- **IAM Group** — Not an identity itself, just a way to bundle users together so a policy only needs to be attached once instead of per-user. Makes onboarding/offboarding and audits much simpler.
- **IAM Role** — A temporary identity with **no permanent credentials**. Something (a user, an AWS service, another AWS account) *assumes* the role and receives short-lived credentials from AWS STS. This is the secure pattern for granting AWS services — like an EC2 instance running an app that needs to read from S3 — access without ever hardcoding an access key on disk.

**Why roles matter more as environments grow:** hardcoded access keys on an EC2 instance are a common real-world security incident — if the instance is compromised, so are the keys, and they don't expire. An IAM Role attached via an **Instance Profile** solves this: the instance gets temporary, auto-rotating credentials it can use, and nothing is stored permanently on disk.

**Creating the three roles (console):**
1. IAM → Roles → Create role
2. Choose trusted entity type:
   - `DevOps-User` and `Admin` → **AWS service** → EC2 (if meant to be attached to instances) or **AWS account** (if meant to be assumed by IAM users)
   - `Test-User` → same, scoped to read-only use
3. Attach permissions:
   - `DevOps-User` → EC2 + S3 read/write (custom or `AmazonEC2FullAccess` + `AmazonS3FullAccess` for learning purposes)
   - `Test-User` → `ReadOnlyAccess` (AWS managed policy)
   - `Admin` → `AdministratorAccess` (used here only for demonstrating role creation — not a production-safe default)
4. Name and create each role

**Equivalent via AWS CLI:**
```bash
# Trust policy allowing EC2 to assume the role
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

for role in DevOps-User Test-User Admin; do
  aws iam create-role --role-name "$role" \
    --assume-role-policy-document file://trust-policy.json
done

aws iam attach-role-policy --role-name DevOps-User \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
aws iam attach-role-policy --role-name Test-User \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam attach-role-policy --role-name Admin \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

**Verification:**
```bash
aws iam list-roles --query "Roles[?RoleName=='DevOps-User' || RoleName=='Test-User' || RoleName=='Admin'].RoleName"
aws iam list-attached-role-policies --role-name DevOps-User
```
> Status: reference implementation — pending execution + console screenshots.

---

## 6. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| Jenkins page doesn't load on `:8080` | Security Group doesn't allow inbound 8080 | Add inbound rule for TCP 8080 from your IP |
| User Data script didn't run | Script missing `#!/bin/bash` shebang, or syntax error | Check `/var/log/cloud-init-output.log` for errors |
| "Access Denied" as IAM user | Policy not attached, or attached to wrong group/user | Verify with `aws iam list-attached-user-policies` / `list-attached-group-policies` |
| Can't SSH into instance | Security Group missing port 22, or wrong key pair | Confirm SG inbound rule and that the correct `.pem` is used |
| Role not usable by EC2 | Role created without EC2 as trusted entity, or no Instance Profile attached | Recheck trust policy; attach role via an Instance Profile at launch |

---

## 7. Key Learnings
- IAM Roles solve the "how does a service get permissions without hardcoded keys" problem — this is the pattern to default to, not access keys on disk.
- Group-level policy attachment scales far better than managing permissions per user, especially onboarding/offboarding a team like the "Avengers."
- EC2 User Data turns manual, error-prone setup into a repeatable, auditable script — this is a small but real step toward Infrastructure as Code thinking.
- Least-privilege custom policies (vs. broad managed policies like `FullAccess`) are worth the extra effort even in a learning environment, since they build the right habit early.

## 8. Resources
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM Roles Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [EC2 User Data Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [Jenkins Installation on Amazon Linux](https://www.jenkins.io/doc/book/installing/linux/)

---