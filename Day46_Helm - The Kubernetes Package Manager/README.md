# Day 46: Helm — The Kubernetes Package Manager

> **DevOps Transformation** | Kubernetes & Container Orchestration Series

---

## 📋 Overview

You have been writing Kubernetes YAML manifests by hand — one file for the Deployment, one for the Service, one for the ConfigMap, one for the Secret, one for the NetworkPolicy. For a single application, that is five files. For NexusCorp's full stack of twelve microservices across three environments, that is potentially 180+ YAML files to manage, version, and deploy.

**Helm** is the solution. It is the package manager for Kubernetes — like `apt` for Ubuntu, `npm` for Node.js, or `pip` for Python. Helm packages all the Kubernetes resources needed for an application into a single unit called a **Chart**, lets you parameterize it with **Values**, and deploys it to your cluster with a single command.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| What is Helm? | Package manager concept for Kubernetes |
| How Helm Works | Charts, Templates, Values, Releases |
| Why Helm is Useful | The problems it solves at scale |
| Helm Installation | Install on Linux, macOS, Windows |
| Chart Structure | Anatomy of a Helm chart |
| Important Commands | create, install, upgrade, rollback, list, uninstall |
| Creating Your Own Chart | Build a chart from scratch |
| Task | Two-tier application Helm chart |

---

## 🏢 NexusCorp Scenario

> NexusCorp deploys the same application to dev, staging, and production. Currently this means maintaining three sets of near-identical YAML files — only the image tag, replica count, resource limits, and database URL differ. When someone updates the Deployment spec, they must remember to update it in all three environments. Last week, a security fix was applied to dev and staging but forgotten in production.
>
> Helm solves this by having one chart (the template) and three values files (the environment-specific differences). One change to the chart propagates everywhere.

---

## Part 1: What Is Helm?

Helm is a **package manager for Kubernetes**. It solves three problems that arise when managing Kubernetes applications at scale:

```
Problem 1: Too many YAML files
  Without Helm: 8 YAML files per microservice × 12 services = 96 files
  With Helm: 1 chart per microservice

Problem 2: Copy-paste configuration
  Without Helm: Change image tag in dev, staging, prod = 3 edits, easy to miss one
  With Helm: Change in values.yaml = applies everywhere

Problem 3: No deployment history
  Without Helm: How do you know what version is running? When was it deployed?
  With Helm: helm list shows all releases, versions, timestamps
             helm rollback restores any previous version instantly
```

**The analogy:**
```
Operating System Package Manager    Kubernetes Package Manager
─────────────────────────────────   ──────────────────────────
apt install nginx                   helm install my-nginx stable/nginx
                                    
apt upgrade nginx                   helm upgrade my-nginx stable/nginx

apt remove nginx                    helm uninstall my-nginx

dpkg --list                         helm list

Package = .deb file                 Package = Helm Chart
```

---

## Part 2: How Helm Works

### Core Concepts

```
Helm Ecosystem:
─────────────────────────────────────────────────────────

Chart Repository (Artifact Hub, Bitnami, etc.)
  │
  │ helm repo add / helm search
  ▼
Helm Chart (your-app-1.0.0.tgz)
  ├── Chart.yaml          ← Chart metadata (name, version, description)
  ├── values.yaml         ← Default values (overridable)
  ├── templates/          ← Kubernetes YAML templates
  │   ├── deployment.yaml
  │   ├── service.yaml
  │   ├── configmap.yaml
  │   └── _helpers.tpl    ← Reusable template snippets
  └── charts/             ← Sub-charts (dependencies)
      └── postgresql/

  │ helm install my-release ./chart --values prod-values.yaml
  ▼
Release (running instance of a chart)
  ├── Release Name: my-release
  ├── Namespace:    nexus-production
  ├── Revision:     1
  └── Status:       deployed
```

---

### Charts

