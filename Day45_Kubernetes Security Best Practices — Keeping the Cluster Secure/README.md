# Day 45: Kubernetes Security Best Practices — Keeping the Cluster Secure

> **DevOps Transformation** | Kubernetes & Container Orchestration Series

---

## 📋 Overview

You have mastered deploying, scaling, and configuring applications in Kubernetes. Today you shift focus to something equally critical: **security**. A misconfigured Kubernetes cluster is one of the most dangerous attack surfaces in modern infrastructure — it can expose entire cloud accounts, sensitive data, and internal systems to the internet.

Kubernetes security is not one setting or one tool — it is a layered approach: control who can do what (RBAC), control what can talk to what (Network Policies), harden the entry point (API Server), secure the workloads (Pod Security), protect sensitive data (Secrets Management), and monitor everything (Auditing and Logging).

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| RBAC | Who can access what in the cluster |
| Network Policies | Which pods can communicate with which |
| API Server Security | Hardening the cluster's front door |
| Pod Security | Securing individual workloads |
| Secrets Management | Encoding, encryption, and best practices |
| Node Security | Hardening the machines that run workloads |
| Cluster Network Security | Securing inter-cluster traffic |
| Auditing and Logging | Tracking who did what and when |

---

## 🏢 NexusCorp Scenario

> NexusCorp's Kubernetes cluster was breached. A developer accidentally deployed a pod with `privileged: true` and no resource limits. An attacker exploited a vulnerability in the container and escaped to the host node, gaining access to cloud credentials mounted on the node. From there, they accessed S3 buckets containing customer data.
>
> The post-mortem identified seven security gaps — each corresponding to one section of today's content. Today you close all seven.

---

## Part 1: Role-Based Access Control (RBAC)

### What Is RBAC?

RBAC controls **who** can do **what** in the Kubernetes cluster. Without RBAC, every user and service account has unrestricted access to all cluster resources — a critical security risk.

```
RBAC Model:
─────────────────────────────────────────────────────────

Subject          Verb        Resource
(who)           (what)      (on what)

├── User         → get       → pods
├── Group        → create    → deployments
└── ServiceAccount→ delete  → secrets

RBAC Objects:
  Role            → Permissions within a namespace
  ClusterRole     → Permissions across entire cluster
  RoleBinding     → Binds Role to Subject (in a namespace)
  ClusterRoleBinding → Binds ClusterRole to Subject (cluster-wide)
```

---

### RBAC Components

#### Role — Namespace-Scoped Permissions

```yaml
# role-pod-reader.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: nexus-production
rules:
- apiGroups: [""]                # "" = core API group
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]

- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list"]

# Available verbs:
# get, list, watch → read-only
# create, update, patch → write
# delete, deletecollection → delete
# * → all verbs (avoid in production)
```

#### ClusterRole — Cluster-Wide Permissions

```yaml
# clusterrole-secret-reader.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nexus-ops-role
rules:
# Read pods in any namespace
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec"]
  verbs: ["get", "list", "watch"]

# Read deployments
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "watch"]

# Read node information
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]

# Allow reading resource metrics
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list"]
```

#### RoleBinding — Assign Role to User/Group

```yaml
# rolebinding-pod-reader.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: nexus-production
subjects:
# Bind to a specific user
- kind: User
  name: jane.smith@nexuscorp.com
  apiGroup: rbac.authorization.k8s.io

# Bind to a group
- kind: Group
  name: nexus-developers
  apiGroup: rbac.authorization.k8s.io

# Bind to a ServiceAccount
- kind: ServiceAccount
  name: nexus-ci-account
  namespace: nexus-ci

roleRef:
  kind: Role                    # Role or ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### ServiceAccount — Identity for Pods

```yaml
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nexus-api-sa
  namespace: nexus-production
  annotations:
    description: "Service account for nexus-api deployment"
automountServiceAccountToken: false   # Don't auto-mount token
```

```yaml
# Use in deployment
spec:
  serviceAccountName: nexus-api-sa
  automountServiceAccountToken: false   # Explicit — don't mount token
