# Day 41: Deployments, Services & Networking in Kubernetes

> **DevOps Transformation** | Kubernetes & Container Orchestration Series

---

## 📋 Overview

Yesterday you learned about Namespaces and Pods — the organizing and atomic units of Kubernetes. Today you learn the objects that make Kubernetes genuinely powerful for production: **Deployments** (managing pods at scale), **Services** (stable network endpoints), and **Networking** (how containers communicate).

These three concepts are what you interact with every single day as a Kubernetes operator. A Deployment ensures your app is always running. A Service ensures it's always reachable. Networking ensures the right parts can talk to each other — and the wrong parts cannot.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| Deployments | YAML manifests, scaling, rolling updates, rollbacks |
| ReplicaSets | How Deployments manage pod counts under the hood |
| Services | ClusterIP, NodePort, LoadBalancer, ExternalName |
| Pod Networking | Pod IPs, flat network model, no NAT |
| Service Networking | Stable IPs, DNS, load balancing |
| Network Policies | Control which pods can talk to which |
| YAML Fundamentals | The syntax you need before writing manifests |
| Tasks | Create deployment, expose it, experiment with network policies |

---

## 🏢 NexusCorp Scenario

> NexusCorp needs to deploy their API service with three replicas, update it without downtime, expose it to internal services and the public internet separately, and ensure the database pod can only be reached by the API pod — not by the frontend. Every one of these requirements maps to exactly one Kubernetes concept covered today.

---

## Part 1: YAML Fundamentals for Kubernetes

Before writing Kubernetes manifests, you need to be comfortable with YAML. Every Kubernetes object is defined in YAML.

### YAML Basics

```yaml
# YAML syntax fundamentals

# Scalars (single values)
name: nexus-api
port: 5000
enabled: true
ratio: 0.95

# Strings (quotes optional unless special characters)
message: "Hello, World!"
path: /app/config
label: 'my-app'

# Lists (sequences)
services:
  - nginx
  - redis
  - postgresql

# Or inline list
ports: [80, 443, 8080]

# Maps (key-value pairs)
resources:
  cpu: "500m"
  memory: "256Mi"

# Nested structures
spec:
  containers:
  - name: nexus-api           # List item (- indicates list)
    image: nginx:1.25
    ports:
    - containerPort: 80       # Nested list item

# Multi-line strings
command: |                    # | preserves newlines
  #!/bin/bash
  echo "Starting app"
  python3 app.py

description: >               # > folds newlines into spaces
  This is a long description
  that spans multiple lines
  but becomes one line.

# Comments
# This is a comment

# Multiple documents in one file (separated by ---)
---
apiVersion: v1
kind: Service
---
apiVersion: apps/v1
kind: Deployment
```

### The Standard Kubernetes Manifest Structure

Every Kubernetes object has four top-level fields:

```yaml
apiVersion: apps/v1      # Which API group and version
kind: Deployment         # What type of object
metadata:                # Information about the object
  name: nexus-api
  namespace: nexus-prod
  labels:
    app: nexus-api
spec:                    # The desired state of the object
  # ... (different for each kind)
```

| Field | Purpose |
|---|---|
| `apiVersion` | API group and version (`v1`, `apps/v1`, `networking.k8s.io/v1`) |
| `kind` | Object type (`Pod`, `Deployment`, `Service`, `ConfigMap`) |
| `metadata` | Name, namespace, labels, annotations |
| `spec` | The desired state — what you want Kubernetes to create |

```bash
# Find the correct apiVersion for any resource
kubectl api-resources | grep -i deployment
# NAME        SHORTNAMES  APIVERSION  NAMESPACED  KIND
# deployments deploy      apps/v1     true        Deployment

kubectl explain deployment           # Full field documentation
kubectl explain deployment.spec      # Spec fields
kubectl explain deployment.spec.template.spec.containers
```

---

## Part 2: Deployments

### What Is a Deployment?

A Deployment is a Kubernetes object that manages a set of identical pods, ensuring:
- The desired number of replicas are always running
- Rolling updates happen without downtime
- Rollbacks are possible if an update fails
- Pods are automatically replaced if they crash

