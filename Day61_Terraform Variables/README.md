# Day 61 — Terraform Variables

## 📌 Overview

Day 61 introduces **Terraform variables** — how to define, store, and reuse values (instance names, config settings, etc.) across a Terraform project instead of hardcoding them directly into resource blocks. It covers creating a `variables.tf` file, referencing variables with the `var` object, using variables to create a local file resource, and exploring Terraform's collection data types: **List**, **Set**, **Map**, and **Object**.

---

## Introduction

Variables in Terraform are important because they let you hold and reuse values — instance names, configuration settings, and more — instead of repeating literal values throughout a configuration. A dedicated `variables.tf` file holds all variable declarations, and those variables are accessed in `main.tf` (or any other `.tf` file) via the `var` object, e.g. `var.filename`.

---

## Tasks / Exercises — Part 1

### 1. Create a local file using Terraform (with variables)

**`variables.tf`**
```hcl
variable "filename" {
  description = "Path of the local file to create"
  type        = string
  default     = "/home/ubuntu/day61_example.txt"
}

variable "file_content" {
  description = "Content to write into the local file"
  type        = string
  default     = "Hello from Terraform variables — Day 61!"
}
```

**`main.tf`**
```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.4"
    }
  }
}

resource "local_file" "example" {
  filename = var.filename
  content  = var.file_content
}
```

Run it:
```bash
terraform init
terraform plan
terraform apply
```

Verify:
```bash
cat /home/ubuntu/day61_example.txt
```

**Hint (Data Types in Terraform): MAP**

A **Map** is a collection of key-value pairs, where each key maps to a single value of the same type. Useful for things like tagging or looking up config values by name:
```hcl
variable "instance_tags" {
  type = map(string)
  default = {
    Name        = "day61-instance"
    Environment = "learning"
    Owner       = "Yukta"
  }
}
```
Accessed as `var.instance_tags["Name"]`.

---

## Data Types in Terraform

Terraform's type system includes:

| Type | Description |
|---|---|
| **String** | A single sequence of characters, e.g. `"nginx"` |
| **Number** | A numeric value, e.g. `2` |
| **Bool** | `true` or `false` |
| **List** | An ordered sequence of values, all of the same type. Values are accessed by index and duplicates are allowed. |
| **Set** | An unordered collection of unique values, all of the same type. No duplicates, no indexing by position. |
| **Map** | A collection of key-value pairs, all values of the same type, accessed by key. |
| **Object** | A structural type with named attributes, where each attribute can have its own (possibly different) type. |
| **Tuple** | An ordered sequence of values where each element can have a different type. |

---

## Tasks / Exercises — Part 2

### 1. Demonstrate List, Set, and Object datatypes

**`variables.tf`**
```hcl
# LIST — ordered, allows duplicates
variable "webserver_names" {
  description = "List of web server names"
  type        = list(string)
  default     = ["web-1", "web-2", "web-3"]
}

# SET — unordered, unique values only
variable "allowed_ports" {
  description = "Set of allowed inbound ports"
  type        = set(number)
  default     = [22, 80, 443]
}

# OBJECT — structural type with named, typed attributes
variable "app_config" {
  description = "Application configuration object"
  type = object({
    name        = string
    version     = string
    replicas    = number
    is_public   = bool
  })
  default = {
    name      = "day61-app"
    version   = "1.0.0"
    replicas  = 3
    is_public = true
  }
}
```

**`main.tf`** — using `local_file` resources to demonstrate the values (so the outputs can be verified without needing cloud resources):

```hcl
resource "local_file" "list_demo" {
  filename = "/home/ubuntu/day61_list_output.txt"
  content  = join(", ", var.webserver_names)
}

resource "local_file" "set_demo" {
  filename = "/home/ubuntu/day61_set_output.txt"
  content  = join(", ", [for p in var.allowed_ports : tostring(p)])
}

resource "local_file" "object_demo" {
  filename = "/home/ubuntu/day61_object_output.txt"
  content  = <<-EOT
    App Name : ${var.app_config.name}
    Version  : ${var.app_config.version}
    Replicas : ${var.app_config.replicas}
    Public   : ${var.app_config.is_public}
  EOT
}
```

**`outputs.tf`**
```hcl
output "webserver_names" {
  value = var.webserver_names
}

output "allowed_ports" {
  value = var.allowed_ports
}

output "app_config" {
  value = var.app_config
}
```

Run it:
```bash
terraform init
terraform plan
terraform apply
```

Sample CLI output:
```
Outputs:

allowed_ports = [
  22,
  80,
  443,
]
app_config = {
  "is_public" = true
  "name" = "day61-app"
  "replicas" = 3
  "version" = "1.0.0"
}
webserver_names = [
  "web-1",
  "web-2",
  "web-3",
]
```

📸 **Screenshots to attach in the repo:**
- Terminal output of `terraform apply` showing the List, Set, and Object outputs
- Contents of `day61_list_output.txt`, `day61_set_output.txt`, and `day61_object_output.txt`

### `terraform refresh`

`terraform refresh` reconciles the state file with the real infrastructure without changing the configuration — it reloads and updates variable/resource values in the state to reflect the actual current state of the infrastructure. It's useful when something may have changed outside of Terraform and you want the state file to catch up before running `plan`.

```bash
terraform refresh
```

> Note: In newer Terraform versions, `terraform plan -refresh-only` and `terraform apply -refresh-only` are the recommended replacements for standalone `terraform refresh`.

---

## 🧠 Key Learnings

- Separating variables into `variables.tf` keeps configuration values out of resource logic and makes a project reusable across environments
- **List** = ordered + duplicates allowed; **Set** = unordered + unique values only; **Map** = key-value lookup; **Object** = structured, named, typed attributes — choosing the right type keeps configs both valid and self-documenting
- The `local_file` + `local` provider combo is a simple, cloud-free way to practice and visually verify Terraform variable behavior
- `terraform refresh` (or `-refresh-only` flags) is the mechanism for syncing state with real-world drift without applying new changes

## ⚠️ Challenges Faced

- Remembering that `Set` values in Terraform have no guaranteed order, which affects how they should be iterated (`for` expressions) versus a `List`
- Getting the heredoc (`<<-EOT`) syntax right for formatting the Object demo's file output

## 🔗 Resources

- Terraform documentation — Variables
- Terraform documentation — Type Constraints (String, Number, Bool, List, Set, Map, Object, Tuple)
- Terraform `local` provider documentation

---
