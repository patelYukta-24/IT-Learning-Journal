# Day 30: Building Custom Docker Images — Dockerfile Basics

> **DevOps Transformation** | Containers & Automation Series

---

## 📋 Overview

Until now you have been using images built by others — pulling `nginx`, `ubuntu`, `postgres` from Docker Hub and running them. Today you cross the line from consumer to creator.

A **Dockerfile** is a plain-text script containing every instruction needed to build a custom Docker image. It is the source code of your container environment — version-controlled, reviewable, and reproducible. Every production container at every serious engineering team starts with a Dockerfile.

By the end of today you will have written Dockerfiles from scratch, built custom images, optimized them using Alpine base images and `.dockerignore`, used multi-stage builds to create lean production images, and pushed your image to Docker Hub.

---

## 🗺️ What You'll Do Today

| Topic | Commands / Concepts | What You Learn |
|---|---|---|
| Dockerfile directives | `FROM`, `RUN`, `COPY`, `ADD`, `WORKDIR`, `ENV`, `EXPOSE`, `CMD`, `ENTRYPOINT` | All the instructions available in a Dockerfile |
| Build an image | `docker build -t name:tag .` | Turn a Dockerfile into a usable image |
| Run your custom image | `docker run -p` | Test what you built |
| Layer caching | — | How Docker optimizes rebuilds |
| Image optimization | Alpine variants, `.dockerignore` | Reduce image size dramatically |
| Multi-stage builds | `FROM ... AS builder` | Separate build environment from runtime |
| Tag and push | `docker tag`, `docker push` | Publish images to Docker Hub |
| Custom MySQL image | Pre-loaded data | Real-world Dockerfile task |

---

## 🏢 NexusCorp Scenario

> NexusCorp's new microservice needs a custom Python runtime image — specific dependencies, a non-root user for security, and a pre-configured working directory. The base `python:3.11` image doesn't have these. A Dockerfile is the answer. Once written, any engineer on the team can build the identical environment with `docker build`. No setup docs. No "it works on my machine." Just the Dockerfile.

---

## Part 1: Introduction to Dockerfile

### What Is a Dockerfile?

A Dockerfile is a plain-text file (literally named `Dockerfile`, no extension) containing a sequence of instructions. Each instruction tells Docker to perform a specific action when building the image — install a package, copy a file, set an environment variable, expose a port.

```
Dockerfile (recipe)
      │
      │ docker build
      ▼
Docker Image (frozen blueprint)
      │
      │ docker run
      ▼
Container (live instance)
```

### Understanding Base Images

Every Dockerfile starts with a `FROM` instruction specifying a **base image** — the starting point your image is built on top of. The base image can be:

| Base Image | Size | Use Case |
|---|---|---|
| `ubuntu:22.04` | ~78MB | Full Ubuntu — general purpose |
| `debian:bookworm-slim` | ~75MB | Lean Debian — good for production |
| `alpine:3.18` | ~7MB | Minimal — smallest possible |
| `python:3.11` | ~1GB | Python + Debian — full dev environment |
| `python:3.11-slim` | ~130MB | Python + minimal Debian |
| `python:3.11-alpine` | ~50MB | Python + Alpine — smallest Python image |
| `node:20-alpine` | ~180MB | Node.js + Alpine |
| `scratch` | 0MB | Empty — for static binaries only |

**Choosing the right base image:**
- Development: use full images (`python:3.11`) — debugging tools are available
- Production: use slim or Alpine (`python:3.11-slim`, `python:3.11-alpine`) — smaller, fewer vulnerabilities
- Compiled languages (Go, Rust): use multi-stage builds with `scratch` or `alpine` as final stage

---

### Understanding Image Layers

Every instruction in a Dockerfile creates a new **read-only layer** in the image. Layers are cached and reused — if nothing changes in a layer, Docker reuses the cached version on the next build.

```
Layer 4: COPY . .                  ← Your app code (changes often)
Layer 3: RUN pip install -r ...    ← Dependencies (changes sometimes)
Layer 2: COPY requirements.txt .   ← Requirements file
Layer 1: FROM python:3.11-slim     ← Base image (rarely changes)
```

