# Day 88 — Advanced Monitoring and Logging & Cost Optimization and Performance Tuning

This day covers two related topics that round out a production-ready deployment: **observability** (monitoring + centralized logging) and **cost/performance efficiency**. Both build directly on the Node.js application and AWS infrastructure used in earlier projects (the `node-todo-cicd` app deployed via the Day 87 Jenkins pipeline).

---

## Part 1: Advanced Monitoring and Logging

### Objective

Implement advanced monitoring and logging for the deployed Node.js application.

### Tasks / Exercises

#### 1. Integrate a Monitoring Tool (Prometheus + Grafana) with the Node.js Application

**Instrument the app to expose metrics:**

```bash
npm install prom-client --save
```

```javascript
// metrics.js
const client = require('prom-client');
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metric: HTTP request duration
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
});
register.registerMetric(httpRequestDuration);

module.exports = { register, httpRequestDuration };
```

```javascript
// app.js (excerpt)
const { register, httpRequestDuration } = require('./metrics');

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    end({ method: req.method, route: req.path, status_code: res.statusCode });
  });
  next();
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

**Deploy Prometheus (Docker) on the EC2 instance:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-todo-app'
    static_configs:
      - targets: ['<EC2_PRIVATE_IP>:3000']
```

```bash
docker run -d --name prometheus -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

**Deploy Grafana (Docker) and connect it to Prometheus:**

```bash
docker run -d --name grafana -p 3001:3000 grafana/grafana
```

- Logged into Grafana (`http://<EC2_PUBLIC_IP>:3001`), added Prometheus (`http://<EC2_PUBLIC_IP>:9090`) as a data source.
- Built a dashboard with panels for: request rate (`rate(http_request_duration_seconds_count[5m])`), request latency (p95 via `histogram_quantile`), error rate (status codes ≥ 500), and default Node.js process metrics (event loop lag, memory, CPU) from `client.collectDefaultMetrics()`.

#### 2. Set Up Centralized Logging (AWS CloudWatch)

Since the application runs in the AWS ecosystem, used **CloudWatch Logs** as the centralized logging solution rather than standing up a full self-managed ELK stack.

**Install and configure the CloudWatch agent on the EC2 instance:**