```
Deployment (desired state: 3 replicas of nexus-api:v2)
    │
    └── ReplicaSet (nexus-api-7d4f9c8b6)
            ├── Pod: nexus-api-7d4f9c8b6-xk2mp  (Running)
            ├── Pod: nexus-api-7d4f9c8b6-abc12   (Running)
            └── Pod: nexus-api-7d4f9c8b6-def34   (Running)
```

The Deployment manages a ReplicaSet. The ReplicaSet manages Pods. You interact with the Deployment — it handles the rest.

---

### Complete Deployment YAML

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  namespace: nexus-production
  labels:
    app: nexus-api
    version: "2.0"
    team: backend
  annotations:
    kubernetes.io/change-cause: "Update to v2.0 with new payment module"

spec:
  # Number of pod replicas to maintain
  replicas: 3

  # How to identify pods this Deployment manages
  selector:
    matchLabels:
      app: nexus-api    # Must match template.metadata.labels

  # Rolling update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1         # At most 1 extra pod during update
      maxUnavailable: 0   # Never have fewer than desired replicas available

  # Pod template (defines the pods to create)
  template:
    metadata:
      labels:
        app: nexus-api    # Must match spec.selector.matchLabels
        version: "2.0"

    spec:
      # How long to wait for pod to terminate gracefully
      terminationGracePeriodSeconds: 30

      containers:
      - name: nexus-api
        image: nexuscorp/nexus-api:v2
        imagePullPolicy: IfNotPresent   # Always | IfNotPresent | Never

        ports:
        - name: http
          containerPort: 5000

        # Environment variables
        env:
        - name: APP_ENV
          value: "production"
        - name: LOG_LEVEL
          value: "INFO"
        envFrom:
        - configMapRef:
            name: nexus-config

        # Resource management
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"

        # Health checks
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5
          successThreshold: 1
          failureThreshold: 3

        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3

        # Graceful shutdown
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 5"]
```

---

### Deployment Strategies

#### Rolling Update (Default)

Updates pods gradually — new pods start before old pods stop. Zero downtime.

```
maxSurge: 1, maxUnavailable: 0, desired: 3

Before update:  [v1][v1][v1]  (3 pods, 3 available)

Step 1:         [v1][v1][v1][v2]  (surge: 4 pods, start v2)
                Wait for v2 to pass readiness probe

Step 2:         [v1][v1][v2]  (remove 1 v1 pod)
                [v1][v1][v2][v2]  (start another v2)

Step 3:         [v1][v2][v2]  (remove 1 v1)
                [v1][v2][v2][v2]  (start another v2)

Step 4:         [v2][v2][v2]  (remove last v1)

After update:   [v2][v2][v2]  (3 pods, all v2, all healthy)
```

#### Recreate Strategy

Terminates all old pods, then starts new ones. Has downtime — use only when you cannot run two versions simultaneously.

```yaml
strategy:
  type: Recreate    # Kills all pods first, then starts new ones
```

---

### Scaling Deployments

```bash
# Scale imperatively
kubectl scale deployment nexus-api --replicas=5 -n nexus-production

# Scale declaratively (edit YAML and apply)
# Change replicas: 3 to replicas: 5 in deployment.yaml
kubectl apply -f deployment.yaml

# Autoscaling (requires metrics-server)
kubectl autoscale deployment nexus-api \
    --min=2 \
    --max=10 \
    --cpu-percent=70

# View the HorizontalPodAutoscaler
kubectl get hpa -n nexus-production
```

---

### Rolling Updates and Rollbacks

```bash
# Update the image (triggers rolling update)
kubectl set image deployment/nexus-api \
    nexus-api=nexuscorp/nexus-api:v3 \
    -n nexus-production \
    --record    # Records the command in history

# Watch the rolling update
kubectl rollout status deployment/nexus-api -n nexus-production
# Waiting for deployment "nexus-api" rollout to finish:
# 1 out of 3 new replicas have been updated...
# 2 out of 3 new replicas have been updated...
# 3 out of 3 new replicas have been updated...
# deployment "nexus-api" successfully rolled out

