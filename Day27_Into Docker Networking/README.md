# Day 27: Into Docker Networking

> **DevOps Transformation** | Containers & Automation Series

---

## 📋 Overview

Yesterday you mastered Docker images and containers — pulling, running, interacting, and cleaning up. Today you tackle the layer that makes multi-container applications possible: **Docker networking**.

Containers are isolated by design, but real applications are never truly standalone. A web server needs to talk to a database. A database needs to talk to a cache. An API needs to reach external services. Docker networking is the system that controls exactly which containers can communicate with what — and how.

By the end of today you will have inspected Docker's default network, created custom networks, connected containers to them, tested container-to-container communication by name, and observed the isolation that different network modes provide.

---

## 🗺️ What You'll Do Today

| Topic | Commands | What You Learn |
|---|---|---|
| Network modes overview | — | bridge, host, overlay, macvlan, ipvlan, none |
| Inspect networks | `docker network inspect` | Understand the default bridge network |
| List networks | `docker network ls` | See all networks on your Docker host |
| Create custom networks | `docker network create` | Build isolated network segments |
| Connect containers | `docker network connect` | Add containers to networks |
| Container-to-container comms | `ping`, `curl` by container name | Test communication between containers |
| Network isolation | Different networks | Observe containers that cannot reach each other |

---

## 🏢 NexusCorp Scenario

> NexusCorp's new microservice architecture has three components: a frontend web server (nginx), an API backend (Python Flask), and a database (PostgreSQL). The frontend needs to talk to the API. The API needs to talk to the database. But the database must not be reachable from the frontend directly, and none of them should be reachable from outside the host except through the frontend's port 80.
>
> This is a classic network segmentation problem — and Docker networking is the tool to solve it. Today you learn how.

---

## Part 1: Docker Networking Basics

### How Docker Networking Works Under the Hood

When Docker is installed, it creates several virtual network interfaces on the host machine. These work together to route traffic between containers, and between containers and the outside world.

```bash
# On the host machine, you can see Docker's virtual interfaces
ip addr show
# You'll see: docker0 (the default bridge), and veth pairs for each container
```

**The key concepts:**

```
Host Machine
├── Physical/Virtual NIC (eth0) — connects to internet
├── docker0 (virtual bridge) — Docker's default bridge network
│   ├── veth0a1b2c ← virtual cable connecting Container 1 to bridge
│   ├── veth0d2e3f ← virtual cable connecting Container 2 to bridge
│   └── veth0g4h5i ← virtual cable connecting Container 3 to bridge
│
Containers
├── Container 1: eth0 (172.17.0.2) — inside container, connected to docker0
├── Container 2: eth0 (172.17.0.3)
└── Container 3: eth0 (172.17.0.4)
```

Docker manipulates **iptables rules** to route traffic between containers, between containers and the host, and between containers and the internet. Each container gets its own **network namespace** — an isolated view of the network stack.

---

### The Five Docker Network Drivers

Docker supports multiple network drivers, each designed for a specific use case:

---

#### 1. `bridge` — Default Network (Most Common)

The bridge driver creates a virtual network bridge between the host and containers. It is the default when no network is specified.

```
Internet
    │
  eth0 (host)
    │
 docker0 (bridge — 172.17.0.1)
    ├── Container A (172.17.0.2)
    ├── Container B (172.17.0.3)
    └── Container C (172.17.0.4)
```

**Characteristics:**
- Containers get their own IP addresses on the bridge subnet
- Containers can communicate with each other via IP
- Containers can reach the internet via NAT
- Containers are isolated from the host's network
- Port mapping (`-p`) is required to expose container ports to the host

**Two types of bridge networks:**
- **Default bridge** (`docker0`) — containers communicate by IP only, no DNS
- **User-defined bridge** — containers communicate by container name (DNS enabled)

**When to use:** Most development scenarios and single-host production deployments.

---

#### 2. `host` — Share Host Network Stack

The container uses the host machine's network stack directly. No network isolation, no separate IP.

```
Host Network
    ├── eth0 (192.168.1.45)
    └── Container (shares eth0 directly — no separate IP)
```

