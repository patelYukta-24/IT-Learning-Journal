# Day 42: Kubernetes ConfigMaps & Secrets

> **DevOps Transformation** | Kubernetes & Container Orchestration Series

---

## 📋 Overview

In the previous sessions you deployed applications to Kubernetes with hardcoded configuration. That works for demos — but in production, the same Docker image must run in development, staging, and production with different database URLs, log levels, and API keys. Rebuilding the image for each environment is slow, error-prone, and defeats the purpose of containerization.

**ConfigMaps** and **Secrets** solve this. They externalize configuration from the image, letting the same container run anywhere — with the right configuration injected at runtime by Kubernetes.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| ConfigMaps | Store non-sensitive config as key-value pairs |
| Secrets | Store sensitive data (passwords, tokens, keys) |
| ConfigMap creation methods | Literal, from file, from directory, YAML |
| Injecting ConfigMaps | As env vars, as files (volume mount) |
| Injecting Secrets | As env vars, as mounted files |
| Secret encoding vs encryption | Base64 encoding limitations and best practices |
| Tasks | Create ConfigMap, use Secrets, mount as volumes |

---

## 🏢 NexusCorp Scenario

> NexusCorp's production deployment broke when a developer accidentally committed a database password directly in the Dockerfile. The password was rotated, the image rebuilt, and an hour was lost. After this incident, the team mandated: no configuration or credentials in container images. All config goes in ConfigMaps. All secrets go in Kubernetes Secrets — managed separately from application code.
>
> Today you build the system that enforces this standard.

---

## Part 1: Why ConfigMaps and Secrets?

### The Problem with Hardcoded Configuration

```
Without ConfigMaps/Secrets:

Dockerfile:
  ENV DATABASE_URL=postgresql://prod-db:5432/nexus
  ENV LOG_LEVEL=WARNING
  ENV API_KEY=sk-1234567890abcdef

Problems:
  ❌ Same image cannot run in dev (different DB URL)
  ❌ Password visible in Dockerfile history
  ❌ Rotating a password requires rebuilding the image
  ❌ Any image pull = credentials exposed
  ❌ Can't use different log levels per environment
```

```
With ConfigMaps and Secrets:

Image: nexuscorp/nexus-api:v2 (no config baked in)
  ↓
  At runtime, Kubernetes injects:
  ├── ConfigMap: database host, log level, port numbers
  └── Secret: database password, API key, TLS cert

Benefits:
  ✅ Same image runs in dev, staging, and production
  ✅ Rotate secrets without rebuilding the image
  ✅ Credentials never visible in image layers
  ✅ Different config per namespace/environment
  ✅ Access control on who can read secrets
```

---

## Part 2: ConfigMaps

### What Is a ConfigMap?

A **ConfigMap** stores non-confidential data as key-value pairs. The data can be individual values (strings, numbers) or entire file contents (config files, scripts).

```
ConfigMap: nexus-config
  ├── APP_ENV = "staging"
  ├── LOG_LEVEL = "INFO"
  ├── DB_HOST = "db.nexus-staging.svc.cluster.local"
  ├── DB_PORT = "5432"
  ├── MAX_CONNECTIONS = "100"
  └── nginx.conf = "server { listen 80; ... }"   ← entire file content
```

---

### Creating ConfigMaps

#### Method 1: YAML Manifest (Recommended)

```yaml
# nexus-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-config
  namespace: nexus-production
  labels:
    app: nexus-api
    managed-by: devops-team
data:
  # Simple key-value pairs
  APP_ENV: "production"
  APP_NAME: "NexusCorp API"
  LOG_LEVEL: "WARNING"
  DB_HOST: "nexus-db-service"
  DB_PORT: "5432"
  DB_NAME: "nexusdb"
  MAX_CONNECTIONS: "100"
  CACHE_TTL: "3600"

  # Multi-line value (entire config file)
  nginx.conf: |
    server {
        listen 80;
        server_name nexuscorp.com;

        location / {
            proxy_pass http://nexus-api-service:5000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /health {
            access_log off;
            return 200 "healthy\n";
        }
    }

  # Another file content
  app-config.json: |
    {
      "environment": "production",
      "features": {
        "dark_mode": true,
        "beta_features": false
      },
      "limits": {
        "max_requests_per_minute": 1000
      }
    }
```

```bash
kubectl apply -f nexus-configmap.yaml
```

