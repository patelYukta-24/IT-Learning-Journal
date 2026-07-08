# Day 31: Docker Cheat Sheet — Consolidation & Quick Reference

> **DevOps Transformation** | Containers & Automation Series

---

## 📋 Overview

Today is consolidation day. Over the past eight sessions (Days 24–30) you have covered Docker from first principles to production-ready image building. Today you build something that will serve you for years: a comprehensive Docker cheat sheet.

This serves three purposes — reinforcing everything you learned by revisiting it, creating a quick reference you can reach for during any Docker task, and producing something shareable that helps others in their Docker journey.

---

## 🗺️ Cheat Sheet Structure

| Section | What It Covers |
|---|---|
| 1. Core Concepts | Containers, images, registries, layers |
| 2. Installation & Setup | Install Docker, post-install config |
| 3. Image Commands | pull, build, tag, push, rmi, history |
| 4. Container Commands | run, ps, stop, start, rm, exec, logs |
| 5. Volume Commands | create, ls, inspect, rm, prune |
| 6. Network Commands | ls, create, inspect, connect, rm |
| 7. Docker Compose | up, down, ps, logs, exec, build |
| 8. Dockerfile Reference | All directives with examples |
| 9. Best Practices | Security, optimization, workflow |
| 10. Troubleshooting | Common errors and fixes |

---

## Section 1: Core Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                    Docker Architecture                          │
│                                                                 │
│  Dockerfile ──build──► Image ──run──► Container               │
│                 │                                               │
│                 └──push──► Registry ──pull──► Image            │
│                                                                 │
│  Image     = Read-only blueprint (layers)                       │
│  Container = Running instance of an image                       │
│  Registry  = Storage for images (Docker Hub, ECR, GCR)         │
│  Volume    = Persistent storage outside container lifecycle     │
│  Network   = Virtual network connecting containers             │
└─────────────────────────────────────────────────────────────────┘
```

| Concept | Definition |
|---|---|
| **Image** | A read-only, layered blueprint for creating containers |
| **Container** | A running (or stopped) instance of an image |
| **Dockerfile** | A script of instructions used to build an image |
| **Registry** | A service that stores and distributes images (Docker Hub, ECR) |
| **Layer** | A single filesystem change created by a Dockerfile instruction |
| **Volume** | Docker-managed persistent storage that survives container removal |
| **Bind Mount** | A host directory mounted into a container |
| **Network** | An isolated virtual network containers connect to |
| **Compose** | A tool for defining multi-container apps in a YAML file |

---

## Section 2: Installation & Setup

### Install Docker

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# Amazon Linux 2
sudo yum install -y docker
sudo systemctl start docker && sudo systemctl enable docker

# Verify
docker --version
docker compose version
```

### Post-Installation (Linux)

```bash
# Add user to docker group (avoid sudo on every command)
sudo usermod -aG docker $USER
newgrp docker                  # Apply immediately (or log out/in)

# Verify
docker run hello-world         # Should work without sudo

# Start Docker on boot
sudo systemctl enable docker

# Check Docker service status
sudo systemctl status docker
```

### Docker Info Commands

```bash
docker version                 # Client + server version info
docker info                    # System-wide information
docker system df               # Disk usage summary
docker system prune            # Remove all unused resources
docker system prune -a         # Remove ALL unused including images
docker system prune -a --volumes  # Remove everything including volumes
```

---

## Section 3: Image Commands

### Pull Images

```bash
docker pull ubuntu:22.04           # Pull specific version
docker pull nginx:latest           # Pull latest tag
docker pull python:3.11-slim       # Pull slim variant
docker pull python:3.11-alpine     # Pull Alpine variant
```

### List and Inspect Images

```bash
docker images                      # List all local images
docker images -a                   # Include intermediate layers
docker images -q                   # Image IDs only
docker images nginx                # Filter by name
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

docker inspect nginx:latest        # Full JSON metadata
docker history nginx:latest        # Show all layers and sizes
docker history --no-trunc nginx    # Full layer commands
```

### Build Images

```bash
# Basic build
docker build -t myapp:latest .

# With specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# With build arguments
docker build --build-arg VERSION=2.0 -t myapp:2.0 .

# No cache (force full rebuild)
docker build --no-cache -t myapp:latest .

# Multi-platform build
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .

# Verbose output
docker build --progress=plain -t myapp:latest .
```

### Tag Images

```bash
docker tag myapp:latest myapp:1.0.0
docker tag myapp:latest username/myapp:latest
docker tag myapp:latest username/myapp:1.0.0
docker tag myapp:latest registry.nexuscorp.com/myapp:latest
```

### Push and Remove Images

```bash
docker login                        # Log in to Docker Hub
docker login registry.nexuscorp.com # Log in to private registry
docker push username/myapp:latest   # Push to Docker Hub
docker push username/myapp:1.0.0

docker rmi nginx:latest             # Remove an image
docker rmi -f nginx:latest          # Force remove
docker image prune                  # Remove dangling images
docker image prune -a               # Remove all unused images
```

