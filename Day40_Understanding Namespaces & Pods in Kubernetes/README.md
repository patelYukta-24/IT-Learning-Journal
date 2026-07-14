# Day 40: Understanding Namespaces & Pods in Kubernetes

> **DevOps Transformation** | Kubernetes & Container Orchestration Series

---

## 📋 Overview

Today you learn the two most foundational concepts in Kubernetes: **Namespaces** and **Pods**. They are the alphabet of Kubernetes — everything else is built from them.

Namespaces solve the organizational problem: how do multiple teams, projects, or environments share the same cluster without stepping on each other? Pods solve the deployment problem: what is the actual unit of work in Kubernetes, and how do you configure it properly?

By the end of today you will have created namespaces with resource quotas, deployed pods with resource limits, experimented with multi-container pods, understood the complete pod lifecycle, and cleaned everything up.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| Namespaces | Definition, use cases, isolation, quotas |
| Working with Namespaces | Create, switch, deploy into, quota |
| Pods | Definition, why not just containers? |
| Pod Lifecycle | States, lifecycle hooks, restart policies |
| Multi-Container Pods | Sidecar, init, ambassador patterns |
| Resource Requests & Limits | CPU and memory management |
| Tasks | Namespace exploration, pod experiments, cleanup |

---

## 🏢 NexusCorp Scenario

> NexusCorp's single Kubernetes cluster now serves three teams: the backend team, the frontend team, and the data engineering team. Without namespaces, their deployments, services, and secrets all live in `default` — a chaotic mix where names conflict, one team's runaway pod consumes all cluster memory, and nobody knows what belongs to whom.
>
> Today you implement the namespace structure that fixes all of this — and understand pods deeply enough to configure them properly for production.

---

## Part 1: Namespaces

### What Is a Namespace?

A Kubernetes **Namespace** is a virtual cluster within a physical cluster. It provides a scope for names and a boundary for resource allocation. Resources in different namespaces are isolated from each other (with some exceptions like nodes and persistent volumes, which are cluster-scoped).

```
Kubernetes Cluster
├── Namespace: default          ← Where resources go if none specified
├── Namespace: kube-system      ← Kubernetes internal components
├── Namespace: kube-public      ← Publicly readable resources
├── Namespace: kube-node-lease  ← Node heartbeat records
│
├── Namespace: nexus-dev        ← Development environment
│   ├── Deployment: nexus-api (dev version)
│   ├── Service: nexus-api-service
│   └── Secret: db-credentials
│
├── Namespace: nexus-staging    ← Staging environment
│   ├── Deployment: nexus-api (staging version)
│   └── Service: nexus-api-service  ← same name, different namespace!
│
└── Namespace: nexus-production ← Production environment
    ├── Deployment: nexus-api (production version)
    └── Service: nexus-api-service
```

**Key namespace properties:**
- Resources within a namespace must have unique names
- The same name can exist in different namespaces
- Namespaces do NOT provide network isolation by default (NetworkPolicies do)
- Some resources are cluster-scoped (not namespaced): Nodes, PersistentVolumes, ClusterRoles, StorageClasses

---

### Default Namespaces

```bash
kubectl get namespaces

# NAME              STATUS   AGE
# default           Active   2d     ← Your resources if no namespace specified
# kube-node-lease   Active   2d     ← Node heartbeat lease objects
# kube-public       Active   2d     ← Readable by all (even unauthenticated)
# kube-system       Active   2d     ← Kubernetes system components
```

| Namespace | Purpose |
|---|---|
| `default` | Default namespace for user resources — avoid using in production |
| `kube-system` | Kubernetes internal components (API server, scheduler, etcd, CoreDNS) |
| `kube-public` | Publicly readable — contains `cluster-info` ConfigMap |
| `kube-node-lease` | Node heartbeat lease objects — improves node failure detection performance |

---

### Working with Namespaces

#### Creating Namespaces

