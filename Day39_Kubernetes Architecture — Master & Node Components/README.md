# Day 39: Kubernetes Architecture — Master & Node Components

> **DevOps Transformation** | Kubernetes & Container Orchestration Series

---

## 📋 Overview

Yesterday you deployed your first application to Kubernetes and ran basic `kubectl` commands. Today you go under the hood — understanding exactly what happens when you type `kubectl apply` and why Kubernetes is architected the way it is.

Every Kubernetes operation involves multiple components working in concert. Knowing what each component does transforms Kubernetes from a magic black box into a comprehensible, debuggable system. When something goes wrong — and it will — you will know exactly where to look.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| Master-Node Architecture | The control plane / worker node split |
| Master Components | API Server, etcd, Scheduler, Controller Manager, Cloud Controller |
| Node Components | kubelet, kube-proxy, Container Runtime |
| End-to-End Workflow | What happens from `kubectl` to running container |
| Task 1 | Cluster inspection with kubectl |
| Task 2 | Accessing and exploring the API server |
| Task 3 | Visualizing components with Dashboard or Lens |

---

## 🏢 NexusCorp Scenario

> NexusCorp's platform team just had their first Kubernetes production incident. A pod was stuck in `Pending` state for 20 minutes. Nobody knew why. The fix was straightforward once someone ran `kubectl describe pod` and saw the scheduler couldn't find a node with enough memory — but it took 20 minutes to diagnose because nobody understood the component that was responsible.
>
> After that incident, the team ran an architecture review session. Today's content is that session. Understanding each component means knowing exactly which log to check when something goes wrong.

---

## Part 1: Kubernetes Architecture Overview

### The Master-Worker Split

Kubernetes follows a **control plane / data plane** architecture (often called master-slave, though the modern terminology is controller/worker):

```
┌─────────────────────────────────────────────────────────────────┐
│                     Kubernetes Cluster                          │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                   Control Plane (Master)                  │  │
│  │                                                          │  │
│  │  ┌──────────────┐  ┌──────┐  ┌───────────┐  ┌────────┐  │  │
│  │  │ kube-        │  │      │  │ kube-     │  │ kube-  │  │  │
│  │  │ apiserver    │←─│ etcd │  │ scheduler │  │control-│  │  │
│  │  │              │  │      │  │           │  │ler-mgr │  │  │
│  │  └──────┬───────┘  └──────┘  └───────────┘  └────────┘  │  │
│  │         │           ↑              ↑              ↑       │  │
│  │         └───────────┴──────────────┴──────────────┘       │  │
│  │                    All talk to API Server                  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                            │ API calls                          │
│         ┌──────────────────┼──────────────────┐                │
│         ▼                  ▼                  ▼                │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│  │ Worker Node │   │ Worker Node │   │ Worker Node │          │
│  │             │   │             │   │             │          │
│  │ kubelet     │   │ kubelet     │   │ kubelet     │          │
│  │ kube-proxy  │   │ kube-proxy  │   │ kube-proxy  │          │
│  │ container   │   │ container   │   │ container   │          │
│  │ runtime     │   │ runtime     │   │ runtime     │          │
│  │             │   │             │   │             │          │
│  │ [Pod][Pod]  │   │ [Pod][Pod]  │   │ [Pod][Pod]  │          │
│  └─────────────┘   └─────────────┘   └─────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

**Master (Control Plane):**
- Manages, plans, schedules, and monitors the entire cluster
- Makes decisions about scheduling pods, detecting failures, responding to events
- Does NOT run application workloads (in production setups)

**Node (Worker):**
- The machine where your application containers actually run
- Can be a VM (EC2 instance, GCE instance) or a physical machine
- Receives instructions from the control plane and executes them

---

## Part 2: Master (Control Plane) Components

The control plane is the brain of Kubernetes. It consists of five components, each with a precise, distinct responsibility.

---

### 1. kube-apiserver — The Front Door

The API server is the **single entry point** for all control plane communication. Every command you run with `kubectl`, every internal component communication, every cloud provider integration — all goes through the API server.

```
Everything talks to the API server:

  kubectl apply -f deployment.yaml
        │
        ▼
  kube-apiserver (port 6443, HTTPS)
        │
        ├── Authenticates the request (who are you?)
        ├── Authorizes the request (are you allowed to do this?)
        ├── Validates the request (is the YAML correct?)
        ├── Persists the object to etcd
        └── Notifies other components (scheduler, controllers)