---

#### Method 2: Imperative — Literal Values

```bash
# From literal key=value pairs
kubectl create configmap nexus-config \
    --from-literal=APP_ENV=production \
    --from-literal=LOG_LEVEL=WARNING \
    --from-literal=DB_HOST=nexus-db-service \
    --from-literal=DB_PORT=5432 \
    -n nexus-production
```

---

#### Method 3: From a File

```bash
# Create config files
cat > app.properties << 'EOF'
APP_ENV=production
LOG_LEVEL=WARNING
DB_HOST=nexus-db-service
DB_PORT=5432
MAX_CONNECTIONS=100
EOF

cat > nginx.conf << 'EOF'
server {
    listen 80;
    location / {
        proxy_pass http://nexus-api-service:5000;
    }
}
EOF

# Create ConfigMap from files
# Key = filename, Value = file content
kubectl create configmap nexus-config \
    --from-file=app.properties \
    --from-file=nginx.conf \
    -n nexus-production

# Or specify a custom key name
kubectl create configmap nexus-config \
    --from-file=config=app.properties \
    -n nexus-production
```

---

#### Method 4: From a Directory

```bash
# Create a directory with multiple config files
mkdir config-files
echo "production" > config-files/environment
echo "WARNING" > config-files/log-level
cat > config-files/features.json << 'EOF'
{"dark_mode": true, "beta": false}
EOF

# All files in directory become entries in the ConfigMap
kubectl create configmap nexus-config \
    --from-file=config-files/ \
    -n nexus-production
```

---

### Viewing ConfigMaps

```bash
# List all ConfigMaps
kubectl get configmaps -n nexus-production
kubectl get cm -n nexus-production              # Short form

# View a ConfigMap's data
kubectl get configmap nexus-config -n nexus-production -o yaml

# Describe a ConfigMap
kubectl describe configmap nexus-config -n nexus-production

# Get a specific key value
kubectl get configmap nexus-config -n nexus-production \
    -o jsonpath='{.data.APP_ENV}'
```

---

### Using ConfigMaps in Pods

#### Method 1: As Environment Variables (Individual Keys)

```yaml
# pod-with-configmap-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-pod
  namespace: nexus-production
spec:
  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v2

    env:
    # Individual key from ConfigMap
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: nexus-config        # ConfigMap name
          key: APP_ENV              # Key within ConfigMap

    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: nexus-config
          key: LOG_LEVEL

    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: nexus-config
          key: DB_HOST
```

#### Method 2: All Keys as Environment Variables

```yaml
spec:
  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v2

    # Load ALL key-value pairs as environment variables
    envFrom:
    - configMapRef:
        name: nexus-config    # All keys become env vars
```

#### Method 3: As Mounted Files (Volume)

```yaml
# pod-with-configmap-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nexus-nginx-pod
  namespace: nexus-production
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: nginx-config-volume
      mountPath: /etc/nginx/conf.d    # Mounted directory
      readOnly: true

    - name: app-config-volume
      mountPath: /app/config           # Single file mount
      readOnly: true

  volumes:
  # Mount entire ConfigMap as directory
  - name: nginx-config-volume
    configMap:
      name: nexus-config
      items:                          # Only mount specific keys as files
      - key: nginx.conf
        path: default.conf            # File name inside the directory

  # Mount a specific key as a single file
  - name: app-config-volume
    configMap:
      name: nexus-config
      items:
      - key: app-config.json
        path: config.json
```

**What this creates on disk:**
```
/etc/nginx/conf.d/
  └── default.conf        ← content of nexus-config.nginx.conf

/app/config/
  └── config.json         ← content of nexus-config.app-config.json
```

**Key advantage of volume mounts:** When the ConfigMap is updated, Kubernetes automatically updates the files in the pod — **no pod restart needed** (takes ~1 minute to propagate). Environment variables are NOT updated automatically — they require a pod restart.

---

## Part 3: Secrets

### What Is a Secret?

A **Secret** is like a ConfigMap but intended for sensitive data — passwords, OAuth tokens, SSH keys, TLS certificates. Secrets are:

- **Base64 encoded** (not encrypted by default in etcd)
- Access-controlled via RBAC
- Not shown in `kubectl describe` output (replaced with `***`)
- Can be encrypted at rest with additional configuration

