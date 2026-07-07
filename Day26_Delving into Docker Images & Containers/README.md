# Day 26: Delving into Docker Images & Containers

> **DevOps Transformation** | Containers & Automation Series

---

## 📋 Overview

Yesterday you installed Docker and ran your first container. Today you go deeper — understanding how images work as blueprints, how containers are live instances of those blueprints, and how to interact with containers in real time.

By the end of today you will have pulled images, launched interactive containers, run commands inside them, managed multiple containers simultaneously, used environment variables, and cleanly removed everything you created. This is the core Docker operational workflow that you will use every day.

---

## 🗺️ What You'll Do Today

| Topic | Commands | What You Learn |
|---|---|---|
| Docker Images | `docker pull`, `docker images` | Fetch and inspect image blueprints |
| Running Containers | `docker run -it` | Launch and interact with live containers |
| Inside the Container | `ls`, `pwd`, `echo`, `apt` | Work inside an isolated environment |
| Container Lifecycle | `docker ps`, `docker stop`, `docker start`, `docker rm` | Manage containers from running to removed |
| Image Cleanup | `docker rmi` | Remove images and understand dependencies |
| Environment Variables | `docker run -e` | Pass configuration into containers at runtime |
| Multi-Container Practice | Multiple `docker run` sessions | Switch between containers, manage lifecycle |

---

## 🏢 NexusCorp Scenario

> The NexusCorp development team is testing a new microservice. Before writing a Dockerfile, they need to explore the base environment — what packages are available, what version of Python is installed, what the default filesystem looks like. They do this by pulling the base image, launching an interactive container, and poking around. Everything they discover informs the Dockerfile they will write next.
>
> This is exactly what you will practice today.

---

## Part 1: Docker Images — The Blueprints of Containers

### What Makes an Image a Blueprint?

An image is a **read-only**, **layered**, **portable snapshot** of a filesystem and its metadata. It contains:

- A base operating system (or a minimal runtime)
- Installed software and libraries
- Configuration files
- The default command to run when a container starts

Images are immutable — you cannot modify an image directly. When you run a container, Docker adds a thin **read-write layer** on top of the image. Any changes you make inside the container live in that layer. When the container is removed, that layer is gone. The underlying image remains untouched.

```
┌─────────────────────────────────────────┐
│         Container (read-write)          │
│  ┌─────────────────────────────────┐    │
│  │  Writable Layer (your changes)  │    │
│  └─────────────────────────────────┘    │
├─────────────────────────────────────────┤
│         Image (read-only layers)        │
│  ┌─────────────────────────────────┐    │
│  │  Layer 4: App files copied in   │    │
│  │  Layer 3: Dependencies installed│    │
│  │  Layer 2: Package lists updated │    │
│  │  Layer 1: Ubuntu 22.04 base     │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

---

### Pulling Images from Docker Hub

`docker pull` downloads an image from Docker Hub (or another registry) to your local machine without running it.

```bash
# Pull the latest Ubuntu image
docker pull ubuntu:latest

# What you'll see:
# latest: Pulling from library/ubuntu
# 7b1a6ab2e44d: Pull complete
# Digest: sha256:...
# Status: Downloaded newer image for ubuntu:latest
# docker.io/library/ubuntu:latest
```

**Understanding image tags:**

```bash
# Pull a specific version (always prefer pinned versions in production)
docker pull ubuntu:22.04
docker pull ubuntu:20.04

# Pull other useful images
docker pull nginx:latest          # Web server
docker pull python:3.11-slim      # Python runtime (slim = smaller size)
docker pull alpine:latest         # Minimal Linux (only ~7MB)
docker pull node:20-alpine        # Node.js on Alpine
docker pull postgres:15           # PostgreSQL database
docker pull redis:7               # Redis cache

# Pull from a specific registry (not Docker Hub)
docker pull ghcr.io/owner/image:tag
```

**Image tag naming convention:**
```
image-name:tag

