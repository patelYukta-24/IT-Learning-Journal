# Day 55: Understanding Infrastructure as Code and Configuration Management

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. IaC vs. Configuration Management](#1-iac-vs-configuration-management)
- [2. IaC/CM Task — Blog Post](#2-iaccm-task--blog-post)
- [3. What is Ansible?](#3-what-is-ansible)
- [4. Ansible Tasks](#4-ansible-tasks)
- [5. Troubleshooting Notes](#5-troubleshooting-notes)
- [6. Key Learnings](#6-key-learnings)
- [7. Resources](#7-resources)

---

## Overview
Day 55 opens Part IX of the curriculum and shifts focus from AWS-native CI/CD services (Days 53–54) to the broader concepts of **Infrastructure as Code (IaC)** and **Configuration Management (CM)** — and introduces the first hands-on CM tool: **Ansible**. The day covers the conceptual distinction between IaC and CM, then moves into setting up an Ansible master/node architecture across EC2 instances.

## Topics Covered
- ❖ IaC vs. Configuration Management — differences, examples, common tools
- ❖ Task — write and publish a LinkedIn blog post on the topic
- ❖ What is Ansible
- ❖ Ansible Tasks — install on EC2 master, inventory/hosts file, node setup + ping test

---

## 1. IaC vs. Configuration Management
Both IaC and CM are core practices in modern cloud/DevOps workflows, and they're complementary rather than competing — but they solve different problems.

### 1.1 Infrastructure as Code (IaC)
IaC manages infrastructure — networks, virtual machines, load balancers, storage — using a **descriptive/declarative model** defined in code. Running the same IaC definition repeatedly produces the same infrastructure state every time, eliminating manual, error-prone provisioning through a console.

**Examples:** Terraform, AWS CloudFormation, Pulumi, Azure Resource Manager (ARM) templates

### 1.2 Configuration Management (CM)
CM manages the **state of software and configuration on already-provisioned systems** — installing packages, managing services, editing config files, ensuring servers stay in a consistent, desired state over time.

**Examples:** Ansible, Chef, Puppet, SaltStack

### 1.3 Side-by-Side Comparison
| Dimension | Infrastructure as Code (IaC) | Configuration Management (CM) |
|---|---|---|
| **What it manages** | The infrastructure itself (servers, networks, VPCs, load balancers) | Software/state *on* infrastructure that already exists |
| **Typical question answered** | "What resources should exist?" | "What state should this resource be in?" |
| **Example tools** | Terraform, CloudFormation, Pulumi | Ansible, Chef, Puppet, SaltStack |
| **Example task** | Provision a VPC, 2 EC2 instances, and a load balancer | Install Nginx, apply a config file, restart a service on those EC2 instances |
| **Execution model** | Mostly declarative (some tools support imperative) | Agent-based (Chef, Puppet) or agentless (Ansible) |

### 1.4 A Concrete Example
- **IaC step:** Terraform provisions 3 EC2 instances, a security group, and an Elastic IP
- **CM step:** Ansible then connects to those 3 instances and installs Docker, configures a firewall, and deploys an application config

In practice, the two are typically used together — IaC to stand up the infrastructure, then CM to configure what runs on it.

---

## 2. IaC/CM Task — Blog Post

**Objective:** Write a creative blog post covering IaC vs. CM differences, examples, and common tools, and publish it to LinkedIn.

**Suggested structure for the post:**
1. A relatable hook — e.g., the difference between *building* a house (IaC) and *furnishing/maintaining* it (CM)
2. Quick definitions of both, in plain language
3. A comparison table or bullet list (similar to §1.3 above)
4. Most common tools in each category, with a one-line note on what each is best known for:
   - **IaC:** Terraform (multi-cloud, most widely adopted), AWS CloudFormation (AWS-native), Pulumi (uses real programming languages)
   - **CM:** Ansible (agentless, YAML-based), Chef (Ruby DSL, agent-based), Puppet (declarative, agent-based)
5. A short personal takeaway — what clicked, or what's still confusing, from a self-study perspective
6. Relevant hashtags (e.g., `#DevOps #IaC #Ansible #90DaysOfDevOps #CloudComputing`)

> Status: pending — to be drafted and published to LinkedIn; link to the published post can be added here once live.

---

## 3. What is Ansible?
Ansible is an open-source, **agentless** automation tool used for configuration management, application deployment, orchestration, and provisioning. Because it doesn't require installing any agent software on managed nodes — it connects over SSH and uses Python already present on most Linux systems — it's often the easiest CM tool to get started with.

### 3.1 Key Ansible Concepts
| Concept | Description |
|---|---|
| **Control Node (Master)** | The machine where Ansible is installed and from which commands/playbooks are run |
| **Managed Node (Node/Host)** | A target machine Ansible connects to and configures — no agent required |
| **Inventory (Hosts file)** | A file (default: `/etc/ansible/hosts`) listing managed nodes, individually or grouped |
| **Module** | A unit of work Ansible executes (e.g., `ping`, `apt`, `copy`, `service`) |
| **Playbook** | A YAML file defining a set of tasks (using modules) to run against specified hosts |
| **Ad-hoc command** | A one-off Ansible command run directly from the CLI, without a full playbook |

---

## 4. Ansible Tasks

### Task 1 — Install Ansible on the Master Node (EC2)

**Step-by-step:**
1. Launch an EC2 instance (Ubuntu, `t2.micro`) — this will act as the **Ansible Control Node / Master**
2. SSH into it:
   ```bash
   ssh -i my-keypair.pem ubuntu@<master-public-ip>
   ```
3. Install Ansible:
   ```bash
   sudo apt-add-repository ppa:ansible/ansible
   sudo apt update
   sudo apt install -y ansible
   ```
4. Verify installation:
   ```bash
   ansible --version
   ```

> Status: reference implementation — pending execution + screenshot of `ansible --version` output.

---

### Task 2 — Understand the Hosts (Inventory) File

**Objective:** Read about and inspect Ansible's inventory file, which defines what hosts Ansible manages.

**Location:** `/etc/ansible/hosts` (the default inventory file, INI or YAML format)

**Viewing/editing it:**
```bash
sudo nano /etc/ansible/hosts
```

**Example inventory entries (INI format):**
```ini
[webservers]
node1 ansible_host=<node1-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/my-keypair.pem
node2 ansible_host=<node2-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/my-keypair.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
```
Grouping hosts (like `[webservers]`) allows a single playbook or ad-hoc command to target multiple nodes at once by group name, rather than listing each host individually every time.

**Viewing the inventory in a structured (JSON-like) format:**
```bash
ansible-inventory --list -y
```
> Note: the task text shows `--list-y`, but the correct flag combination is `--list` with `-y` (or `--yaml`) to output in YAML format — `ansible-inventory --list -y`.

> Status: reference implementation — pending execution + screenshot of the populated hosts file and `ansible-inventory --list -y` output.

---

### Task 3 — Set Up 2 Node Instances + SSH Key Sharing + Ping Test

**Objective:** Launch 2 more EC2 instances using the same key pair as the master, copy the private key to the master, and verify connectivity with an Ansible `ping`.

**Step 1: Launch 2 node instances**
- EC2 → Launch Instance → same AMI (Ubuntu) as the master
- **Key pair: reuse the exact same `.pem` key pair used for the master node** (this is what the task means by "same Private keys")
- Security group: allow inbound `22` (SSH) **from the master node's security group** (or its private IP), not just your own IP — the master needs to reach these nodes
- Launch both, name them `node1` and `node2`

**Step 2: Copy the private key onto the master node**
From your local machine:
```bash
scp -i my-keypair.pem my-keypair.pem ubuntu@<master-public-ip>:/home/ubuntu/.ssh/
```
Then, on the master node, fix permissions (SSH refuses to use overly permissive key files):
```bash
ssh ubuntu@<master-public-ip>
chmod 400 ~/.ssh/my-keypair.pem
```

**Step 3: Update the hosts file on the master with both nodes' private IPs**
```bash
sudo nano /etc/ansible/hosts
```
```ini
[nodes]
node1 ansible_host=<node1-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/my-keypair.pem
node2 ansible_host=<node2-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/my-keypair.pem
```

**Step 4: Test connectivity with Ansible's `ping` module**
```bash
ansible nodes -m ping
```
Expected output for each node:
```json
node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
> Note: Ansible's `ping` module is not an ICMP ping — it verifies Ansible can connect via SSH and successfully run a Python module on the remote host, which is a more meaningful check for automation purposes than a raw network ping.

> Status: reference implementation — pending execution + screenshot of `SUCCESS`/`pong` responses from both nodes.

---

## 5. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| `ansible nodes -m ping` fails with permission denied | Wrong key path in inventory, or key permissions too open | Confirm `ansible_ssh_private_key_file` path is correct and `chmod 400` was applied |
| Connection timed out reaching nodes | Node's security group doesn't allow inbound 22 from the master | Add an inbound rule on node SGs allowing SSH from the master's SG/private IP |
| `Host key verification failed` | First-time SSH connection to a new host, strict host key checking | SSH into each node manually once first (accept the host key), or set `host_key_checking = False` in `ansible.cfg` for lab use |
| `python: not found` on node | Older AMI missing Python 3 by default | Install `python3` on the node, or explicitly set `ansible_python_interpreter` in inventory |
| `apt-add-repository: command not found` | `software-properties-common` not installed | `sudo apt install -y software-properties-common` before adding the PPA |

---

## 6. Key Learnings
- IaC and CM aren't competing tools — they answer different questions ("what infrastructure should exist" vs. "what state should it be in") and are commonly used together in a real pipeline.
- Ansible's agentless design (SSH + Python, no software to install on managed nodes) is what makes it approachable to get started with compared to agent-based tools like Chef/Puppet.
- The inventory (hosts) file — and grouping hosts logically (e.g., `[webservers]`, `[nodes]`) — is the foundation everything else in Ansible builds on; playbooks target groups, not individual IPs hardcoded everywhere.
- Ansible's `ping` module tests actual Ansible connectivity (SSH + Python execution), not just network reachability — a more useful signal before running real automation against a host.
- Security groups need to allow the **master to reach the nodes**, not just allow inbound access from your own IP — an easy thing to miss when moving from single-instance to multi-instance setups.

## 7. Resources
- [Infrastructure as Code — AWS Overview](https://aws.amazon.com/what-is/iac/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Inventory Guide](https://docs.ansible.com/ansible/latest/inventory_guide/index.html)
- [Ansible `ping` Module Reference](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ping_module.html)

---