```bash
# Imperative (quick)
kubectl create namespace nexus-dev
kubectl create namespace nexus-staging
kubectl create namespace nexus-production

# Declarative (recommended — version controlled)
cat > namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: nexus-dev
  labels:
    team: backend
    environment: development
    managed-by: devops-team
EOF

kubectl apply -f namespace.yaml
```

#### Switching Between Namespaces

```bash
# Specify namespace per command
kubectl get pods -n nexus-dev
kubectl get pods --namespace nexus-staging
kubectl get pods -n nexus-production

# View resources in ALL namespaces
kubectl get pods --all-namespaces
kubectl get pods -A               # shorthand

# Set default namespace for kubectl (changes context)
kubectl config set-context --current --namespace=nexus-dev

# Verify current namespace
kubectl config view --minify | grep namespace

# Switch back to default
kubectl config set-context --current --namespace=default
```

**Tip — use `kubens` for easy namespace switching:**
```bash
# Install kubectx/kubens (namespace switcher)
# Linux:
curl -L https://github.com/ahmetb/kubectx/releases/latest/download/kubens \
    -o /usr/local/bin/kubens
chmod +x /usr/local/bin/kubens

# List namespaces
kubens

# Switch namespace
kubens nexus-dev

# Switch back
kubens -
```

---

### Resource Quotas

Resource quotas let administrators limit how much of the cluster's resources a namespace can consume. This prevents one team from monopolizing all cluster resources.

```yaml
# resource-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: nexus-dev-quota
  namespace: nexus-dev
spec:
  hard:
    # Compute resources
    requests.cpu: "2"           # Total CPU requested by all pods
    requests.memory: 4Gi        # Total memory requested by all pods
    limits.cpu: "4"             # Total CPU limit for all pods
    limits.memory: 8Gi          # Total memory limit for all pods

    # Object counts
    pods: "20"                  # Maximum number of pods
    services: "10"              # Maximum number of services
    secrets: "20"               # Maximum number of secrets
    configmaps: "20"            # Maximum number of configmaps
    persistentvolumeclaims: "5" # Maximum number of PVCs
```

```bash
kubectl apply -f resource-quota.yaml

# View quota usage
kubectl get resourcequota -n nexus-dev
kubectl describe resourcequota nexus-dev-quota -n nexus-dev

# Output:
# Name:                   nexus-dev-quota
# Namespace:              nexus-dev
# Resource                Used   Hard
# --------                ----   ----
# configmaps              0      20
# limits.cpu              0      4
# limits.memory           0      8Gi
# persistentvolumeclaims  0      5
# pods                    0      20
# requests.cpu            0      2
# requests.memory         0      4Gi
# secrets                 0      20
# services                0      10
```

---

### LimitRange — Default Pod Resource Limits

A LimitRange sets default resource requests and limits for pods in a namespace that don't specify their own.

```yaml
# limit-range.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: nexus-dev-limits
  namespace: nexus-dev
spec:
  limits:
  - type: Pod
    max:
      cpu: "2"
      memory: 2Gi
    min:
      cpu: 100m
      memory: 64Mi
  - type: Container
    default:           # Applied when container doesn't specify limits
      cpu: 500m
      memory: 256Mi
    defaultRequest:    # Applied when container doesn't specify requests
      cpu: 200m
      memory: 128Mi
    max:
      cpu: "2"
      memory: 2Gi
    min:
      cpu: 100m
      memory: 64Mi
```

```bash
kubectl apply -f limit-range.yaml

# View limit ranges
kubectl get limitrange -n nexus-dev
kubectl describe limitrange nexus-dev-limits -n nexus-dev
```

---

### Namespace-Scoped vs Cluster-Scoped Resources

```bash
# Namespaced resources (belong to a namespace)
kubectl api-resources --namespaced=true
# pods, services, deployments, configmaps, secrets,
# replicasets, statefulsets, daemonsets, jobs, cronjobs...

# Cluster-scoped resources (no namespace)
kubectl api-resources --namespaced=false
# nodes, persistentvolumes, clusterroles, clusterrolebindings,
# namespaces, storageclasses, customresourcedefinitions...
```

---

## Part 2: Pods