**Characteristics:**
- Container uses the host's IP address and network interfaces directly
- Port mapping (`-p`) is not needed — ports are published directly on the host
- No network isolation between container and host
- Better performance (no NAT overhead)

**When to use:** When a container needs maximum network performance, or when running network monitoring tools that need direct host network access. Not recommended for most application containers.

```bash
# Run with host networking
docker run --network host nginx
# nginx now listens on host's port 80 directly — no -p needed
```

---

#### 3. `overlay` — Multi-Host Container Networking

The overlay driver creates a distributed network that spans multiple Docker hosts (nodes). Used with Docker Swarm or Kubernetes.

```
Host 1                          Host 2
├── Container A ─────────────── Container C
└── Container B     (overlay    └── Container D
                    network)
```

**Characteristics:**
- Enables containers on different physical/virtual hosts to communicate as if on the same network
- Requires Docker Swarm mode or an external key-value store
- Encrypts traffic between hosts (optional)

**When to use:** Multi-host Docker deployments, Docker Swarm clusters, microservices spread across multiple servers.

---

#### 4. `macvlan` — Direct Physical Network Access

Assigns each container a unique MAC address, making it appear as a physical device on the host network. The container gets an IP from the physical network's subnet.

```
Physical Network (192.168.1.0/24)
├── Host (192.168.1.45)
├── Container A (192.168.1.100) ← appears as physical device
└── Container B (192.168.1.101) ← appears as physical device
```

**Characteristics:**
- Container appears as a real network device on the physical LAN
- No NAT — direct network access
- Best performance for network-intensive workloads
- Requires the host's NIC to be in promiscuous mode

**When to use:** Legacy applications that need to be on the physical network, network monitoring, IoT deployments.

---

#### 5. `ipvlan` — Precise IP and VLAN Control

Similar to macvlan but shares the host's MAC address. Provides more control over IP addressing and VLAN tagging.

**Characteristics:**
- Containers share the host's MAC address (avoids promiscuous mode requirement)
- Supports L2 mode (Layer 2, like macvlan) and L3 mode (Layer 3 routing)
- Precise VLAN tagging control

**When to use:** Environments with strict MAC address policies, VLAN-segmented networks.

---

#### 6. `none` — No Networking

Completely disables networking for the container. Only the loopback interface is available.

```bash
docker run --network none ubuntu
# Inside: only lo (127.0.0.1) — no eth0, no internet
```

**When to use:** Maximum isolation, batch processing jobs that need no network access, security-sensitive workloads.

---

### Network Driver Comparison

| Driver | Isolation | Multi-Host | Performance | Use Case |
|---|---|---|---|---|
| `bridge` (default) | Medium | No | Good | Dev, single-host apps |
| User-defined bridge | Medium | No | Good | Multi-container apps on one host |
| `host` | None | No | Best | Performance-critical, monitoring |
| `overlay` | Medium | Yes | Good | Docker Swarm, multi-host |
| `macvlan` | Low | No | Excellent | Physical network integration |
| `ipvlan` | Low | No | Excellent | VLAN environments |
| `none` | Maximum | No | N/A | Fully isolated workloads |

---

## Part 2: Docker Network Commands

### Core Network Commands

```bash
# List all networks on this Docker host
docker network ls

# Expected output on a fresh install:
# NETWORK ID     NAME      DRIVER    SCOPE
# a1b2c3d4e5f6   bridge    bridge    local
# b2c3d4e5f6a1   host      host      local
# c3d4e5f6a1b2   none      null      local
```

**The three default networks:**

| Network | Driver | Description |
|---|---|---|
| `bridge` | bridge | Default network — containers connect here unless specified |
| `host` | host | Shares the host network stack |
| `none` | null | No networking — loopback only |

---

### Inspecting Networks

```bash
# Inspect the default bridge network
docker network inspect bridge

# Key information in the output:
# - "Subnet": the IP range assigned to this network (e.g., 172.17.0.0/16)
# - "Gateway": the bridge IP (e.g., 172.17.0.1)
# - "Containers": which containers are currently connected

# Pretty-print only the containers on the network
docker network inspect bridge \
    --format '{{json .Containers}}' | python3 -m json.tool

# Inspect a custom network
docker network inspect my-network
```