```
⚠️ Important: Base64 encoding ≠ encryption

echo -n "my-password" | base64
bXktcGFzc3dvcmQ=

echo -n "bXktcGFzc3dvcmQ=" | base64 -d
my-password

Base64 encoding is reversible by anyone with the value!
Kubernetes Secrets are NOT encrypted by default in etcd.
For production: enable etcd encryption at rest,
or use external secret managers (Vault, AWS Secrets Manager).
```

---

### Secret Types

| Type | Use Case | Example |
|---|---|---|
| `Opaque` | Arbitrary user-defined data | Passwords, API keys |
| `kubernetes.io/tls` | TLS certificates | HTTPS certificates |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials | Pull from private registry |
| `kubernetes.io/service-account-token` | Service account tokens | Pod authentication |
| `kubernetes.io/ssh-auth` | SSH private keys | Git SSH access |
| `kubernetes.io/basic-auth` | Basic authentication | Username + password |

---

### Creating Secrets

#### Method 1: YAML Manifest (Base64 encoded)

```bash
# First, base64 encode your values
echo -n "nexus-db-password-123" | base64
# bmV4dXMtZGItcGFzc3dvcmQtMTIz

echo -n "sk-nexus-api-key-abcdef" | base64
# c2stbmV4dXMtYXBpLWtleS1hYmNkZWY=

echo -n "nexususer" | base64
# bmV4dXN1c2Vy
```

```yaml
# nexus-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: nexus-secrets
  namespace: nexus-production
  labels:
    app: nexus-api
type: Opaque
data:
  # Values MUST be base64 encoded
  DB_PASSWORD: bmV4dXMtZGItcGFzc3dvcmQtMTIz
  DB_USERNAME: bmV4dXN1c2Vy
  API_KEY: c2stbmV4dXMtYXBpLWtleS1hYmNkZWY=
  JWT_SECRET: anduLXNlY3JldC1rZXktMjAyNA==

# Alternative: use stringData (plain text — Kubernetes encodes automatically)
stringData:
  REDIS_PASSWORD: "redis-secret-456"     # Plain text — auto-encoded
  SMTP_PASSWORD: "smtp-pass-789"
```

```bash
kubectl apply -f nexus-secret.yaml
```

#### Method 2: Imperative — From Literal (Recommended for Secrets)

```bash
# Kubernetes automatically base64 encodes the values
kubectl create secret generic nexus-secrets \
    --from-literal=DB_PASSWORD=nexus-db-password-123 \
    --from-literal=DB_USERNAME=nexususer \
    --from-literal=API_KEY=sk-nexus-api-key-abcdef \
    -n nexus-production
```

#### Method 3: From Files

```bash
# Store secrets in files (don't commit these files!)
echo -n "nexus-db-password-123" > db-password.txt
echo -n "nexususer" > db-username.txt

kubectl create secret generic nexus-db-creds \
    --from-file=DB_PASSWORD=db-password.txt \
    --from-file=DB_USERNAME=db-username.txt \
    -n nexus-production

# Clean up files immediately
rm db-password.txt db-username.txt
```

#### Method 4: TLS Secret

```bash
# Generate a self-signed certificate (for testing)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout tls.key \
    -out tls.crt \
    -subj "/CN=nexuscorp.com/O=NexusCorp"

# Create TLS secret
kubectl create secret tls nexus-tls-cert \
    --cert=tls.crt \
    --key=tls.key \
    -n nexus-production

# Clean up cert files if sensitive
rm tls.key tls.crt
```

#### Method 5: Docker Registry Secret

```bash
kubectl create secret docker-registry nexus-registry-creds \
    --docker-server=registry.nexuscorp.com \
    --docker-username=nexus-ci \
    --docker-password=docker-access-token \
    --docker-email=devops@nexuscorp.com \
    -n nexus-production

# Use it to pull from private registry
# Add to pod spec:
# imagePullSecrets:
# - name: nexus-registry-creds
```

---

### Viewing Secrets

```bash
# List secrets (values are NOT shown)
kubectl get secrets -n nexus-production

# Describe a secret (keys shown, values hidden)
kubectl describe secret nexus-secrets -n nexus-production
# Data
# ====
# API_KEY:      38 bytes
# DB_PASSWORD:  24 bytes
# DB_USERNAME:  9 bytes

# View secret YAML (values are base64 encoded)
kubectl get secret nexus-secrets -n nexus-production -o yaml

# Decode a specific secret value
kubectl get secret nexus-secrets -n nexus-production \
    -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
# nexus-db-password-123

# All values decoded
kubectl get secret nexus-secrets -n nexus-production \
    -o jsonpath='{.data}' | \
    python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
for k, v in data.items():
    print(f'{k}: {base64.b64decode(v).decode()}')
"
```

