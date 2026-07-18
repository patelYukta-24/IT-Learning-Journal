# Day 44: StatefulSets — Managing Stateful Applications in Kubernetes

## 📌 Overview

Most applications run on Kubernetes are stateless — they can be destroyed and recreated at will without data loss, which is exactly what Deployments are built for. But certain applications, like databases, message queues, and other systems that need a maintained state, need something more disciplined. **StatefulSets** are the Kubernetes workload API built specifically for deploying and scaling stateful applications, giving each Pod a durable identity, stable storage, and a predictable startup/shutdown order.

## 🎯 Learning Objectives

- Understand what distinguishes a StatefulSet from a Deployment and why that distinction matters for stateful workloads.
- Understand how Pod identity (name, ordinal, hostname) stays stable across the lifetime of a StatefulSet.
- Understand how per-Pod storage is provisioned and retained via `volumeClaimTemplates`.
- Understand ordered, graceful scaling up/down.
- Understand rolling updates and rollbacks for StatefulSets.
- Understand how stable network identity (via a Headless Service) makes each Pod individually addressable.

## 📚 Topics Covered

### 1. Understanding StatefulSets
Unlike Deployments — where Pods are interchangeable and identity doesn't matter — StatefulSets guarantee that Pods are unique and created/terminated in a strict order. Each Pod in a StatefulSet has a persistent identity that survives rescheduling, making StatefulSets the right tool whenever Pods are *not* interchangeable (e.g., each node in a database cluster plays a distinct role).

### 2. Pod Identity
Every Pod in a StatefulSet gets:
- A **stable, predictable name**: `<statefulset-name>-<ordinal>` (e.g., `mongo-0`, `mongo-1`, `mongo-2`), starting at 0 and incrementing.
- A **stable hostname** matching that name, so the Pod is always addressable the same way even after being rescheduled to a different Node.

### 3. Storage in StatefulSets
Each Pod in a StatefulSet can get its own **PersistentVolumeClaim**, generated automatically from a `volumeClaimTemplates` spec on the StatefulSet. When a Pod is rescheduled, Kubernetes reattaches it to the *same* PVC (and therefore the same data) rather than creating a fresh one — this is what gives stateful workloads durability across restarts. Different Pods' PVCs can be backed by different StorageClasses depending on the performance/durability tradeoffs needed.

### 4. Scaling with StatefulSets
Scaling is **ordered and one-at-a-time**:
- Scaling **up** creates new Pods in increasing ordinal order (`mongo-0` before `mongo-1` before `mongo-2`), waiting for each to be Running and Ready before starting the next.
- Scaling **down** terminates Pods in *reverse* ordinal order (highest ordinal first), so the "oldest" members of the set are the last to go.
This ordering matters a lot for clustered databases, where node 0 is often a primary/seed node.

### 5. Updates and Rollbacks
StatefulSets support rolling updates just like Deployments, but again applied in ordered fashion (by default, highest ordinal to lowest). The `updateStrategy` can be `RollingUpdate` (the default) or `OnDelete` (manual, Pod-by-Pod control). A `partition` value can be set on a `RollingUpdate` strategy to stage a canary rollout to only the highest-ordinal Pods first. Rollbacks work the same way as Deployments, via revision history (`kubectl rollout undo`).

### 6. Stable Network ID
StatefulSets are typically paired with a **Headless Service** (`clusterIP: None`). This gives each Pod its own stable DNS record: `<pod-name>.<service-name>.<namespace>.svc.cluster.local` — e.g. `mongo-0.mongo.default.svc.cluster.local`. Because this DNS name is derived from the Pod's stable identity, it never changes across restarts or rescheduling, making direct Pod-to-Pod addressing (essential for clustering/replication protocols) reliable.

## 🛠️ Hands-On Practice (Lab Notes)

Practiced locally using Minikube with a 3-node MongoDB StatefulSet.