---

### Creating Custom Networks

```bash
# Create a user-defined bridge network (recommended over default bridge)
docker network create my-network

# Create with a specific driver
docker network create --driver bridge my-bridge
docker network create --driver overlay my-overlay   # Requires Swarm

# Create with a specific subnet and gateway
docker network create \
    --driver bridge \
    --subnet 10.10.0.0/24 \
    --gateway 10.10.0.1 \
    nexus-network

# Create with a custom IP range within the subnet
docker network create \
    --subnet 192.168.100.0/24 \
    --ip-range 192.168.100.128/25 \
    --gateway 192.168.100.1 \
    nexus-controlled-network

# List networks to confirm creation
docker network ls
```

---

### Connecting and Disconnecting Containers

```bash
# Connect a running container to a network
docker network connect my-network my-container

# Connect with a specific IP address
docker network connect --ip 10.10.0.50 my-network my-container

# Disconnect a container from a network
docker network disconnect my-network my-container

# Run a container on a specific network at startup
docker run --network my-network --name app1 nginx

# Run a container on multiple networks
docker run --network network1 --name app1 nginx
docker network connect network2 app1    # Also add to network2

# Remove a network (only works if no containers are connected)
docker network rm my-network

# Remove all unused networks
docker network prune
```

---

### Finding a Container's IP Address

```bash
# Method 1: docker inspect
docker inspect container-name | grep IPAddress

# Method 2: docker inspect with format
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container-name

# Method 3: exec into the container
docker exec container-name hostname -I

# Method 4: ip addr inside the container
docker exec -it container-name ip addr show eth0

# See all network details for a container
docker inspect container-name --format '{{json .NetworkSettings.Networks}}' | \
    python3 -m json.tool
```

---

## Part 3: Key Networking Concepts

### Default Bridge vs User-Defined Bridge

This is one of the most important distinctions in Docker networking:

| Feature | Default Bridge (`docker0`) | User-Defined Bridge |
|---|---|---|
| **DNS resolution** | ❌ No — containers must use IPs | ✅ Yes — use container names |
| **Automatic connection** | All containers join by default | Must specify `--network` |
| **Isolation** | All containers share one network | Can create separate segments |
| **Recommended for** | Quick testing only | All real applications |

**Why user-defined bridges are better:**

```bash
# Default bridge — containers can only reach each other by IP
docker run -d --name app1 nginx
docker run -d --name app2 nginx

docker exec app2 ping 172.17.0.2    # Works (by IP)
docker exec app2 ping app1          # FAILS — no DNS on default bridge


# User-defined bridge — containers reach each other by name
docker network create nexus-net
docker run -d --name app1 --network nexus-net nginx
docker run -d --name app2 --network nexus-net nginx

docker exec app2 ping app1          # WORKS — DNS resolution by name
docker exec app2 curl http://app1   # WORKS — HTTP by name
```

---

### Container Name DNS Resolution

On user-defined bridge networks, Docker provides an embedded DNS server that resolves container names to their IP addresses. This is how microservices find each other in real deployments:

```
Container: api-server
  Needs to connect to: database

Instead of hardcoding: postgresql://172.17.0.3:5432/nexusdb
The app can use:       postgresql://postgres:5432/nexusdb
                                    ^^^^^^^^
                                    Container name — Docker DNS resolves this
```

This means containers can be replaced (getting new IPs) without changing application configuration — the name stays the same.

---

### Network Isolation

Containers on different networks cannot communicate by default:

```
nexus-frontend-net:              nexus-backend-net:
├── nginx (frontend)             ├── flask-api
└── (cannot reach backend)       └── postgres

nginx → flask-api: BLOCKED (different networks)
nginx → postgres: BLOCKED (different networks)
flask-api → postgres: ALLOWED (same network)
```

To allow communication between networks, a container must be connected to both:

