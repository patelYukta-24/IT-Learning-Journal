# Day 68: Introduction to Prometheus and its Ecosystem + Mastering Resource Monitoring with Grafana

## Table of Contents
- [Part 1: Introduction to Prometheus and its Ecosystem](#part-1-introduction-to-prometheus-and-its-ecosystem)
  - [Introduction to Prometheus](#introduction-to-prometheus)
  - [Metrics: The Pulse of Your Systems](#metrics-the-pulse-of-your-systems)
  - [PromQL: Unleashing Querying Power](#promql-unleashing-querying-power)
  - [Prometheus's Components Unpacked](#prometheuss-components-unpacked)
  - [Why Prometheus?](#why-prometheus)
  - [Grafana: The Window to Your Data](#grafana-the-window-to-your-data)
  - [Loki and Promtail: Streamlining Logs](#loki-and-promtail-streamlining-logs)
  - [How It All Fits Together](#how-it-all-fits-together)
- [Part 2: Mastering Resource Monitoring with Grafana](#part-2-mastering-resource-monitoring-with-grafana)
  - [What is Grafana?](#what-is-grafana)
  - [Features of Grafana](#features-of-grafana)
  - [Types of Monitoring with Grafana](#types-of-monitoring-with-grafana)
  - [Databases Compatible with Grafana](#databases-compatible-with-grafana)
  - [Metrics and Visualizations in Grafana](#metrics-and-visualizations-in-grafana)
  - [Grafana vs Prometheus (Overview)](#grafana-vs-prometheus-overview)
  - [Tasks / Exercises — Completed](#tasks--exercises--completed)
  - [Example Prompts Used](#example-prompts-used)
- [Key Takeaways](#key-takeaways)

---

# Part 1: Introduction to Prometheus and its Ecosystem

## Introduction to Prometheus

Prometheus is an open-source monitoring and alerting toolkit that has become the de-facto standard for observability in cloud-native and containerized environments. At its core sits a **time-series database** — a data store purpose-built to hold values that change over time, each one stamped with a timestamp. This makes it well suited to answering questions like "how did CPU usage change over the last hour?" rather than just "what is CPU usage right now?"

What sets Prometheus apart is a combination of:
- **Simplicity** — a straightforward pull-based model and a single, self-contained binary with no heavyweight dependencies to run.
- **Scalability** — it can comfortably monitor everything from a handful of services to large, dynamic, container-orchestrated fleets.

Rather than being just a dashboarding tool, Prometheus is the layer that continuously watches a system's applications and infrastructure, collects numerical signals about their behavior, and makes that data queryable — forming the foundation that alerting and visualization tools build on top of.

## Metrics: The Pulse of Your Systems

Metrics are the numerical measurements that describe how a system is behaving at any given moment — things like request counts, error rates, memory usage, or response latency. If logs tell you *what happened* in detail and traces tell you *the path a request took*, metrics tell you *how healthy the system is* in aggregate, at a glance.

Prometheus collects metrics by **scraping** them: at regular intervals, it reaches out over HTTP to each configured target and pulls the current values of whatever metrics that target exposes. Once collected, each metric is stored in Prometheus's time-series database, indexed by its name and a set of key-value **labels** (e.g. `method="GET"`, `status="500"`) that let the same metric be sliced and filtered in many different ways.

Because every data point is timestamped and stored efficiently, Prometheus enables real-time analysis: you can see the current value of a metric, or query how it has trended over the last five minutes, five hours, or several days.

## PromQL: Unleashing Querying Power

**PromQL (Prometheus Query Language)** is the language used to ask Prometheus questions about the metrics it has collected. It's what turns a database of raw numbers into meaningful insight — whether that's a single instantaneous value, a rate of change, or an aggregation across many instances of a service.

At a basic level, PromQL lets you:
- Select a metric by name (e.g. `http_requests_total`) and filter it by labels (e.g. `http_requests_total{status="500"}`).
- Calculate rates of change over time, such as requests-per-second from a continuously increasing counter.
- Aggregate values across dimensions — for example, summing request counts across every instance of a service to get a total.
- Combine and compare metrics using arithmetic and logical operators.

PromQL results feed directly into both **alerting rules** (deciding when something needs attention) and **dashboards** (visualizing trends), which makes it the common thread connecting Prometheus's data collection to everything built on top of it.

## Prometheus's Components Unpacked

Prometheus isn't a single monolithic tool — it's an ecosystem of cooperating pieces, each responsible for one part of the monitoring pipeline:

- **Service Discovery** — Automatically detects new service instances as they appear (e.g. new containers or pods spinning up) so that Prometheus knows what to scrape without needing every target hand-configured. This is especially important in dynamic, auto-scaling environments where instances come and go frequently.
- **Exporters** — Small helper processes that fetch metrics from third-party systems that don't natively expose data in Prometheus's format, and translate them into something Prometheus can scrape (e.g. a database exporter that exposes database health metrics, or a node exporter that exposes host-level system metrics).
- **Alerting Rules** — PromQL-based rules that continuously evaluate metric conditions and fire when a defined threshold or condition is met (e.g. "error rate above 5% for 5 minutes"), turning raw metrics into actionable signals.
- **AlertManager** — Receives fired alerts from Prometheus and takes care of what happens next: deduplicating, grouping, silencing, and routing them to the right notification channel (email, Slack, PagerDuty, etc.) so the right people are notified in the right way.

Together, these components form a pipeline: service discovery finds targets → exporters make third-party data scrapable → Prometheus collects and stores it → alerting rules evaluate it → AlertManager delivers the resulting alerts.

## Why Prometheus?

Adopting Prometheus isn't just about having numbers to look at — it's about building a monitoring foundation that lets a team move from being reactive to being proactive. By continuously collecting metrics, evaluating alerting rules against them, and routing meaningful alerts through AlertManager, a team can catch degrading performance or emerging failures *before* they escalate into outages, rather than finding out only after users are already affected. In that sense, Prometheus supports building a system that is **responsive** (it reacts quickly to changing conditions) and **resilient** (issues are surfaced and addressed early, reducing overall downtime).

## Grafana: The Window to Your Data

If Prometheus is responsible for collecting and storing metrics, **Grafana** is responsible for making sense of them visually. It takes the raw, numerical time-series data sitting in Prometheus (or other data sources) and turns it into dashboards — graphs, gauges, tables, and other visual panels that make trends and anomalies immediately obvious, rather than requiring someone to manually run PromQL queries to notice a problem.

**Crafting Effective Dashboards** — A good Grafana dashboard is built around **actionable insight**, not just data for its own sake. That means choosing the right metrics to surface, arranging panels so the most important indicators are visible at a glance, and designing dashboards so that a team member can look at them and immediately understand the current state of a system — without needing to dig through raw data first.

**Data Sources and Panels** — Grafana connects to data sources like Prometheus (though it supports many others as well) and queries them to populate its visualizations. Each **panel** on a dashboard represents a chosen way of displaying a specific query's result — line graphs for trends over time, gauges for current values against thresholds, tables for detailed breakdowns, and more.

**Alerting in Grafana** — In addition to Prometheus's own alerting rules and AlertManager, Grafana can also be configured to evaluate conditions on the data it displays and fire its own notifications, keeping teams proactive without relying solely on someone actively watching a dashboard.

**Why Grafana?** — Grafana is valuable because raw metrics, on their own, are hard for people to reason about quickly. By presenting that data in a clear, visual, and shareable format, Grafana makes monitoring data accessible to the whole team — not just to whoever knows how to write PromQL — which fosters better collaboration and faster, more informed decisions.

## Loki and Promtail: Streamlining Logs

Metrics tell you *that* something is wrong; logs tell you *what exactly* happened. That's where Loki and Promtail come in, extending the same monitoring philosophy to log data.

**Loki** is a log aggregation system designed to be lightweight and cost-effective. Unlike some traditional logging systems that index the full text of every log line, Loki takes a more efficient approach — it indexes only a small set of labels (similar in spirit to how Prometheus labels metrics) and keeps the actual log content compressed and unindexed. This keeps storage costs and complexity much lower, while still allowing logs to be found quickly through their labels. Loki is designed to integrate closely with Grafana, so logs can be explored right alongside metrics.

**Promtail** is the agent that gets logs *into* Loki in the first place. It runs close to the log sources (e.g. on each host or node), discovers where relevant log files live, attaches useful labels to them, and ships that log data over to Loki's centralized store — acting essentially as the courier between scattered log sources and Loki's single, queryable repository.

**From Loki to Grafana: Visualizing Logs** — Once logs are collected by Promtail and stored in Loki, Grafana can query Loki the same way it queries Prometheus — letting logs be explored, filtered, and viewed directly inside the same dashboards used for metrics. This means a team investigating an issue can go from "the error-rate graph just spiked" straight to "here are the actual log lines from that exact time window," all within one interface.

## How It All Fits Together

| Tool | Role |
|---|---|
| **Prometheus** | Collects, stores, and lets you query metrics (the "what's the current state" data) |
| **PromQL** | The query language used to analyze and alert on those metrics |
| **AlertManager** | Routes fired alerts to the right people/channels |
| **Grafana** | Visualizes metrics (and logs) as dashboards for human-friendly insight |
| **Loki** | Aggregates and stores logs efficiently, using label-based indexing |
| **Promtail** | Ships logs from their sources into Loki |

Together, this stack (often nicknamed **PLG** — Prometheus, Loki, Grafana) covers both halves of observability: metrics for the high-level pulse of a system, and logs for the detailed story behind any anomaly — with Grafana tying both together into one visual, actionable layer.

---

# Part 2: Mastering Resource Monitoring with Grafana

## What is Grafana?

Grafana is a multi-platform, open-source analytics and interactive visualization web application. Once connected to a supported data source, it produces charts, graphs, and alerts through the web — acting as the visual and alerting layer on top of whatever system is actually storing the raw data (metrics, logs, or otherwise).

## Features of Grafana

- **Rich panel and dashboard library** — line graphs, bar charts, gauges, heatmaps, tables, and more, all assembled into customizable dashboards.
- **Alert rules** — conditions that continuously evaluate query results and fire notifications when thresholds are breached.
- **Multi-tenant support** — organizations and teams can be isolated within the same Grafana instance, each with their own dashboards, users, and permissions.
- **Broad data source integration** — connects to a large number of databases and monitoring backends (Prometheus, InfluxDB, Elasticsearch, MySQL, PostgreSQL, Loki, and many others) through a plugin-based data source model.

## Types of Monitoring with Grafana

Because Grafana is data-source agnostic, the same tool can be pointed at very different kinds of monitoring data depending on what's connected to it:

- **Infrastructure Monitoring** — CPU, memory, disk, and network metrics from servers, VMs, or containers (typically sourced from Prometheus via exporters).
- **Application Performance Monitoring (APM)** — request rates, latency, and error rates for applications and services.
- **Log Monitoring** — via Loki, viewing and searching log lines alongside metric dashboards.
- **Business/Custom Metrics** — dashboards built on top of relational or analytical databases (e.g. MySQL, PostgreSQL) to track operational or business KPIs alongside technical ones.

## Databases Compatible with Grafana

| Data Source | Typical Use |
|---|---|
| **Prometheus** | Time-series infrastructure and application metrics |
| **InfluxDB** | General-purpose time-series data, often for IoT/sensor-style metrics |
| **Elasticsearch** | Log and full-text search data |
| **MySQL / PostgreSQL** | Relational/business data queried directly for dashboards |
| **Loki** | Log aggregation, label-indexed |
| **CloudWatch / Azure Monitor / Google Cloud Monitoring** | Native cloud-provider metrics |

## Metrics and Visualizations in Grafana

Metrics reach Grafana through a configured data source — for example, Prometheus, which has already collected and stored them via scraping. Grafana doesn't collect or store the metrics itself; it queries the data source (using that source's native query language, e.g. PromQL for Prometheus) and renders the results as a **panel**. A **dashboard** is simply a collection of panels arranged together, each one built from its own query and its own choice of visualization type (graph, gauge, stat, table, etc.).

To create a first panel: add a new panel to a dashboard, select the data source (e.g. Prometheus), write a query (e.g. a PromQL expression for CPU usage), and choose a visualization type that best represents that data — a time-series graph for something that changes continuously, or a gauge for a single current value against a threshold.

## Grafana vs Prometheus (Overview)

At a high level: **Prometheus collects and stores metrics; Grafana visualizes them.** They are complementary, not competing, tools — Prometheus does the "backend" work of scraping, storing, and providing a query language for time-series data, while Grafana does the "frontend" work of turning query results into human-readable dashboards. (A full comparison is given in Task 4 below.)

---

## Tasks / Exercises — Completed

### Task 1: Create a Dashboard

**Goal:** Assemble a Grafana dashboard with different panel types, visualizing CPU usage, memory consumption, and network I/O from a Prometheus data source.

**Answer / Approach:**

1. In Grafana, go to **Dashboards → New → New Dashboard → Add visualization**, and select the existing Prometheus data source.
2. Add three panels, each with a different panel type and PromQL query, assuming metrics are being supplied by `node_exporter`:

   | Panel | Type | Example PromQL Query |
   |---|---|---|
   | CPU Usage | Time-series graph | `100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` |
   | Memory Consumption | Gauge | `(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100` |
   | Network I/O | Time-series graph (dual series) | `rate(node_network_receive_bytes_total{device="eth0"}[5m])` and `rate(node_network_transmit_bytes_total{device="eth0"}[5m])` |

3. For CPU, a **time-series graph** best shows how usage trends over time.
4. For memory, a **gauge** panel gives an immediate visual sense of "how full is memory right now," with color thresholds (e.g. green under 70%, yellow 70–90%, red above 90%).
5. For network I/O, a time-series graph with two series (receive and transmit) shows inbound vs outbound traffic on the same timeline.
6. Arrange all three panels on one dashboard row so infrastructure health can be assessed at a glance, and save the dashboard (e.g. named "Infrastructure Overview").

This demonstrates the core Grafana workflow: one data source, multiple panel types, each chosen to fit the shape of the metric it displays.

### Task 2: Alert Configuration

**Goal:** Set up alerting rules for the dashboard so notifications are received for critical thresholds like high CPU or memory usage.

**Answer / Approach:**

1. From the CPU usage panel, open the panel's **Alert** tab (in modern Grafana, this is managed under **Alerting → Alert rules**, referencing the panel's query).
2. Define a rule condition, e.g.:
   - **Query:** the same CPU usage expression from Task 1.
   - **Condition:** `IS ABOVE 85` evaluated over a period of `5m` (to avoid alerting on brief spikes).
3. Repeat for memory: condition `IS ABOVE 90` over `5m`, using the memory query from Task 1.
4. Set the **evaluation interval** (e.g. every 1 minute) and a **"for"** duration (e.g. 5 minutes) so the condition must hold consistently before firing — this avoids noisy, flapping alerts from momentary blips.
5. Configure a **notification policy/contact point** (e.g. email or Slack webhook) so that when a rule fires, the right channel receives it.
6. Add clear labels/annotations to each rule (e.g. `severity: critical`) so downstream routing and on-call teams can prioritize appropriately.

This ensures the dashboard isn't just something a person has to remember to check — it proactively raises a flag the moment CPU or memory usage crosses into concerning territory.

### Task 3: Data Source Integration

**Goal:** Connect a new data source to the Grafana instance — ideally one not previously used — and explore its integration capabilities.

**Answer — Chosen data source: InfluxDB** (a general-purpose time-series database, distinct from Prometheus in that it uses a push-based write model rather than Prometheus's pull-based scraping).

**Approach:**

1. In Grafana, go to **Connections → Data sources → Add data source** and select **InfluxDB**.
2. Provide the InfluxDB connection details: URL/endpoint, organization, bucket (for InfluxDB 2.x), and an authentication token (or username/password for 1.x).
3. Set the query language — InfluxDB 2.x uses **Flux**, while 1.x uses **InfluxQL** — and choose accordingly based on the InfluxDB version being connected to.
4. Click **Save & Test** to confirm Grafana can successfully reach and authenticate against the InfluxDB instance.
5. Create a new panel using this data source, writing a basic Flux/InfluxQL query against a sample bucket/measurement to confirm data flows through correctly.

**What this shows:** unlike Prometheus, where Grafana queries in PromQL, each data source Grafana connects to brings its own query language and connection model — Grafana's role is to abstract the *visualization* layer consistently, while still needing source-specific configuration and query syntax underneath.

### Task 4: Compare and Contrast — Grafana vs Prometheus

**Answer:**

Grafana and Prometheus are often mentioned in the same breath, but they solve different problems and are designed to be used **together**, not as alternatives to each other.

**Prometheus's strength** is in the collection and storage side of monitoring: it pulls metrics from configured targets on a schedule, stores them efficiently as time series, and provides PromQL for querying and defining alerting rules directly against that stored data. Its alerting (combined with AlertManager) is tightly coupled to the metrics it collects, making it a strong choice when the priority is having a reliable, self-contained metrics pipeline with alerting baked close to the data.

**Grafana's strength** is in visualization and cross-source presentation: it doesn't collect or store data on its own, but it can connect to many different backends at once (Prometheus, InfluxDB, Elasticsearch, MySQL, Loki, etc.) and present all of it through consistent, customizable dashboards. Its strength is making data understandable and actionable for a broader audience — including people who don't know a query language — and unifying visibility across systems that Prometheus alone wouldn't cover (like logs or business data).

**When to choose one over the other:**
- If the need is purely "collect and store infrastructure/application metrics reliably, with alerting close to the data" — Prometheus (with AlertManager) can operate on its own, no Grafana required.
- If the need is "visualize data from multiple systems in one place, in a way that's easy for a whole team — including non-technical stakeholders — to read" — Grafana is essential, regardless of whether the underlying data source is Prometheus or something else.
- In practice, most real-world monitoring setups use **both together**: Prometheus as the metrics engine doing the collecting, storing, and often the alerting, with Grafana layered on top as the shared visualization and dashboarding surface — each playing to its own strength rather than one replacing the other.

---

## Example Prompts Used

As part of working through this lab, the following prompts were used to get help drafting PromQL expressions and alert-rule wording for the exercises above:

**Prompt 1:**
> "Give me a PromQL expression using node_exporter metrics to calculate overall CPU usage percentage as a time-series graph in Grafana."

**Prompt 2:**
> "Write a Grafana alert rule condition for memory usage above 90% sustained for 5 minutes, based on node_memory_MemTotal_bytes and node_memory_MemAvailable_bytes."

---

## Key Takeaways

- Prometheus is a time-series database and monitoring toolkit built around a simple, scalable, pull-based metrics model; PromQL is the language used to query and act on the metrics it collects.
- Prometheus's ecosystem — service discovery, exporters, alerting rules, and AlertManager — together form a full pipeline from metric collection to alert delivery.
- Grafana is a visualization and alerting layer, not a data store — it connects to data sources like Prometheus, InfluxDB, Elasticsearch, and more, and turns their data into dashboards built from panels (time-series graphs for trends, gauges for current-state thresholds, etc.).
- Loki and Promtail extend the same efficient, label-based philosophy to logs — Promtail ships logs, Loki stores them cheaply, and Grafana visualizes them alongside metrics.
- Grafana alerting lets you get proactively notified when thresholds like high CPU or memory are crossed, instead of relying on someone actively watching a dashboard; the same dashboarding skillset carries over even when the underlying data source (Prometheus, InfluxDB, etc.) changes.
- Prometheus and Grafana are complementary: Prometheus is strong at collecting, storing, and alerting on metrics at the source; Grafana is strong at unifying and visualizing data — often from several sources at once — for a broader audience.
