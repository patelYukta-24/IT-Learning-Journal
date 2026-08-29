# Day 86 — Project 10: Mounting an AWS S3 Bucket on an Amazon EC2 Linux Instance Using S3FS

## Project Overview

This project focuses on integrating core AWS services — IAM, EC2, and S3 — to mount an S3 bucket directly onto an EC2 Linux instance using **S3FS**, a FUSE-based (Filesystem in Userspace) driver. Once mounted, the S3 bucket behaves like a local directory on the EC2 instance, allowing standard file operations (create, read, update, delete, list) to be performed with familiar Linux commands, while the underlying data is actually stored durably in S3.

This is a common pattern for legacy applications that expect a local filesystem but need the durability, scalability, and low cost of S3 object storage — without rewriting the application to use the S3 API directly.

## Project Objective

- Understand and implement S3 bucket mounting on an EC2 Linux instance.
- Learn to manage AWS IAM users and policies with least-privilege access.
- Gain hands-on experience using the AWS CLI for resource management and verification.

## Skills Showcased

- AWS S3 and EC2 instance management
- IAM user and policy creation (least-privilege scoping)
- Utilization of S3FS to mount S3 buckets on Linux
- Command-line proficiency with the AWS CLI
- Linux file operations and persistent mount configuration (`/etc/fstab`)

---

## Task 1: Successfully Mount an AWS S3 Bucket on an Amazon EC2 Linux Instance

### Subtask 1 — Create an IAM User with Necessary Permissions to Access S3 Resources

1. Open the **IAM Console** → Users → **Create user**.
2. Name the user, e.g. `s3fs-ec2-user`.
3. Select **Programmatic access** (Access key – Access key ID and secret access key), since S3FS authenticates using AWS credentials, not console login.
4. Attach a **custom least-privilege policy** scoped only to the target bucket, instead of the broad `AmazonS3FullAccess` managed policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::my-s3fs-demo-bucket"
    },
    {
      "Sid": "ObjectAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-s3fs-demo-bucket/*"
    }
  ]
}
```

5. Name the policy `S3FSBucketAccessPolicy` and attach it to the `s3fs-ec2-user`.
6. On user creation, download or copy the **Access Key ID** and **Secret Access Key** — these are used by S3FS for authentication and are shown only once.

> **Security note:** Scoping the policy to a single bucket ARN (rather than `"Resource": "*"`) follows least-privilege principle — the IAM user can only interact with the one bucket needed for this project, nothing else in the account.

### Prerequisite — Create the S3 Bucket

```bash
aws s3api create-bucket \
  --bucket my-s3fs-demo-bucket \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
```

Verify the bucket exists:

```bash
aws s3 ls
```

### Prerequisite — Launch an EC2 Linux Instance

- Launched an Amazon Linux 2023 EC2 `t2.micro` instance in the same region as the bucket.
- Security group allowed inbound SSH (port 22) from my IP.
- Connected via SSH:

```bash
ssh -i my-key.pem ec2-user@<EC2_PUBLIC_IP>
```

### Subtask 2 — Install and Configure S3FS on the EC2 Linux Instance

1. Install build dependencies and S3FS from the package repository:

```bash
# Amazon Linux 2023 / Amazon Linux 2 (EPEL required on AL2)
sudo yum install -y s3fs-fuse

# If not available directly, build from source:
sudo yum install -y automake fuse fuse-devel gcc-c++ git libcurl-devel \
  libxml2-devel make openssl-devel

git clone https://github.com/s3fs-fuse/s3fs-fuse.git
cd s3fs-fuse
./autogen.sh
./configure
make
sudo make install
```

2. Verify installation:

```bash
s3fs --version
```

3. Configure AWS credentials for S3FS in a password file, using the IAM user's access key created in Subtask 1:

```bash
echo "AKIAxxxxxxxxxxxxxxxx:wJalrxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" > $HOME/.passwd-s3fs
chmod 600 $HOME/.passwd-s3fs
```

> The file format is `ACCESS_KEY_ID:SECRET_ACCESS_KEY`. Restricting permissions to `600` ensures only the file owner can read the credentials.

4. Create the local mount point directory:

```bash
sudo mkdir -p /mnt/s3-bucket
```

### Subtask 3 — Mount the S3 Bucket and Perform File Operations to Validate the Setup

1. Mount the bucket using S3FS:

```bash
sudo s3fs my-s3fs-demo-bucket /mnt/s3-bucket \
  -o passwd_file=$HOME/.passwd-s3fs \
  -o url=https://s3.ap-south-1.amazonaws.com \
  -o use_path_request_style \
  -o allow_other