# View rollout history
kubectl rollout history deployment/nexus-api -n nexus-production
# REVISION  CHANGE-CAUSE
# 1         kubectl create --image=nexuscorp/nexus-api:v1
# 2         kubectl set image --image=nexuscorp/nexus-api:v2
# 3         kubectl set image --image=nexuscorp/nexus-api:v3

# View details of a specific revision
kubectl rollout history deployment/nexus-api \
    --revision=2 \
    -n nexus-production

# Rollback to previous version
kubectl rollout undo deployment/nexus-api -n nexus-production

# Rollback to a specific revision
kubectl rollout undo deployment/nexus-api \
    --to-revision=1 \
    -n nexus-production

# Pause a rolling update (to inspect mid-rollout)
kubectl rollout pause deployment/nexus-api -n nexus-production

# Resume a paused rollout
kubectl rollout resume deployment/nexus-api -n nexus-production
```

---

### Deployment Commands Reference

```bash
# Apply (create or update)
kubectl apply -f deployment.yaml

# Get deployments
kubectl get deployments -n nexus-production
kubectl get deploy                          # Short form

# Describe a deployment
kubectl describe deployment nexus-api -n nexus-production

# Delete a deployment (and all its pods)
kubectl delete deployment nexus-api

# Edit a deployment in-place
kubectl edit deployment nexus-api -n nexus-production

# Get deployment YAML
kubectl get deployment nexus-api -o yaml -n nexus-production
```

---

## Part 3: Services

### What Is a Service?

Pods are ephemeral — they can be created, destroyed, and replaced at any time. Each time a pod is replaced, it gets a new IP address. If another component tries to connect to a pod by its IP, the connection breaks when the pod is replaced.

A **Service** provides a **stable network endpoint** — a fixed IP and DNS name that always routes traffic to healthy pods, regardless of which pod IPs have changed.

```
Without Service:                    With Service:
─────────────────                   ────────────────────────────────
Frontend connects to:               Frontend connects to:
  10.244.1.5 (Pod 1)               nexus-api-service (stable DNS)
  Pod 1 dies, new IP: 10.244.2.3     │
  Frontend connection breaks!         └── Service automatically routes to
                                          whichever pods are healthy
                                          10.244.1.5 (Pod 1) ← alive
                                          10.244.2.3 (Pod 2) ← alive
                                          10.244.3.1 (Pod 3) ← alive
```

---

### How Services Work

Services use **labels and selectors** to identify their target pods. Any pod with matching labels receives traffic from the Service.

```
Service (selector: app=nexus-api)
    │
    ├── Routes to pods with label: app=nexus-api
    ├── Maintains a list of healthy pod IPs (Endpoints)
    └── Load balances traffic across all healthy pods

Endpoints object (auto-managed):
  nexus-api-service:
    - 10.244.1.5:5000  (Pod 1)
    - 10.244.2.3:5000  (Pod 2)
    - 10.244.3.1:5000  (Pod 3)
```

---

### Service Types

#### 1. ClusterIP (Default)

Exposes the service on an internal IP only. Only accessible from within the cluster.

```yaml
# clusterip-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-internal
  namespace: nexus-production
spec:
  type: ClusterIP          # Default — can omit this line
  selector:
    app: nexus-api         # Routes to pods with this label
  ports:
  - name: http
    protocol: TCP
    port: 80               # Port the Service listens on
    targetPort: 5000       # Port on the pod to forward to
```

```bash
kubectl apply -f clusterip-service.yaml

# Get the ClusterIP
kubectl get service nexus-api-internal -n nexus-production
# NAME                 TYPE        CLUSTER-IP     PORT(S)   AGE
# nexus-api-internal   ClusterIP   10.96.123.45   80/TCP    1m

# Access from another pod (inside cluster)
kubectl exec -it some-other-pod -- curl http://nexus-api-internal:80
# OR by DNS: curl http://nexus-api-internal.nexus-production.svc.cluster.local
```

**DNS format:** `SERVICE-NAME.NAMESPACE.svc.cluster.local`
```
nexus-api-internal.nexus-production.svc.cluster.local
      │                  │             │       │
      │                  │             │       └── Kubernetes cluster domain
      │                  │             └── Always "svc"
      │                  └── Namespace
      └── Service name
