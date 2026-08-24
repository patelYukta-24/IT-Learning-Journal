# Day 74: Mastering Grafana Alerting

## 📌 Introduction

Every day so far has built toward *seeing* data — dashboards, panels, queries. Today's focus is on *acting* on that data: Grafana Alerting. A dashboard someone has to keep watching isn't proactive monitoring; an alert that fires and notifies you is.

> **Note on scope:** This is self-study lab work completed independently using a personal Grafana Cloud free-tier account and the EC2/Docker/Prometheus setup from Days 70–73, not on production or employer infrastructure. All configurations and outputs described below are from that sandbox.

---

## 🎯 Objectives

- Understand why alerting matters for reliability and reducing downtime
- Set up Grafana Cloud as a managed alternative to self-hosting Grafana/alerting/metrics infrastructure
- Learn how alert rules, conditions, and notification channels fit together
- Implement a basic threshold-based alert on an existing collected metric
- Review Grafana's official alerting documentation to understand the full workflow

---

## 🛠️ Introduction to Grafana Alerting

### 1. Importance of Alerting

Reviewed why alerting is treated as the backbone of monitoring rather than a nice-to-have:

- Dashboards require someone to actively look at them — alerts flip that to a push model, where the system tells you when something's wrong instead of you polling for it.
- Faster detection directly reduces downtime, since the gap between "issue starts" and "someone notices" is often the largest chunk of an incident's duration.
- A well-tuned alert (not too noisy, not too quiet) builds trust in the monitoring system itself — over-alerting leads to ignored alerts, which defeats the purpose entirely.

### 2. Capabilities of Grafana Alerting

Grafana's unified alerting system supports:

- Alert rules across any connected data source (CloudWatch, Prometheus, Loki, etc.) from a single alerting engine, rather than a separate alerting tool per data source
- Multiple notification channels — email, Slack, PagerDuty, webhooks, and more — configured as **contact points**
- **Notification policies** that route different alerts to different contact points/teams based on labels, so not every alert needs to go to the same inbox

### 3. Alert Rules and Conditions

An alert rule in Grafana is built from:

- A **query** (the same kind used in a dashboard panel) against a data source
- A **condition/threshold** evaluated against that query's result (e.g., "is the value above X for Y minutes")
- An **evaluation interval** — how often Grafana checks the condition
- A **pending period** — how long the condition must stay true before the alert actually fires, which helps avoid firing on a single noisy data point

---

## ✅ Tasks / Exercises

### Task 1 — Set Up Grafana Cloud

Signed up for the Grafana Cloud free tier to avoid self-managing the alerting/metrics/logging infrastructure that self-hosted Grafana would otherwise require:

- Created a Grafana Cloud instance and logged into the hosted Grafana UI
- Connected the same Prometheus metrics used in Days 72–73 as a remote-write target, so the EC2 sandbox's container metrics were available inside Grafana Cloud's managed environment
- Confirmed dashboards built earlier could be recreated/imported against the Grafana Cloud instance

### Task 2 — Implement a Sample Alert

Built a basic threshold alert on a metric already being collected (container CPU usage from the Day 72 Docker dashboard):

- **Query:** `rate(container_cpu_usage_seconds_total{name="todo-app"}[5m])`
- **Condition:** fire when the value is above `0.8` (80% CPU) for a **5 minute** pending period, so a brief spike doesn't trigger a false alarm
- **Evaluation interval:** every 1 minute
- **Contact point:** configured email as the notification channel for this alert
- Tested the alert by manually generating CPU load inside the container (`stress` utility) and confirmed the alert transitioned from **Normal → Pending → Firing**, and that the email notification arrived correctly
- Removed the load and confirmed the alert resolved back to **Normal** and a resolved notification was sent

### Task 3 — Review Grafana's Alerting Documentation

Read through Grafana's official alerting documentation to map out the full workflow end to end:

- How alert rules, folders, and evaluation groups are organized
- The distinction between contact points (where notifications go) and notification policies (which alerts go where)
- Alert states (`Normal`, `Pending`, `Firing`, `NoData`, `Error`) and what each one means for troubleshooting

---

## 🔑 Key Takeaways

- Alerting turns monitoring from something you have to check into something that checks itself — the biggest reliability win isn't the dashboard, it's not having to watch it.
- A pending period matters as much as the threshold itself — without one, a single noisy sample can fire (and immediately resolve) an alert, which erodes trust in the system.
- Grafana Cloud removes the operational overhead of self-hosting the alerting/metrics stack, which is worth it for a solo/small-scale setup even outside of production use.
- Contact points and notification policies are separate concerns by design — it's the difference between "where can this alert go" and "where should this specific alert go," which matters once there's more than one team or severity level involved.

---

## 📚 References

- [Grafana Alerting Documentation](https://grafana.com/docs/grafana/latest/alerting/)
- [Grafana Cloud Documentation](https://grafana.com/docs/grafana-cloud/)
- [Alert Rules and Conditions — Grafana Documentation](https://grafana.com/docs/grafana/latest/alerting/alerting-rules/)
- [Contact Points and Notification Policies — Grafana Documentation](https://grafana.com/docs/grafana/latest/alerting/fundamentals/notifications/contact-points/)

---
