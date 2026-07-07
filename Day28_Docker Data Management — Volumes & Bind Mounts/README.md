# Day 28: Docker Data Management — Volumes & Bind Mounts

> **DevOps Transformation** | Containers & Automation Series

---

## 📋 Overview

Yesterday you controlled how containers communicate. Today you tackle a problem that breaks every stateful container that isn't properly configured: **data persistence**.

By default, everything inside a container is temporary. The moment you remove a container, all data it created or modified disappears. For a stateless web server that serves static files from an image, this is fine. For a database, a file upload service, or any application that stores user data — it is catastrophic.

Docker provides two mechanisms to solve this: **Volumes** (Docker-managed, preferred for production) and **Bind Mounts** (host-directory-mapped, preferred for development). By the end of today you will have used both, understood the difference, and proven to yourself — with a MySQL database — exactly why volumes are non-negotiable for stateful applications.

---

## 🗺️ What You'll Do Today

| Topic | Commands | What You Learn |
|---|---|---|
| Docker storage mechanics | — | Why container storage is ephemeral by default |
| Named volumes | `docker volume create`, `docker volume inspect` | Docker-managed persistent storage |
| Attach volumes to containers | `docker run -v` | Mount volumes at specific container paths |
| Bind mounts | `docker run -v /host:/container` | Map host directories into containers |
| Database persistence demo | MySQL with and without volumes | Why volumes matter for stateful apps |
| Volume lifecycle | `docker volume ls`, `docker volume rm`, `docker volume prune` | Manage volume lifecycle |

---

## 🏢 NexusCorp Scenario

> NexusCorp's team just lost two weeks of test data. A developer ran `docker rm` on a MySQL container without realizing the database was not backed by a volume. The data existed only in the container's writable layer — and it vanished instantly.
>
> The team now has a rule: every stateful container must declare a named volume. Today you learn why that rule exists — and how to follow it.

---

## Part 1: Understanding Docker's Storage Mechanics

### Why Container Storage Is Ephemeral

When you run a container, Docker stacks three layers:

```
┌────────────────────────────────────────────┐
│         Container (running)                │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │  Writable Layer (thin R/W layer)     │  │
│  │  → All changes made inside the       │  │
│  │    container are stored here         │  │
│  │  → DELETED when container is removed │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │  Image Layers (read-only)            │  │
│  │  Layer 3: App files                  │  │
│  │  Layer 2: Dependencies installed     │  │
│  │  Layer 1: Base OS                    │  │
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘
```

The writable layer is temporary and tied to the container's lifecycle. When `docker rm` is run, the writable layer is discarded. The read-only image layers remain — they are shared across all containers created from the same image.

**The consequence:**

```bash
# Create a container, write data inside it
docker run -it ubuntu bash
echo "critical data" > /tmp/important.txt
cat /tmp/important.txt    # It's there
exit

# Remove the container
docker rm $(docker ps -aq -f status=exited)

# The data is gone — permanently
```

This behavior is intentional. Containers are designed to be **ephemeral** — disposable, replaceable, stateless. The data storage problem is solved outside the container, not inside it.

---

### The Three Ways Docker Stores Data

```
┌─────────────────────────────────────────────────────────┐
│                  Host Machine                           │
│                                                         │
│  /var/lib/docker/volumes/    ← Named Volumes (Docker)   │
│                                                         │
│  /any/host/path/             ← Bind Mounts (you choose) │
│                                                         │
│  tmpfs (RAM)                 ← tmpfs mounts (temp only) │
└─────────────────────────────────────────────────────────┘
         │                │                │
         ▼                ▼                ▼
    Container         Container        Container
    /data             /config          /tmp/cache
```

| Storage Type | Managed By | Location | Lifecycle | Best For |
|---|---|---|---|---|
| **Named Volume** | Docker | `/var/lib/docker/volumes/` | Persists until explicitly removed | Production databases, persistent app data |
| **Bind Mount** | You | Any host path you choose | As long as host path exists | Development, config injection, live code reload |
| **tmpfs Mount** | Docker (RAM) | Host RAM only | Container lifetime | Secrets, sensitive temp data, performance |

---

## Part 2: Docker Volumes

### What Is a Named Volume?

A named volume is a Docker-managed storage area that exists independently of any container. Docker creates and manages the storage location (inside `/var/lib/docker/volumes/` on Linux). You reference it by name.