```

---

#### 2. NodePort

Exposes the service on each node's IP at a static port. Accessible from outside the cluster.

```yaml
# nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-nodeport
  namespace: nexus-production
spec:
  type: NodePort
  selector:
    app: nexus-api
  ports:
  - name: http
    protocol: TCP
    port: 80               # ClusterIP port (internal)
    targetPort: 5000       # Pod port
    nodePort: 30080        # External port on each node (30000-32767)
                           # Omit to let Kubernetes choose
```

```
External client
      │
      │ http://NODE-IP:30080
      ▼
Any Worker Node (port 30080 open)
      │
      │ Routes to Service ClusterIP
      ▼
Service (ClusterIP: 10.96.123.45:80)
      │
      │ Load balances to healthy pods
      ▼
Pod (port 5000)
```

---

#### 3. LoadBalancer

Creates an external load balancer via the cloud provider (AWS ELB, GCP GLB, Azure LB). The most common way to expose services publicly in cloud deployments.

```yaml
# loadbalancer-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-public
  namespace: nexus-production
  annotations:
    # AWS-specific annotations (optional)
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: nexus-api
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 5000
  - name: https
    protocol: TCP
    port: 443
    targetPort: 5000
```

```bash
kubectl apply -f loadbalancer-service.yaml

# Watch for external IP assignment (takes 1-2 minutes on cloud)
kubectl get service nexus-api-public -n nexus-production -w
# NAME              TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)
# nexus-api-public  LoadBalancer   10.96.45.12    <pending>        80:30080/TCP
# nexus-api-public  LoadBalancer   10.96.45.12    18.234.56.78     80:30080/TCP

# Access externally
curl http://18.234.56.78/health
```

**On Minikube — simulate LoadBalancer:**
```bash
minikube tunnel    # Run in a separate terminal
# Then EXTERNAL-IP will be assigned (localhost)
```

---

#### 4. ExternalName

Maps a service to an external DNS name. No proxying — just DNS aliasing.

```yaml
# externalname-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nexus-external-db
  namespace: nexus-production
spec:
  type: ExternalName
  externalName: db.nexuscorp.com   # External DNS name
  # Pods use: nexus-external-db → resolves to db.nexuscorp.com
```

---

### Service Type Comparison

| Type | Cluster-Internal? | External Access? | Use Case |
|---|---|---|---|
| `ClusterIP` | ✅ Yes | ❌ No | Service-to-service communication |
| `NodePort` | ✅ Yes | ✅ Via node IP | Development, testing, bare-metal |
| `LoadBalancer` | ✅ Yes | ✅ Via cloud LB | Production external access |
| `ExternalName` | ✅ DNS only | ❌ No | Alias for external services |

---

### Endpoints and EndpointSlices

```bash
# View the endpoints (pod IPs) behind a service
kubectl get endpoints nexus-api-internal -n nexus-production

# Output:
# NAME                 ENDPOINTS                                          AGE
# nexus-api-internal   10.244.1.5:5000,10.244.2.3:5000,10.244.3.1:5000  5m

# When a pod fails readiness probe:
# Its IP is REMOVED from endpoints → no traffic sent to it
# When it recovers:
# Its IP is ADDED back → traffic resumes
```

---

## Part 4: Kubernetes Networking

### The Kubernetes Networking Model

Kubernetes enforces a specific networking model:

```
1. Every pod gets its own unique IP address
   → No NAT needed for pod-to-pod communication
   → Pod on Node 1 can reach pod on Node 2 directly by IP

2. All pods can communicate with all other pods
   → Flat network model (unless NetworkPolicies restrict it)
   → No firewall between pods by default

3. Agents on a node can communicate with all pods on that node

4. Services provide stable IPs for groups of pods
   → Service IP is stable even as pod IPs change
   → Implemented via iptables/IPVS rules on each node (kube-proxy)
```

---

### Pod Networking Deep Dive

```
Node 1 (172.16.0.1)          Node 2 (172.16.0.2)
─────────────────────         ─────────────────────
Pod A: 10.244.1.5             Pod C: 10.244.2.3
Pod B: 10.244.1.6             Pod D: 10.244.2.4
  │                              │
  └── cbr0 (bridge)              └── cbr0 (bridge)
       │                               │
  ─────┼────────────────────────────── │ ────
       │         Overlay Network        │
       │    (Flannel/Calico/Weave)      │
  ─────┼────────────────────────────── │ ────

