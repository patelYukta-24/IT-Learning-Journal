# Day 25: Docker Installation & First Steps

> **DevOps Transformation** | Containers & Automation Series

---

## 📋 Overview

Yesterday you built the mental model — containers, images, Docker's purpose. Today you install Docker and run your first container.

This session covers installation across all major platforms, the recommended Linux setup using an AWS EC2 instance, post-installation configuration (including the critical `docker group` step), and your first three Docker commands. By the end of today, Docker will be running on your machine and you will have confirmed it with a live container.

---

## 🗺️ What You'll Do Today

| Task | Activity | Goal |
|---|---|---|
| Install Docker | Platform-specific installation | Docker running on your machine |
| Post-install configuration | Add user to docker group | Run Docker without `sudo` |
| Verify installation | `docker --version` | Confirm Docker is correctly installed |
| Run Hello World | `docker run hello-world` | Confirm Docker can pull and run containers |
| List images | `docker images` | See your local image cache |

---

## 🏢 NexusCorp Scenario

> Every new engineer at NexusCorp gets Docker installed on their workstation on day one. The operations team standardizes on Linux EC2 instances for all Docker work because it eliminates virtualization overhead and gives the purest Docker experience. Today you set up that environment — the same way the NexusCorp ops team does it.

---

## Part 1: Docker Installation

Docker is platform-independent — it runs on Windows, macOS, and Linux. The installation method and underlying behavior differ slightly per platform. Choose the section that matches your environment.

---

### Option 1: Docker Desktop for Windows

**Requirements:**
- Windows 10 64-bit: Pro, Enterprise, or Education (Build 19041 or higher)
- Windows 11 64-bit: Home, Pro, Enterprise, or Education
- WSL 2 (Windows Subsystem for Linux 2) — recommended backend
- Hardware virtualization enabled in BIOS

**Installation steps:**

```
1. Visit: https://docs.docker.com/desktop/install/windows-install/
2. Download Docker Desktop Installer.exe
3. Run the installer
   → When prompted, ensure "Use WSL 2 instead of Hyper-V" is checked (recommended)
4. Follow the installation wizard
5. Restart your computer when prompted
6. Launch Docker Desktop from the Start menu
7. Complete the initial setup wizard
```

**Verify in Command Prompt or PowerShell:**
```powershell
docker --version
docker run hello-world
```

**Notes:**
- Docker Desktop includes Docker Engine, Docker CLI, Docker Compose, and a GUI dashboard
- WSL 2 backend gives Linux-like performance and is the recommended mode
- Docker Desktop for Windows requires a free account for personal use

---

### Option 2: Docker Desktop for Mac

**Requirements:**
- macOS 12 (Monterey) or newer recommended
- At least 4 GB RAM
- Apple Silicon (M1/M2/M3) or Intel processor

**Installation steps:**

```
1. Visit: https://docs.docker.com/desktop/install/mac-install/
2. Choose your chip type:
   → Apple Silicon: Download Docker Desktop for Mac with Apple Silicon
   → Intel chip: Download Docker Desktop for Mac with Intel chip
3. Open the downloaded .dmg file
4. Drag Docker to the Applications folder
5. Launch Docker from Applications
6. Authorize with your system password when prompted
7. Complete the initial setup
```

**Verify in Terminal:**
```bash
docker --version
docker run hello-world
```

**Notes:**
- Docker Desktop for Mac runs a lightweight Linux VM under the hood to host containers
- Apple Silicon Macs run Docker natively — excellent performance
- Intel Macs may see slightly more overhead due to x86 emulation for some images

---

### Option 3: Docker on Linux (Recommended for DevOps)

Linux is Docker's **native environment**. There is no virtualization layer — Docker runs containers directly on the Linux kernel. This is the most efficient, most production-representative way to run Docker, and it is what every cloud server runs.

**Recommended approach: AWS EC2 Instance**

Using an EC2 instance gives you:
- A clean, fresh Linux environment every time
- No interference from your local machine's configuration
- The same environment you will use in production
- Free tier eligible (t2.micro or t3.micro with Amazon Linux or Ubuntu)

---

#### Install Docker on Ubuntu / Debian (EC2 or local)

```bash
# Step 1: Update package index
sudo apt update

# Step 2: Install prerequisite packages
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Step 3: Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Step 4: Add Docker's repository to apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 5: Update package index again (now includes Docker's repo)
sudo apt update

# Step 6: Install Docker Engine, CLI, and Compose plugin
sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin

# Step 7: Verify Docker service is running
sudo systemctl status docker
```

---

#### Install Docker on Amazon Linux 2 / Amazon Linux 2023 (EC2)

```bash
# Amazon Linux 2
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker   # Start Docker on boot

# Amazon Linux 2023
sudo dnf update -y
sudo dnf install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```

---