### What Is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes. It is a wrapper around one or more containers that share:
- The same **network namespace** (same IP address and port space)
- The same **storage** (shared volumes)
- The same **lifecycle** (start and stop together)

```
Pod: nexus-api-pod
  │
  ├── Shared Network (IP: 10.244.1.5)
  │   ├── Container: nexus-api (port 5000)
  │   └── Container: metrics-sidecar (port 9090)
  │       Both containers can reach each other via localhost
  │
  ├── Shared Storage
  │   └── Volume: /shared-logs
  │       Both containers read/write the same directory
  │
  └── Shared Lifecycle
      Both containers start and stop together
```

---

### Why Not Just Containers?

| Directly using containers | Using Pods |
|---|---|
| No shared networking between related containers | Related containers share IP — communicate via localhost |
| No shared storage | Shared volumes work across all containers in the pod |
| Manual orchestration | Kubernetes manages scheduling, health, and restarts |
| No resource management | CPU/memory limits enforced per pod |
| No declarative configuration | YAML manifests define exact desired state |

---

### Pod Lifecycle States

Every pod moves through a series of states from creation to termination:

```
Phase 1: Pending
  → Pod accepted by cluster but containers not yet running
  → Possible reasons:
    - Scheduler finding a suitable node
    - Image being pulled from registry
    - Waiting for volumes to be provisioned

Phase 2: Running
  → Pod assigned to a node
  → All containers created
  → At least one container is running, starting, or restarting

Phase 3: Succeeded
  → All containers in the pod terminated successfully (exit code 0)
  → Pod will not restart
  → Typical for Job pods

Phase 4: Failed
  → All containers terminated
  → At least one container failed (non-zero exit code or killed)

Phase 5: Unknown
  → Pod state cannot be determined
  → Usually because of node communication failure

Container-level states (separate from pod phase):
  Waiting    → Container hasn't started yet (pulling image, init containers running)
  Running    → Container is executing
  Terminated → Container finished (success or failure)
```

```bash
# View pod phases
kubectl get pods -o wide
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP,NODE:.spec.nodeName"

# Detailed pod status
kubectl describe pod pod-name

# Key fields in describe output:
# Status:           Running/Pending/Failed
# IP:               Pod's IP address
# Containers:
#   State:          Container state (Running/Waiting/Terminated)
#   Ready:          Is this container ready?
#   Restart Count:  How many times restarted
# Conditions:
#   PodScheduled:   Has the pod been assigned to a node?
#   Initialized:    Have init containers completed?
#   ContainersReady: Are all containers ready?
#   Ready:          Is the pod ready to serve traffic?
# Events:
#   Pulling image, Pulled, Created, Started...
```

---

### Pod Restart Policies

```yaml
spec:
  restartPolicy: Always    # Default — always restart on failure
  # restartPolicy: OnFailure  # Restart only if container exits with non-zero
  # restartPolicy: Never      # Never restart — run once
```

| Policy | When Container Fails | When Container Succeeds | Use For |
|---|---|---|---|
| `Always` (default) | Restart | Restart | Long-running services |
| `OnFailure` | Restart | Do not restart | Batch jobs |
| `Never` | Do not restart | Do not restart | One-time jobs |

---

### Writing Pod YAML