**Key properties:**
- Persists after the container that created it is removed
- Can be mounted into multiple containers simultaneously
- Easily backed up, migrated, or restored
- Works on Linux, macOS, and Windows (Docker Desktop manages the path)
- Content pre-populated from the image if the mount point existed in the image

---

### Volume Commands

```bash
# ── Create a volume ────────────────────────────────────────────
docker volume create myvolume

# Create with a specific driver (default is local)
docker volume create --driver local nexus-db-data

# Create with labels
docker volume create \
    --label project=nexuscorp \
    --label env=staging \
    nexus-staging-data


# ── List volumes ───────────────────────────────────────────────
docker volume ls

# Output:
# DRIVER    VOLUME NAME
# local     myvolume
# local     nexus-db-data
# local     nexus-staging-data

# Filter by label
docker volume ls --filter label=project=nexuscorp

# List only volume names
docker volume ls -q


# ── Inspect a volume ───────────────────────────────────────────
docker volume inspect myvolume

# Output:
# [
#     {
#         "CreatedAt": "2024-06-10T14:00:00Z",
#         "Driver": "local",
#         "Labels": {},
#         "Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
#         "Name": "myvolume",
#         "Options": {},
#         "Scope": "local"
#     }
# ]

# The Mountpoint is where Docker stores the data on the host


# ── Remove volumes ─────────────────────────────────────────────
# Remove a specific volume (fails if a container uses it)
docker volume rm myvolume

# Force remove (if no containers are actively using it)
docker volume rm --force myvolume

# Remove all unused volumes (not referenced by any container)
docker volume prune

# Remove all volumes including named ones (destructive!)
docker volume prune --all
```

---

### Attaching Volumes to Containers

Use the `-v` flag or `--mount` flag to attach a volume when running a container.

```bash
# ── -v syntax (shorter, widely used) ──────────────────────────
docker run -d \
    --name mycontainer \
    -v myvolume:/app \
    nginx

# Format: -v VOLUME_NAME:CONTAINER_PATH

# ── --mount syntax (more explicit, recommended for clarity) ────
docker run -d \
    --name mycontainer \
    --mount type=volume,source=myvolume,target=/app \
    nginx

# ── If the volume doesn't exist — Docker creates it automatically
docker run -d \
    --name auto-vol-container \
    -v auto-created-volume:/data \
    ubuntu sleep infinity
# Docker creates 'auto-created-volume' if it doesn't exist

# ── Read-only volume (container cannot write to it) ────────────
docker run -d \
    --name readonly-container \
    -v myvolume:/app:ro \
    nginx
```

---

### Volume Persistence — Demonstrated

```bash
# Step 1: Create a volume and run a container
docker volume create nexus-persist
docker run -it --name writer \
    -v nexus-persist:/data \
    ubuntu bash

# Inside the container — write data
echo "NexusCorp persistent data — $(date)" > /data/important.txt
echo "Server config: production" >> /data/important.txt
cat /data/important.txt
exit

# Step 2: Remove the container
docker rm writer
docker ps -a  # Container is gone

# Step 3: The volume still exists
docker volume ls  # nexus-persist is still there

# Step 4: Create a NEW container with the same volume
docker run -it --name reader \
    -v nexus-persist:/data \
    ubuntu bash

# Inside the new container — data is still there!
cat /data/important.txt
# Output: NexusCorp persistent data — (date)
#         Server config: production
exit

# Cleanup
docker rm reader
docker volume rm nexus-persist
```

This is the fundamental value of volumes — data outlives any individual container.

---

### Inspect Volume Data on the Host

```bash
# On Linux — view volume data directly on the host
sudo ls /var/lib/docker/volumes/myvolume/_data/
sudo cat /var/lib/docker/volumes/myvolume/_data/important.txt

# On Docker Desktop (macOS/Windows) — data is inside a VM
# Access via docker run:
docker run --rm \
    -v myvolume:/data \
    alpine cat /data/important.txt
```

---

## Part 3: Bind Mounts

### What Is a Bind Mount?

A bind mount maps a **specific path on the host machine** directly into the container. Changes on the host are immediately visible inside the container, and vice versa. The host path must exist before the container starts.

```
Host Machine                    Container
/home/user/myapp/        ──────► /app/
  ├── app.py                       ├── app.py
  ├── config.yml                   ├── config.yml
  └── templates/                   └── templates/

Edit app.py on host → instantly visible inside container
```

---

### Using Bind Mounts