```

---

### RBAC Best Practices

```
Principle of Least Privilege:
  ✅ Grant only the permissions needed for the specific task
  ✅ Prefer namespace-scoped Roles over cluster-wide ClusterRoles
  ✅ Never use cluster-admin unless absolutely necessary
  ✅ Use ServiceAccounts per application — not the default SA
  ✅ Set automountServiceAccountToken: false when not needed
  ✅ Audit permissions regularly: kubectl auth can-i --list

  ❌ Don't use wildcards (*) in resources or verbs
  ❌ Don't bind cluster-admin to users or SAs
  ❌ Don't share ServiceAccounts between applications
  ❌ Don't let pods access the Kubernetes API unless required
```

```bash
# Check what permissions a user has
kubectl auth can-i get pods -n nexus-production --as=jane.smith@nexuscorp.com
kubectl auth can-i delete secrets -n nexus-production --as=jane.smith@nexuscorp.com

# List all permissions for a user
kubectl auth can-i --list --as=jane.smith@nexuscorp.com -n nexus-production

# View all roles and bindings
kubectl get roles,rolebindings -n nexus-production
kubectl get clusterroles,clusterrolebindings
```

---

## Part 2: Network Policies

Network Policies restrict which pods can communicate with each other and with external endpoints. By default, Kubernetes allows all traffic — Network Policies change this to a deny-by-default model.

```
Without Network Policies:
  All pods → All pods (unrestricted)

With Network Policies:
  Frontend → API only (on port 5000)
  API → Database only (on port 5432)
  Database → nobody (no egress needed)
  Monitoring → all pods (port 9090 metrics)
```

### Essential Network Policy Patterns

#### Default Deny All

```yaml
# deny-all.yaml — Apply to every namespace first
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: nexus-production
spec:
  podSelector: {}          # Applies to ALL pods in namespace
  policyTypes:
  - Ingress
  - Egress
  # No rules = deny everything
```

#### Allow Only Specific Traffic

```yaml
# allow-frontend-to-api.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: nexus-production
spec:
  podSelector:
    matchLabels:
      app: nexus-api
  policyTypes:
  - Ingress
  ingress:
  # Allow from frontend pods only
  - from:
    - podSelector:
        matchLabels:
          app: nexus-frontend
    ports:
    - protocol: TCP
      port: 5000

  # Allow from monitoring namespace
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 9090

---
# allow-api-to-db.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: nexus-production
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
      port: 5432

---
# allow-dns-egress.yaml — Always allow DNS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: nexus-production
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

---

## Part 3: Securing the API Server

The API server is the single entry point to the Kubernetes control plane. Hardening it is critical.

### Key API Server Security Flags

```bash
# View current API server configuration (Minikube)
kubectl get pod kube-apiserver-minikube -n kube-system -o yaml | \
    grep -A 100 "command:"

# Key security flags to verify are set:
--anonymous-auth=false            # Disable anonymous access
--authorization-mode=Node,RBAC    # Enable RBAC (not AlwaysAllow)
--enable-admission-plugins=...    # Enable security admission controllers
--tls-cert-file=...               # TLS certificate for API server
--tls-private-key-file=...        # TLS private key
--etcd-cafile=...                 # CA for etcd communication
--etcd-certfile=...               # Client cert for etcd
--etcd-keyfile=...                # Client key for etcd
--kubelet-certificate-authority=. # CA for kubelet
--audit-log-path=...              # Enable audit logging
--audit-log-maxage=30             # Keep audit logs 30 days
--audit-log-maxbackup=10          # Keep 10 audit log files
--audit-log-maxsize=100           # 100MB per audit log file
```

### Admission Controllers

Admission controllers intercept API server requests and can validate, mutate, or reject them before objects are stored:

```bash
# Check enabled admission plugins
kubectl exec -n kube-system kube-apiserver-minikube -- \
    kube-apiserver -h | grep enable-admission-plugins

# Essential admission controllers for security:
# NodeRestriction        → Nodes can only modify their own resources
# PodSecurityAdmission   → Enforce pod security standards
# ResourceQuota          → Enforce namespace resource limits
# LimitRanger            → Enforce resource limit defaults
# ServiceAccount         → Enforce service account automation
```