ubuntu:latest         → Latest stable Ubuntu
ubuntu:22.04          → Ubuntu version 22.04 (pinned)
python:3.11           → Python 3.11 (full image)
python:3.11-slim      → Python 3.11 (minimal, no build tools)
python:3.11-alpine    → Python 3.11 on Alpine (smallest possible)
nginx:1.25            → Nginx version 1.25
nginx:1.25-alpine     → Nginx 1.25 on Alpine base
```

**`latest` vs pinned tags — best practice:**

| | `latest` tag | Pinned tag (e.g., `22.04`) |
|---|---|---|
| **What it means** | Most recent published version | A specific, fixed version |
| **Reproducible?** | No — changes when new version released | Yes — always the same image |
| **For learning** | Fine | Fine |
| **For production** | Avoid | Always use pinned tags |

---

### Listing Local Images

```bash
# List all locally stored images
docker images
```

**Expected output after pulling ubuntu:**
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    174c8c134b2a   3 weeks ago    77.9MB
hello-world  latest    d2c94e258dcb   2 months ago   13.3kB
```

**More image inspection commands:**

```bash
# Show detailed metadata for an image
docker inspect ubuntu:latest

# Show image history — every layer that makes up the image
docker history ubuntu:latest

# Show history in a readable format
docker history ubuntu:latest --human --format "table {{.CreatedBy}}\t{{.Size}}"

# Show image size breakdown
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Filter images by name
docker images ubuntu

# List only image IDs (useful for scripting)
docker images -q
```

---

## Part 2: Running and Interacting with Containers

### The `docker run` Command — Core Flags

`docker run` creates a new container from an image and starts it. It is the most-used Docker command and has many flags.

```bash
# Basic syntax
docker run [OPTIONS] IMAGE [COMMAND] [ARGS]

# Essential flags to know now:
# -it         Interactive + TTY (for interactive shell sessions)
# -d          Detached (run in background)
# --name      Assign a name to the container
# -p          Port mapping: host_port:container_port
# -e          Environment variable: KEY=VALUE
# -v          Volume mount: host_path:container_path
# --rm        Automatically remove container when it exits
# -w          Set working directory inside the container
```

---

### Run and Interact with an Ubuntu Container

```bash
# Launch an Ubuntu container with an interactive bash shell
docker run -it ubuntu /bin/bash
```

**What each part means:**

| Part | Meaning |
|---|---|
| `docker run` | Create and start a new container |
| `-i` | Keep STDIN open — allows you to type input |
| `-t` | Allocate a pseudo-TTY — gives you a proper terminal experience |
| `ubuntu` | The image to use (uses `ubuntu:latest` if no tag specified) |
| `/bin/bash` | The command to run inside the container |

**Your prompt changes when you enter the container:**
```
# Before (host machine):
ubuntu@ip-172-31-10-45:~$

# After docker run -it ubuntu /bin/bash (inside container):
root@a3f2b1c4d5e6:/#
              ^^^^^^^^^^^^
              Container ID (first 12 characters)
```

The `#` at the end of the prompt indicates you are running as root inside the container.

---

### Commands to Try Inside the Container

Once inside the Ubuntu container, explore the environment:

```bash
# ── Navigation and filesystem ─────────────────────────────────
ls                        # List files in current directory
ls /                      # List root filesystem
ls /etc                   # View configuration files
pwd                       # Print working directory (should be /)
cd /tmp                   # Change to temp directory
pwd                       # Should now show /tmp

# ── System information ────────────────────────────────────────
echo "Hello from inside the container!"
cat /etc/os-release       # Show OS version (Ubuntu)
uname -a                  # Kernel version (same as host — shared kernel)
hostname                  # Container's hostname (= container ID)
whoami                    # Should show: root
id                        # Show user and group IDs

# ── Processes and resources ───────────────────────────────────
ps aux                    # List running processes (very few — just bash)
# Notice: no systemd, no cron, no sshd — a container is minimal

# ── Networking inside the container ──────────────────────────
cat /etc/hosts            # Container's host entries
cat /etc/resolv.conf      # DNS configuration (inherited from host)

# ── Installing packages inside the container ──────────────────
apt update                          # Update package lists
apt install -y curl                 # Install curl
curl https://ifconfig.me            # Check container's external IP

apt install -y python3              # Install Python
python3 --version
python3 -c "print('Python in Docker!')"

# ── Create a file inside the container ────────────────────────
echo "I am inside a Docker container" > /tmp/my_note.txt
cat /tmp/my_note.txt

# ── Exit the container ────────────────────────────────────────
exit
# You're back on the host machine
# The container is stopped — your /tmp/my_note.txt is gone
```