---

### Using Secrets in Pods

#### Method 1: As Environment Variables (Individual Keys)

```yaml
# pod-with-secret-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-secure
  namespace: nexus-production
spec:
  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v2

    env:
    # Individual secret key as environment variable
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: nexus-secrets       # Secret name
          key: DB_PASSWORD          # Key within secret
          optional: false           # Fail if not found

    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: nexus-secrets
          key: DB_USERNAME

    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: nexus-secrets
          key: API_KEY

    # Combine ConfigMap and Secrets
    envFrom:
    - configMapRef:
        name: nexus-config          # Non-sensitive config
    - secretRef:
        name: nexus-secrets         # Sensitive config
```

#### Method 2: As Mounted Files (More Secure)

```yaml
# pod-with-secret-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nexus-api-vault
  namespace: nexus-production
spec:
  containers:
  - name: nexus-api
    image: nexuscorp/nexus-api:v2

    volumeMounts:
    # Secrets mounted as files in a directory
    - name: secrets-volume
      mountPath: /run/secrets       # Standard Linux secrets location
      readOnly: true

  volumes:
  - name: secrets-volume
    secret:
      secretName: nexus-secrets
      defaultMode: 0400             # Read-only for owner only (very restrictive)
      items:                        # Mount specific keys only
      - key: DB_PASSWORD
        path: db-password           # File: /run/secrets/db-password
      - key: API_KEY
        path: api-key               # File: /run/secrets/api-key
```

**Result inside the container:**
```
/run/secrets/
├── db-password    ← contains: nexus-db-password-123
└── api-key        ← contains: sk-nexus-api-key-abcdef
```

**Read in application:**
```python
# Python: reading secrets from files
with open('/run/secrets/db-password', 'r') as f:
    db_password = f.read().strip()

with open('/run/secrets/api-key', 'r') as f:
    api_key = f.read().strip()
```

**Why file mounts are more secure than environment variables:**
```
Environment variables:
  ❌ Visible in process listing (ps aux, /proc/environ)
  ❌ Often logged by crash reporters
  ❌ Passed to child processes automatically
  ❌ Accessible to all code running in the container

File mounts:
  ✅ Only accessible to processes that open the file
  ✅ Can have strict permissions (0400)
  ✅ Not visible in process environment
  ✅ Updated automatically when secret rotates
  ✅ Easier to audit file access
```

#### Method 3: Using Secrets as imagePullSecrets

```yaml
spec:
  imagePullSecrets:
  - name: nexus-registry-creds    # Docker registry secret

  containers:
  - name: nexus-api
    image: registry.nexuscorp.com/nexus-api:v2   # Private registry
```

---

## Part 4: Complete Example — ConfigMap + Secret Together

```yaml
# nexus-complete-config.yaml
---
# Non-sensitive configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-app-config
  namespace: nexus-production
data:
  APP_ENV: "production"
  APP_NAME: "NexusCorp API"
  LOG_LEVEL: "WARNING"
  DB_HOST: "nexus-db-service"
  DB_PORT: "5432"
  DB_NAME: "nexusdb"
  REDIS_HOST: "nexus-redis-service"
  REDIS_PORT: "6379"
  MAX_CONNECTIONS: "100"

---
# Sensitive configuration
apiVersion: v1
kind: Secret
metadata:
  name: nexus-app-secrets
  namespace: nexus-production
type: Opaque
stringData:                   # stringData = plain text, auto-encoded
  DB_USERNAME: "nexususer"
  DB_PASSWORD: "super-secret-db-password"
  REDIS_PASSWORD: "redis-secret-456"
  JWT_SECRET: "jwt-signing-key-very-long-random-string"
  API_KEY: "sk-nexus-production-api-key"

---
# Deployment using both
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus-api
  namespace: nexus-production
spec:
  replicas: 3
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

        # Load all ConfigMap keys as env vars
        envFrom:
        - configMapRef:
            name: nexus-app-config

        # Load specific secret keys as env vars
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: nexus-app-secrets
              key: DB_USERNAME
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: nexus-app-secrets
              key: DB_PASSWORD
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: nexus-app-secrets
              key: JWT_SECRET

        # Mount sensitive files separately
        volumeMounts:
        - name: secret-files
          mountPath: /run/secrets
          readOnly: true

        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"

      volumes:
      - name: secret-files
        secret:
          secretName: nexus-app-secrets
          defaultMode: 0400
          items:
          - key: API_KEY
            path: api-key
          - key: JWT_SECRET
            path: jwt-secret
```

