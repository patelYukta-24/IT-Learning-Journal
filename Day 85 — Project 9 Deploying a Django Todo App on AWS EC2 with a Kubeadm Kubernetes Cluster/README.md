# Day 85 — Project 9: Deploying a Django Todo App on AWS EC2 with a Kubeadm Kubernetes Cluster

**Part XII: Project 9 (Day 85)**
**Curriculum:** NexusCorp DevOps Transformation — Shubham Londhe

---

## ❖ Project Overview

This project deploys a Django Todo application on a Kubernetes cluster that was hand-built on AWS EC2 instances using **Kubeadm**, rather than a managed Kubernetes offering (EKS) or a local single-node cluster (Minikube, used on Day 81). Building the cluster this way exposes the underlying control-plane/worker-node bootstrapping that managed services normally hide, while Kubernetes' auto-scaling and self-healing features keep the application available under varying load once deployed.

## ❖ Project Objective

- Deploy a full-stack Django application on a Kubernetes cluster.
- Learn how to set up a Kubernetes cluster on AWS EC2 instances using Kubeadm.
- Utilize Kubernetes' auto-scaling and self-healing to manage application deployment.

## ❖ Skills Showcased

- Kubernetes Cluster Creation and Management with Kubeadm
- Django Application Deployment on a Kubernetes Cluster
- AWS EC2 Instance Configuration and Management
- Understanding of Kubernetes Deployment and Service Objects

## ❖ Environment

- **AWS infrastructure:** 3 EC2 instances (Ubuntu 22.04) — 1 control-plane node, 2 worker nodes, all within the same VPC/subnet
- **Cluster bootstrap:** Kubeadm
- **Container runtime:** containerd
- **CNI:** Calico
- **App:** Django Todo application (Python/Django + SQLite or Postgres backend, per the source repo)

---

## ❖ Tasks / Exercises

### Task 1: Deploy a Django Full-Stack Application to a Kubernetes Cluster on AWS EC2

#### Subtask 1 — Clone a Django Full Stack application

```bash
git clone https://github.com/<username>/django-todo-app.git
cd django-todo-app
```