**The caching rule:** When a layer changes, all layers above it are invalidated and rebuilt. This is why you should:
1. Put instructions that change rarely at the top
2. Put instructions that change often at the bottom
3. Copy dependency files before copying application code

```dockerfile
# GOOD — requirements.txt changes less often than app code
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .                    # Only this layer re-runs when code changes

# BAD — invalidates pip install every time any file changes
COPY . .
RUN pip install -r requirements.txt
```

---

## Part 2: Dockerfile Directives — Complete Reference

### `FROM` — Set the Base Image

```dockerfile
# Basic
FROM python:3.11-slim

# With a specific digest (most reproducible — pinned to exact image)
FROM python:3.11-slim@sha256:abc123...

# Named stage (for multi-stage builds)
FROM python:3.11-slim AS builder
FROM python:3.11-alpine AS production

# Start from scratch (empty filesystem)
FROM scratch
```

---

### `RUN` — Execute Commands During Build

```dockerfile
# Shell form (runs via /bin/sh -c)
RUN apt-get update && apt-get install -y curl

# Exec form (no shell — more reliable for complex args)
RUN ["apt-get", "install", "-y", "curl"]

# Best practice — chain related commands to reduce layers
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        curl \
        git \
        vim \
    && rm -rf /var/lib/apt/lists/*    # Clean apt cache in same layer
```

**Why `&&` and cleanup in one `RUN`:**
Each `RUN` creates a layer. If you `apt-get update` in one layer and `apt-get clean` in another, the cache files still exist in the first layer — they are not removed. Chaining with `&&` and cleaning up in the same `RUN` keeps the layer lean.

---

### `COPY` — Copy Files from Build Context

```dockerfile
# Copy a single file
COPY requirements.txt .

# Copy multiple files
COPY package.json package-lock.json ./

# Copy a directory
COPY src/ /app/src/

# Copy with ownership (--chown)
COPY --chown=appuser:appgroup . /app
```

---

### `ADD` — Copy Files with Extra Features

```dockerfile
# ADD does everything COPY does, plus:
# - Automatically extracts .tar.gz archives
# - Can download from URLs (not recommended — use curl/wget in RUN instead)

ADD archive.tar.gz /app/         # Extracts into /app/
ADD https://example.com/file .   # Downloads file (avoid — not cacheable)
```

**Best practice:** Use `COPY` by default. Use `ADD` only when you need automatic tar extraction.

---

### `WORKDIR` — Set Working Directory

```dockerfile
# Set the working directory for all subsequent RUN, COPY, CMD, ENTRYPOINT
WORKDIR /app

# If the directory doesn't exist, Docker creates it
# This replaces using RUN mkdir /app && cd /app
```

---

### `ENV` — Set Environment Variables

```dockerfile
# Single variable
ENV FLASK_ENV=production

# Multiple variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=5000

# Available during build AND at runtime in containers
```

---

### `EXPOSE` — Document Port Intentions

```dockerfile
# Document which port the container listens on
EXPOSE 80
EXPOSE 5000
EXPOSE 5432

# NOTE: EXPOSE is documentation only — it does NOT publish the port
# Port publishing still requires -p flag at docker run time
```

---

### `CMD` — Default Command for the Container

```dockerfile
# Exec form (preferred)
CMD ["python", "app.py"]
CMD ["nginx", "-g", "daemon off;"]
CMD ["flask", "run", "--host=0.0.0.0"]

# Shell form
CMD python app.py

# CMD is the default — it can be overridden at docker run time:
# docker run myimage python other_script.py  ← overrides CMD
```

---

### `ENTRYPOINT` — Container's Main Process

```dockerfile
# Exec form (preferred)
ENTRYPOINT ["python", "app.py"]

# ENTRYPOINT is NOT easily overridden (unlike CMD)
# CMD provides default arguments to ENTRYPOINT when both are set

# Combined pattern — ENTRYPOINT defines the executable, CMD the default args
ENTRYPOINT ["gunicorn"]
CMD ["--bind", "0.0.0.0:5000", "app:application"]
# Run: gunicorn --bind 0.0.0.0:5000 app:application
# Override: docker run myimage --bind 0.0.0.0:8080 app:application
```