---

## Part 5: Practical Tasks

### Task 1: Creating and Using a ConfigMap

```bash
# Step 1: Create namespace
kubectl create namespace nexus-config-demo

# Step 2: Create a ConfigMap with multiple data types
cat > app-configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-demo-config
  namespace: nexus-config-demo
data:
  APP_ENV: "development"
  LOG_LEVEL: "DEBUG"
  DB_HOST: "localhost"
  DB_PORT: "5432"
  MAX_RETRIES: "3"
  ENABLE_METRICS: "true"

  # A multi-line config file
  app.conf: |
    [server]
    host = 0.0.0.0
    port = 5000
    workers = 4

    [database]
    host = localhost
    port = 5432
    pool_size = 10

    [logging]
    level = DEBUG
    format = json
EOF

kubectl apply -f app-configmap.yaml
kubectl describe configmap nexus-demo-config -n nexus-config-demo

# Step 3: Create a pod that uses the ConfigMap
cat > pod-configmap.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nexus-config-test
  namespace: nexus-config-demo
spec:
  containers:
  - name: test-app
    image: busybox:latest
    command: ['sh', '-c',
              'echo "Environment Variables:";
               echo "  APP_ENV=$APP_ENV";
               echo "  LOG_LEVEL=$LOG_LEVEL";
               echo "  DB_HOST=$DB_HOST";
               echo "  DB_PORT=$DB_PORT";
               echo "";
               echo "Config file content:";
               cat /etc/app/app.conf;
               sleep 3600']
    envFrom:
    - configMapRef:
        name: nexus-demo-config
    volumeMounts:
    - name: config-volume
      mountPath: /etc/app
  volumes:
  - name: config-volume
    configMap:
      name: nexus-demo-config
      items:
      - key: app.conf
        path: app.conf
EOF

kubectl apply -f pod-configmap.yaml
kubectl logs nexus-config-test -n nexus-config-demo

# Step 4: Verify config file was mounted
kubectl exec -it nexus-config-test -n nexus-config-demo -- \
    cat /etc/app/app.conf

# Step 5: Update the ConfigMap and observe
kubectl patch configmap nexus-demo-config \
    -n nexus-config-demo \
    --type=merge \
    -p '{"data": {"LOG_LEVEL": "WARNING"}}'

# Volume-mounted files update automatically (~1 min)
# Env vars do NOT update — pod needs restart
kubectl rollout restart deployment/nexus-api -n nexus-config-demo 2>/dev/null || true
```

---

### Task 2: Using Secrets as Environment Variables

```bash
# Step 1: Create a secret
kubectl create secret generic nexus-demo-secrets \
    --from-literal=DB_PASSWORD=my-secret-password \
    --from-literal=API_KEY=sk-test-api-key-12345 \
    --from-literal=JWT_SECRET=super-secret-jwt-key \
    -n nexus-config-demo

# Verify (values are hidden)
kubectl describe secret nexus-demo-secrets -n nexus-config-demo

# Step 2: Create a pod that uses the secret
cat > pod-secret-env.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nexus-secret-env-test
  namespace: nexus-config-demo
spec:
  containers:
  - name: test-app
    image: busybox:latest
    command: ['sh', '-c',
              'echo "=== Secret Environment Variables ===";
               echo "DB_PASSWORD length: ${#DB_PASSWORD}";
               echo "DB_PASSWORD first 3 chars: ${DB_PASSWORD:0:3}***";
               echo "API_KEY is set: $([ -n "$API_KEY" ] && echo yes || echo no)";
               echo "JWT_SECRET length: ${#JWT_SECRET}";
               echo "=== App would use these to connect to DB ===";
               echo "Connecting as user: admin";
               echo "Using password: [REDACTED]";
               sleep 3600']
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: nexus-demo-secrets
          key: DB_PASSWORD
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: nexus-demo-secrets
          key: API_KEY
    - name: JWT_SECRET
      valueFrom:
        secretKeyRef:
          name: nexus-demo-secrets
          key: JWT_SECRET
EOF

kubectl apply -f pod-secret-env.yaml
kubectl logs nexus-secret-env-test -n nexus-config-demo

# Verify the secret values are accessible but not plainly visible
kubectl exec -it nexus-secret-env-test -n nexus-config-demo -- \
    sh -c 'echo "Password length: ${#DB_PASSWORD}"'
```

