# Day 29: Docker Compose — Orchestrating Multi-Container Applications

> **DevOps Transformation** | Containers & Automation Series

---

## 📋 Overview

Over the past five days you've mastered individual Docker building blocks: images, containers, networking, and volumes. Today everything comes together.

Real-world applications are never a single container. A typical web application has a web server, an application server, a database, a cache, and maybe a message queue — each running in its own container, communicating over a network, persisting data to volumes. Managing all of that with individual `docker run` commands is error-prone, difficult to reproduce, and impossible to version-control.

**Docker Compose** solves this by letting you define your entire multi-container application in a single `docker-compose.yml` file. One file. One command. Your full application stack is up and running.

---

## 🗺️ What You'll Do Today

| Topic | Commands | What You Learn |
|---|---|---|
| Install Docker Compose | `docker compose version` | Confirm Compose is available |
| Write a Compose file | `docker-compose.yml` | Define services, networks, volumes in YAML |
| Run a stack | `docker-compose up` | Start all services with one command |
| Stop a stack | `docker-compose down` | Stop and clean up all services |
| Real-world deployment | WordPress + MySQL | Full production-style multi-container app |
| Compose commands | `ps`, `logs`, `start`, `stop`, `exec` | Day-to-day Compose operations |
| Service networking | Service name as hostname | How containers find each other in Compose |
| Environment variables | `environment:`, `.env` file | Configurable, secret-free Compose files |

---

## 🏢 NexusCorp Scenario

> NexusCorp's dev team is onboarding a new project: a Python Flask API backed by Redis for caching and PostgreSQL for persistence. Previously, three separate `docker run` commands with flags were copy-pasted from a Slack message. Half the time someone missed a flag and the stack wouldn't start. The new engineer asks: "Why don't we have a Compose file?" By the end of today, they do.

---

## Part 1: Understanding Docker Compose

### What Docker Compose Does

Docker Compose is a tool for defining and running multi-container Docker applications. Instead of running each container manually, you describe your entire application — services, networks, volumes, environment variables, port mappings — in a single YAML file called `docker-compose.yml`.

```
Without Compose:
docker network create myapp-net
docker volume create db-data
docker run -d --name postgres --network myapp-net -v db-data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret postgres:15
docker run -d --name redis --network myapp-net redis:7-alpine
docker run -d --name api --network myapp-net -p 5000:5000 -e DATABASE_URL=postgresql://postgres:secret@postgres/mydb myapi:latest
docker run -d --name nginx --network myapp-net -p 80:80 nginx

With Compose:
docker-compose up -d
```

One command replaces four, and everything is defined in a file you can commit to Git.

---

### The `docker-compose.yml` File Structure

```yaml
version: "3.8"          # Compose file format version

services:               # Define each container as a "service"
  service-name:
    image: image:tag    # OR build: ./path-to-dockerfile
    ports:
      - "host:container"
    environment:
      - KEY=VALUE
    volumes:
      - volume-name:/path/in/container
    networks:
      - network-name
    depends_on:
      - other-service

networks:               # Define custom networks
  network-name:
    driver: bridge

volumes:                # Define named volumes
  volume-name:
```

---

### Installing Docker Compose

**Docker Compose V2** is now bundled with Docker Engine as a plugin. If you installed Docker Engine recently, you likely already have it.

```bash
# Check if Compose is available (V2 — plugin style)
docker compose version
# Output: Docker Compose version v2.21.0

# Or the legacy standalone binary (V1)
docker-compose --version
# Output: docker-compose version 1.29.2

# Install on Ubuntu/Debian if missing
sudo apt update
sudo apt install -y docker-compose-plugin

# Verify
docker compose version
```

**V1 vs V2 — what changed:**

| | Docker Compose V1 | Docker Compose V2 |
|---|---|---|
| Command | `docker-compose` (hyphen) | `docker compose` (space) |
| Installation | Separate binary | Docker plugin (bundled) |
| Status | Legacy, deprecated | Current — use this |
| File format | Same `docker-compose.yml` | Same `docker-compose.yml` |

Both syntaxes work for this curriculum — the examples use V2 (`docker compose`) where possible, with V1 (`docker-compose`) equivalents noted.

---

## Part 2: Writing Docker Compose Files

### Core `docker-compose.yml` Fields

#### `services`

Each service is a container definition. Services replace individual `docker run` commands.