**Headless Service + StatefulSet:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    app: mongo
spec:
  clusterIP: None
  selector:
    app: mongo
  ports:
    - port: 27017
      name: mongo
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 3
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:6.0
          ports:
            - containerPort: 27017
              name: mongo
          volumeMounts:
            - name: mongo-data
              mountPath: /data/db
  volumeClaimTemplates:
    - metadata:
        name: mongo-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

### Task 1 — Create the StatefulSet
Applied the manifest above and watched Pods come up strictly in order:
```bash
kubectl apply -f mongo-statefulset.yaml
kubectl get pods -w
# mongo-0 Running before mongo-1 is even created, then mongo-1, then mongo-2
```
Confirmed each Pod has its own PVC:
```bash
kubectl get pvc
# mongo-data-mongo-0, mongo-data-mongo-1, mongo-data-mongo-2
```

### Task 2 — Scale the StatefulSet
```bash
kubectl scale statefulset mongo --replicas=5
kubectl get pods -w
# mongo-3 then mongo-4 created in order

kubectl scale statefulset mongo --replicas=2
kubectl get pods -w
# mongo-4 terminated first, then mongo-3, then mongo-2 (reverse order)
```

### Task 3 — Rolling Update and Rollback
```bash
kubectl set image statefulset/mongo mongo=mongo:6.0.6
kubectl rollout status statefulset/mongo
# highest ordinal (mongo-2) updated first, then mongo-1, then mongo-0

kubectl rollout history statefulset/mongo
kubectl rollout undo statefulset/mongo
```

### Task 4 — Data Persistence
```bash
kubectl exec -it mongo-0 -- mongosh --eval "db.test.insertOne({msg: 'hello'})"
kubectl delete pod mongo-0
kubectl get pods -w   # mongo-0 recreated, reattached to the same PVC
kubectl exec -it mongo-0 -- mongosh --eval "db.test.find()"
# document is still there — same underlying volume, not a fresh one
```

### Bonus Task — Simulated Node Failure / Multi-Node Failover
Set up a 3-node MongoDB replica set (`mongo-0` as initial primary, `mongo-1`/`mongo-2` as secondaries) using `rs.initiate()`. Simulated a node failure by deleting the primary Pod:
```bash
kubectl delete pod mongo-0
```
Observed the replica set automatically elect a new primary among the remaining members while `mongo-0` was rescheduled and rejoined the set as a secondary once its data volume was reattached and it caught up via replication.

## 🔑 Key Commands

```bash
kubectl get statefulsets
kubectl describe statefulset mongo
kubectl get pods -l app=mongo -o wide
kubectl get pvc
kubectl rollout status statefulset/mongo
kubectl rollout history statefulset/mongo
kubectl rollout undo statefulset/mongo
kubectl scale statefulset mongo --replicas=<n>
kubectl delete pod <statefulset-pod-name>
```

## 📝 Key Takeaways

- StatefulSets exist for exactly one reason: some workloads care about *which* Pod they are, not just *how many* Pods exist.
- Ordinal-based naming + per-Pod PVCs + a Headless Service together give a Pod a durable, addressable identity that survives rescheduling — that's the whole value proposition.
- Scaling and updates are intentionally ordered and one-at-a-time by default; this is slower than a Deployment's rollout but is what keeps clustered/replicated stateful apps safe during changes.
- Deleting a Pod in a StatefulSet does **not** delete its PVC — data persists across Pod recreation unless the PVC is deleted explicitly.
- Real-world stateful apps (MongoDB, Kafka, Cassandra, etc.) layer their own clustering/replication logic on top of what StatefulSets provide — Kubernetes guarantees identity and storage stability, not database-level consensus or failover, which the app itself must handle.

## 📖 Resources

- [Kubernetes Official Documentation — StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- Kubernetes StatefulSet — What is it, Examples & Best Practices

## ✅ Status

Self-study complete — concepts reviewed and validated with a local Minikube lab (3-node MongoDB StatefulSet: creation order, scaling order, rolling update/rollback, Pod deletion + data persistence, and a simulated primary-node failure/failover).

---
*Part of the [IT Learning Journal](../../README.md) — DevOps Transformation curriculum, Day 44.*
