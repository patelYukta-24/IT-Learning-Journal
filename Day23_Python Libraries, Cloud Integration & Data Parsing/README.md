# Day 23: Python Libraries, Cloud Integration & Data Parsing

> **DevOps Transformation** | Python & Automation Series — Final Day

---

## 📋 Overview

This is the final day of the NexusCorp DevOps Transformation guided curriculum.

Over the past 22 days you have built a complete foundation: Linux command line, file permissions, process management, networking, package management, Git version control, shell scripting, and Python fundamentals. Today you connect Python to the real world — libraries that extend what Python can do, cloud SDKs that let you manage infrastructure programmatically, and data formats (JSON, CSV, XML) that are the lingua franca of modern systems.

By the end of today you will have written scripts that install and use third-party libraries, read and write JSON and CSV files, interact with cloud service APIs, and process real-world data formats — the exact skills that power every automation pipeline in production DevOps.

---

## 🗺️ What You'll Do Today

| Task | Topic | What You Build |
|---|---|---|
| Task 1 | Python Libraries & `pip` | Install and use `requests` |
| Task 2 | Python in the Cloud | Overview of cloud SDKs and `boto3` |
| Task 3 | Working with JSON | Read, modify, and write JSON files |
| Task 4 | Other Data Formats | Read and process CSV files |
| Task 5 | Exploring More Libraries | Install and explore `boto3` or `docker` |
| Task 6 | Final Reflection | Consolidate key takeaways |

---

## 🏢 NexusCorp Scenario

> NexusCorp's automation team needs three things this sprint: a script that fetches server health data from an internal REST API and saves it as JSON, a report generator that reads that JSON and outputs a CSV for the management team, and a tool that uses the AWS SDK to list all EC2 instances across the account. Every one of these is a task for today's toolkit — `requests`, `json`, `csv`, and `boto3`.

---

## Part 1: Understanding Python's Library Ecosystem

### The Python Standard Library

Python ships with a **standard library** — a large collection of modules that are available immediately, with no installation required. These cover an enormous range of tasks:

| Module | What It Does | DevOps Use Case |
|---|---|---|
| `os` | Operating system interface | File paths, env variables, directory operations |
| `sys` | System-specific parameters | Exit codes, command-line arguments |
| `json` | JSON encoding and decoding | Parse API responses, read/write config files |
| `csv` | CSV file reading and writing | Reports, data exports, inventory files |
| `pathlib` | Object-oriented file paths | Cross-platform path handling |
| `datetime` | Dates and times | Timestamps, log rotation, scheduling |
| `subprocess` | Run shell commands | Execute system commands from Python |
| `re` | Regular expressions | Parse log files, validate inputs |
| `logging` | Structured logging | Replace `print()` in production scripts |
| `argparse` | Command-line argument parsing | Build CLI tools |
| `shutil` | File operations (copy, move, archive) | Backup scripts, file management |
| `socket` | Low-level networking | Port scanners, connectivity checks |
| `hashlib` | Cryptographic hashing | File integrity verification |
| `configparser` | `.ini` config file parsing | Read application config files |
| `collections` | Specialized data structures | `defaultdict`, `Counter`, `namedtuple` |
| `itertools` | Efficient iteration | Complex loops and data pipelines |
| `functools` | Higher-order functions | `partial`, `lru_cache` for optimization |
| `threading` | Thread-based parallelism | Concurrent server checks |
| `multiprocessing` | Process-based parallelism | CPU-intensive automation tasks |
| `urllib` | URL handling and HTTP | Basic HTTP requests (use `requests` for more) |
| `zipfile` / `tarfile` | Archive handling | Packaging deployments, backup archives |

---

### Third-Party Libraries — The PyPI Ecosystem

Beyond the standard library, the **Python Package Index (PyPI)** hosts over 400,000 packages installable via `pip`. This is Python's greatest strength — almost any integration already has a maintained library.

**Essential DevOps and Cloud libraries:**

| Library | Install | Purpose |
|---|---|---|
| `requests` | `pip install requests` | HTTP API calls — cleaner than `urllib` |
| `boto3` | `pip install boto3` | AWS SDK — manage EC2, S3, IAM, etc. |
| `google-cloud` | `pip install google-cloud` | GCP SDK |
| `azure-sdk` | `pip install azure-mgmt-compute` | Azure SDK |
| `paramiko` | `pip install paramiko` | SSH connections and remote execution |
| `docker` | `pip install docker` | Docker Engine API |
| `kubernetes` | `pip install kubernetes` | Kubernetes API client |
| `ansible-runner` | `pip install ansible-runner` | Run Ansible playbooks from Python |
| `pyyaml` | `pip install pyyaml` | Parse YAML files (k8s manifests, etc.) |
| `python-dotenv` | `pip install python-dotenv` | Load `.env` files |
| `click` | `pip install click` | Build CLI tools elegantly |
| `rich` | `pip install rich` | Beautiful terminal output |
| `httpx` | `pip install httpx` | Modern async HTTP client |
| `fabric` | `pip install fabric` | Remote server automation over SSH |
| `psutil` | `pip install psutil` | System and process utilities |
| `prometheus-client` | `pip install prometheus-client` | Expose metrics to Prometheus |
| `cryptography` | `pip install cryptography` | Encryption, certificates, secrets |

