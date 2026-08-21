# Day 59 — Exploring Terraform

## 📌 Overview

Day 59 shifts focus from configuration management (Ansible) to **Infrastructure as Code (IaC)** with **Terraform** — a tool for defining, provisioning, and managing infrastructure resources (VMs, networks, storage, and more) through declarative code rather than manual setup. The day covers installing Terraform, understanding its core concepts, and learning the purpose of its basic commands.

---

## What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool that allows you to create, manage, and update infrastructure resources — such as virtual machines, networks, and storage — in a repeatable, scalable, and automated way. Instead of manually clicking through a cloud console, infrastructure is described in configuration files and Terraform handles creating and updating the actual resources to match that description.

---

## Tasks / Exercises

### 1. Install Terraform

Installed Terraform on Ubuntu using HashiCorp's official APT repository:

```bash
sudo apt update && sudo apt install -y gnupg software-properties-common curl

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install -y terraform
```

Verify installation:
```bash
terraform -version
```

---

### 2. Conceptual Questions

**❖ Why do we use Terraform?**
Terraform lets infrastructure be defined as code — version-controlled, reviewable, and repeatable. It removes manual, error-prone console clicking, makes environments reproducible across dev/staging/prod, and works across multiple cloud providers (AWS, Azure, GCP, etc.) using a single consistent workflow.

**❖ What is Infrastructure as Code (IaC)?**
IaC is the practice of managing and provisioning infrastructure through machine-readable configuration files instead of manual processes. It brings software engineering practices — version control, code review, testing — to infrastructure management.

**❖ What is a Resource?**
A resource is the basic building block in Terraform — a single infrastructure object it manages, such as an EC2 instance, an S3 bucket, or a VPC. Resources are declared in `.tf` files using a `resource` block:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t2.micro"
}
```

**❖ What is a Provider?**
A provider is a plugin that lets Terraform talk to a specific platform's API — AWS, Azure, GCP, Docker, GitHub, etc. The provider translates Terraform configuration into actual API calls to create, read, update, and delete resources on that platform:
```hcl
provider "aws" {
  region = "ap-south-1"
}
```

**❖ What is a State file in Terraform? What's the importance of it?**
The state file (`terraform.tfstate`) is Terraform's record of what infrastructure it has actually created and how those real-world resources map to the resources declared in the configuration. It's important because:
- It lets Terraform know what already exists, so it can calculate what needs to change rather than recreating everything on every run
- It stores resource metadata and dependencies used for planning
- It should be protected and often stored remotely (e.g., S3 + DynamoDB lock) when working in a team, so multiple people don't overwrite each other's state

**❖ What is the Desired and Current State?**
- **Desired state** — what the `.tf` configuration files say infrastructure *should* look like
- **Current state** — what's actually recorded in the state file (and, by extension, what actually exists in the real infrastructure)

Terraform's core job is comparing desired vs. current state and generating a plan of the minimal changes needed to make current state match desired state.

---

### 3. Purpose of Basic Terraform Commands

| Command | Purpose |
|---|---|
| `terraform init` | Initializes a working directory — downloads the required providers and sets up the backend for state storage. Must be run first in any new Terraform project. |
| `terraform init -upgrade` | Re-initializes the directory and upgrades provider plugins to the latest version allowed by the configuration's version constraints. |
| `terraform plan` | Shows an execution plan — what Terraform *would* do (create, update, destroy) to move current state to desired state — without making any actual changes. |
| `terraform apply` | Executes the plan, creating/updating/destroying real infrastructure to match the configuration. Prompts for confirmation unless run with `-auto-approve`. |
| `terraform validate` | Checks whether the configuration files are syntactically valid and internally consistent, without contacting any provider or checking real infrastructure. |
| `terraform fmt` | Automatically reformats `.tf` files into Terraform's canonical style (consistent indentation, spacing) for readability and consistency across a team. |
| `terraform destroy` | Destroys all resources managed by the current Terraform configuration — the reverse of `apply`. Prompts for confirmation unless run with `-auto-approve`. |

---

## 🧠 Key Learnings

- Terraform's core value is turning infrastructure management into a repeatable, version-controlled process instead of manual console work
- Providers are the bridge between Terraform's generic language (HCL) and a specific platform's API
- The state file is the single most important artifact in a Terraform project — losing or corrupting it makes Terraform "forget" what it manages
- `plan` before `apply` is the safety net that lets you review changes before they touch real infrastructure — same spirit as `--check` in Ansible
- `validate` and `fmt` are cheap, fast sanity checks worth running before every `plan`/`apply`

## ⚠️ Challenges Faced

- Getting the HashiCorp GPG key and APT repository set up correctly on the first install attempt
- Understanding the distinction between "current state" (the state file) and the *actual* real-world infrastructure — they can drift apart if resources are changed outside Terraform

## 🔗 Resources

- Official Terraform installation guide (curriculum link)
- Free Terraform video course (curriculum link)
- Terraform documentation — Core concepts, State, Providers

---
