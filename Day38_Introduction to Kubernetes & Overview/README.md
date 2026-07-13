# Day 38: Introduction to Kubernetes & Overview

> **DevOps Transformation** | Kubernetes & Container Orchestration Series

---

## 📋 Overview

You have mastered Docker — packaging applications into containers. You have mastered Docker Compose — running multi-container applications on a single machine. Now comes the next leap: **Kubernetes**, the system that runs containers at scale, across multiple machines, with self-healing, automatic scaling, and zero-downtime deployments built in.

Kubernetes is not just another tool. It represents a fundamental shift in how we think about deploying and operating software. Understanding it changes how you design systems, how you think about reliability, and how you approach the entire software delivery lifecycle.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| Introduction to Kubernetes | What it is, where it came from, why it matters |
| Why Kubernetes? | The problems it solves — microservices, scaling, self-healing |
| Key Concepts | Cluster, node, pod, deployment, service, namespace |
| Setting Up Your Environment | Minikube, kubectl, local cluster setup |
| Basic kubectl Commands | cluster-info, get nodes, describe, create deployment |
| Task 1 | Your first deployment |
| Task 2 | Scaling and updating deployments |

---

## 🏢 NexusCorp Scenario

> NexusCorp's application is running in Docker containers managed by Docker Compose on a single EC2 instance. Everything works — until that EC2 instance goes down. When it does, the entire application is offline until someone notices and restarts it. Meanwhile, during peak hours, the single instance can't handle the load. After a particularly painful outage on a Friday evening, the engineering team makes a decision: they're moving to Kubernetes.
>
> Today is Day 1 of that journey.

---

## Part 1: Introduction to Kubernetes

### The Evolution to Kubernetes

Understanding Kubernetes requires understanding the problem it was built to solve. The journey from single machines to Kubernetes spans three major eras:

```
Era 1: Physical Servers (1990s–2000s)
────────────────────────────────────────
One application per server.
Expensive, slow to provision, massive waste of resources.
Scaling meant buying more hardware.

Era 2: Virtual Machines (2000s–2010s)
────────────────────────────────────────
Multiple VMs on one physical server.
Better resource utilization.
Still: large, slow to start, full OS per VM.
Scaling: spin up more VMs (minutes).

Era 3: Containers (2013–present)
────────────────────────────────────────
Multiple containers on one host.
Lightweight, fast (seconds), consistent environments.
Docker made containers accessible to everyone.
Problem: how do you manage hundreds of containers
         across dozens of machines?

Era 4: Container Orchestration — Kubernetes (2014–present)
────────────────────────────────────────────────────────────
Manages containers across a cluster of machines.
Handles: scheduling, scaling, self-healing, networking.
Created by Google, open-sourced in 2014.
Donated to CNCF (Cloud Native Computing Foundation) in 2016.
Now: the standard for production container deployments.
```

---

### What Is Kubernetes?

**Kubernetes** (Greek for "helmsman" or "pilot") — abbreviated as **K8s** (K + 8 letters + s) — is an open-source platform designed to **automate the deployment, scaling, and operation of application containers**.

```
Kubernetes in one sentence:
"A system that ensures your containers are always running,
 always healthy, and always appropriately scaled — automatically."

What you tell Kubernetes:
  "I want 3 copies of this web server running at all times,
   accessible on port 80, with 512MB RAM each."

What Kubernetes does:
  → Finds 3 nodes with available capacity
  → Starts the containers
  → Monitors them continuously
  → If one dies: starts a replacement automatically
  → If traffic spikes: starts more instances automatically
  → When you push a new version: rolls it out without downtime
```

---

### Kubernetes Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Control Plane (Master)                  │   │
│  │                                                      │   │
│  │  API Server ─── etcd ─── Scheduler ─── Controller   │   │
│  │  (gateway)    (database)  (placement)  Manager       │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│              ┌────────────┼────────────┐                   │
│              │            │            │                    │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐    │
│  │   Worker Node │ │   Worker Node │ │   Worker Node │    │
│  │               │ │               │ │               │    │
│  │  kubelet      │ │  kubelet      │ │  kubelet      │    │
│  │  kube-proxy   │ │  kube-proxy   │ │  kube-proxy   │    │
│  │               │ │               │ │               │    │
│  │  ┌─────────┐  │ │  ┌─────────┐  │ │  ┌─────────┐  │    │
│  │  │  Pod    │  │ │  │  Pod    │  │ │  │  Pod    │  │    │
│  │  │[app v2] │  │ │  │[app v2] │  │ │  │[app v2] │  │    │
│  │  └─────────┘  │ │  └─────────┘  │ │  └─────────┘  │    │
│  └───────────────┘ └───────────────┘ └───────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Control Plane Components:**