---

## Task 1: Exploring Python Libraries & `pip`

### Understanding `pip`

`pip` is Python's package manager — it downloads, installs, and manages third-party libraries from PyPI.

```bash
# Check pip version
pip3 --version

# Install a package
pip3 install requests

# Install a specific version
pip3 install requests==2.31.0

# Install multiple packages at once
pip3 install requests boto3 pyyaml

# Install from a requirements file (standard practice)
pip3 install -r requirements.txt

# List installed packages
pip3 list

# Show details about an installed package
pip3 show requests

# Check for outdated packages
pip3 list --outdated

# Upgrade a package
pip3 install --upgrade requests

# Uninstall a package
pip3 uninstall requests

# Save current environment to requirements.txt
pip3 freeze > requirements.txt
```

---

### Virtual Environments — Best Practice

Always use a **virtual environment** for Python projects. It isolates dependencies so different projects don't conflict.

```bash
# Create a virtual environment
python3 -m venv venv

# Activate it (Linux/macOS)
source venv/bin/activate

# Activate it (Windows)
venv\Scripts\activate

# Your prompt changes to show (venv)
# (venv) $

# Now pip install only affects this environment
pip install requests boto3 pyyaml

# Save dependencies
pip freeze > requirements.txt

# Deactivate when done
deactivate

# Later, recreate the environment from requirements.txt
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

### The `requests` Library

`requests` is the standard Python library for making HTTP requests — calling REST APIs, fetching web pages, sending data to services.

```bash
pip3 install requests
```

```python
# requests_demo.py
import requests
import json

# ── Basic GET request ──────────────────────────────────────────
response = requests.get("https://httpbin.org/get")

print(f"Status Code : {response.status_code}")
print(f"Content-Type: {response.headers['Content-Type']}")
print(f"Response URL: {response.url}")

# Parse JSON response
data = response.json()
print(f"Your IP     : {data['origin']}")


# ── GET with parameters ────────────────────────────────────────
params = {"key": "value", "env": "staging"}
response = requests.get("https://httpbin.org/get", params=params)
print(f"\nRequest URL : {response.url}")


# ── POST request ───────────────────────────────────────────────
payload = {
    "server": "web-01",
    "status": "healthy",
    "cpu_usage": 45.2
}
response = requests.post(
    "https://httpbin.org/post",
    json=payload
)
print(f"\nPOST status : {response.status_code}")


# ── Error handling ─────────────────────────────────────────────
try:
    response = requests.get("https://httpbin.org/status/404")
    response.raise_for_status()   # Raises exception for 4xx/5xx
except requests.exceptions.HTTPError as e:
    print(f"HTTP Error: {e}")
except requests.exceptions.ConnectionError as e:
    print(f"Connection Error: {e}")
except requests.exceptions.Timeout as e:
    print(f"Timeout Error: {e}")
except requests.exceptions.RequestException as e:
    print(f"Request Error: {e}")


# ── With timeout ───────────────────────────────────────────────
response = requests.get("https://httpbin.org/delay/1", timeout=5)
print(f"\nDelayed response: {response.status_code}")


# ── With headers (e.g., API authentication) ────────────────────
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE",
    "Content-Type": "application/json"
}
response = requests.get("https://httpbin.org/headers", headers=headers)
print(f"\nSent headers: {response.status_code}")
```

```bash
python3 requests_demo.py
```

---

## Task 2: Python in the Cloud

### Why Cloud SDKs Matter

Cloud providers expose their entire infrastructure through REST APIs. Doing this manually with raw `requests` calls would require constructing complex signed HTTP requests. Cloud SDKs handle all of that — authentication, request signing, pagination, retries — so you can interact with cloud services using clean Python objects and methods.

```
Without SDK:                    With SDK (boto3):
requests.get(                   import boto3
  "https://ec2.amazonaws.com",  ec2 = boto3.client('ec2')
  headers={...signed headers},  instances = ec2.describe_instances()
  params={...}
)
```

---

### Overview of Major Cloud SDKs

| Cloud | SDK | Install | Documentation |
|---|---|---|---|
| **AWS** | `boto3` | `pip install boto3` | [boto3.amazonaws.com](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) |
| **Google Cloud** | `google-cloud-*` | `pip install google-cloud-storage` | [cloud.google.com/python](https://cloud.google.com/python/docs) |
| **Azure** | `azure-*` | `pip install azure-mgmt-compute` | [docs.microsoft.com/python](https://docs.microsoft.com/azure/developer/python) |
| **DigitalOcean** | `pydo` | `pip install pydo` | [developers.digitalocean.com](https://developers.digitalocean.com) |

---

### `boto3` — AWS SDK for Python

`boto3` gives you Python objects that represent AWS resources — EC2 instances, S3 buckets, IAM users, Lambda functions — and methods to create, read, update, and delete them.

```bash
pip3 install boto3