A **Chart** is a collection of files that describe a related set of Kubernetes resources. It is the installable package — like a `.deb` or `.rpm` file, but for Kubernetes.

A chart can be:
- A simple chart (single application — e.g., nginx)
- A complex chart with dependencies (e.g., WordPress + MySQL)
- A library chart (reusable helpers shared by other charts)

---

### Templates

Templates are Kubernetes YAML files with **Go template syntax** — placeholders that get filled in with values when the chart is installed.

```yaml
# templates/deployment.yaml — Template with placeholders
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}    # Filled at install time
  namespace: {{ .Release.Namespace }}
  labels:
    app: {{ .Values.app.name }}                   # From values.yaml
    version: {{ .Values.image.tag }}
spec:
  replicas: {{ .Values.replicaCount }}            # Configurable
  selector:
    matchLabels:
      app: {{ .Values.app.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.app.name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.targetPort }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

---

### Values

**Values** are the configuration inputs to a chart. They are defined in `values.yaml` with sensible defaults, and can be overridden at install time.

```yaml
# values.yaml — Default values
replicaCount: 2

app:
  name: nexus-api
  environment: production

image:
  repository: nexuscorp/nexus-api
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 5000

resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

ingress:
  enabled: false
  host: ""

database:
  host: "nexus-db-service"
  port: 5432
  name: "nexusdb"
```

---

### Installation and Releases

When you install a chart, Helm:
1. Takes the templates
2. Fills in values (defaults + your overrides)
3. Sends the rendered YAML to the Kubernetes API server
4. Tracks the deployment as a **Release**

```
helm install nexus-production ./nexus-chart \
    --values prod-values.yaml \
    --namespace nexus-production

Creates a Release:
  Name:      nexus-production
  Chart:     nexus-chart-1.0.0
  Status:    deployed
  Revision:  1
  Namespace: nexus-production
```

---

### Upgrade and Rollback

```
Revision History:

Revision 1: nexus-api:v1, replicas=2  (initial install)
Revision 2: nexus-api:v2, replicas=3  (upgrade)
Revision 3: nexus-api:v2, replicas=5  (scale up)

helm rollback nexus-production 2
→ Reverts to Revision 2 (nexus-api:v2, replicas=3)
→ A new Revision 4 is created with Revision 2's config
→ History is never destroyed
```

---

## Part 3: Why Is Helm Useful?

| Benefit | Without Helm | With Helm |
|---|---|---|
| **Installation** | Apply 8+ YAML files manually | `helm install` — one command |
| **Reusability** | Copy-paste YAML, modify per project | Share charts via repositories |
| **Consistency** | Manual YAML = human error | Template + values = always consistent |
| **Multi-environment** | 3 environments = 3 × 8 YAML files | 1 chart + 3 values files |
| **Updates** | Edit YAML, re-apply all files | `helm upgrade` with new values |
| **Rollbacks** | Recreate old YAML, apply manually | `helm rollback` — one command |
| **Dependencies** | Manually install prerequisites | Declared in `Chart.yaml` |
| **History** | No record of what was deployed | Full revision history in Helm |
| **Sharing** | Email YAML files | Push to Artifact Hub |

---

## Part 4: Helm Installation

### Linux

```bash
# Method 1: Official install script (recommended)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Method 2: Manual installation
HELM_VERSION="v3.14.0"
curl -LO "https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz"
tar -zxvf helm-${HELM_VERSION}-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
rm -rf linux-amd64 helm-${HELM_VERSION}-linux-amd64.tar.gz

# Method 3: Package manager (Ubuntu/Debian)
curl https://baltocdn.com/helm/signing.asc | \
    gpg --dearmor | \
    sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https -y
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] \
    https://baltocdn.com/helm/stable/debian/ all main" | \
    sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm -y
```

### macOS

```bash
brew install helm
```

### Windows

```powershell
winget install Helm.Helm
# Or:
choco install kubernetes-helm
```

### Verify Installation

```bash
helm version
# version.BuildInfo{Version:"v3.14.0", ...}