---

## Section 4: Container Commands

### Running Containers

```bash
# Basic run
docker run nginx

# Detached (background) mode
docker run -d nginx

# Interactive terminal
docker run -it ubuntu bash

# Named container
docker run --name my-nginx nginx

# Port mapping
docker run -p 8080:80 nginx        # host:container
docker run -p 8080:80 -p 443:443 nginx

# Environment variables
docker run -e KEY=value nginx
docker run --env-file .env nginx

# Volume mount
docker run -v myvolume:/data nginx
docker run -v $(pwd):/app nginx    # Bind mount

# Network
docker run --network mynet nginx

# Auto-remove on exit
docker run --rm ubuntu echo "hello"

# Resource limits
docker run --memory 512m --cpus 1.5 nginx

# Restart policy
docker run --restart unless-stopped nginx
docker run --restart always nginx
docker run --restart on-failure:3 nginx

# Run as specific user
docker run --user 1000:1000 nginx

# Read-only filesystem
docker run --read-only nginx

# Full example
docker run -d \
    --name nexus-api \
    --network nexus-net \
    --restart unless-stopped \
    -p 5000:5000 \
    -e APP_ENV=production \
    -v app-data:/data \
    --memory 256m \
    nexus-api:1.0.0
```

### Managing Containers

```bash
# List containers
docker ps                           # Running only
docker ps -a                        # All containers
docker ps -q                        # IDs only
docker ps -n 5                      # Last 5 created
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# Stop and start
docker stop my-container           # Graceful (SIGTERM → SIGKILL)
docker stop -t 30 my-container     # 30s timeout before SIGKILL
docker kill my-container           # Immediate (SIGKILL)
docker start my-container          # Start stopped container
docker restart my-container        # Stop + start
docker pause my-container          # Freeze (SIGSTOP)
docker unpause my-container        # Unfreeze

# Remove containers
docker rm my-container             # Remove stopped container
docker rm -f my-container          # Force remove (stops first)
docker rm $(docker ps -aq)         # Remove all stopped containers
docker container prune             # Remove all stopped containers
docker container prune -f          # No confirmation prompt
```

### Interacting with Containers

```bash
# Execute commands in running container
docker exec my-container ls /app
docker exec -it my-container bash
docker exec -it --user root my-container bash
docker exec -e VAR=value my-container printenv

# View logs
docker logs my-container           # All logs
docker logs -f my-container        # Follow (stream) logs
docker logs --tail 50 my-container # Last 50 lines
docker logs -t my-container        # With timestamps
docker logs --since 30m my-container  # Last 30 minutes

# Copy files
docker cp my-container:/etc/nginx/nginx.conf ./  # From container
docker cp ./config.yml my-container:/app/        # To container

# Inspect
docker inspect my-container        # Full JSON metadata
docker inspect -f '{{.State.Status}}' my-container
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-container

# Resource usage
docker stats                       # Live stats (all containers)
docker stats --no-stream           # One-time snapshot
docker stats my-container          # Single container
docker top my-container            # Running processes inside
```

---

## Section 5: Volume Commands

```bash
# Create
docker volume create myvolume
docker volume create --driver local myvolume
docker volume create --label project=nexus myvolume

# List and inspect
docker volume ls
docker volume ls -q                # Names only
docker volume ls --filter label=project=nexus
docker volume inspect myvolume

# Mount in container
docker run -v myvolume:/data nginx           # Named volume
docker run -v /host/path:/data nginx         # Bind mount
docker run -v /host/path:/data:ro nginx      # Read-only
docker run --mount type=volume,source=myvolume,target=/data nginx

# Remove
docker volume rm myvolume
docker volume rm $(docker volume ls -q)      # Remove all
docker volume prune                          # Remove unused
docker volume prune -f                       # No confirmation

# Backup a volume
docker run --rm \
    -v myvolume:/data \
    -v $(pwd):/backup \
    ubuntu tar czf /backup/backup.tar.gz -C /data .

# Restore a volume
docker run --rm \
    -v myvolume:/data \
    -v $(pwd):/backup \
    ubuntu tar xzf /backup/backup.tar.gz -C /data
```

---

## Section 6: Network Commands

```bash
# List networks
docker network ls
docker network ls --filter driver=bridge

# Inspect networks
docker network inspect bridge
docker network inspect mynet \
    --format '{{range .Containers}}{{.Name}} {{.IPv4Address}}{{println}}{{end}}'

# Create networks
docker network create mynet
docker network create --driver bridge mynet
docker network create --subnet 10.10.0.0/24 --gateway 10.10.0.1 mynet
docker network create --driver overlay myswarmnet  # Requires Swarm

# Connect/disconnect containers
docker network connect mynet my-container
docker network connect --ip 10.10.0.50 mynet my-container
docker network disconnect mynet my-container

# Remove networks
docker network rm mynet
docker network prune               # Remove unused networks

# Find container IP
docker inspect -f \
    '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-container
```

