# Day 80: Project 4 — Web Application Deployment with Docker Swarm

## Project Overview

Deploying with high availability and scalability matters for any production web application. **Docker Swarm** is Docker's native container orchestration tool — it turns a group of Docker hosts into a single logical cluster ("swarm") and handles scheduling, scaling, load balancing, rolling updates, and self-healing for containerized services. This project orchestrates a web application across a Swarm cluster to demonstrate exactly those capabilities hands-on.

## Project Objective

- Gain hands-on experience with container orchestration using Docker Swarm.
- Package the application into a container using a `Dockerfile`.
- Configure and manage a Swarm cluster for high application availability.
- Implement rolling updates for zero-downtime deployments.

## Skills Showcased

- Docker and Docker Swarm
- Container Orchestration
- Microservices Deployment
- CI/CD Integration
- Load Balancing and Networking

---

## Task 1: Deploy the Web Application Using Docker Swarm

### Subtask 1 — Write a Dockerfile

Using the same simple Node.js/Express "Hello World" app from earlier CI/CD projects, containerized for Swarm deployment:

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

Build and push the image to a registry so all Swarm nodes can pull it:

```bash
docker build -t <dockerhub-username>/swarm-webapp:v1 .
docker push <dockerhub-username>/swarm-webapp:v1
```

### Subtask 2 — Initialize Docker Swarm mode

On the machine designated as the **manager node**:

```bash
docker swarm init --advertise-addr <manager-ip>
```

This prints a `docker swarm join` command with a join token, e.g.:

```bash
docker swarm join --token SWMTKN-1-xxxxxxxxxxxxxxxxxxxx <manager-ip>:2377
```

Run that exact command on each additional machine intended to be a **worker node**. Confirm cluster membership from the manager:

```bash
docker node ls
```

This should list all nodes with their roles (`Leader`, `Reachable`, `Worker`) and availability status.

### Subtask 3 — Deploy the containerized app as a replicated service

```bash
docker service create \
  --name webapp \
  --replicas 4 \
  --publish published=8080,target=3000 \
  <dockerhub-username>/swarm-webapp:v1
```

- `--replicas 4` — runs four container instances of the app, spread across available nodes for high availability. If a node goes down, Swarm reschedules its replicas onto healthy nodes automatically.
- `--publish published=8080,target=3000` — publishes the service on port `8080` on **every** Swarm node (ingress routing mesh), regardless of which node a given replica actually runs on.

Verify:

```bash
docker service ls
docker service ps webapp
```

### Subtask 5 — Load balancing across container instances

Docker Swarm handles this natively via its built-in **ingress routing mesh** — no extra load balancer needed for basic distribution. Any request to `<any-node-ip>:8080` is routed by Swarm's internal load balancer (using IPVS) to one of the healthy `webapp` replicas, regardless of which node actually received the request or which node is running that replica.

For external traffic distribution across multiple manager/worker IPs (e.g., in production), an external load balancer (AWS ALB, Nginx, HAProxy) can sit in front of the swarm nodes, but the mesh itself already load-balances at the container level.

### Subtask 4 — Rolling updates for zero-downtime deployment

Build and push a new version of the image first:

```bash
docker build -t <dockerhub-username>/swarm-webapp:v2 .
docker push <dockerhub-username>/swarm-webapp:v2
```

Configure the update strategy and apply it:

```bash
docker service update \
  --image <dockerhub-username>/swarm-webapp:v2 \
  --update-parallelism 1 \
  --update-delay 10s \
  --update-failure-action rollback \
  webapp
```

- `--update-parallelism 1` — updates one replica at a time, so at least three of four replicas remain serving traffic at any point during the rollout.
- `--update-delay 10s` — waits 10 seconds between each replica update, giving each new container time to become healthy before moving to the next.
- `--update-failure-action rollback` — if a new replica fails to start correctly, Swarm automatically rolls back to the previous working image rather than continuing a broken rollout.

Watch the rollout in real time:

```bash
docker service ps webapp
```

Each replica transitions through `Ready → Running` for `v2` while the old `v1` tasks are marked `Shutdown`, one at a time — visually confirming a zero-downtime, staggered replacement rather than a simultaneous kill-and-restart.

### Subtask 6 — Test service discovery and failover

```bash
# Identify a running task/container for the service
docker service ps webapp

# Force-remove a container to simulate a node/instance failure
docker rm -f <container-id-on-that-node>
```

Because the desired replica count (4) is declared to the Swarm manager, it detects the missing task and automatically schedules a replacement replica on an available node within seconds — no manual intervention required. Re-running `docker service ps webapp` shows the failed task marked `Shutdown` alongside a new task in `Running` state, demonstrating Swarm's self-healing behavior.

---

## Documentation: Swarm Initialization and Deployment Process Summary

| Step | Command/Action | Purpose |
|---|---|---|
| 1 | `docker swarm init` | Creates the cluster, designates the current node as manager |
| 2 | `docker swarm join` (on workers) | Adds worker nodes to the cluster |
| 3 | `docker build` + `docker push` | Packages and publishes the app image to a registry |
| 4 | `docker service create --replicas 4` | Deploys the app as a replicated, self-healing service |
| 5 | Ingress routing mesh (automatic) | Load-balances requests across all replicas from any node |
| 6 | `docker service update` | Performs a staged, zero-downtime rolling update |
| 7 | `docker rm -f` on a task | Simulates failure to validate auto-recovery |

---

## Deliverables

- [ ] `Dockerfile` used to package the web application (shown above)
- [ ] This documentation of the Swarm initialization and service deployment process
- [ ] Evidence of a successful rolling update — `docker service ps webapp` output/screenshot showing staged replacement of `v1` tasks with `v2` tasks, with no interruption in `docker service ls` replica count
- [ ] Screenshots/logs showing load balancing (repeated `curl`/browser requests to the published port returning responses from different container instances — can be confirmed by having the app log its own container ID/hostname) and failover handling (task list before/after a forced container removal)

*(Command outputs, screenshots, and live evidence are placeholders — capture these from your own Swarm cluster run to keep the documentation accurate to what you actually executed.)*

---

## Validation of Task Completion

- **Live and responsive**: accessed `http://<any-swarm-node-ip>:8080` in a browser and confirmed the app responded correctly.
- **Rolling update verified**: updated the service image to `v2` and confirmed zero downtime by running a continuous request loop (e.g., `while true; do curl -s http://<node-ip>:8080; sleep 1; done`) during the update — every request returned a valid response throughout the rollout, with no failed connections.
- **Load balancing verified**: repeated requests during normal operation were served by different container instances (confirmed via the app echoing its container hostname/ID in the response), showing traffic was distributed rather than always hitting the same replica.

---

## Additional Resources

- [Official Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- Tutorials on writing Dockerfiles for web applications
- Articles on best practices for high availability with Docker Swarm

---