### API Server Access Control

```bash
# Test API server accessibility
curl -k https://$(kubectl config view --minify \
    -o jsonpath='{.clusters[0].cluster.server}')/api/v1/pods

# Should return 403 Forbidden (not 200) for anonymous access
# If it returns data — anonymous auth is enabled (security risk!)

# Test with credentials
kubectl run test-curl --image=curlimages/curl --restart=Never -- \
    curl -sk https://kubernetes.default.svc/api/v1/namespaces
# Should return 403 if RBAC is properly configured
```

---

## Part 4: Pod Security

### Pod Security Standards

Kubernetes defines three security profiles for pods:

```
Privileged:   No restrictions (dangerous — dev/testing only)
Baseline:     Prevents known privilege escalations
Restricted:   Hardened — follows pod security best practices
```

### Secure Pod Configuration

```yaml
# secure-pod.yaml — Production-hardened pod
apiVersion: v1
kind: Pod
metadata:
  name: nexus-secure-pod
  namespace: nexus-production
spec:
  # Use a dedicated ServiceAccount with minimal permissions
  serviceAccountName: nexus-api-sa
  automountServiceAccountToken: false

  # Pod-level security context
  securityContext:
    runAsNonRoot: true             # Never run as root
    runAsUser: 1000                # Specific non-root UID
    runAsGroup: 3000               # Specific GID
    fsGroup: 2000                  # Group for mounted volumes
    seccompProfile:
      type: RuntimeDefault         # Apply default seccomp profile

  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v2

    # Container-level security context
    securityContext:
      allowPrivilegeEscalation: false   # Cannot gain more privileges
      readOnlyRootFilesystem: true      # Root filesystem is read-only
      privileged: false                  # Never privileged
      capabilities:
        drop:
        - ALL                           # Drop all Linux capabilities
        add:
        - NET_BIND_SERVICE              # Add only what's needed

    # Always set resource limits
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"        # Prevents memory exhaustion
        cpu: "500m"            # Prevents CPU starvation

    # Write temporary files to specific writable volumes
    volumeMounts:
    - name: tmp-dir
      mountPath: /tmp
    - name: var-run
      mountPath: /var/run

  volumes:
  - name: tmp-dir
    emptyDir: {}
  - name: var-run
    emptyDir: {}
```

### Pod Security Admission (Kubernetes 1.23+)

```yaml
# Label namespace to enforce security standards
kubectl label namespace nexus-production \
    pod-security.kubernetes.io/enforce=restricted \
    pod-security.kubernetes.io/enforce-version=latest \
    pod-security.kubernetes.io/audit=restricted \
    pod-security.kubernetes.io/warn=restricted
```

### Security Anti-Patterns to Avoid

```yaml
# ❌ NEVER do these in production:

spec:
  hostNetwork: true          # Shares host network namespace
  hostPID: true              # Shares host PID namespace
  hostIPC: true              # Shares host IPC namespace

  containers:
  - securityContext:
      privileged: true       # Full root access to host
      runAsUser: 0           # Running as root
      allowPrivilegeEscalation: true

  volumes:
  - name: host-root
    hostPath:
      path: /                # Mounting host filesystem
```

---

## Part 5: Secrets Management

### Secrets in Depth

```bash
# Check if secrets are encrypted at rest
kubectl get apiserver -o yaml | grep encryption

# View secret (base64 encoded — NOT encrypted by default)
kubectl get secret nexus-secrets -n nexus-production -o yaml
```

### Enable Encryption at Rest

```yaml
# /etc/kubernetes/encryption-config.yaml (on control plane node)
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:                        # AES-CBC encryption
      keys:
      - name: key1
        secret: <base64-32-byte-random-key>   # openssl rand -base64 32
  - identity: {}                   # Fallback for unencrypted secrets
```

```bash
# Generate an encryption key
openssl rand -base64 32

# Apply encryption config to kube-apiserver:
# --encryption-provider-config=/etc/kubernetes/encryption-config.yaml

# After enabling, re-encrypt existing secrets
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

### External Secret Management

```
For production, use an external secret manager:

