# Day 72: Integrating Docker Container Monitoring with Grafana

## 📌 Introduction

Today's focus shifted from EC2-level metrics to container-level monitoring — using cAdvisor and Prometheus to expose Docker container metrics, then visualizing them in Grafana. This closes a gap from the earlier Grafana days: infrastructure metrics (CPU, memory, network) at the host level don't tell you what's happening *inside* individual containers, which matters once multiple apps share a single EC2 instance.

> **Note on scope:** This is self-study lab work completed independently on a personal AWS free-tier EC2 instance, not on production or employer infrastructure. All setup steps, screenshots, and outputs described below are from that sandbox.

---

## 🎯 Objectives

- Understand which Docker metrics matter (CPU, memory, network I/O, block I/O) and why
- Automate Docker environment setup on EC2 using User Data
- Deploy at least two containerized applications instrumented for metrics
- Expose Docker metrics via cAdvisor in Prometheus format
- Integrate Prometheus as a Grafana data source and build a dedicated container dashboard
- (Bonus) Wire up the full Prometheus + Grafana + Docker monitoring stack independently

---

## 🛠️ Deep Dive into Container Metrics

### 1. Understanding Docker Metrics

Before building anything, reviewed what Docker actually exposes per container:

- **CPU usage** — how much of the host's CPU each container is consuming, useful for spotting runaway processes
- **Memory usage** — resident memory vs. limit, critical for catching leaks or containers approaching their memory cap
- **Network I/O** — bytes in/out per container, useful for spotting unusual traffic patterns
- **Block I/O** — disk read/write, relevant for containers doing heavy file or database operations

These four map directly to a "golden signals" style of monitoring, just scoped to a single container instead of a whole host.

### 2. Setting Up Docker Metrics Collection

Docker's engine-level Prometheus metrics endpoint requires explicit daemon configuration, so I used **cAdvisor** (Google's Container Advisor) instead, since it runs as its own container and requires no Docker daemon config changes:

```bash
docker run -d \
  --name=cadvisor \
  --restart=unless-stopped \
  -p 8080:8080 \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -v /sys:/sys:ro \
  -v /var/lib/docker/:/var/lib/docker:ro \
  -v /dev/disk/:/dev/disk:ro \
  gcr.io/cadvisor/cadvisor:latest
```

Confirmed cAdvisor was exposing per-container metrics in Prometheus format at `http://<ec2-public-ip>:8080/metrics`.

### 3. Installing and Configuring Prometheus + the Grafana Connection

Rather than a dedicated "Docker plugin," the standard and better-supported path is: **cAdvisor → Prometheus → Grafana**, using Grafana's built-in Prometheus data source.

- Installed Prometheus on the same EC2 instance and configured a scrape job pointing at cAdvisor:
  ```yaml
  scrape_configs:
    - job_name: 'cadvisor'
      static_configs:
        - targets: ['localhost:8080']
  ```
- Started Prometheus and confirmed the cAdvisor target showed as **UP** on Prometheus's `/targets` page.
- In Grafana, added Prometheus as a new data source (**Connections → Data sources → Prometheus**), pointing at `http://localhost:9090`, and verified the connection with a test query.

---

## ✅ Tasks / Exercises

### Task 1 — Automated Docker Environment via EC2 User Data

Automated the Docker install so any new EC2 instance comes pre-configured, using a User Data script at launch:

```bash
#!/bin/bash
sudo apt-get update -y
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

Launched a fresh EC2 instance with this script attached and confirmed `docker --version` and `docker ps` worked immediately after boot, with no manual installation steps.

### Task 2 — Deploy and Instrument Two Containers

Deployed two simple containerized applications on the instance:

- A basic **to-do list app** container (Node.js-based sample app)
- An **Nginx** container serving a static page, reused from earlier Grafana/EC2 exercises as a second workload

Both containers were left running under Docker so cAdvisor (already scraping the Docker socket) automatically picked up their per-container metrics — no extra instrumentation was needed inside the app containers themselves, since cAdvisor collects at the Docker/cgroup level.

### Task 3 & 4 — Integrate Docker Metrics into Grafana and Visualize Them

With Prometheus already scraping cAdvisor and connected as a Grafana data source, built PromQL queries scoped to the two running containers, for example:

```promql
rate(container_cpu_usage_seconds_total{name="todo-app"}[5m])
container_memory_usage_bytes{name="todo-app"}
rate(container_network_receive_bytes_total{name="nginx"}[5m])
```

Then built a new Grafana dashboard, **"Docker Container Monitoring,"** with:

- CPU usage panel per container (time series)
- Memory usage panel per container (time series, with the container's memory limit as a reference line)
- Network I/O panel (in/out bytes) per container
- A `$container` dashboard variable (Prometheus label query on `name`) so any panel can be filtered to a specific container without rebuilding the dashboard
- A log panel pulling `docker logs` output via Grafana's Explore log view for the to-do app container, so logs and metrics are visible together

---

## 🌟 Bonus Challenge — Full Prometheus + Grafana + Docker Stack

Set up the complete monitoring pipeline independently (Prometheus scraping cAdvisor, Grafana visualizing Prometheus) as described above rather than relying solely on Docker's native Prometheus output — this ended up being the more standard and better-documented approach for container-level monitoring, and matched the pattern shown in the referenced tutorial videos.

---

## 🔑 Key Takeaways

- Docker doesn't need a special "Grafana plugin" for container monitoring — the standard, well-supported path is cAdvisor (metrics exporter) → Prometheus (scraper/storage) → Grafana (visualization), using Grafana's native Prometheus data source.
- cAdvisor collects metrics at the cgroup level, so containers don't need custom instrumentation to show up — this makes it a low-friction way to get visibility into any container running on the host.
- Automating environment setup with EC2 User Data means monitoring tooling can be baked into instance launch instead of configured manually every time.
- A `$container` template variable turns one dashboard into a reusable view across every container on the host, the same pattern used for `$instance_id` with EC2 metrics on Day 71.

---

## 📚 References

- [cAdvisor GitHub Repository](https://github.com/google/cadvisor)
- [Prometheus Docker/cAdvisor Monitoring Guide](https://prometheus.io/docs/guides/cadvisor/)
- [Grafana Prometheus Data Source Documentation](https://grafana.com/docs/grafana/latest/datasources/prometheus/)
- Monitoring Docker Using Grafana — reference video (course resource)
- Monitoring Docker Using Prometheus and Grafana — reference video (course resource)

---