Pod A (10.244.1.5) → Pod D (10.244.2.4):
  → Pod A sends packet to 10.244.2.4
  → Node's routing: 10.244.2.x is on Node 2
  → Packet encapsulated and sent to Node 2
  → Node 2 decapsulates and delivers to Pod D
  → No NAT — Pod A's IP is preserved
```

**CNI Plugins** (Container Network Interface — implements this model):

| Plugin | Description | Features |
|---|---|---|
| **Calico** | Most popular — network policies, BGP routing | Network policies, performance |
| **Flannel** | Simple overlay network | Easy setup, basic features |
| **Weave** | Mesh overlay network | Encryption built-in |
| **Cilium** | eBPF-based — very high performance | Advanced observability |
| **AWS VPC CNI** | Native AWS networking — pods get VPC IPs | AWS integration |

---

### Service Networking

```
Service nexus-api-internal (ClusterIP: 10.96.123.45)

When a pod sends traffic to 10.96.123.45:80:
  1. kube-proxy's iptables rules intercept the packet
  2. Rules randomly select one of the healthy pod IPs
  3. Packet is destination-NAT'd to the pod IP
  4. Pod responds → source IP translated back to service IP

DNS in Kubernetes (CoreDNS):
  Every service gets a DNS entry:
  nexus-api-internal.nexus-production.svc.cluster.local → 10.96.123.45

  Short forms (within same namespace):
  nexus-api-internal → 10.96.123.45    (same namespace only)
```

---

### Network Policies

By default, all pods can communicate with all other pods. **NetworkPolicies** let you restrict this — defining exactly which pods can talk to which.

```yaml
# deny-all-policy.yaml — Block all traffic to nexus-db
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-to-db
  namespace: nexus-production
spec:
  podSelector:
    matchLabels:
      app: nexus-db        # Apply this policy to nexus-db pods
  policyTypes:
  - Ingress                # Restrict incoming traffic
  - Egress                 # Restrict outgoing traffic
  # No ingress/egress rules = deny all traffic
```

```yaml
# allow-api-to-db.yaml — Allow only nexus-api to reach nexus-db
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: nexus-production
spec:
  podSelector:
    matchLabels:
      app: nexus-db        # Policy applies to DB pods
  policyTypes:
  - Ingress                # Only restrict incoming traffic
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: nexus-api   # Only allow traffic FROM API pods
    ports:
    - protocol: TCP
      port: 5432           # Only on postgres port
```

```yaml
# allow-frontend-to-api.yaml — Allow frontend to reach API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: nexus-production
spec:
  podSelector:
    matchLabels:
      app: nexus-api       # Policy applies to API pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: nexus-frontend
    ports:
    - protocol: TCP
      port: 5000

  # Also allow from monitoring namespace
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 9090           # Metrics port
```

**NetworkPolicy rules:**
```
podSelector:      {} (empty) = all pods in namespace
                  {matchLabels: {app: x}} = only matching pods

ingress.from:
  podSelector     = pods matching labels
  namespaceSelector = all pods in matching namespaces
  ipBlock         = traffic from/to specific CIDRs

policyTypes:
  Ingress only    = restricts incoming, allows all outgoing
  Egress only     = restricts outgoing, allows all incoming
  Both            = restricts both directions
```

---

## Part 5: Practical Tasks

### Task 1: Creating a Deployment

```bash
# Step 1: Create the namespace
kubectl create namespace nexus-demo

# Step 2: Create a ConfigMap for app config
cat > nexus-configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-config
  namespace: nexus-demo
data:
  APP_NAME: "NexusCorp API"
  APP_ENV: "demo"
  LOG_LEVEL: "INFO"
EOF

kubectl apply -f nexus-configmap.yaml

# Step 3: Create the Deployment
cat > nexus-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  namespace: nexus-demo
  labels:
    app: nexus-api
  annotations:
    kubernetes.io/change-cause: "Initial deployment v1"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nexus-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: nexus-api
        version: v1
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
            cpu: "250m"
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