---

### Task 3: Secrets via Volume Mounts

```bash
# Step 1: Create a pod that mounts secrets as files
cat > pod-secret-volume.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: nexus-secret-volume-test
  namespace: nexus-config-demo
spec:
  containers:
  - name: test-app
    image: busybox:latest
    command: ['sh', '-c',
              'echo "=== Files in /run/secrets ===";
               ls -la /run/secrets/;
               echo "";
               echo "=== File permissions ===";
               stat /run/secrets/db-password;
               echo "";
               echo "=== Reading secret values ===";
               echo "DB_PASSWORD: $(cat /run/secrets/db-password | wc -c) characters";
               echo "API_KEY first 5: $(cut -c1-5 /run/secrets/api-key)***";
               echo "";
               echo "Simulating database connection:";
               DB_PASS=$(cat /run/secrets/db-password)
               echo "Connected with password of length: ${#DB_PASS}";
               sleep 3600']
    volumeMounts:
    - name: secrets-volume
      mountPath: /run/secrets
      readOnly: true
  volumes:
  - name: secrets-volume
    secret:
      secretName: nexus-demo-secrets
      defaultMode: 0400             # -r-------- (owner read only)
      items:
      - key: DB_PASSWORD
        path: db-password
      - key: API_KEY
        path: api-key
EOF

kubectl apply -f pod-secret-volume.yaml
kubectl logs nexus-secret-volume-test -n nexus-config-demo

# Verify file contents and permissions
kubectl exec -it nexus-secret-volume-test -n nexus-config-demo -- \
    ls -la /run/secrets/

# Try to read the secret file
kubectl exec -it nexus-secret-volume-test -n nexus-config-demo -- \
    cat /run/secrets/db-password

# Show that the actual secret is accessible from the file
kubectl exec -it nexus-secret-volume-test -n nexus-config-demo -- \
    sh -c 'DB_PASS=$(cat /run/secrets/db-password); echo "Password: $DB_PASS"'
```

---

## Part 6: Best Practices

### ConfigMap Best Practices

```
✅ DO:
├── Use ConfigMaps for environment-specific values
│   (database hosts, ports, log levels, feature flags)
├── Use descriptive key names that match env variable conventions
├── Store ConfigMaps in version control as YAML
├── Use namespace-specific ConfigMaps (one per environment)
├── Reference ConfigMaps by name in pods — update ConfigMap
│   without changing pod spec
└── Use volume mounts for large configs or config files

❌ DON'T:
├── Store passwords, tokens, or keys in ConfigMaps
├── Store more than 1MiB of data (etcd limit)
└── Put configuration that rarely changes in ConfigMaps
    (bake stable config into the image instead)
```

### Secret Best Practices

```
✅ DO:
├── Use Secrets for ALL sensitive data (no exceptions)
├── Use volume mounts instead of environment variables
│   for sensitive data where possible
├── Enable etcd encryption at rest in production clusters
├── Use external secret managers in production:
│   AWS Secrets Manager + External Secrets Operator
│   HashiCorp Vault + Vault Agent Injector
│   Google Secret Manager
├── Rotate secrets regularly without redeploying pods
├── Use RBAC to restrict who can read secrets
├── Use specific secretKeyRef (not secretRef for all keys)
│   to minimize exposure
└── Audit secret access in production

❌ DON'T:
├── Commit Secret YAML files to version control
│   (they contain base64-encoded secrets!)
├── Assume base64 = encrypted (it does NOT)
├── Share secrets between namespaces unnecessarily
├── Use the same secret for multiple applications
│   (rotate one without affecting others)
└── Log environment variables that contain secrets
```

### Secrets at Rest Encryption

```bash
# Check if encryption at rest is enabled
kubectl get apiserver -o yaml | grep -A5 encryption

# Enable encryption (requires cluster admin):
# /etc/kubernetes/encryption-config.yaml:
# apiVersion: apiserver.config.k8s.io/v1
# kind: EncryptionConfiguration
# resources:
# - resources:
#   - secrets
#   providers:
#   - aescbc:
#       keys:
#       - name: key1
#         secret: <base64-encoded-32-byte-key>
#   - identity: {}   # fallback
```