**Key observation:** After typing `exit`, the file you created is gone. The container's writable layer is discarded when the container stops and is removed. The ubuntu image remains untouched on your machine.

---

### Running a Named Container

```bash
# Give the container a name for easier management
docker run -it --name my-ubuntu ubuntu /bin/bash

# Now you can reference it by name instead of ID
docker stop my-ubuntu
docker start my-ubuntu
docker rm my-ubuntu
```

---

### Running Containers in Different Modes

```bash
# Interactive mode (-it) — for exploration and debugging
docker run -it ubuntu /bin/bash

# Detached mode (-d) — for services running in the background
docker run -d --name web-server -p 8080:80 nginx

# Interactive detached — start detached, attach later
docker run -d --name my-ubuntu ubuntu sleep infinity
docker exec -it my-ubuntu bash    # Attach to running container

# One-off command — run and exit
docker run ubuntu echo "Hello from Ubuntu"
docker run python:3.11 python -c "print('2 + 2 =', 2 + 2)"

# Auto-remove after exit (--rm) — no cleanup needed
docker run --rm ubuntu echo "This container cleans itself up"
```

---

## Part 3: Common Commands and Their Nuances

### Container Lifecycle Commands

```bash
# ── Listing containers ────────────────────────────────────────

# List RUNNING containers only
docker ps

# List ALL containers (running + stopped + exited)
docker ps -a

# List only container IDs (useful for scripting)
docker ps -q

# Custom format
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# Show last N containers created (any state)
docker ps -n 5
```

**Sample `docker ps -a` output:**
```
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS                    NAMES
a3f2b1c4d5e6   ubuntu    "/bin/bash"   2 min ago      Exited (0) 1 min ago      my-ubuntu
b4c3d2e1f0a1   nginx     "/docker…"   5 min ago      Up 5 minutes              web-server
c5d4e3f2a1b0   alpine    "/bin/sh"    10 min ago     Exited (130) 8 min ago    bold_tesla
```

**Container status values:**

| Status | Meaning |
|---|---|
| `Up 5 minutes` | Currently running |
| `Exited (0)` | Stopped cleanly (exit code 0 = success) |
| `Exited (1)` | Stopped with an error (non-zero exit code) |
| `Exited (130)` | Stopped by Ctrl+C (SIGINT) |
| `Paused` | Paused via `docker pause` |
| `Restarting` | Container is restarting (due to restart policy) |
| `Created` | Created but never started |
| `Dead` | Container failed to stop properly |

---

### Stopping and Starting Containers

```bash
# Stop a running container (sends SIGTERM, waits, then SIGKILL)
docker stop my-ubuntu
docker stop a3f2b1c4d5e6    # By container ID

# Stop immediately without waiting (SIGKILL)
docker kill my-ubuntu

# Stop multiple containers at once
docker stop container1 container2 container3

# Stop ALL running containers
docker stop $(docker ps -q)

# Start a stopped container (preserves its state)
docker start my-ubuntu

# Start and attach interactively
docker start -ai my-ubuntu

# Restart a running container
docker restart my-ubuntu

# Pause and unpause (freezes the process)
docker pause my-ubuntu
docker unpause my-ubuntu
```

---

### Removing Containers and Images