```yaml
services:
  web:
    image: nginx:1.25              # Use an existing image
    ports:
      - "8080:80"                  # host_port:container_port
    restart: always                # Restart policy

  api:
    build: ./api                   # Build from local Dockerfile
    ports:
      - "5000:5000"
    environment:                   # Environment variables
      - FLASK_ENV=development
      - DATABASE_URL=postgresql://postgres:secret@db/nexusdb
    depends_on:
      - db                         # Start db before api
    volumes:
      - ./api:/app                 # Bind mount for development

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: nexusdb
    volumes:
      - db-data:/var/lib/postgresql/data   # Named volume
```

#### `networks`

```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

services:
  nginx:
    networks:
      - frontend
  api:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend
```

#### `volumes`

```yaml
volumes:
  db-data:           # Docker-managed named volume
  redis-data:
  uploads:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/nfs/uploads   # NFS or specific path
```

#### `depends_on`

```yaml
services:
  api:
    depends_on:
      db:
        condition: service_healthy  # Wait for healthcheck to pass
      redis:
        condition: service_started  # Just wait for it to start

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

### Environment Variables in Compose

There are three ways to pass environment variables in Compose:

```yaml
# Method 1: Inline in the Compose file (not for secrets)
services:
  api:
    environment:
      - FLASK_ENV=development
      - LOG_LEVEL=INFO

# Method 2: Using a .env file (auto-loaded from same directory)
# .env file:
# DB_PASSWORD=secret
# API_PORT=5000

services:
  api:
    environment:
      - DB_PASSWORD=${DB_PASSWORD}    # Reads from .env
      - API_PORT=${API_PORT}

# Method 3: env_file directive (use a specific file)
services:
  api:
    env_file:
      - ./config/api.env
      - ./config/db.env
```

**The `.env` file pattern — best practice:**

```bash
# .env (never commit to Git — add to .gitignore)
POSTGRES_PASSWORD=super-secret-password
POSTGRES_DB=nexusdb
REDIS_PASSWORD=redis-secret
API_SECRET_KEY=your-secret-key-here
```

```yaml
# docker-compose.yml (safe to commit)
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
```

---

## Part 3: Task — Simple Flask + Redis Application

### Step 1: Create the Project Structure

```bash
mkdir nexus-flask-app
cd nexus-flask-app

mkdir app
```

### Step 2: Write the Flask Application

```python
# app/app.py
from flask import Flask, jsonify
import redis
import os

app = Flask(__name__)

# Redis connection — uses service name 'redis' as hostname
redis_client = redis.Redis(
    host=os.getenv('REDIS_HOST', 'redis'),
    port=int(os.getenv('REDIS_PORT', 6379)),
    decode_responses=True
)

@app.route('/')
def index():
    # Increment visit counter in Redis
    visits = redis_client.incr('visit_count')
    return jsonify({
        'message': 'NexusCorp Flask API',
        'visits': visits,
        'redis_host': os.getenv('REDIS_HOST', 'redis')
    })

@app.route('/health')
def health():
    try:
        redis_client.ping()
        return jsonify({'status': 'healthy', 'redis': 'connected'})
    except Exception as e:
        return jsonify({'status': 'unhealthy', 'error': str(e)}), 503

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

### Step 3: Write the Dockerfile

```dockerfile
# app/Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

### Step 4: Write Requirements

```txt
# app/requirements.txt
flask==3.0.0
redis==5.0.1
```

### Step 5: Write the Compose File

```yaml
# docker-compose.yml
version: "3.8"

services:
  flask:
    build: ./app
    container_name: nexus-flask
    ports:
      - "5000:5000"
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - FLASK_ENV=development
    depends_on:
      - redis
    restart: unless-stopped
    networks:
      - nexus-net

  redis:
    image: redis:7-alpine
    container_name: nexus-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    restart: unless-stopped
    networks:
      - nexus-net

networks:
  nexus-net:
    driver: bridge

volumes:
  redis-data:
```

### Step 6: Run the Stack

```bash
# Start all services (foreground — see logs)
docker-compose up

# Start all services (detached — background)
docker-compose up -d

# Build images before starting (if Dockerfiles changed)
docker-compose up -d --build

# Verify services are running
docker-compose ps
```

**Expected output of `docker-compose ps`:**
```
NAME            IMAGE              COMMAND             STATUS    PORTS
nexus-flask     nexus-flask-app_flask  "python app.py"     Up        0.0.0.0:5000->5000/tcp
nexus-redis     redis:7-alpine     "docker-entrypoint…"  Up        0.0.0.0:6379->6379/tcp
```

### Step 7: Test It

```bash
# Test the Flask API
curl http://localhost:5000
# {"message": "NexusCorp Flask API", "visits": 1, "redis_host": "redis"}

