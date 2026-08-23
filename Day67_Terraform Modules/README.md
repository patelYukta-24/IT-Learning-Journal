# Day 67: Terraform Modules

## Table of Contents
- [Introduction](#introduction)
- [Module File/Folder Structure](#module-filefolder-structure)
- [How to Call a Module](#how-to-call-a-module)
- [Tasks / Exercises](#tasks--exercises)
  - [Task 1: Different Types of Modules in Terraform](#task-1-different-types-of-modules-in-terraform)
  - [Task 2: Root Module vs Child Module](#task-2-root-module-vs-child-module)
  - [Task 3: Are Modules and Namespaces the Same?](#task-3-are-modules-and-namespaces-the-same)
- [Key Takeaways](#key-takeaways)

---

## Introduction

A **module** in Terraform is simply a container that groups multiple resources together so they can be managed and reused as a single unit. Practically speaking, a module is nothing more than a directory containing one or more `.tf` (or `.tf.json`) configuration files.

Modules can reference other modules — a "parent" configuration can **call** a "child" module and pull its resources into the overall configuration. This is done through a `module` block. Because a module is just reusable configuration, it can be called more than once — either multiple times inside the same configuration (e.g. one module call per environment) or reused across entirely separate projects — which is what makes modules the primary way Terraform enables packaging and re-use of infrastructure patterns.

---

## Module File/Folder Structure

A typical module-based project looks something like this:

```
project/
├── main.tf              # Root module — calls child module(s)
├── variables.tf          # Root-level input variables
├── outputs.tf            # Root-level outputs
└── modules/
    └── server/
        ├── main.tf        # Child module — actual resource definitions
        ├── variables.tf   # Inputs the child module accepts
        └── outputs.tf     # Values the child module exposes back
```

## How to Call a Module

```hcl
module "server" {
  source        = "./modules/server"
  instance_type = "t2.micro"
  ami           = "ami-08c40ec9ead489470"
}
```

- `source` is the only required argument — it tells Terraform where to find the module (a local path, a Terraform Registry address, a Git URL, etc.).
- Any other arguments inside the block correspond to the input variables the child module declares.
- Values the child module exposes via its own `output` blocks can be referenced from outside as `module.server.<output_name>`.

---

## Tasks / Exercises

> The following are explained in my own words, based on general Terraform documentation and concepts — not copied from any single source.

### Task 1: Different Types of Modules in Terraform

Terraform doesn't have rigid "types" of modules baked into the language — any directory with `.tf` files is technically a module. But in practice, modules tend to fall into a few recognizable categories based on where they come from and how they're used:

1. **Root Module**
   The module Terraform starts from — the configuration in the directory where you run `terraform init`/`plan`/`apply`. Every Terraform configuration has exactly one root module, even if it never calls any child modules.

2. **Child Module**
   Any module that gets called from another module using a `module` block. A child module can itself call further child modules, so this can nest arbitrarily deep.

3. **Local Modules**
   Modules whose source is a path on your own filesystem (e.g. `source = "./modules/server"`). These are typically used for organizing a single project's own resources into logical, reusable chunks without needing to publish them anywhere.

4. **Registry / Remote Modules**
   Modules pulled in from an external source such as the public Terraform Registry, a private registry, a Git repository, or an S3/HTTP URL. These are typically pre-built, community- or vendor-maintained modules for common patterns (e.g. a VPC module, an EKS cluster module) that you don't want to write and maintain yourself.

5. **Published/Reusable Modules**
   Modules that a team deliberately designs to be generic and configurable — with well-defined input variables and outputs — so that the same module can be called repeatedly across many projects or environments (dev/staging/prod) with different input values each time, instead of duplicating resource blocks.

The common thread across all of these is that a "type" of module is really just describing **where it's called from** or **how it's sourced/reused** — the underlying mechanics (a directory of `.tf` files, called via a `module` block) are identical either way.

### Task 2: Root Module vs Child Module

| Aspect | Root Module | Child Module |
|---|---|---|
| What it is | The top-level configuration Terraform runs against directly | A module invoked by another module (root or another child) via a `module` block |
| How many exist | Exactly one per configuration | Zero, one, or many — can be nested |
| Where you run commands | `terraform init`, `plan`, `apply` are run from here | Never run commands directly inside a child module's folder as part of normal usage — it's consumed by whatever calls it |
| Purpose | Orchestrates the overall configuration — decides which child modules to call and with what inputs | Encapsulates a reusable, self-contained piece of infrastructure logic |
| Visibility of resources | Can see and reference outputs from any child module it calls | Cannot see anything in the root module or in sibling modules unless it's explicitly passed in as an input variable |

In short: the **root module is the entry point and orchestrator**, while a **child module is a reusable building block** the root (or another module) assembles into the full infrastructure. Think of the root module as the "main function" of a program, and child modules as the reusable functions/libraries it calls — each child module only knows what's passed into it as arguments, and only returns what it explicitly exposes as outputs.

### Task 3: Are Modules and Namespaces the Same?

**No — modules and namespaces are not the same thing, though they can look related on the surface.**

**Why they seem similar:** When you call a module, Terraform does give each resource inside it a distinct "address" that includes the module's instance name — e.g. `module.server.aws_instance.web` — which superficially resembles how a namespace prefixes an identifier to avoid collisions (like `std::vector` in C++ or `os.path` in Python). Because of this addressing behavior, it's easy to assume a module *is* a namespace.

**Why they're actually different:**

- **A module is a packaging/reuse mechanism.** Its primary purpose is to group a set of resources together so that whole group can be reused, configured through variables, and instantiated multiple times with different inputs. Its job is about *organizing and reusing infrastructure logic*.
- **A namespace is purely an identifier-scoping mechanism.** Its only job is to prevent naming collisions between things that would otherwise share the same name, with no concept of "inputs," "outputs," "resource lifecycle," or "reusable configuration."
- **A module has behavior and structure a namespace doesn't:** it accepts input variables, produces outputs, can call other modules, can be sourced from a registry/Git/local path, and has its own dependency graph inside it. A namespace has none of this — it's a flat scoping boundary, not a self-contained, configurable unit of infrastructure.
- **The address-prefixing effect is a side-effect of instantiation, not the purpose of modules.** Terraform prefixes resource addresses with `module.<name>` mainly so that when a module is called multiple times (e.g. `count` or `for_each` on a module block, or calling the same module twice with different names), each instance's resources can be uniquely tracked in state — that's a state-management necessity, not a namespacing feature designed for the user.

So while calling a module does incidentally *scope* the resource addresses beneath it (which is namespace-*like* behavior), a module is fundamentally a **unit of reusable, configurable infrastructure**, whereas a namespace is a **naming/scoping construct with no configuration or reuse semantics of its own**. Equating the two would be like calling a function the same thing as a variable scope — related, but not the same concept.

---

## Key Takeaways

- A module is just a directory of `.tf`/`.tf.json` files, grouped together so a set of resources can be managed and reused as a unit.
- Modules can call other modules and can be called multiple times, which is how Terraform enables reusable, DRY infrastructure code.
- Every configuration has exactly one root module (the entry point); everything it calls via `module` blocks is a child module.
- Modules and namespaces are not the same — modules are a reuse/configuration mechanism with inputs and outputs; namespaces are a pure identifier-scoping mechanism. Any resemblance is a side-effect of how Terraform addresses resources for state tracking.
