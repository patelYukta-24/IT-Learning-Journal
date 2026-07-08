# Day 32: CI/CD — The Heartbeat of DevOps & Introduction to Jenkins

> **DevOps Transformation** | CI/CD & Automation Series

---

## 📋 Overview

Everything you have built so far — Linux skills, Git workflows, Python scripts, Docker containers — exists to serve one purpose: **shipping working software reliably and repeatedly**. CI/CD is the system that makes that happen automatically.

Continuous Integration and Continuous Delivery/Deployment are not tools — they are **practices** and **philosophies** that fundamentally change how software is built and released. Jenkins is one of the most widely used tools that implements these practices. Today you understand both.

By the end of this session you will have a clear mental model of what CI/CD is and why it exists, understand the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment, and have Jenkins installed and your first job running.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| Understanding CI/CD | The problem CI/CD solves and the philosophy behind it |
| Key Concepts | CI vs CD vs Continuous Deployment — precise definitions |
| Benefits of CI/CD | Why every modern team adopts it |
| CI/CD in the Real World | End-to-end pipeline walkthrough |
| Introduction to Jenkins | What Jenkins is and where it fits |
| Installing Jenkins | Setup on Linux, Docker, and locally |
| Your First Jenkins Job | Create and run a Freestyle project |
| Jenkins in CI/CD | Pipelines, SCM integration, distributed builds |

---

## 🏢 NexusCorp Scenario

> NexusCorp's engineering team deploys their web application manually. Every Friday, a developer SSHes into the production server, pulls the latest code, runs migrations, restarts services, and hopes nothing breaks. When it does break — and it does — the whole team scrambles on a Friday evening.
>
> After one too many painful Fridays, the team decides to implement CI/CD. Within two weeks, deployments happen automatically, every commit is tested, and the last Friday deployment incident is a distant memory.
>
> Today you learn the system they built.

---

## Part 1: Understanding CI/CD — The Foundation of Modern DevOps

### The Problem CI/CD Solves

Before CI/CD became standard practice, software teams operated with a pattern called **"big bang integration"**:

```
Traditional (Big Bang) Workflow:
─────────────────────────────────────────────────────

Developer A works in isolation for 2 weeks
Developer B works in isolation for 2 weeks
Developer C works in isolation for 2 weeks

Then: everyone merges at once
      → massive conflicts
      → tests fail everywhere
      → nobody knows what broke what
      → "integration hell" for days or weeks

Result: delayed releases, stressed teams, broken production
```

CI/CD replaces big bang integration with small, frequent, automated steps:

```
CI/CD Workflow:
─────────────────────────────────────────────────────

Developer commits small change
      ↓ (immediately, automatically)
Code is pulled by CI server
      ↓
Tests run automatically (unit, integration, lint)
      ↓
If tests pass → build artifact (Docker image, binary, etc.)
      ↓
Deploy to staging automatically
      ↓
Further tests (UAT, performance, security)
      ↓
If all clear → deploy to production (CD)
      ↓
Monitor production

Result: problems caught immediately, releases are boring and routine
```

---

### The Three Practices — Precisely Defined

#### Continuous Integration (CI)

**Definition:** The practice of automatically integrating, building, and testing code changes from all developers into a shared repository multiple times per day.

```
Developer commits → Trigger → Build → Test → Report
(every commit)               (compile  (unit,   (pass/fail
                             if needed) integration notification)
                                        lint)
```

**What CI catches:**
- Merge conflicts that would have been hidden for weeks
- Code that compiles but fails tests
- Regressions — new code that breaks existing functionality
- Style and lint violations
- Missing dependencies

**CI requirement:** Developers must commit to the main branch (or short-lived feature branches) frequently — at least once per day. Long-lived feature branches defeat the purpose of CI.

---

#### Continuous Delivery (CD — "Delivery")

**Definition:** An extension of CI where the codebase is **always in a deployable state**. Releases to production are a manual decision but can happen at any time with a single click.