---

### `USER` — Set Runtime User

```dockerfile
# Create a non-root user (security best practice)
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

# Switch to non-root user for all subsequent instructions
USER appuser

# All CMD, ENTRYPOINT, and RUN commands after this run as appuser
```

---

### `ARG` — Build-Time Variables

```dockerfile
# Define a build argument (only available during build, not at runtime)
ARG PYTHON_VERSION=3.11
ARG APP_VERSION=1.0.0

FROM python:${PYTHON_VERSION}-slim

# Pass at build time:
# docker build --build-arg APP_VERSION=2.0.0 .
```

---

### `HEALTHCHECK` — Container Health Monitoring

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# Docker checks health and reports:
# (healthy), (unhealthy), (starting)
```

---

### Directive Summary Table

| Directive | Purpose | Build or Runtime? |
|---|---|---|
| `FROM` | Set base image | Build |
| `RUN` | Execute command during build | Build only |
| `COPY` | Copy files from build context | Build |
| `ADD` | Copy + extract/download | Build |
| `WORKDIR` | Set working directory | Both |
| `ENV` | Set environment variable | Both |
| `EXPOSE` | Document port (no actual binding) | Documentation |
| `CMD` | Default container command | Runtime |
| `ENTRYPOINT` | Main container process | Runtime |
| `USER` | Set user for subsequent instructions | Both |
| `ARG` | Build-time variable | Build only |
| `HEALTHCHECK` | Define container health test | Runtime |
| `VOLUME` | Declare mount point | Runtime |
| `LABEL` | Add metadata to image | Build |
| `ONBUILD` | Instructions for child images | Build (deferred) |

---

## Part 3: Task 1 — Your First Dockerfile (Flask App)

### Project Structure

```bash
mkdir nexus-flask-image
cd nexus-flask-image
```

### Application Files

```python
# app.py
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        'message': 'Hello from NexusCorp!',
        'environment': os.getenv('APP_ENV', 'development'),
        'version': '1.0.0'
    })

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

```txt
# requirements.txt
flask==3.0.0
```

### The Dockerfile

```dockerfile
# Dockerfile

# Step 1: Choose base image
FROM python:3.11-slim

# Step 2: Set metadata
LABEL maintainer="devops@nexuscorp.com"
LABEL version="1.0.0"
LABEL description="NexusCorp Flask Application"

# Step 3: Set working directory
WORKDIR /app

# Step 4: Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    APP_ENV=production

# Step 5: Install dependencies BEFORE copying app code (caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Step 6: Copy application code
COPY app.py .

# Step 7: Create non-root user (security)
RUN groupadd -r nexus && useradd -r -g nexus nexus
USER nexus

# Step 8: Document the port
EXPOSE 80

# Step 9: Define the default command
CMD ["python", "app.py"]
```

---

### Task 2: Build the Image

```bash
# Build the image
docker build -t flask-app:latest .

# Watch each step — notice each FROM/RUN/COPY creates a layer

# Build with verbose output
docker build --progress=plain -t flask-app:latest .

# Build with a specific tag (version)
docker build -t flask-app:1.0.0 -t flask-app:latest .

# View the built image
docker images flask-app
```

**Expected build output:**
```
[+] Building 12.3s (10/10) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [1/6] FROM python:3.11-slim
 => [2/6] WORKDIR /app
 => [3/6] COPY requirements.txt .
 => [4/6] RUN pip install --no-cache-dir -r requirements.txt
 => [5/6] COPY app.py .
 => [6/6] RUN groupadd -r nexus && useradd -r -g nexus nexus
 => exporting to image
 => naming to docker.io/library/flask-app:latest
```

**Observe layer caching — rebuild without changes:**
```bash
docker build -t flask-app:latest .
# All steps show: CACHED
# Total time: < 1 second
```

**Observe cache invalidation — change app.py then rebuild:**
```bash
echo "# comment" >> app.py
docker build -t flask-app:latest .
# Steps 1-4: CACHED (nothing changed before COPY app.py)
# Step 5: COPY app.py . — rebuilt
# Step 6: CMD — rebuilt
# Steps 1-4 are still cached — this is why dependency order matters
```