kubectl apply -f nexus-deployment.yaml

# Step 4: Watch pods being created
kubectl get pods -n nexus-demo -w

# Step 5: Verify deployment
kubectl get deployment nexus-api -n nexus-demo
kubectl describe deployment nexus-api -n nexus-demo

# Step 6: View the ReplicaSet created by the Deployment
kubectl get replicaset -n nexus-demo

# Step 7: Test rolling update
kubectl set image deployment/nexus-api \
    nexus-api=nginx:latest \
    -n nexus-demo \
    --record

kubectl rollout status deployment/nexus-api -n nexus-demo
kubectl rollout history deployment/nexus-api -n nexus-demo

# Step 8: Test rollback
kubectl rollout undo deployment/nexus-api -n nexus-demo
kubectl rollout status deployment/nexus-api -n nexus-demo
```

---

### Task 2: Exposing the Deployment

```bash
# Step 1: Create a ClusterIP Service (internal access)
cat > nexus-clusterip-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-internal
  namespace: nexus-demo
spec:
  type: ClusterIP
  selector:
    app: nexus-api
  ports:
  - name: http
    port: 80
    targetPort: 80
EOF

kubectl apply -f nexus-clusterip-service.yaml

# Step 2: Create a LoadBalancer Service (external access)
cat > nexus-loadbalancer-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-public
  namespace: nexus-demo
spec:
  type: LoadBalancer
  selector:
    app: nexus-api
  ports:
  - name: http
    port: 80
    targetPort: 80
EOF

kubectl apply -f nexus-loadbalancer-service.yaml

# Step 3: View services
kubectl get services -n nexus-demo
# NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
# nexus-api-internal   ClusterIP      10.96.45.12    <none>        80/TCP
# nexus-api-public     LoadBalancer   10.96.89.23    <pending>     80:30080/TCP

# Step 4: On Minikube — get access URL
minikube service nexus-api-public -n nexus-demo --url
# http://192.168.49.2:30080

# Access the application
curl $(minikube service nexus-api-public -n nexus-demo --url)

# Step 5: Test using imperative expose command
kubectl expose deployment nexus-api \
    -n nexus-demo \
    --name=nexus-api-nodeport \
    --type=NodePort \
    --port=80 \
    --target-port=80

kubectl get service nexus-api-nodeport -n nexus-demo

# Step 6: View endpoints (pod IPs behind the service)
kubectl get endpoints nexus-api-internal -n nexus-demo

# Step 7: Test internal DNS (from within cluster)
kubectl run test-pod \
    --image=busybox:latest \
    --restart=Never \
    -n nexus-demo \
    --command -- sleep 300

kubectl exec -it test-pod -n nexus-demo -- \
    wget -q -O- http://nexus-api-internal.nexus-demo.svc.cluster.local

kubectl delete pod test-pod -n nexus-demo
```

---

### Task 3: Network Policies

```bash
# Step 1: Create two applications
cat > nexus-apps.yaml << 'EOF'
# API deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  namespace: nexus-demo
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
        ports:
        - containerPort: 80
---
# Database deployment (simulated)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-db
  namespace: nexus-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nexus-db
  template:
    metadata:
      labels:
        app: nexus-db
    spec:
      containers:
      - name: nexus-db
        image: nginx:1.25
        ports:
        - containerPort: 80
---
# DB Service
apiVersion: v1
kind: Service
metadata:
  name: nexus-db-service
  namespace: nexus-demo
spec:
  selector:
    app: nexus-db
  ports:
  - port: 80
    targetPort: 80
EOF

kubectl apply -f nexus-apps.yaml

# Step 2: Verify all pods can reach db (before policy)
kubectl exec -it \
    $(kubectl get pod -n nexus-demo -l app=nexus-api -o jsonpath='{.items[0].metadata.name}') \
    -n nexus-demo -- \
    wget -q --timeout=5 -O- http://nexus-db-service.nexus-demo.svc.cluster.local
# Should succeed (html output)

# Step 3: Apply deny-all policy to database
cat > deny-all-db.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-to-db
  namespace: nexus-demo