```bash
# ── -v syntax for bind mount ───────────────────────────────────
# Format: -v /absolute/host/path:/container/path

# Mount current directory into container
docker run -it \
    --name bindtest \
    -v $(pwd):/workspace \
    ubuntu bash

# Mount a specific host directory
docker run -d \
    --name mybindcontainer \
    -v /home/user/myapp:/app \
    nginx

# Mount with read-only access
docker run -d \
    -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
    nginx

# ── --mount syntax for bind mount ─────────────────────────────
docker run -d \
    --name bindcontainer \
    --mount type=bind,source=/home/user/myapp,target=/app \
    nginx

# Read-only with --mount
docker run -d \
    --mount type=bind,source=/host/config,target=/app/config,readonly \
    myapp
```

---

### Bind Mount — Development Workflow

The most common use of bind mounts in development is **live code reloading** — edit files on the host, see changes immediately in the container:

```bash
# Create a project directory on the host
mkdir -p /tmp/nexus-dev
cat > /tmp/nexus-dev/index.html << 'EOF'
<!DOCTYPE html>
<html>
<body>
<h1>NexusCorp Dev Server</h1>
<p>Version 1.0 — edit me!</p>
</body>
</html>
EOF

# Run nginx with the host directory mounted
docker run -d \
    --name nexus-dev \
    -p 8080:80 \
    -v /tmp/nexus-dev:/usr/share/nginx/html \
    nginx

# Verify it works
curl http://localhost:8080
# Output: Version 1.0 — edit me!

# Edit the file on the HOST
cat > /tmp/nexus-dev/index.html << 'EOF'
<!DOCTYPE html>
<html>
<body>
<h1>NexusCorp Dev Server</h1>
<p>Version 2.0 — updated without restarting!</p>
</body>
</html>
EOF

# Immediately visible inside the container — no restart needed
curl http://localhost:8080
# Output: Version 2.0 — updated without restarting!

# Cleanup
docker stop nexus-dev && docker rm nexus-dev
```

---

### Volumes vs Bind Mounts — When to Use Which

| | Named Volume | Bind Mount |
|---|---|---|
| **Managed by** | Docker | You (host path) |
| **Location** | Docker's internal storage | Any path you specify |
| **Host path required?** | No — Docker manages it | Yes — path must exist |
| **Portability** | High — works across hosts | Lower — depends on host path |
| **Performance** | Excellent (Linux) | Excellent (Linux), slower on Mac/Windows |
| **Development use** | Less common | ✅ Live code reload, config injection |
| **Production use** | ✅ Databases, app data | Config files, secrets |
| **Pre-populate from image?** | ✅ Yes | ❌ No — host path overwrites |
| **Backup/migrate** | `docker volume` commands | `cp` or `rsync` |

---

## Part 4: The Database Persistence Demo

This exercise proves — with real data — why volumes are essential for stateful containers.

### Demo 1: MySQL Without a Volume (Data Loss)

```bash
# Step 1: Run MySQL without a volume
docker run -d \
    --name mysqltest \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    mysql:latest

# Wait for MySQL to initialize
echo "Waiting for MySQL to start..."
sleep 20

# Step 2: Connect and create data
docker exec -it mysqltest mysql -uroot -pmy-secret-pw

# Inside MySQL:
CREATE DATABASE nexuscorp;
USE nexuscorp;
CREATE TABLE servers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    environment VARCHAR(20),
    status VARCHAR(20)
);
INSERT INTO servers (name, environment, status)
VALUES
    ('web-01', 'production', 'running'),
    ('db-01', 'production', 'running'),
    ('cache-01', 'staging', 'running');
SELECT * FROM servers;
exit

# Step 3: Verify the data is there
docker exec mysqltest mysql -uroot -pmy-secret-pw \
    -e "SELECT * FROM nexuscorp.servers;"

# Step 4: Remove the container
docker stop mysqltest
docker rm mysqltest

# Step 5: Recreate — data is GONE
docker run -d \
    --name mysqltest \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    mysql:latest

sleep 20

docker exec mysqltest mysql -uroot -pmy-secret-pw \
    -e "SHOW DATABASES;"
# nexuscorp database is NOT there — all data lost

# Cleanup
docker stop mysqltest && docker rm mysqltest
```

---

### Demo 2: MySQL With a Volume (Data Persists)