```

2. Confirm the mount succeeded:

```bash
df -hT | grep s3fs
mount | grep s3fs
```

3. Validate with file operations inside the mounted directory:

```bash
cd /mnt/s3-bucket

# Create a file
echo "Hello from EC2 via S3FS" > testfile.txt

# List contents
ls -l

# Read the file back
cat testfile.txt

# Modify the file
echo "Appending a second line" >> testfile.txt
cat testfile.txt

# Create a subdirectory
mkdir demo-folder
echo "nested file" > demo-folder/nested.txt
ls -R
```

4. Cross-verify directly through S3 (via the AWS CLI, independent of the mount) that the objects actually landed in the bucket:

```bash
aws s3 ls s3://my-s3fs-demo-bucket/ --recursive
```

This confirmed that files created through the mounted local path (`/mnt/s3-bucket`) were correctly written as objects in the underlying S3 bucket, and vice versa — objects uploaded via `aws s3 cp` appeared as files under the mount point.

5. (Optional but implemented) Make the mount persistent across reboots by adding an entry to `/etc/fstab`:

```
s3fs#my-s3fs-demo-bucket /mnt/s3-bucket fuse _netdev,allow_other,passwd_file=/home/ec2-user/.passwd-s3fs,url=https://s3.ap-south-1.amazonaws.com,use_path_request_style 0 0
```

Tested by unmounting and remounting via `fstab`:

```bash
sudo umount /mnt/s3-bucket
sudo mount -a
df -hT | grep s3fs
```

### Troubleshooting Notes (encountered during self-study)

| Issue | Cause | Fix |
|---|---|---|
| `fuse: mountpoint is not empty` | Mount directory had leftover files | Emptied the directory or used a fresh one before mounting |
| `Transport endpoint is not connected` | Previous mount crashed/hung | `sudo umount -l /mnt/s3-bucket` (lazy unmount), then remount |
| `Permission denied` on `.passwd-s3fs` | File permissions too open | `chmod 600 ~/.passwd-s3fs` — S3FS refuses to use credential files with loose permissions |
| Slow directory listing | S3FS makes a `ListBucket` API call per `ls` (no true local caching by default) | Used `-o stat_cache_expire=<seconds>` and `-o enable_noobj_cache` for lighter workloads |

---

## Deliverables

- ✅ An EC2 Linux instance with a mounted S3 bucket, accessible and operable as a local file system (`/mnt/s3-bucket`).
- ✅ IAM user (`s3fs-ec2-user`) and a custom, bucket-scoped IAM policy (`S3FSBucketAccessPolicy`) enabling secure, least-privilege access to the S3 bucket.
- ✅ Evidence of file operations — creating, listing, modifying, and nesting files/directories — validated both locally on the EC2 instance and independently via `aws s3 ls`.
- ✅ Persistent mount configuration via `/etc/fstab`, tested across unmount/remount.

## Cleanup (to avoid ongoing charges)

```bash
sudo umount /mnt/s3-bucket
aws s3 rm s3://my-s3fs-demo-bucket --recursive
aws s3api delete-bucket --bucket my-s3fs-demo-bucket --region ap-south-1
```
- Terminated the EC2 instance from the console.
- Deleted the IAM user's access key, then the IAM user and custom policy.

---

## Key Takeaways

- S3FS bridges object storage and traditional filesystem semantics via FUSE, but it is **not** a true POSIX filesystem — operations like partial writes, file locking, and rapid small writes are slower and less reliable than on native block storage (e.g., EBS).
- Scoping IAM policies to a specific bucket ARN (rather than using broad managed policies) is a practical way to apply least-privilege access for automation and instance-level integrations.
- Persisting the mount through `/etc/fstab` with the `_netdev` option ensures the mount waits for networking to be available before attempting to connect to S3, which matters for EC2 boot-time reliability.
- This pattern is best suited for read-heavy or archival workloads (e.g., serving static assets, log archival) rather than high-throughput transactional file I/O.

---

## Resources

- [Official AWS documentation on IAM users and policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)
- [S3FS-FUSE GitHub repository and documentation](https://github.com/s3fs-fuse/s3fs-fuse)
- [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/reference/s3/)

---