# Configure AWS credentials (required before any boto3 call)
aws configure
# Or set environment variables:
# export AWS_ACCESS_KEY_ID=your_key
# export AWS_SECRET_ACCESS_KEY=your_secret
# export AWS_DEFAULT_REGION=ap-south-1
```

```python
# boto3_overview.py
# NOTE: Requires valid AWS credentials configured via aws configure

import boto3

# ── Two ways to use boto3 ──────────────────────────────────────
# 1. Client — low-level, returns raw dicts (maps to AWS API directly)
ec2_client = boto3.client('ec2', region_name='ap-south-1')

# 2. Resource — high-level, returns Python objects (more Pythonic)
ec2_resource = boto3.resource('ec2', region_name='ap-south-1')


# ── List all EC2 instances (client) ───────────────────────────
print("=== EC2 Instances ===")
response = ec2_client.describe_instances()

for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        instance_id = instance['InstanceId']
        state = instance['State']['Name']
        instance_type = instance['InstanceType']
        # Get the Name tag if it exists
        name = next(
            (tag['Value'] for tag in instance.get('Tags', [])
             if tag['Key'] == 'Name'),
            'No Name'
        )
        print(f"  {name:<20} {instance_id}  {instance_type}  [{state}]")


# ── List S3 buckets ────────────────────────────────────────────
print("\n=== S3 Buckets ===")
s3_client = boto3.client('s3')
response = s3_client.list_buckets()

for bucket in response['Buckets']:
    print(f"  {bucket['Name']}")


# ── Get current IAM user ───────────────────────────────────────
print("\n=== Current AWS Identity ===")
sts = boto3.client('sts')
identity = sts.get_caller_identity()
print(f"  Account : {identity['Account']}")
print(f"  User ARN: {identity['Arn']}")
```

---

### Cloud SDK Pattern — Common Operations

```python
# common_cloud_patterns.py

import boto3
import json
from datetime import datetime

# ── Upload a file to S3 ────────────────────────────────────────
def upload_to_s3(file_path, bucket_name, object_key):
    """Upload a local file to an S3 bucket."""
    s3 = boto3.client('s3')
    try:
        s3.upload_file(file_path, bucket_name, object_key)
        print(f"Uploaded {file_path} → s3://{bucket_name}/{object_key}")
    except Exception as e:
        print(f"Upload failed: {e}")


# ── Read a file from S3 ────────────────────────────────────────
def read_from_s3(bucket_name, object_key):
    """Download and return the content of an S3 object."""
    s3 = boto3.client('s3')
    response = s3.get_object(Bucket=bucket_name, Key=object_key)
    content = response['Body'].read().decode('utf-8')
    return content


# ── Start/stop an EC2 instance ─────────────────────────────────
def stop_instance(instance_id, region='ap-south-1'):
    """Stop an EC2 instance by ID."""
    ec2 = boto3.client('ec2', region_name=region)
    ec2.stop_instances(InstanceIds=[instance_id])
    print(f"Stopping instance: {instance_id}")


# ── Tag a resource ─────────────────────────────────────────────
def tag_resource(resource_id, tags, region='ap-south-1'):
    """Add tags to an AWS resource."""
    ec2 = boto3.client('ec2', region_name=region)
    ec2.create_tags(
        Resources=[resource_id],
        Tags=[{'Key': k, 'Value': v} for k, v in tags.items()]
    )
    print(f"Tagged {resource_id}: {tags}")