```bash
# Step 1: Run MySQL WITH a named volume
docker run -d \
    --name mysqlvol \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    -v mysql_data:/var/lib/mysql \
    mysql:latest

echo "Waiting for MySQL to start..."
sleep 20

# Step 2: Create the same data
docker exec -it mysqlvol mysql -uroot -pmy-secret-pw

# Inside MySQL:
CREATE DATABASE nexuscorp;
USE nexuscorp;
CREATE TABLE servers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    environment VARCHAR(20),
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO servers (name, environment, status)
VALUES
    ('web-01', 'production', 'running'),
    ('db-01', 'production', 'running'),
    ('cache-01', 'staging', 'running'),
    ('dev-01', 'development', 'running');
SELECT * FROM servers;
exit

# Step 3: Check the volume exists
docker volume ls | grep mysql_data
docker volume inspect mysql_data

# Step 4: Remove the container (NOT the volume)
docker stop mysqlvol
docker rm mysqlvol
# Volume 'mysql_data' still exists

# Step 5: Recreate with the SAME volume
docker run -d \
    --name mysqlvol \
    -e MYSQL_ROOT_PASSWORD=my-secret-pw \
    -v mysql_data:/var/lib/mysql \
    mysql:latest

sleep 20

# Step 6: Data is still there!
docker exec mysqlvol mysql -uroot -pmy-secret-pw \
    -e "SELECT * FROM nexuscorp.servers;"
# All 4 rows are present — data survived container removal

echo "SUCCESS: Data persisted across container lifecycle"

# Cleanup
docker stop mysqlvol
docker rm mysqlvol
docker volume rm mysql_data
```

---

## Part 5: Advanced Volume Techniques

### Sharing a Volume Between Multiple Containers

```bash
# Create a shared volume
docker volume create shared-data

# Container 1: Write data
docker run -d \
    --name writer \
    -v shared-data:/data \
    ubuntu \
    sh -c 'while true; do
        echo "$(date): heartbeat from writer" >> /data/log.txt
        sleep 2
    done'

# Container 2: Read data written by Container 1
docker run -it \
    --name reader \
    -v shared-data:/data \
    ubuntu \
    tail -f /data/log.txt
# You will see real-time output from the writer container

# Ctrl+C to stop tail
docker stop writer reader
docker rm writer reader
docker volume rm shared-data
```

---

### Running a Container with Both a Volume and a Bind Mount

```bash
# Create host config directory
mkdir -p /tmp/nexus-config
echo "LOG_LEVEL=INFO" > /tmp/nexus-config/app.env
echo "MAX_CONNECTIONS=100" >> /tmp/nexus-config/app.env

# Create a volume for app data
docker volume create nexus-app-data

# Run with BOTH — different paths inside the container
docker run -d \
    --name nexus-dual \
    -v nexus-app-data:/app/data \         # Volume — persists app data
    -v /tmp/nexus-config:/app/config:ro \ # Bind mount — injects config (read-only)
    ubuntu \
    sleep infinity

# Verify both are mounted
docker exec nexus-dual ls /app/
# data/   config/

docker exec nexus-dual cat /app/config/app.env
# LOG_LEVEL=INFO
# MAX_CONNECTIONS=100

docker exec nexus-dual sh -c \
    'echo "App data entry: $(date)" > /app/data/record.txt'
docker exec nexus-dual cat /app/data/record.txt
# Data written to volume

# Cleanup
docker stop nexus-dual
docker rm nexus-dual
docker volume rm nexus-app-data
rm -rf /tmp/nexus-config
```

---

### Backing Up and Restoring Volumes

```bash
# ── Backup a volume ────────────────────────────────────────────
# Technique: mount volume + mount backup dir, then tar the volume contents
docker run --rm \
    -v mysql_data:/data \
    -v $(pwd):/backup \
    ubuntu \
    tar czf /backup/mysql_data_backup.tar.gz -C /data .

# Verify backup was created
ls -lh mysql_data_backup.tar.gz


# ── Restore a volume ───────────────────────────────────────────
# Create a new volume for the restore
docker volume create mysql_data_restored

# Extract the backup into the new volume
docker run --rm \
    -v mysql_data_restored:/data \
    -v $(pwd):/backup \
    ubuntu \
    tar xzf /backup/mysql_data_backup.tar.gz -C /data


# ── Copy files directly to/from a volume ──────────────────────
# To volume: copy a file from host into a volume
docker run --rm \
    -v myvolume:/data \
    -v $(pwd):/host \
    alpine cp /host/config.yml /data/config.yml

# From volume: copy a file from volume to host
docker run --rm \
    -v myvolume:/data \
    -v $(pwd):/host \
    alpine cp /data/config.yml /host/config_backup.yml
```