┌─────────────────────────────────────────────────────────┐
│  External Secrets Operator                              │
│                                                         │
│  AWS Secrets Manager ──→ ExternalSecret ──→ K8s Secret │
│  HashiCorp Vault ──────→ ExternalSecret ──→ K8s Secret │
│  GCP Secret Manager ───→ ExternalSecret ──→ K8s Secret │
└─────────────────────────────────────────────────────────┘
```

```yaml
# external-secret.yaml (using External Secrets Operator)
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: nexus-db-secret
  namespace: nexus-production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: nexus-db-secret    # K8s Secret to create
  data:
  - secretKey: DB_PASSWORD
    remoteRef:
      key: nexuscorp/production/db
      property: password
```

---

## Part 6: Node Security

### Hardening Kubernetes Nodes

```bash
# 1. Keep OS and packages updated
sudo apt update && sudo apt upgrade -y

# 2. Enable automatic security updates (Ubuntu)
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades

# 3. Configure firewall — allow only necessary ports
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Kubernetes required ports:
# Master nodes:
sudo ufw allow 6443/tcp    # API server
sudo ufw allow 2379:2380/tcp # etcd
sudo ufw allow 10250/tcp   # Kubelet API
sudo ufw allow 10259/tcp   # kube-scheduler
sudo ufw allow 10257/tcp   # kube-controller-manager

# Worker nodes:
sudo ufw allow 10250/tcp   # Kubelet API
sudo ufw allow 30000:32767/tcp # NodePort Services

sudo ufw enable

# 4. Disable swap (required by Kubernetes)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# 5. Restrict kubelet configuration
# /var/lib/kubelet/config.yaml
# authentication:
#   anonymous:
#     enabled: false          # Disable anonymous kubelet access
#   webhook:
#     enabled: true
# authorization:
#   mode: Webhook             # Use RBAC for kubelet authorization
# readOnlyPort: 0            # Disable read-only kubelet API
```

### Node Taints for Workload Isolation

```bash
# Taint a node to only accept specific workloads
kubectl taint nodes node1 dedicated=production:NoSchedule

# Only pods with this toleration run on the tainted node:
# tolerations:
# - key: "dedicated"
#   operator: "Equal"
#   value: "production"
#   effect: "NoSchedule"

# Taint master nodes to prevent application workloads
kubectl taint nodes master-node node-role.kubernetes.io/master:NoSchedule
```

---

## Part 7: Cluster Network Security

### TLS Everywhere

```
Kubernetes uses TLS for ALL internal communication:

kubectl → API Server:         TLS (port 6443)
API Server → etcd:            Mutual TLS (both verify each other)
API Server → kubelet:         TLS
kubelet → API Server:         TLS
kube-proxy → API Server:      TLS
Pods → Pods:                  Plain by default → use mTLS (Istio, Linkerd)
```

### Service Mesh for mTLS (Istio)

```bash
# Install Istio for automatic mTLS between pods
curl -L https://istio.io/downloadIstio | sh -
cd istio-*/
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y

# Label namespace for automatic sidecar injection
kubectl label namespace nexus-production istio-injection=enabled

# All pods in this namespace get an Envoy sidecar
# Pod-to-pod communication is automatically encrypted
```

### Ingress Security

```yaml
# Secure Ingress with TLS
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nexus-ingress
  namespace: nexus-production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/hsts: "true"
spec:
  tls:
  - hosts:
    - api.nexuscorp.com
    secretName: nexus-tls-cert      # TLS secret
  rules:
  - host: api.nexuscorp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nexus-api-service
            port:
              number: 80
```

---

## Part 8: Auditing and Logging

### Kubernetes Audit Policy

```yaml
# audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
# Log all requests at the metadata level
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets", "configmaps"]

# Log full request/response for sensitive operations
- level: RequestResponse
  verbs: ["delete", "deletecollection"]
  resources:
  - group: ""
    resources: ["pods", "services"]
  - group: "apps"
    resources: ["deployments"]

# Log authentication failures
- level: Metadata
  users: ["system:anonymous"]

# Skip logging for read-only health checks
- level: None
  users: ["system:kube-proxy"]
  verbs: ["watch"]
  resources:
  - group: ""
    resources: ["endpoints", "services"]