```

---

## Task 3: Working with JSON Data

### What Is JSON?

**JSON (JavaScript Object Notation)** is a lightweight, human-readable data format used for:
- REST API request and response bodies
- Configuration files (`package.json`, AWS CloudFormation templates)
- Data exchange between services
- Storing structured data

JSON maps directly to Python data types:

| JSON Type | Python Type | Example |
|---|---|---|
| Object `{}` | `dict` | `{"host": "web-01"}` |
| Array `[]` | `list` | `["web-01", "web-02"]` |
| String `""` | `str` | `"production"` |
| Number | `int` / `float` | `8080`, `73.5` |
| Boolean | `bool` | `true` → `True` |
| Null | `NoneType` | `null` → `None` |

---

### Creating the Mock JSON File

```bash
# Create a servers.json file to work with
cat > servers.json << 'EOF'
{
    "environment": "staging",
    "last_updated": "2024-06-10",
    "servers": [
        {
            "name": "web-01",
            "ip": "192.168.1.10",
            "status": "running",
            "cpu_usage": 45.2,
            "disk_usage": 62,
            "services": ["nginx", "node"]
        },
        {
            "name": "web-02",
            "ip": "192.168.1.11",
            "status": "running",
            "cpu_usage": 78.9,
            "disk_usage": 81,
            "services": ["nginx", "node"]
        },
        {
            "name": "db-01",
            "ip": "192.168.1.20",
            "status": "running",
            "cpu_usage": 33.1,
            "disk_usage": 55,
            "services": ["postgresql"]
        }
    ]
}
EOF
```

---

### Reading JSON

```python
# read_json.py
import json

# ── Read a JSON file ───────────────────────────────────────────
with open("servers.json", "r") as file:
    data = json.load(file)    # Parse JSON → Python dict

# Access top-level fields
print(f"Environment : {data['environment']}")
print(f"Last Updated: {data['last_updated']}")
print(f"Server count: {len(data['servers'])}")

# Iterate over the servers list
print("\nServer Status:")
print("-" * 45)
for server in data['servers']:
    print(f"  {server['name']:<10} | {server['ip']:<15} | "
          f"CPU: {server['cpu_usage']:>5.1f}% | "
          f"Status: {server['status']}")

# Access nested data
print("\nServices per server:")
for server in data['servers']:
    services = ", ".join(server['services'])
    print(f"  {server['name']}: {services}")
```

---

### Modifying and Writing JSON

```python
# modify_json.py
import json
from datetime import date

# ── Read existing data ─────────────────────────────────────────
with open("servers.json", "r") as file:
    data = json.load(file)

# ── Modify the data ────────────────────────────────────────────
# Update the timestamp
data["last_updated"] = str(date.today())

# Add a new server
new_server = {
    "name": "cache-01",
    "ip": "192.168.1.30",
    "status": "running",
    "cpu_usage": 22.5,
    "disk_usage": 40,
    "services": ["redis"]
}
data["servers"].append(new_server)

# Update an existing server's status
for server in data["servers"]:
    if server["name"] == "web-02":
        server["status"] = "maintenance"
        server["cpu_usage"] = 0.0

# ── Write back to file ─────────────────────────────────────────
with open("servers.json", "w") as file:
    json.dump(data, file, indent=4)    # indent=4 for pretty formatting

print("servers.json updated successfully.")
print(f"Total servers: {len(data['servers'])}")

# ── Verify by reading back ─────────────────────────────────────
with open("servers.json", "r") as file:
    verified = json.load(file)

print(f"\nVerification — server count: {len(verified['servers'])}")
for server in verified["servers"]:
    print(f"  {server['name']}: {server['status']}")
```

---

### JSON in Memory — Without Files

```python
# json_strings.py
import json

# Python dict → JSON string (serialize)
server_data = {
    "hostname": "web-01",
    "port": 80,
    "healthy": True,
    "tags": ["web", "production"]
}

json_string = json.dumps(server_data, indent=2)
print("JSON string:")
print(json_string)
print(f"Type: {type(json_string)}")  # <class 'str'>

# JSON string → Python dict (deserialize)
parsed = json.loads(json_string)
print(f"\nParsed hostname: {parsed['hostname']}")
print(f"Type: {type(parsed)}")      # <class 'dict'>

# Useful json.dumps options
compact = json.dumps(server_data, separators=(',', ':'))  # No whitespace
sorted_keys = json.dumps(server_data, sort_keys=True)     # Alphabetical keys
print(f"\nCompact: {compact}")
```

---

## Task 4: Other Data Formats

### CSV — Comma-Separated Values

CSV files are the standard format for tabular data — spreadsheets, database exports, inventory files, and reports.

```bash
# Create a mock CSV file
cat > servers.csv << 'EOF'
name,ip_address,environment,cpu_usage,disk_usage,status
web-01,192.168.1.10,production,45.2,62,running
web-02,192.168.1.11,production,78.9,81,running
db-01,192.168.1.20,production,33.1,55,running
cache-01,192.168.1.30,staging,22.5,40,running
dev-01,10.0.0.5,development,12.0,30,running
EOF
```

```python
# csv_reader.py
import csv

# ── Read CSV ───────────────────────────────────────────────────
print("Reading servers.csv:")
print("-" * 65)

with open("servers.csv", "r", newline="") as csvfile:
    reader = csv.DictReader(csvfile)    # Reads rows as dicts (uses header row)

    for row in reader:
        print(f"  {row['name']:<12} | {row['ip_address']:<15} | "
              f"ENV: {row['environment']:<12} | "
              f"CPU: {row['cpu_usage']:>5}% | "
              f"Status: {row['status']}")

