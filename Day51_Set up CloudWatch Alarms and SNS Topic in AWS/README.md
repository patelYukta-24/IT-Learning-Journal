# Day 51: Set up CloudWatch Alarms and SNS Topic in AWS

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. What is Amazon CloudWatch?](#1-what-is-amazon-cloudwatch)
- [2. What is Amazon SNS?](#2-what-is-amazon-sns)
- [3. How CloudWatch + SNS Work Together for Billing Alerts](#3-how-cloudwatch--sns-work-together-for-billing-alerts)
- [4. Tasks](#4-tasks)
- [5. Troubleshooting Notes](#5-troubleshooting-notes)
- [6. Key Learnings](#6-key-learnings)
- [7. Resources](#7-resources)

---

## Overview
Day 51 is a genuinely practical one: setting up a safety net so a Free Tier account doesn't silently rack up charges. The task combines two services — **CloudWatch** (monitoring/alarms) and **SNS** (notifications) — to get an email the moment estimated AWS charges cross a defined threshold ($2 in this case), then walks through tearing that alarm back down again.

## Topics Covered
- ❖ Amazon CloudWatch — metrics, alarms
- ❖ Amazon SNS — topics, subscriptions, notification delivery
- ❖ Task — billing alarm at $2 threshold with email notification, then deletion

---

## 1. What is Amazon CloudWatch?
Amazon CloudWatch is AWS's monitoring and observability service. It collects **metrics** (measurable data points like CPU utilization, request counts, or — relevant here — estimated billing charges) from AWS resources and applications in near real-time, and lets you set **alarms** that trigger an action when a metric crosses a defined threshold.

### 1.1 Core CloudWatch Concepts
| Concept | Description |
|---|---|
| **Metric** | A time-ordered set of data points for something measurable (e.g., `EstimatedCharges`, `CPUUtilization`) |
| **Namespace** | A container that groups related metrics (e.g., `AWS/Billing`, `AWS/EC2`) |
| **Alarm** | A watcher on a metric that changes state (OK → ALARM) when a threshold condition is met over a defined period |
| **Alarm Action** | What happens when the alarm state changes — commonly, publishing a notification to an SNS topic |

---

## 2. What is Amazon SNS?
Amazon Simple Notification Service (SNS) is a fully managed pub/sub messaging service, available since 2010, built for low-cost, high-throughput delivery of messages — commonly to email, SMS, mobile push, or other AWS services (like Lambda or SQS).

### 2.1 Core SNS Concepts
| Concept | Description |
|---|---|
| **Topic** | A named communication channel that messages are published to |
| **Subscription** | An endpoint (email address, phone number, Lambda function, etc.) that receives messages published to a topic |
| **Publisher** | Whatever sends a message to the topic — here, a CloudWatch alarm |
| **Subscriber** | Whatever receives it — here, your email address |

---

## 3. How CloudWatch + SNS Work Together for Billing Alerts
CloudWatch on its own can *detect* that a threshold was crossed, but it has no built-in way to email you directly — that's what SNS is for. The flow is:

```
AWS Billing Metric (EstimatedCharges)
          │
          ▼
   CloudWatch Alarm  ──(threshold breached)──►  SNS Topic  ──►  Email Subscription  ──►  Your Inbox
```

CloudWatch watches the metric and changes state when the threshold is crossed → it publishes to the SNS Topic as its configured alarm action → SNS delivers the message to every subscriber on that topic (your email, in this case).

---

## 4. Tasks

### Task — Billing Alarm at $2 Threshold with Email Notification, Then Deletion

**Prerequisite: Enable Billing Alerts (one-time account setting)**
1. Sign in as the **root user** (billing alerts must be enabled at the account/root level)
2. Billing and Cost Management → Billing preferences
3. Check **"Receive Billing Alerts"** → Save preferences
> Note: `EstimatedCharges` billing metrics are only published to CloudWatch in the **us-east-1 (N. Virginia)** region, regardless of which region your resources run in — the alarm must be created there.

---

**Step 1: Create the SNS Topic + Email Subscription**
1. SNS → Topics → Create topic
2. Type: **Standard**
3. Name: `billing-alerts-topic`
4. Create topic
5. Inside the topic → **Create subscription**
   - Protocol: **Email**
   - Endpoint: your email address
6. Create subscription → **check your inbox** and click the confirmation link AWS sends (subscription stays "Pending confirmation" until you do)

**Equivalent via AWS CLI:**
```bash
# Create topic (must be in us-east-1 for billing alarms)
aws sns create-topic --name billing-alerts-topic --region us-east-1

# Subscribe your email
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:<account-id>:billing-alerts-topic \
  --protocol email \
  --notification-endpoint you@example.com \
  --region us-east-1
```
> You still need to manually click the confirmation link emailed by AWS — this step can't be automated via CLI.

---

**Step 2: Create the CloudWatch Billing Alarm**
1. CloudWatch (region: **us-east-1**) → Alarms → Create alarm
2. Select metric → **Billing → Total Estimated Charge** → select `EstimatedCharges` (currency: USD)
3. Conditions: **Static** → Greater than → threshold value: `2` (USD)
4. Notification: select the existing SNS topic `billing-alerts-topic` (or create a new one from this screen)
5. Name the alarm: `billing-alarm-2usd`
6. Create alarm

**Equivalent via AWS CLI:**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "billing-alarm-2usd" \
  --alarm-description "Alarm when estimated AWS charges exceed $2" \
  --namespace "AWS/Billing" \
  --metric-name "EstimatedCharges" \
  --dimensions Name=Currency,Value=USD \
  --statistic Maximum \
  --period 21600 \
  --evaluation-periods 1 \
  --threshold 2 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:us-east-1:<account-id>:billing-alerts-topic \
  --region us-east-1
```

**Verification:**
```bash
aws cloudwatch describe-alarms --alarm-names "billing-alarm-2usd" --region us-east-1
```
The alarm's `StateValue` will show `OK` (below threshold), `ALARM` (threshold breached — you should have received an email), or `INSUFFICIENT_DATA` (not enough billing data yet, common right after account creation).

> Status: reference implementation — pending execution + screenshot of the created alarm and the confirmation/test email received.

---

**Step 3: Delete the Billing Alarm**
1. CloudWatch → Alarms → select `billing-alarm-2usd` → Actions → **Delete**
2. Confirm deletion

**Equivalent via AWS CLI:**
```bash
aws cloudwatch delete-alarms --alarm-names "billing-alarm-2usd" --region us-east-1
```

**Verification:**
```bash
aws cloudwatch describe-alarms --alarm-names "billing-alarm-2usd" --region us-east-1
```
Should return an empty `MetricAlarms` list, confirming deletion.

> Note: the task only asks to delete the *alarm*, not the SNS topic/subscription — leaving the topic in place means it's ready to reuse for a future alarm without re-confirming the email subscription.

> Status: reference implementation — pending execution + screenshot confirming the alarm no longer appears in the console.

---

## 5. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| Can't find "Billing" as a metric namespace in CloudWatch | Wrong region selected, or billing alerts not enabled | Billing metrics only exist in `us-east-1`; confirm "Receive Billing Alerts" is checked in Billing preferences |
| No email received after creating the alarm | SNS email subscription still "Pending confirmation" | Check inbox (and spam folder) for the AWS confirmation email and click the link |
| Alarm stuck in `INSUFFICIENT_DATA` | Billing metrics update roughly every few hours, and only after some usage exists | Wait — this is expected shortly after account creation or if usage is minimal |
| Alarm triggers immediately even at $0 usage | Comparison operator or threshold value set incorrectly | Confirm "Greater than" `2`, not "Greater than or equal to" `0` or similar |
| `put-metric-alarm` CLI command fails | Run outside `us-east-1`, or SNS topic ARN incorrect | Add `--region us-east-1` explicitly and double-check the topic ARN |

---

## 6. Key Learnings
- CloudWatch alarms alone don't send notifications — SNS is the actual delivery mechanism, with CloudWatch only triggering the publish action.
- Billing metrics are a special case: they only exist in `us-east-1`, regardless of which region your workloads run in — an easy thing to trip over.
- SNS email subscriptions require manual confirmation via a clicked link — this can't be skipped or automated, which is a good manual verification step in practice.
- Knowing how to tear down monitoring resources (not just create them) matters just as much — an alarm left dangling after its purpose is served is still a resource to keep track of.
- This is a genuinely worthwhile habit for a personal Free Tier account, not just a curriculum exercise — a $2 threshold means you'd hear about unexpected charges almost immediately.

## 7. Resources
- [Amazon CloudWatch — User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [Amazon SNS — Developer Guide](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [Creating a Billing Alarm — AWS Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor-estimated-charges-with-cloudwatch.html)

---