---

## 🔧 Mini-Project: NexusCorp Stateful Application Stack

**Scenario:** Build a complete data management setup — a PostgreSQL database with a persistent volume, an application container that writes to both a volume and uses a bind-mounted config, and demonstrate full data survival across container replacements.

```bash
#!/bin/bash
# nexus_data_stack.sh
# NexusCorp stateful application data management demo

echo "=============================================="
echo "  NexusCorp Data Management Demo"
echo "=============================================="

# ── Setup ───────────────────────────────────────────────────────
VOLUME_NAME="nexus-postgres-data"
CONFIG_DIR="/tmp/nexus-db-config"
APP_VOLUME="nexus-app-logs"

mkdir -p "$CONFIG_DIR"

cat > "$CONFIG_DIR/pg.conf" << 'EOF'
# NexusCorp PostgreSQL configuration
max_connections = 100
shared_buffers = 128MB
log_statement = 'all'
EOF

echo ""
echo "Step 1: Creating volumes and config..."
docker volume create "$VOLUME_NAME"
docker volume create "$APP_VOLUME"
echo "  ✓ Volume: $VOLUME_NAME"
echo "  ✓ Volume: $APP_VOLUME"

# ── Start PostgreSQL with persistent volume ──────────────────────
echo ""
echo "Step 2: Starting PostgreSQL with persistent volume..."
docker run -d \
    --name nexus-postgres \
    -e POSTGRES_PASSWORD=nexuspass \
    -e POSTGRES_DB=nexusdb \
    -v "$VOLUME_NAME":/var/lib/postgresql/data \
    postgres:15-alpine

sleep 15
echo "  ✓ PostgreSQL started"

# ── Create data ─────────────────────────────────────────────────
echo ""
echo "Step 3: Creating database schema and inserting data..."
docker exec nexus-postgres psql -U postgres -d nexusdb << 'EOSQL'
CREATE TABLE IF NOT EXISTS deployments (
    id SERIAL PRIMARY KEY,
    app_name VARCHAR(50) NOT NULL,
    version VARCHAR(20) NOT NULL,
    environment VARCHAR(20) NOT NULL,
    deployed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'success'
);
INSERT INTO deployments (app_name, version, environment) VALUES
    ('nexus-api', 'v2.1.0', 'staging'),
    ('nexus-web', 'v1.8.3', 'staging'),
    ('nexus-worker', 'v1.2.0', 'staging');
SELECT * FROM deployments;
EOSQL
echo "  ✓ Data inserted"

# ── Simulate container replacement ──────────────────────────────
echo ""
echo "Step 4: Removing and recreating the database container..."
docker stop nexus-postgres
docker rm nexus-postgres
echo "  ✓ Container removed (volume preserved)"

docker run -d \
    --name nexus-postgres \
    -e POSTGRES_PASSWORD=nexuspass \
    -e POSTGRES_DB=nexusdb \
    -v "$VOLUME_NAME":/var/lib/postgresql/data \
    postgres:15-alpine

sleep 15
echo "  ✓ Container recreated with same volume"

# ── Verify data survived ─────────────────────────────────────────
echo ""
echo "Step 5: Verifying data persistence after container replacement..."
ROWS=$(docker exec nexus-postgres psql -U postgres -d nexusdb \
    -t -c "SELECT COUNT(*) FROM deployments;" | tr -d ' ')

if [ "$ROWS" -eq 3 ]; then
    echo "  ✓ PASS: All $ROWS rows present — data survived container removal"
else
    echo "  ✗ FAIL: Expected 3 rows, found $ROWS"
fi

docker exec nexus-postgres psql -U postgres -d nexusdb \
    -c "SELECT app_name, version, environment, status FROM deployments;"

# ── Volume inspection ────────────────────────────────────────────
echo ""
echo "Step 6: Volume inspection..."
docker volume inspect "$VOLUME_NAME" | \
    python3 -m json.tool 2>/dev/null | \
    grep -E '"Name"|"Mountpoint"|"CreatedAt"'

# ── Cleanup ──────────────────────────────────────────────────────
echo ""
echo "Step 7: Cleaning up..."
docker stop nexus-postgres
docker rm nexus-postgres
docker volume rm "$VOLUME_NAME" "$APP_VOLUME"
rm -rf "$CONFIG_DIR"
echo "  ✓ All containers, volumes, and config removed"

echo ""
echo "=============================================="
echo "  Demo complete."
echo "  Key result: Data persisted across full"
echo "  container stop → remove → recreate cycle"
echo "=============================================="
```