| Component | Role |
|---|---|
| **API Server** | The front door to Kubernetes — all commands go through here |
| **etcd** | The cluster's distributed key-value database — stores all cluster state |
| **Scheduler** | Decides which node a new pod should run on |
| **Controller Manager** | Runs control loops that drive the cluster toward desired state |

**Worker Node Components:**

| Component | Role |
|---|---|
| **kubelet** | Agent on each node — communicates with the API server, manages pods |
| **kube-proxy** | Network proxy — maintains network rules for pod communication |
| **Container Runtime** | Runs containers (containerd, CRI-O) |

---

## Part 2: Why Kubernetes?

### The Problems Docker Compose Can't Solve

```
Docker Compose is great for:           Kubernetes solves:
─────────────────────────────          ─────────────────────────────
✅ Single machine deployments          ✅ Multi-machine cluster management
✅ Development environments            ✅ Production-grade deployments
✅ Simple multi-container apps         ✅ Hundreds of microservices
✅ Manual scaling                      ✅ Automatic scaling
✅ Manual recovery after crashes       ✅ Automatic self-healing
✅ Simple networking                   ✅ Service discovery at scale
❌ Automatic failover                  ✅ Node failure handled automatically
❌ Rolling updates with zero downtime  ✅ Built-in rolling deployments
❌ Cross-node communication            ✅ Flat network across all nodes
❌ Load balancing across nodes         ✅ Built-in load balancing
❌ Resource management                 ✅ CPU and memory limits per container
```

---

### Core Reasons to Use Kubernetes

#### 1. Microservices Architecture Support

Modern applications are not monoliths — they are composed of dozens or hundreds of small, independently deployable services. Kubernetes excels at managing them:

```
NexusCorp Application:
  ├── User Service (3 replicas)
  ├── Auth Service (2 replicas)
  ├── Payment Service (5 replicas)  ← More replicas (critical)
  ├── Notification Service (1 replica)
  ├── API Gateway (3 replicas)
  ├── Frontend (3 replicas)
  └── Analytics Service (2 replicas)

Each service:
  - Deployed independently
  - Scaled independently
  - Updated independently
  - Monitored independently
```

#### 2. Portability

The same Kubernetes manifests (YAML files) work across:
- AWS (EKS — Elastic Kubernetes Service)
- Google Cloud (GKE — Google Kubernetes Engine)
- Azure (AKS — Azure Kubernetes Service)
- Your own servers (kubeadm)
- Your laptop (Minikube, Kind)

Write once, run anywhere.

#### 3. Self-Healing

Kubernetes continuously monitors the cluster and automatically corrects deviations from the desired state:

```
You declare: "I want 3 replicas of nexus-api running"

Scenario: Node 2 crashes (taking Pod 2 with it)
Kubernetes responds (automatically, in ~30 seconds):
  → Detects Pod 2 is gone
  → Selects another node with available capacity
  → Starts a new Pod on Node 3
  → Pod count returns to 3
  → No human intervention required
  → No alert at 2 AM (unless all 3 pods die simultaneously)
```

#### 4. Automated Rollouts and Rollbacks

```
Deploy a new version:
  kubectl set image deployment/nexus-api api=nexus-api:v2

Kubernetes does:
  → Starts 1 new pod with v2
  → Waits for it to pass health check
  → Terminates 1 old pod with v1
  → Repeats until all pods run v2
  → Zero downtime — traffic always served by running pods

Something goes wrong with v2:
  kubectl rollout undo deployment/nexus-api
  → Kubernetes rolls back to v1 automatically
```

#### 5. Automatic Scaling

```
Horizontal Pod Autoscaler (HPA):
  → CPU usage > 70%? Add more pods.
  → CPU usage < 30%? Remove pods.
  → Happens automatically — no manual intervention.

Cluster Autoscaler:
  → Pods can't be scheduled (not enough nodes)?
  → Add a new EC2 instance to the cluster.
  → Nodes sitting idle? Remove them to save cost.
```