spec:
  podSelector:
    matchLabels:
      app: nexus-db
  policyTypes:
  - Ingress
  # No ingress rules = deny all inbound traffic
EOF

kubectl apply -f deny-all-db.yaml

# Step 4: Verify traffic is now blocked
kubectl exec -it \
    $(kubectl get pod -n nexus-demo -l app=nexus-api -o jsonpath='{.items[0].metadata.name}') \
    -n nexus-demo -- \
    wget -q --timeout=5 -O- http://nexus-db-service.nexus-demo.svc.cluster.local
# Should TIMEOUT now — blocked by NetworkPolicy

# Step 5: Allow only nexus-api to reach nexus-db
cat > allow-api-to-db.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: nexus-demo
spec:
  podSelector:
    matchLabels:
      app: nexus-db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: nexus-api
    ports:
    - protocol: TCP
      port: 80
EOF

kubectl apply -f allow-api-to-db.yaml

# Step 6: Verify nexus-api can reach nexus-db again
kubectl exec -it \
    $(kubectl get pod -n nexus-demo -l app=nexus-api -o jsonpath='{.items[0].metadata.name}') \
    -n nexus-demo -- \
    wget -q --timeout=5 -O- http://nexus-db-service.nexus-demo.svc.cluster.local | head -3
# Should succeed again!

# Step 7: View network policies
kubectl get networkpolicy -n nexus-demo
kubectl describe networkpolicy allow-api-to-db -n nexus-demo
```

---

## 🔧 Mini-Project: Complete NexusCorp Application Stack

```yaml
# nexus-complete-stack.yaml — Full deployment + service + networking

---
apiVersion: v1
kind: Namespace
metadata:
  name: nexus-stack
  labels:
    project: nexuscorp

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-config
  namespace: nexus-stack
data:
  APP_ENV: "production"
  LOG_LEVEL: "INFO"

---
# Frontend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-frontend
  namespace: nexus-stack
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nexus-frontend
  template:
    metadata:
      labels:
        app: nexus-frontend
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests: {memory: "64Mi", cpu: "100m"}
          limits: {memory: "128Mi", cpu: "200m"}

---
# Frontend Service (external)
apiVersion: v1
kind: Service
metadata:
  name: nexus-frontend-service
  namespace: nexus-stack
spec:
  type: LoadBalancer
  selector:
    app: nexus-frontend
  ports:
  - port: 80
    targetPort: 80

---
# API Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  namespace: nexus-stack
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nexus-api
  template:
    metadata:
      labels:
        app: nexus-api
        tier: backend
    spec:
      containers:
      - name: api
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests: {memory: "128Mi", cpu: "200m"}
          limits: {memory: "256Mi", cpu: "500m"}

---
# API Service (internal only)
apiVersion: v1
kind: Service
metadata:
  name: nexus-api-service
  namespace: nexus-stack
spec:
  type: ClusterIP
  selector:
    app: nexus-api
  ports:
  - port: 80
    targetPort: 80

---
# Database Service (internal only)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-db
  namespace: nexus-stack
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nexus-db
  template:
    metadata:
      labels:
        app: nexus-db
        tier: database
    spec:
      containers:
      - name: db
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests: {memory: "256Mi", cpu: "250m"}
          limits: {memory: "512Mi", cpu: "500m"}

---
apiVersion: v1
kind: Service
metadata:
  name: nexus-db-service
  namespace: nexus-stack
spec:
  type: ClusterIP
  selector:
    app: nexus-db
  ports:
  - port: 80
    targetPort: 80

---
# NetworkPolicy: Deny all to DB by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-deny-all
  namespace: nexus-stack
spec:
  podSelector:
    matchLabels:
      app: nexus-db
  policyTypes: [Ingress]

---
# NetworkPolicy: Only API can reach DB
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-api
  namespace: nexus-stack
spec:
  podSelector:
    matchLabels:
      app: nexus-db
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: nexus-api
```

```bash
# Deploy everything
kubectl apply -f nexus-complete-stack.yaml

# Verify
kubectl get all -n nexus-stack
kubectl get networkpolicy -n nexus-stack

# Access frontend
minikube service nexus-frontend-service -n nexus-stack --url