### Network Modes Reference

| Mode | Flag | Description |
|---|---|---|
| Bridge (default) | `--network bridge` | Isolated virtual network — recommended |
| User-defined bridge | `--network mynet` | Bridge + DNS by container name |
| Host | `--network host` | Share host network stack |
| None | `--network none` | No networking (loopback only) |
| Overlay | `--network myoverlay` | Multi-host (requires Swarm) |

---

## Section 7: Docker Compose Commands

```bash
# Start services
docker-compose up                  # Foreground
docker-compose up -d               # Detached
docker-compose up -d --build       # Rebuild images first
docker-compose up -d --scale api=3 # Run 3 API instances
docker-compose up api redis        # Start specific services only

# Stop services
docker-compose stop                # Stop (keep containers)
docker-compose stop api            # Stop specific service
docker-compose start               # Start stopped containers
docker-compose restart             # Restart all services
docker-compose down                # Stop + remove containers/networks
docker-compose down -v             # + remove volumes
docker-compose down --rmi all      # + remove images

# View status
docker-compose ps                  # Service status
docker-compose top                 # Running processes per service
docker-compose images              # Images used by services

# Logs
docker-compose logs                # All service logs
docker-compose logs -f             # Follow logs
docker-compose logs -f api         # Follow specific service
docker-compose logs --tail 50      # Last 50 lines

# Execute commands
docker-compose exec api bash       # Shell in running container
docker-compose exec api python3 manage.py migrate
docker-compose run --rm api pytest # One-off command in new container

# Build
docker-compose build               # Build all service images
docker-compose build api           # Build specific service
docker-compose pull                # Pull latest images

# Configuration
docker-compose config              # Validate + print resolved config

# Specify compose file
docker-compose -f docker-compose.prod.yml up -d
```

### `docker-compose.yml` Template

```yaml
version: "3.8"

services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        BUILD_ENV: production
    image: nexuscorp/app:${VERSION:-latest}
    container_name: nexus-app
    ports:
      - "${APP_PORT:-5000}:5000"
    environment:
      - DATABASE_URL=postgresql://postgres:${DB_PASS}@db:5432/nexusdb
      - REDIS_URL=redis://cache:6379
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - app-data:/app/data
      - ./config:/app/config:ro
    networks:
      - frontend
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASS}
      POSTGRES_DB: nexusdb
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  app-data:
  db-data:
  redis-data:
```

---

## Section 8: Dockerfile Reference

### Complete Directive Reference

```dockerfile
# ── Base image ────────────────────────────────────────────────
FROM python:3.11-slim                    # Base image
FROM python:3.11-slim AS builder         # Named stage
FROM scratch                             # Empty base

# ── Metadata ──────────────────────────────────────────────────
LABEL maintainer="devops@nexuscorp.com"
LABEL version="1.0.0"
LABEL description="NexusCorp API Service"

# ── Build arguments (build-time only) ─────────────────────────
ARG VERSION=1.0.0
ARG BUILD_ENV=production

# ── Environment variables (build + runtime) ───────────────────
ENV APP_PORT=5000 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# ── Working directory ─────────────────────────────────────────
WORKDIR /app

# ── Copy files ────────────────────────────────────────────────
COPY requirements.txt .              # Single file
COPY src/ /app/src/                  # Directory
COPY --chown=appuser:appgroup . .    # With ownership

# ── Add (with extract/URL capabilities) ───────────────────────
ADD archive.tar.gz /app/             # Auto-extract
# Prefer COPY over ADD unless extraction is needed

# ── Run commands ──────────────────────────────────────────────
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir -r requirements.txt

# ── Create non-root user ──────────────────────────────────────
RUN groupadd -r appgroup \
    && useradd -r -g appgroup appuser
USER appuser

# ── Expose port (documentation only) ─────────────────────────
EXPOSE 5000

# ── Volume declaration ────────────────────────────────────────
VOLUME ["/app/data"]

# ── Health check ──────────────────────────────────────────────
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:${APP_PORT}/health || exit 1

# ── Default command (overridable) ─────────────────────────────
CMD ["python", "app.py"]

# ── Main process (harder to override) ─────────────────────────
ENTRYPOINT ["gunicorn"]
CMD ["--bind", "0.0.0.0:5000", "app:application"]
```

### Multi-Stage Build Template

```dockerfile
# ── Stage 1: Build/compile environment ───────────────────────
FROM python:3.11-slim AS builder

WORKDIR /build

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        gcc libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Create and use virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt


# ── Stage 2: Production runtime (lean) ────────────────────────
FROM python:3.11-slim AS production

# Copy only the virtual environment from builder
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

WORKDIR /app
COPY . .

# Non-root user
RUN groupadd -r nexus && useradd -r -g nexus nexus
USER nexus

EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:application"]
```

---

## Section 9: Best Practices

### Dockerfile Best Practices