Reviewed the app structure (`manage.py`, `requirements.txt`, a `Dockerfile` was added since the source repo didn't include one) and built the container image:

```dockerfile
# Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

```bash
docker build -t django-todo-app:v1 .
docker tag django-todo-app:v1 <dockerhub-username>/django-todo-app:v1
docker push <dockerhub-username>/django-todo-app:v1
```

Used Docker Hub as the image registry here since the cluster is self-managed (no built-in registry like ECR's tight EKS integration) — kept the repository private and pulled with an `imagePullSecret` on the cluster side.

#### Subtask 2 — Set up a Kubernetes cluster using Kubeadm on AWS EC2 instances

**Provisioned 3 EC2 instances** (`t3.medium`, 2 vCPU / 4 GB minimum for kubeadm) with a security group allowing:
- Port `6443` (Kubernetes API server) — control plane
- Ports `2379–2380` (etcd) — control plane
- Ports `10250–10259` — kubelet/scheduler/controller-manager
- Port `179` (Calico BGP) and UDP `4789` (VXLAN) — pod networking
- Port `22` (SSH) and `8000`/`30000–32767` (NodePort range) — app access

**On all three nodes** — common prerequisites:

```bash
sudo swapoff -a
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sudo sysctl --system

# Install containerd
sudo apt update && sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd

# Install kubeadm, kubelet, kubectl
sudo apt install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

**On the control-plane node** — initialize the cluster:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<control-plane-private-ip>

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Installed the Calico CNI so pods could get IPs and communicate across nodes:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

`kubeadm init` printed a `kubeadm join` command containing a bootstrap token and CA cert hash — saved this for the worker nodes.

**On each worker node** — joined the cluster using the printed command:

```bash
sudo kubeadm join <control-plane-private-ip>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

**Verified the cluster from the control-plane node:**

```bash
kubectl get nodes -o wide
```

All three nodes showed `STATUS: Ready` once Calico finished initializing pod networking on each.

#### Subtask 3 — Create and apply Kubernetes Deployment and Service configurations

**deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-todo-app
  labels:
    app: django-todo-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: django-todo-app
  template:
    metadata:
      labels:
        app: django-todo-app
    spec:
      containers:
        - name: django-todo-app
          image: <dockerhub-username>/django-todo-app:v1
          ports:
            - containerPort: 8000
          env:
            - name: DJANGO_ALLOWED_HOSTS
              value: "*"
          resources:
            requests:
              cpu: "150m"
              memory: "256Mi"
            limits:
              cpu: "300m"
              memory: "512Mi"
          readinessProbe:
            httpGet:
              path: /
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 8000
            initialDelaySeconds: 20
            periodSeconds: 20
```

**service.yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: django-todo-service
spec:
  type: NodePort
  selector:
    app: django-todo-app
  ports:
    - port: 8000
      targetPort: 8000
      nodePort: 30080
```

Applied both manifests from the control-plane node:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

kubectl get deployments
kubectl get pods -o wide
kubectl get svc
```

Pods were scheduled across both worker nodes (visible via the `NODE` column in `kubectl get pods -o wide`), confirming the scheduler was distributing the workload rather than stacking everything on one node.

Accessed the application from a browser using any worker node's public IP and the NodePort:

```
http://<worker-node-public-ip>:30080
```

The Django Todo app loaded correctly and todo items could be created/marked complete, confirming the full request path — Service → kube-proxy → pod — was working end to end.

---

## ❖ Auto-Scaling and Self-Healing Verification

```bash
# Self-healing: delete a pod and confirm the ReplicaSet replaces it
kubectl delete pod <pod-name>
kubectl get pods -w

# Scaling: increase replica count
kubectl scale deployment django-todo-app --replicas=5
kubectl get pods -o wide -w
```

Observed a replacement pod scheduled automatically within seconds of the manual deletion, and the two additional replicas from the scale-up landing on whichever worker node had available capacity — confirming both self-healing and horizontal scaling behavior on the self-managed cluster.

---

## ❖ Deliverables

- [x] **Functioning Django Todo application** running on a Kubeadm-provisioned Kubernetes cluster hosted on AWS EC2
- [x] **Kubernetes Deployment configuration** — `deployment.yaml`
- [x] **Kubernetes Service configuration** — `service.yaml`
- [x] **Screenshots/logs verifying the app is running and accessible** — `kubectl get nodes/pods/svc` output, browser screenshot of the Todo app at the NodePort URL

---

## ❖ Notes / Learnings

- Kubeadm exposes every piece that a managed service like EKS or a local tool like Minikube normally handles invisibly — CNI installation, control-plane/worker join tokens, kubelet cgroup driver alignment (`SystemdCgroup = true` in containerd's config was a common failure point if left mismatched with kubelet's driver).
- `swapoff -a` and the sysctl bridge/forwarding settings are mandatory prerequisites on every node — kubeadm's preflight checks fail immediately without them.
- Running the app as a **Deployment** (rather than a bare Pod) is what enabled both the self-healing and scaling tests — the ReplicaSet controller is the mechanism actually doing the reconciliation work in both cases.
- Distributing pods across the two worker nodes (visible via `-o wide`) is a direct, hands-on illustration of why multi-node clusters provide higher availability than the single-node Minikube setup used on Day 81 — a node failure here would only take out the pods scheduled on that one node.
- This was completed as a **self-study lab exercise** on personal AWS EC2 instances — documented here as practice toward the Kubeadm cluster-building and Kubernetes Deployment/Service skill areas of the curriculum, not a production deployment.

---

## ❖ Cleanup

To avoid ongoing EC2 charges after validation:

```bash
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
```
Then terminate all three EC2 instances from the AWS Console or via:
```bash
aws ec2 terminate-instances --instance-ids <control-plane-id> <worker-1-id> <worker-2-id>
```

## ❖ Resources

- Kubeadm official documentation (cluster bootstrapping)
- Project Calico documentation (CNI installation)
