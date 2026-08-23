# Day 66: Meta-Arguments in Terraform

## Table of Contents
- [Introduction](#introduction)
- [Count](#count)
- [for_each](#for_each)
- [Count vs for_each — When to Use Which](#count-vs-for_each--when-to-use-which)
- [Tasks / Exercises](#tasks--exercises)
  - [Task 1: Demonstrate `count`](#task-1-demonstrate-count)
  - [Task 2: Demonstrate `for_each` (Set of Strings)](#task-2-demonstrate-for_each-set-of-strings)
  - [Task 3: Demonstrate `for_each` (Map — Key/Value Iteration)](#task-3-demonstrate-for_each-map--keyvalue-iteration)
- [Write-up: Meta-Arguments and Their Use in Terraform](#write-up-meta-arguments-and-their-use-in-terraform)
- [Key Takeaways](#key-takeaways)

---

## Introduction

When a `resource` block is defined in Terraform, by default it describes **one** resource to be created. Managing several copies of the same resource would normally mean copy-pasting the same block over and over, which is repetitive and hard to maintain.

Terraform solves this with **meta-arguments** — special arguments defined by the Terraform language itself (not by a provider) that can be used inside any `resource` or `module` block to change how that block behaves. Meta-arguments help meet cross-cutting requirements such as:

- Creating multiple instances of a resource (`count`, `for_each`)
- Controlling resource creation/destruction order (`depends_on`)
- Managing resource lifecycle behavior (`lifecycle`)
- Specifying a non-default provider configuration (`provider`)

This document focuses on the two meta-arguments used for **creating multiple resource instances from a single block**: `count` and `for_each`.

---

## Count

The `count` meta-argument accepts a **whole number** and creates that many instances of the resource. Each instance created this way is a **distinct infrastructure object** — Terraform tracks and manages each one separately, and each can be created, updated, or destroyed independently when the configuration is applied.

Inside a block that uses `count`, Terraform exposes `count.index` (a zero-based index of the current instance), which is commonly used to make each instance's attributes unique (e.g. naming).

### Example

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "server" {
  count         = 4
  ami           = "ami-08c40ec9ead489470"
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${count.index}"
  }
}
```

This creates **4** EC2 instances: `aws_instance.server[0]` through `aws_instance.server[3]`, named `Server 0`, `Server 1`, `Server 2`, and `Server 3`.

**Best suited for:** identical resources where the only thing that differs is an index-based label (e.g. spinning up N interchangeable servers).

---

## for_each

Like `count`, the `for_each` meta-argument also creates multiple instances of a resource or module from a single block. The difference is *how* the instances are specified: instead of a number, `for_each` accepts a **map** or a **set of strings**. Each instance is then keyed by that map key or set value rather than by a numeric index.

This makes `for_each` the better choice when the resources being created **aren't truly identical** — for example, when each one needs a distinct, meaningful value (such as each Active Directory group requiring a different owner, or each server requiring a different AMI).

Inside a block that uses `for_each`, Terraform exposes:
- `each.key` — the map key (or the set value, since for a set the key and value are the same)
- `each.value` — the map value (for a set, identical to `each.key`)

### Example 1 — `for_each` with a Set of Strings

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-east-1"
}

locals {
  ami_ids = toset([
    "ami-0b0dcb5067f052a63",
    "ami-08c40ec9ead489470",
  ])
}

resource "aws_instance" "server" {
  for_each = local.ami_ids

  ami           = each.key
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${each.key}"
  }
}
```

Here, Terraform creates one instance per AMI ID in the set, keyed by the AMI ID itself (`aws_instance.server["ami-0b0dcb5067f052a63"]`, etc.).

### Example 2 — `for_each` with a Map (Key/Value Iteration)

```hcl
locals {
  ami_ids = {
    "linux"  : "ami-0b0dcb5067f052a63",
    "ubuntu" : "ami-08c40ec9ead489470",
  }
}

resource "aws_instance" "server" {
  for_each = local.ami_ids

  ami           = each.value
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${each.key}"
  }
}
```

Here, each map **key** (`"linux"`, `"ubuntu"`) becomes a meaningful, human-readable resource key (`aws_instance.server["linux"]`), and each map **value** supplies the corresponding AMI ID. This produces clearly named, purpose-specific instances — `Server linux` and `Server ubuntu` — instead of anonymous index-numbered ones.

**Best suited for:** resources that differ from one another in some meaningful way, where a stable, descriptive key is more useful than a numeric index.

---

## Count vs for_each — When to Use Which

| Aspect | `count` | `for_each` |
|---|---|---|
| Input type | Whole number | Map or set of strings |
| Instance identifier | Numeric index (`count.index`) | Map key / set value (`each.key`, `each.value`) |
| Best for | Identical, interchangeable resources | Resources with distinct, meaningful attributes |
| Stability on list changes | Removing a middle item shifts all subsequent indices, which can cause unnecessary destroy/recreate of unrelated resources | Removing a map/set entry only affects that specific key — other resources are untouched |
| Readability of state addresses | `aws_instance.server[0]` | `aws_instance.server["ubuntu"]` |

In general, `for_each` is considered safer for long-lived infrastructure because instances are tracked by a stable key rather than a position-dependent index, which avoids the "everything after this shifts and gets recreated" problem that `count` can cause when an item is removed from the middle of a list.

---

## Tasks / Exercises

### Task 1: Demonstrate `count`

**Goal:** Provision 4 identical EC2 instances using the `count` meta-argument.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "server" {
  count         = 4
  ami           = "ami-08c40ec9ead489470"
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${count.index}"
  }
}

output "count_instance_ids" {
  value = aws_instance.server[*].id
}

output "count_instance_names" {
  value = [for s in aws_instance.server : s.tags.Name]
}
```

**Steps to apply:**

```bash
terraform init
terraform plan
terraform apply
```

**Expected result:** 4 instances created — `Server 0`, `Server 1`, `Server 2`, `Server 3` — addressable in state as `aws_instance.server[0]` … `aws_instance.server[3]`.

---

### Task 2: Demonstrate `for_each` (Set of Strings)

**Goal:** Provision one EC2 instance per AMI ID in a set, using `for_each`.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-east-1"
}

locals {
  ami_ids = toset([
    "ami-0b0dcb5067f052a63",
    "ami-08c40ec9ead489470",
  ])
}

resource "aws_instance" "server" {
  for_each = local.ami_ids

  ami           = each.key
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${each.key}"
  }
}

output "for_each_set_instance_ids" {
  value = { for k, s in aws_instance.server : k => s.id }
}
```

**Expected result:** 2 instances, one per AMI, addressable as `aws_instance.server["ami-0b0dcb5067f052a63"]` and `aws_instance.server["ami-08c40ec9ead489470"]`.

---

### Task 3: Demonstrate `for_each` (Map — Key/Value Iteration)

**Goal:** Provision one EC2 instance per entry in a map, using the map's key as a label and the map's value as the AMI ID.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-east-1"
}

locals {
  ami_ids = {
    "linux"  : "ami-0b0dcb5067f052a63",
    "ubuntu" : "ami-08c40ec9ead489470",
  }
}

resource "aws_instance" "server" {
  for_each = local.ami_ids

  ami           = each.value
  instance_type = "t2.micro"

  tags = {
    Name = "Server ${each.key}"
  }
}

output "for_each_map_instance_ids" {
  value = { for k, s in aws_instance.server : k => s.id }
}
```

**Expected result:** 2 instances, clearly labeled — `Server linux` (using `ami-0b0dcb5067f052a63`) and `Server ubuntu` (using `ami-08c40ec9ead489470`) — addressable as `aws_instance.server["linux"]` and `aws_instance.server["ubuntu"]`.

---

## Write-up: Meta-Arguments and Their Use in Terraform

Meta-arguments are arguments recognized by Terraform's core language itself, rather than by any specific provider. They can be applied to `resource` or `module` blocks to change *how* Terraform manages that block, independent of the resource type being configured. Because they're part of the language core, the same meta-arguments work consistently across every provider — an `aws_instance`, an `azurerm_virtual_machine`, and a `google_compute_instance` can all use `count`, `for_each`, `depends_on`, `provider`, and `lifecycle` in exactly the same way.

The main meta-arguments in Terraform are:

- **`count`** — Creates a specified whole number of instances of a resource or module. Each instance is tracked by a numeric index (`count.index`). Useful when several identical copies of a resource are needed.
- **`for_each`** — Creates one instance per element of a given map or set of strings. Each instance is tracked by its map key (or set value) rather than a numeric index, which makes it well suited to resources that need distinct, meaningful configuration values (like different owners, AMIs, or environment names).
- **`depends_on`** — Explicitly declares a dependency on another resource or module, for situations where Terraform cannot automatically infer the dependency from configuration references alone.
- **`lifecycle`** — A nested block used to customize how a resource is created, updated, or destroyed, using arguments like `create_before_destroy`, `prevent_destroy`, and `ignore_changes`.
- **`provider`** — Selects a non-default provider configuration for a resource, useful when working with multiple configurations of the same provider (e.g. multiple AWS regions or accounts).

The two multi-instance meta-arguments, `count` and `for_each`, exist specifically to remove the need to hand-write a separate, near-identical `resource` block for every copy of infrastructure needed. Instead of duplicating a 10-line `resource` block four times to get four servers, a single block with `count = 4` achieves the same result. This:

- **Reduces code duplication and overhead** — one block instead of many.
- **Improves maintainability** — a change (like updating the instance type) is made in exactly one place and applies to every instance.
- **Keeps configuration scalable** — going from 4 servers to 40 is a one-line change with `count`, or an update to the input map/set with `for_each`.
- **Preserves per-instance manageability** — even though instances share a block, Terraform still tracks each one as a distinct object in state, so any single instance can be independently updated, tainted, or destroyed.

Choosing between them comes down to the *nature* of the resources being created. `count` fits scenarios where instances are essentially interchangeable and only need a numeric label to tell them apart. `for_each` fits scenarios where each instance genuinely differs in some way — different AMIs, different owners, different environment names — and benefits from being addressed by a stable, descriptive key instead of a position in a list. This also makes `for_each` more resilient to configuration changes: since instances are keyed by value rather than position, removing one entry from the middle of a map or set does not cause Terraform to shift and unnecessarily recreate every instance that follows it, which is a common pitfall with `count`.

---

## Key Takeaways

- Meta-arguments are core Terraform language features (not provider features) that change how a `resource` or `module` block behaves.
- `count` creates N identical instances, indexed numerically via `count.index`.
- `for_each` creates one instance per map/set entry, keyed by `each.key` / `each.value`, and is better suited to resources with distinct configuration values.
- Both meta-arguments eliminate repetitive, duplicated `resource` blocks — reducing overhead and improving code readability and maintainability.
- `for_each` is generally more stable than `count` when the underlying list of resources changes over time, since it avoids index-shifting side effects.
