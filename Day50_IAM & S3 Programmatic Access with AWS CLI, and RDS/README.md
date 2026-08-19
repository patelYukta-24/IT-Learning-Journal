# Day 50: IAM & S3 Programmatic Access with AWS CLI, and RDS

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. IAM Programmatic Access](#1-iam-programmatic-access)
- [2. AWS CLI](#2-aws-cli)
- [3. CLI Setup Tasks](#3-cli-setup-tasks)
- [4. S3 (Simple Storage Service)](#4-s3-simple-storage-service)
- [5. S3 Tasks](#5-s3-tasks)
- [6. Relational Database Service (RDS)](#6-relational-database-service-rds)
- [7. RDS Tasks](#7-rds-tasks)
- [8. Troubleshooting Notes](#8-troubleshooting-notes)
- [9. Key Learnings](#9-key-learnings)
- [10. Resources](#10-resources)

---

## Overview
Day 50 shifts from console-driven work to **programmatic AWS access** — generating and using Access Keys, configuring the AWS CLI, and using it to work with S3 and connect EC2 to RDS. This is the day the curriculum starts feeling more like real DevOps work: automating interactions with AWS instead of clicking through the console every time.

## Topics Covered
- ❖ IAM Programmatic Access (Access Key / Secret Access Key)
- ❖ AWS CLI — setup and configuration
- ❖ S3 — object storage, bucket creation, CLI file access
- ❖ RDS — managed MySQL database, EC2-to-RDS connectivity via IAM role

---

## 1. IAM Programmatic Access
Console access uses a username/password. **Programmatic access** — from a terminal, script, or the AWS CLI/SDK — instead uses a pair of long-term credentials:

- **AWS Access Key ID** — like a username; identifies the IAM user/role making the request
- **AWS Secret Access Key** — like a password; must be kept private and is only ever shown once, at creation time

### 1.1 Security Notes (important habits, not just theory)
- Never commit access keys to Git — this is one of the most common real-world AWS security incidents
- Prefer **IAM Roles** over long-term access keys wherever possible (e.g., for EC2-to-AWS-service access — see the RDS section below)
- If a key pair is ever exposed, deactivate/delete it immediately from IAM → Users → Security credentials
- Rotate access keys periodically even when not exposed

### 1.2 Creating an Access Key (Console)
1. IAM → Users → select your user → **Security credentials** tab
2. Under "Access keys" → Create access key
3. Choose use case: **Command Line Interface (CLI)**
4. Acknowledge the recommendation to use IAM roles where possible → Create
5. **Download the `.csv` or copy both values immediately** — the Secret Access Key is not retrievable again after this screen

---

## 2. AWS CLI
The AWS Command Line Interface (CLI) is a single tool for managing AWS services from the terminal, and for scripting/automating tasks that would otherwise require the console. **AWS CLI v2** (current version) adds improved installers, IAM Identity Center (successor to AWS SSO) support, and interactive features like auto-prompting.

### 2.1 Why It Matters for DevOps
- Enables automation and scripting (CI/CD pipelines, cron jobs, provisioning scripts)
- One consistent interface across all AWS services, instead of navigating different console UIs
- Required for many "headless"/server-side workflows — e.g., an EC2 instance pulling a file from S3 has no browser to click through the console with

---

## 3. CLI Setup Tasks

### Task — Create Access Keys + Install & Configure AWS CLI

**Step 1: Create the Access Key** — see §1.2 above.

**Step 2: Install AWS CLI v2**

*On Ubuntu/Linux (EC2 instance or local machine):*
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get install -y unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

*On Windows:* download and run the [AWS CLI MSI installer](https://awscli.amazonaws.com/AWSCLIV2.msi)

*On macOS:*
```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

**Step 3: Configure credentials**
```bash
aws configure
```
This prompts for, in order:
```
AWS Access Key ID [None]: <your access key>
AWS Secret Access Key [None]: <your secret key>
Default region name [None]: ap-south-1
Default output format [None]: json
```

**Verification:**
```bash
aws sts get-caller-identity
```
This should return your IAM user's Account ID, User ID, and ARN — confirming the CLI is authenticated correctly.

> Status: reference implementation — pending execution; screenshot of `get-caller-identity` output to be added once run.

---

## 4. S3 (Simple Storage Service)
Amazon S3 is object storage — designed to store and retrieve any amount of data (files, images, backups, logs) reliably and at scale. Data is stored as **objects** inside **buckets**, addressed by a unique key (essentially a file path).

### 4.1 Core S3 Concepts
| Concept | Description |
|---|---|
| **Bucket** | A top-level container for objects; the name must be globally unique across all of AWS |
| **Object** | The actual file/data stored, along with metadata |
| **Key** | The unique identifier (path-like string) for an object within a bucket |
| **Region** | Buckets are created in a specific region — data doesn't automatically replicate elsewhere unless configured |

---

## 5. S3 Tasks

### Task 1 — EC2 + SSH, S3 Bucket + Upload, Access File from EC2 via CLI

**Objective:** Launch an EC2 instance, SSH into it, create an S3 bucket, upload a file via the console, then read that file from the EC2 instance using the AWS CLI.

**Step-by-step:**
1. **Launch EC2** (console): Amazon Linux 2, `t2.micro`, key pair, Security Group allowing inbound `22`
2. **SSH in:**
   ```bash
   chmod 400 my-keypair.pem
   ssh -i my-keypair.pem ec2-user@<public-ip>
   ```
3. **Create the S3 bucket** (console): S3 → Create bucket → name: `yukta-day50-bucket` (must be globally unique) → keep defaults → Create
4. **Upload a file** (console): open the bucket → Upload → select a local file (e.g., `notes.txt`) → Upload
5. **Install AWS CLI on the EC2 instance** (if not already) and run `aws configure` with the same or an appropriately scoped access key
6. **Access the file from EC2 via CLI:**
   ```bash
   aws s3 ls s3://yukta-day50-bucket/
   aws s3 cp s3://yukta-day50-bucket/notes.txt .
   cat notes.txt
   ```

**IAM requirement:** the credentials configured on the EC2 instance need at least `s3:GetObject` and `s3:ListBucket` permissions on the bucket — either via an attached IAM Role (recommended, see note below) or a configured IAM user's access keys.

> **Best practice note:** rather than running `aws configure` with a personal access key on an EC2 instance, the AWS-recommended pattern is to attach an **IAM Role** (with an S3-read policy) to the instance via an Instance Profile — this avoids storing long-term credentials on disk at all. Worth doing this way once comfortable, even though the task describes CLI credential configuration.

> Status: reference implementation — pending execution + screenshots of the CLI download/`cat` output on the EC2 instance.

---

### Task 2 — AMI Snapshot, New Instance, CLI Download, Verify File Consistency

**Objective:** Create a snapshot/AMI of the EC2 instance from Task 1, launch a new instance from it, download the S3 file on the new instance via CLI, and confirm the file matches the original.

**Step-by-step:**
1. **Create an AMI from the running instance:**
   - EC2 → Instances → select instance → Actions → Image and templates → **Create image**
   - Name: `day50-ec2-snapshot`
   - No reboot: leave unchecked (safer, ensures filesystem consistency) → Create image
2. **Launch a new instance from that AMI:**
   - EC2 → AMIs → select `day50-ec2-snapshot` → Launch instance from AMI
   - Same instance type/key pair/security group considerations as before
3. **SSH into the new instance** and download the same file:
   ```bash
   ssh -i my-keypair.pem ec2-user@<new-instance-public-ip>
   aws s3 cp s3://yukta-day50-bucket/notes.txt .
   ```
4. **Verify contents match** across both instances:
   ```bash
   md5sum notes.txt
   ```
   Run this on both the original and the new instance — matching checksums confirm the file is identical.

**Equivalent via AWS CLI (for the snapshot/AMI creation):**
```bash
aws ec2 create-image \
  --instance-id <original-instance-id> \
  --name "day50-ec2-snapshot" \
  --no-reboot
```

> Status: reference implementation — pending execution + screenshots of matching `md5sum` output on both instances.

---

## 6. Relational Database Service (RDS)
Amazon RDS is a managed database service — AWS handles provisioning, patching, backups, and infrastructure maintenance for the underlying database engine (MySQL, PostgreSQL, MariaDB, SQL Server, Oracle, etc.), so the focus stays on the application/data rather than server administration.

### 6.1 Why RDS Instead of Self-Managed MySQL on EC2
- Automated backups and point-in-time recovery
- Automated patching and minor version upgrades
- Multi-AZ deployment option for high availability
- Read replicas for scaling read-heavy workloads
- Less operational overhead than maintaining a database server manually

---

## 7. RDS Tasks

### Task — Free Tier MySQL RDS + EC2 + IAM Role + MySQL Client Connection

**Objective:** Create a Free Tier MySQL RDS instance, an EC2 instance, an IAM role with RDS access attached to the EC2 instance, then connect from EC2 to RDS using a MySQL client.

**Step 1: Create the RDS MySQL instance (console)**
1. RDS → Create database
2. Engine: **MySQL**
3. Templates: **Free tier**
4. DB instance identifier: `day50-mysql-db`
5. Master username/password: set and note securely (not stored in this repo)
6. DB instance class: default Free Tier class (e.g., `db.t3.micro` / `db.t4g.micro`, depending on current Free Tier offering)
7. Connectivity: same VPC as your EC2 instance; **Public access: No** (best practice — EC2 connects via VPC, not the public internet)
8. VPC security group: create/select one that will allow inbound `3306` from the EC2 instance's security group specifically (not `0.0.0.0/0`)
9. Create database (provisioning takes a few minutes)

**Step 2: Create an IAM Role with RDS access**
1. IAM → Roles → Create role → Trusted entity: **AWS service → EC2**
2. Attach policy: `AmazonRDSFullAccess` (for learning; a scoped custom policy is better practice in production)
3. Name: `EC2-RDS-Access-Role` → Create role

**Step 3: Attach the role to EC2**
- If launching a new instance: Launch Instance → Advanced details → **IAM instance profile** → select `EC2-RDS-Access-Role`
- If attaching to an existing instance: EC2 → Instances → select instance → Actions → Security → **Modify IAM role** → select `EC2-RDS-Access-Role`

> Note: the IAM role governs the instance's permission to call the *RDS API* (e.g., describe/manage the database via AWS). Actually **connecting to the database itself** (via the MySQL protocol on port 3306) is controlled separately by the **security group** rules and the database username/password — both are needed together.

**Step 4: Install a MySQL client on EC2**
```bash
sudo yum install -y mysql        # Amazon Linux 2
# or, on Ubuntu:
sudo apt-get install -y mysql-client
```

**Step 5: Get RDS connection details**
- RDS → Databases → `day50-mysql-db` → copy the **Endpoint** and **Port** (default `3306`)

**Step 6: Connect from EC2 to RDS**
```bash
mysql -h <rds-endpoint> -P 3306 -u <master-username> -p
```
Enter the master password when prompted. On success, you'll land in the `mysql>` prompt.

**Quick sanity check once connected:**
```sql
SHOW DATABASES;
```

> Status: reference implementation — pending execution. Per the task instructions, a screenshot of the EC2-to-RDS `mysql` connection succeeding is the deliverable to capture here once run.

---

## 8. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| `aws sts get-caller-identity` fails | Access key/secret entered incorrectly, or not configured | Re-run `aws configure` and double-check both values |
| `aws s3 cp` gives "Access Denied" | IAM user/role lacks S3 permissions on that bucket | Attach a policy with `s3:GetObject`/`s3:ListBucket` scoped to the bucket ARN |
| Can't SSH into new instance from AMI | Wrong key pair selected at launch, or SG missing port 22 | Reconfirm key pair and Security Group inbound rules |
| `mysql -h <endpoint>` times out | RDS security group doesn't allow inbound 3306 from EC2's SG | Edit RDS SG inbound rule: source = EC2 instance's security group |
| `mysql: command not found` | Client not installed | Install per §7 Step 4, matching the instance's OS package manager |
| EC2 can't reach RDS despite correct SG | RDS instance is in a different VPC/subnet with no route | Ensure RDS and EC2 share a VPC (or have proper VPC peering/routing) |

---

## 9. Key Learnings
- Programmatic access (Access Key/Secret Key) is how automation and CLI tools authenticate — but IAM Roles are the safer pattern for anything running *on* AWS (like EC2), since they avoid storing long-term credentials on disk.
- The AWS CLI turns console-only tasks into scriptable, repeatable commands — a necessary skill for any DevOps/automation-focused role.
- S3 access from EC2 needs the right IAM permissions in place before `aws s3 cp` will work — "Access Denied" is almost always a policy gap, not a bucket problem.
- Connecting EC2 to RDS involves two separate layers of access control: the IAM role for the AWS API, and the Security Group + DB credentials for the actual MySQL connection — both need to be correct.
- Keeping RDS **not publicly accessible** and only reachable from EC2's Security Group is the correct default for a real environment, even for Free Tier practice.

## 10. Resources
- [AWS CLI — Getting Started](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
- [IAM Access Keys — AWS Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)
- [Amazon S3 — Getting Started](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html)
- [AWS CLI S3 Commands Reference](https://docs.aws.amazon.com/cli/latest/reference/s3/)
- [Amazon RDS — User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [Connecting to a MySQL DB Instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html)

---
