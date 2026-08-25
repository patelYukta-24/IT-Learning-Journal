# Day 76: Diving Deeper into Prometheus

## Introduction

I've already seen Prometheus in action integrating with Grafana in earlier days — feeding metrics into dashboards, powering PromQL queries and alert rules. Today's focus shifts from *using* Prometheus to *understanding* it: how its architecture pulls, stores, and processes time-series data, what components make up the ecosystem, and how it manages storage and retention at scale. This is core monitoring knowledge that sits right at the center of any DevOps/SRE role.

## Understanding Prometheus

Prometheus is an open-source monitoring and alerting toolkit originally built at SoundCloud and now a graduated CNCF project. Its architecture is purpose-built around **time-series data** — metrics identified by a name and a set of key-value labels, sampled at regular intervals. It works primarily on a **pull model**: Prometheus scrapes metrics from HTTP endpoints exposed by the systems it monitors, rather than waiting for those systems to push data to it. This design makes it efficient, self-contained, and easy to reason about, which is a big part of why it has become the de facto standard for Kubernetes and cloud-native monitoring.

## Today's Learning Goals

1. **Architecture** — Comprehend how Prometheus's architecture allows it to gather, store, process, and alert on metrics.
2. **Features** — Study the features that make Prometheus a popular choice: its data model, query language, and integration capabilities.
3. **Components** — Understand the various components (Prometheus server, Alertmanager, exporters, Pushgateway) and how they interact.
4. **Data Storage** — Learn about the database Prometheus uses and how it's optimized for time-series data.
5. **Data Retention** — Discover the default retention period and how it can be configured.

---

## Task 1: Prometheus Architecture — Data Flow, Bottlenecks, and Reliability

### How data flows through Prometheus

```
 ┌────────────┐     scrape (pull, HTTP/S)     ┌──────────────────┐
 │  Targets    │ <───────────────────────────  │ Prometheus Server │
 │ (exporters, │                                │  ┌─────────────┐ │
 │  apps with  │ ────────────────────────────>  │  │  Retrieval  │ │
 │  /metrics)  │      exposes /metrics           │  └──────┬──────┘ │
 └────────────┘                                 │         │        │
                                                  │         ▼        │
       ┌────────────────┐   service discovery    │  ┌─────────────┐ │
       │ Service         │ ───────────────────►  │  │    TSDB      │ │
       │ Discovery (K8s, │                        │  │ (local disk) │ │
       │ Consul, static) │                        │  └──────┬──────┘ │
       └────────────────┘                         │         │        │
                                                    │         ▼        │
                                                    │  ┌─────────────┐ │
                                                    │  │  PromQL      │ │
                                                    │  │  Engine      │ │
                                                    │  └──────┬──────┘ │
                                                    └─────────┼────────┘
                                    ┌────────────┬────────────┼───────────┐
                                    ▼            ▼            ▼           ▼
                              ┌──────────┐ ┌───────────┐ ┌─────────┐ ┌──────────┐
                              │ Grafana/ │ │ Rule       │ │ HTTP API│ │ Remote   │
                              │ API      │ │ Evaluation │ │ /Web UI │ │ Write    │
                              │ clients  │ │ (alerts &  │ │         │ │ (long-   │
                              │          │ │ recording) │ │         │ │ term)    │
                              └──────────┘ └─────┬──────┘ └─────────┘ └──────────┘
                                                  ▼
                                          ┌────────────────┐
                                          │  Alertmanager   │
                                          │ (dedup, group,  │
                                          │  route, silence)│
                                          └────────┬────────┘
                                                    ▼
                                      Slack / Email / PagerDuty / Webhook
```

**Step-by-step flow:**

1. **Service discovery** finds the list of targets to monitor — statically configured, or dynamically via Kubernetes SD, Consul, EC2, file-based SD, etc.
2. **Retrieval (scraping)** — the Prometheus server pulls metrics from each target's `/metrics` HTTP endpoint on a configured interval (default `15s`).
3. **Storage (TSDB)** — every scraped sample is written to Prometheus's local time-series database on disk, indexed by metric name and labels.
4. **Rule evaluation** — recording rules pre-compute expensive queries into new time series; alerting rules continuously evaluate PromQL expressions against stored data.
5. **Alertmanager** — when an alerting rule fires, Prometheus sends the alert to Alertmanager, which deduplicates, groups, silences, and routes it to the right notification channel.
6. **Querying/visualization** — PromQL queries run against the TSDB via the HTTP API, consumed by the Web UI, Grafana, or any API client.
7. **Remote write (optional)** — for long-term retention beyond local disk, Prometheus can forward samples to a remote storage backend (Thanos, Cortex, Mimir, VictoriaMetrics).