```

### Enable Audit Logging

```bash
# Add to kube-apiserver flags:
# --audit-policy-file=/etc/kubernetes/audit-policy.yaml
# --audit-log-path=/var/log/kubernetes/audit.log
# --audit-log-maxage=30
# --audit-log-maxbackup=10
# --audit-log-maxsize=100

# View audit logs
sudo tail -f /var/log/kubernetes/audit.log | python3 -m json.tool
```

### Monitoring with Prometheus and Grafana

```bash
# Install kube-prometheus-stack (Prometheus + Grafana + Alertmanager)
helm repo add prometheus-community \
    https://prometheus-community.github.io/helm-charts
helm repo update

helm install kube-prometheus \
    prometheus-community/kube-prometheus-stack \
    --namespace monitoring \
    --create-namespace \
    --set grafana.adminPassword=nexus-grafana-pass

# Access Grafana dashboard
kubectl port-forward svc/kube-prometheus-grafana 3000:80 -n monitoring
# Open: http://localhost:3000
# Default login: admin / nexus-grafana-pass

# Access Prometheus
kubectl port-forward svc/kube-prometheus-kube-prometheus 9090:9090 -n monitoring
# Open: http://localhost:9090
```

### Key Metrics to Monitor

```
Security-Relevant Metrics:
  ├── Authentication failures
  ├── Unauthorized API calls (403 responses)
  ├── Pods running as root
  ├── Privileged pod deployments
  ├── Secret access patterns
  ├── Node CPU/memory pressure
  └── Unusual network traffic patterns

Useful Prometheus Queries:
  # Pods running as root
  kube_pod_container_info{container!=""}
    unless on(pod, namespace) kube_pod_spec_containers_security_context_run_as_non_root

  # API server error rate
  rate(apiserver_request_total{code=~"5.."}[5m])

  # Failed authentication attempts
  sum(rate(apiserver_authentication_attempts_total{result="error"}[5m]))
```

---

## Part 9: Practical Tasks

### Task 1: Configure RBAC

```bash
# Step 1: Create a namespace
kubectl create namespace nexus-rbac-test

# Step 2: Create a ServiceAccount (simulates a user)
kubectl create serviceaccount nexus-dev-user -n nexus-rbac-test

# Step 3: Create a Role — read-only access to pods
cat > pod-reader-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: nexus-rbac-test
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
EOF

kubectl apply -f pod-reader-role.yaml

# Step 4: Bind the Role to the ServiceAccount
cat > pod-reader-binding.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: nexus-rbac-test
subjects:
- kind: ServiceAccount
  name: nexus-dev-user
  namespace: nexus-rbac-test
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f pod-reader-binding.yaml

# Step 5: Get the token for the service account
TOKEN=$(kubectl create token nexus-dev-user -n nexus-rbac-test)

# Step 6: Test permissions — should be ALLOWED
kubectl auth can-i get pods -n nexus-rbac-test \
    --as=system:serviceaccount:nexus-rbac-test:nexus-dev-user
# yes

# Step 7: Test permissions — should be DENIED
kubectl auth can-i delete pods -n nexus-rbac-test \
    --as=system:serviceaccount:nexus-rbac-test:nexus-dev-user
# no

kubectl auth can-i get secrets -n nexus-rbac-test \
    --as=system:serviceaccount:nexus-rbac-test:nexus-dev-user
# no

kubectl auth can-i get pods -n default \
    --as=system:serviceaccount:nexus-rbac-test:nexus-dev-user
# no (different namespace)

echo "RBAC configured correctly — user can only read pods in nexus-rbac-test"
```

---

### Task 2: Implement Network Policies

```bash
# Step 1: Create namespace and deploy two pods
kubectl create namespace nexus-netpol-test

# Deploy frontend pod
kubectl run frontend \
    --image=nginx:latest \
    --labels="app=frontend" \
    -n nexus-netpol-test

# Deploy backend pod
kubectl run backend \
    --image=nginx:latest \
    --labels="app=backend" \
    -n nexus-netpol-test