---

## Part 3: Key Concepts in Kubernetes

### Cluster

A **cluster** is the complete Kubernetes deployment — the control plane plus all worker nodes working together as one system.

```
Your cluster = Control Plane + Worker Nodes
             = The Kubernetes "computer"
```

---

### Node

A **node** is a machine (physical or virtual) in the Kubernetes cluster. Worker nodes run your application pods.

```bash
# View all nodes in your cluster
kubectl get nodes

# Example output:
# NAME            STATUS   ROLES           AGE   VERSION
# minikube        Ready    control-plane   10m   v1.28.0
# worker-node-1   Ready    <none>          8m    v1.28.0
# worker-node-2   Ready    <none>          8m    v1.28.0
```

---

### Pod

A **pod** is the smallest deployable unit in Kubernetes. A pod wraps one or more containers that share the same network namespace and storage.

```yaml
# A pod with one container
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-pod
  labels:
    app: nexus-api
spec:
  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v2
    ports:
    - containerPort: 5000
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

**Important:** You rarely create pods directly. You create **Deployments** which manage pods for you.

---

### Deployment

A **Deployment** describes the desired state of your application — how many replicas to run, which image to use, and the update strategy. The Deployment controller continuously works to match reality to this desired state.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  labels:
    app: nexus-api
spec:
  replicas: 3                    # I want 3 pods
  selector:
    matchLabels:
      app: nexus-api
  template:
    metadata:
      labels:
        app: nexus-api
    spec:
      containers:
      - name: nexus-api
        image: nexuscorp/nexus-api:v2
        ports:
        - containerPort: 5000
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5
```

---

### Service

A **Service** provides a stable network endpoint to access pods. Since pods are ephemeral (they come and go), Services provide a consistent IP and DNS name that load-balances across all matching pods.

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-service
spec:
  selector:
    app: nexus-api           # Routes to pods with this label
  ports:
  - protocol: TCP
    port: 80                 # Service port
    targetPort: 5000         # Pod port
  type: LoadBalancer         # Expose externally
```

**Service types:**

| Type | Description | Use Case |
|---|---|---|
| `ClusterIP` | Internal cluster IP only (default) | Service-to-service communication |
| `NodePort` | Exposes on each node's IP:port | Development, direct node access |
| `LoadBalancer` | Cloud provider creates a load balancer | Production external access |
| `ExternalName` | Maps to external DNS name | Redirect to external service |

---

### Namespace

**Namespaces** provide logical isolation within a cluster — think of them as virtual clusters. Different teams, environments, or applications can have their own namespace.

```bash
# Create a namespace
kubectl create namespace nexus-production
kubectl create namespace nexus-staging
kubectl create namespace nexus-dev

# List namespaces
kubectl get namespaces

# Default namespaces created by Kubernetes:
# default          - Where resources go if no namespace specified
# kube-system      - Kubernetes system components
# kube-public      - Publicly accessible resources
# kube-node-lease  - Node heartbeat leases
```

---

### ConfigMap and Secret

```yaml
# ConfigMap — non-sensitive configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-config
data:
  APP_ENV: "staging"
  LOG_LEVEL: "INFO"
  MAX_CONNECTIONS: "100"

---
# Secret — sensitive data (base64 encoded)
apiVersion: v1
kind: Secret
metadata:
  name: nexus-secrets
type: Opaque
data:
  DB_PASSWORD: bmV4dXNwYXNz      # base64 encoded
  API_KEY: c2VjcmV0a2V5MTIz      # base64 encoded
```

---

### The Desired State Model

Everything in Kubernetes is declarative. You describe what you want, not how to achieve it.

```
Imperative (Docker):              Declarative (Kubernetes):
"Run container X"                 "I want 3 containers of X running"
"Stop container X"                "I want 0 containers of X running"
"If it crashes, restart it"       (Kubernetes figures out the how)

You manage actions.               Kubernetes manages state.
```

---

## Part 4: Setting Up Your Kubernetes Environment

### Option 1: Minikube (Recommended for Learning)

Minikube runs a single-node Kubernetes cluster on your local machine inside a VM or container. Perfect for learning and development.

```bash
# Install Minikube

# macOS (Homebrew)
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64

# Windows (PowerShell as Administrator)
winget install Kubernetes.minikube

