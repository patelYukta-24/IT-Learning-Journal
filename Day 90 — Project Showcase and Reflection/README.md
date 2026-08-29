# Day 90 — Project Showcase and Reflection

## Objective

Consolidate 90 days of learning and projects into a portfolio, and reflect on the journey — the challenges faced and how they were overcome.

---

## Tasks / Exercises

### 1. Portfolio Compilation

A portfolio is only as useful as it is easy to navigate, so this step consolidated documentation, code, and deployment evidence across the curriculum into a single, linkable structure rather than leaving it scattered across 90 individual day-folders.

**Repositories consolidated:**

| Repository | Contents |
|---|---|
| `IT-Learning-Journal` | Day-by-day self-study documentation, audited for consistent naming, working links, and no duplicate content |
| `SQL-Zero-to-Pro` | 21-module SQL curriculum, beginner to advanced |
| `MySQL-Zero-to-Pro` | Original MySQL content |
| `SQL-Learning-Portfolio` | 60-topic SQL portfolio, consistent 15-section format per topic |
| Security tooling repos (Nessus, Zscaler, SCCM, BitLocker, Microsoft Defender, Microsoft Intune) | Lab-practice-framed documentation, each with explicit self-study disclaimers |
| Portfolio website | Four-page bento-grid site (Home, Career, Projects, Contact) plus a separate CV/resume page |

**Project documentation compiled (Days 86–89, capstone stretch):**

- **Day 86 — Project 10:** Mounting an AWS S3 Bucket on EC2 with S3FS (IAM least-privilege access, FUSE mount, validated file operations)
- **Day 87 — Final Project:** Comprehensive CI/CD Pipeline with Advanced Security Scanning (Jenkins, SonarQube, OWASP Dependency-Check, Trivy, DockerHub, deployment of `node-todo-cicd`)
- **Day 88:** Advanced Monitoring/Logging (Prometheus, Grafana, CloudWatch) & Cost Optimization/Performance Tuning
- **Day 89:** Disaster Recovery and High Availability (Multi-AZ RDS, S3 versioning/CRR, ASG + ALB across Availability Zones)

**Deployment evidence linked per project:**

- Application URLs (where left running for demonstration) or, where instances were terminated for cost control post-validation, the corresponding screenshots/logs documented in each day's README as evidence of successful deployment.
- Pipeline run evidence (Jenkins Stage View, SonarQube Quality Gate result, Trivy scan report) linked from the Day 87 README.
- Dashboard evidence (Grafana panels, CloudWatch Logs Insights query results) linked from the Day 88 README.

**Portfolio honesty framing (carried consistently across the whole body of work):**

- Every project README is explicitly framed as **self-study / lab practice**, not professional work experience.
- The portfolio website and resume/CV use only verified, defensible metrics from actual professional experience (15,000+ users supported, 99%+ POS uptime, 95%+ SLA compliance) — kept clearly separate from the self-study project work.
- No certification is represented as complete unless it has actually been completed; in-progress items are labeled "in progress," not implied as finished.

### 2. Reflective Piece — Learning Journey, Challenges, and How They Were Overcome

---

**On the journey:**

This 90-day curriculum moved from Linux fundamentals and Git through networking, Python, containers, and CI/CD, into progressively more integrated capstone projects — culminating in a security-scanned CI/CD pipeline (Day 87), full observability stack (Day 88), and a disaster-recovery/high-availability design (Day 89). The arc mattered as much as any single day: early days built isolated skills (a Linux command, a Git workflow, a Terraform resource block), while the final stretch forced those skills to work together inside one coherent, production-shaped system.

**On challenges:**

A recurring theme across the later projects was that individual tools work fine in isolation but expose friction the moment they're wired together. A few concrete examples from this stretch:

- **Day 87:** Getting SonarQube's Quality Gate webhook to actually reach Jenkins required thinking about network reachability between two services, not just installing each one correctly — a reminder that integration work is its own skill, separate from knowing each tool.
- **Day 88:** Distinguishing an application-level problem from an infrastructure-level one required correlating two different signal sources (Node.js process metrics vs. OS-level CloudWatch metrics) rather than trusting either one alone.
- **Day 89:** Deciding *how much* disaster recovery is appropriate (Multi-AZ vs. full Multi-Region) was less a technical question than a judgment call about matching investment to actual risk and cost tolerance — a genuinely different kind of thinking than executing a checklist.

The common thread: the hardest parts of this final stretch weren't "how do I use tool X," but "how do these pieces depend on each other, and what's the honest tradeoff in choosing one design over another." That shift — from tool proficiency to systems thinking — is the clearest marker of progress across the 90 days.

**On what's next:**

This curriculum sits within a longer roadmap: IT Support → System Administrator → Infrastructure Engineer → Cloud Engineer → DevOps/Security Engineer, with a particular pull toward the cloud engineering track. The next concrete steps are finishing the AWS Cloud Practitioner certification currently in progress, and using the hands-on project base built here — S3, EC2, IAM, CI/CD, monitoring, DR/HA — as direct interview-preparation material, since every project in this portfolio maps to a question an interviewer could plausibly ask.

---

### 3. Sharing on Professional Networks

- **GitHub:** portfolio repositories kept current, with the profile README giving an honest, clearly-labeled overview of self-study work versus professional experience, and forked repositories properly attributed rather than presented as original work.
- **LinkedIn:** profile represents current IT Technical Support work accurately and solidly; certifications in progress are described as "currently pursuing," never implied as complete, consistent with the honesty standard applied throughout this whole body of work.
- **Portfolio website:** serves as the single link to share across both platforms — Home, Career, Projects, and Contact pages, plus the CV/resume page, all pointing back to the underlying GitHub repositories for anyone who wants to go deeper.

---

## Outcome

A consolidated, honestly-framed portfolio spanning the full 90-day curriculum, paired with a reflective account of the journey — the shift from individual tool proficiency to systems-level thinking, the specific integration challenges worked through in the final capstone stretch, and a clear next step (AWS certification + interview prep) tying this learning journey to the broader Cloud Engineering career roadmap.

---

## Note

*As stated in the curriculum: these final activities were designed to provide a rounded experience — not only technical skills, but practical judgment around cost management, disaster recovery, and real-world application performance. That rounding-out is reflected in this showcase: the portfolio isn't just a list of tools used, but a record of tradeoffs reasoned through.*

---