---

## 🔧 Mini-Project: NexusCorp Configuration Management System

```bash
#!/bin/bash
# nexus_config_setup.sh
# Set up NexusCorp's complete configuration and secrets management

set -e
NAMESPACE="nexus-config-prod"

echo "=============================================="
echo "  NexusCorp Config Management Setup"
echo "=============================================="

# Create namespace
kubectl create namespace $NAMESPACE 2>/dev/null || true

# ── Step 1: Create ConfigMaps ───────────────────────────────────
echo ""
echo "Creating ConfigMaps..."

kubectl apply -n $NAMESPACE -f - << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-app-config
data:
  APP_NAME: "NexusCorp Production API"
  APP_ENV: "production"
  LOG_LEVEL: "WARNING"
  DB_HOST: "nexus-postgres.nexus-config-prod.svc.cluster.local"
  DB_PORT: "5432"
  DB_NAME: "nexusdb"
  REDIS_HOST: "nexus-redis.nexus-config-prod.svc.cluster.local"
  REDIS_PORT: "6379"
  MAX_POOL_SIZE: "20"
  REQUEST_TIMEOUT: "30"
  ENABLE_METRICS: "true"
  METRICS_PORT: "9090"
EOF
echo "  ✓ App ConfigMap created"

# ── Step 2: Create Secrets ──────────────────────────────────────
echo ""
echo "Creating Secrets..."

kubectl create secret generic nexus-db-secret \
    --from-literal=DB_USERNAME=nexusadmin \
    --from-literal=DB_PASSWORD=NexusSecure#2024 \
    -n $NAMESPACE \
    2>/dev/null || \
kubectl create secret generic nexus-db-secret \
    --from-literal=DB_USERNAME=nexusadmin \
    --from-literal=DB_PASSWORD=NexusSecure#2024 \
    -n $NAMESPACE \
    --dry-run=client -o yaml | kubectl apply -f -
echo "  ✓ Database secret created"

kubectl create secret generic nexus-api-secret \
    --from-literal=JWT_SECRET=nexus-jwt-signing-key-change-in-production \
    --from-literal=API_KEY=sk-nexus-prod-api-key \
    --from-literal=REDIS_PASSWORD=redis-secure-pass \
    -n $NAMESPACE \
    2>/dev/null || true
echo "  ✓ API secrets created"

# ── Step 3: Deploy app using all config ─────────────────────────
echo ""
echo "Deploying application..."

kubectl apply -n $NAMESPACE -f - << 'EOF'
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
      containers:
      - name: nexus-api
        image: nginx:1.25
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: nexus-app-config
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: nexus-db-secret
              key: DB_USERNAME
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: nexus-db-secret
              key: DB_PASSWORD
        volumeMounts:
        - name: api-secrets
          mountPath: /run/secrets
          readOnly: true
        resources:
          requests: {memory: "128Mi", cpu: "200m"}
          limits: {memory: "256Mi", cpu: "400m"}
      volumes:
      - name: api-secrets
        secret:
          secretName: nexus-api-secret
          defaultMode: 0400
EOF

kubectl rollout status deployment/nexus-api -n $NAMESPACE
echo "  ✓ Application deployed"

# ── Step 4: Verify ──────────────────────────────────────────────
echo ""
echo "=============================================="
echo "  Verification"
echo "=============================================="
echo ""
echo "ConfigMaps:"
kubectl get configmaps -n $NAMESPACE

echo ""
echo "Secrets:"
kubectl get secrets -n $NAMESPACE

echo ""
echo "Pods:"
kubectl get pods -n $NAMESPACE

POD=$(kubectl get pod -n $NAMESPACE -l app=nexus-api \
    -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)

if [ -n "$POD" ]; then
    echo ""
    echo "Environment in pod (non-sensitive):"
    kubectl exec $POD -n $NAMESPACE -- \
        sh -c 'echo "APP_ENV=$APP_ENV, LOG_LEVEL=$LOG_LEVEL, DB_HOST=$DB_HOST"'

    echo ""
    echo "Secret files mounted:"
    kubectl exec $POD -n $NAMESPACE -- ls -la /run/secrets/
fi

echo ""
echo "=============================================="
echo "  Setup complete!"
echo ""
echo "  To clean up:"
echo "  kubectl delete namespace $NAMESPACE"
echo "=============================================="
```