---

### Task 3: Run Your Custom Image

```bash
# Run the Flask app — map host port 4000 to container port 80
docker run -p 4000:80 flask-app:latest

# Run in detached mode with a name
docker run -d --name nexus-flask -p 4000:80 flask-app:latest

# Test it
curl http://localhost:4000
# {"environment":"production","message":"Hello from NexusCorp!","version":"1.0.0"}

curl http://localhost:4000/health
# {"status":"healthy"}

# Override the environment variable at runtime
docker run -d \
    --name nexus-flask-staging \
    -p 4001:80 \
    -e APP_ENV=staging \
    flask-app:latest

curl http://localhost:4001
# {"environment":"staging","message":"Hello from NexusCorp!","version":"1.0.0"}

# View image history — see all layers
docker history flask-app:latest

# Inspect the image
docker inspect flask-app:latest
```

---

## Part 4: Image Optimization

### Using `.dockerignore`

Like `.gitignore`, a `.dockerignore` file tells Docker which files to exclude from the **build context** — the set of files sent to the Docker daemon when building.

```bash
# .dockerignore
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
.env
.env.*
venv/
.venv/
env/
*.log
.git/
.gitignore
README.md
docs/
tests/
.DS_Store
Thumbs.db
*.md
Dockerfile
docker-compose*.yml
.dockerignore
```

**Why it matters:**
```
Without .dockerignore:              With .dockerignore:
Build context: 450MB                Build context: 12MB
(includes .git, venv, node_modules) (only app code and config)
Build time: 45 seconds              Build time: 8 seconds
```

---

### Optimizing with Alpine Base Images

```dockerfile
# Original: python:3.11-slim
FROM python:3.11-slim
# Image size result: ~180MB

# Optimized: python:3.11-alpine
FROM python:3.11-alpine
RUN apk add --no-cache gcc musl-dev    # Alpine uses apk, not apt
# Image size result: ~65MB
```

```bash
# Compare sizes
docker build -t flask-slim:latest -f Dockerfile.slim .
docker build -t flask-alpine:latest -f Dockerfile.alpine .
docker images | grep flask
# flask-slim    latest    ...    180MB
# flask-alpine  latest    ...     65MB
```

**Alpine caveats:**
- Uses `apk` instead of `apt-get`
- Uses `musl libc` instead of `glibc` — some Python packages need `gcc` and `musl-dev` to compile
- Smaller attack surface — fewer pre-installed packages

---

## Part 5: Multi-Stage Builds

Multi-stage builds use multiple `FROM` statements in one Dockerfile. Each stage can have a name and build on the previous. The final image only contains what you explicitly copy from earlier stages — all the build tools, compilers, and temporary files are left behind.

### Why Multi-Stage Builds?

```
Single-stage build (Python example):
  python:3.11 base + build tools + app + dependencies = 1.2GB

Multi-stage build:
  Stage 1 (builder): Install everything needed to compile/build
  Stage 2 (final): Copy only the compiled output and runtime
  Result: 130MB final image
```

---

### Multi-Stage Flask Example

```dockerfile
# Dockerfile.multistage

# ── Stage 1: Builder ──────────────────────────────────────────
FROM python:3.11-slim AS builder

WORKDIR /build

# Install build tools
RUN apt-get update && apt-get install -y --no-install-recommends \
        gcc \
        libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies into a virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt


# ── Stage 2: Production Runtime ───────────────────────────────
FROM python:3.11-slim AS production

# Copy only the virtual environment from builder — no gcc, no build tools
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Set up application
WORKDIR /app
COPY app.py .

# Security: non-root user
RUN groupadd -r nexus && useradd -r -g nexus nexus
USER nexus

EXPOSE 80
CMD ["python", "app.py"]
```

```bash
# Build the multi-stage image
docker build -f Dockerfile.multistage -t flask-optimized:latest .

# Compare sizes
docker images | grep flask
# flask-app          latest    ...    180MB    (single stage)
# flask-optimized    latest    ...    135MB    (multi-stage — no build tools)
```

---

### Multi-Stage Build for a Go Application

Multi-stage builds shine brightest for compiled languages:

```dockerfile
# ── Stage 1: Compile ──────────────────────────────────────────
FROM golang:1.21-alpine AS builder

WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server ./cmd/server


# ── Stage 2: Minimal Runtime ──────────────────────────────────
FROM scratch AS production
# 'scratch' is empty — 0 bytes

COPY --from=builder /app/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]

# Result: ~10MB final image (just the compiled binary)
# vs ~400MB if built in a single stage with the full Go toolchain
```

---

## Part 6: Real-World Task — Customized MySQL Image

**Scenario:** NexusCorp's staging environment needs a MySQL database pre-loaded with the schema and seed data every time a container starts. Instead of manually running SQL after every `docker run`, the data is baked into the image startup.

### How the Official MySQL Image Works

The official `mysql` image has a special directory: `/docker-entrypoint-initdb.d/`. Any `.sql`, `.sh`, or `.sql.gz` file placed here is automatically executed when the container starts for the first time (only if the data directory is empty).

### Project Structure

```
nexus-mysql/
├── Dockerfile
├── init/
│   ├── 01-schema.sql
│   └── 02-seed-data.sql
└── .dockerignore
```

### SQL Files

```sql
-- init/01-schema.sql
CREATE DATABASE IF NOT EXISTS nexusdb;
USE nexusdb;

CREATE TABLE IF NOT EXISTS servers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    ip_address VARCHAR(15) NOT NULL,
    environment VARCHAR(20) NOT NULL,
    status VARCHAR(20) DEFAULT 'running',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS deployments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    server_id INT,
    app_name VARCHAR(50) NOT NULL,
    version VARCHAR(20) NOT NULL,
    deployed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (server_id) REFERENCES servers(id)
);
```

```sql
-- init/02-seed-data.sql
USE nexusdb;

INSERT INTO servers (name, ip_address, environment, status) VALUES
    ('web-01', '192.168.1.10', 'production', 'running'),
    ('web-02', '192.168.1.11', 'production', 'running'),
    ('db-01',  '192.168.1.20', 'production', 'running'),
    ('cache-01','192.168.1.30','staging',    'running');

INSERT INTO deployments (server_id, app_name, version) VALUES
    (1, 'nexus-api', 'v2.1.0'),
    (2, 'nexus-api', 'v2.1.0'),
    (1, 'nexus-web', 'v1.8.3');
```

### The Dockerfile

```dockerfile
# Dockerfile
FROM mysql:8.0

# Set metadata
LABEL maintainer="devops@nexuscorp.com"
LABEL description="NexusCorp pre-loaded MySQL image"

# Set default environment variables (can be overridden at docker run)
ENV MYSQL_DATABASE=nexusdb \
    MYSQL_ROOT_PASSWORD=changeme

# Copy initialization scripts into the MySQL init directory
# Scripts are executed alphabetically on first startup
COPY init/01-schema.sql /docker-entrypoint-initdb.d/
COPY init/02-seed-data.sql /docker-entrypoint-initdb.d/

# MySQL's default port
EXPOSE 3306
```

### Build and Test

```bash
# Build the custom MySQL image
docker build -t nexus-mysql:1.0.0 .

# Run it
docker run -d \
    --name nexus-db \
    -e MYSQL_ROOT_PASSWORD=nexuspass \
    -p 3306:3306 \
    nexus-mysql:1.0.0

# Wait for initialization
echo "Waiting for MySQL to initialize..."
sleep 20

# Verify the data was loaded
docker exec nexus-db mysql -uroot -pnexuspass \
    -e "USE nexusdb; SELECT * FROM servers;"
# All 4 seed rows should be present

docker exec nexus-db mysql -uroot -pnexuspass \
    -e "USE nexusdb; SELECT * FROM deployments;"
# All 3 deployment records should be present

# Clean up
docker stop nexus-db && docker rm nexus-db
```

---

## Part 7: Tagging and Pushing Images to Docker Hub

### Tagging

```bash
# Tag syntax: docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

# Add a version tag
docker tag flask-app:latest flask-app:1.0.0

# Tag for Docker Hub (format: username/repository:tag)
docker tag flask-app:latest yourusername/nexus-flask-app:latest
docker tag flask-app:latest yourusername/nexus-flask-app:1.0.0

# View all tags for an image
docker images yourusername/nexus-flask-app
```