```
✅ DO:
├── Pin exact base image versions (python:3.11-slim, not python:latest)
├── Use .dockerignore to exclude unnecessary files
├── Chain RUN commands with && to minimize layers
├── Clean up apt/apk cache in the same RUN instruction
├── Copy dependency files BEFORE application code (cache optimization)
├── Use non-root USER for security
├── Use COPY instead of ADD (unless you need tar extraction)
├── Use multi-stage builds for compiled or build-heavy apps
├── Add HEALTHCHECK for production containers
├── Use specific ENTRYPOINT + CMD for flexibility
├── Set PYTHONDONTWRITEBYTECODE=1 and PYTHONUNBUFFERED=1 for Python
├── Use --no-cache-dir with pip install
└── Label images with metadata (maintainer, version, description)

❌ DON'T:
├── Use :latest tag in production (unpredictable)
├── Run as root in production containers
├── Store secrets in Dockerfiles (use secrets/env vars at runtime)
├── Copy entire directories when you only need specific files
├── Install unnecessary packages (increases attack surface + size)
├── Use ADD for simple file copies (use COPY instead)
├── Ignore the build cache (order matters — stable layers first)
└── Put frequently changing files early in the Dockerfile
```

### Layer Caching Order

```dockerfile
# CORRECT ORDER — maximizes cache efficiency
FROM python:3.11-slim                  # Layer 1: Rarely changes
WORKDIR /app                           # Layer 2: Rarely changes
COPY requirements.txt .                # Layer 3: Changes when deps change
RUN pip install -r requirements.txt    # Layer 4: Only rebuilds when Layer 3 changes
COPY . .                               # Layer 5: Changes with every code edit
CMD ["python", "app.py"]              # Layer 6: Rebuilt with Layer 5

# WRONG ORDER — cache busted on every code change
FROM python:3.11-slim
WORKDIR /app
COPY . .                               # Any code change busts cache here
RUN pip install -r requirements.txt    # Reinstalls ALL packages every time!
CMD ["python", "app.py"]
```

### Security Best Practices

```
Container Security:
├── Never run containers as root — use USER directive
├── Use read-only filesystems: docker run --read-only
├── Drop capabilities: docker run --cap-drop ALL --cap-add NET_BIND_SERVICE
├── Scan images for vulnerabilities: docker scout cves myimage
├── Use secrets management (Docker Secrets, AWS Secrets Manager)
├── Never put credentials in Dockerfiles or environment variables in images
├── Keep base images updated (regular rebuilds)
└── Use private registries for proprietary images

Network Security:
├── Use user-defined bridge networks (not default bridge)
├── Isolate tiers: frontend network, backend network
├── Never expose database ports to the host in production
├── Use TLS for registry communication
└── Limit exposed ports to only what is necessary

Volume Security:
├── Use :ro (read-only) for config and secret mounts
├── Never bind-mount sensitive host directories
├── Use named volumes over bind mounts in production
└── Regularly backup critical volumes
```

### Image Optimization Best Practices

```
Size Reduction:
├── Use Alpine or slim base images
├── Use multi-stage builds — separate build from runtime
├── Use .dockerignore to exclude: .git, node_modules, __pycache__, .env, tests/
├── Chain and clean up in single RUN: apt-get install && rm -rf /var/lib/apt/lists/*
├── Use --no-install-recommends with apt-get
├── Use --no-cache-dir with pip
└── Remove temp files in the same layer they are created

Image Size Comparison:
python:3.11           → ~1GB
python:3.11-slim      → ~130MB
python:3.11-alpine    → ~50MB
Multi-stage + alpine  → ~40MB
```

### Docker Compose Best Practices

```
✅ DO:
├── Use .env files for sensitive values (never hardcode in compose file)
├── Add .env to .gitignore
├── Use named volumes for persistent data
├── Define healthchecks and use condition: service_healthy in depends_on
├── Use restart: unless-stopped for services that should always run
├── Define explicit networks (don't rely on default Compose network)
├── Use specific image tags, not :latest
├── Separate compose files: docker-compose.yml (base) + docker-compose.prod.yml (overrides)
└── Use docker-compose config to validate before running

❌ DON'T:
├── Commit .env files with real credentials to Git
├── Use --link (deprecated — use networks instead)
├── Expose database ports to host in production
└── Use :latest tags for images in production
```

---

## Section 10: Troubleshooting

### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `Cannot connect to Docker daemon` | Docker not running | `sudo systemctl start docker` |
| `permission denied` on socket | User not in docker group | `sudo usermod -aG docker $USER` then log out/in |
| `Port already in use` | Host port occupied | Use different host port: `-p 8081:80` |
| `No space left on device` | Disk full | `docker system prune -a` |
| `Image not found` | Wrong name/tag or not pulled | `docker pull` first, check tag spelling |
| `Container exits immediately` | CMD/ENTRYPOINT fails | Check logs: `docker logs container-name` |
| `OCI runtime error` | Permission or resource issue | Check `--user`, `--privileged`, or resource limits |
| `Network not found` | Network doesn't exist | `docker network create networkname` |
| `Volume in use` | Container using volume | Stop/remove container first, then remove volume |
| `Build context too large` | Missing .dockerignore | Add `.dockerignore`, exclude `node_modules`, `.git` |
| `Layer cache miss` | File changed before stable layers | Reorder Dockerfile — stable instructions first |
| `Healthcheck failing` | Service not ready | Increase `--start-period`, check CMD syntax |