```bash
chmod +x nexus_config_setup.sh
./nexus_config_setup.sh
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **ConfigMap** | A Kubernetes object for storing non-sensitive configuration as key-value pairs |
| **Secret** | A Kubernetes object for storing sensitive data — passwords, tokens, certificates |
| **`Opaque`** | The default Secret type for arbitrary user-defined key-value data |
| **Base64 encoding** | A reversible text encoding scheme used to store Secret values — NOT encryption |
| **`stringData`** | A Secret field that accepts plain text values and auto-encodes them to base64 |
| **`data`** | A Secret/ConfigMap field that accepts base64-encoded values directly |
| **`configMapRef`** | Pod spec field that loads all ConfigMap keys as environment variables (`envFrom`) |
| **`secretRef`** | Pod spec field that loads all Secret keys as environment variables (`envFrom`) |
| **`configMapKeyRef`** | Pod spec field that loads a single ConfigMap key as an environment variable |
| **`secretKeyRef`** | Pod spec field that loads a single Secret key as an environment variable |
| **Volume mount** | Mounting ConfigMap or Secret data as files inside a container |
| **`defaultMode`** | Octal permission bits for Secret files when mounted as a volume |
| **`items`** | Volume field that selects specific keys to mount and renames them as files |
| **`optional`** | Field that controls whether missing ConfigMap/Secret causes pod failure |
| **Encryption at rest** | Encrypting Secret data in etcd — requires explicit cluster configuration |
| **RBAC** | Role-Based Access Control — controls who can read/write Secrets |
| **External Secrets Operator** | A Kubernetes operator that syncs secrets from external vaults to K8s Secrets |
| **imagePullSecrets** | Pod spec field referencing a Secret containing Docker registry credentials |
| **TLS Secret** | A Secret of type `kubernetes.io/tls` storing a TLS certificate and private key |
| **`envFrom`** | Pod spec field that loads all keys from a ConfigMap or Secret as environment variables |
| **Secret rotation** | Updating a Secret value — volume-mounted files update automatically |
| **`kubectl create secret`** | Imperative command to create a Secret (Kubernetes auto-encodes values) |
| **`kubectl create configmap`** | Imperative command to create a ConfigMap |
| **`--from-literal`** | Flag to provide key=value pairs directly on the command line |
| **`--from-file`** | Flag to create a ConfigMap/Secret key from a file's contents |
| **`--dry-run=client`** | Generates the YAML without applying — useful for piping to `kubectl apply` |

---

## 📚 Resources

- [ConfigMaps — Official Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/) — complete ConfigMap reference
- [Secrets — Official Documentation](https://kubernetes.io/docs/concepts/configuration/secret/) — complete Secret reference with all types
- [Secrets Best Practices — Official Documentation](https://kubernetes.io/docs/concepts/security/secrets-good-practices/) — official security guidance
- [Encrypting Secret Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) — enable etcd encryption
- [External Secrets Operator](https://external-secrets.io) — sync secrets from AWS/GCP/Vault to Kubernetes
- [HashiCorp Vault + Kubernetes](https://developer.hashicorp.com/vault/docs/platform/k8s) — enterprise secret management

---

## 🔭 Day 43 Preview: Kubernetes Storage — PersistentVolumes & StatefulSets

Today you externalized configuration with ConfigMaps and Secrets. Tomorrow you solve the data persistence challenge — how Kubernetes handles storage for stateful applications like databases.

You will learn:
- PersistentVolumes (PV) — cluster-level storage resources
- PersistentVolumeClaims (PVC) — how pods request storage
- StorageClasses — dynamic volume provisioning from cloud providers (AWS EBS, GCP PD)
- StatefulSets — for pods that need stable identities and persistent storage
- Running a PostgreSQL database with persistent storage in Kubernetes
- The difference between stateless (Deployment) and stateful (StatefulSet) workloads

Storage is often the hardest part of Kubernetes — Day 43 makes it concrete and practical.

---

> 💬 *"ConfigMaps and Secrets are Kubernetes's promise: your images are portable, your configuration is not your image. Separate them, and you gain the freedom to run anywhere."*
>
> — DevOps Learning By Yukta