```
CI passes → Deploy to Staging → Automated Tests → ✅ READY TO RELEASE
                                                        ↓
                                                  Human decision:
                                                  "Release now?"
                                                        ↓
                                                  Production Deploy
```

**Key distinction:** Continuous Delivery means you *can* deploy at any time. Whether you *do* is a business decision.

**NexusCorp example:** The ops team decides to release every Tuesday at 2 PM. CI runs continuously, staging always has the latest passing build, and the Tuesday release is a single button click — not a stressful multi-hour event.

---

#### Continuous Deployment (CD — "Deployment")

**Definition:** Every change that passes all automated stages is **automatically released to production** without any human intervention.

```
CI passes → Deploy to Staging → All Tests Pass → Deploy to Production
                                                  (automatic, no human click)
```

**Key distinction from Delivery:** In Continuous Deployment, **humans are out of the release loop entirely**. If tests pass, production gets updated automatically.

**Requirements for Continuous Deployment:**
- Comprehensive automated test coverage (you need confidence that passing tests means it's safe)
- Feature flags (to release code without activating features)
- Robust monitoring and automatic rollback capability
- A culture that trusts the pipeline

**Example teams:** Amazon reportedly deploys to production every 11.6 seconds on average. That is only possible with Continuous Deployment.

---

### The Three Practices — Side by Side

```
┌─────────────────────────────────────────────────────────────┐
│              Code  →  Build  →  Test  →  Stage  →  Prod    │
├─────────────────────────────────────────────────────────────┤
│  CI          ✅          ✅        ✅                        │
│  (Integration automated; deployment manual/not included)    │
├─────────────────────────────────────────────────────────────┤
│  Cont. Delivery  ✅    ✅        ✅       ✅      👤 Manual  │
│  (Everything automated; production deploy = human decision) │
├─────────────────────────────────────────────────────────────┤
│  Cont. Deployment ✅   ✅        ✅       ✅       ✅ Auto   │
│  (Everything automated including production deploy)         │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 2: Key Concepts in CI/CD

### Core Components of a CI/CD System

| Component | Role | Examples |
|---|---|---|
| **Source Control** | Stores code and triggers pipelines | Git, GitHub, GitLab, Bitbucket |
| **CI/CD Server** | Orchestrates the pipeline | Jenkins, GitHub Actions, GitLab CI |
| **Build System** | Compiles and packages code | Maven, Gradle, npm, Docker |
| **Test Framework** | Runs automated tests | pytest, JUnit, Jest, Selenium |
| **Artifact Registry** | Stores build outputs | Docker Hub, ECR, Nexus, JFrog |
| **Deployment Tool** | Pushes to environments | Ansible, Kubernetes, AWS CodeDeploy |
| **Monitoring** | Observes production | Prometheus, Grafana, CloudWatch |

---

### Types of Tests in a CI Pipeline

```
Test Pyramid:
                 ┌────┐
                 │ E2E │  ← Slowest, most expensive, least numerous
                ┌┤────├┐
                ││Integ││ ← Medium speed and cost
               ┌┤│────│├┐
               │││Unit │││ ← Fastest, cheapest, most numerous
               └┴┴────┴┴┘

Unit tests:        Test individual functions/classes in isolation
Integration tests: Test interactions between components
End-to-end (E2E):  Test the entire user flow in a real browser/environment
Performance tests: Measure speed and stability under load
Security tests:    Scan for vulnerabilities (SAST, DAST)
Smoke tests:       Basic "is it alive?" checks after deployment
```

---

### The Deployment Environments

```
Developer's Machine  →  CI Build  →  Dev  →  Staging  →  Production
                                    (auto)    (auto)      (auto or manual)
```

| Environment | Purpose | Who Deploys |
|---|---|---|
| **Local** | Developer's machine | Developer manually |
| **Development** | Integration testing by devs | CI automatically |
| **Staging** | Pre-production replica | CI/CD automatically |
| **Production** | Live, user-facing | CD automatically or manual |

---

## Part 3: Benefits of CI/CD

```
┌──────────────────────────────────────────────────────────────┐
│                   Benefits of CI/CD                          │
├──────────────────────────────────────────────────────────────┤
│ 🚀 Faster Releases                                           │
│    Deploy in minutes instead of weeks                        │
│    Multiple deployments per day instead of per quarter       │
│                                                              │
│ 🐛 Early Bug Detection                                       │
│    Catch bugs minutes after they're introduced               │
│    Fix is cheap when context is fresh                        │
│                                                              │
│ 😌 Reduced Risk                                              │
│    Small changes are easier to debug than large releases     │
│    Rollback is easy when each change is isolated             │
│                                                              │
│ 🤝 Better Collaboration                                      │
│    Everyone works on the same codebase, not diverging branches│
│    Integration problems surface immediately                  │
│                                                              │
│ 📊 Increased Visibility                                      │
│    Pipeline status visible to whole team                     │
│    Build and test history creates an audit trail             │
│                                                              │
│ 🔒 Consistent Quality                                        │
│    Same tests run the same way every time                    │
│    No "it passed on my machine" scenarios                    │
│                                                              │
│ 🧠 Developer Confidence                                      │
│    Developers know immediately if their change broke anything│
│    Fear of deployment disappears                             │
└──────────────────────────────────────────────────────────────┘
```

---

## Part 4: CI/CD in the Real World

### End-to-End Pipeline Walkthrough

Using NexusCorp's web application as the example:

```
Step 1: Developer Commits
─────────────────────────
Developer writes a feature, writes tests, commits to a feature branch
git push origin feature/payment-gateway
→ Pull request opened on GitHub

Step 2: CI Server Picks Up the Change
─────────────────────────────────────
Jenkins (or GitHub Actions) webhook fires
→ Jenkins clones the repository
→ Jenkins starts the pipeline defined in Jenkinsfile

Step 3: Automated Testing
─────────────────────────
Jenkins runs:
  ✓ Code linting (flake8, eslint)
  ✓ Unit tests (pytest) — 847 tests, 12 seconds
  ✓ Integration tests — test API endpoints with test database
  ✓ Security scan (Bandit, Trivy)
  → If any fail: developer gets notified immediately, PR blocked

Step 4: Build Artifact
──────────────────────
All tests pass:
  → docker build -t nexuscorp/api:${GIT_SHA} .
  → docker push nexuscorp/api:${GIT_SHA}
  → Image stored in ECR for later deployment

Step 5: Deploy to Staging
─────────────────────────
  → kubectl set image deployment/nexus-api api=nexuscorp/api:${GIT_SHA}
  → Health check waits until new pods are healthy
  → Staging URL: staging.nexuscorp.com

Step 6: Automated Acceptance Tests on Staging
──────────────────────────────────────────────
  → Selenium E2E tests run against staging URL
  → Performance test: verify response time < 200ms
  → Security headers check
  → Smoke tests on critical user journeys

Step 7: Production Deploy (Continuous Delivery)
────────────────────────────────────────────────
  → Tech lead approves PR and merges to main
  → PR merge triggers production deploy
  → Zero-downtime rolling update in Kubernetes
  → Monitor: error rate, latency, CPU/memory

Step 8: Post-Deploy Monitoring
───────────────────────────────
  → Prometheus alerts if error rate > 1%
  → PagerDuty notifies on-call if SLO breached
  → Automatic rollback if health check fails
```

---

## Part 5: CI/CD Tool Landscape

### Major CI/CD Tools

| Tool | Type | Hosted/Self-hosted | Notes |
|---|---|---|---|
| **Jenkins** | CI/CD Server | Self-hosted | Open source, highly flexible, large plugin ecosystem |
| **GitHub Actions** | CI/CD Platform | Cloud (GitHub) | Native GitHub integration, YAML-based |
| **GitLab CI** | CI/CD Platform | Cloud + Self-hosted | Built into GitLab, excellent for GitLab users |
| **CircleCI** | CI/CD Platform | Cloud + Self-hosted | Fast, Docker-native, good DX |
| **Travis CI** | CI/CD Platform | Cloud | Popular for open source; declining adoption |
| **AWS CodePipeline** | CI/CD Platform | Cloud (AWS) | Native AWS integration |
| **Azure DevOps** | CI/CD Platform | Cloud (Azure) | Full DevOps suite from Microsoft |
| **Tekton** | CI/CD Framework | Kubernetes-native | Cloud-native, container-based |
| **ArgoCD** | CD Platform | Kubernetes-native | GitOps — Git as source of truth for k8s |
| **Spinnaker** | CD Platform | Self-hosted | Multi-cloud deployment, Netflix-born |
| **Drone CI** | CI/CD Server | Self-hosted | Lightweight, container-native |
| **Buildkite** | CI/CD Platform | Cloud + agents | Hybrid — runs agents in your infra |

---

### Task 1: Identify CI/CD Stages in Your Workflow

Reflect on a project you have worked on or are working on now. Map its stages to a CI/CD pipeline:

```markdown
## My CI/CD Pipeline Analysis

### Project: [Name of project]

### Current Manual Steps:
1. [What do you do now when deploying?]
2.
3.

### CI/CD Equivalent:
| Manual Step | CI/CD Automation |
|---|---|
| Run tests manually | pytest runs automatically on every push |
| Build Docker image | docker build in pipeline |
| SSH to server | kubectl apply or AWS CodeDeploy |
| Check app is running | Automated health check |

### What Would Require Most Work to Automate?
[Your honest assessment]

### Which CI/CD Tool Would You Choose and Why?
[Based on your research]
```

---

### Task 2: CI/CD Tool Research

Use the table above as a starting point and write a brief note on each tool:

```markdown
## CI/CD Tool Research Notes

### Jenkins
- Type: Self-hosted open-source automation server
- Best for: Teams that want maximum flexibility and control
- Key feature: 1800+ plugins for almost any integration
- Learning curve: Moderate — requires infrastructure to run
- My take: [Your assessment]

### GitHub Actions
- Type: Cloud-hosted, built into GitHub
- Best for: Teams already using GitHub
- Key feature: Zero infrastructure setup, marketplace of actions
- Learning curve: Low — YAML-based, good documentation
- My take:

### GitLab CI
- Type: Built into GitLab
- Best for: Teams using GitLab for source control
- Key feature: Single platform for code, CI/CD, and operations
- My take:

### CircleCI
- Type: Cloud + self-hosted
- Best for: Teams wanting fast builds with Docker support
- My take:

[Continue for each tool you research]
```

---

## Part 6: Jenkins — The Automation Maestro

### What Is Jenkins?

Jenkins is an open-source, self-hosted automation server written in Java. It is one of the oldest and most widely deployed CI/CD platforms, with an ecosystem of over 1,800 plugins that integrate it with virtually every tool in the DevOps landscape.

```
Jenkins Architecture:
──────────────────────────────────────────────
Jenkins Master (Controller)
  ├── Web UI (port 8080)
  ├── Job scheduling and management
  ├── Plugin management
  ├── Build queue management
  └── Distributes work to Agents

Jenkins Agents (Workers)
  ├── Agent 1: Linux build node
  ├── Agent 2: Windows test node
  └── Agent 3: Docker container (ephemeral)

Source Code Management:
  └── GitHub/GitLab → Webhook → Jenkins → Build triggered
```

**Jenkins key capabilities:**

| Capability | Description |
|---|---|
| **Freestyle Jobs** | Simple, GUI-configured build jobs |
| **Pipelines** | Code-defined workflows (Jenkinsfile) |
| **Multibranch Pipeline** | Automatic pipeline per Git branch |
| **Shared Libraries** | Reusable pipeline code across projects |
| **Plugin Ecosystem** | 1,800+ plugins for any integration |
| **Distributed Builds** | Master-agent architecture for scale |
| **SCM Integration** | Git, SVN, Mercurial support |
| **Notifications** | Email, Slack, Teams, PagerDuty |

---

## Part 7: Installing Jenkins

### Option 1: Jenkins in Docker (Recommended for Learning)

Running Jenkins inside Docker is the fastest way to get started with no system dependencies:

```bash
# Create a Docker network for Jenkins
docker network create jenkins

# Create a volume for persistent Jenkins data
docker volume create jenkins-data

# Run Jenkins
docker run -d \
    --name jenkins \
    --network jenkins \
    --restart unless-stopped \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins-data:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    jenkins/jenkins:lts-jdk17

# Watch startup logs
docker logs -f jenkins

# Wait for this line:
# Jenkins initial setup is required. An admin user has been created and...
# Please use the following password to proceed to installation:
# <32-character-password>
```

**Port reference:**
- `8080` — Jenkins Web UI
- `50000` — Jenkins agent connection port (JNLP)
- `/var/run/docker.sock` mounted — allows Jenkins to run Docker commands

---

### Option 2: Install Jenkins on Ubuntu/Debian (EC2)

```bash
# Step 1: Install Java (Jenkins requires Java 11 or 17)
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre

# Verify Java
java -version
# Output: openjdk version "17.x.x"

# Step 2: Add Jenkins repository
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | \
    sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Step 3: Install Jenkins
sudo apt update
sudo apt install -y jenkins

# Step 4: Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Step 5: Check status
sudo systemctl status jenkins

# Step 6: Get the initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

### Option 3: Install Jenkins on Amazon Linux 2 (EC2)

```bash
# Step 1: Install Java
sudo amazon-linux-extras install java-openjdk11 -y

# Step 2: Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import \
    https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Step 3: Install Jenkins
sudo yum install -y jenkins

# Step 4: Start and enable
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Step 5: Get initial password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

### Access Jenkins Web UI

```
1. Open browser: http://localhost:8080
   (Or http://your-ec2-public-ip:8080 for EC2)

2. If using EC2 — open port 8080 in Security Group:
   Inbound rule: Custom TCP, Port 8080, Source: Your IP

3. Enter the initial admin password from:
   - Docker: docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   - Linux:  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## Part 8: Setting Up Jenkins — First Launch

### The Setup Wizard

```
Screen 1: Unlock Jenkins
  → Enter the initial admin password

Screen 2: Customize Jenkins — Install Plugins
  → Choose "Install suggested plugins" (recommended for beginners)
  → Jenkins installs: Git, Pipeline, Docker, credentials plugins, etc.
  → Wait 2-5 minutes

Screen 3: Create First Admin User
  → Username: admin (or your choice)
  → Password: (choose a strong password)
  → Full name: Your name
  → Email: your@email.com
  → Click "Save and Continue"

Screen 4: Instance Configuration
  → Jenkins URL: http://localhost:8080/ (or your EC2 URL)
  → Click "Save and Finish"

Screen 5: Jenkins is ready!
  → Click "Start using Jenkins"
```

---

### Recommended Plugins to Install

After initial setup, install additional useful plugins:

```
Manage Jenkins → Plugins → Available Plugins

Search and install:
□ Blue Ocean              — Modern pipeline UI
□ Docker Pipeline         — Build/push Docker images in pipelines
□ Docker Commons          — Docker integration
□ GitHub Integration      — GitHub webhook support
□ Slack Notification      — Send build status to Slack
□ Pipeline: Stage View    — Visualize pipeline stages
□ Credentials Binding     — Use secrets in pipelines
□ Workspace Cleanup       — Clean workspaces after builds
□ AnsiColor               — Colorize build output
```

---

## Part 9: Your First Jenkins Job

### Create a Freestyle Project

```
1. Jenkins Dashboard → "New Item"
2. Enter name: "nexus-hello-world"
3. Select "Freestyle project"
4. Click OK

5. Configure the job:

   General:
   → Description: "My first Jenkins job"

   Build Steps:
   → Click "Add build step"
   → Select "Execute shell"
   → Enter:
     #!/bin/bash
     echo "=============================="
     echo " NexusCorp Jenkins First Build"
     echo "=============================="
     echo "Build number: ${BUILD_NUMBER}"
     echo "Build URL: ${BUILD_URL}"
     echo "Workspace: ${WORKSPACE}"
     echo "Node: ${NODE_NAME}"
     echo "Date: $(date)"
     echo "=============================="
     echo "Running basic checks..."
     which python3 && python3 --version
     which docker && docker --version
     echo "All checks passed!"

6. Click "Save"
7. Click "Build Now"
8. Click the build number (#1) → "Console Output"
```

**Expected console output:**
```
Started by user admin
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/nexus-hello-world
[nexus-hello-world] $ /bin/bash /tmp/jenkins-script...
==============================
 NexusCorp Jenkins First Build
==============================
Build number: 1
Build URL: http://localhost:8080/job/nexus-hello-world/1/
Workspace: /var/jenkins_home/workspace/nexus-hello-world
Node: built-in
Date: Mon Jun 10 14:32:01 UTC 2024
==============================
Running basic checks...
/usr/bin/python3
Python 3.11.2
/usr/bin/docker
Docker version 24.0.5, build ced0996
All checks passed!
Finished: SUCCESS
```

---

### Create a Job with SCM Integration

Connect Jenkins to a Git repository:

```
1. Create new Freestyle project: "nexus-git-build"

2. Source Code Management:
   → Select "Git"
   → Repository URL: https://github.com/yourusername/your-repo.git
   → For private repos: Add Credentials
     → Kind: Username with password
     → Username: your-github-username
     → Password: your-personal-access-token (not password)

3. Build Triggers:
   → Check "GitHub hook trigger for GITScm polling"
   → (Or "Poll SCM": H/5 * * * *  for every 5 minutes)

4. Build Steps → Execute shell:
   #!/bin/bash
   echo "Cloned repository: $GIT_URL"
   echo "Branch: $GIT_BRANCH"
   echo "Commit: $GIT_COMMIT"
   ls -la
   if [ -f requirements.txt ]; then
       pip3 install -r requirements.txt
   fi
   if [ -f pytest.ini ] || [ -d tests/ ]; then
       python3 -m pytest tests/ -v
   fi

5. Post-build Actions:
   → Add "Archive the artifacts" (optional)
   → Add "Publish JUnit test result report" (if tests output XML)
   → Add "Email Notification" → your@email.com

6. Save and Build Now
```

---

### Jenkins Built-In Variables

| Variable | Value | Example |
|---|---|---|
| `BUILD_NUMBER` | Sequential build number | `42` |
| `BUILD_URL` | Full URL to this build | `http://jenkins:8080/job/myapp/42/` |
| `JOB_NAME` | Name of the job | `nexus-api-build` |
| `WORKSPACE` | Path to build workspace | `/var/jenkins_home/workspace/nexus-api-build` |
| `GIT_COMMIT` | Full SHA of current commit | `a1b2c3d4e5f6...` |
| `GIT_BRANCH` | Branch being built | `origin/main` |
| `GIT_URL` | Repository URL | `https://github.com/nexus/api.git` |
| `NODE_NAME` | Jenkins agent name | `built-in` |
| `JENKINS_URL` | Jenkins server URL | `http://localhost:8080/` |

---

## Part 10: Jenkins Pipelines

### What Is a Jenkinsfile?

A Jenkinsfile is a text file that defines a Jenkins pipeline as code. It lives in the root of your repository alongside your application code — versioned, reviewed, and testable.

```groovy
// Jenkinsfile — NexusCorp API Build Pipeline
pipeline {
    agent any          // Run on any available agent

    environment {
        APP_NAME = 'nexus-api'
        DOCKER_REGISTRY = 'nexuscorp'
        IMAGE_TAG = "${env.GIT_COMMIT[0..7]}"   // First 8 chars of commit SHA
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out branch: ${env.GIT_BRANCH}"
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r requirements.txt'
            }
        }

        stage('Lint') {
            steps {
                sh 'flake8 . --count --max-line-length=88'
            }
        }

        stage('Test') {
            steps {
                sh 'pytest tests/ -v --junitxml=test-results.xml'
            }
            post {
                always {
                    junit 'test-results.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} .
                    docker tag ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} \
                               ${DOCKER_REGISTRY}/${APP_NAME}:latest
                """
            }
        }

        stage('Push to Registry') {
            when {
                branch 'main'    // Only push on main branch builds
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                        docker push ${DOCKER_REGISTRY}/${APP_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    docker pull ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                    docker stop nexus-api-staging 2>/dev/null || true
                    docker rm nexus-api-staging 2>/dev/null || true
                    docker run -d \
                        --name nexus-api-staging \
                        -p 5001:5000 \
                        -e APP_ENV=staging \
                        ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Smoke Test') {
            when {
                branch 'main'
            }
            steps {
                sleep 10  // Wait for container to start
                sh 'curl -f http://localhost:5001/health || exit 1'
                echo "Smoke test passed — staging is healthy"
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded! Build: ${env.BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed at stage: ${env.STAGE_NAME}"
            // mail to: 'team@nexuscorp.com', subject: "Build Failed: ${env.JOB_NAME}"
        }
        always {
            cleanWs()    // Clean workspace after every build
        }
    }
}
```

---

### Declarative vs Scripted Pipeline

```groovy
// Declarative (recommended — structured, readable)
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }
}

// Scripted (flexible — Groovy code, fewer constraints)
node {
    stage('Build') {
        sh 'make build'
    }
}
```

**Use Declarative** for most pipelines — it enforces structure, is easier to read, and has better tooling support. Use Scripted only when you need logic that Declarative cannot express.

---

### Jenkins Distributed Builds

Jenkins uses a **controller-agent** (formerly master-slave) architecture for distributed builds:

```
Jenkins Controller
    ├── Manages jobs and scheduling
    ├── Web UI
    └── Distributes work to agents

Agent 1: Linux (Ubuntu 22.04)
    → Used for: Python, Docker builds

Agent 2: Linux (Amazon Linux)
    → Used for: AWS deployments

Agent 3: Docker Container
    → Ephemeral — spun up per build, destroyed after
    → Cleanest isolation between builds
```

```groovy
// Target a specific agent by label
pipeline {
    agent {
        label 'linux-docker'    // Only run on agents with this label
    }
    // ...
}

// Use a Docker container as the agent
pipeline {
    agent {
        docker {
            image 'python:3.11-slim'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Test') {
            steps {
                sh 'python3 -m pytest'    // Runs inside python:3.11-slim container
            }
        }
    }
}
```

---

## 🔧 Task: Install Jenkins and Set Up Your First Job

### Step-by-Step Exercise

```bash
# Option A: Quick start with Docker

# Step 1: Run Jenkins
docker run -d \
    --name jenkins \
    --restart unless-stopped \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins-data:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    jenkins/jenkins:lts-jdk17

# Step 2: Wait for Jenkins to start
docker logs -f jenkins 2>&1 | grep -A 3 "initial setup"

# Step 3: Get the initial password
docker exec jenkins \
    cat /var/jenkins_home/secrets/initialAdminPassword

# Step 4: Open http://localhost:8080
# Complete the setup wizard

# Step 5: Create and run your first job (see instructions above)

# Step 6: Create a Jenkinsfile in a local directory
mkdir nexus-pipeline-demo
cd nexus-pipeline-demo
git init

cat > Jenkinsfile << 'EOF'
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo 'Hello from NexusCorp Pipeline!'
                sh 'echo "Build: ${BUILD_NUMBER}" && date'
            }
        }
        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    python3 -c "assert 1 + 1 == 2, 'Math is broken'; print('Tests passed!')"
                '''
            }
        }
        stage('Report') {
            steps {
                echo "Pipeline complete for build ${BUILD_NUMBER}"
            }
        }
    }
    post {
        success { echo 'SUCCESS' }
        failure { echo 'FAILURE' }
    }
}
EOF

git add Jenkinsfile
git commit -m "Add initial Jenkinsfile"

# Step 7: In Jenkins, create a Pipeline job pointing to this repo
# Dashboard → New Item → Pipeline → Pipeline from SCM → Git
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **CI (Continuous Integration)** | Automatically building and testing code changes on every commit |
| **CD (Continuous Delivery)** | Keeping the codebase always deployable; production release requires manual trigger |
| **CD (Continuous Deployment)** | Automatically deploying every passing change to production without human intervention |
| **Pipeline** | An automated sequence of stages (build → test → deploy) |
| **Jenkinsfile** | A file defining a Jenkins pipeline as code, stored in the repository |
| **Build** | The process of compiling, packaging, and preparing code for deployment |
| **Artifact** | A build output (Docker image, JAR, binary) stored in a registry |
| **Agent** | A Jenkins worker node that executes pipeline stages |
| **Controller** | The Jenkins master server that manages jobs, agents, and the web UI |
| **Webhook** | An HTTP callback that triggers a Jenkins build when code is pushed |
| **SCM** | Source Code Management — Git, SVN, etc. |
| **Stage** | A logical group of steps in a pipeline (Build, Test, Deploy) |
| **Step** | A single task within a stage (run a shell command, copy a file) |
| **Post** | Actions that run after a pipeline or stage completes (always, success, failure) |
| **Plugin** | A Jenkins extension that adds new capabilities |
| **Blue Ocean** | A modern Jenkins UI designed for pipeline visualization |
| **Declarative Pipeline** | A structured Jenkinsfile syntax using `pipeline {}` block |
| **Scripted Pipeline** | A flexible Groovy-based Jenkinsfile using `node {}` block |
| **Multibranch Pipeline** | A Jenkins job type that automatically creates pipelines for each branch |
| **Integration Hell** | The pain caused by merging long-lived diverged branches — what CI prevents |
| **Smoke Test** | A quick basic test to verify a deployment is alive before deeper testing |
| **Rollback** | Reverting a deployment to the previous working version |
| **Feature Flag** | A toggle that enables/disables a feature without deploying new code |
| **GitOps** | Using Git as the single source of truth for infrastructure and deployment state |

---

## 📚 Resources

- [Jenkins Official Documentation](https://www.jenkins.io/doc/) — complete Jenkins reference
- [Jenkins Pipeline Syntax Reference](https://www.jenkins.io/doc/book/pipeline/syntax/) — Declarative pipeline guide
- [What is Cloud Native CI? — GitLab](https://about.gitlab.com/topics/ci-cd/cloud-native-continuous-integration/) — cloud-native CI/CD overview
- [GitLab CI/CD Overview — Official Docs](https://docs.gitlab.com/ee/ci/) — alternative CI/CD platform reference
- [Martin Fowler — Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html) — the original CI article by the person who popularized the concept
- [Trunk Based Development](https://trunkbaseddevelopment.com) — the branching strategy that makes CI possible
- [The DevOps Handbook](https://itrevolution.com/the-devops-handbook/) — essential reading on CI/CD culture and practice

---

## 🔭 Day 33 Preview: Jenkins Pipelines in Depth

Tomorrow you move from understanding CI/CD and basic Jenkins jobs to building **real pipelines** that connect Git, Docker, and deployment.

You will learn:
- Writing complete `Jenkinsfile`s for real applications
- Connecting Jenkins to GitHub with webhooks (auto-trigger on push)
- Storing secrets in Jenkins Credentials (no hardcoded passwords)
- Building and pushing Docker images from within a pipeline
- Deploying containers automatically after a successful build
- The full `git push` → `Jenkins builds` → `Docker image pushed` → `Container deployed` cycle

Tomorrow is where the entire curriculum — Linux, Git, Python, Docker, and now Jenkins — runs as one connected system.

---

> 💬 *"CI/CD doesn't just automate deployments — it changes the culture. When releases become routine and boring, engineers stop fearing them and start improving them."*
>
> — DevOps Learning by Yukta