### Potential bottlenecks

- **Single-node scale limits** — a vanilla Prometheus server is a single process with local disk storage; a very high cardinality of label combinations (too many unique time series) can blow up memory and slow down queries — this is the classic "cardinality explosion" problem.
- **Scrape interval vs. target count** — too many targets or too-frequent scraping on a single server can saturate CPU/network before the next scrape cycle completes.
- **Disk I/O** — the TSDB is disk-heavy; slow or undersized storage becomes a bottleneck for both ingestion and query performance.
- **Long-range/heavy PromQL queries** — queries spanning long time windows or using expensive aggregations can spike memory usage and slow the whole server.
- **Alertmanager as a single point** — without clustering, a single Alertmanager instance is a bottleneck/SPOF for notification delivery.

### How Prometheus addresses high availability and reliability

- **Local-first, no external dependency** — each Prometheus server can scrape, store, and alert independently without relying on a remote database, which reduces the blast radius of any one component failing.
- **HA via redundant servers** — running two (or more) identical Prometheus servers scraping the same targets provides basic HA; if one goes down, the other keeps collecting data (though this doesn't dedupe/merge data by default — that's where Thanos/Cortex/Mimir come in for a global, deduplicated view).
- **Alertmanager clustering** — Alertmanager supports a gossip-based cluster mode so multiple instances can share silence/notification state and avoid duplicate or missed alerts.
- **Federation and remote-write** — Prometheus can federate metrics between servers or remote-write to long-term storage systems, decoupling short-term local reliability from long-term durability.
- **WAL (write-ahead log)** — the TSDB uses a write-ahead log so in-flight data survives a crash and is replayed on restart, protecting against data loss from unexpected shutdowns.

---

## Task 2: Key Features of Prometheus

- **Multi-dimensional data model** — metrics are identified by a name plus a set of key-value **labels** (e.g., `http_requests_total{method="GET", status="200"}`), which allows very flexible slicing and dicing of data compared to flat, hierarchical metric names.
- **PromQL (Prometheus Query Language)** — a purpose-built functional query language for selecting and aggregating time-series data in real time, supporting rate calculations, percentiles, joins across metrics, and more.
- **Pull-based scraping model** — Prometheus reaches out to targets rather than requiring them to push data, which simplifies target health checking (a failed scrape is itself a signal) and avoids the need for a message queue in front of the monitoring system.
- **Service discovery integrations** — native, first-class integration with Kubernetes, Consul, EC2, Azure, GCE, and more, so targets can be discovered dynamically instead of hand-maintained.
- **No reliance on distributed storage** — each Prometheus server is self-sufficient with local on-disk storage, making it simple to run and reason about.
- **Built-in alerting** — the Alertmanager component handles deduplication, grouping, silencing, inhibition, and routing to notification channels, decoupled from the core server.
- **Pushgateway for edge cases** — supports short-lived/batch jobs that can't be scraped directly by allowing them to push metrics to an intermediary gateway.
- **Rich exporter ecosystem** — hundreds of community-maintained exporters (Node Exporter, cAdvisor, Blackbox Exporter, MySQL Exporter, etc.) make it easy to instrument almost any system.
- **Native visualization + wide integration** — a built-in expression browser/Web UI, and first-class integration with Grafana and other visualization tools via a standard HTTP API.

### What makes it unique for time-series data specifically

- Purpose-built **TSDB storage engine** optimized for high-throughput sequential writes and efficient compression of repetitive numeric samples.
- **Label-based cardinality model** lets you query trends across dimensions (e.g., "error rate by service and region") in a single expression, something flat metric names or general-purpose databases handle far less naturally.
- **Recording rules** allow precomputing and caching expensive queries as new time series, keeping dashboards fast even as raw data volume grows.

---

## Task 3: Prometheus Components and How They Interact