helm env
# Shows Helm environment variables (cache, config, data directories)
```

---

## Part 5: Helm Chart Structure

```bash
# Create a new chart scaffold
helm create nexus-chart

# Generated structure:
nexus-chart/
├── Chart.yaml            ← Chart metadata
├── values.yaml           ← Default configuration values
├── charts/               ← Sub-chart dependencies (empty initially)
├── templates/            ← Kubernetes resource templates
│   ├── NOTES.txt         ← Post-install instructions shown to user
│   ├── _helpers.tpl      ← Reusable Go template helpers
│   ├── deployment.yaml   ← Deployment template
│   ├── service.yaml      ← Service template
│   ├── serviceaccount.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml          ← HorizontalPodAutoscaler template
│   └── tests/
│       └── test-connection.yaml
└── .helmignore           ← Files to ignore when packaging
```

### Chart.yaml — Chart Metadata

```yaml
# Chart.yaml
apiVersion: v2             # Helm 3 uses v2 (Helm 2 used v1)
name: nexus-chart
description: "NexusCorp microservice application chart"
type: application          # application or library

# Chart version (increment when chart changes)
version: 1.0.0

# Application version (the app this chart deploys)
appVersion: "2.0.0"

maintainers:
- name: DevOps Team
  email: devops@nexuscorp.com

keywords:
- nexuscorp
- api
- microservice

# Dependencies (sub-charts)
dependencies:
- name: postgresql
  version: "12.x.x"
  repository: "https://charts.bitnami.com/bitnami"
  condition: postgresql.enabled