# ── Read as list of lists ──────────────────────────────────────
print("\nRaw CSV data (list of lists):")
with open("servers.csv", "r", newline="") as csvfile:
    reader = csv.reader(csvfile)
    for i, row in enumerate(reader):
        if i == 0:
            print(f"  HEADERS: {row}")
        else:
            print(f"  Row {i}:  {row}")
```

```python
# csv_writer.py
import csv
from datetime import datetime

# ── Write a new CSV file ───────────────────────────────────────
report_data = [
    {"server": "web-01", "check_time": datetime.now().isoformat(),
     "cpu_status": "OK", "disk_status": "OK", "overall": "HEALTHY"},
    {"server": "web-02", "check_time": datetime.now().isoformat(),
     "cpu_status": "WARNING", "disk_status": "WARNING", "overall": "DEGRADED"},
    {"server": "db-01",  "check_time": datetime.now().isoformat(),
     "cpu_status": "OK", "disk_status": "OK", "overall": "HEALTHY"},
]

fieldnames = ["server", "check_time", "cpu_status", "disk_status", "overall"]

with open("health_report.csv", "w", newline="") as csvfile:
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
    writer.writeheader()            # Write the header row
    writer.writerows(report_data)   # Write all data rows

print("health_report.csv written successfully.")


# ── Append to existing CSV ─────────────────────────────────────
new_entry = {
    "server": "cache-01",
    "check_time": datetime.now().isoformat(),
    "cpu_status": "OK",
    "disk_status": "OK",
    "overall": "HEALTHY"
}

with open("health_report.csv", "a", newline="") as csvfile:
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
    writer.writerow(new_entry)

print("Appended cache-01 entry.")
```

---

### YAML — Yet Another Markup Language

YAML is the standard format for Kubernetes manifests, Ansible playbooks, Docker Compose files, and GitHub Actions pipelines.

```bash
pip3 install pyyaml
```

```python
# yaml_demo.py
import yaml

# ── Create a sample YAML config in Python ─────────────────────
config = {
    "application": {
        "name": "nexuscorp-api",
        "version": "2.1.0",
        "environment": "staging"
    },
    "database": {
        "host": "db.nexuscorp.internal",
        "port": 5432,
        "name": "nexusdb",
        "ssl": True
    },
    "replicas": 3,
    "resources": {
        "cpu": "500m",
        "memory": "256Mi"
    }
}

# ── Write to YAML file ─────────────────────────────────────────
with open("config.yaml", "w") as f:
    yaml.dump(config, f, default_flow_style=False)

print("config.yaml written.")

# ── Read YAML file ─────────────────────────────────────────────
with open("config.yaml", "r") as f:
    loaded = yaml.safe_load(f)

print(f"\nApp name   : {loaded['application']['name']}")
print(f"DB host    : {loaded['database']['host']}")
print(f"Replicas   : {loaded['replicas']}")
print(f"SSL enabled: {loaded['database']['ssl']}")
```

---

### XML — A Brief Introduction

```python
# xml_demo.py
import xml.etree.ElementTree as ET

# ── Create sample XML ──────────────────────────────────────────
xml_data = """<?xml version="1.0"?>
<servers>
    <server name="web-01">
        <ip>192.168.1.10</ip>
        <status>running</status>
        <cpu>45.2</cpu>
    </server>
    <server name="db-01">
        <ip>192.168.1.20</ip>
        <status>running</status>
        <cpu>33.1</cpu>
    </server>
</servers>"""

# ── Parse XML ──────────────────────────────────────────────────
root = ET.fromstring(xml_data)

print("Servers from XML:")
for server in root.findall("server"):
    name = server.get("name")
    ip = server.find("ip").text
    status = server.find("status").text
    cpu = server.find("cpu").text
    print(f"  {name:<10} | IP: {ip:<15} | CPU: {cpu}% | {status}")
```

---

### Format Comparison

| Format | Extension | Python Module | Best For |
|---|---|---|---|
| JSON | `.json` | `json` (stdlib) | APIs, config, data exchange |
| CSV | `.csv` | `csv` (stdlib) | Tabular data, reports, spreadsheets |
| YAML | `.yaml`, `.yml` | `pyyaml` (third-party) | Kubernetes, Ansible, Docker Compose |
| XML | `.xml` | `xml.etree` (stdlib) | Legacy systems, SOAP APIs, RSS |
| TOML | `.toml` | `tomllib` (stdlib 3.11+) | Python project config (`pyproject.toml`) |
| INI | `.ini`, `.cfg` | `configparser` (stdlib) | Application config files |

---

## Task 5: Exploring More Libraries

### `boto3` — AWS Automation in Practice

```bash
pip3 install boto3
```

```python
# boto3_exploration.py
# Explore boto3's structure without requiring real AWS credentials