```bash
# ── Containers ────────────────────────────────────────────────

# Remove a stopped container
docker rm my-ubuntu

# Force remove a running container (stop + remove)
docker rm -f my-ubuntu

# Remove multiple containers
docker rm container1 container2

# Remove ALL stopped containers
docker container prune

# Remove ALL stopped containers without confirmation prompt
docker container prune -f

# Remove a container automatically when it exits (--rm flag at run time)
docker run --rm ubuntu echo "auto-cleaned"


# ── Images ────────────────────────────────────────────────────

# Remove an image (fails if a container — running or stopped — uses it)
docker rmi ubuntu:latest
docker rmi 174c8c134b2a     # By image ID

# Force remove an image (even if containers reference it)
docker rmi -f ubuntu:latest

# Remove multiple images
docker rmi ubuntu:latest nginx:latest

# Remove ALL unused images (not referenced by any container)
docker image prune

# Remove ALL images including those referenced by stopped containers
docker image prune -a


# ── Full cleanup ──────────────────────────────────────────────

# Remove all stopped containers, unused networks, dangling images
docker system prune

# Remove everything including unused images
docker system prune -a

# Show disk usage before cleanup
docker system df
```

**The dependency chain — why order matters:**

```
You cannot remove an image if a container (even stopped) uses it.
You cannot remove a running container without -f.

Safe removal order:
1. docker stop <container>      ← Stop if running
2. docker rm <container>        ← Remove container
3. docker rmi <image>           ← Now remove the image
```

---

### Executing Commands in Running Containers

```bash
# Run a command in an already-running container
docker exec -it web-server bash

# Run a specific command without an interactive shell
docker exec web-server ls /etc/nginx
docker exec web-server cat /etc/nginx/nginx.conf
docker exec web-server nginx -t    # Test nginx config

# Run as a specific user inside the container
docker exec -it --user root web-server bash

# Set environment variables for the exec'd command
docker exec -e MY_VAR=hello web-server printenv MY_VAR
```

---

### Viewing Container Logs

```bash
# View all logs from a container
docker logs web-server

# Follow logs in real time (like tail -f)
docker logs -f web-server

# Show last N lines
docker logs --tail 20 web-server

# Show logs with timestamps
docker logs -t web-server

# Show logs since a specific time
docker logs --since 2024-06-10T14:00:00 web-server
docker logs --since 30m web-server    # Last 30 minutes
```

---

### Inspecting Containers

```bash
# Full JSON metadata about a container
docker inspect web-server

# Extract specific fields using Go template format
docker inspect web-server --format '{{.State.Status}}'
docker inspect web-server --format '{{.NetworkSettings.IPAddress}}'
docker inspect web-server --format '{{.Config.Env}}'

# Live resource usage (CPU, memory, network, disk I/O)
docker stats

# One-time snapshot (no live update)
docker stats --no-stream

# Stats for a specific container
docker stats web-server --no-stream

# Copy files between host and container
docker cp web-server:/etc/nginx/nginx.conf ./nginx.conf   # From container
docker cp ./my-config.conf web-server:/tmp/               # To container
```

---

## Part 4: Environment Variables

### Passing Configuration via `-e`

Environment variables are how you pass configuration into a container at runtime — database URLs, API keys, feature flags, log levels. This avoids hardcoding configuration into images.

```bash
# Pass a single environment variable
docker run -e MY_VARIABLE="Hello Docker" ubuntu echo $MY_VARIABLE
# Output: Hello Docker

# Pass multiple environment variables
docker run \
    -e APP_ENV="staging" \
    -e DB_HOST="db.nexuscorp.internal" \
    -e DB_PORT="5432" \
    -e LOG_LEVEL="INFO" \
    ubuntu printenv

# Pass environment variables interactively
docker run -it \
    -e APP_ENV="development" \
    -e DEBUG="true" \
    ubuntu /bin/bash

# Inside the container:
# echo $APP_ENV     → development
# echo $DEBUG       → true
# printenv          → show all environment variables
```

---

### Using an Environment File (`--env-file`)

For multiple variables, use a file instead of many `-e` flags:

```bash
# Create an env file
cat > app.env << 'EOF'
APP_ENV=staging
DB_HOST=db.nexuscorp.internal
DB_PORT=5432
DB_NAME=nexusdb
LOG_LEVEL=INFO
MAX_CONNECTIONS=100
EOF

# Pass the file to the container
docker run --env-file app.env ubuntu printenv
```

**⚠️ Security note:** Never commit `.env` files containing real secrets to Git. Add them to `.gitignore`. In production, use secret managers (AWS Secrets Manager, HashiCorp Vault) rather than environment variables for sensitive credentials.

---

## Part 5: Practical Exercises

### Exercise 1: Pull and Explore Multiple Images

```bash
# Pull several images and compare them
docker pull ubuntu:22.04
docker pull alpine:latest
docker pull nginx:latest

# List all images and compare sizes
docker images
# Notice: Alpine is ~7MB, Ubuntu is ~78MB, nginx is ~143MB

# Explore what's different inside each
docker run -it ubuntu:22.04 /bin/bash
# Inside: cat /etc/os-release → Ubuntu 22.04
# Try: apt --version, python3 --version
exit

docker run -it alpine:latest /bin/sh    # Alpine uses sh, not bash
# Inside: cat /etc/os-release → Alpine Linux
# Try: apk --version (Alpine's package manager)
exit
```

---

### Exercise 2: Run Multiple Containers Simultaneously

```bash
# Terminal 1: Start a named nginx container
docker run -d --name nexus-web -p 8080:80 nginx
docker run -d --name nexus-api -p 8081:80 nginx

# Check both are running
docker ps
# You should see two nginx containers

# View logs from each
docker logs nexus-web
docker logs nexus-api

# Check different ports
curl http://localhost:8080
curl http://localhost:8081

# Exec into the first one
docker exec -it nexus-web bash
echo "I am nexus-web" > /usr/share/nginx/html/identity.txt
exit

# Exec into the second one
docker exec -it nexus-api bash
echo "I am nexus-api" > /usr/share/nginx/html/identity.txt
exit

# Verify they are independent
curl http://localhost:8080/identity.txt   # nexus-web
curl http://localhost:8081/identity.txt   # nexus-api

# Stop and remove both
docker stop nexus-web nexus-api
docker rm nexus-web nexus-api
```

---

### Exercise 3: Environment Variables Practice

```bash
# Run with environment variables and verify inside
docker run -it \
    -e ENVIRONMENT="staging" \
    -e MAX_RETRIES="3" \
    -e SERVICE_NAME="nexuscorp-api" \
    ubuntu /bin/bash

# Inside the container:
echo $ENVIRONMENT       # staging
echo $MAX_RETRIES       # 3
echo $SERVICE_NAME      # nexuscorp-api
printenv | grep -E "ENVIRONMENT|MAX_RETRIES|SERVICE_NAME"
exit

# One-liner: run, print var, exit
docker run \
    -e MY_VARIABLE="Hello Docker" \
    ubuntu \
    printenv MY_VARIABLE
# Output: Hello Docker

# Python container using environment variables
docker run \
    -e DB_HOST="localhost" \
    -e DB_PORT="5432" \
    python:3.11-slim \
    python3 -c "
import os
print(f'DB Host: {os.getenv(\"DB_HOST\", \"not set\")}')
print(f'DB Port: {os.getenv(\"DB_PORT\", \"not set\")}')
"
```

---

### Exercise 4: Complete Lifecycle — Pull, Run, Interact, Clean Up