```bash
sudo yum install -y amazon-cloudwatch-agent
```

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/lib/docker/containers/*/*.log",
            "log_group_name": "node-todo-app-logs",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%Y-%m-%dT%H:%M:%S"
          },
          {
            "file_path": "/var/log/jenkins/jenkins.log",
            "log_group_name": "jenkins-logs",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  },
  "metrics": {
    "metrics_collected": {
      "cpu": { "measurement": ["cpu_usage_idle", "cpu_usage_user", "cpu_usage_system"] },
      "mem": { "measurement": ["mem_used_percent"] },
      "disk": { "measurement": ["used_percent"], "resources": ["/"] }
    }
  }
}
```

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -s \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/config.json
```

- Verified log streams appearing in **CloudWatch → Log groups → `node-todo-app-logs`**.
- Created a **CloudWatch metric filter** on the log group to count `ERROR` occurrences, and an accompanying **CloudWatch Alarm** to trigger on a threshold of errors within a 5-minute window, notifying via an SNS topic (same pattern used for the Day 51 billing alarm).
- Used **CloudWatch Logs Insights** to run structured queries against the log group, e.g.:

```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

#### 3. Documenting Log/Metric Analysis for Troubleshooting and Performance Tuning

- **For request errors:** start in the Grafana error-rate panel to identify *when* an error spike occurred, then pivot to CloudWatch Logs Insights filtered to that time window to see the actual stack traces/messages.
- **For latency issues:** used the Prometheus p95 latency panel to distinguish a general slowdown (infra/resource issue) from a route-specific slowdown (application logic issue), since the histogram is labeled by route.
- **For resource exhaustion:** correlated the CloudWatch `mem_used_percent`/`cpu_usage` metrics against the Node.js process-level metrics from `prom-client` (event loop lag, heap usage) to tell apart an OS-level resource problem from an application-level memory leak.
- **General workflow:** Grafana dashboards for "is something wrong and roughly when," CloudWatch Logs Insights for "what exactly happened," and the Prometheus histogram breakdown for "which endpoint/operation is responsible."

### Outcome

Gained hands-on experience instrumenting a Node.js application for metrics (Prometheus/Grafana), centralizing its logs into CloudWatch, and building a repeatable workflow for correlating dashboards, alarms, and log queries during troubleshooting — directly applicable to maintaining production environments.

---

## Part 2: Cost Optimization and Performance Tuning

### Objective

Learn about cloud cost optimization and performance tuning.

### Tasks / Exercises

#### 1. Cost Optimization Strategies for AWS Services Used (EC2, S3, ECS)

**EC2:**
- Right-sized instances after observing actual CPU/memory utilization in CloudWatch (e.g., the `t2.large` used for Jenkins+SonarQube in Day 87 was oversized once Jenkins was idle outside of builds — noted as a candidate for scheduled start/stop rather than running 24/7).
- Evaluated **Reserved Instances / Savings Plans** for any long-running, predictable workloads (e.g., a persistent Jenkins controller) versus **On-Demand** for short-lived lab/test instances.
- Used **EC2 Auto Scaling** (from Day 48/65) so capacity — and cost — tracks actual load instead of running peak-sized capacity constantly.
- Stopped/terminated lab instances immediately after each project's validation was complete rather than leaving them running (applied throughout this curriculum's cleanup steps).

**S3:**
- Reviewed **S3 Storage Classes** and identified which project buckets (e.g., the Day 86 S3FS bucket, Day 79/83 static website buckets) were good candidates for **S3 Intelligent-Tiering** or **Lifecycle Policies** transitioning infrequently accessed objects to S3-IA/Glacier.
- Enabled **S3 Lifecycle rules** to auto-expire incomplete multipart uploads and old object versions where versioning was enabled (Day 64).
- Confirmed public-read buckets used for static hosting had no unnecessary versioning enabled, since duplicate object versions increase storage costs.

**ECS:**
- For the Day 82 ECS Fargate deployment, compared **Fargate on-demand pricing** against **Fargate Spot** for interruption-tolerant workloads, since Spot offers a significant discount for non-critical or dev/test tasks.
- Right-sized the Fargate **task CPU/memory** allocation based on observed CloudWatch Container Insights utilization instead of using default oversized task definitions.

**General/cross-cutting:**
- Set up a **CloudWatch billing alarm** (reusing the Day 51 pattern) to catch unexpected cost spikes early.
- Used **AWS Cost Explorer** to review spend by service across the projects completed in this curriculum, identifying EC2 as the dominant cost driver (as expected, given multiple lab instances across the Jenkins, Kubernetes, and Terraform projects).
- Tagged resources by project (`Project: Day87-CICD`, etc.) to make cost attribution per project visible in Cost Explorer.

#### 2. Basic Performance Tuning for the Node.js Application

- **Enabled gzip compression** via the `compression` middleware to reduce response payload size:

```javascript
const compression = require('compression');
app.use(compression());
```

- **Used the Node.js cluster module / PM2** to run one worker process per CPU core, rather than a single-threaded process handling all traffic:

```bash
npm install pm2 -g
pm2 start app.js -i max   # spawns one instance per available CPU core
```

- **Reviewed and reduced synchronous/blocking calls** in request handlers (identified via the event-loop-lag metric surfaced through `prom-client`'s default metrics) that were blocking the event loop under load.
- **Added basic response caching** for read-heavy endpoints where data doesn't change per-request, reducing redundant work per request.
- **Load-tested** the application (`autocannon` / `ab`) before and after tuning to confirm measurable improvement in requests/sec and p95 latency, using the same Grafana latency panel from Part 1 to visualize the before/after.

#### 3. Tools and Practices for Efficient Resource Utilization

- **AWS Compute Optimizer** — reviewed its EC2 rightsizing recommendations against the instances used across this curriculum's projects.
- **CloudWatch Container Insights** — used for the ECS Fargate task to see actual vs. allocated CPU/memory, informing task-definition rightsizing.
- **`docker stats`** — used locally/on EC2 to check container-level resource usage for the Docker Swarm (Day 80) and standalone container deployments.
- **Trusted Advisor** (where available on the account) — reviewed cost-optimization checks (idle load balancers, low-utilization EC2 instances, unattached EBS volumes) as a final sweep across all projects' leftover resources.

### Outcome

Developed a practical understanding of how to balance cloud cost against application performance — rightsizing compute, choosing appropriate storage tiers and pricing models, tuning the Node.js application itself for better throughput, and using AWS's native tooling (Cost Explorer, Compute Optimizer, Container Insights, billing alarms) to keep both cost and performance visible on an ongoing basis rather than as a one-time exercise.

---

## Key Takeaways

- Observability (Part 1) and cost/performance (Part 2) are two sides of the same coin: you can't optimize cost or performance without first having metrics and logs that show where the money and the latency are actually going.
- CloudWatch is sufficient for centralized logging within a fully AWS-native stack; a self-managed ELK stack becomes more justified when logs need to be correlated across non-AWS or multi-cloud sources, or when more advanced full-text search/visualization (Kibana) is needed beyond what CloudWatch Logs Insights offers.
- Cost optimization is not a one-time task — billing alarms, resource tagging, and periodic Cost Explorer / Compute Optimizer review are what make it sustainable across an evolving set of projects.

---

## Resources

- [Prometheus documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana documentation](https://grafana.com/docs/)
- [AWS CloudWatch Logs documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
- [AWS Cost Optimization best practices](https://aws.amazon.com/aws-cost-management/aws-cost-optimization/)
- [AWS Compute Optimizer](https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html)

---