import boto3
import json

# ── Explore boto3 structure ────────────────────────────────────
print("boto3 version:", boto3.__version__)
print("\nAvailable services (first 20):")
session = boto3.session.Session()
services = session.get_available_services()
for service in sorted(services)[:20]:
    print(f"  - {service}")

# ── What boto3 can do with EC2 ─────────────────────────────────
print("\nEC2 client methods (sample):")
ec2_client = boto3.client('ec2', region_name='ap-south-1')
ec2_methods = [m for m in dir(ec2_client) if not m.startswith('_')]
for method in ec2_methods[:15]:
    print(f"  - {method}")
```

---

### `docker` — Docker SDK for Python

```bash
pip3 install docker
```

```python
# docker_demo.py
import docker

# Connect to local Docker daemon
client = docker.from_env()

# List running containers
print("Running containers:")
for container in client.containers.list():
    print(f"  {container.name:<30} | {container.status} | {container.image.tags}")

# List all images
print("\nDocker images:")
for image in client.images.list():
    tags = image.tags if image.tags else ["<none>"]
    print(f"  {tags[0]:<40} | {image.short_id}")

# Pull an image
# client.images.pull("nginx:latest")

# Run a container
# container = client.containers.run(
#     "ubuntu",
#     "echo Hello from Python",
#     remove=True
# )
# print(container.decode())
```

---

### `psutil` — System and Process Utilities

```bash
pip3 install psutil
```

```python
# psutil_demo.py
import psutil

# CPU information
print(f"CPU cores    : {psutil.cpu_count()}")
print(f"CPU usage    : {psutil.cpu_percent(interval=1)}%")

# Memory information
mem = psutil.virtual_memory()
print(f"\nMemory total : {mem.total / (1024**3):.1f} GB")
print(f"Memory used  : {mem.used / (1024**3):.1f} GB ({mem.percent}%)")
print(f"Memory free  : {mem.available / (1024**3):.1f} GB")

# Disk information
disk = psutil.disk_usage('/')
print(f"\nDisk total   : {disk.total / (1024**3):.1f} GB")
print(f"Disk used    : {disk.used / (1024**3):.1f} GB ({disk.percent}%)")
print(f"Disk free    : {disk.free / (1024**3):.1f} GB")

# Network information
net = psutil.net_io_counters()
print(f"\nBytes sent   : {net.bytes_sent / (1024**2):.1f} MB")
print(f"Bytes recv   : {net.bytes_recv / (1024**2):.1f} MB")

# Top 5 processes by CPU
print("\nTop 5 processes by CPU:")
processes = []
for proc in psutil.process_iter(['pid', 'name', 'cpu_percent', 'memory_percent']):
    try:
        processes.append(proc.info)
    except psutil.NoSuchProcess:
        pass

top5 = sorted(processes, key=lambda x: x['cpu_percent'], reverse=True)[:5]
for proc in top5:
    print(f"  PID {proc['pid']:<6} | {proc['name']:<20} | "
          f"CPU: {proc['cpu_percent']:>5.1f}% | "
          f"Mem: {proc['memory_percent']:>4.1f}%")
```

---

## 🔧 Mini-Project: NexusCorp Data Pipeline

**Scenario:** Build a complete data pipeline that reads server inventory from JSON, processes it, outputs a health report as CSV, and archives the original JSON with a timestamp.

```python
#!/usr/bin/env python3
"""
data_pipeline.py
NexusCorp Server Data Pipeline
Demonstrates: json, csv, os, datetime, requests-style patterns
"""

import json
import csv
import os
import shutil
from datetime import datetime

# ── Configuration ──────────────────────────────────────────────
INPUT_FILE = "servers.json"
OUTPUT_CSV = "health_report.csv"
ARCHIVE_DIR = "archive"
CPU_WARN = 75
CPU_CRIT = 90
DISK_WARN = 80


# ── Helper Functions ───────────────────────────────────────────
def load_json(filepath):
    """Load and return JSON data from a file."""
    if not os.path.exists(filepath):
        raise FileNotFoundError(f"Input file not found: {filepath}")
    with open(filepath, "r") as f:
        return json.load(f)


def determine_status(server):
    """Determine overall server health status."""
    cpu = server.get("cpu_usage", 0)
    disk = server.get("disk_usage", 0)

    if cpu >= CPU_CRIT:
        return "CRITICAL"
    elif cpu >= CPU_WARN or disk >= DISK_WARN:
        return "WARNING"
    else:
        return "HEALTHY"