### Pushing to Docker Hub

```bash
# Step 1: Log in to Docker Hub
docker login
# Enter your Docker Hub username and password

# Step 2: Push the image
docker push yourusername/nexus-flask-app:latest
docker push yourusername/nexus-flask-app:1.0.0

# Step 3: Verify — pull from Docker Hub to confirm
docker pull yourusername/nexus-flask-app:latest

# Step 4: Anyone can now pull your image
docker run -p 4000:80 yourusername/nexus-flask-app:latest
```

### Tagging Best Practices

| Tag Pattern | Example | Use Case |
|---|---|---|
| `latest` | `myapp:latest` | Most recent stable version |
| Semantic version | `myapp:2.1.0` | Full version — most specific |
| Major.minor | `myapp:2.1` | Tracks patch updates automatically |
| Major only | `myapp:2` | Tracks minor updates automatically |
| Git SHA | `myapp:a1b2c3d` | Exact commit — most reproducible |
| Environment | `myapp:staging` | Environment-specific build |

---

## 🔧 Mini-Project: Complete NexusCorp Image Pipeline

```bash
#!/bin/bash
# image_pipeline.sh
# Build, optimize, test, tag, and inspect a production-ready image

set -e

APP_NAME="nexus-flask"
VERSION="1.0.0"
REGISTRY="yourusername"

echo "=============================================="
echo "  NexusCorp Image Build Pipeline"
echo "=============================================="

# ── Step 1: Build standard image ────────────────────────────────
echo ""
echo "Step 1: Building standard image..."
docker build -t ${APP_NAME}:${VERSION} -t ${APP_NAME}:latest .
SIZE_STANDARD=$(docker images ${APP_NAME}:${VERSION} \
    --format "{{.Size}}")
echo "  ✓ Built: ${APP_NAME}:${VERSION} (${SIZE_STANDARD})"

# ── Step 2: Build Alpine variant ────────────────────────────────
echo ""
echo "Step 2: Building Alpine optimized image..."
docker build -f Dockerfile.alpine \
    -t ${APP_NAME}-alpine:${VERSION} .
SIZE_ALPINE=$(docker images ${APP_NAME}-alpine:${VERSION} \
    --format "{{.Size}}")
echo "  ✓ Built: ${APP_NAME}-alpine:${VERSION} (${SIZE_ALPINE})"

# ── Step 3: Compare sizes ────────────────────────────────────────
echo ""
echo "Step 3: Size comparison:"
echo "  Standard : ${SIZE_STANDARD}"
echo "  Alpine   : ${SIZE_ALPINE}"

# ── Step 4: Test both images ────────────────────────────────────
echo ""
echo "Step 4: Testing images..."

docker run -d --name test-standard -p 4000:80 ${APP_NAME}:${VERSION}
sleep 3
RESPONSE=$(curl -s http://localhost:4000/health | python3 -m json.tool)
docker stop test-standard && docker rm test-standard
echo "  ✓ Standard image health check passed"
echo "    $RESPONSE"

# ── Step 5: Show image history ──────────────────────────────────
echo ""
echo "Step 5: Image layers:"
docker history ${APP_NAME}:${VERSION} --format \
    "table {{.CreatedBy}}\t{{.Size}}" | head -10

# ── Step 6: Tag for registry ────────────────────────────────────
echo ""
echo "Step 6: Tagging for Docker Hub..."
docker tag ${APP_NAME}:${VERSION} ${REGISTRY}/${APP_NAME}:${VERSION}
docker tag ${APP_NAME}:${VERSION} ${REGISTRY}/${APP_NAME}:latest
echo "  ✓ Tagged: ${REGISTRY}/${APP_NAME}:${VERSION}"
echo "  ✓ Tagged: ${REGISTRY}/${APP_NAME}:latest"

echo ""
echo "=============================================="
echo "  Pipeline complete!"
echo "  To push: docker push ${REGISTRY}/${APP_NAME}:${VERSION}"
echo "=============================================="
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Dockerfile** | A plain-text script containing instructions for building a Docker image |
| **`docker build`** | Command that reads a Dockerfile and creates an image |
| **`-t`** | Tag flag for `docker build` — assigns a name and tag to the built image |
| **Build context** | The set of files sent to the Docker daemon when building — the directory specified in `docker build` |
| **`.dockerignore`** | A file listing patterns to exclude from the build context |
| **Layer** | A read-only filesystem change created by each Dockerfile instruction |
| **Layer caching** | Docker's optimization that reuses unchanged layers from previous builds |
| **Cache invalidation** | When a layer changes, all subsequent layers must be rebuilt |
| **`FROM`** | Sets the base image for the Dockerfile |
| **`RUN`** | Executes a command during the image build process |
| **`COPY`** | Copies files from the build context into the image |
| **`ADD`** | Like COPY but also extracts archives and can download URLs |
| **`WORKDIR`** | Sets the working directory for subsequent instructions |
| **`ENV`** | Sets an environment variable available during build and at runtime |
| **`EXPOSE`** | Documents which port the container listens on (does not bind the port) |
| **`CMD`** | Default command executed when a container starts (overridable) |
| **`ENTRYPOINT`** | Main process of the container (harder to override than CMD) |
| **`USER`** | Sets the user for subsequent instructions and the container runtime |
| **`ARG`** | A build-time variable passed via `--build-arg` |
| **`HEALTHCHECK`** | Defines how Docker tests if a container is healthy |
| **`LABEL`** | Adds key-value metadata to an image |
| **Multi-stage build** | A Dockerfile with multiple `FROM` statements — each stage can copy artifacts from previous stages |
| **Alpine Linux** | A minimal Linux distribution (~7MB) widely used as a Docker base image |
| **`python:3.11-slim`** | Python image with minimal Debian — smaller than full Python image |
| **`python:3.11-alpine`** | Python image on Alpine base — smallest Python image |
| **`scratch`** | An empty base image — used for statically compiled binaries |
| **`docker tag`** | Assigns a new name/tag to an existing image |
| **`docker push`** | Uploads a local image to a registry |
| **`docker login`** | Authenticates with a Docker registry |
| **Semantic versioning** | Version format: MAJOR.MINOR.PATCH (e.g., `2.1.0`) |
| **`/docker-entrypoint-initdb.d/`** | MySQL's special directory — SQL scripts here run on first container startup |
| **Non-root user** | Running containers as a non-root user — a security best practice |
| **`--no-cache-dir`** | pip flag that prevents caching of downloaded packages — reduces image size |

---

## 📚 Resources

- [Dockerfile Reference — Official Docs](https://docs.docker.com/engine/reference/builder/) — every instruction documented
- [Docker Build Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) — official guide to writing efficient Dockerfiles
- [Multi-Stage Builds — Docker Docs](https://docs.docker.com/build/building/multi-stage/) — official multi-stage build guide
- [Docker Hub](https://hub.docker.com) — browse official base images
- [Dive](https://github.com/wagoodman/dive) — a tool for exploring Docker image layers
- [Hadolint](https://github.com/hadolint/hadolint) — Dockerfile linter that catches common mistakes

---

## 🔭 Day 31 Preview: Introduction to CI/CD with Jenkins

Every Dockerfile you write today is a building block in tomorrow's automation. Day 31 introduces **Jenkins** — the open-source automation server that connects Git commits to Docker builds to deployments.

You will learn:
- What CI/CD (Continuous Integration / Continuous Delivery) means in practice
- Running Jenkins itself in a Docker container
- Creating your first Jenkins pipeline
- Writing a `Jenkinsfile` — the pipeline-as-code equivalent of a Dockerfile
- Connecting Jenkins to a Git repository — automatic builds on every push
- A pipeline that runs: `git push` → `docker build` → `docker push` → `docker run`

Every tool from Days 1–30 is an ingredient in this pipeline. Tomorrow you wire them together.

---

> 💬 *"A Dockerfile is a Makefile for your environment. Once it exists, 'works on my machine' stops being an excuse — it becomes a Dockerfile bug."*
>
> — DevOps Learning By Yukta
