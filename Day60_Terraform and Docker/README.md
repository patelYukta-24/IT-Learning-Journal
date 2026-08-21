# Day 60 — Terraform and Docker

## 📌 Overview

Day 60 builds directly on Day 59's Terraform basics by applying them to a real provider — **Docker**. The focus is on understanding Terraform's core building blocks (the **Provider Block** and **Resource Block**) and using them to define, pull, and run an Nginx Docker container entirely through Terraform code instead of manual `docker run` commands.

---

## Blocks and Resources in Terraform

### Provider Block

The provider block configures the specific provider Terraform will use — in this case, Docker. A provider is a plugin Terraform uses to create and manage resources on a given platform. Terraform needs to be told which provider to use, including its **source** and **version**.

`kreuzwerker/docker` is shorthand for `registry.terraform.io/kreuzwerker/docker`.

**`main.tf` — Terraform + Provider block:**
```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}
```

### Resource

Resource blocks define components of the infrastructure being managed — a physical or virtual component such as a Docker container, or a logical resource such as a Heroku application. A resource block has two strings before its body: the **resource type** and the **resource name**. For example, `docker_image` is the resource type and `nginx` is the resource name.

---

## Tasks / Exercises

### 1. Create a Terraform script with Blocks and Resources

**`main.tf`**
```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}
```

Run it:
```bash
terraform init
terraform validate
terraform fmt
terraform plan
terraform apply
```

This pulls the `nginx:latest` image via Docker, managed as Terraform state.

---

### 2. Create a resource block for running an Nginx Docker container

#### Install Terraform (if not already installed)

Refer to the official Terraform installation guide (same steps as Day 59 — HashiCorp APT repository).

#### Install Docker (if not already installed)

```bash
sudo apt-get install docker.io
sudo docker ps
sudo chown $USER /var/run/docker.sock
```

The `chown` step lets the current user talk to the Docker socket without needing `sudo` for every Docker/Terraform command.

#### Full `main.tf` — image + running container

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  name  = "day60-nginx"
  image = docker_image.nginx.image_id

  ports {
    internal = 80
    external = 8080
  }
}
```

Apply it:
```bash
terraform init
terraform plan
terraform apply
```

Verify the container is running:
```bash
docker ps
curl http://localhost:8080
```

Tear it down when done:
```bash
terraform destroy
```

---

## 🧠 Key Learnings

- The **provider block** tells Terraform which plugin (and version) to use to talk to a platform — here, Docker instead of a cloud API
- The **resource block**'s two labels (type + name) uniquely identify a managed object — `docker_image.nginx` and `docker_container.nginx` are separate resources that can reference each other
- Resources can depend on each other implicitly — `docker_container` references `docker_image.nginx.image_id`, so Terraform automatically creates the image before the container
- Managing Docker containers through Terraform brings the same benefits as managing cloud infrastructure: `plan` shows what will change, state tracks what's running, and `destroy` cleanly tears everything down
- The `chown $USER /var/run/docker.sock` step is a common fix for Docker permission errors without resorting to `sudo` everywhere

## ⚠️ Challenges Faced

- Initial `terraform apply` failed with a Docker socket permission error until `sudo chown $USER /var/run/docker.sock` was applied
- Understanding that `docker_image` and `docker_container` are two separate resources — forgetting the image resource means the container resource has nothing to reference

## 🔗 Resources

- Terraform installation guide (curriculum link)
- Free Terraform video course (curriculum link)
- Terraform Docker Provider documentation — `kreuzwerker/docker`

---