### Debugging Commands

```bash
# Container won't start — check logs
docker logs container-name
docker logs --tail 50 container-name

# Container exits — run interactively to debug
docker run -it --entrypoint bash myimage:latest

# Check what's inside an image
docker run --rm -it myimage:latest ls -la /app

# Inspect full container config
docker inspect container-name

# Check resource usage
docker stats --no-stream

# Check disk usage by images/containers/volumes
docker system df -v

# Find which container uses a port
docker ps --filter "publish=8080"

# Check container's network connectivity
docker exec container-name curl http://other-container
docker exec container-name ping -c 2 other-container

# Inspect network to see connected containers
docker network inspect mynet

# View real-time events from Docker
docker events

# View image build layers and size
docker history myimage:latest

# Scan image for vulnerabilities
docker scout cves myimage:latest
```

---

## Quick Reference Cards

### Essential `docker run` Flags

| Flag | Short Form | Example | Description |
|---|---|---|---|
| `--detach` | `-d` | `-d` | Run in background |
| `--interactive` | `-i` | `-it` | Keep STDIN open |
| `--tty` | `-t` | `-it` | Allocate pseudo-TTY |
| `--name` | — | `--name web` | Container name |
| `--publish` | `-p` | `-p 8080:80` | Port mapping |
| `--env` | `-e` | `-e KEY=val` | Environment variable |
| `--volume` | `-v` | `-v vol:/data` | Mount volume |
| `--network` | — | `--network net` | Connect to network |
| `--rm` | — | `--rm` | Auto-remove on exit |
| `--restart` | — | `--restart unless-stopped` | Restart policy |
| `--memory` | — | `--memory 512m` | Memory limit |
| `--cpus` | — | `--cpus 1.5` | CPU limit |
| `--user` | `-u` | `--user 1000` | Run as user |
| `--read-only` | — | `--read-only` | Read-only filesystem |
| `--env-file` | — | `--env-file .env` | Load env from file |

---

### Container Status Quick Reference

| Status | Meaning | Action |
|---|---|---|
| `Up 5 minutes` | Running normally | — |
| `Exited (0)` | Stopped cleanly | `docker start` to resume |
| `Exited (1)` | Stopped with error | `docker logs` to debug |
| `Exited (130)` | Stopped by Ctrl+C | `docker start` to resume |
| `Restarting` | Crash-looping | `docker logs`, check config |
| `Paused` | Frozen | `docker unpause` |
| `Created` | Never started | `docker start` |
| `Dead` | Failed to stop | `docker rm -f` |

---

### Port Ranges Reference

| Range | Type | Common Examples |
|---|---|---|
| 0–1023 | System/well-known | 22 (SSH), 80 (HTTP), 443 (HTTPS) |
| 1024–49151 | Registered | 3000 (Node), 5000 (Flask), 5432 (Postgres) |
| 49152–65535 | Dynamic/ephemeral | Assigned automatically to client connections |

---

## Bonus Task: Interactive Cheat Sheet

For an interactive experience, create an HTML page where clicking a command shows its description and examples:

```html
<!-- docker-cheatsheet.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Docker Cheat Sheet — NexusCorp</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background: #0d1117;
            color: #c9d1d9;
            padding: 2rem;
        }
        h1 { color: #58a6ff; margin-bottom: 0.5rem; }
        .subtitle { color: #8b949e; margin-bottom: 2rem; }
        .search-bar {
            width: 100%;
            padding: 0.75rem 1rem;
            background: #161b22;
            border: 1px solid #30363d;
            border-radius: 8px;
            color: #c9d1d9;
            font-size: 1rem;
            margin-bottom: 2rem;
            outline: none;
        }
        .search-bar:focus { border-color: #58a6ff; }
        .section { margin-bottom: 2rem; }
        .section-title {
            color: #58a6ff;
            font-size: 1.1rem;
            font-weight: 600;
            margin-bottom: 1rem;
            padding-bottom: 0.5rem;
            border-bottom: 1px solid #30363d;
        }
        .commands-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 0.75rem;
        }
        .command-card {
            background: #161b22;
            border: 1px solid #30363d;
            border-radius: 8px;
            padding: 1rem;
            cursor: pointer;
            transition: all 0.2s;
        }
        .command-card:hover { border-color: #58a6ff; transform: translateY(-2px); }
        .command-card.active {
            border-color: #3fb950;
            background: #1a2a1a;
        }
        .cmd-name {
            font-family: 'Courier New', monospace;
            color: #3fb950;
            font-size: 0.9rem;
            font-weight: bold;
            margin-bottom: 0.5rem;
        }
        .cmd-description { color: #8b949e; font-size: 0.85rem; }
        .cmd-detail {
            display: none;
            margin-top: 0.75rem;
            padding-top: 0.75rem;
            border-top: 1px solid #30363d;
        }
        .command-card.active .cmd-detail { display: block; }
        .cmd-example {
            background: #0d1117;
            border-radius: 4px;
            padding: 0.5rem 0.75rem;
            margin-top: 0.5rem;
            font-family: 'Courier New', monospace;
            font-size: 0.8rem;
            color: #79c0ff;
            overflow-x: auto;
        }
        .cmd-example span { color: #8b949e; }
        .copy-btn {
            background: #21262d;
            border: 1px solid #30363d;
            color: #8b949e;
            padding: 0.25rem 0.5rem;
            border-radius: 4px;
            font-size: 0.75rem;
            cursor: pointer;
            float: right;
            margin-top: -0.25rem;
        }
        .copy-btn:hover { background: #30363d; color: #c9d1d9; }
        .hidden { display: none; }
    </style>
</head>
<body>
    <h1>🐳 Docker Cheat Sheet</h1>
    <p class="subtitle">NexusCorp DevOps Transformation — Click any command to expand</p>

    <input
        class="search-bar"
        type="text"
        id="search"
        placeholder="Search commands... (e.g. 'run', 'volume', 'network')"
        oninput="filterCommands()"
    >

    <div id="cheatsheet"></div>

    <script>
        const commands = {
            "Images": [
                {
                    cmd: "docker pull IMAGE:TAG",
                    desc: "Download an image from a registry",
                    example: "docker pull nginx:1.25\ndocker pull python:3.11-slim",
                    tags: ["pull", "image", "download", "registry"]
                },
                {
                    cmd: "docker build -t NAME:TAG .",
                    desc: "Build an image from a Dockerfile in current directory",
                    example: "docker build -t myapp:latest .\ndocker build -f Dockerfile.prod -t myapp:prod .",
                    tags: ["build", "image", "dockerfile"]
                },
                {
                    cmd: "docker images",
                    desc: "List all locally stored images",
                    example: "docker images\ndocker images nginx\ndocker images -q  # IDs only",
                    tags: ["images", "list", "ls"]
                },
                {
                    cmd: "docker rmi IMAGE",
                    desc: "Remove a local image",
                    example: "docker rmi nginx:latest\ndocker image prune -a  # Remove all unused",
                    tags: ["rmi", "remove", "image", "delete"]
                },
                {
                    cmd: "docker tag SOURCE TARGET",
                    desc: "Create a new tag for an existing image",
                    example: "docker tag myapp:latest myapp:1.0.0\ndocker tag myapp:latest user/myapp:latest",
                    tags: ["tag", "image", "version"]
                },
                {
                    cmd: "docker push IMAGE:TAG",
                    desc: "Push an image to a registry",
                    example: "docker push username/myapp:latest\ndocker push registry.nexuscorp.com/myapp:1.0.0",
                    tags: ["push", "image", "registry", "publish"]
                },
                {
                    cmd: "docker history IMAGE",
                    desc: "Show the layer history of an image",
                    example: "docker history nginx:latest\ndocker history --no-trunc myapp",
                    tags: ["history", "layers", "image"]
                }
            ],
            "Containers": [
                {
                    cmd: "docker run [FLAGS] IMAGE [CMD]",
                    desc: "Create and start a container from an image",
                    example: "docker run -d --name web -p 8080:80 nginx\ndocker run -it ubuntu bash\ndocker run --rm python:3.11 python -c 'print(42)'",
                    tags: ["run", "container", "start", "create"]
                },
                {
                    cmd: "docker ps",
                    desc: "List running containers (-a for all)",
                    example: "docker ps\ndocker ps -a  # All including stopped\ndocker ps -q  # IDs only",
                    tags: ["ps", "list", "containers", "running"]
                },
                {
                    cmd: "docker stop CONTAINER",
                    desc: "Gracefully stop a running container",
                    example: "docker stop my-container\ndocker stop $(docker ps -q)  # Stop all",
                    tags: ["stop", "container"]
                },
                {
                    cmd: "docker rm CONTAINER",
                    desc: "Remove a stopped container",
                    example: "docker rm my-container\ndocker rm -f my-container  # Force stop + remove\ndocker container prune  # Remove all stopped",
                    tags: ["rm", "remove", "container", "delete"]
                },
                {
                    cmd: "docker exec -it CONTAINER CMD",
                    desc: "Execute a command in a running container",
                    example: "docker exec -it my-container bash\ndocker exec my-container ls /app\ndocker exec -e VAR=val my-container printenv",
                    tags: ["exec", "shell", "interactive", "container"]
                },
                {
                    cmd: "docker logs CONTAINER",
                    desc: "View container log output",
                    example: "docker logs my-container\ndocker logs -f my-container  # Follow\ndocker logs --tail 50 --since 30m my-container",
                    tags: ["logs", "output", "debug", "container"]
                },
                {
                    cmd: "docker inspect CONTAINER",
                    desc: "Return detailed JSON metadata about a container",
                    example: "docker inspect my-container\ndocker inspect -f '{{.State.Status}}' my-container",
                    tags: ["inspect", "metadata", "container"]
                },
                {
                    cmd: "docker stats",
                    desc: "Display live resource usage statistics",
                    example: "docker stats\ndocker stats --no-stream  # One-time snapshot\ndocker stats my-container",
                    tags: ["stats", "cpu", "memory", "resources"]
                }
            ],
            "Volumes": [
                {
                    cmd: "docker volume create NAME",
                    desc: "Create a named volume",
                    example: "docker volume create myvolume\ndocker volume create --label project=nexus db-data",
                    tags: ["volume", "create", "storage"]
                },
                {
                    cmd: "docker volume ls",
                    desc: "List all volumes",
                    example: "docker volume ls\ndocker volume ls -q  # Names only",
                    tags: ["volume", "list", "ls"]
                },
                {
                    cmd: "docker volume inspect NAME",
                    desc: "Show volume details including mountpoint",
                    example: "docker volume inspect myvolume",
                    tags: ["volume", "inspect", "mountpoint"]
                },
                {
                    cmd: "docker volume rm NAME",
                    desc: "Remove a volume (must not be in use)",
                    example: "docker volume rm myvolume\ndocker volume prune  # Remove all unused",
                    tags: ["volume", "remove", "rm", "delete"]
                }
            ],
            "Networks": [
                {
                    cmd: "docker network ls",
                    desc: "List all Docker networks",
                    example: "docker network ls\ndocker network ls --filter driver=bridge",
                    tags: ["network", "list", "ls"]
                },
                {
                    cmd: "docker network create NAME",
                    desc: "Create a new network",
                    example: "docker network create mynet\ndocker network create --subnet 10.10.0.0/24 mynet",
                    tags: ["network", "create"]
                },
                {
                    cmd: "docker network connect NET CONTAINER",
                    desc: "Connect a container to a network",
                    example: "docker network connect mynet my-container\ndocker network connect --ip 10.10.0.50 mynet my-container",
                    tags: ["network", "connect"]
                },
                {
                    cmd: "docker network inspect NAME",
                    desc: "Show detailed network info including connected containers",
                    example: "docker network inspect bridge\ndocker network inspect mynet",
                    tags: ["network", "inspect", "containers", "ip"]
                }
            ],
            "Compose": [
                {
                    cmd: "docker-compose up -d",
                    desc: "Start all services in detached mode",
                    example: "docker-compose up -d\ndocker-compose up -d --build  # Rebuild first\ndocker-compose up -d --scale api=3",
                    tags: ["compose", "up", "start"]
                },
                {
                    cmd: "docker-compose down",
                    desc: "Stop and remove containers and networks",
                    example: "docker-compose down\ndocker-compose down -v  # + remove volumes\ndocker-compose down --rmi all  # + remove images",
                    tags: ["compose", "down", "stop", "remove"]
                },
                {
                    cmd: "docker-compose logs -f",
                    desc: "Follow log output from all services",
                    example: "docker-compose logs -f\ndocker-compose logs -f api  # Specific service\ndocker-compose logs --tail 50",
                    tags: ["compose", "logs", "follow"]
                },
                {
                    cmd: "docker-compose ps",
                    desc: "List status of all Compose services",
                    example: "docker-compose ps",
                    tags: ["compose", "ps", "status", "list"]
                },
                {
                    cmd: "docker-compose exec SERVICE CMD",
                    desc: "Execute a command in a running Compose service",
                    example: "docker-compose exec api bash\ndocker-compose exec db psql -U postgres",
                    tags: ["compose", "exec", "shell"]
                },
                {
                    cmd: "docker-compose build",
                    desc: "Build or rebuild Compose service images",
                    example: "docker-compose build\ndocker-compose build api  # Specific service",
                    tags: ["compose", "build", "image"]
                }
            ],
            "System": [
                {
                    cmd: "docker system prune",
                    desc: "Remove all unused Docker resources",
                    example: "docker system prune\ndocker system prune -a  # Include unused images\ndocker system prune -a --volumes  # + volumes",
                    tags: ["system", "prune", "cleanup", "disk"]
                },
                {
                    cmd: "docker system df",
                    desc: "Show Docker disk usage",
                    example: "docker system df\ndocker system df -v  # Verbose",
                    tags: ["system", "df", "disk", "space"]
                },
                {
                    cmd: "docker info",
                    desc: "Display system-wide Docker information",
                    example: "docker info",
                    tags: ["info", "system", "version"]
                },
                {
                    cmd: "docker events",
                    desc: "Stream real-time Docker events",
                    example: "docker events\ndocker events --filter type=container",
                    tags: ["events", "monitor", "real-time"]
                }
            ]
        };

        function renderCommands(filter = "") {
            const container = document.getElementById("cheatsheet");
            container.innerHTML = "";
            const q = filter.toLowerCase();

            for (const [section, cmds] of Object.entries(commands)) {
                const visible = cmds.filter(c =>
                    !q || c.cmd.toLowerCase().includes(q) ||
                    c.desc.toLowerCase().includes(q) ||
                    c.tags.some(t => t.includes(q))
                );
                if (!visible.length) continue;

                const sec = document.createElement("div");
                sec.className = "section";
                sec.innerHTML = `<div class="section-title">${section}</div>`;

                const grid = document.createElement("div");
                grid.className = "commands-grid";

                visible.forEach(c => {
                    const card = document.createElement("div");
                    card.className = "command-card";
                    const examples = c.example.split("\n").map(line =>
                        `<div class="cmd-example"><button class="copy-btn" onclick="copyText(event, '${line.replace(/'/g,"\\'")}')">copy</button>${line}</div>`
                    ).join("");
                    card.innerHTML = `
                        <div class="cmd-name">${c.cmd}</div>
                        <div class="cmd-description">${c.desc}</div>
                        <div class="cmd-detail">${examples}</div>
                    `;
                    card.addEventListener("click", () => card.classList.toggle("active"));
                    grid.appendChild(card);
                });

                sec.appendChild(grid);
                container.appendChild(sec);
            }
        }

        function filterCommands() {
            renderCommands(document.getElementById("search").value);
        }

        function copyText(e, text) {
            e.stopPropagation();
            navigator.clipboard.writeText(text);
            e.target.textContent = "✓";
            setTimeout(() => e.target.textContent = "copy", 1500);
        }

        renderCommands();
    </script>