#### Install Docker on RHEL / CentOS / Fedora

```bash
# Step 1: Remove old versions if present
sudo yum remove docker \
    docker-client docker-client-latest \
    docker-common docker-latest \
    docker-latest-logrotate docker-logrotate docker-engine

# Step 2: Install yum-utils
sudo yum install -y yum-utils

# Step 3: Add Docker repo
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# Step 4: Install Docker
sudo yum install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# Step 5: Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
```

---

#### Quick Install — Convenience Script (Testing Only)

Docker provides a convenience script for quick installation in non-production environments:

```bash
# Download and run Docker's install script
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

**⚠️ Warning:** Never use this convenience script on production systems. It installs the latest version without version pinning and should only be used for learning and testing environments.

---

### Setting Up an EC2 Instance for Docker

If you are using AWS, here is the recommended setup:

```
1. Log into AWS Console → EC2 → Launch Instance
2. Choose AMI:
   → Ubuntu Server 22.04 LTS (recommended for learning)
   → Or: Amazon Linux 2023
3. Instance type: t2.micro or t3.micro (free tier eligible)
4. Key pair: Create or select an existing key pair for SSH access
5. Security Group: Allow SSH (port 22) from your IP
6. Launch the instance
7. Connect via SSH:
   ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
8. Follow the Ubuntu Docker installation steps above
```

---

## Part 2: Post-Installation Steps for Linux

### The `sudo` Problem

After installing Docker on Linux, every Docker command requires `sudo`:

```bash
# Without configuration — requires sudo every time
sudo docker run hello-world
sudo docker images
sudo docker ps
```

This happens because the Docker daemon (`dockerd`) runs as root, and only root (and members of the `docker` group) can communicate with it by default.

For a development environment, prepending every command with `sudo` is tedious and error-prone. The solution is to add your user to the `docker` group.

---

### Add Your User to the Docker Group

```bash
# Step 1: Add the current user to the docker group
sudo usermod -aG docker $USER

# Breaking this down:
# sudo        → run with superuser privileges
# usermod     → modify a user account
# -aG         → -a = append (don't remove other groups)
#               G = specify a supplementary group
# docker      → the group to add the user to
# $USER       → environment variable containing your username

# Step 2: Apply the group change (choose one method)

# Method A: Log out and log back in (full session restart)
exit
# SSH back in, or log back into your desktop session

# Method B: Apply immediately in current session
newgrp docker

# Step 3: Verify the change
groups
# Output should include: docker
# Example: ubuntu adm dialout cdrom floppy sudo audio dip video docker
```

---

### Verify the Docker Group Was Added

```bash
# Check your group memberships
id $USER
# Output: uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),4(adm),...,999(docker)

# Confirm docker group exists on the system
cat /etc/group | grep docker
# Output: docker:x:999:ubuntu

# Test: run docker WITHOUT sudo
docker run hello-world
# If this works without sudo, the group was added correctly
```

---

### Why Add Your User to the Docker Group?

**The reason:**
By default, the Docker socket (`/var/run/docker.sock`) is owned by root and the docker group. Only root and docker group members can write to this socket — which is how you communicate with the Docker daemon.

```bash
# See the socket permissions
ls -la /var/run/docker.sock
# srw-rw---- 1 root docker 0 Jun 10 14:00 /var/run/docker.sock
#                   ^^^^^^
#                   docker group has read/write access
```

**The security trade-off:**

Adding a user to the docker group is convenient but carries a security implication worth understanding:

| | Without docker group | With docker group |
|---|---|---|
| Run Docker commands | Requires `sudo` every time | No `sudo` needed |
| Security level | Higher — explicit privilege escalation | Lower — equivalent to root in practice |
| Production servers | Recommended | Use with caution |
| Development machines | Acceptable with sudo | Acceptable — convenience wins |

**The risk explained:** A user in the docker group can run:
```bash
docker run -v /:/host -it ubuntu bash
# This mounts the host filesystem inside the container
# giving effectively unrestricted root access to the host
```

This is why the Docker documentation describes docker group membership as equivalent to passwordless sudo. On a personal development machine or EC2 learning instance, this is acceptable. On a shared production server, use `sudo` for Docker commands or configure rootless Docker.

---

### Enable Docker to Start on Boot

```bash
# Enable Docker to start automatically when the system boots
sudo systemctl enable docker

# Start Docker now if it isn't already running
sudo systemctl start docker

# Check Docker service status
sudo systemctl status docker
# Look for: Active: active (running)
```

---

## Part 3: Practical Tasks

### Task 1: Verify Docker Installation

```bash
# Check the installed Docker version
docker --version
```

**Expected output:**
```
Docker version 24.0.5, build ced0996
```

Your version number will differ — any version 20+ is fine for all learning exercises.

**More detailed version information:**
```bash
# Get full version info including server
docker version