```yaml
# pod.yaml — A complete, production-grade pod definition
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-pod
  namespace: nexus-dev
  labels:
    app: nexus-api
    version: "1.0"
    team: backend
  annotations:
    description: "NexusCorp API service pod"
    contact: "backend-team@nexuscorp.com"

spec:
  restartPolicy: Always

  # Init containers run to completion BEFORE main containers start
  initContainers:
  - name: init-db-check
    image: busybox:latest
    command: ['sh', '-c',
              'until nslookup nexus-db; do echo waiting for db; sleep 2; done']

  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v1
    imagePullPolicy: IfNotPresent   # Always, Never, IfNotPresent

    ports:
    - name: http
      containerPort: 5000
      protocol: TCP

    # Environment variables
    env:
    - name: APP_ENV
      value: "development"
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: nexus-config
          key: DB_HOST
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: nexus-secrets
          key: DB_PASSWORD

    # Resource management
    resources:
      requests:           # Minimum guaranteed resources
        memory: "128Mi"
        cpu: "250m"
      limits:             # Maximum allowed resources
        memory: "256Mi"
        cpu: "500m"

    # Volume mounts
    volumeMounts:
    - name: config-volume
      mountPath: /app/config
      readOnly: true
    - name: shared-logs
      mountPath: /app/logs

    # Health checks
    readinessProbe:
      httpGet:
        path: /health
        port: 5000
      initialDelaySeconds: 10   # Wait 10s before first check
      periodSeconds: 5           # Check every 5s
      failureThreshold: 3        # Mark unready after 3 failures

    livenessProbe:
      httpGet:
        path: /health
        port: 5000
      initialDelaySeconds: 30   # Wait 30s (longer than readiness)
      periodSeconds: 10
      failureThreshold: 3        # Restart pod after 3 failures

    startupProbe:
      httpGet:
        path: /health
        port: 5000
      failureThreshold: 30       # 30 * 10s = 5 minutes to start
      periodSeconds: 10

  # Volumes
  volumes:
  - name: config-volume
    configMap:
      name: nexus-config
  - name: shared-logs
    emptyDir: {}             # Temporary storage, lost when pod is deleted

  # Scheduling hints
  nodeSelector:
    disk: ssd                # Only run on nodes labeled disk=ssd

  # Tolerations (allow running on tainted nodes)
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "backend"
    effect: "NoSchedule"
```

---

### Resource Requests and Limits Explained

```
CPU Units:
  1 CPU = 1000m (millicores)
  500m = 0.5 CPU
  250m = 0.25 CPU

Memory Units:
  Ki = Kibibytes (1024 bytes)
  Mi = Mebibytes (1024 KiB)
  Gi = Gibibytes (1024 MiB)

requests vs limits:

  requests.cpu = 250m
    → Kubernetes scheduler reserves 0.25 CPU for this pod on the node
    → The pod is GUARANTEED this amount
    → If a node doesn't have 0.25 CPU free, pod won't schedule there

  limits.cpu = 500m
    → The pod can BURST up to 0.5 CPU if available
    → If it tries to use more, it is CPU-throttled (not killed)

  requests.memory = 128Mi
    → The pod is guaranteed 128MB of RAM
    → Used for scheduling decisions

  limits.memory = 256Mi
    → If the pod uses more than 256MB, it is KILLED (OOMKilled)
    → Unlike CPU, memory cannot be throttled — pods are terminated

QoS Classes (determined by requests/limits):
  Guaranteed → requests == limits for all containers
  Burstable  → requests < limits (at least one resource)
  BestEffort → no requests or limits set (evicted first under pressure)
```

```bash
# View resource usage
kubectl top pod pod-name -n nexus-dev

# See what QoS class a pod has
kubectl get pod pod-name -o jsonpath='{.status.qosClass}'
```

---

### Multi-Container Pods

Multiple containers in one pod is a powerful pattern, but use it only when containers are tightly coupled and must share resources.

#### Sidecar Pattern

A helper container that enhances the main container:

```yaml
# sidecar-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-with-sidecar
  namespace: nexus-dev
spec:
  containers:

  # Main container
  - name: nexus-api
    image: nexuscorp/nexus-api:v1
    ports:
    - containerPort: 5000
    volumeMounts:
    - name: shared-logs
      mountPath: /app/logs

  # Sidecar: log shipper
  - name: log-shipper
    image: fluent/fluent-bit:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app    # Reads from same volume as main container
    env:
    - name: LOG_DESTINATION
      value: "elasticsearch:9200"

  volumes:
  - name: shared-logs
    emptyDir: {}

# Pattern: Main app writes logs to /app/logs
#          Log shipper reads from /var/log/app (same volume)
#          and ships to Elasticsearch
```

#### Init Container Pattern

Init containers run and complete BEFORE main containers start — perfect for setup tasks:

```yaml
# init-container-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-with-init
  namespace: nexus-dev
spec:
  initContainers:

  # Init 1: Wait for database to be ready
  - name: wait-for-db
    image: busybox:latest
    command: ['sh', '-c',
              'until nc -z nexus-db-service 5432; do
                 echo "Waiting for database...";
                 sleep 3;
               done;
               echo "Database ready!"']

  # Init 2: Run database migrations
  - name: run-migrations
    image: nexuscorp/nexus-api:v1
    command: ["python3", "manage.py", "migrate"]
    env:
    - name: DB_HOST
      value: "nexus-db-service"

  # Main container only starts after both init containers complete
  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v1
    ports:
    - containerPort: 5000
```

#### Ambassador Pattern

A container that proxies network traffic for the main container:

```yaml
# ambassador-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-with-ambassador
spec:
  containers:

  # Main container talks to localhost:5432
  # (thinks it's talking to a local database)
  - name: nexus-api
    image: nexuscorp/nexus-api:v1
    env:
    - name: DB_HOST
      value: "localhost"   # Always uses localhost
    - name: DB_PORT
      value: "5432"

  # Ambassador container proxies to the real database
  - name: db-ambassador
    image: haproxy:latest
    # Listens on localhost:5432
    # Proxies to real-db.nexuscorp.com:5432
    # Handles connection pooling, TLS, retries
```

---

### Pod Lifecycle Hooks

Kubernetes provides two lifecycle hooks for containers:

```yaml
containers:
- name: nexus-api
  image: nexuscorp/nexus-api:v1
  lifecycle:
    postStart:                    # Runs immediately AFTER container starts
      exec:
        command: ["/bin/sh", "-c",
                  "echo 'Container started' >> /var/log/lifecycle.log"]

    preStop:                      # Runs immediately BEFORE container is terminated
      exec:
        command: ["/bin/sh", "-c",
                  "nginx -s quit; while killall -0 nginx; do sleep 1; done"]
      # Use preStop for graceful shutdown:
      # - Drain connections
      # - Finish in-flight requests
      # - Flush write buffers
      # - Close database connections
```

**Graceful termination sequence:**
```
1. Pod receives SIGTERM signal (or preStop hook runs)
2. Pod removed from Service endpoints (no new traffic)
3. terminationGracePeriodSeconds countdown starts (default: 30s)
4. App has 30 seconds to finish in-flight requests
5. After 30s: SIGKILL sent (forced termination)
```

```yaml
spec:
  terminationGracePeriodSeconds: 60  # Give 60s for graceful shutdown
```

---

## Part 3: Practical Tasks

### Task 1: Exploring Namespaces

```bash
# Step 1: Create three namespaces for NexusCorp teams
kubectl create namespace nexus-dev
kubectl create namespace nexus-staging
kubectl create namespace nexus-production

# Step 2: Add labels and verify
kubectl label namespace nexus-dev environment=development team=backend
kubectl label namespace nexus-staging environment=staging team=backend
kubectl label namespace nexus-production environment=production team=backend

kubectl get namespaces --show-labels

# Step 3: Deploy the same application name in different namespaces
kubectl create deployment nexus-api \
    -n nexus-dev \
    --image=nginx:latest \
    --replicas=1

kubectl create deployment nexus-api \
    -n nexus-staging \
    --image=nginx:1.25 \
    --replicas=2

# Step 4: Same name, different namespaces — both work!
kubectl get deployments -n nexus-dev
kubectl get deployments -n nexus-staging
# Both show: nexus-api

# Step 5: Apply a ResourceQuota to dev namespace
cat > dev-quota.yaml << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: nexus-dev-quota
  namespace: nexus-dev
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    services: "5"
    secrets: "10"
    configmaps: "10"
EOF

kubectl apply -f dev-quota.yaml
kubectl describe resourcequota nexus-dev-quota -n nexus-dev

# Step 6: Try to exceed the quota
for i in {1..12}; do
    kubectl run test-pod-$i \
        --image=nginx \
        -n nexus-dev \
        --restart=Never \
        --requests='cpu=100m,memory=100Mi' \
        --limits='cpu=200m,memory=200Mi' 2>&1 | tail -1
done
# When you hit 10 pods: "forbidden: exceeded quota"

# Step 7: Switch default namespace context
kubectl config set-context --current --namespace=nexus-dev
kubectl get pods    # Now defaults to nexus-dev

# Step 8: Reset to default
kubectl config set-context --current --namespace=default
```