# Verify installation
minikube version
```

```bash
# Start Minikube with Docker driver (most common)
minikube start --driver=docker

# Start with specific resources
minikube start \
    --driver=docker \
    --cpus=2 \
    --memory=4096 \
    --disk-size=20g

# Expected output:
# 😄  minikube v1.32.0 on Ubuntu 22.04
# ✨  Using the docker driver based on user configuration
# 🔥  Creating docker container (CPUs=2, Memory=4096MB) ...
# 🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
# 🔎  Verifying Kubernetes components...
# 🌟  Enabled addons: default-storageclass, storage-provisioner
# 🏄  Done! kubectl is now configured to use "minikube" cluster
```

---

### Option 2: Kind (Kubernetes in Docker)

Kind (Kubernetes IN Docker) runs Kubernetes cluster nodes as Docker containers. Faster to start than Minikube.

```bash
# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind && sudo mv kind /usr/local/bin/

# Create a cluster
kind create cluster --name nexus-cluster

# Create multi-node cluster
cat > kind-config.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --name nexus-cluster --config kind-config.yaml

# Delete a cluster
kind delete cluster --name nexus-cluster
```

---

### Option 3: AWS EKS (Production)

For production deployments, use managed Kubernetes services:

```bash
# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | \
    tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create an EKS cluster
eksctl create cluster \
    --name nexus-cluster \
    --region ap-south-1 \
    --nodegroup-name standard-workers \
    --node-type t3.medium \
    --nodes 2 \
    --nodes-min 1 \
    --nodes-max 4

# This takes 15-20 minutes and creates:
# - EKS control plane
# - 2 EC2 worker nodes
# - VPC, subnets, security groups
# - Automatically configures kubectl
```

---

### Install kubectl — The Kubernetes CLI

```bash
# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# macOS
brew install kubectl

# Windows
winget install Kubernetes.kubectl

# Verify
kubectl version --client
# Client Version: v1.28.3

# View kubectl config (which cluster it's pointing to)
kubectl config get-contexts
kubectl config current-context
```

---

## Part 5: Basic kubectl Commands

### Cluster Information

```bash
# Check cluster connectivity and get cluster details
kubectl cluster-info

# Output:
# Kubernetes control plane is running at https://192.168.49.2:8443
# CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/...

# Get cluster configuration
kubectl config view

# Get API resources available in cluster
kubectl api-resources

# Get API versions
kubectl api-versions
```

---

### Working with Nodes

```bash
# List all nodes
kubectl get nodes

# Detailed node listing
kubectl get nodes -o wide

# Describe a specific node (detailed info)
kubectl describe node minikube

# Watch node status in real time
kubectl get nodes -w
```

---

### The `kubectl get` Command

```bash
# Get all resources in default namespace
kubectl get all

# Get pods
kubectl get pods
kubectl get pods -o wide           # With node and IP info
kubectl get pods -n kube-system    # In specific namespace
kubectl get pods --all-namespaces  # In all namespaces
kubectl get pods -w                # Watch (live updates)

# Get deployments
kubectl get deployments
kubectl get deploy                 # Short form

# Get services
kubectl get services
kubectl get svc                    # Short form

# Get everything
kubectl get all -n default

# Output formats
kubectl get pods -o yaml           # Full YAML output
kubectl get pods -o json           # JSON output
kubectl get pods -o wide           # Wide table with extra columns
kubectl get pods --show-labels     # Show labels
```

---

### The `kubectl describe` Command

```bash
# Describe a deployment (detailed status, events, conditions)
kubectl describe deployment nexus-api

# Describe a pod
kubectl describe pod nexus-api-xyz-123

# Describe a node
kubectl describe node minikube

# Describe a service
kubectl describe service nexus-api-service

# Key things to look for in describe output:
# Events:      Shows what happened recently (errors, scheduling)
# Conditions:  Current state of the resource
# Status:      Is it ready, running, failing?
```

---

### Create and Manage Deployments

```bash
# Create a deployment imperatively (quick testing)
kubectl create deployment nexus-demo --image=nginx:latest

# Create from a YAML file (recommended)
kubectl apply -f deployment.yaml

# Apply all YAML files in a directory
kubectl apply -f ./manifests/

# Scale a deployment
kubectl scale deployment nexus-demo --replicas=3

# Update the image
kubectl set image deployment/nexus-demo nexus-demo=nginx:1.25