```bash
# Step 1: Pull three images
docker pull ubuntu:22.04
docker pull alpine:latest
docker pull python:3.11-slim

# Verify all three are present
docker images

# Step 2: Run containers from each
docker run -d --name ubuntu-c ubuntu:22.04 sleep 300
docker run -d --name alpine-c alpine:latest sleep 300
docker run -d --name python-c python:3.11-slim sleep 300

# Verify all three containers are running
docker ps

# Step 3: Interact with each
docker exec -it ubuntu-c bash
# Inside: echo "In ubuntu container" && exit

docker exec -it alpine-c sh
# Inside: echo "In alpine container" && exit

docker exec -it python-c python3 -c "print('In python container')"

# Step 4: Stop all containers
docker stop ubuntu-c alpine-c python-c

# Verify they are stopped
docker ps       # None running
docker ps -a    # All three present as Exited

# Step 5: Remove all containers
docker rm ubuntu-c alpine-c python-c

# Verify containers are gone
docker ps -a

# Step 6: Remove all images
docker rmi ubuntu:22.04 alpine:latest python:3.11-slim

# Verify images are gone
docker images

# Final: Check disk usage is clean
docker system df
```

---

## 🔧 Mini-Project: NexusCorp Container Exploration

**Scenario:** The NexusCorp team needs to evaluate three different base images for their Python microservice. They want to compare: Ubuntu (full), Alpine (minimal), and Python-slim (purpose-built). You will run the same test inside each to compare environments.

```bash
#!/bin/bash
# container_compare.sh
# Compare three base images for Python microservice use

IMAGES=("ubuntu:22.04" "alpine:latest" "python:3.11-slim")
NAMES=("ubuntu-test" "alpine-test" "python-test")
COMMANDS=("bash" "sh" "bash")

echo "===================================================="
echo "  NexusCorp Base Image Comparison"
echo "===================================================="

# Pull all images
echo ""
echo "Pulling images..."
for image in "${IMAGES[@]}"; do
    docker pull "$image" -q
    echo "  ✓ Pulled: $image"
done

# Run each container and collect info
for i in "${!IMAGES[@]}"; do
    image="${IMAGES[$i]}"
    name="${NAMES[$i]}"

    echo ""
    echo "----------------------------------------------------"
    echo "  Testing: $image"
    echo "----------------------------------------------------"

    # Get OS info
    OS_INFO=$(docker run --rm "$image" cat /etc/os-release 2>/dev/null | \
              grep PRETTY_NAME | cut -d'"' -f2)
    echo "  OS        : $OS_INFO"

    # Get image size
    SIZE=$(docker images "$image" --format "{{.Size}}")
    echo "  Image size: $SIZE"

    # Check for Python
    PYTHON=$(docker run --rm "$image" python3 --version 2>/dev/null || \
             echo "Not installed")
    echo "  Python    : $PYTHON"

    # Check for package manager
    APT=$(docker run --rm "$image" which apt 2>/dev/null && echo "apt" || echo "")
    APK=$(docker run --rm "$image" which apk 2>/dev/null && echo "apk" || echo "")
    echo "  Pkg mgr   : ${APT}${APK}"
done

echo ""
echo "===================================================="
echo "  Comparison complete"
echo "===================================================="

# Cleanup
echo ""
echo "Cleaning up images..."
for image in "${IMAGES[@]}"; do
    docker rmi "$image" -f > /dev/null 2>&1
    echo "  ✓ Removed: $image"
done
echo "Done."
```

```bash
chmod +x container_compare.sh
./container_compare.sh
```

**Expected output:**
```
====================================================
  NexusCorp Base Image Comparison
====================================================

Pulling images...
  ✓ Pulled: ubuntu:22.04
  ✓ Pulled: alpine:latest
  ✓ Pulled: python:3.11-slim

----------------------------------------------------
  Testing: ubuntu:22.04
----------------------------------------------------
  OS        : Ubuntu 22.04.3 LTS
  Image size: 77.9MB
  Python    : Not installed
  Pkg mgr   : apt

----------------------------------------------------
  Testing: alpine:latest
----------------------------------------------------
  OS        : Alpine Linux v3.18
  Image size: 7.34MB
  Python    : Not installed
  Pkg mgr   : apk

----------------------------------------------------
  Testing: python:3.11-slim
----------------------------------------------------
  OS        : Debian GNU/Linux 12 (bookworm)
  Image size: 130MB
  Python    : Python 3.11.6
  Pkg mgr   : apt

====================================================
  Comparison complete
====================================================
```