| Component | Role |
|---|---|
| **Prometheus Server** | The core engine — service discovery, scraping (retrieval), local TSDB storage, and the PromQL query engine all live here. |
| **Exporters** | Small HTTP servers that expose metrics in Prometheus's text format for systems that don't natively expose them (e.g., Node Exporter for host metrics, cAdvisor for containers, MySQL Exporter for databases). |
| **Pushgateway** | An intermediary metrics cache for short-lived/batch jobs that finish before Prometheus could scrape them — the job pushes its final metrics here, and Prometheus scrapes the Pushgateway like any other target. |
| **Alertmanager** | Receives fired alerts from the Prometheus server, then handles deduplication, grouping, silencing, inhibition rules, and routing to notification channels (email, Slack, PagerDuty, webhooks). |
| **Service Discovery** | Mechanisms (Kubernetes SD, Consul, file SD, cloud-provider SD) that keep the list of scrape targets current without manual config updates. |
| **Client Libraries** | Language-specific libraries (Go, Java, Python, etc.) used to instrument application code directly, exposing custom application metrics on a `/metrics` endpoint. |
| **Web UI / HTTP API** | Built-in interface for running ad-hoc PromQL queries and browsing targets/alerts; the same API is what Grafana and other tools query against. |

**How they interact (in one line each):**
- The **server** discovers targets via **service discovery**, then scrapes metrics directly from instrumented apps (via **client libraries**) or from **exporters** standing in for uninstrumented systems.
- **Batch/short-lived jobs** push their metrics to the **Pushgateway**, which the server then scrapes normally.
- The server evaluates alerting rules against stored data and forwards firing alerts to **Alertmanager**, which decides who gets notified and how.
- External tools like Grafana, or the built-in **Web UI**, query the server's **HTTP API** to visualize and explore the data.

---

## Task 4: Prometheus's Time-Series Database (TSDB) — How It Works Under the Hood

Prometheus uses its own **custom-built local TSDB** (not a general-purpose database like MySQL or MongoDB), purpose-designed for append-heavy, high-cardinality time-series workloads.

**Key mechanics:**

- **Samples and chunks** — each unique time series (a metric name + label combination) is written as a sequence of `(timestamp, value)` samples. Recent samples are held in memory and periodically compacted into immutable **chunks** on disk, organized in 2-hour blocks by default.
- **Write-ahead log (WAL)** — every incoming sample is first written to a WAL before being held in memory, so a crash doesn't lose recent, not-yet-persisted data — on restart, Prometheus replays the WAL to recover state.
- **Block compaction** — the TSDB periodically merges smaller 2-hour blocks into larger ones in the background, reducing the number of files and improving query efficiency, similar in spirit to LSM-tree compaction in other databases.
- **Indexing** — an inverted index maps label name/value pairs to the time series that contain them, which is what makes label-based PromQL queries (e.g., `{job="api", status=~"5.."}`) fast even with millions of series.
- **Compression** — Prometheus uses a variant of **Gorilla compression** (delta-of-delta encoding for timestamps, XOR-based encoding for float values), which is extremely effective for typical metric data — often getting down to around 1–2 bytes per sample on disk.
- **Local-only by default** — data lives on local disk under Prometheus's data directory; there's no built-in clustering or replication for the local TSDB, which is why long-term/HA setups pair Prometheus with **Thanos, Cortex, Mimir, or VictoriaMetrics** for durable, horizontally scalable storage.

This design trades built-in distributed durability for simplicity, speed, and operational self-sufficiency — a single Prometheus server can ingest and query millions of samples with very low latency.

---

## Task 5: Default Data Retention and Configuration Implications

- **Default retention**: Prometheus retains local TSDB data for **15 days** by default.
- **How to change it**: controlled via command-line flags when starting the server:
  - `--storage.tsdb.retention.time=30d` — retain data for a given duration (e.g., 30 days, 6h, 1y).
  - `--storage.tsdb.retention.size=10GB` — retain data up to a given disk size instead of (or alongside) a time limit; whichever limit is hit first triggers deletion of the oldest blocks.

**Implications of storing more data:**
- **Pros**: enables longer historical trend analysis, capacity planning, and postmortem investigation of past incidents.
- **Cons**: significantly increases local disk usage, can slow down compaction and startup (WAL replay), and doesn't scale indefinitely on a single node — this is exactly the gap that remote-write to Thanos/Cortex/Mimir/VictoriaMetrics is designed to fill for true long-term storage.

**Implications of pruning more aggressively (shorter retention):**
- **Pros**: keeps disk usage and memory footprint small, keeps the server fast and lightweight, reduces backup/storage cost.
- **Cons**: loses the ability to investigate incidents or trends beyond the retention window — a shorter retention period is a real trade-off against operational visibility, so retention should be sized around how far back the team realistically needs to look, not left at the default without thinking about it.

---

## Resources

- [DevOpsSchool blog post on Prometheus](https://www.devopsschool.com/) — used for interview-prep style reinforcement of these concepts.
- [Official Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Prometheus Storage Documentation](https://prometheus.io/docs/prometheus/latest/storage/)

---