# Deploy test client
kubectl run test-client \
    --image=busybox:latest \
    --labels="app=test-client" \
    --restart=Never \
    -n nexus-netpol-test \
    -- sleep 3600

# Wait for pods to start
kubectl get pods -n nexus-netpol-test -w

# Step 2: Test connectivity before policies (should all work)
BACKEND_IP=$(kubectl get pod backend -n nexus-netpol-test \
    -o jsonpath='{.status.podIP}')
FRONTEND_IP=$(kubectl get pod frontend -n nexus-netpol-test \
    -o jsonpath='{.status.podIP}')

echo "Backend IP: $BACKEND_IP"
echo "Frontend IP: $FRONTEND_IP"

kubectl exec test-client -n nexus-netpol-test -- \
    wget -q --timeout=3 -O- http://$BACKEND_IP | head -2
# Works — returns nginx HTML

# Step 3: Apply deny-all policy
cat > deny-all.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: nexus-netpol-test
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
EOF

kubectl apply -f deny-all.yaml

# Test: should now FAIL
kubectl exec test-client -n nexus-netpol-test -- \
    wget -q --timeout=3 -O- http://$BACKEND_IP
# Should timeout — blocked!

# Step 4: Allow only frontend to reach backend
cat > allow-frontend-to-backend.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: nexus-netpol-test
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
EOF

kubectl apply -f allow-frontend-to-backend.yaml

# Test: test-client should STILL be blocked
kubectl exec test-client -n nexus-netpol-test -- \
    wget -q --timeout=3 -O- http://$BACKEND_IP
# Still blocked (test-client doesn't have app=frontend label)

echo "NetworkPolicy working — only frontend can reach backend"
```

---

### Task 3: Secure the API Server

```bash
# Check current API server configuration
kubectl get pod kube-apiserver-minikube \
    -n kube-system \
    -o jsonpath='{.spec.containers[0].command}' | \
    python3 -m json.tool

# Verify key security settings
kubectl get pod kube-apiserver-minikube -n kube-system -o yaml | \
    grep -E "anonymous-auth|authorization-mode|admission-plugins"

# Check if API server is accessible anonymously
API_SERVER=$(kubectl config view --minify \
    -o jsonpath='{.clusters[0].cluster.server}')

curl -sk $API_SERVER/api/v1/namespaces
# Should return 401 Unauthorized (not actual data)

# Test RBAC is working
curl -sk $API_SERVER/api/v1/pods \
    -H "Authorization: Bearer invalid-token"
# Should return 401 Unauthorized

echo "API server security checks complete"
```

---

### Task 4: Explore Secrets Security

```bash
# Step 1: Create a secret and verify it's base64 encoded
kubectl create secret generic security-test-secret \
    --from-literal=password=super-secret-123 \
    --from-literal=api-key=sk-test-key-456 \
    -n nexus-rbac-test

# View raw secret (shows base64 encoding)
kubectl get secret security-test-secret \
    -n nexus-rbac-test -o yaml

# Step 2: Decode the secret
kubectl get secret security-test-secret \
    -n nexus-rbac-test \
    -o jsonpath='{.data.password}' | base64 -d
echo ""
# Outputs: super-secret-123
# PROOF: base64 != encryption!

# Step 3: Verify service account cannot read secrets
kubectl auth can-i get secrets -n nexus-rbac-test \
    --as=system:serviceaccount:nexus-rbac-test:nexus-dev-user
# no — pod-reader role has no access to secrets

echo "Secret security verification complete"
```

---

### Task 5: Node Hardening

```bash
# Check node security status
kubectl describe node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')

# View node conditions and taints
kubectl get nodes -o custom-columns=\
"NAME:.metadata.name,STATUS:.status.conditions[-1].type,TAINTS:.spec.taints"

# Check what's running on the node
kubectl get pods -A -o wide | grep $(kubectl get nodes \
    -o jsonpath='{.items[0].metadata.name}')

# Simulate node hardening — add a taint
# (Warning: this may prevent pods from scheduling)
# kubectl taint nodes NODE_NAME dedicated=production:NoSchedule