Other components that talk to API server:
  ├── kubelet (on each node)
  ├── kube-scheduler
  ├── kube-controller-manager
  ├── kube-proxy
  └── Any external tool (Helm, ArgoCD, monitoring)
```

**Key properties:**
- Stateless — all state is stored in etcd, not in the API server itself
- Horizontally scalable — can run multiple API server instances behind a load balancer
- RESTful — exposes a REST API, which is why `curl` can interact with it
- All API calls are logged and auditable

```bash
# The API server exposes all Kubernetes resources as REST endpoints
# GET all pods:
curl -k https://CLUSTER-IP:6443/api/v1/pods \
     -H "Authorization: Bearer TOKEN"

# GET a specific deployment:
curl -k https://CLUSTER-IP:6443/apis/apps/v1/namespaces/default/deployments/nexus-api \
     -H "Authorization: Bearer TOKEN"
```

---

### 2. etcd — The Cluster Brain's Memory

**etcd** is a distributed, highly-available key-value store that serves as Kubernetes' backing store for all cluster data. Think of it as the cluster's database — everything Kubernetes knows about the cluster is stored here.

```
What etcd stores:
  ├── All pod definitions
  ├── All deployment configurations
  ├── All node registrations
  ├── All service definitions
  ├── All secrets and config maps
  ├── All RBAC policies
  ├── All events
  └── The "desired state" — what you told Kubernetes you want

