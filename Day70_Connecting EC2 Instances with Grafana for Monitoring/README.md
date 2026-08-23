# Day 70: Connecting EC2 Instances with Grafana for Monitoring

## Table of Contents
- [Introduction](#introduction)
- [Task 1: Prepare Your EC2 Instances](#task-1-prepare-your-ec2-instances)
- [Task 2: Set Up Data Source in Grafana](#task-2-set-up-data-source-in-grafana)
- [Task 3: Create Dashboards for Monitoring](#task-3-create-dashboards-for-monitoring)
- [Task 4: Set Up Alerts (Optional)](#task-4-set-up-alerts-optional)
- [Verifying the Setup](#verifying-the-setup)
- [Common Issues & Troubleshooting](#common-issues--troubleshooting)
- [Key Takeaways](#key-takeaways)

---

## Introduction

With Grafana already running on EC2, today's exercise extends it into **cross-platform monitoring** — connecting it to both a **Linux** and a **Windows** EC2 instance, using **AWS CloudWatch** as the bridge between the servers and Grafana. This is a realistic scenario for many teams, since production environments are rarely single-OS, and a monitoring stack needs to give visibility across all of them from one place.

---

## Task 1: Prepare Your EC2 Instances

**Goal:** Have a running Linux and a running Windows EC2 instance, both reporting metrics and logs to CloudWatch via the CloudWatch agent.

**Steps:**

1. **Launch two EC2 instances** (if not already running):
   - One using an **Ubuntu/Amazon Linux** AMI.
   - One using a **Windows Server** AMI.
   - Both need an attached **IAM role** with the `CloudWatchAgentServerPolicy` permission so the agent can push metrics/logs to CloudWatch.

2. **Install the CloudWatch agent on the Linux instance:**

   ```bash
   # Download and install the agent
   wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
   sudo dpkg -i amazon-cloudwatch-agent.deb

   # Run the interactive config wizard to select which metrics/logs to collect
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

   # Start the agent using the generated config
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
     -a fetch-config -m ec2 \
     -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
   ```

   The config wizard lets you choose metrics beyond EC2's default set (which only covers things like basic CPU) — including **memory usage** and **disk usage**, which aren't collected by default and need the agent to be reported at all.

3. **Install the CloudWatch agent on the Windows instance:**
   - Connect via **RDP**.
   - Download the CloudWatch agent MSI installer from the official AWS S3 location (`https://s3.amazonaws.com/amazoncloudwatch-agent/windows/amd64/latest/amazon-cloudwatch-agent.msi`) and run it.
   - Use the same config wizard (`amazon-cloudwatch-agent-config-wizard.exe`, run from PowerShell) to select Windows-specific metrics — CPU, memory, disk, and Windows Performance Counters.
   - Start the agent as a Windows service so it runs continuously and survives reboots:

     ```powershell
     & "C:\Program Files\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent-ctl.ps1" -a fetch-config -m ec2 -c file:"C:\Program Files\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent.json" -s
     ```

4. **Confirm data is flowing** by checking the **CloudWatch console → Metrics → CWAgent namespace** — both instances should start appearing with their custom metrics within a few minutes.

---

## Task 2: Set Up Data Source in Grafana

**Goal:** Add AWS CloudWatch as a data source inside the existing Grafana instance so it can query the metrics both EC2 instances are now sending.

**Steps:**

1. In Grafana, go to **Connections → Data sources → Add data source → CloudWatch**.
2. Choose an authentication method:
   - **IAM role (recommended)** — if Grafana itself is running on an EC2 instance, attach an IAM role to *that* instance with CloudWatch read permissions (`CloudWatchReadOnlyAccess` or a scoped-down equivalent). This avoids storing long-lived credentials inside Grafana.
   - **Access keys** — alternatively, provide an IAM user's Access Key ID and Secret Access Key with CloudWatch read permissions, if Grafana isn't running with an attachable role (e.g. running elsewhere outside AWS).
3. Set the **default region** to match where the EC2 instances are running (e.g. `us-east-1`).
4. Click **Save & Test** to confirm Grafana can successfully query CloudWatch.

**Principle of least privilege:** whichever method is used, the credentials should only be granted the minimum permissions needed to read CloudWatch metrics/logs — not broader EC2 or account-wide access — since this is a monitoring integration, not an administrative one.

---

## Task 3: Create Dashboards for Monitoring

**Goal:** Build separate dashboards for the Linux and Windows instances, each surfacing relevant metrics, filtered so each dashboard only shows the metrics from its own instance.

**Steps:**

1. Create a new dashboard named **"Linux Instance Monitoring"**:
   - Add a panel for **CPU Utilization** — CloudWatch namespace `AWS/EC2`, metric `CPUUtilization`, dimension `InstanceId = <linux-instance-id>`.
   - Add a panel for **Memory Usage** — CloudWatch namespace `CWAgent`, metric `mem_used_percent` (only available because the CloudWatch agent was installed and configured in Task 1 — EC2's default metrics don't include memory).
   - Add a panel for **Disk I/O** — CloudWatch namespace `CWAgent`, metrics such as `disk_used_percent`, `diskio_read_bytes`, `diskio_write_bytes`.
   - Add a panel for **Network Stats** — CloudWatch namespace `AWS/EC2`, metrics `NetworkIn` and `NetworkOut`.

2. Create a second dashboard named **"Windows Instance Monitoring"**:
   - Same metric categories (CPU, memory, disk, network), but pointed at the Windows instance's `InstanceId` dimension, and pulling from the Windows-specific performance counters the CloudWatch agent was configured to collect (e.g. `Memory % Committed Bytes In Use`, `LogicalDisk % Free Space`).

3. **Differentiating the instances:** since both instances report into the same CloudWatch namespaces, each panel's query is filtered by the **`InstanceId` dimension** (and, where relevant, the **`ImageId`/OS tag**) so the Linux dashboard never accidentally shows Windows data and vice versa. Using a Grafana **template variable** for `InstanceId` (populated from CloudWatch) is a clean way to make each dashboard reusable if more instances of the same OS get added later.

4. Arrange panels sensibly on each dashboard — CPU and memory near the top as the most-glanced-at indicators, disk and network below — so each dashboard tells a quick "at a glance" story about that server's health.

---

## Task 4: Set Up Alerts (Optional)

**Goal:** Get notified automatically when a metric crosses a concerning threshold, rather than needing to watch the dashboards continuously.

**Steps:**

1. On the **CPU Utilization** panel (for either dashboard), open **Alerting → New alert rule**.
2. Define the condition:
   - **Query:** the same CloudWatch CPU utilization query used in the panel.
   - **Condition:** `IS ABOVE 80`
   - **Evaluation:** check every `1m`, and require the condition to hold for `5m` before firing — this matches the example in the exercise ("CPU usage exceeds 80% for more than 5 minutes") and avoids alerting on brief, harmless spikes.
3. Repeat similarly for other metrics worth alerting on (e.g. memory usage above a threshold, disk usage nearing capacity).
4. Configure a **contact point** under **Alerting → Contact points** — e.g. an email address or a Slack incoming webhook — so fired alerts have somewhere to go.
5. Attach a **notification policy** that routes alerts from these rules to the chosen contact point, optionally labeling rules (e.g. `os: linux` vs `os: windows`) so alerts are clearly identifiable by which server they came from.

This closes the loop: instead of relying on someone to notice a problem on a dashboard, both the Linux and Windows environments now proactively surface serious issues the moment they're detected.

---

## Verifying the Setup

- [ ] CloudWatch agent installed and running on both the Linux and Windows instances
- [ ] `CWAgent` namespace metrics (memory, disk) visible in the CloudWatch console for both instances
- [ ] Grafana's CloudWatch data source shows "working" after **Save & Test**
- [ ] Linux dashboard shows only the Linux instance's metrics; Windows dashboard shows only the Windows instance's metrics
- [ ] (If configured) test alert fires correctly when a threshold is deliberately breached, and the notification arrives at the chosen channel

---

## Common Issues & Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| No memory/disk metrics in CloudWatch | CloudWatch agent not installed, or config wizard didn't include those metrics | Re-run the config wizard and confirm `mem_used_percent`/disk metrics are enabled, then restart the agent |
| Grafana CloudWatch data source fails "Save & Test" | IAM role/credentials missing CloudWatch read permissions, or wrong region | Confirm the IAM policy includes CloudWatch read access and that the configured region matches the instances |
| Dashboard shows both instances' data mixed together | Missing or incorrect `InstanceId` dimension filter on panel queries | Add/verify the `InstanceId` dimension filter (or template variable) on every panel |
| Alerts don't fire even when threshold is exceeded | Evaluation/"for" duration too long, or contact point misconfigured | Check the alert rule's evaluation interval and confirm the contact point is correctly linked in the notification policy |

---

## Key Takeaways

- CloudWatch agent installation is what unlocks metrics beyond EC2's defaults — memory and disk usage specifically require the agent, on both Linux and Windows.
- Grafana's CloudWatch data source acts as the bridge, letting one Grafana instance visualize metrics from many EC2 instances, across operating systems, without needing OS-specific dashboards tools.
- Filtering by `InstanceId` (and using template variables) is what keeps multi-instance, multi-OS dashboards clean and correctly scoped rather than blending everything together.
- Alerting on sustained thresholds (e.g. CPU > 80% for 5 minutes) rather than instantaneous spikes keeps notifications meaningful instead of noisy.
- This exercise mirrors a realistic production scenario: most environments are multi-OS, and a monitoring setup needs to give consistent visibility across all of them from a single pane of glass.