# Check kubelet security configuration
kubectl get configmap kubelet-config -n kube-system -o yaml 2>/dev/null || \
    echo "Kubelet config not exposed as ConfigMap on this cluster"
```

---

### Task 6: Set Up Audit Logging and Monitoring

```bash
# Enable metrics server (for kubectl top)
minikube addons enable metrics-server

# Verify metrics are available
kubectl top nodes
kubectl top pods -A

# Install kube-prometheus-stack (if Helm is available)
if command -v helm &>/dev/null; then
    helm repo add prometheus-community \
        https://prometheus-community.github.io/helm-charts
    helm repo update

    helm install kube-prometheus \
        prometheus-community/kube-prometheus-stack \
        --namespace monitoring \
        --create-namespace \
        --set grafana.adminPassword=nexus-grafana-pass \
        --wait

    # Access dashboards
    echo "Grafana: kubectl port-forward svc/kube-prometheus-grafana 3000:80 -n monitoring"
    echo "Login: admin / nexus-grafana-pass"
    echo ""
    echo "Prometheus: kubectl port-forward svc/kube-prometheus-kube-prometheus 9090:9090 -n monitoring"
else
    echo "Helm not installed. Install from: https://helm.sh/docs/intro/install/"
fi

# View cluster events as a basic audit trail
kubectl get events -A --sort-by='.lastTimestamp' | tail -20
```

---

## 🔧 Mini-Project: NexusCorp Security Hardening

```bash
#!/bin/bash
# nexus_security_hardening.sh
# Apply security best practices to a NexusCorp namespace

set -e
NS="nexus-secure"

echo "=============================================="
echo "  NexusCorp Cluster Security Hardening"
echo "=============================================="

kubectl create namespace $NS 2>/dev/null || true

# Label namespace with pod security standards
kubectl label namespace $NS \
    pod-security.kubernetes.io/enforce=baseline \
    pod-security.kubernetes.io/warn=restricted \
    --overwrite

echo ""
echo "Step 1: RBAC Setup..."
kubectl apply -n $NS -f - << 'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nexus-app-sa
automountServiceAccountToken: false
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: nexus-app-role
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: nexus-app-binding
subjects:
- kind: ServiceAccount
  name: nexus-app-sa
roleRef:
  kind: Role
  name: nexus-app-role
  apiGroup: rbac.authorization.k8s.io
EOF
echo "  ✓ RBAC: ServiceAccount + minimal Role created"

echo ""
echo "Step 2: Network Policies..."
kubectl apply -n $NS -f - << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: UDP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-ingress
spec:
  podSelector:
    matchLabels:
      app: nexus-api
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
EOF
echo "  ✓ NetworkPolicies: deny-all + selective allow"

echo ""
echo "Step 3: Secure Deployment..."
kubectl apply -n $NS -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
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
      serviceAccountName: nexus-app-sa
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: nexus-api
        image: nginx:1.25
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          privileged: false
          capabilities:
            drop: [ALL]
        resources:
          requests: {memory: "64Mi", cpu: "100m"}
          limits: {memory: "128Mi", cpu: "200m"}
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: var-cache
          mountPath: /var/cache/nginx
        - name: var-run
          mountPath: /var/run
      volumes:
      - name: tmp
        emptyDir: {}
      - name: var-cache
        emptyDir: {}
      - name: var-run
        emptyDir: {}
EOF

kubectl rollout status deployment/nexus-api -n $NS
echo "  ✓ Hardened Deployment: non-root, read-only FS, no privilege escalation"

echo ""
echo "=============================================="
echo "  Security Hardening Summary"
echo "=============================================="
echo ""
echo "✓ Namespace: $NS (pod-security: baseline)"
echo "✓ RBAC: minimal ServiceAccount + Role"
echo "✓ NetworkPolicy: deny-all + selective ingress"
echo "✓ Pod: non-root, read-only FS, no capabilities"
echo "✓ Resources: requests + limits enforced"
echo ""
echo "Security checks:"
kubectl auth can-i delete pods -n $NS \
    --as=system:serviceaccount:$NS:nexus-app-sa && \
    echo "  ✗ SA can delete pods (unexpected)" || \
    echo "  ✓ SA cannot delete pods (correct)"