```bash
# flask-api connects frontend to backend
docker network connect nexus-frontend-net flask-api
# Now: nginx → flask-api: ALLOWED (both on frontend net)
# But: nginx → postgres: STILL BLOCKED (nginx not on backend net)
```

---

## Part 4: Practical Exercises

### Task 1: Inspect the Default Bridge Network

```bash
# Step 1: Inspect the default bridge network before any containers
docker network inspect bridge

# Note: Subnet (172.17.0.0/16), Gateway (172.17.0.1), Containers: {}

# Step 2: Run two containers on the default bridge
docker run -d --name default1 nginx
docker run -d --name default2 nginx

# Step 3: Inspect again — now shows connected containers
docker network inspect bridge
# Note the IP addresses assigned to each container

# Step 4: Get the IP of default1
IP1=$(docker inspect -f \
    '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' default1)
echo "default1 IP: $IP1"

# Step 5: Try to reach default1 from default2 by IP
docker exec default2 curl -s --max-time 3 http://$IP1 | head -5
# This works — same network, same bridge

# Step 6: Try by container name (this will FAIL on default bridge)
docker exec default2 ping -c 2 default1
# ping: default1: Name or service not known
# This is the limitation of the default bridge — no DNS

# Step 7: Clean up
docker stop default1 default2
docker rm default1 default2
```

---

### Task 2: Run Containers in Different Network Modes

```bash
# ── Bridge mode (default) ───────────────────────────────────────
docker run -d --name bridge-container nginx
docker inspect bridge-container | grep IPAddress
# Gets its own IP (e.g., 172.17.0.2)
docker stop bridge-container && docker rm bridge-container


# ── Host mode ───────────────────────────────────────────────────
docker run -d --network host --name host-container nginx
# nginx listens on host's port 80 directly — no -p needed
docker inspect host-container | grep IPAddress
# IPAddress is empty — uses host's IP
curl http://localhost    # Direct access on host port 80
docker stop host-container && docker rm host-container


# ── None mode ───────────────────────────────────────────────────
docker run -it --network none --name isolated ubuntu /bin/bash
# Inside the container:
ip addr show           # Only lo (127.0.0.1) — no eth0
ping -c 1 8.8.8.8     # ping: connect: Network is unreachable
exit
docker rm isolated
```

---

### Exercise 1: Communication Between Containers on a Custom Network

```bash
# Step 1: Create a custom bridge network
docker network create nexus-app-net

# Step 2: Start two containers on the custom network
docker run -d \
    --name nexus-web \
    --network nexus-app-net \
    nginx

docker run -d \
    --name nexus-api \
    --network nexus-app-net \
    nginx

# Step 3: Verify both are on the network
docker network inspect nexus-app-net
# Both containers should appear under "Containers"

# Step 4: Test communication by CONTAINER NAME (DNS)
docker exec nexus-web curl -s http://nexus-api | head -5
# Should return nginx welcome page HTML

# Step 5: Test reverse communication
docker exec nexus-api curl -s http://nexus-web | head -5
# Should also work

# Step 6: Test ping by name
docker exec nexus-web apt update -q && \
    docker exec nexus-web apt install -y iputils-ping -q

docker exec nexus-web ping -c 3 nexus-api
# Should succeed — container name resolves to IP via Docker DNS

# Step 7: Check IPs
docker inspect nexus-web \
    --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
docker inspect nexus-api \
    --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'

# Step 8: Clean up
docker stop nexus-web nexus-api
docker rm nexus-web nexus-api
docker network rm nexus-app-net
```

---

### Exercise 2: Network Isolation