</body>
</html>
```

```bash
# Save the HTML file and open it
# On Linux:
xdg-open docker-cheatsheet.html

# On macOS:
open docker-cheatsheet.html

# Or serve it:
docker run -d -p 8888:80 \
    -v $(pwd)/docker-cheatsheet.html:/usr/share/nginx/html/index.html \
    nginx
# Open: http://localhost:8888
```

---

## Sharing Your Cheat Sheet

Once built, share your cheat sheet to get feedback and help others:

**Where to share:**
- **LinkedIn** — post the cheat sheet as an image or PDF with a summary
- **Dev.to** — write a blog post version with explanations
- **GitHub** — create a public repository with your `README.md` cheat sheet
- **Twitter/X** — share the most useful commands with `#Docker #DevOps`

**Suggested LinkedIn caption:**
```
🐳 After completing 8 days of Docker (Days 24–31 of #100DaysOfDevOps),
I created this personal Docker cheat sheet covering:

📦 Image management
🏃 Container lifecycle
🌐 Networking
💾 Volumes
🔧 Docker Compose
📋 Dockerfile best practices
🛠️ Troubleshooting

Key lesson: Docker isn't just about running containers —
it's about designing reproducible, isolated, and portable
environments for every stage of the software lifecycle.

What's your most-used Docker command? Drop it below 👇

#Docker #Containerization #DevOps #LearningInPublic
#100DaysOfDevOps #CloudNative
```