# View rollout status
kubectl rollout status deployment/nexus-demo

# View rollout history
kubectl rollout history deployment/nexus-demo

# Rollback to previous version
kubectl rollout undo deployment/nexus-demo

# Rollback to specific revision
kubectl rollout undo deployment/nexus-demo --to-revision=2
```

---

### Expose a Deployment as a Service

```bash
# Expose a deployment
kubectl expose deployment nexus-demo \
    --type=LoadBalancer \
    --port=80 \
    --target-port=80

# For Minikube — get the service URL
minikube service nexus-demo --url
# http://192.168.49.2:32000

# Or tunnel (for LoadBalancer type on Minikube)
minikube tunnel
# Then: http://localhost:80
```

---

### Pod Operations

```bash
# Get logs from a pod
kubectl logs pod-name
kubectl logs pod-name -f                  # Follow logs
kubectl logs pod-name --previous          # Logs from crashed container
kubectl logs pod-name -c container-name   # Specific container in pod

# Execute a command in a pod
kubectl exec -it pod-name -- bash
kubectl exec -it pod-name -- sh           # If bash not available
kubectl exec pod-name -- ls /app          # Non-interactive

# Copy files to/from a pod
kubectl cp pod-name:/app/logs.txt ./logs.txt
kubectl cp ./config.yaml pod-name:/app/config.yaml

# Port forward (access pod without a Service)
kubectl port-forward pod/pod-name 8080:5000
# Now: curl http://localhost:8080

# Port forward a service
kubectl port-forward service/nexus-api-service 8080:80
```

---

### Delete Resources

```bash
# Delete a deployment
kubectl delete deployment nexus-demo

# Delete from file
kubectl delete -f deployment.yaml

# Delete a pod (Deployment will recreate it — this is intentional)
kubectl delete pod pod-name

# Delete all pods in namespace (they will be recreated by deployments)
kubectl delete pods --all

# Delete namespace (and everything in it!)
kubectl delete namespace nexus-staging
```

---

## Part 6: Practical Tasks

### Task 1: Your First Deployment

```bash
# Step 1: Start Minikube
minikube start

# Step 2: Verify cluster is running
kubectl cluster-info
kubectl get nodes

# Step 3: Create your first deployment
kubectl create deployment nexus-hello \
    --image=nginx:latest

# Step 4: Watch the deployment come up
kubectl rollout status deployment/nexus-hello
# Waiting for deployment "nexus-hello" rollout to finish:
# deployment "nexus-hello" successfully rolled out

# Step 5: Verify the pod is running
kubectl get pods
# NAME                           READY   STATUS    RESTARTS   AGE
# nexus-hello-7d4f9c8b6-xk2mp   1/1     Running   0          30s

# Step 6: Expose the deployment
kubectl expose deployment nexus-hello \
    --type=LoadBalancer \
    --port=80

# Step 7: Access the application
minikube service nexus-hello --url
# Open the URL in browser → nginx welcome page

# Step 8: Describe the deployment
kubectl describe deployment nexus-hello

# Step 9: View all resources created
kubectl get all
```

---

### Task 2: Scaling and Updating

```bash
# Step 1: Check current pod count
kubectl get pods
# One pod running

# Step 2: Scale to 3 replicas
kubectl scale deployment nexus-hello --replicas=3

# Step 3: Watch pods spin up
kubectl get pods -w
# nexus-hello-7d4f9c8b6-xk2mp   1/1   Running   0   5m
# nexus-hello-7d4f9c8b6-abc12   1/1   Running   0   10s
# nexus-hello-7d4f9c8b6-def34   1/1   Running   0   10s

# Three instances of your application are now running!
# Kubernetes ensures all 3 stay running

# Step 4: Delete one pod manually (Kubernetes should recreate it)
POD=$(kubectl get pods -o name | head -1)
kubectl delete $POD

# Step 5: Watch Kubernetes self-heal
kubectl get pods -w
# Pod disappears → New pod appears within seconds

# Step 6: Update the image (rolling update)
kubectl set image deployment/nexus-hello nexus-hello=nginx:1.25

# Step 7: Watch rolling update
kubectl rollout status deployment/nexus-hello
# Rolling out: new pods start, old pods terminate one at a time

# Step 8: Verify update
kubectl describe deployment nexus-hello | grep Image
# Image: nginx:1.25