echo ""
echo "Cleanup: kubectl delete namespace $NS"
echo "=============================================="
```

```bash
chmod +x nexus_security_hardening.sh
./nexus_security_hardening.sh
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **RBAC** | Role-Based Access Control — controls who can perform which operations on which resources |
| **Role** | A namespace-scoped set of permissions |
| **ClusterRole** | A cluster-wide set of permissions |
| **RoleBinding** | Assigns a Role to a subject (user, group, ServiceAccount) within a namespace |
| **ClusterRoleBinding** | Assigns a ClusterRole to a subject across the entire cluster |
| **Subject** | The entity receiving permissions: User, Group, or ServiceAccount |
| **ServiceAccount** | A Kubernetes identity for pods — used instead of human user accounts |
| **Least Privilege** | Security principle — grant only the minimum permissions required |
| **NetworkPolicy** | Kubernetes object that controls pod-to-pod and pod-to-external traffic |
| **Admission Controller** | API server plugin that validates or modifies requests before they are stored |
| **Pod Security Standards** | Kubernetes built-in security profiles: Privileged, Baseline, Restricted |
| **securityContext** | Pod/container field that configures security settings |
| **runAsNonRoot** | Security setting that prevents running containers as the root user |
| **readOnlyRootFilesystem** | Mounts container's root filesystem as read-only |
| **allowPrivilegeEscalation** | Controls whether a process can gain more privileges than its parent |
| **privileged** | Grants container full access to the host — equivalent to root on the node |
| **capabilities** | Linux kernel capabilities that can be selectively granted or dropped |
| **seccompProfile** | Restricts system calls a container can make |
| **Encryption at rest** | Encrypting Secrets in etcd — requires explicit cluster configuration |
| **External Secrets Operator** | Syncs secrets from external managers (Vault, AWS) to Kubernetes Secrets |
| **mTLS** | Mutual TLS — both sides verify each other's certificates |
| **Istio** | A service mesh that provides automatic mTLS between pods |
| **Audit Policy** | Rules defining which API requests are logged and at what detail level |
| **Audit Log** | A record of all API server requests — who did what and when |
| **Prometheus** | Open-source monitoring system that collects metrics from Kubernetes |
| **Grafana** | Visualization platform for metrics — displays Prometheus data as dashboards |
| **Node taint** | A property on a node that repels pods without matching tolerations |
| **`kubectl auth can-i`** | Command to test whether a user/SA has a specific permission |
| **`automountServiceAccountToken`** | Controls whether the SA token is auto-mounted into pods |

---

## 📚 Resources

- [Kubernetes Security Official Documentation](https://kubernetes.io/docs/concepts/security/) — complete security reference
- [Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) — RBAC configuration guide
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) — Privileged, Baseline, Restricted profiles
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) — controlling pod communication
- [Kubernetes Security Best Practices — CNCF Blog](https://www.cncf.io/blog/2019/01/14/9-kubernetes-security-best-practices-everyone-must-follow/) — community best practices
- [Encrypting Secret Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) — etcd encryption guide
- [External Secrets Operator](https://external-secrets.io/latest/) — external secret management
- [kube-bench](https://github.com/aquasecurity/kube-bench) — CIS Kubernetes Benchmark automated checker

---

## 🔭 Day 46 Preview: Helm — The Kubernetes Package Manager

Today you hardened your cluster with security best practices. Tomorrow you learn how to package, share, and deploy complex Kubernetes applications with a single command using **Helm**.

You will learn:
- What Helm is and why it exists (the problem with raw YAML at scale)
- Helm Charts — packages of pre-configured Kubernetes resources
- `helm install`, `helm upgrade`, `helm rollback`
- Writing your own Helm chart for the NexusCorp application
- Helm values — parameterize your deployments for different environments
- Popular community charts: nginx-ingress, cert-manager, prometheus-stack

Helm is the `apt` or `npm` of Kubernetes — it dramatically simplifies complex deployments.

---

> 💬 *"Security is not a feature you add at the end — it is the foundation you build on from the beginning. In Kubernetes, every default is a choice. Know what those choices mean."*
>
> — DevOps Learning by Yukta