---

### Task 2: Playing with Pods

#### Part A: Single Container Pod

```bash
# Create a simple pod YAML
cat > simple-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nexus-simple-pod
  namespace: nexus-dev
  labels:
    app: nexus-demo
    type: simple
spec:
  containers:
  - name: nexus-app
    image: nginx:1.25
    ports:
    - containerPort: 80
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
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 10
EOF

kubectl apply -f simple-pod.yaml

# Watch it start
kubectl get pod nexus-simple-pod -n nexus-dev -w

# Inspect the pod
kubectl describe pod nexus-simple-pod -n nexus-dev

# Access its logs
kubectl logs nexus-simple-pod -n nexus-dev

# Execute a command inside
kubectl exec -it nexus-simple-pod -n nexus-dev -- bash

# Port-forward to test locally
kubectl port-forward pod/nexus-simple-pod 8080:80 -n nexus-dev &
curl http://localhost:8080
```

---

#### Part B: Multi-Container Pod

```bash
cat > multi-container-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nexus-multi-pod
  namespace: nexus-dev
  labels:
    app: nexus-multi
spec:
  containers:

  # Main application container
  - name: nexus-app
    image: nginx:1.25
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "250m"
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx

  # Sidecar: log watcher
  - name: log-watcher
    image: busybox:latest
    command: ['sh', '-c',
              'while true; do
                 if [ -f /logs/access.log ]; then
                   echo "[$(date)] Log lines: $(wc -l < /logs/access.log)";
                 else
                   echo "[$(date)] Waiting for logs...";
                 fi;
                 sleep 10;
               done']
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
    volumeMounts:
    - name: shared-logs
      mountPath: /logs

  volumes:
  - name: shared-logs
    emptyDir: {}
EOF

kubectl apply -f multi-container-pod.yaml

# Watch both containers start
kubectl get pod nexus-multi-pod -n nexus-dev -w

# View logs from specific container
kubectl logs nexus-multi-pod -n nexus-dev -c nexus-app
kubectl logs nexus-multi-pod -n nexus-dev -c log-watcher

# Follow log-watcher's output
kubectl logs nexus-multi-pod -n nexus-dev -c log-watcher -f

# Exec into a specific container
kubectl exec -it nexus-multi-pod -n nexus-dev -c nexus-app -- bash

# Both containers can reach each other via localhost
kubectl exec -it nexus-multi-pod -n nexus-dev -c log-watcher -- \
    wget -q -O- http://localhost:80 | head -5
```

---

#### Part C: Pod with Resource Limits and Quota Impact

```bash
cat > resource-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nexus-resource-pod
  namespace: nexus-dev
spec:
  containers:
  - name: nexus-app
    image: nginx:1.25

    # Without requests, pod is BestEffort class (evicted first)
    # With requests < limits, pod is Burstable class
    # With requests == limits, pod is Guaranteed class

    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
EOF

kubectl apply -f resource-pod.yaml

# Check QoS class
kubectl get pod nexus-resource-pod -n nexus-dev \
    -o jsonpath='{.status.qosClass}'
# Output: Burstable

# Check quota usage now
kubectl describe resourcequota nexus-dev-quota -n nexus-dev
# Resources shows the pod is consuming from the quota

# Try creating a pod that exceeds the namespace quota
cat > oversized-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nexus-oversized-pod
  namespace: nexus-dev
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "10Gi"    # Exceeds namespace quota of 4Gi
        cpu: "8"          # Exceeds namespace quota of 2 CPUs
      limits:
        memory: "10Gi"
        cpu: "8"
EOF

kubectl apply -f oversized-pod.yaml
# Error: exceeded quota: requests.cpu, requests.memory

echo "Quota enforcement working correctly!"
```