# Clean up
kubectl delete namespace nexus-stack
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Deployment** | A Kubernetes object that manages a set of identical pods — handles scaling, updates, rollbacks |
| **ReplicaSet** | Ensures a specified number of pod replicas are always running — managed by Deployments |
| **Rolling Update** | Update strategy that gradually replaces old pods with new ones — zero downtime |
| **maxSurge** | Rolling update setting — max pods above desired count during update |
| **maxUnavailable** | Rolling update setting — max pods below desired count during update |
| **`kubectl rollout`** | Command to manage deployment rollouts: status, history, undo, pause, resume |
| **`--record`** | Flag that records the command in rollout history (deprecated, use annotations) |
| **Service** | A stable network endpoint that load-balances traffic to a set of pods |
| **ClusterIP** | Service type that creates an internal-only cluster IP — default type |
| **NodePort** | Service type that exposes the service on each node's IP at a static port |
| **LoadBalancer** | Service type that provisions a cloud load balancer for external access |
| **ExternalName** | Service type that maps a Kubernetes service name to an external DNS name |
| **Endpoints** | The list of pod IPs and ports that a Service routes traffic to |
| **Selector** | A label query used to identify which pods a Service or Deployment manages |
| **Labels** | Key-value pairs attached to objects used for identification and selection |
| **CNI** | Container Network Interface — the plugin standard for Kubernetes networking |
| **Calico** | Popular CNI plugin providing networking and network policies |
| **Flannel** | Simple CNI plugin providing basic overlay networking |
| **NetworkPolicy** | Kubernetes object that controls which pods can communicate with each other |
| **Ingress (policy)** | NetworkPolicy direction controlling incoming traffic to pods |
| **Egress (policy)** | NetworkPolicy direction controlling outgoing traffic from pods |
| **podSelector** | Selects pods to apply a NetworkPolicy to |
| **namespaceSelector** | Selects pods from specific namespaces in a NetworkPolicy |
| **CoreDNS** | The DNS server in Kubernetes that resolves service names to ClusterIPs |
| **`svc.cluster.local`** | The DNS domain suffix for Kubernetes services |
| **HPA** | HorizontalPodAutoscaler — automatically scales deployments based on CPU/memory |
| **`kubectl expose`** | Imperative command to create a Service for an existing deployment |
| **`kubectl scale`** | Imperative command to change the number of replicas |
| **YAML manifest** | A YAML file describing a Kubernetes resource |
| **`apiVersion`** | The API group and version for a Kubernetes resource |
| **`kind`** | The type of Kubernetes object (Deployment, Service, Pod, etc.) |

---

## 📚 Resources

- [Understanding Kubernetes Objects — Official Docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/) — objects, manifests, labels
- [Kubernetes Deployments — Official Docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) — complete Deployment reference
- [Kubernetes Services — Official Docs](https://kubernetes.io/docs/concepts/services-networking/service/) — Service types and networking
- [Kubernetes Networking — Official Docs](https://kubernetes.io/docs/concepts/services-networking/) — the networking model
- [Network Policies — Official Docs](https://kubernetes.io/docs/concepts/services-networking/network-policies/) — NetworkPolicy reference
- [YAML Tutorial for DevOps — YouTube](https://www.youtube.com/results?search_query=YAML+tutorial+DevOps+Engineer) — the YAML video referenced in curriculum

---

## 🔭 Day 42 Preview: ConfigMaps, Secrets & Persistent Storage

Today you deployed applications with Services and networking. Tomorrow you make those deployments properly configurable and stateful.

You will learn:
- ConfigMaps — inject configuration into pods without rebuilding images
- Secrets — safely store and use sensitive data (passwords, tokens, certificates)
- PersistentVolumes and PersistentVolumeClaims — request durable storage for stateful apps
- StorageClasses — dynamic volume provisioning from cloud providers
- StatefulSets — for applications that need stable network identities and persistent storage (databases)

These are the pieces that let you run real-world applications — not just stateless web servers, but databases, caches, and services that need configuration management.

---

> 💬 *"Deployments define what should run. Services define how to reach it. Network Policies define who is allowed to. Together, they are the complete picture of a running application in Kubernetes."*
>
> — DevOps Learning By Yukta