**What this tells you:**
- Alpine is the smallest but needs Python installed separately
- Python-slim has Python ready but is larger than Alpine
- Ubuntu is familiar and flexible but the largest and needs Python added
- For a Python microservice: `python:3.11-slim` or `python:3.11-alpine` are the best choices

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **`docker pull`** | Downloads an image from a registry to the local machine |
| **`docker run`** | Creates and starts a new container from an image |
| **`docker run -it`** | Runs a container with an interactive terminal |
| **`docker run -d`** | Runs a container detached (in the background) |
| **`docker run --rm`** | Automatically removes the container when it exits |
| **`docker run --name`** | Assigns a human-readable name to the container |
| **`docker run -e`** | Passes an environment variable into the container |
| **`docker run --env-file`** | Passes multiple environment variables from a file |
| **`docker run -p`** | Maps a host port to a container port |
| **`docker ps`** | Lists running containers |
| **`docker ps -a`** | Lists all containers including stopped ones |
| **`docker stop`** | Gracefully stops a running container (SIGTERM → SIGKILL) |
| **`docker kill`** | Immediately terminates a container (SIGKILL) |
| **`docker start`** | Starts a previously stopped container |
| **`docker restart`** | Stops and restarts a container |
| **`docker rm`** | Removes a stopped container |
| **`docker rm -f`** | Force removes a running container |
| **`docker rmi`** | Removes a local image |
| **`docker exec`** | Runs a command in an already-running container |
| **`docker exec -it`** | Opens an interactive shell in a running container |
| **`docker logs`** | Shows the stdout/stderr output of a container |
| **`docker logs -f`** | Follows logs in real time |
| **`docker inspect`** | Returns full JSON metadata about a container or image |
| **`docker stats`** | Shows live resource usage for running containers |
| **`docker cp`** | Copies files between the host and a container |
| **`docker system prune`** | Removes all stopped containers and unused images |
| **`docker container prune`** | Removes all stopped containers |
| **`docker image prune`** | Removes all dangling (unused) images |
| **Writable layer** | The read-write layer added on top of the read-only image when a container runs |
| **Container ID** | A unique identifier for each container — shown in `docker ps` and in the shell prompt |
| **Exit code** | The status code returned when a container stops — 0 = success, non-zero = error |
| **Alpine Linux** | An extremely minimal Linux distribution (~7MB) widely used as a Docker base image |
| **`-slim` image variant** | A smaller version of an official image with fewer pre-installed tools |
| **`printenv`** | Linux command that prints all environment variables |
| **`sleep infinity`** | A command used to keep a container running indefinitely for testing |

---

## 📚 Resources

- [Docker Official Documentation — Run a Container](https://docs.docker.com/engine/reference/run/) — complete `docker run` flag reference
- [Docker Hub — Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official) — browse all official base images
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/) — every Docker command documented
- [Play with Docker](https://labs.play-with-docker.com) — free browser-based Docker environment
- [Alpine Linux Packages](https://pkgs.alpinelinux.org/packages) — find Alpine packages (`apk add`)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/) — official guide on secure container usage

---

## 🔭 Day 27 Preview: Writing Dockerfiles & Building Custom Images

Today you used images others built. Tomorrow you build your own.

You will learn:
- Dockerfile structure and all core instructions: `FROM`, `RUN`, `COPY`, `ADD`, `WORKDIR`, `EXPOSE`, `ENV`, `CMD`, `ENTRYPOINT`
- Building an image with `docker build -t`
- Layer caching — how Docker optimizes rebuilds
- `.dockerignore` — what to exclude from the build context
- Multi-stage builds — building lean production images
- Building your first custom image: a containerized Python script that runs as a service

Every production deployment starts with a Dockerfile. Day 27 is where you take full control of your container environments.

---

> 💬 *"Every time you type `docker run -it ubuntu bash` and find yourself inside a container, remember: that environment travelled with the image. It will be identical on every machine that runs it. That's the promise Docker keeps."*
>
> — DevOps Learning by Yukta