---

### Task 3: Complete Cleanup

```bash
# Delete individual pods
kubectl delete pod nexus-simple-pod -n nexus-dev
kubectl delete pod nexus-multi-pod -n nexus-dev
kubectl delete pod nexus-resource-pod -n nexus-dev

# Delete from files
kubectl delete -f simple-pod.yaml
kubectl delete -f multi-container-pod.yaml

# Delete all pods in a namespace
kubectl delete pods --all -n nexus-dev

# Delete deployments
kubectl delete deployment nexus-api -n nexus-dev
kubectl delete deployment nexus-api -n nexus-staging

# Delete the resource quota
kubectl delete resourcequota nexus-dev-quota -n nexus-dev

# Delete namespaces (and ALL resources inside them)
kubectl delete namespace nexus-dev
kubectl delete namespace nexus-staging
kubectl delete namespace nexus-production

# Verify everything is cleaned up
kubectl get all -A | grep nexus
kubectl get namespaces

# Reset kubectl context
kubectl config set-context --current --namespace=default
```

---

## 🔧 Mini-Project: NexusCorp Namespace Architecture

Build the complete NexusCorp namespace structure with quotas, LimitRanges, and a sample deployment in each tier:

```bash
#!/bin/bash
# nexus_namespace_setup.sh
# Set up NexusCorp's complete namespace architecture

set -e
echo "=============================================="
echo "  NexusCorp Namespace Architecture Setup"
echo "=============================================="

# ── Create namespaces ───────────────────────────────────────────
for ns in nexus-dev nexus-staging nexus-production; do
    kubectl create namespace $ns 2>/dev/null || true
    kubectl label namespace $ns \
        project=nexuscorp \
        managed-by=devops-team \
        --overwrite
    echo "  ✓ Namespace: $ns"
done

# ── Apply ResourceQuotas ────────────────────────────────────────
echo ""
echo "Applying Resource Quotas..."

kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: nexus-dev
spec:
  hard:
    pods: "20"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: staging-quota
  namespace: nexus-staging
spec:
  hard:
    pods: "30"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: nexus-production
spec:
  hard:
    pods: "50"
    requests.cpu: "8"
    requests.memory: 16Gi
    limits.cpu: "16"
    limits.memory: 32Gi
EOF
echo "  ✓ Resource quotas applied"

# ── Apply LimitRanges ───────────────────────────────────────────
echo ""
echo "Applying LimitRanges..."

for ns in nexus-dev nexus-staging nexus-production; do
    kubectl apply -n $ns -f - << 'EOF'
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - type: Container
    default:
      cpu: 300m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    max:
      cpu: "2"
      memory: 2Gi
    min:
      cpu: 50m
      memory: 32Mi
EOF
done
echo "  ✓ LimitRanges applied to all namespaces"

# ── Deploy sample application in each namespace ─────────────────
echo ""
echo "Deploying sample applications..."

for ns in nexus-dev nexus-staging nexus-production; do
    cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  namespace: ${ns}
spec:
  replicas: 1
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
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "300m"
EOF
    echo "  ✓ Deployed nexus-api in ${ns}"
done

# ── Summary ─────────────────────────────────────────────────────
echo ""
echo "=============================================="
echo "  Architecture Summary"
echo "=============================================="
echo ""
echo "Namespaces:"
kubectl get namespaces -l project=nexuscorp

echo ""
echo "Resource Quotas:"
kubectl get resourcequota -A | grep nexus

echo ""
echo "Deployments per namespace:"
for ns in nexus-dev nexus-staging nexus-production; do
    PODS=$(kubectl get pods -n $ns --no-headers 2>/dev/null | wc -l)
    echo "  $ns: $PODS pod(s)"
done

echo ""
echo "To clean up everything:"
echo "  kubectl delete namespace nexus-dev nexus-staging nexus-production"
echo "=============================================="
```

