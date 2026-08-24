# Day 71: Advanced Grafana Integrations and Dashboard Customizations

## 📌 Introduction

Building on Day 70's work of connecting EC2 instances to Grafana via CloudWatch, this day focused on going beyond a single-source setup — pulling in a second data source, installing a community plugin, building more advanced visualizations, and layering annotations and logs onto existing dashboards. The goal was to turn a basic CloudWatch dashboard into a more complete, interactive monitoring workspace.

> **Note on scope:** This is self-study lab work completed independently, not on production infrastructure or as part of any employer environment. All screenshots, configurations, and outputs described below are from a personal AWS free-tier sandbox and local Grafana instance.

---

## 🎯 Objectives

- Add a second data source to Grafana beyond AWS CloudWatch
- Explore and install a plugin from the Grafana plugin catalog
- Build advanced dashboard panels (heatmap, templated/variable-driven panels)
- Configure annotations to mark events on graphs
- Wire up log visibility (CloudWatch Logs or Loki) alongside metrics

---

## 🛠️ Enhancing Monitoring Capabilities with Grafana

### 1. Exploring Additional Data Sources

CloudWatch only tells part of the story — infrastructure-level metrics like CPU, memory, and network. To get a fuller picture, I added a second data source:

- **Data source added:** Infinity (JSON/CSV/REST API data source plugin) — used to pull a small public REST API's JSON response into a panel, and to simulate what a SQL or web-analytics-style data source would look like in a real setup.
- **Why this one:** Unlike CloudWatch, this doesn't require AWS IAM permissions or a specific metric namespace, which made it a good low-friction way to practice connecting Grafana to a completely different kind of source in a sandbox environment.
- Configured under **Connections → Data sources → Add data source**, then built a simple panel querying a public JSON endpoint to confirm the connection worked end-to-end.

*(In a production setup, this same slot could just as easily be a MySQL/PostgreSQL data source for application metrics, or a web analytics source — the connection pattern is the same: add source → authenticate → query → visualize.)*

### 2. Incorporating a Plugin

Browsed the [Grafana Plugins catalog](https://grafana.com/grafana/plugins/) and installed:

- **Plugin:** Infinity Data Source (also serves as the plugin used above) — chosen because it's officially maintained, doesn't need external infrastructure to test, and directly extends what Grafana can query.
- **Installation:**
  ```bash
  grafana-cli plugins install yesoreyeram-infinity-datasource
  sudo systemctl restart grafana-server
  ```
- Verified installation under **Administration → Plugins**, then confirmed it appeared as a selectable data source type.

### 3. Customizing Dashboards

- **Heatmap panel:** Built a heatmap visualization on top of CPU utilization metrics (bucketed by time) to see load variation patterns more clearly than a standard time-series line allows.
- **Templating/Variables:** Added a dashboard variable (`$instance_id`) sourced from the CloudWatch data source's EC2 instance dimension, so a single dashboard can switch between EC2 instances via a dropdown instead of duplicating panels per instance.
- Also added a `$time_range` convenience using Grafana's built-in time picker combined with the variable dropdown, so switching instance and time window together updates every panel on the dashboard at once.

### 4. Annotations and Logs

- **Annotations:** Manually added annotations on the CPU/network graphs to mark a simulated "deployment" event (a test EC2 reboot) and confirmed the vertical marker lines appeared correctly aligned with the metric spike.
- **Logs:** Enabled CloudWatch Logs as a queryable source in Grafana's **Explore** view, pointed at the EC2 instance's system log group, and confirmed logs could be viewed side-by-side with the metrics panels for basic correlation.

---

## ✅ Tasks / Exercises

### Task 1 — Real-time + Historical Dashboard with Annotations

Built a single dashboard combining:
- A live-updating CPU/network panel (short refresh interval) for real-time data
- A longer time-range panel (24h) for historical trend context
- An annotation marking the test reboot event, visible on both panels

This confirmed that a single dashboard can hold both "what's happening right now" and "what happened over the last day" views, with annotations tying specific events to metric changes.

### Task 2 — Custom Visualization from the New Data Source

Created a table/stat panel driven by the Infinity data source, pulling a small public JSON API's fields into Grafana-native visualizations (stat panel + table), demonstrating that Grafana isn't limited to time-series infrastructure metrics — it can visualize arbitrary structured data from non-AWS sources using the same dashboard.

---

## 🔑 Key Takeaways

- Grafana's real strength shows up once more than one data source is in play — metrics, logs, and arbitrary API data can sit on the same dashboard.
- The plugin ecosystem makes it straightforward to extend Grafana without touching its core — install, restart, configure.
- Dashboard variables turn a single dashboard into a reusable template instead of one-off panels per resource.
- Annotations are a small feature with an outsized payoff for incident correlation — a vertical line on a graph next to a metric spike answers "what changed here?" immediately.

---

## 📚 References

- [Grafana Data Sources Documentation](https://grafana.com/docs/grafana/latest/datasources/)
- [Grafana Plugins Catalog](https://grafana.com/grafana/plugins/)
- [Grafana Annotations Documentation](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/annotate-visualizations/)
- [Grafana Template Variables Documentation](https://grafana.com/docs/grafana/latest/dashboards/variables/)

---
