# Day 89 — Disaster Recovery and High Availability

## Objective

Establish a basic disaster recovery (DR) plan and ensure high availability (HA) for the deployed Node.js application (`node-todo-cicd`, built and shipped via the Day 87 Jenkins pipeline).

---

## Tasks / Exercises

### 1. Data Backup and Recovery Strategy for the Node.js Application

The application's persistent state and artifacts come from a few distinct sources, each needing its own backup approach:

**a) Database backups (if using a datastore such as MongoDB/DynamoDB/RDS for the app's data):**

- For an **RDS**-backed setup: enabled **automated backups** with a defined retention window, and took a **manual DB snapshot** before any risky change (e.g., before a schema-impacting deploy):

```bash
aws rds modify-db-instance \
  --db-instance-identifier node-todo-db \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --apply-immediately

aws rds create-db-snapshot \
  --db-instance-identifier node-todo-db \
  --db-snapshot-identifier node-todo-db-manual-snapshot-$(date +%F)
```

- Enabled **Multi-AZ** on the RDS instance so a standby replica is kept in sync in a second Availability Zone, giving automatic failover if the primary AZ has an issue (covered further under HA below, since it serves both purposes).

**b) Application artifact / image backups:**

- Docker images produced by the CI/CD pipeline (Day 87) are already versioned in DockerHub with both a `latest` tag and a build-number tag (`${IMAGE_TAG}`), so any previous known-good image can be redeployed immediately — this doubles as a lightweight "backup" of the deployable artifact itself.
- Source code is backed by GitHub itself (the `node-todo-cicd` repo), which is the system of record for recovery of the codebase.

**c) S3 data backups (for any S3-backed storage used, e.g. Day 86's mounted bucket or Day 79/83 static assets):**

- Enabled **S3 Versioning** on buckets holding meaningful data, so accidental overwrites/deletes can be rolled back to a prior object version.
- Enabled **Cross-Region Replication (CRR)** for buckets holding critical data, replicating objects to a bucket in a second AWS region as a geographic backup.

**d) EC2 instance-level backups:**

- Where the app or supporting tools (Jenkins) run on EC2 rather than managed services, used **AWS Backup** to schedule automated **EBS snapshots** of the instance's root volume on a daily cadence with a retention policy, rather than relying on manual snapshotting.

**Recovery procedure (documented and tested):**

1. Identify the last known-good DB snapshot / RDS restore point.
2. Restore RDS from the snapshot to a new instance: `aws rds restore-db-instance-from-db-snapshot`.
3. Update the application's DB connection configuration (or, better, a Route 53 CNAME/endpoint) to point at the restored instance.
4. Redeploy the last known-good Docker image tag via the Jenkins pipeline (or manually via `docker run` against the restored DB).
5. Validate application functionality against the restored data before cutting traffic over fully.

### 2. High Availability Configuration

Implemented HA at two layers — compute and data — rather than relying on a single EC2 instance:

**Compute layer — EC2 Auto Scaling across multiple AZs:**

- Reused the **Launch Template** approach from Day 48/65, this time deliberately spreading the Auto Scaling Group across **at least two Availability Zones** within the region.

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name node-todo-asg \
  --launch-template LaunchTemplateName=node-todo-lt,Version='$Latest' \
  --min-size 2 \
  --max-size 4 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-aaaa1111,subnet-bbbb2222" \
  --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/node-todo-tg/xxxx \
  --health-check-type ELB \
  --health-check-grace-period 60
```

- Fronted the ASG with an **Application Load Balancer** (same pattern as Day 49), so traffic is automatically routed only to healthy instances across both AZs — if an entire AZ becomes unavailable, the ALB and ASG continue serving traffic from the surviving AZ, and the ASG launches replacement capacity there.
- Configured a **scaling policy** (target tracking on average CPU utilization) so capacity grows under load and shrinks back down when load drops, keeping both availability and cost in balance (ties back to Day 88's cost-optimization work).

**Data layer — Multi-AZ RDS (or equivalent):**

- As noted above, enabled **Multi-AZ** for the RDS instance backing the application, so a synchronously-replicated standby in a second AZ is ready to take over automatically (via a DNS endpoint flip, handled by RDS) if the primary fails — no manual intervention needed for a single-AZ outage.

**Why this combination was chosen (rationale):**

- A **single EC2 instance** is a single point of failure regardless of how well it's backed up — HA needs redundant *running* capacity, not just backups, hence the ASG + ALB layer.
- **Multi-AZ over Multi-Region** was chosen as the baseline HA posture because it protects against the most common failure mode (an AZ-level outage) at a much lower cost and operational complexity than full multi-region active-active, which was judged out of scope for this application's actual availability requirements. Multi-region was instead handled more selectively via S3 CRR for critical object data, rather than replicating the full compute/DB stack.
- **Target-tracking auto scaling** (vs. a fixed fleet size) means the HA setup doesn't just survive a failure — it also absorbs normal traffic spikes without manual resizing.

### 3. Documented Steps and Rationale

All setup steps, commands, and design rationale are captured inline above under each task. Summary of the overall DR/HA design:

| Layer | Failure Scenario Covered | Mechanism | Recovery Approach |
|---|---|---|---|
| Compute (EC2) | Single instance failure | ASG health checks + auto-replace | Automatic — ASG launches a replacement instance |
| Compute (AZ) | Entire AZ outage | ASG spread across 2+ AZs behind ALB | Automatic — ALB routes to healthy AZ, ASG rebalances |
| Database | Primary DB instance failure | RDS Multi-AZ synchronous standby | Automatic — RDS DNS failover to standby |
| Database | Data corruption / accidental deletion | Automated + manual RDS snapshots | Manual — restore snapshot to new instance, repoint app |
| Application artifact | Bad deployment | Versioned DockerHub image tags | Manual/pipeline — redeploy previous known-good tag |
| Object storage | Accidental overwrite/delete | S3 Versioning | Manual — restore prior object version |
| Object storage | Regional outage | S3 Cross-Region Replication | Manual — serve/read from replica region bucket |
| Source code | Repository-level loss | GitHub as system of record | N/A — GitHub's own durability guarantees |

---

## Outcome

This exercise built a working understanding of the fundamentals of disaster recovery and high availability in a cloud environment: the distinction between **backups** (protecting against data loss, recovered manually/deliberately) and **high availability** (protecting against downtime, handled automatically by redundant infrastructure); why HA requires redundant *running* capacity across failure domains (AZs) rather than just data backups; and how to reason about the right level of DR investment (Multi-AZ vs. full Multi-Region) based on the actual criticality and cost tolerance of the application, rather than defaulting to the most complex option available.

---

## Resources

- [AWS Well-Architected Framework — Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
- [Amazon RDS Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [AWS Auto Scaling documentation](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)
- [AWS Backup documentation](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
- [Amazon S3 Cross-Region Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)

---