```bash
# Step 1: Create two separate networks
docker network create nexus-frontend-net
docker network create nexus-backend-net

# Step 2: Place containers on separate networks
docker run -d \
    --name frontend \
    --network nexus-frontend-net \
    nginx

docker run -d \
    --name backend \
    --network nexus-backend-net \
    nginx

# Step 3: Try to communicate across networks — should FAIL
docker exec frontend apt update -q
docker exec frontend apt install -y iputils-ping curl -q

# Get backend's IP
BACKEND_IP=$(docker inspect backend \
    --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}')
echo "Backend IP: $BACKEND_IP"

# Try to reach backend from frontend
docker exec frontend ping -c 2 $BACKEND_IP
# Should fail — no route to host

docker exec frontend ping -c 2 backend
# Should fail — different network, no DNS

echo "Isolation confirmed: frontend cannot reach backend"


# Step 4: Bridge the gap — connect a container to both networks
docker run -d \
    --name api-bridge \
    --network nexus-frontend-net \
    nginx

# Also connect api-bridge to the backend network
docker network connect nexus-backend-net api-bridge

# Step 5: Verify api-bridge can now reach both
docker exec api-bridge apt update -q
docker exec api-bridge apt install -y iputils-ping curl -q

docker exec api-bridge ping -c 2 frontend
# SUCCESS — api-bridge and frontend are on the same network

docker exec api-bridge ping -c 2 $BACKEND_IP
# SUCCESS — api-bridge is also on the backend network

# Step 6: frontend still cannot reach backend directly
docker exec frontend ping -c 2 $BACKEND_IP
# STILL FAILS — frontend is not on backend network


# Step 7: Clean up
docker stop frontend backend api-bridge
docker rm frontend backend api-bridge
docker network rm nexus-frontend-net nexus-backend-net
```

---

### Exercise 3: Exploring Network Drivers

**Research questions to investigate:**

```
1. What is the default Docker network used for container communication?
   → The default bridge network (docker0). Containers on the same
     bridge can communicate via IP. But DNS name resolution only
     works on user-defined bridge networks.

2. Besides IP addresses, what other names can be used for container communication?
   → On user-defined bridge networks: container name, container
     hostname (set with --hostname), and network aliases
     (set with --network-alias).

3. How do you determine a container's IP address on a Docker network?
   → docker inspect <container> | grep IPAddress
   → docker inspect -f '{{range .NetworkSettings.Networks}}
     {{.IPAddress}}{{end}}' <container>
   → docker exec <container> hostname -I
   → docker network inspect <network>
```

**Exploring overlay and macvlan:**

```bash
# Overlay networks require Docker Swarm — initialize first
docker swarm init

# Create an overlay network
docker network create --driver overlay nexus-overlay

# List to confirm
docker network ls | grep overlay

# Leave Swarm when done
docker swarm leave --force


# macvlan requires knowing your host network interface
# and should be done carefully (requires promiscuous mode)
# Research only — do not run on cloud instances without understanding impact
docker network create \
    --driver macvlan \
    --subnet 192.168.1.0/24 \
    --gateway 192.168.1.1 \
    -o parent=eth0 \
    nexus-macvlan
```

---

## 🔧 Mini-Project: NexusCorp Multi-Tier Network Architecture

**Scenario:** Build a three-tier network architecture — frontend, application, and database tiers — with proper isolation between each tier.