# Output includes:
# Client: Docker Engine - Community
#  Version:           24.0.5
#  API version:       1.43
#  ...
# Server: Docker Engine - Community
#  Engine:
#   Version:          24.0.5
#   ...
```

**Check Docker system information:**
```bash
docker info
```

This shows:
- Number of containers (running, paused, stopped)
- Number of images
- Storage driver in use
- Operating system and kernel version
- Total memory
- Docker root directory

---

### Task 2: Run Hello World

```bash
docker run hello-world
```

**What happens behind the scenes:**

```
Step 1: Docker CLI receives the command
         "run the container named hello-world"

Step 2: Docker checks local image cache
         → Is "hello-world" image stored locally?
         → Not found (first time running)

Step 3: Docker pulls the image from Docker Hub
         Unable to find image 'hello-world:latest' locally
         latest: Pulling from library/hello-world
         ...

Step 4: Docker creates a container from the image

Step 5: Docker runs the container
         → Container executes its startup command
         → Prints the Hello World message
         → Container exits (it has nothing else to do)
```

**Full expected output:**
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
719385e32844: Pull complete
Digest: sha256:4f53e2564790c8e7856ec08e384732aa38dc43c52f02952483e3f003afbf23db
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**If you run `docker run hello-world` a second time:**
```
# Docker finds the image in local cache — no pull needed
Hello from Docker!
...
```

This demonstrates image caching — once pulled, the image is stored locally and reused.

---

### Task 3: List Docker Images

```bash
docker images
```

**Expected output after running hello-world:**
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d2c94e258dcb   2 months ago   13.3kB
```

**Column breakdown:**

| Column | Meaning |
|---|---|
| `REPOSITORY` | Image name — the repository on Docker Hub |
| `TAG` | Version label — `latest` is the default |
| `IMAGE ID` | Short SHA hash uniquely identifying this image |
| `CREATED` | When this image was built (not when you pulled it) |
| `SIZE` | Disk space used by this image |

**Additional image commands to explore:**

```bash
# List all images including intermediate build layers
docker images -a

# Show only image IDs (useful for scripting)
docker images -q

# Filter images by name
docker images hello-world

# Show full (non-truncated) image IDs
docker images --no-trunc

# Format output as JSON
docker images --format "{{json .}}"

# Custom format — table view
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

---

### Explore Further: Try More Containers

Once hello-world works, experiment with real containers from Docker Hub:

```bash
# Run an Ubuntu container interactively
# -it = interactive terminal
# bash = the command to run inside the container
docker run -it ubuntu bash
# You're now INSIDE the container
# Try: ls, pwd, cat /etc/os-release
# Type 'exit' to leave the container

# Run nginx web server
# -d = detached (run in background)
# -p = map host port 8080 to container port 80
docker run -d -p 8080:80 nginx
# Visit: http://localhost:8080 in your browser

# Run a Python script inside a container
docker run python:3.11 python -c "print('Hello from Python in Docker!')"

# Run Alpine Linux (tiny, only 7MB)
docker run -it alpine sh

# List running containers
docker ps

# List ALL containers (including stopped)
docker ps -a

# Stop a running container
docker stop <container_id_or_name>

# Remove a stopped container
docker rm <container_id_or_name>

# Remove an image
docker rmi hello-world
```

---

### Troubleshooting Common Issues

| Problem | Error Message | Solution |
|---|---|---|
| Docker not running | `Cannot connect to the Docker daemon` | `sudo systemctl start docker` |
| Permission denied | `permission denied while trying to connect` | Add user to docker group, then log out and back in |
| Port already in use | `Bind for 0.0.0.0:8080 failed: port is already allocated` | Use a different host port: `-p 8081:80` |
| Image not found | `Unable to find image locally` | This is normal — Docker will pull from Hub automatically |
| No space left | `no space left on device` | `docker system prune` to clean up unused images/containers |
| DNS failure on pull | `Error response from daemon: Get https://registry-1.docker.io` | Check internet connectivity, DNS settings |

**General debug commands:**
```bash
# Check Docker service status
sudo systemctl status docker

# View Docker daemon logs
sudo journalctl -u docker -f

# Check Docker disk usage
docker system df

# Clean up unused resources (use with caution)
docker system prune

# Clean up including unused images
docker system prune -a
```

---

## 🔧 Mini-Project: Your First Docker Workflow

Walk through the complete Docker workflow — pull, inspect, run, interact, stop, clean up.