---

## 📖 Key Terms Summary

| Term | One-Line Definition |
|---|---|
| **Container** | A running instance of a Docker image |
| **Image** | A read-only blueprint for creating containers |
| **Dockerfile** | Instructions file for building a Docker image |
| **Registry** | Storage and distribution service for images |
| **Layer** | A read-only filesystem snapshot created by one Dockerfile instruction |
| **Volume** | Docker-managed persistent storage outside the container lifecycle |
| **Bind Mount** | A host directory mapped into a container |
| **Bridge Network** | Default Docker network with isolated virtual bridge |
| **Docker Compose** | Tool for defining multi-container apps in a YAML file |
| **Multi-stage Build** | Dockerfile with multiple FROM stages for lean production images |
| **`.dockerignore`** | File excluding paths from the build context |
| **ENTRYPOINT** | The main process of a container (harder to override) |
| **CMD** | Default arguments for ENTRYPOINT, or the default command |
| **`docker system prune`** | Removes all unused containers, images, networks, and volumes |
| **Health check** | A test Docker runs to determine if a service is healthy |

---

## 🔭 What Comes Next

With Docker fundamentals complete, the next chapter is **CI/CD** — connecting your Docker knowledge to automated pipelines.

**Upcoming sessions:**
- Jenkins — CI/CD automation server running in Docker
- GitHub Actions — cloud-native CI/CD triggered by Git events
- Kubernetes basics — container orchestration at scale
- Infrastructure as Code — Terraform for cloud resources
- AWS integration — EC2, ECR, ECS, and deploying containers to the cloud

Every `Dockerfile` you write, every `docker-compose.yml` you define, every image you push — these become the building blocks of fully automated delivery pipelines.

---

> 💬 *"A cheat sheet is not a shortcut — it is a map of territory you have already explored. The best ones are built by people who needed them before they knew they needed them."*
>
> — DevOps Learning by Yukta