```bash
#!/bin/bash
# nexus_network_setup.sh
# Build NexusCorp's isolated multi-tier network

echo "=================================================="
echo "  NexusCorp Multi-Tier Network Setup"
echo "=================================================="

# ── Step 1: Create network tiers ────────────────────────────────
echo ""
echo "Creating network tiers..."
docker network create nexus-frontend-tier
docker network create nexus-backend-tier
echo "  ✓ nexus-frontend-tier created"
echo "  ✓ nexus-backend-tier created"


# ── Step 2: Start the database (backend tier only) ───────────────
echo ""
echo "Starting database container..."
docker run -d \
    --name nexus-db \
    --network nexus-backend-tier \
    -e MYSQL_ROOT_PASSWORD=nexuspass \
    -e MYSQL_DATABASE=nexusdb \
    alpine sleep infinity
echo "  ✓ nexus-db started (backend tier only)"


# ── Step 3: Start the API (both tiers — the bridge) ─────────────
echo ""
echo "Starting API container..."
docker run -d \
    --name nexus-api \
    --network nexus-backend-tier \
    nginx
# Also connect to frontend tier
docker network connect nexus-frontend-tier nexus-api
echo "  ✓ nexus-api started (both tiers)"


# ── Step 4: Start the frontend (frontend tier only) ──────────────
echo ""
echo "Starting frontend container..."
docker run -d \
    --name nexus-frontend \
    --network nexus-frontend-tier \
    -p 8080:80 \
    nginx
echo "  ✓ nexus-frontend started (frontend tier only, port 8080)"


# ── Step 5: Inspect the architecture ────────────────────────────
echo ""
echo "=================================================="
echo "  Network Architecture"
echo "=================================================="

echo ""
echo "nexus-frontend-tier:"
docker network inspect nexus-frontend-tier \
    --format '{{range .Containers}}  - {{.Name}} ({{.IPv4Address}}){{println}}{{end}}'

echo "nexus-backend-tier:"
docker network inspect nexus-backend-tier \
    --format '{{range .Containers}}  - {{.Name}} ({{.IPv4Address}}){{println}}{{end}}'


# ── Step 6: Verify isolation rules ──────────────────────────────
echo ""
echo "=================================================="
echo "  Verifying Network Isolation"
echo "=================================================="

# Install ping on containers
docker exec nexus-frontend apt-get update -qq && \
    docker exec nexus-frontend apt-get install -y iputils-ping curl -qq

DB_IP=$(docker inspect nexus-db \
    --format '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' | \
    awk '{print $1}')

echo ""
echo "Test 1: Frontend → API (should SUCCEED)"
docker exec nexus-frontend curl -s --max-time 3 http://nexus-api > /dev/null && \
    echo "  ✓ PASS: Frontend can reach API" || \
    echo "  ✗ FAIL"

echo ""
echo "Test 2: Frontend → Database (should FAIL — isolated)"
docker exec nexus-frontend ping -c 1 -W 2 $DB_IP > /dev/null 2>&1 && \
    echo "  ✗ FAIL: Frontend can reach DB (unexpected)" || \
    echo "  ✓ PASS: Frontend cannot reach DB (isolation working)"

echo ""
echo "Test 3: API → Database (should SUCCEED)"
docker exec nexus-api apt-get update -qq && \
    docker exec nexus-api apt-get install -y iputils-ping -qq
docker exec nexus-api ping -c 1 $DB_IP > /dev/null && \
    echo "  ✓ PASS: API can reach Database" || \
    echo "  ✗ FAIL"


# ── Step 7: Clean up ────────────────────────────────────────────
echo ""
echo "=================================================="
echo "  Cleanup"
echo "=================================================="
docker stop nexus-frontend nexus-api nexus-db
docker rm nexus-frontend nexus-api nexus-db
docker network rm nexus-frontend-tier nexus-backend-tier
echo "  ✓ All containers and networks removed"
echo ""
echo "Done."
```

```bash
chmod +x nexus_network_setup.sh
./nexus_network_setup.sh
```

**Architecture diagram of what you built:**