```bash
# ── Step 1: Pull an image without running it ────────────────
docker pull nginx
# Observe each layer being downloaded

# ── Step 2: Inspect the image ───────────────────────────────
docker images nginx
docker inspect nginx | head -50    # Show first 50 lines of metadata

# ── Step 3: Run the container ───────────────────────────────
docker run -d --name my-nginx -p 8080:80 nginx
# -d = detach (background)
# --name = give it a memorable name
# -p 8080:80 = host port 8080 → container port 80

# ── Step 4: Confirm it's running ────────────────────────────
docker ps
# You should see my-nginx listed with status "Up"

# ── Step 5: Test it works ───────────────────────────────────
curl http://localhost:8080
# Should return the nginx welcome page HTML

# ── Step 6: Inspect the running container ───────────────────
docker inspect my-nginx | grep IPAddress
docker logs my-nginx           # View nginx access logs
docker stats my-nginx --no-stream  # One-time resource usage snapshot

# ── Step 7: Execute a command inside the running container ──
docker exec -it my-nginx bash
# You're inside the nginx container
ls /etc/nginx/           # Nginx config files
cat /etc/nginx/nginx.conf
exit

# ── Step 8: Stop the container ──────────────────────────────
docker stop my-nginx
docker ps                  # Should no longer appear
docker ps -a               # Appears as "Exited"

# ── Step 9: Start it again ──────────────────────────────────
docker start my-nginx
docker ps                  # Running again

# ── Step 10: Clean up ───────────────────────────────────────
docker stop my-nginx
docker rm my-nginx         # Remove container
docker rmi nginx           # Remove image

# Verify everything is cleaned up
docker ps -a
docker images
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Docker Engine** | The core server-side component that runs and manages containers |
| **Docker Desktop** | GUI application for Windows and macOS that includes Docker Engine, CLI, and Compose |
| **Docker CE** | Community Edition — the free, open-source version of Docker |
| **Docker daemon (`dockerd`)** | The background service that manages Docker objects (images, containers, networks, volumes) |
| **Docker socket** | The Unix socket (`/var/run/docker.sock`) through which the CLI communicates with the daemon |
| **docker group** | A Linux user group whose members can run Docker commands without `sudo` |
| **`sudo usermod -aG docker $USER`** | Command to add the current user to the docker group |
| **`newgrp docker`** | Activates group membership changes in the current shell session without logging out |
| **`docker run`** | Creates and starts a container from an image |
| **`docker images`** | Lists all locally stored images |
| **`docker ps`** | Lists currently running containers |
| **`docker ps -a`** | Lists all containers including stopped ones |
| **`docker pull`** | Downloads an image from a registry without running it |
| **`docker stop`** | Gracefully stops a running container (sends SIGTERM, then SIGKILL) |
| **`docker rm`** | Removes a stopped container |
| **`docker rmi`** | Removes a locally stored image |
| **`docker exec`** | Runs a command inside an already-running container |
| **`docker logs`** | Displays the stdout/stderr output of a container |
| **`docker inspect`** | Returns detailed JSON metadata about a container or image |
| **`docker stats`** | Displays live resource usage statistics for running containers |
| **`docker system prune`** | Removes all stopped containers, dangling images, and unused networks |
| **`-d` flag** | Detached mode — runs the container in the background |
| **`-it` flags** | Interactive (`-i`) + TTY (`-t`) — enables interactive terminal access |
| **`-p host:container`** | Port mapping — forwards traffic from host port to container port |
| **`--name`** | Assigns a human-readable name to a container |
| **Image cache** | The local store of pulled images — Docker checks here before pulling from a registry |
| **Rootless Docker** | A Docker configuration that runs the daemon without root privileges |

---

## 📚 Resources

- [Docker Official Install Guide](https://docs.docker.com/engine/install/) — platform-specific installation documentation
- [Docker Desktop Download](https://www.docker.com/products/docker-desktop/) — Windows and macOS installer
- [Play with Docker](https://labs.play-with-docker.com) — free browser-based Docker playground (no install required)
- [Docker Hub](https://hub.docker.com) — browse official images: nginx, ubuntu, python, postgres, redis, alpine
- [Docker Post-Installation Steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/) — official guide for user group setup and service configuration
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/) — complete command reference
- [Docker Security — Rootless Mode](https://docs.docker.com/engine/security/rootless/) — running Docker without root privileges

---

## 🔭 Day 26 Preview: Dockerfiles & Building Custom Images

Now that Docker is installed and running, Day 26 moves into creating your own images.

You will learn:
- What a Dockerfile is and how it is structured
- Core Dockerfile instructions: `FROM`, `RUN`, `COPY`, `WORKDIR`, `EXPOSE`, `CMD`, `ENTRYPOINT`
- Building an image with `docker build`
- Image tagging and versioning
- Best practices — layer caching, minimal base images, `.dockerignore`
- Building your first custom image: a containerized Python health-check script

Every production container starts with a Dockerfile. Day 26 is where you go from running other people's images to building your own.

---

> 💬 *"The first `docker run hello-world` is a rite of passage. Every container you run from here builds on that moment."*
>
> — DevOps learning by Yukta