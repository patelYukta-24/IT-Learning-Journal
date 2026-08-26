# Day 81 — Project 5: Deploying a Netflix Clone on Kubernetes

**Part XII: Project 5 (Day 81)**
**Curriculum:** NexusCorp DevOps Transformation — Shubham Londhe

---

## ❖ Project Overview

Kubernetes has emerged as a de facto standard for orchestrating containerized applications. This project involved deploying a Netflix clone — a multi-component web application — on a Kubernetes cluster. The work covered building Docker images, writing Kubernetes manifests to define the application's infrastructure, and managing the deployed workload using `kubectl` and the Kubernetes Dashboard for monitoring and scalability.

## ❖ Project Objective

- Learn how to deploy and manage applications on Kubernetes.
- Understand the creation of Docker images and Kubernetes manifests.
- Implement high availability, scaling, and self-healing features provided by Kubernetes.
- Gain practical skills with the Kubernetes Dashboard and `kubectl` for application management.

## ❖ Skills Showcased

- Kubernetes Deployment and Management
- Docker Containerization
- Application Scaling and Load Balancing
- Infrastructure as Code (IaC) with Kubernetes Manifests
- Application Monitoring

## ❖ Environment

- **Cluster:** Local single-node cluster (Minikube) — used for self-study/practice instead of a cloud-managed cluster, to keep the exercise reproducible on a personal machine.
- **Container runtime:** Docker
- **CLI tools:** `kubectl`, `minikube`, Kubernetes Dashboard addon
- **App:** Open-source Netflix clone (React front end + TMDB API integration)

---

## ❖ Tasks / Exercises

### Task 1: Deploy a Netflix Clone Application on a Kubernetes Cluster

#### Subtask 1 — Locate a suitable Netflix clone repository

Selected a publicly available open-source Netflix clone project on GitHub (React-based front end that consumes The Movie Database, TMDB, API). Cloned it locally:

```bash
git clone https://github.com/<netflix-clone-repo>.git
cd netflix-clone
```

> Note: A free TMDB API key is required for the app to fetch movie data. This was generated separately and injected as a build-time environment variable — no secret is committed to the repository.

#### Subtask 2 — Create Docker images for the Netflix clone and its dependencies

Wrote a multi-stage `Dockerfile` to keep the final image small — a Node build stage followed by an Nginx runtime stage.

**Dockerfile**
```dockerfile
# ---- Build Stage ----
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ARG TMDB_V3_API_KEY
ENV VITE_APP_TMDB_V3_API_KEY=${TMDB_V3_API_KEY}
RUN npm run build

# ---- Runtime Stage ----
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Built and tagged the image, then loaded it into the local cluster's image cache (required since Minikube runs its own Docker daemon):

```bash
docker build --build-arg TMDB_V3_API_KEY=<api_key> -t netflix-clone:v1 .
minikube image load netflix-clone:v1
```

#### Subtask 3 — Write Kubernetes manifests

Created a `k8s/` folder with a Deployment and a Service.

**deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netflix-clone
  labels:
    app: netflix-clone
spec:
  replicas: 3
  selector:
    matchLabels:
      app: netflix-clone
  template:
    metadata:
      labels:
        app: netflix-clone
    spec:
      containers:
        - name: netflix-clone
          image: netflix-clone:v1
          imagePullPolicy: Never   # local image loaded via `minikube image load`
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
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
```

**service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: netflix-clone-service
spec:
  type: NodePort
  selector:
    app: netflix-clone
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30036
```

#### Subtask 4 — Deploy using `kubectl`

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

kubectl get deployments
kubectl get pods -o wide
kubectl get svc
```

All three replica pods reached `Running` status with `1/1 READY`.

#### Subtask 5 — Expose the application to the internet

Used the `NodePort` Service above for local access, and separately tested exposure via `kubectl expose` and Minikube's tunnel/service URL:

```bash
minikube service netflix-clone-service --url
```

This returned a local URL (`http://127.0.0.1:<port>`) that opened the running Netflix clone UI in a browser — confirming the Service correctly routed traffic to the backing pods.

For completeness, also drafted an **Ingress** resource to represent how this would be exposed in a cloud environment behind a real domain/load balancer:

**ingress.yaml**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: netflix-clone-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: netflix-clone.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: netflix-clone-service
                port:
                  number: 80
```

Enabled the Minikube ingress addon and mapped `netflix-clone.local` to the Minikube IP in the local hosts file to validate this path as well:

```bash
minikube addons enable ingress
echo "$(minikube ip) netflix-clone.local" | sudo tee -a /etc/hosts
kubectl apply -f k8s/ingress.yaml
```

#### Subtask 6 — Use the Kubernetes Dashboard

```bash
minikube addons enable dashboard
minikube dashboard
```

Used the Dashboard to visually confirm:
- Deployment `netflix-clone` with 3/3 pods available
- Pod resource usage, restart counts, and events
- Service `netflix-clone-service` with its endpoints
- ReplicaSet history for the Deployment

---

## ❖ Scaling and Self-Healing Verification

To exercise Kubernetes' HA/self-healing features rather than just deploying once:

```bash
# Scale up
kubectl scale deployment netflix-clone --replicas=5
kubectl get pods -w

# Simulate a pod failure
kubectl delete pod <pod-name>
kubectl get pods -w   # confirms a replacement pod is scheduled automatically
```

Observed the ReplicaSet controller immediately reconcile pod count back to the desired replica count after the manual deletion, and the Service continued routing traffic without downtime during the scale-up.

---

## ❖ Deliverables

- [x] **GitHub repository link** for the Netflix clone source code (cloned open-source project, referenced in repo notes)
- [x] **Dockerfile** used to build the application image
- [x] **Kubernetes manifest files** — `deployment.yaml`, `service.yaml`, `ingress.yaml`
- [x] **Evidence of successful deployment** — accessed via browser through the NodePort Service URL and the Ingress host
- [x] **Screenshots of the Kubernetes Dashboard** showing the Deployment, Pods, and Service resources (see `/screenshots` folder)

---

## ❖ Notes / Learnings

- `imagePullPolicy: Never` combined with `minikube image load` is the key detail for using a locally-built image on Minikube without pushing to a registry — a common stumbling point when the cluster otherwise tries to pull from Docker Hub and fails with `ImagePullBackOff`.
- Readiness and liveness probes were essential to see Kubernetes' self-healing in action; without them, a hung container can stay marked `Running` indefinitely.
- NodePort was sufficient for local validation; Ingress more accurately represents how this would be exposed in a real/cloud cluster behind a domain name and a proper load balancer/Ingress controller.
- This was completed as a **self-study lab exercise on a local Minikube cluster**, not a production or cloud deployment — documented here as practice toward the Kubernetes Deployment and Management skill area of the curriculum.

---

## ❖ References

- Kubernetes official documentation — Deployments, Services, Ingress
- Minikube documentation
- Open-source Netflix clone project (React + TMDB API)