# Step 9: Rollback
kubectl rollout undo deployment/nexus-hello

# Verify rollback
kubectl rollout history deployment/nexus-hello
kubectl describe deployment nexus-hello | grep Image
```

---

### Full YAML-Based Deployment Exercise

```bash
# Create complete deployment using YAML files

# Create namespace
kubectl create namespace nexus-demo

# deployment.yaml
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-webapp
  namespace: nexus-demo
  labels:
    app: nexus-webapp
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nexus-webapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: nexus-webapp
        version: v1
    spec:
      containers:
      - name: nexus-webapp
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "125m"
          limits:
            memory: "128Mi"
            cpu: "250m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
EOF

# service.yaml
cat > service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nexus-webapp-service
  namespace: nexus-demo
spec:
  selector:
    app: nexus-webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
EOF

# Apply both files
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Verify
kubectl get all -n nexus-demo

# Access
minikube service nexus-webapp-service -n nexus-demo --url

# Scale
kubectl scale deployment nexus-webapp \
    -n nexus-demo \
    --replicas=5

# Update image
kubectl set image deployment/nexus-webapp \
    -n nexus-demo \
    nexus-webapp=nginx:latest

# Watch rolling update
kubectl rollout status deployment/nexus-webapp -n nexus-demo

# Clean up
kubectl delete namespace nexus-demo
```

---

## 🔧 Mini-Project: NexusCorp First Kubernetes Deployment

```bash
#!/bin/bash
# nexus_k8s_demo.sh
# Deploy NexusCorp application to Kubernetes

set -e

echo "=============================================="
echo "  NexusCorp Kubernetes Deployment Demo"
echo "=============================================="

# Step 1: Ensure Minikube is running
echo ""
echo "Step 1: Checking cluster..."
minikube status > /dev/null 2>&1 || minikube start --driver=docker
kubectl cluster-info
echo "  ✓ Cluster ready"

# Step 2: Create namespace
echo ""
echo "Step 2: Creating namespace..."
kubectl create namespace nexus-production 2>/dev/null || true
echo "  ✓ Namespace: nexus-production"

# Step 3: Create ConfigMap
echo ""
echo "Step 3: Creating ConfigMap..."
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-config
  namespace: nexus-production
data:
  APP_ENV: "production"
  LOG_LEVEL: "INFO"
  APP_NAME: "NexusCorp API"
EOF
echo "  ✓ ConfigMap created"

# Step 4: Create Deployment
echo ""
echo "Step 4: Creating Deployment..."
kubectl apply -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  namespace: nexus-production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nexus-api
  template:
    metadata:
      labels:
        app: nexus-api
    spec:
      containers:
      - name: nexus-api
        image: nginx:1.25
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: nexus-config
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
EOF
echo "  ✓ Deployment created"

# Step 5: Create Service
echo ""
echo "Step 5: Creating Service..."
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-service
  namespace: nexus-production
spec:
  selector:
    app: nexus-api
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
EOF
echo "  ✓ Service created"

# Step 6: Wait for deployment
echo ""
echo "Step 6: Waiting for deployment..."
kubectl rollout status deployment/nexus-api -n nexus-production
echo "  ✓ Deployment ready"

# Step 7: Show status
echo ""
echo "=============================================="
echo "  Deployment Status"
echo "=============================================="
echo ""
echo "Pods:"
kubectl get pods -n nexus-production -o wide

echo ""
echo "Services:"
kubectl get services -n nexus-production

echo ""
echo "Deployment:"
kubectl get deployment nexus-api -n nexus-production

# Step 8: Scale up
echo ""
echo "Scaling to 3 replicas..."
kubectl scale deployment nexus-api -n nexus-production --replicas=3
kubectl rollout status deployment/nexus-api -n nexus-production

echo ""
kubectl get pods -n nexus-production

