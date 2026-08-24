# Day 75: Alert Configuration for AWS Resources in Grafana Cloud

## 📌 Introduction

Day 74 covered alerting mechanics using a Docker/Prometheus metric. Today applies the same alerting engine to AWS itself — EC2 instance health and, just as importantly, AWS billing. Cost overruns are one of the most common "surprise" incidents in cloud environments, so treating billing as something to alert on (not just check manually) closes a real gap.

> **Note on scope:** This is self-study lab work completed independently using a personal AWS free-tier account and Grafana Cloud free tier, not on production or employer infrastructure. All configurations and outputs described below are from that sandbox, and no real billing thresholds beyond small test values were involved.

---

## 🎯 Objectives

- Understand what AWS-side signals are worth alerting on for EC2 (CPU, disk, network, status checks)
- Understand why billing alerts matter as part of a monitoring strategy, not just a finance concern
- Configure EC2-based alert rules in Grafana Cloud using the CloudWatch data source
- Configure a billing alert in Grafana Cloud, centralizing it alongside infrastructure alerts
- Compare this centralized approach against AWS's native billing alarms

---

## 🛠️ Understanding AWS Alerts

### 1. EC2 Instance Monitoring

Reviewed the CloudWatch metrics most worth alerting on for an EC2 instance:

- **CPU Utilization** — sustained high CPU can indicate an undersized instance or a runaway process
- **Disk I/O** (read/write bytes and ops) — spikes can signal an application bottleneck or unexpected load
- **Network In/Out** — unusual spikes can indicate anything from a traffic surge to a misconfigured process
- **Status Check Failed** (system + instance) — CloudWatch's own health signal for the instance; this one matters most for detecting outright failures rather than performance degradation

The key distinction drawn here: performance metrics (CPU, disk, network) are about *degradation*, while status checks are about *failure* — both deserve alerts, but they call for different urgency and response.

### 2. AWS Billing Alerts

AWS already provides native billing alarms via CloudWatch (used back on Day 51 for a billing threshold), but centralizing that alert inside Grafana Cloud alongside infrastructure alerts means:

- One alerting system and one set of contact points/notification policies for everything, instead of a separate AWS-only channel for cost alerts
- Billing trends are visible on the same dashboards as infrastructure usage, making it easier to correlate a cost spike with what actually caused it (e.g., an instance that was left running)

---

## ✅ Tasks / Exercises

### Task 1 — EC2 Alert Rules in Grafana Cloud

Using the CloudWatch data source connected in Grafana Cloud, built alert rules on the EC2 instance from earlier labs:

- **High CPU alert**
  - Query: CloudWatch `CPUUtilization` metric for the target instance
  - Condition: average above `75%` for a **5 minute** pending period
  - Evaluation interval: 1 minute

- **Status check failed alert**
  - Query: CloudWatch `StatusCheckFailed` metric
  - Condition: value greater than `0` for **1 minute** (no long pending period here, since a failed status check is urgent by nature)

- **Disk I/O alert**
  - Query: CloudWatch `DiskWriteBytes` metric
  - Condition: sustained spike above baseline for **5 minutes**, to catch unusual write activity without firing on routine short bursts

All three were set to notify the same email contact point configured on Day 74, with distinct alert names so each shows up clearly in the notification.

### Task 2 — Billing Alert in Grafana Cloud

Set up a billing alert centralized in Grafana Cloud rather than only relying on AWS's native CloudWatch billing alarm:

- Connected AWS Billing's `EstimatedCharges` CloudWatch metric (in `us-east-1`, where AWS billing metrics are published) as a query in Grafana
- **Condition:** fire when estimated charges exceed a small test threshold (`$5`) — kept intentionally low since this is a free-tier sandbox account, mirroring the same approach used for the Day 51 native CloudWatch billing alarm
- Verified the alert appeared correctly in Grafana Cloud's alert list with state `Normal`, and reviewed (without needing to actually breach it) how it would transition to `Pending` → `Firing` if usage crossed the threshold, consistent with the alert lifecycle covered on Day 74

---

## 🔑 Key Takeaways

- Not every EC2 metric deserves the same alert urgency — a failed status check needs a near-immediate alert, while CPU or disk spikes benefit from a pending period to filter out normal short-term noise.
- Billing deserves the same alerting discipline as infrastructure metrics — cost overruns are a real operational risk, not just something to check at the end of the month.
- Centralizing AWS billing alerts inside Grafana Cloud (rather than only using AWS's native CloudWatch billing alarm) means one alerting system, one set of contact points, and billing trends visible alongside the infrastructure metrics that likely caused them.
- Reusing the same contact points and alerting concepts from Day 74 across a completely different data source (CloudWatch vs. Prometheus) shows why Grafana's alerting engine being data-source-agnostic is genuinely useful in practice.

---

## 📚 References

- [AWS CloudWatch Data Source Plugin for Grafana](https://grafana.com/docs/grafana/latest/datasources/aws-cloudwatch/)
- [Grafana Alerting Documentation](https://grafana.com/docs/grafana/latest/alerting/)
- [AWS Billing and Cost Management — Monitoring Your Costs](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html)
- LinkedIn reference post on AWS + Grafana alerting (course resource)

---