```

### _helpers.tpl — Reusable Template Snippets

```
{{/*
Expand the name of the chart.
*/}}
{{- define "nexus-chart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "nexus-chart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{/*
Common labels — applied to all resources
*/}}
{{- define "nexus-chart.labels" -}}
helm.sh/chart: {{ include "nexus-chart.chart" . }}
app.kubernetes.io/name: {{ include "nexus-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```

---

## Part 6: Important Helm Commands

### Repository Management

```bash
# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update repository index
helm repo update

# List configured repositories
helm repo list

# Remove a repository
helm repo remove bitnami

# Search for charts in repositories
helm search repo nginx
helm search repo postgresql --versions    # Show all versions
helm search hub wordpress                 # Search Artifact Hub (public registry)
```

---

### Chart Development

```bash
# Create a new chart scaffold
helm create nexus-chart

# Render templates locally (preview what would be applied)
helm template nexus-release ./nexus-chart
helm template nexus-release ./nexus-chart --values prod-values.yaml

# Lint chart for errors
helm lint ./nexus-chart

# Package chart into .tgz archive
helm package ./nexus-chart
# Creates: nexus-chart-1.0.0.tgz

# Show chart information
helm show chart ./nexus-chart
helm show values ./nexus-chart       # Show default values
helm show all ./nexus-chart          # Show everything
```

---

### Release Management

```bash
# Install a chart (creates a release)
helm install nexus-api ./nexus-chart
helm install nexus-api ./nexus-chart -n nexus-production --create-namespace

# Install with value overrides
helm install nexus-api ./nexus-chart \
    --set image.tag=v2.0 \
    --set replicaCount=3

# Install with values file
helm install nexus-api ./nexus-chart \
    --values prod-values.yaml

# Install with multiple value sources (right overrides left)
helm install nexus-api ./nexus-chart \
    --values base-values.yaml \
    --values prod-values.yaml \
    --set image.tag=v2.1.0

# Dry run (simulate install without applying)
helm install nexus-api ./nexus-chart --dry-run

# Install or upgrade in one command
helm upgrade --install nexus-api ./nexus-chart \
    --values prod-values.yaml \
    -n nexus-production \
    --create-namespace
```

---

### Upgrade and Rollback

```bash
# Upgrade a release
helm upgrade nexus-api ./nexus-chart
helm upgrade nexus-api ./nexus-chart --values prod-values.yaml
helm upgrade nexus-api ./nexus-chart --set image.tag=v3.0

# Upgrade and wait for pods to be ready
helm upgrade nexus-api ./nexus-chart --wait --timeout 5m

# View release history
helm history nexus-api
# REVISION  STATUS    CHART              APP VERSION  DESCRIPTION
# 1         deployed  nexus-chart-1.0.0  2.0.0        Initial install
# 2         deployed  nexus-chart-1.0.0  2.1.0        Upgraded image
# 3         deployed  nexus-chart-1.1.0  2.1.0        Updated chart

# Rollback to previous release
helm rollback nexus-api

# Rollback to specific revision
helm rollback nexus-api 1

# Rollback and wait
helm rollback nexus-api 1 --wait
```

---

### Inspecting Releases

```bash
# List all releases
helm list
helm list -n nexus-production           # In specific namespace
helm list -A                             # All namespaces
helm list --all                          # Including failed/uninstalled

# Get release status
helm status nexus-api
helm status nexus-api -n nexus-production

# Get computed values for a release
helm get values nexus-api               # User-supplied values
helm get values nexus-api --all         # All values including defaults

# Get rendered manifests for a release
helm get manifest nexus-api

# Get release notes
helm get notes nexus-api
```

---

### Uninstall

```bash
# Uninstall a release
helm uninstall nexus-api
helm uninstall nexus-api -n nexus-production

# Uninstall but keep history
helm uninstall nexus-api --keep-history

# Reinstall from history
helm rollback nexus-api 1
```

---

## Part 7: Building Your Own Helm Chart

### Step-by-Step: NexusCorp API Chart

```bash
# Step 1: Create chart scaffold
helm create nexus-api-chart
cd nexus-api-chart

# Step 2: Customize Chart.yaml
cat > Chart.yaml << 'EOF'
apiVersion: v2
name: nexus-api-chart
description: "NexusCorp API microservice"
type: application
version: 1.0.0
appVersion: "2.0.0"
maintainers:
- name: DevOps Team
  email: devops@nexuscorp.com
EOF
```

```yaml
# Step 3: Define values.yaml
cat > values.yaml << 'EOF'
replicaCount: 2

image:
  repository: nexuscorp/nexus-api
  tag: "latest"
  pullPolicy: IfNotPresent

nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80
  targetPort: 5000

ingress:
  enabled: false
  hostname: api.nexuscorp.com

resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

config:
  appEnv: production
  logLevel: WARNING
  dbHost: nexus-db-service
  dbPort: "5432"
  dbName: nexusdb

secrets:
  dbUsername: ""       # Set via --set or external secret
  dbPassword: ""       # Set via --set or external secret

serviceAccount:
  create: true
  name: ""
  automountToken: false

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000

containerSecurityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: [ALL]

nodeSelector: {}
tolerations: []
affinity: {}
EOF
```

```yaml
# Step 4: Write templates/deployment.yaml
cat > templates/deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "nexus-api-chart.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "nexus-api-chart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "nexus-api-chart.selectorLabels" . | nindent 6 }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        {{- include "nexus-api-chart.selectorLabels" . | nindent 8 }}
    spec:
      serviceAccountName: {{ include "nexus-api-chart.serviceAccountName" . }}
      automountServiceAccountToken: {{ .Values.serviceAccount.automountToken }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        securityContext:
          {{- toYaml .Values.containerSecurityContext | nindent 10 }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
        envFrom:
        - configMapRef:
            name: {{ include "nexus-api-chart.fullname" . }}-config
        env:
        {{- if .Values.secrets.dbPassword }}
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ include "nexus-api-chart.fullname" . }}-secrets
              key: DB_PASSWORD
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: {{ include "nexus-api-chart.fullname" . }}-secrets
              key: DB_USERNAME
        {{- end }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        readinessProbe:
          httpGet:
            path: /health
            port: {{ .Values.service.targetPort }}
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: {{ .Values.service.targetPort }}
          initialDelaySeconds: 30
          periodSeconds: 10
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
EOF
```

```yaml
# Step 5: Write templates/service.yaml
cat > templates/service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{ include "nexus-api-chart.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "nexus-api-chart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  selector:
    {{- include "nexus-api-chart.selectorLabels" . | nindent 4 }}
  ports:
  - name: http
    protocol: TCP
    port: {{ .Values.service.port }}
    targetPort: {{ .Values.service.targetPort }}
EOF
```

```yaml
# Step 6: Write templates/configmap.yaml
cat > templates/configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "nexus-api-chart.fullname" . }}-config
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "nexus-api-chart.labels" . | nindent 4 }}
data:
  APP_ENV: {{ .Values.config.appEnv | quote }}
  LOG_LEVEL: {{ .Values.config.logLevel | quote }}
  DB_HOST: {{ .Values.config.dbHost | quote }}
  DB_PORT: {{ .Values.config.dbPort | quote }}
  DB_NAME: {{ .Values.config.dbName | quote }}
EOF
```

```yaml
# Step 7: Write templates/secret.yaml (conditional)
cat > templates/secret.yaml << 'EOF'
{{- if and .Values.secrets.dbPassword .Values.secrets.dbUsername }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "nexus-api-chart.fullname" . }}-secrets
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "nexus-api-chart.labels" . | nindent 4 }}
type: Opaque
stringData:
  DB_PASSWORD: {{ .Values.secrets.dbPassword | quote }}
  DB_USERNAME: {{ .Values.secrets.dbUsername | quote }}
{{- end }}
EOF
```

```
# Step 8: Write templates/NOTES.txt (post-install message)
cat > templates/NOTES.txt << 'EOF'
========================================
  NexusCorp API Chart Deployed!
========================================

Release:   {{ .Release.Name }}
Namespace: {{ .Release.Namespace }}
Chart:     {{ .Chart.Name }}-{{ .Chart.Version }}
Image:     {{ .Values.image.repository }}:{{ .Values.image.tag }}
Replicas:  {{ .Values.replicaCount }}

To access the API:
  kubectl port-forward svc/{{ include "nexus-api-chart.fullname" . }} \
    8080:{{ .Values.service.port }} -n {{ .Release.Namespace }}

  curl http://localhost:8080/health

To see pods:
  kubectl get pods -n {{ .Release.Namespace }} -l app.kubernetes.io/instance={{ .Release.Name }}
========================================
EOF
```

---

### Environment-Specific Values Files

```yaml
# values-dev.yaml
replicaCount: 1
image:
  tag: "develop"
config:
  appEnv: development
  logLevel: DEBUG
resources:
  requests:
    memory: "64Mi"
    cpu: "100m"
  limits:
    memory: "128Mi"
    cpu: "200m"
service:
  type: NodePort
```

```yaml
# values-staging.yaml
replicaCount: 2
image:
  tag: "v2.1.0-rc1"
config:
  appEnv: staging
  logLevel: INFO
  dbHost: staging-db-service
resources:
  requests:
    memory: "128Mi"
    cpu: "200m"
  limits:
    memory: "256Mi"
    cpu: "400m"
```

```yaml
# values-prod.yaml
replicaCount: 5
image:
  tag: "v2.1.0"
config:
  appEnv: production
  logLevel: WARNING
  dbHost: prod-db-service
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1000m"
autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
ingress:
  enabled: true
  hostname: api.nexuscorp.com
```

---

## Part 8: Using Community Charts

### Installing Popular Charts

```bash
# nginx Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx \
    -n ingress-nginx --create-namespace

# cert-manager (automatic TLS certificates)
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
    -n cert-manager --create-namespace \
    --set installCRDs=true

# Prometheus + Grafana monitoring stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prometheus prometheus-community/kube-prometheus-stack \
    -n monitoring --create-namespace \
    --set grafana.adminPassword=nexus-admin

# PostgreSQL database
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install nexus-db bitnami/postgresql \
    -n nexus-production \
    --set auth.database=nexusdb \
    --set auth.username=nexususer \
    --set auth.password=nexus-db-pass

# Redis cache
helm install nexus-cache bitnami/redis \
    -n nexus-production \
    --set auth.password=redis-pass \
    --set replica.replicaCount=1
```

---

## Part 9: Practical Tasks

### Task: Create a Helm Chart for a Two-Tier Application

Build a complete Helm chart for NexusCorp's backend API + PostgreSQL database.

```bash
# Step 1: Create the chart
helm create nexus-two-tier
cd nexus-two-tier

# Step 2: Update Chart.yaml with dependency
cat > Chart.yaml << 'EOF'
apiVersion: v2
name: nexus-two-tier
description: "NexusCorp two-tier application: API + Database"
type: application
version: 1.0.0
appVersion: "2.0.0"
dependencies:
- name: postgresql
  version: "12.x.x"
  repository: "https://charts.bitnami.com/bitnami"
  condition: postgresql.enabled
EOF

# Step 3: Update values.yaml
cat > values.yaml << 'EOF'
# Backend API configuration
backend:
  replicaCount: 2
  image:
    repository: nginx
    tag: "1.25"
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 80
    targetPort: 80
  resources:
    requests:
      memory: "128Mi"
      cpu: "250m"
    limits:
      memory: "256Mi"
      cpu: "500m"
  config:
    appEnv: production
    logLevel: INFO

# PostgreSQL sub-chart configuration
postgresql:
  enabled: true
  auth:
    database: nexusdb
    username: nexususer
    password: nexus-db-pass
  primary:
    persistence:
      enabled: true
      size: 5Gi
EOF

# Step 4: Update templates for two-tier

# Backend deployment
cat > templates/deployment.yaml << 'TMPL'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-backend
  namespace: {{ .Release.Namespace }}
  labels:
    app.kubernetes.io/name: backend
    app.kubernetes.io/instance: {{ .Release.Name }}
    helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version }}
spec:
  replicas: {{ .Values.backend.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-backend
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-backend
    spec:
      containers:
      - name: backend
        image: "{{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}"
        imagePullPolicy: {{ .Values.backend.image.pullPolicy }}
        ports:
        - containerPort: {{ .Values.backend.service.targetPort }}
        env:
        - name: APP_ENV
          value: {{ .Values.backend.config.appEnv | quote }}
        - name: LOG_LEVEL
          value: {{ .Values.backend.config.logLevel | quote }}
        {{- if .Values.postgresql.enabled }}
        - name: DB_HOST
          value: {{ .Release.Name }}-postgresql
        - name: DB_NAME
          value: {{ .Values.postgresql.auth.database | quote }}
        - name: DB_USERNAME
          value: {{ .Values.postgresql.auth.username | quote }}
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Release.Name }}-postgresql
              key: password
        {{- end }}
        resources:
          {{- toYaml .Values.backend.resources | nindent 10 }}
TMPL

# Backend service
cat > templates/service.yaml << 'TMPL'
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-backend
  namespace: {{ .Release.Namespace }}
spec:
  type: {{ .Values.backend.service.type }}
  selector:
    app: {{ .Release.Name }}-backend
  ports:
  - name: http
    port: {{ .Values.backend.service.port }}
    targetPort: {{ .Values.backend.service.targetPort }}
TMPL

# Step 5: Download dependencies
helm dependency update .

# Step 6: Lint the chart
helm lint .

# Step 7: Preview what would be deployed
helm template nexus-prod . | head -80

# Step 8: Install the chart
kubectl create namespace nexus-two-tier-demo 2>/dev/null || true

helm install nexus-prod . \
    -n nexus-two-tier-demo \
    --wait \
    --timeout 5m

# Step 9: Verify
helm list -n nexus-two-tier-demo
kubectl get all -n nexus-two-tier-demo

# Step 10: Show the post-install notes
helm status nexus-prod -n nexus-two-tier-demo

# Step 11: Test upgrade
helm upgrade nexus-prod . \
    --set backend.replicaCount=3 \
    -n nexus-two-tier-demo

# Step 12: View history
helm history nexus-prod -n nexus-two-tier-demo

# Step 13: Rollback
helm rollback nexus-prod 1 -n nexus-two-tier-demo

# Step 14: Clean up
helm uninstall nexus-prod -n nexus-two-tier-demo
kubectl delete namespace nexus-two-tier-demo
```

---

## 🔧 Mini-Project: NexusCorp Full Helm Chart Repository

```bash
#!/bin/bash
# nexus_helm_demo.sh
# Complete Helm workflow demonstration

set -e
CHART_DIR="./nexus-chart-demo"
RELEASE="nexus-demo"
NAMESPACE="nexus-helm-test"

echo "=============================================="
echo "  NexusCorp Helm Chart Demo"
echo "=============================================="

# Step 1: Create chart
echo ""
echo "Step 1: Creating chart..."
helm create $CHART_DIR
echo "  ✓ Chart scaffold created: $CHART_DIR"

# Step 2: Lint
echo ""
echo "Step 2: Linting chart..."
helm lint $CHART_DIR
echo "  ✓ Chart passed lint checks"

# Step 3: Template preview
echo ""
echo "Step 3: Template preview (first 30 lines)..."
helm template $RELEASE $CHART_DIR | head -30

# Step 4: Install
echo ""
echo "Step 4: Installing chart..."
kubectl create namespace $NAMESPACE 2>/dev/null || true

helm install $RELEASE $CHART_DIR \
    --namespace $NAMESPACE \
    --set replicaCount=2 \
    --wait \
    --timeout 3m

echo "  ✓ Chart installed"

# Step 5: Show status
echo ""
echo "Step 5: Release status..."
helm status $RELEASE -n $NAMESPACE
echo ""
helm list -n $NAMESPACE

# Step 6: Upgrade
echo ""
echo "Step 6: Upgrading release (scale to 3)..."
helm upgrade $RELEASE $CHART_DIR \
    --namespace $NAMESPACE \
    --set replicaCount=3 \
    --wait

echo "  ✓ Upgraded to 3 replicas"

# Step 7: History
echo ""
echo "Step 7: Release history..."
helm history $RELEASE -n $NAMESPACE

# Step 8: Rollback
echo ""
echo "Step 8: Rolling back to revision 1..."
helm rollback $RELEASE 1 -n $NAMESPACE --wait
echo "  ✓ Rolled back"

helm history $RELEASE -n $NAMESPACE

# Step 9: Get values
echo ""
echo "Step 9: Current values..."
helm get values $RELEASE -n $NAMESPACE

# Cleanup
echo ""
echo "Step 10: Cleaning up..."
helm uninstall $RELEASE -n $NAMESPACE
kubectl delete namespace $NAMESPACE
rm -rf $CHART_DIR

echo ""
echo "=============================================="
echo "  Demo complete!"
echo ""
echo "  Key commands summary:"
echo "  helm create [chart]      — scaffold a chart"
echo "  helm lint [chart]        — validate chart"
echo "  helm template [chart]    — preview manifests"
echo "  helm install [rel] [chart] — deploy"
echo "  helm upgrade [rel] [chart] — update"
echo "  helm rollback [rel] [rev]  — revert"
echo "  helm uninstall [rel]       — remove"
echo "=============================================="
```

```bash
chmod +x nexus_helm_demo.sh
./nexus_helm_demo.sh
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Helm** | The package manager for Kubernetes |
| **Chart** | A Helm package containing all Kubernetes resources for an application |
| **Release** | A running instance of a chart installed in a cluster |
| **Revision** | A numbered snapshot of a release — increments on every install/upgrade/rollback |
| **Values** | Configuration inputs to a chart — defined in `values.yaml`, overridable at install time |
| **Template** | A Kubernetes YAML file with Go template placeholders (`{{ }}`) |
| **`_helpers.tpl`** | A file containing reusable template snippets (named templates) |
| **`Chart.yaml`** | Metadata file for a chart — name, version, description, dependencies |
| **`values.yaml`** | Default values file for a chart |
| **`NOTES.txt`** | Post-install message shown to the user after `helm install` |
| **`helm create`** | Scaffolds a new chart with default structure |
| **`helm lint`** | Validates a chart for syntax and best practices |
| **`helm template`** | Renders chart templates locally without installing |
| **`helm install`** | Installs a chart and creates a release |
| **`helm upgrade`** | Updates an existing release to a new chart version or values |
| **`helm rollback`** | Reverts a release to a previous revision |
| **`helm uninstall`** | Removes a release from the cluster |
| **`helm list`** | Lists all releases in a namespace |
| **`helm status`** | Shows the current status of a release |
| **`helm history`** | Shows the revision history of a release |
| **`helm get values`** | Shows the values used for a release |
| **`helm get manifest`** | Shows the rendered Kubernetes manifests for a release |
| **`helm repo add`** | Adds a chart repository |
| **`helm repo update`** | Updates the local chart repository index |
| **`helm search repo`** | Searches for charts in configured repositories |
| **`helm dependency update`** | Downloads sub-chart dependencies |
| **`--set`** | Flag to override individual values on the command line |
| **`--values`** | Flag to specify a values override file |
| **`--dry-run`** | Simulates install/upgrade without applying changes |
| **`--wait`** | Waits for all resources to be ready before returning |
| **`upgrade --install`** | Installs if not present, upgrades if already installed |
| **Artifact Hub** | The public Helm chart repository (like npm registry for charts) |
| **Bitnami** | Publisher of widely-used, well-maintained Helm charts |
| **Sub-chart** | A chart dependency included inside another chart's `charts/` directory |
| **Go templates** | The templating language used in Helm templates |

---

## 📚 Resources

- [Helm Official Documentation](https://helm.sh/docs/) — complete Helm reference
- [Artifact Hub](https://artifacthub.io) — discover and share Helm charts
- [Helm Chart Best Practices](https://helm.sh/docs/chart_best_practices/) — official guide
- [Bitnami Helm Charts](https://github.com/bitnami/charts) — production-ready charts for PostgreSQL, Redis, etc.
- [HELM Packaging of Two-Tier Applications — YouTube](https://www.youtube.com/results?search_query=Helm+two+tier+application+DevOps) — video referenced in curriculum
- [Kubestarter Helm Examples](https://github.com/LondheShubham153/kubestarter/tree/main/examples/helm) — Shubham Londhe's reference examples

---

## 🔭 Day 47 Preview: Kubernetes Monitoring with Prometheus & Grafana

Today you packaged applications with Helm. Tomorrow you make sure those applications are always healthy using monitoring.

You will learn:
- Why monitoring is essential in Kubernetes
- Prometheus — metrics collection and alerting
- Grafana — visualization dashboards for Kubernetes
- Installing the kube-prometheus-stack with Helm (one command)
- Key metrics to watch: pod CPU/memory, node resources, request rates, error rates
- Creating custom dashboards for NexusCorp's applications
- Setting up alerts — get notified before problems become outages

Monitoring completes the DevOps loop: deploy with confidence, observe in production, act on data.

---

> 💬 *"Helm is not about avoiding YAML — it's about writing YAML once and running it everywhere. The chart is the artifact; the values are the configuration; the release is the truth."*
>
> — DevOps Learning By Yukta