def process_servers(data):
    """Process server data and return report rows."""
    report = []
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    for server in data.get("servers", []):
        status = determine_status(server)
        report.append({
            "timestamp": timestamp,
            "environment": data.get("environment", "unknown"),
            "server_name": server["name"],
            "ip_address": server["ip"],
            "cpu_usage": server["cpu_usage"],
            "disk_usage": server["disk_usage"],
            "status": status,
            "services": ", ".join(server.get("services", []))
        })

    return report


def write_csv_report(report, filepath):
    """Write report data to a CSV file."""
    if not report:
        print("No data to write.")
        return

    fieldnames = list(report[0].keys())
    with open(filepath, "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(report)


def archive_input(filepath, archive_dir):
    """Archive the input file with a timestamp suffix."""
    os.makedirs(archive_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = os.path.basename(filepath)
    name, ext = os.path.splitext(filename)
    archive_path = os.path.join(archive_dir, f"{name}_{timestamp}{ext}")
    shutil.copy2(filepath, archive_path)
    return archive_path


def print_summary(report):
    """Print a human-readable summary to the terminal."""
    total = len(report)
    healthy = sum(1 for r in report if r["status"] == "HEALTHY")
    warning = sum(1 for r in report if r["status"] == "WARNING")
    critical = sum(1 for r in report if r["status"] == "CRITICAL")

    print("\n" + "=" * 55)
    print("  NexusCorp Data Pipeline — Summary")
    print("=" * 55)
    print(f"  Servers processed : {total}")
    print(f"  Healthy           : {healthy}")
    print(f"  Warning           : {warning}")
    print(f"  Critical          : {critical}")
    print("=" * 55)

    if warning > 0 or critical > 0:
        print("\n  ALERTS:")
        for row in report:
            if row["status"] != "HEALTHY":
                print(f"  ⚠  {row['server_name']}: {row['status']} "
                      f"(CPU: {row['cpu_usage']}%, Disk: {row['disk_usage']}%)")
    print()


# ── Main Pipeline ──────────────────────────────────────────────
def main():
    print("Starting NexusCorp Data Pipeline...")

    try:
        # Step 1: Load input data
        print(f"  Loading: {INPUT_FILE}")
        data = load_json(INPUT_FILE)

        # Step 2: Process
        print(f"  Processing {len(data.get('servers', []))} servers...")
        report = process_servers(data)

        # Step 3: Write CSV report
        print(f"  Writing report: {OUTPUT_CSV}")
        write_csv_report(report, OUTPUT_CSV)

        # Step 4: Archive input file
        archive_path = archive_input(INPUT_FILE, ARCHIVE_DIR)
        print(f"  Archived input: {archive_path}")

        # Step 5: Print summary
        print_summary(report)
        print(f"  Pipeline complete. Report saved to: {OUTPUT_CSV}")

    except FileNotFoundError as e:
        print(f"ERROR: {e}")
        print("Tip: Make sure servers.json exists in the current directory.")
    except json.JSONDecodeError as e:
        print(f"ERROR: Invalid JSON in input file — {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")


if __name__ == "__main__":
    main()
```

```bash
python3 data_pipeline.py
```

---

## Task 6: Final Reflection

### Three Key Takeaways Template

Spend 10–15 minutes writing your three takeaways from today and from the full Python series. Be specific — a vague takeaway ("Python is useful") is less valuable than a precise one ("JSON's direct mapping to Python dicts means I don't need to learn a new mental model to parse API responses").

```markdown
# Day 23 Reflection — Python Libraries, Cloud & Data Parsing

## Key Takeaway 1: [Something interesting]
What it is:
Why it matters for DevOps:
Where I'll use this first:

## Key Takeaway 2: [Something challenging]
What was hard:
How I worked through it:
What I understand now that I didn't before:

## Key Takeaway 3: [A new idea or connection]
The insight:
How it connects to something I already knew:
What I want to explore further:

## What I'll Build First
(One practical automation script I could write this week
using what I learned today)
```

---

## The Guided Learning Series — Complete

Today marks the final day of the NexusCorp DevOps Transformation guided curriculum. Here is what you have built across the full 23 days:

### Part I: Linux Fundamentals (Days 1–12)
| Day | Topic |
|---|---|
| 1–4 | Linux history, filesystem navigation, file operations |
| 5 | Text manipulation: `cat`, `echo`, `nano`, `vi` |
| 6 | Permissions & ownership: `chmod`, `chown`, SUID, SGID |
| 7 | Process management: `ps`, `top`, `kill`, `nice` |
| 8 | Package management: `apt`, `yum` |
| 9 | Networking basics & text processing: `ping`, `grep`, `awk`, `sed` |
| 10 | Shell scripting & task scheduling: Bash, `cron`, `at` |
| 11–12 | Filesystem hierarchy, logs, monitoring |

### Part II: Git & Collaboration (Days 13–18)
| Day | Topic |
|---|---|
| 13 | Version control systems — LVCS, CVCS, DVCS |
| 14 | Git setup, basic commands: `init`, `add`, `commit`, `status` |
| 15 | Remotes, branching, branching strategies |
| 16 | Merging and conflict resolution |
| 17 | Advanced Git: rebase, stash, reset, cherry-pick |
| 18 | Reflect, share, engage — learning in public |

### Part III: Networking & Cloud (Day 19)
| Day | Topic |
|---|---|
| 19 | Networking roadmap: DNS, TCP/UDP, NAT, firewalls, VPN, SSL/TLS, CDN |

### Part IV: Python for DevOps (Days 20–23)
| Day | Topic |
|---|---|
| 20 | Why Python matters for DevOps |
| 21 | Python setup, data types, first script |
| 22 | Variables, control flow, loops, functions, built-ins |
| 23 | Libraries, cloud SDKs, JSON, CSV, YAML |

---

## What Comes After This Series

The guided curriculum is complete — but the learning continues. The next chapter in the DevOps journey:

| Topic | Tools | What You'll Build |
|---|---|---|
| **Cloud Infrastructure** | AWS EC2, S3, VPC, IAM | Deploy and manage cloud resources |
| **Infrastructure as Code** | Terraform | Provision infrastructure from version-controlled config |
| **Containers** | Docker, Docker Compose | Package and run applications in containers |
| **Container Orchestration** | Kubernetes | Deploy and scale containerized applications |
| **CI/CD Pipelines** | GitHub Actions, Jenkins | Automate build, test, and deploy on every push |
| **Monitoring & Observability** | Prometheus, Grafana | Measure what's happening in production |
| **Configuration Management** | Ansible | Automate server configuration at scale |

Every skill you built in this series is a foundation for what comes next. Linux knowledge is needed to understand containers. Git is the trigger for every CI/CD pipeline. Python automates the infrastructure. Networking knowledge explains why your container can't reach the database.

The foundation is built. Keep building on it.

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Library** | A collection of pre-written code modules providing reusable functionality |
| **Standard library** | Libraries included with Python that require no installation |
| **Third-party library** | Libraries installed via `pip` from PyPI |
| **PyPI** | Python Package Index — the official repository of third-party Python packages |
| **`pip`** | Python's package installer — `pip install package-name` |
| **Virtual environment** | An isolated Python environment with its own packages, separate from the system Python |
| **`requirements.txt`** | A file listing all required packages for a project, used with `pip install -r` |
| **SDK** | Software Development Kit — a library providing access to a cloud or service API |
| **`boto3`** | The official AWS SDK for Python |
| **REST API** | An API that uses HTTP methods (GET, POST, PUT, DELETE) and returns JSON responses |
| **`requests`** | A third-party Python library for making HTTP requests |
| **JSON** | JavaScript Object Notation — lightweight data format; maps directly to Python dicts and lists |
| **`json.load()`** | Reads JSON from a file into a Python object |
| **`json.dump()`** | Writes a Python object to a file as JSON |
| **`json.loads()`** | Parses a JSON string into a Python object |
| **`json.dumps()`** | Converts a Python object to a JSON string |
| **CSV** | Comma-Separated Values — tabular data format |
| **`csv.DictReader`** | Reads CSV rows as dictionaries using the header row as keys |
| **`csv.DictWriter`** | Writes dictionaries to CSV with a header row |
| **YAML** | Yet Another Markup Language — human-readable format for configs (Kubernetes, Ansible) |
| **`pyyaml`** | Third-party Python library for reading and writing YAML |
| **XML** | Extensible Markup Language — structured document format |
| **`psutil`** | Third-party Python library for system and process information |
| **Docker SDK** | Third-party Python library for interacting with the Docker Engine API |
| **Data pipeline** | A sequence of processing steps that transforms raw data into a useful output format |

---

## 📚 Resources

- [Python Standard Library Reference](https://docs.python.org/3/library/index.html) — complete standard library documentation
- [PyPI — Python Package Index](https://pypi.org) — browse 400,000+ packages
- [boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) — AWS SDK for Python
- [Requests Library Documentation](https://requests.readthedocs.io/en/latest/) — HTTP for humans
- [Real Python — Working with JSON](https://realpython.com/python-json/) — comprehensive JSON tutorial
- [Python CSV Module Documentation](https://docs.python.org/3/library/csv.html) — official CSV reference
- [PyYAML Documentation](https://pyyaml.org/wiki/PyYAMLDocumentation) — YAML parsing reference
- [Automate the Boring Stuff — Chapter 16: CSV and JSON](https://automatetheboringstuff.com/2e/chapter16/) — free, practical guide

---

> 💬 *"The end of a guided curriculum is not the end of learning — it is the moment learning becomes self-directed. Everything you built in these 23 days is a tool. What you build with those tools is up to you."*
>
> — DevOps Learning by Yukta