What etcd does NOT store:
  └── Actual container data (that's on the nodes)

etcd Architecture:
  ┌────────┐   ┌────────┐   ┌────────┐
  │ etcd 1 │ ─ │ etcd 2 │ ─ │ etcd 3 │  ← Odd number (for quorum)
  │(leader)│   │(follow)│   │(follow)│
  └────────┘   └────────┘   └────────┘
       ▲
       │ API server writes/reads here
```

**Key properties:**
- Uses the **Raft consensus algorithm** — requires a majority (quorum) to write
- Highly available — typically runs as 3 or 5 instances
- **Backing up etcd = backing up the entire cluster state**
- Production clusters: etcd is often separated onto dedicated machines

```bash
# On a running cluster, you can see etcd pods:
kubectl get pods -n kube-system | grep etcd

# Backup etcd (critical for disaster recovery):
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key
```

---

### 3. kube-scheduler — The Placement Engine

The scheduler watches for newly created pods that have no node assigned, and selects a node for them to run on based on a series of criteria.

```
Scheduling decision process:

New Pod created (no node assigned)
        │
        ▼
kube-scheduler watches API server
        │
        ▼
Filtering phase:
  Can this pod run on each node?
  ├── Does the node have enough CPU?
  ├── Does the node have enough memory?
  ├── Does the node have the required storage?
  ├── Does the node match node selectors/affinity rules?
  ├── Does the node have the required taints tolerated?
  └── → Filtered nodes: [node1, node3]  (node2 eliminated)

Scoring phase:
  Rank remaining nodes:
  ├── node1: score 85 (has more free resources)
  ├── node3: score 72 (fewer free resources)
  └── → Winner: node1

Binding phase:
  → Updates the pod spec: nodeName = node1
  → API server writes to etcd
  → kubelet on node1 sees the pod and starts it
```

**Scheduling factors:**
- Resource requests (CPU, memory)
- Node selectors (`nodeSelector: disktype: ssd`)
- Node affinity (prefer or require certain nodes)
- Pod affinity/anti-affinity (co-locate or spread pods)
- Taints and tolerations (mark nodes for specific workloads)
- Priority and preemption (high-priority pods can evict lower-priority ones)

---

### 4. kube-controller-manager — The Reconciliation Engine

The controller manager runs a set of **controllers** — each a control loop that watches the current state of some resource and drives it toward the desired state.

```
The Control Loop pattern:

while true:
    current_state  = observe what's actually running
    desired_state  = read from etcd (what you declared)
    if current_state != desired_state:
        take_action_to_make_them_match()
    sleep(a few seconds)

Multiple controllers running:

Node Controller:
  → Monitors node health
  → Marks nodes as unavailable after timeout
  → Evicts pods from unavailable nodes

Replication Controller / ReplicaSet Controller:
  → Ensures desired number of pod replicas are running
  → You said "3 replicas" → current is 2 → starts 1 more
  → You said "3 replicas" → current is 4 → deletes 1

Deployment Controller:
  → Manages ReplicaSets for rolling updates
  → Creates new ReplicaSet on update
  → Scales up new, scales down old

Service Account Controller:
  → Creates default service accounts in new namespaces

Endpoints Controller:
  → Populates Service endpoints with pod IPs

Job Controller:
  → Watches Job resources, creates pods to run to completion

CronJob Controller:
  → Creates Jobs on schedule
```

---

### 5. cloud-controller-manager — The Cloud Integration Layer

The cloud controller manager interfaces with cloud providers (AWS, GCP, Azure) to integrate Kubernetes with cloud-specific features.

```
Cloud Controller Manager responsibilities:

Node Controller (cloud):
  → When a node is deleted in the cloud, removes it from cluster

Route Controller:
  → Configures routes in cloud networking for pod traffic

Service Controller:
  → When you create a LoadBalancer service, calls AWS/GCP/Azure API
  → Creates an actual cloud load balancer (AWS ELB, GCP GLB)
  → Configures it to point to the correct nodes

Volume Controller:
  → Interacts with cloud storage (AWS EBS, GCP PD)
  → Creates, attaches, and mounts volumes for pods
```

---

### Master Components Summary

| Component | Primary Job | What breaks if it goes down |
|---|---|---|
| **kube-apiserver** | Entry point for all cluster operations | Nothing works — complete outage |
| **etcd** | Store all cluster state | Cluster loses its memory — no changes possible |
| **kube-scheduler** | Assign pods to nodes | New pods never get placed (stay Pending) |
| **kube-controller-manager** | Drive cluster to desired state | Failed pods not replaced, scaling doesn't work |
| **cloud-controller-manager** | Cloud provider integration | LoadBalancers not created, cloud volumes not attached |

---

## Part 3: Node (Worker) Components

Each worker node runs three components that collectively receive instructions from the control plane and execute them.

---

### 1. kubelet — The Node Agent

The **kubelet** is an agent that runs on every worker node. It is the primary point of contact between the node and the control plane. The kubelet continuously watches the API server for pods that have been scheduled to its node, and ensures those pods are running.

```
kubelet responsibilities:

Pod Lifecycle Management:
  → Sees a new pod scheduled to this node
  → Pulls the container image (if not cached)
  → Creates and starts the containers
  → Monitors container health
  → Restarts containers that fail health checks
  → Reports pod status back to API server

Container Probes:
  → Runs liveness probes → restart container if unhealthy
  → Runs readiness probes → remove from service if not ready
  → Runs startup probes → give slow containers time to start

Node Status Reporting:
  → Reports node capacity (CPU, memory, disk)
  → Reports node conditions (Ready, MemoryPressure, DiskPressure)
  → Sends heartbeat to API server every few seconds

Volume Management:
  → Mounts volumes defined in pod specs
  → Manages secrets and configmaps as volume mounts
```

```bash
# View kubelet status (on a node)
sudo systemctl status kubelet

# View kubelet logs
sudo journalctl -u kubelet -f

# kubelet configuration
cat /var/lib/kubelet/config.yaml
```

---

### 2. kube-proxy — The Network Manager

**kube-proxy** runs on every node and maintains network rules. It is responsible for making Kubernetes Services work — when you access a Service, kube-proxy routes your traffic to the correct pod.

```
kube-proxy network model:

Service: nexus-api-service (ClusterIP: 10.96.45.12)
    │
    │ Traffic to 10.96.45.12:80
    ▼
kube-proxy (iptables rules on each node)
    │
    │ Load balance across pod IPs
    ├── 10.244.1.4:5000 (Pod 1 on Node 1)
    ├── 10.244.2.3:5000 (Pod 2 on Node 2)
    └── 10.244.3.5:5000 (Pod 3 on Node 3)

kube-proxy modes:
  iptables mode (default):
    → Programs iptables rules to intercept and redirect traffic
    → Very fast (kernel-level)
    → Cannot do real load balancing (uses random selection)

  ipvs mode (modern, recommended):
    → Uses Linux IPVS (IP Virtual Server)
    → True load balancing with multiple algorithms
    → Better performance at scale (10,000+ services)
```

---

### 3. Container Runtime — The Execution Engine

The container runtime is the software that actually runs containers on a node. Kubernetes doesn't run containers directly — it delegates to the container runtime via the **Container Runtime Interface (CRI)**.

```
Container Runtime Interface (CRI):

kubectl apply → API Server → kubelet
                                │
                                │ CRI calls
                                ▼
                        Container Runtime
                        ┌──────────────────────┐
                        │ containerd (default) │
                        │ CRI-O                │
                        │ Docker (via shim)    │
                        └──────────────────────┘
                                │
                                │ OCI (Open Container Initiative)
                                ▼
                        Container (runc, etc.)
```

**Container runtimes:**

| Runtime | Description | Notes |
|---|---|---|
| **containerd** | Lightweight, CNCF-graduated | Default in most distributions, K8s 1.20+ |
| **CRI-O** | Purpose-built for Kubernetes | Default in OpenShift |
| **Docker** | Via dockershim (deprecated) | Support removed in K8s 1.24 |

```bash
# Check which runtime your nodes use
kubectl get nodes -o wide
# CONTAINER-RUNTIME column shows: containerd://1.7.x

# On a node, see running containers via containerd
sudo ctr containers list
sudo crictl ps
```

---

### Node Components Summary

| Component | Primary Job | What breaks if it goes down |
|---|---|---|
| **kubelet** | Runs and monitors pods on this node | Pods on this node stop being managed |
| **kube-proxy** | Maintains network routing to pods | Services can't route traffic to pods |
| **Container Runtime** | Actually starts and stops containers | No containers can run on this node |

---

## Part 4: End-to-End Workflow

What actually happens when you run `kubectl apply -f deployment.yaml`?

```
Step 1: kubectl sends request
──────────────────────────────
kubectl reads deployment.yaml
→ Sends HTTP PUT/POST to https://api-server:6443/apis/apps/v1/.../deployments
→ Includes your authentication token

Step 2: API Server validates and stores
──────────────────────────────────────
kube-apiserver:
  → Authenticates: Is this token valid?
  → Authorizes: Can this user create Deployments in this namespace?
  → Validates: Is the YAML schema correct?
  → Writes the Deployment object to etcd
  → Responds to kubectl: "deployment.apps/nexus-api created"

Step 3: Deployment Controller reacts
──────────────────────────────────────
kube-controller-manager (Deployment Controller):
  → Watches API server for new Deployments
  → Sees "nexus-api" Deployment with replicas: 3
  → Creates a ReplicaSet to manage 3 pods
  → ReplicaSet controller creates 3 Pod objects in API server
  → Pods are written to etcd with status: Pending, nodeName: ""

Step 4: Scheduler assigns pods to nodes
─────────────────────────────────────────
kube-scheduler:
  → Watches API server for unscheduled pods
  → Sees 3 pending pods with no nodeName
  → For each pod: filters nodes, scores them, picks the best
  → Updates pod in etcd: nodeName = "worker-node-2"

Step 5: kubelet starts the containers
──────────────────────────────────────
kubelet on worker-node-2:
  → Watches API server for pods assigned to this node
  → Sees a new pod assigned to it
  → Calls container runtime (containerd):
    → Pull image: nexuscorp/nexus-api:v2 from registry
    → Create container with specified resources
    → Start container
  → Sets up volumes (ConfigMaps, Secrets, PVCs)
  → Runs readiness probe: GET /health → waits for 200 OK
  → Updates pod status in API server: Running

Step 6: kube-proxy enables networking
──────────────────────────────────────
kube-proxy on each node:
  → Watches API server for Endpoints changes
  → Sees new Pod IPs added to nexus-api-service endpoints
  → Updates iptables rules to route traffic to new pods

Step 7: Application is accessible
──────────────────────────────────────
User request → Service (ClusterIP/LoadBalancer)
→ kube-proxy routes to one of the healthy pods
→ Container handles request
→ Response returned to user
```

This entire sequence — from `kubectl apply` to serving traffic — typically takes **10-30 seconds** for a simple deployment.

---

## Part 5: Practical Tasks

### Task 1: Cluster Inspection

```bash
# ── Inspect all nodes ──────────────────────────────────────────
kubectl get nodes
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   10m   v1.28.0

kubectl get nodes -o wide
# Shows: OS, container runtime, kernel version, internal/external IPs

# Describe a node in full detail
kubectl describe node minikube

# Key sections in describe output:
# Addresses:          IP addresses of the node
# Conditions:         Ready, MemoryPressure, DiskPressure, PIDPressure
# Capacity:           Total CPU, memory, pods the node can handle
# Allocatable:        Resources available for pods (after system reservation)
# System Info:        OS, kernel, Docker/containerd version, kubelet version
# Non-terminated Pods: All pods currently running on this node
# Events:             Recent events on this node
```

```bash
# ── Inspect control plane components ──────────────────────────
kubectl get pods -n kube-system

# You'll see (on Minikube):
# NAME                               READY   STATUS    RESTARTS
# coredns-xxx                        1/1     Running   0
# etcd-minikube                      1/1     Running   0
# kube-apiserver-minikube            1/1     Running   0
# kube-controller-manager-minikube   1/1     Running   0
# kube-proxy-xxx                     1/1     Running   0
# kube-scheduler-minikube            1/1     Running   0
# storage-provisioner                1/1     Running   0

# Describe each control plane component
kubectl describe pod kube-apiserver-minikube -n kube-system
kubectl describe pod etcd-minikube -n kube-system
kubectl describe pod kube-scheduler-minikube -n kube-system
kubectl describe pod kube-controller-manager-minikube -n kube-system

# View component logs
kubectl logs -n kube-system kube-apiserver-minikube
kubectl logs -n kube-system kube-scheduler-minikube
kubectl logs -n kube-system kube-controller-manager-minikube
```

```bash
# ── Check cluster component health ─────────────────────────────
kubectl get componentstatuses
# or (newer clusters):
kubectl get --raw='/readyz?verbose'
kubectl get --raw='/healthz?verbose'
kubectl get --raw='/livez?verbose'

# View node resource usage
kubectl top nodes   # Requires metrics-server

# Enable metrics server on Minikube
minikube addons enable metrics-server
kubectl top nodes
kubectl top pods -A
```

---

### Task 2: Exploring the API Server

```bash
# ── Access API server with kubectl proxy ───────────────────────
# Start a proxy to the API server (handles authentication)
kubectl proxy --port=8001 &

# Now access the API via localhost (no auth headers needed)
# List all API groups
curl http://localhost:8001/apis | python3 -m json.tool | head -50

# List all pods in default namespace
curl http://localhost:8001/api/v1/namespaces/default/pods | \
    python3 -m json.tool

# Get a specific deployment
curl http://localhost:8001/apis/apps/v1/namespaces/default/deployments

# Get nodes
curl http://localhost:8001/api/v1/nodes | python3 -m json.tool

# Stop the proxy
kill %1


# ── Direct API server access ───────────────────────────────────
# Get the API server address
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
echo $APISERVER
# https://192.168.49.2:8443 (Minikube)
# https://YOUR-EKS-ENDPOINT.region.eks.amazonaws.com (EKS)

# Get a service account token for authentication
TOKEN=$(kubectl get secret \
    $(kubectl get sa default -o jsonpath='{.secrets[0].name}') \
    -o jsonpath='{.data.token}' | base64 -d 2>/dev/null || \
    kubectl create token default)

# Access API server directly (skip TLS verification for local)
curl -k $APISERVER/api/v1/nodes \
    -H "Authorization: Bearer $TOKEN"

# In production, use the CA cert instead of -k:
curl $APISERVER/api/v1/nodes \
    --cacert /path/to/ca.crt \
    -H "Authorization: Bearer $TOKEN"


# ── Explore API resources ──────────────────────────────────────
# List all available API resources
kubectl api-resources

# List all API versions
kubectl api-versions

# Get raw API resource definition
kubectl get --raw /api/v1 | python3 -m json.tool | head -50

# Watch API events in real time
kubectl get events -w

# Watch all resource changes
kubectl get all -w
```

---

### Task 3: Visualizing Components

#### Option A: Kubernetes Dashboard

```bash
# Enable on Minikube
minikube addons enable dashboard

# Open the dashboard
minikube dashboard
# Opens browser automatically at:
# http://127.0.0.1:PORT/api/v1/namespaces/kubernetes-dashboard/services/...

# Or deploy manually:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Create admin service account
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Get the login token
kubectl -n kubernetes-dashboard create token admin-user

# Access dashboard (in another terminal)
kubectl proxy

# Open:
# http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
# → Paste the token to log in
```

---

#### Option B: Lens Desktop (Recommended)

```
Lens is a desktop application for managing Kubernetes clusters.

Install:
  Download from: https://k8slens.dev/
  Available for: Windows, macOS, Linux

Features:
  → Visual cluster overview (nodes, pods, deployments)
  → Real-time resource monitoring (CPU, memory)
  → Log viewer per pod
  → Terminal inside pods
  → YAML editor for resources
  → Multi-cluster management

Connect to Minikube:
  1. Open Lens
  2. File → Add Cluster
  3. Select kubeconfig: ~/.kube/config
  4. Select context: minikube
  5. Click "Add to Lens"
```

---

#### Option C: K9s — Terminal Dashboard

```bash
# Install k9s (lightweight terminal UI)

# Linux
curl -sS https://webinstall.dev/k9s | bash

# macOS
brew install k9s

# Run k9s
k9s

# Inside k9s:
# :pods     → View all pods
# :nodes    → View all nodes
# :deploy   → View deployments
# :svc      → View services
# :events   → View cluster events
# /pattern  → Filter by pattern
# d         → Describe resource
# l         → View logs
# s         → Shell into container
# Ctrl+A    → View all resources
# ?         → Help / keyboard shortcuts
```

---

## 🔧 Mini-Project: NexusCorp Architecture Deep Inspection

```bash
#!/bin/bash
# k8s_architecture_inspection.sh
# Inspect every component of a running Kubernetes cluster

set -e
echo "=============================================="
echo "  NexusCorp Kubernetes Architecture Inspection"
echo "=============================================="

# ── Control Plane ──────────────────────────────────────────────
echo ""
echo "=== CONTROL PLANE COMPONENTS ==="

echo ""
echo "1. API Server:"
kubectl get pod -n kube-system -l component=kube-apiserver \
    -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName"

echo ""
echo "2. etcd:"
kubectl get pod -n kube-system -l component=etcd \
    -o custom-columns="NAME:.metadata.name,STATUS:.status.phase"

echo ""
echo "3. Scheduler:"
kubectl get pod -n kube-system -l component=kube-scheduler \
    -o custom-columns="NAME:.metadata.name,STATUS:.status.phase"

echo ""
echo "4. Controller Manager:"
kubectl get pod -n kube-system -l component=kube-controller-manager \
    -o custom-columns="NAME:.metadata.name,STATUS:.status.phase"


# ── Worker Nodes ───────────────────────────────────────────────
echo ""
echo "=== WORKER NODES ==="

echo ""
echo "5. Nodes:"
kubectl get nodes -o wide

echo ""
echo "6. Node Capacity:"
kubectl get nodes \
    -o custom-columns="NAME:.metadata.name,CPU:.status.capacity.cpu,MEMORY:.status.capacity.memory,PODS:.status.capacity.pods"

echo ""
echo "7. Node Conditions:"
kubectl get nodes \
    -o custom-columns="NAME:.metadata.name,READY:.status.conditions[-1].type,STATUS:.status.conditions[-1].status"

echo ""
echo "8. kube-proxy (one per node):"
kubectl get pod -n kube-system -l k8s-app=kube-proxy \
    -o custom-columns="NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase"

echo ""
echo "9. CoreDNS:"
kubectl get pod -n kube-system -l k8s-app=kube-dns \
    -o custom-columns="NAME:.metadata.name,STATUS:.status.phase"


# ── All System Pods ────────────────────────────────────────────
echo ""
echo "=== ALL SYSTEM PODS ==="
kubectl get pods -n kube-system \
    --sort-by=.metadata.name \
    -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName"


# ── Cluster Health Summary ─────────────────────────────────────
echo ""
echo "=============================================="
echo "  Cluster Health Summary"
echo "=============================================="

NODES_READY=$(kubectl get nodes \
    --no-headers | grep -c "Ready" || echo "0")
NODES_TOTAL=$(kubectl get nodes \
    --no-headers | wc -l)
PODS_RUNNING=$(kubectl get pods -A \
    --no-headers | grep -c "Running" || echo "0")
PODS_TOTAL=$(kubectl get pods -A \
    --no-headers | wc -l)

echo ""
echo "  Nodes:   ${NODES_READY}/${NODES_TOTAL} Ready"
echo "  Pods:    ${PODS_RUNNING}/${PODS_TOTAL} Running"
echo ""

kubectl get componentstatuses 2>/dev/null || \
    echo "  (componentstatuses deprecated — cluster appears healthy)"

echo ""
echo "=============================================="
echo "  Inspection complete"
echo "=============================================="
```

```bash
chmod +x k8s_architecture_inspection.sh
./k8s_architecture_inspection.sh
```

---

## Reference: kubectl Quick Commands for Component Inspection

```bash
# ── Master components ──────────────────────────────────────────
kubectl get pods -n kube-system                      # All system pods
kubectl describe pod kube-apiserver-minikube -n kube-system
kubectl describe pod etcd-minikube -n kube-system
kubectl describe pod kube-scheduler-minikube -n kube-system
kubectl describe pod kube-controller-manager-minikube -n kube-system

# ── Node inspection ────────────────────────────────────────────
kubectl get nodes                                    # List all nodes
kubectl get nodes -o wide                            # With extra details
kubectl describe node minikube                       # Full node details

# ── Logs from components ───────────────────────────────────────
kubectl logs -n kube-system kube-apiserver-minikube
kubectl logs -n kube-system kube-scheduler-minikube
kubectl logs -n kube-system etcd-minikube

# ── API server exploration ─────────────────────────────────────
kubectl api-resources                                # All resource types
kubectl api-versions                                 # All API versions
kubectl proxy &                                      # Start proxy
curl http://localhost:8001/apis                      # Explore API groups
curl http://localhost:8001/api/v1/nodes              # Raw node data

# ── Component status ───────────────────────────────────────────
kubectl get --raw='/healthz'                         # API server health
kubectl get --raw='/readyz'                          # Ready check
kubectl get events -A --sort-by='.lastTimestamp'     # Recent events

# ── Resource usage ─────────────────────────────────────────────
kubectl top nodes                                    # Node CPU/memory
kubectl top pods -A                                  # Pod CPU/memory
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Control Plane** | The set of components that manage the Kubernetes cluster — the "brain" |
| **Worker Node** | A machine in the cluster that runs application pods |
| **kube-apiserver** | The single entry point for all Kubernetes API operations |
| **etcd** | The distributed key-value store holding all cluster state |
| **kube-scheduler** | Assigns new pods to nodes based on resource availability and constraints |
| **kube-controller-manager** | Runs control loops that maintain desired cluster state |
| **cloud-controller-manager** | Integrates Kubernetes with cloud provider APIs |
| **kubelet** | Node agent that manages pod lifecycle and reports status to control plane |
| **kube-proxy** | Maintains network routing rules for Kubernetes Services on each node |
| **Container Runtime** | Software that runs containers — containerd, CRI-O |
| **CRI** | Container Runtime Interface — how kubelet talks to container runtimes |
| **OCI** | Open Container Initiative — standards for container images and runtimes |
| **containerd** | The default container runtime in modern Kubernetes clusters |
| **Raft** | The consensus algorithm etcd uses to ensure consistency across replicas |
| **Quorum** | Majority of etcd members needed to write — requires odd number (3, 5) |
| **Control Loop** | A loop that reads desired state, observes current state, and takes action |
| **Desired State** | What you declared you want (in YAML) — Kubernetes drives toward this |
| **Current State** | What's actually running in the cluster right now |
| **Reconciliation** | The process of comparing desired and current state and correcting differences |
| **Endpoints** | The list of pod IPs backing a Kubernetes Service |
| **iptables** | Linux firewall tool used by kube-proxy to route traffic to pods |
| **IPVS** | IP Virtual Server — alternative to iptables for high-performance kube-proxy |
| **Node Conditions** | Status reports from each node: Ready, MemoryPressure, DiskPressure |
| **Allocatable** | Node resources available for pods after reserving for system processes |
| **ComponentStatus** | Kubernetes API endpoint showing health of control plane components |
| **kubectl proxy** | Command that starts a proxy to the Kubernetes API server on localhost |
| **Kubernetes Dashboard** | Web UI for managing and visualizing Kubernetes clusters |
| **Lens** | Desktop application for managing Kubernetes clusters visually |
| **k9s** | Terminal-based visual dashboard for Kubernetes |
| **CoreDNS** | The DNS server running in kube-system that provides cluster DNS |

---

## 📚 Resources

- [Kubernetes Components — Official Documentation](https://kubernetes.io/docs/concepts/overview/components/) — authoritative description of every component
- [Kubernetes Architecture Explained — Kubernetes.io](https://kubernetes.io/docs/concepts/architecture/) — official architecture guide
- [etcd Documentation](https://etcd.io/docs/) — official etcd reference
- [Kubernetes Dashboard](https://github.com/kubernetes/dashboard) — installation and usage guide
- [Lens Desktop](https://k8slens.dev) — download and documentation
- [k9s](https://k9scli.io) — terminal dashboard for Kubernetes
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) — build a cluster from scratch to understand every component

---

## 🔭 Day 40 Preview: Kubernetes Pods, ReplicaSets & Deployments in Depth

Now that you understand the components that make Kubernetes work, Day 40 goes deep into the objects you interact with most.

You will learn:
- Pod lifecycle states in detail (Pending, Running, Succeeded, Failed, Unknown)
- How ReplicaSets work under Deployments
- Deployment update strategies — RollingUpdate vs Recreate
- How `maxSurge` and `maxUnavailable` control rolling updates
- Writing production-grade Deployment YAML with resource limits, probes, and affinity rules
- Debugging pods — what each error state means and how to fix it

Understanding these objects deeply means you can design, deploy, and debug any Kubernetes workload.

---

> 💬 *"Kubernetes is not complex because it does too much. It's precise because distributed systems demand precision. Every component exists because a specific problem demanded a specific solution."*
>
> — DevOps Learning By Yukta