```
Internet
    │
    │ port 8080
    ▼
nexus-frontend (nginx)
    │
    │ nexus-frontend-tier
    ▼
nexus-api (nginx — on BOTH tiers)
    │
    │ nexus-backend-tier
    ▼
nexus-db (database — backend tier ONLY)

✓ Frontend → API:        ALLOWED (same frontend tier)
✓ API → Database:        ALLOWED (same backend tier)
✗ Frontend → Database:   BLOCKED (different tiers — isolation working)
✗ External → API:        BLOCKED (no port mapping on API)
✗ External → Database:   BLOCKED (no port mapping on DB)
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Docker network** | An isolated network that containers can be connected to |
| **Network driver** | The technology that implements a Docker network (bridge, host, overlay, macvlan, ipvlan, none) |
| **`bridge`** | Default Docker network driver — creates a virtual bridge between host and containers |
| **`host`** | Network driver that makes containers share the host's network stack |
| **`overlay`** | Network driver for multi-host container communication (requires Swarm) |
| **`macvlan`** | Network driver that gives each container a unique MAC address on the physical network |
| **`ipvlan`** | Network driver with precise IP and VLAN control, sharing host MAC address |
| **`none`** | Network driver that disables all networking (loopback only) |
| **`docker0`** | The default virtual bridge interface created by Docker on the host |
| **Default bridge network** | The `bridge` network containers join automatically — no DNS resolution |
| **User-defined bridge** | A custom bridge network with DNS resolution by container name |
| **Network namespace** | An isolated network stack (interfaces, routing, iptables) assigned to each container |
| **`iptables`** | Linux firewall tool that Docker manipulates to route traffic between containers |
| **veth pair** | A virtual Ethernet cable pair — one end in the container, one in the bridge |
| **`docker network ls`** | Lists all networks on the Docker host |
| **`docker network create`** | Creates a new network |
| **`docker network inspect`** | Shows detailed JSON metadata about a network |
| **`docker network connect`** | Connects a running container to an additional network |
| **`docker network disconnect`** | Removes a container from a network |
| **`docker network rm`** | Deletes a network (must have no connected containers) |
| **`docker network prune`** | Removes all unused networks |
| **DNS resolution** | The ability to reach a container by its name rather than its IP address |
| **Network alias** | An alternative name for a container on a specific network (`--network-alias`) |
| **Subnet** | A range of IP addresses assigned to a network (e.g., `172.17.0.0/16`) |
| **Gateway** | The IP address of the bridge interface — the default route for containers |
| **Port mapping (`-p`)** | Forwards traffic from a host port to a container port |
| **NAT** | Network Address Translation — how containers reach the internet via the host's IP |
| **Promiscuous mode** | NIC mode required by macvlan — allows receiving packets for other MAC addresses |
| **Docker Swarm** | Docker's native cluster orchestration — required for overlay networks |
| **`--network-alias`** | An additional DNS name for a container on a specific network |

---

## 📚 Resources

- [Docker Networking Overview — Official Docs](https://docs.docker.com/network/) — complete networking reference
- [Networking with Standalone Containers — Docker Docs](https://docs.docker.com/network/network-tutorial-standalone/) — official hands-on tutorial
- [Bridge Network Tutorial — Docker Docs](https://docs.docker.com/network/network-tutorial-standalone/) — user-defined bridge walkthrough
- [Docker Network Drivers](https://docs.docker.com/network/drivers/) — detailed explanation of each driver
- [Overlay Networking Tutorial — Docker Docs](https://docs.docker.com/network/network-tutorial-overlay/) — multi-host networking guide

---

## Self-Study Questions

Work through these before moving to Day 28. Research the answers if needed and add them to your reference guide.

```
1. What is the default Docker network used for container communication?

2. Besides IP addresses, what other names can be used for container
   communication on user-defined bridge networks?
   Hint: container name, hostname (--hostname), network alias (--network-alias)

3. How do you determine a container's IP address on a Docker network?
   List at least three methods.

4. What is the difference between docker network connect and
   specifying --network at docker run time?

5. If Container A is on network1 and Container B is on network2,
   what must you do to allow them to communicate?

6. Why would you use an overlay network instead of a bridge network?

7. What security risk exists when using host networking?

8. In a production microservice architecture, how would you use
   Docker networks to implement a security boundary between
   your web tier, application tier, and database tier?
```

---

## 🔭 Day 28 Preview: Docker Volumes & Persistent Storage

Today you controlled how containers communicate. Tomorrow you tackle how containers **store data**.

By default, data inside a container disappears when the container is removed. For databases, file uploads, logs, and configuration — this is unacceptable. Docker volumes solve this.

You will learn:
- Why container storage is ephemeral by default
- `docker volume create`, `docker volume ls`, `docker volume inspect`
- Bind mounts — mounting host directories into containers
- Named volumes — Docker-managed persistent storage
- Volume sharing between containers
- Best practices for database containers and persistent data

Understanding volumes completes the container fundamentals — after networking and storage, you have everything needed to build real multi-container applications.

---

> 💬 *"In Docker networking, isolation is the default and communication is the exception you explicitly permit. Design your networks before your containers."*
>
> — DevOps Learning By Yukta