```bash
chmod +x nexus_namespace_setup.sh
./nexus_namespace_setup.sh
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Namespace** | A virtual cluster within a Kubernetes cluster — provides scope for names and resource isolation |
| **ResourceQuota** | Sets hard limits on the total resources a namespace can consume |
| **LimitRange** | Sets default and maximum resource requests/limits for containers in a namespace |
| **Pod** | The smallest deployable unit in Kubernetes — wraps one or more containers |
| **Pod Phase** | The high-level state of a pod: Pending, Running, Succeeded, Failed, Unknown |
| **Container State** | The state of an individual container: Waiting, Running, Terminated |
| **Restart Policy** | Determines when a container should be restarted: Always, OnFailure, Never |
| **requests** | The minimum resources guaranteed to a container — used for scheduling |
| **limits** | The maximum resources a container can use — enforced at runtime |
| **CPU throttling** | Slowing down a container that exceeds its CPU limit (not killed) |
| **OOMKilled** | Out of Memory Kill — container terminated because it exceeded memory limit |
| **QoS Class** | Quality of Service class assigned based on resource configuration: Guaranteed, Burstable, BestEffort |
| **Guaranteed** | QoS class where requests == limits for all containers |
| **Burstable** | QoS class where at least one resource has requests < limits |
| **BestEffort** | QoS class where no requests or limits are set — evicted first |
| **Init Container** | A container that runs and completes before the main containers start |
| **Sidecar Container** | A helper container that runs alongside the main container in a pod |
| **Ambassador Container** | A container that proxies network traffic to/from the main container |
| **Shared Network Namespace** | All containers in a pod share the same IP and port space |
| **emptyDir** | A temporary volume that exists as long as the pod — deleted when pod is removed |
| **postStart hook** | A lifecycle hook that runs immediately after a container starts |
| **preStop hook** | A lifecycle hook that runs before a container is terminated |
| **terminationGracePeriodSeconds** | Time Kubernetes waits after SIGTERM before sending SIGKILL |
| **Readiness Probe** | Test to determine if a container is ready to receive traffic |
| **Liveness Probe** | Test to determine if a container needs to be restarted |
| **Startup Probe** | Test used during startup — disables liveness probe until it passes |
| **`kubectl top`** | Shows actual resource usage (requires metrics-server addon) |
| **`kubens`** | Community tool for quickly switching between Kubernetes namespaces |
| **nodeSelector** | Constrains a pod to run only on nodes with matching labels |
| **tolerations** | Allows a pod to run on nodes with matching taints |
| **`-A` flag** | Short for `--all-namespaces` in kubectl commands |

---

## 📚 Resources

- [Kubernetes Namespaces — Official Docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) — complete namespace reference
- [Pod Lifecycle — Official Docs](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) — all lifecycle states and hooks
- [Resource Management for Pods — Official Docs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) — requests, limits, and QoS
- [Init Containers — Official Docs](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) — patterns and examples
- [Resource Quotas — Official Docs](https://kubernetes.io/docs/concepts/policy/resource-quotas/) — quota configuration reference
- [kubectx/kubens](https://github.com/ahmetb/kubectx) — fast namespace and context switcher

---

## 🔭 Day 41 Preview: Kubernetes Deployments, ReplicaSets & Services

Today you learned about the atomic units — Namespaces (for organization) and Pods (for running containers). Tomorrow you learn the objects that manage pods at scale.

You will learn:
- ReplicaSets — how Kubernetes maintains a desired number of pod replicas
- Deployments — the higher-level object that manages ReplicaSets and enables rolling updates
- Rolling update strategy — `maxSurge` and `maxUnavailable` in practice
- Services in depth — ClusterIP, NodePort, LoadBalancer, ExternalName
- Labels and selectors — the glue that connects Deployments to ReplicaSets to Pods to Services
- Writing production-grade Deployment YAML with all best practices

This is the layer most Kubernetes users spend most of their time in — understanding it deeply means you can manage any production workload.

---

> 💬 *"Namespaces are the rooms in your house. Pods are the furniture. You need both to make the house livable — organization and function working together."*
>
> — DevOps Learning By Yukta