echo ""
echo "=============================================="
echo "  Demo complete!"
echo ""
echo "  To access the app:"
echo "  minikube service nexus-api-service -n nexus-production --url"
echo ""
echo "  To clean up:"
echo "  kubectl delete namespace nexus-production"
echo "=============================================="
```

```bash
chmod +x nexus_k8s_demo.sh
./nexus_k8s_demo.sh
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Kubernetes (K8s)** | Open-source platform for automating deployment, scaling, and management of containerized applications |
| **Cluster** | A set of machines (nodes) running Kubernetes — the complete deployment unit |
| **Control Plane** | The master components that make global decisions about the cluster |
| **Worker Node** | A machine in the cluster that runs application workloads (pods) |
| **Pod** | The smallest deployable unit in Kubernetes — wraps one or more containers |
| **Deployment** | A higher-level object that manages pods — ensures desired replica count and handles updates |
| **Service** | A stable network endpoint that load-balances traffic to matching pods |
| **Namespace** | A logical partition within a cluster for isolating resources |
| **ConfigMap** | Kubernetes object for storing non-sensitive configuration data as key-value pairs |
| **Secret** | Kubernetes object for storing sensitive data (passwords, tokens) — base64 encoded |
| **kubectl** | The Kubernetes command-line interface — used to interact with the cluster |
| **kubelet** | Agent running on each worker node — manages pods and communicates with control plane |
| **kube-proxy** | Network proxy running on each node — maintains network rules for pod communication |
| **etcd** | The distributed key-value store that holds all Kubernetes cluster state |
| **API Server** | The front-end for the Kubernetes control plane — all commands go through it |
| **Scheduler** | Control plane component that assigns pods to nodes |
| **Controller Manager** | Runs control loops that keep cluster state matching desired state |
| **Replica** | A copy of a pod — Deployments manage the desired number of replicas |
| **Rolling Update** | Kubernetes update strategy — new pods start before old pods terminate |
| **Rollback** | Reverting a Deployment to a previous version |
| **Readiness Probe** | A test Kubernetes runs to know when a pod is ready to receive traffic |
| **Liveness Probe** | A test Kubernetes runs to know if a pod needs to be restarted |
| **Minikube** | Tool that runs a single-node Kubernetes cluster locally (for development) |
| **Kind** | Kubernetes IN Docker — runs cluster nodes as Docker containers |
| **EKS** | AWS Elastic Kubernetes Service — managed Kubernetes on AWS |
| **YAML manifest** | A YAML file describing a Kubernetes resource (Deployment, Service, etc.) |
| **`kubectl apply`** | Creates or updates a resource from a YAML file (declarative) |
| **`kubectl create`** | Creates a resource imperatively |
| **`kubectl get`** | Lists resources of a specified type |
| **`kubectl describe`** | Shows detailed information about a resource including events |
| **`kubectl logs`** | Shows log output from a pod's container |
| **`kubectl exec`** | Executes a command inside a running container |
| **`kubectl scale`** | Changes the number of replicas for a Deployment |
| **`kubectl rollout`** | Manages rollouts — status, history, undo |
| **Self-healing** | Kubernetes automatically replaces failed pods to maintain desired state |
| **Desired state** | What you declare you want — Kubernetes continuously works to match reality to this |

---

## 📚 Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/home/) — the complete, authoritative Kubernetes reference
- [Minikube Installation Guide](https://minikube.sigs.k8s.io/docs/start/) — get Kubernetes running locally
- [kubectl Cheat Sheet — Official](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) — quick reference for all kubectl commands
- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/) — deep dives into every Kubernetes concept
- [Play with Kubernetes](https://labs.play-with-k8s.com) — free browser-based Kubernetes playground
- [CNCF Cloud Native Interactive Landscape](https://landscape.cncf.io) — the broader ecosystem around Kubernetes
- [Kind Documentation](https://kind.sigs.k8s.io/docs/user/quick-start/) — Kubernetes in Docker

---

## 🔭 Day 39 Preview: Kubernetes Pods, Deployments & Services in Depth

Today you got your first Kubernetes cluster running and deployed your first application. Tomorrow you go deeper into the core objects.

You will learn:
- Pod lifecycle in detail — pending, running, succeeded, failed, unknown
- Deployment strategies — RollingUpdate vs Recreate
- ReplicaSets — what manages your pods under the hood
- Service types in depth — ClusterIP, NodePort, LoadBalancer, Ingress
- Labels and selectors — how Kubernetes connects objects
- Resource requests and limits — preventing one pod from starving others
- Health checks — readiness and liveness probes in practice

The concepts from today become the foundation for everything in Kubernetes — understanding them deeply makes all advanced topics much easier.

---

> 💬 *"Docker asks: 'What should run?' Kubernetes asks: 'What state should the world be in?' — and then makes it so."*
>
> — DevOps Learning By Yukta
