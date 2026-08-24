# Day 73: Building and Refining Grafana Dashboards

## 📌 Introduction

Today was less about adding new integrations and more about consolidation — taking everything set up across Days 70–72 (CloudWatch, a second data source, cAdvisor/Prometheus for Docker) and practicing the actual craft of dashboard building: panel types, query efficiency, and legends that are readable at a glance rather than just technically correct.

> **Note on scope:** This is self-study lab work completed independently on a personal AWS free-tier EC2 instance running the Prometheus/cAdvisor/Grafana stack from Day 72, not on production or employer infrastructure. All queries, panels, and outputs described below are from that sandbox.

---

## 🎯 Objectives

- Understand what separates a "good" dashboard from a data dump
- Practice panel configuration across different visualization types
- Write efficient, purpose-built PromQL queries instead of over-broad ones
- Build a panel showing request rate by route, with a clear, renamed legend
- Save a dashboard with a descriptive name reflecting its actual purpose

---

## 🛠️ Understanding the Power of Dashboards

### 1. What Makes a Good Dashboard

Before building, thought through the difference between a dashboard that just *shows* data and one that's actually useful:

- **Purpose first** — every panel should answer a specific question ("is CPU healthy?", "which route is slow?") rather than existing because the metric happened to be available.
- **Signal over noise** — fewer, well-chosen panels beat a dashboard crammed with every metric a data source can produce.
- **Actionable, not just descriptive** — a good panel makes the next step obvious (e.g., a threshold-colored gauge tells you immediately whether something needs attention, without reading exact numbers).

### 2. Panel Configuration

Reviewed and used a few different panel types for different kinds of "story":

- **Time series** — for anything trend-based (request rate, CPU over time)
- **Gauge** — for a single current value against a threshold (e.g., current memory usage vs. limit)
- **Table** — for comparing multiple containers/routes side by side at a glance
- **Stat panel** — for a single glanceable number (e.g., total requests in the last 5 minutes)

The goal in each case was picking the panel type the data actually needs, rather than defaulting to a time-series graph for everything.

### 3. Query Efficiency

Kept queries scoped and rate-windowed rather than pulling raw counters, since raw cumulative counters aren't useful for dashboards and get expensive to render at scale:

- Used `rate(...[5m])` instead of querying raw counter values directly, so the data source only computes a rolling rate rather than every raw sample.
- Avoided unnecessary label matching (`{}` wildcards) in favor of specific label selectors, to keep each query's result set small and the panel responsive.

---

## ✅ Tasks / Exercises

### Task 1 — New Dashboard and Panel Types

Created a new Grafana dashboard and explored the panel type picker (time series, gauge, table, stat, bar chart) before deciding which types fit which metrics, per the notes above.

### Task 2 — Request Rate by Route Panel

Added a panel using a PromQL query to show the rate of requests over the last five minutes, grouped by route:

```promql
sum(rate(http_requests_total[5m])) by (route)
```

Confirmed the query executed correctly in the panel editor (Shift+Enter to run) and returned one time series per route.

### Task 3 — Legend Renaming

Applied a legend template to make each series human-readable instead of showing the raw label set:

```
{{route}}
```

This renamed each line in the legend to just the route path (e.g., `/`, `/api/tasks`) instead of the full metric name plus label set, making the panel readable at a glance.

### Task 4 — Panel Appearance and Titles

Customized the panel:
- **Title:** "Request Rate by Route (5m)" — descriptive enough to understand the panel without opening the query editor
- Set the Y-axis unit to requests/sec
- Adjusted line width and enabled the legend table view (with min/max/current columns) for quick comparison across routes

### Task 5 — Save the Dashboard

Saved the dashboard as **"Application Traffic Overview"** — a name that reflects its actual purpose (route-level traffic patterns) rather than a generic label like "Dashboard 1," so it's identifiable later alongside the CloudWatch and Docker dashboards from Days 70–72.

---

## 🤔 Looking Ahead

With a working baseline dashboard in place, a few directions worth exploring next:

- **Error rate alongside request rate** — a route with high traffic but also a rising error rate tells a very different story than raw volume alone
- **Latency percentiles (p50/p95/p99)** per route, to catch tail-latency issues that averages hide
- **Panel arrangement** — grouping traffic panels together at the top and resource panels (CPU/memory from Day 72) below, so the dashboard reads top-down from "what's happening" to "why"

---

## 🔑 Key Takeaways

- A dashboard's value comes from the questions it answers, not the number of panels it has — fewer, well-chosen panels beat a wall of graphs.
- Picking the right panel type (gauge vs. time series vs. table) matters as much as the query itself for making data understandable at a glance.
- Rate-windowed PromQL queries (`rate(...[5m])`) scale far better than raw counters, especially once a dashboard is queried frequently or by multiple viewers.
- Legend templates (`{{label}}`) turn a wall of raw label strings into something readable in a few seconds.
- Descriptive dashboard names matter once there's more than one dashboard in the org — "Application Traffic Overview" is far more useful than "Dashboard 1" six dashboards later.

---

## 📚 References

- [Grafana Panels and Visualizations Documentation](https://grafana.com/docs/grafana/latest/panels-visualizations/)
- [PromQL Basics — Prometheus Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Grafana Legend Configuration Documentation](https://grafana.com/docs/grafana/latest/panels-visualizations/configure-legend/)

---