curl http://localhost:5000
# {"message": "NexusCorp Flask API", "visits": 2, "redis_host": "redis"}

# Test health endpoint
curl http://localhost:5000/health
# {"status": "healthy", "redis": "connected"}
```

### Step 8: Stop the Stack

```bash
# Stop all services (preserves volumes and networks)
docker-compose stop

# Stop AND remove containers and networks (preserves volumes)
docker-compose down

# Stop AND remove everything including volumes
docker-compose down -v

# Stop AND remove including images
docker-compose down --rmi all
```

---

## Part 4: Real-World Task — WordPress Deployment

WordPress is a perfect real-world Compose example — it needs a web server, a PHP runtime, and a MySQL database. All three are defined in a single Compose file.

### The WordPress `docker-compose.yml`

```yaml
# wordpress-compose.yml
version: "3.8"

services:
  wordpress:
    image: wordpress:latest
    container_name: nexus-wordpress
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ${WORDPRESS_DB_PASSWORD}
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db
    restart: unless-stopped
    networks:
      - wordpress-net

  db:
    image: mysql:8.0
    container_name: nexus-mysql
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: ${WORDPRESS_DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql
    restart: unless-stopped
    networks:
      - wordpress-net

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: nexus-phpmyadmin
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: wordpress
      PMA_PASSWORD: ${WORDPRESS_DB_PASSWORD}
    depends_on:
      - db
    networks:
      - wordpress-net

networks:
  wordpress-net:
    driver: bridge

volumes:
  wordpress-data:
  db-data:
```

### The `.env` File

```bash
# .env (do NOT commit to Git)
WORDPRESS_DB_PASSWORD=wordpress-secret-123
MYSQL_ROOT_PASSWORD=root-secret-456
```

### Deploy WordPress

```bash
# Save the compose file as wordpress-compose.yml
# Create the .env file

# Start WordPress
docker-compose -f wordpress-compose.yml up -d

# Check all three services are running
docker-compose -f wordpress-compose.yml ps

# Follow logs
docker-compose -f wordpress-compose.yml logs -f

# Access in browser:
# WordPress:   http://localhost:8080
# phpMyAdmin:  http://localhost:8081

# Tear down (preserves volumes — WordPress data survives)
docker-compose -f wordpress-compose.yml down

# Tear down everything including data
docker-compose -f wordpress-compose.yml down -v
```

**What this demonstrates:**
- `wordpress` uses `db` as its database hostname — Docker Compose DNS resolves service names automatically
- Both `wordpress-data` and `db-data` volumes persist content across container restarts
- Three services start with one command, stop with one command
- `.env` file keeps credentials out of the Compose file

---

## Part 5: Docker Compose Commands Reference

### Essential Commands

```bash
# ── Starting and stopping ──────────────────────────────────────

# Start all services (foreground)
docker-compose up

# Start all services (background/detached)
docker-compose up -d

# Start and rebuild images before starting
docker-compose up -d --build

# Start a specific service only
docker-compose up -d redis

# Stop all services (preserves containers)
docker-compose stop

# Stop a specific service
docker-compose stop flask

# Start stopped services
docker-compose start

# Stop + remove containers and default network (keeps volumes)
docker-compose down

# Stop + remove containers, networks, AND volumes
docker-compose down -v

# Stop + remove containers, networks, volumes, AND images
docker-compose down -v --rmi all


# ── Inspecting state ───────────────────────────────────────────

# List services and their status
docker-compose ps

# Show running processes in each service
docker-compose top

# View logs (all services)
docker-compose logs

# Follow logs in real time
docker-compose logs -f

# Logs for a specific service
docker-compose logs -f flask

# Last 50 lines of logs
docker-compose logs --tail 50

# Show logs with timestamps
docker-compose logs -t


# ── Running commands ───────────────────────────────────────────

# Execute a command in a running service container
docker-compose exec flask bash
docker-compose exec flask python3 -c "import flask; print(flask.__version__)"

# Run a one-off command in a new container (not the running service)
docker-compose run --rm flask python3 manage.py migrate


# ── Building and pulling ───────────────────────────────────────

# Build or rebuild service images
docker-compose build

# Build a specific service
docker-compose build flask

# Pull latest images for all services
docker-compose pull

# Push images to a registry
docker-compose push


# ── Configuration ──────────────────────────────────────────────

# Validate and display the resolved Compose config
docker-compose config

# Show which images are used
docker-compose images

# Show service events
docker-compose events


# ── Scaling ───────────────────────────────────────────────────

# Run multiple instances of a service (use ports range for multiple)
docker-compose up -d --scale flask=3
```

---

## Part 6: Multi-Service Networking in Compose

### Service Name as Hostname

In Docker Compose, services within the same Compose project are automatically connected to a default network. Each service is reachable by its **service name** as a DNS hostname.

```yaml
services:
  api:
    image: myapi
    environment:
      # Use service name 'db' — not an IP address
      DATABASE_URL: postgresql://postgres:secret@db:5432/nexusdb
      # Use service name 'cache' — not an IP address
      REDIS_URL: redis://cache:6379

  db:
    image: postgres:15

  cache:
    image: redis:7-alpine
```

The `api` service connects to `db` using hostname `db` and to `cache` using hostname `cache` — Docker Compose DNS resolves these automatically.

---

### Custom Networks in Compose

```yaml
version: "3.8"

services:
  nginx:
    image: nginx
    networks:
      - frontend

  api:
    image: myapi
    networks:
      - frontend    # Can reach nginx and be reached from nginx
      - backend     # Can reach db

  db:
    image: postgres:15
    networks:
      - backend     # Only reachable from api — not from nginx

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

This recreates the three-tier isolation you built manually in Day 27 — in six lines of YAML.

---

## 🔧 Mini-Project: NexusCorp Full Application Stack

**Scenario:** Build the complete NexusCorp microservice stack — nginx reverse proxy, Flask API, PostgreSQL database, and Redis cache — all defined in a single Compose file with proper networking, volumes, and environment variable configuration.

### Project Structure

```
nexus-stack/
├── docker-compose.yml
├── .env
├── .gitignore
├── nginx/
│   └── nginx.conf
└── api/
    ├── Dockerfile
    ├── requirements.txt
    └── app.py
```

### Application Files

```python
# api/app.py
from flask import Flask, jsonify, request
import redis
import psycopg2
import os
from datetime import datetime

app = Flask(__name__)

def get_db():
    return psycopg2.connect(os.getenv('DATABASE_URL'))

def get_redis():
    return redis.Redis(
        host=os.getenv('REDIS_HOST', 'redis'),
        port=6379,
        decode_responses=True
    )

@app.route('/')
def index():
    r = get_redis()
    visits = r.incr('visit_count')
    return jsonify({
        'service': 'NexusCorp API',
        'version': '1.0.0',
        'total_visits': visits,
        'timestamp': datetime.utcnow().isoformat()
    })

@app.route('/health')
def health():
    status = {'api': 'ok', 'redis': 'unknown', 'postgres': 'unknown'}
    try:
        get_redis().ping()
        status['redis'] = 'ok'
    except Exception as e:
        status['redis'] = str(e)
    try:
        conn = get_db()
        conn.close()
        status['postgres'] = 'ok'
    except Exception as e:
        status['postgres'] = str(e)
    overall = 'healthy' if all(v == 'ok' for v in status.values()) else 'degraded'
    return jsonify({'status': overall, 'services': status})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```dockerfile
# api/Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

```txt
# api/requirements.txt
flask==3.0.0
redis==5.0.1
psycopg2-binary==2.9.9
```

```nginx
# nginx/nginx.conf
upstream api {
    server api:5000;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /health {
        proxy_pass http://api/health;
    }
}
```

### The Docker Compose File

```yaml
# docker-compose.yml
version: "3.8"

services:

  nginx:
    image: nginx:1.25-alpine
    container_name: nexus-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - api
    networks:
      - frontend
    restart: unless-stopped

  api:
    build: ./api
    container_name: nexus-api
    environment:
      - REDIS_HOST=redis
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      - FLASK_ENV=${FLASK_ENV}
    depends_on:
      - db
      - redis
    networks:
      - frontend
      - backend
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    container_name: nexus-db
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: nexus-redis
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  db-data:
  redis-data:
```

### The `.env` File

```bash
# .env — do not commit to Git
POSTGRES_USER=nexus
POSTGRES_PASSWORD=nexus-db-secret
POSTGRES_DB=nexusdb
FLASK_ENV=development
```

### `.gitignore`

```gitignore
.env
*.pyc
__pycache__/
```

### Deploy and Test

```bash
cd nexus-stack

# Start the full stack
docker-compose up -d --build

# Watch startup logs
docker-compose logs -f

# Check all services
docker-compose ps

# Test through nginx (port 80)
curl http://localhost/
curl http://localhost/health

# View individual service logs
docker-compose logs api
docker-compose logs db

# Execute into the API container
docker-compose exec api bash

# Scale the API (nginx upstream will load balance)
docker-compose up -d --scale api=2

# Bring down everything
docker-compose down -v
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Docker Compose** | A tool for defining and running multi-container Docker applications via a YAML file |
| **`docker-compose.yml`** | The configuration file that defines services, networks, and volumes for a Compose application |
| **Service** | A Compose definition of a container — equivalent to a `docker run` command |
| **`docker-compose up`** | Starts all services defined in the Compose file |
| **`docker-compose up -d`** | Starts all services in detached (background) mode |
| **`docker-compose down`** | Stops and removes containers and networks (preserves volumes) |
| **`docker-compose down -v`** | Stops and removes containers, networks, AND volumes |
| **`docker-compose ps`** | Lists the status of all services in the Compose project |
| **`docker-compose logs`** | Shows log output from all services |
| **`docker-compose logs -f`** | Follows (streams) logs in real time |
| **`docker-compose exec`** | Runs a command in a running service container |
| **`docker-compose build`** | Builds or rebuilds service images |
| **`docker-compose pull`** | Pulls the latest images for services that use pre-built images |
| **`docker-compose stop`** | Stops running services without removing containers |
| **`docker-compose start`** | Starts stopped service containers |
| **`docker-compose restart`** | Restarts service containers |
| **`docker-compose config`** | Validates and prints the resolved Compose configuration |
| **`docker-compose scale`** | Runs multiple instances of a service |
| **`depends_on`** | Defines startup order — a service waits for its dependencies |
| **`healthcheck`** | A command Docker runs to determine if a service is healthy |
| **`condition: service_healthy`** | `depends_on` condition that waits for a healthcheck to pass |
| **`.env` file** | A file containing environment variable definitions auto-loaded by Compose |
| **Service name DNS** | In Compose, each service name acts as a DNS hostname for other services |
| **`restart: unless-stopped`** | Restart policy — restarts container on failure unless manually stopped |
| **`restart: always`** | Always restarts the container, including on Docker daemon restart |
| **`build:`** | Compose directive to build an image from a local Dockerfile |
| **`image:`** | Compose directive to use a pre-built image from a registry |
| **Project** | A Compose application — all services, networks, and volumes share a project name prefix |
| **`--scale`** | Flag to run multiple instances of a service |
| **Reverse proxy** | A server (nginx here) that forwards client requests to backend services |
| **Upstream** | In nginx, the backend service(s) that nginx forwards requests to |

---

## 📚 Resources

- [Docker Compose Overview — Official Docs](https://docs.docker.com/compose/) — the complete Compose reference
- [Compose File Reference](https://docs.docker.com/compose/compose-file/) — every YAML directive explained
- [Docker Compose CLI Reference](https://docs.docker.com/compose/reference/) — every command documented
- [Compose Networking — Official Docs](https://docs.docker.com/compose/networking/) — how service name DNS works
- [Environment Variables in Compose](https://docs.docker.com/compose/environment-variables/) — `.env` files and variable substitution
- [Docker Compose Samples](https://github.com/docker/awesome-compose) — community example Compose files for dozens of tech stacks

---

## 🔭 Day 30 Preview: Introduction to CI/CD with Jenkins

Docker Compose closes the chapter on containerization fundamentals. Day 30 opens the next chapter: **automation**.

CI/CD (Continuous Integration / Continuous Delivery) is the practice of automatically building, testing, and deploying code every time a developer pushes a change. Jenkins is one of the most widely used CI/CD platforms — and it runs in Docker.

You will learn:
- What CI/CD is and why it matters
- Jenkins architecture — master, agents, pipelines
- Running Jenkins in Docker
- Creating your first Jenkins pipeline
- Connecting Git to Jenkins — trigger a build on every push
- How NexusCorp's CI/CD pipeline takes code from `git push` to deployed container automatically

Everything from Days 1–29 — Linux, Git, Python, networking, Docker — is an ingredient in the CI/CD pipeline you will build next.

---

> 💬 *"A `docker-compose.yml` file is a contract. It says: given these images and this configuration, the application will always start the same way. That contract is the foundation of reproducible infrastructure."*
>
> — DevOps Learning By Yukta