```bash
chmod +x nexus_data_stack.sh
./nexus_data_stack.sh
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Ephemeral storage** | Storage that exists only for the lifetime of a container — deleted when the container is removed |
| **Writable layer** | A thin read-write layer added on top of read-only image layers when a container runs |
| **Named volume** | A Docker-managed persistent storage unit, referenced by name and stored in Docker's internal directory |
| **Bind mount** | A mapping of a specific host filesystem path into a container |
| **tmpfs mount** | A mount that stores data in host RAM only — not persisted to disk |
| **`docker volume create`** | Creates a new named volume |
| **`docker volume ls`** | Lists all volumes on the Docker host |
| **`docker volume inspect`** | Shows detailed metadata about a volume including its mountpoint |
| **`docker volume rm`** | Removes a named volume (fails if a container is using it) |
| **`docker volume prune`** | Removes all volumes not referenced by any container |
| **`-v VOLUME:PATH`** | Short flag syntax for mounting a volume or bind mount |
| **`--mount`** | Explicit, verbose syntax for volume and bind mount configuration |
| **`:ro`** | Read-only modifier for volume or bind mount — container cannot write |
| **`/var/lib/docker/volumes/`** | Default location where Docker stores named volume data on Linux |
| **Mountpoint** | The path on the host where a named volume's data is stored |
| **Volume driver** | The plugin that manages a volume's storage backend (default: `local`) |
| **Data persistence** | The ability for data to survive container removal and restarts |
| **Stateful container** | A container that stores data that must persist (e.g., database, file storage) |
| **Stateless container** | A container that stores no persistent data — can be replaced freely |
| **Volume sharing** | Mounting the same volume into multiple containers simultaneously |
| **Live code reload** | Development pattern where bind-mounted source code reflects changes instantly inside the container |
| **`docker run -v $(pwd):/app`** | Mounts the current host directory into `/app` inside the container |
| **Pre-population** | When a volume is first mounted into a container, Docker copies the image's existing content at that path into the volume |
| **Volume backup** | Archiving a volume's data using `tar` via a temporary container |
| **`:rw`** | Read-write modifier (default) — container can read and write |

---

## 📚 Resources

- [Docker Storage Overview — Official Docs](https://docs.docker.com/storage/) — complete storage reference
- [Volumes Documentation — Docker Docs](https://docs.docker.com/storage/volumes/) — named volumes deep dive
- [Bind Mounts Documentation — Docker Docs](https://docs.docker.com/storage/bind-mounts/) — bind mount reference
- [tmpfs Mounts — Docker Docs](https://docs.docker.com/storage/tmpfs/) — in-memory storage
- [Docker and Databases Best Practices](https://docs.docker.com/samples/databases/) — official guidance on database containers

---

## Key Points Summary

```
1. Volume Manipulation
   → Create, inspect, and delete volumes
   → Attach to different containers
   → Observe data surviving container removal

2. Bind Mount Practice
   → Edit files on the host
   → Changes reflect immediately inside the container
   → Ideal for development workflows

3. Multiple Mount Points
   → Run a container with both a volume and a bind mount
   → Volume at /app/data (persisted, Docker-managed)
   → Bind mount at /app/config (host-controlled, read-only)
   → Each serves a distinct purpose
```

---

## 🔭 Day 29 Preview: Docker Compose — Multi-Container Applications

Today you mastered how individual containers store data. Tomorrow you orchestrate **multiple containers together** as a complete application stack using Docker Compose.

You will learn:
- What Docker Compose is and why it exists
- The `docker-compose.yml` file structure
- Defining services, networks, and volumes in one file
- `docker-compose up`, `docker-compose down`, `docker-compose logs`
- Building a complete NexusCorp application: nginx + Flask API + PostgreSQL + Redis
- All with networking and volume persistence configured in a single file

Everything you learned across Days 24–28 — images, containers, networking, volumes — comes together in Docker Compose. It is the bridge between "running containers manually" and "defining infrastructure as code."

---

> 💬 *"A container without a volume is a notebook with disappearing ink. Everything you write is temporary unless you persist it somewhere that outlives the page."*
>
> — DevOps Learning By Yukta