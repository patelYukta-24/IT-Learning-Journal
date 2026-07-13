# Day 37: Scaling Jenkins — Master-Agent Architecture

> **DevOps Transformation** | CI/CD & Automation Series

---

## 📋 Overview

Until now, every Jenkins build has run on the master server itself — the same machine that manages the UI, schedules jobs, and stores all configurations. For a small team with light build loads, this works fine. But as NexusCorp adds more developers, more projects, and more complex builds, a single Jenkins master becomes a bottleneck.

The **Master-Agent architecture** is Jenkins' answer to scale. The master orchestrates and the agents do the work — and you can add as many agents as you need, each with its own environment, tools, and capacity.

By the end of today you will have provisioned an EC2 instance as a Jenkins agent, connected it to the master, run jobs on the agent, validated its environment, and demonstrated parallel execution across multiple agents.

---

## 🗺️ What You'll Do Today

| Topic | What It Covers |
|---|---|
| Jenkins Master role | Orchestration, UI, scheduling, configuration |
| Jenkins Agent role | Execution, labeled identity, distribution |
| Why this architecture? | Scalability, versatility, efficiency |
| Pre-requisites | Java, Docker, SSH, user permissions |
| Task 1 | Set up a new EC2 agent node |
| Task 2 | Run existing jobs on the new agent |
| Task 3 | Agent environment validation job |
| Task 4 | Parallel execution across agents |

---

## 🏢 NexusCorp Scenario

> NexusCorp's Jenkins master is struggling. Build queue times have reached 20 minutes during peak hours. The backend team needs Python 3.11 and Docker. The data team needs Python 3.9 and Spark. Running both on the same server causes conflicts. The frontend team needs Node.js 20, which conflicts with Node.js 16 used by a legacy project.
>
> The solution: dedicated agents. One agent for the backend team (Python 3.11, Docker), one for the data team (Python 3.9, Spark), one for the frontend team (Node.js 20). Each team's builds run in isolation. The master manages everything but executes nothing.

---

## Part 1: Jenkins Master (Server)

### The Master's Role

The Jenkins master is the **brain** of the CI/CD system. It does not run build steps itself (in a properly configured master-agent setup) — it manages everything that makes builds happen.

```
Jenkins Master Responsibilities:
─────────────────────────────────────────────────────────────

Orchestration:
  ├── Receives build triggers (webhooks, schedules, manual)
  ├── Queues jobs when agents are busy
  ├── Selects the right agent based on labels
  └── Reports overall status to the user

Configuration Management:
  ├── Stores all job configurations
  ├── Manages credentials and secrets
  ├── Maintains plugin configurations
  └── Stores user accounts and permissions

User Interface:
  ├── Serves the web UI on port 8080
  ├── Shows build history and logs
  ├── Provides Blue Ocean visualization
  └── REST API for external integrations

Build History and Artifacts:
  ├── Stores console output from all builds
  ├── Archives artifacts from completed builds
  └── Maintains test result history
```

### What the Master Should NOT Do

In a scaled Jenkins setup, the master should be configured with **zero executors** — meaning it runs no build jobs itself. All execution is delegated to agents. This keeps the master responsive, secure, and focused on orchestration.

```
Dashboard → Manage Jenkins → Configure System
→ "# of executors": set to 0

# This prevents any job from running on the master
# All builds must specify an agent label or use 'agent any'
# (which will only match available agents, not the master)
```

---

## Part 2: Jenkins Agent

### The Agent's Role

Jenkins agents (historically called "slaves," now called "nodes" or "agents") are worker machines that execute build steps on behalf of the master.

```
Jenkins Agent Responsibilities:
─────────────────────────────────────────────────────────────

Execution:
  ├── Clones the repository from SCM
  ├── Runs build steps (compile, test, package)
  ├── Executes Docker commands
  ├── Runs deployments to target environments
  └── Reports results back to master

Labeled Identity:
  ├── Each agent has one or more labels
  ├── Labels describe the agent's capabilities
  │   Examples: 'linux', 'docker', 'python3.11', 'ubuntu-22'
  └── Jobs target agents by label

Workspace:
  ├── Each job gets its own workspace on the agent
  ├── Files are checked out and built here
  └── Cleaned up after build (if configured)
```

### Agent Communication Methods

| Method | How It Works | Best For |
|---|---|---|
| **SSH** | Master SSHes into agent and starts the agent JAR | Linux/macOS agents — most common |
| **JNLP / WebSocket** | Agent initiates connection to master | Agents behind firewalls, Windows agents |
| **Docker** | Spins up a Docker container as an agent | Ephemeral, clean build environments |
| **Kubernetes** | Spins up a K8s pod as an agent | Cloud-native, auto-scaling |
| **Windows service** | Agent runs as a Windows service | Windows build agents |

---

## Part 3: Why Master-Agent Architecture?

### The Single-Master Problem

```
Without agents (single master):

Build A starts: npm install (2 min)
Build B queued: pytest (4 min)     ← waiting for A to finish
Build C queued: docker build (5min) ← waiting for A and B
Build D queued: integration test   ← waiting for A, B, and C

Total time: 2 + 4 + 5 + 8 = 19 minutes
Developer frustration: HIGH
```

```
With agents (master + 3 agents):

Agent 1: Build A → npm install (2 min)
Agent 2: Build B → pytest (4 min)       ← simultaneous
Agent 3: Build C → docker build (5 min) ← simultaneous
Agent 1: Build D → integration test (after A finishes)

Total time: max(2, 4, 5) + 8 = 13 minutes
Developer frustration: LOW
```

### Three Core Benefits

#### 1. Scalability

Add agents as workload grows. No reconfiguration of the master required. One Jenkins master can manage hundreds of agents.

```
Today: 1 master + 2 agents = handles 20 jobs/day
Next month: 1 master + 8 agents = handles 80 jobs/day
Next year: 1 master + 30 agents = handles 300 jobs/day
```

#### 2. Versatility

Different agents can have different environments. No conflicts, no version juggling on a shared server.

```
Agent: python-3.11-agent
  → Python 3.11, pip, pytest, Docker
  → Label: python, docker, backend

Agent: python-3.9-agent
  → Python 3.9, Spark, Hadoop
  → Label: python, data, spark

Agent: node-20-agent
  → Node.js 20, npm, Cypress
  → Label: node, frontend, e2e

Agent: windows-agent
  → Windows 11, .NET SDK, PowerShell
  → Label: windows, dotnet

Master runs no builds — just orchestrates
```

#### 3. Efficiency

The master stays fast and responsive. When it handles no builds, it can focus on scheduling, logging, and serving the UI without resource contention.

---

## Part 4: Pre-requisites for Setting Up an Agent

Before connecting an agent to Jenkins master, the agent machine needs:

### Required Software

```bash
# 1. Java — MUST match the version on Jenkins master
# Check master's Java version:
java -version
# Output: openjdk version "17.x.x"

# Install matching Java on agent (Ubuntu/Debian):
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre

# Verify
java -version
# Must match master's version

# 2. Docker (if containerized builds needed)
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# 3. Git (for cloning repositories)
sudo apt install -y git

# 4. Python (if Python builds needed)
sudo apt install -y python3 python3-pip
```

---

### Create the Jenkins User on the Agent

```bash
# Create jenkins user on the agent machine
sudo useradd -m -s /bin/bash jenkins

# Add jenkins to docker group (if Docker builds needed)
sudo usermod -aG docker jenkins

# Create Jenkins workspace directory
sudo mkdir -p /var/jenkins
sudo chown -R jenkins:jenkins /var/jenkins

# Verify
sudo -u jenkins docker ps    # Should work without permission errors
```

---

### Set Up SSH Key Authentication

The Jenkins master connects to agents via SSH. The master's public key must be in the agent's `authorized_keys`.

```bash
# On the MASTER machine:
# Generate SSH key pair (if not already done)
ssh-keygen -t ed25519 -C "jenkins-master" -f ~/.ssh/jenkins_agent_key
# This creates:
#   ~/.ssh/jenkins_agent_key     (private key — stays on master)
#   ~/.ssh/jenkins_agent_key.pub (public key — goes to agent)

# View the public key (you'll need to copy this)
cat ~/.ssh/jenkins_agent_key.pub
# ssh-ed25519 AAAA... jenkins-master
```

```bash
# On the AGENT machine:
# Add master's public key to jenkins user's authorized_keys
sudo -u jenkins mkdir -p /home/jenkins/.ssh
sudo -u jenkins chmod 700 /home/jenkins/.ssh

# Paste the master's public key here:
echo "ssh-ed25519 AAAA... jenkins-master" | \
    sudo tee /home/jenkins/.ssh/authorized_keys

sudo chown jenkins:jenkins /home/jenkins/.ssh/authorized_keys
sudo chmod 600 /home/jenkins/.ssh/authorized_keys

# Test SSH connection from master to agent
ssh -i ~/.ssh/jenkins_agent_key jenkins@AGENT_IP
# Should connect without password
```

---

## Part 5: Setting Up an EC2 Agent — Step by Step

### Step 1: Launch an EC2 Instance for the Agent

```
AWS Console → EC2 → Launch Instance

Name: nexus-jenkins-agent-01

AMI: Ubuntu Server 22.04 LTS

Instance type: t3.medium
  (2 vCPU, 4GB RAM — recommended for build agents)

Key pair: Select existing or create new
  → Download .pem file (you'll need it)

Security Group:
  → Allow SSH (port 22) from Jenkins master's IP only
  → NOT publicly accessible if possible

Storage: 30GB gp3 (builds need disk space)

→ Launch Instance
```

---

### Step 2: Install Pre-requisites on the Agent EC2

```bash
# SSH into the new EC2 instance
ssh -i "your-key.pem" ubuntu@EC2-AGENT-PUBLIC-IP

# Update packages
sudo apt update && sudo apt upgrade -y

# Install Java (match Jenkins master version)
sudo apt install -y fontconfig openjdk-17-jre
java -version

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# Install Git and Python
sudo apt install -y git python3 python3-pip

# Create Jenkins user
sudo useradd -m -s /bin/bash jenkins
sudo usermod -aG docker jenkins
sudo mkdir -p /var/jenkins
sudo chown -R jenkins:jenkins /var/jenkins

# Verify setup
echo "Java:"    && java -version
echo "Docker:"  && docker --version
echo "Git:"     && git --version
echo "Python:"  && python3 --version
```

---

### Step 3: Add SSH Credentials to Jenkins Master

```
Jenkins Master → Manage Jenkins → Credentials
→ System → Global credentials → Add Credentials

Kind: SSH Username with private key
Scope: Global
ID: jenkins-agent-ssh-key
Description: SSH key for EC2 Jenkins agents
Username: jenkins

Private Key:
  ✅ Enter directly
  → Paste the PRIVATE key content from master:
    cat ~/.ssh/jenkins_agent_key
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...paste entire key including header/footer...
    -----END OPENSSH PRIVATE KEY-----

→ Create
```

---

### Step 4: Add the Agent Node in Jenkins

```
Dashboard → Manage Jenkins → Nodes and Clouds
→ New Node (or "New Agent" in newer versions)

Node name: nexus-agent-01
Type: ✅ Permanent Agent
→ Create

Node Configuration:
  Description: Ubuntu 22.04 build agent on EC2
  Number of executors: 2  (builds that can run simultaneously)
  Remote root directory: /var/jenkins
  Labels: linux docker python ubuntu-22  (space-separated)
  Usage: ✅ Use this node as much as possible

Launch method: Launch agents via SSH
  Host: EC2-AGENT-PRIVATE-IP  (use private IP if same VPC)
  Credentials: jenkins-agent-ssh-key (select from dropdown)
  Host Key Verification Strategy: Non verifying Verification Strategy
  (Or: Manually Trusted Key Verification — more secure)

Availability: ✅ Keep this agent online as much as possible

→ Save
```

---

### Step 5: Verify Agent Connection

```
After saving, Jenkins automatically tries to connect:

Dashboard → Manage Jenkins → Nodes and Clouds
→ nexus-agent-01

Agent should show:
  Status: ● (blue/green dot) — Connected
  Executors: 2 (shows current usage)

If it shows ● (red) — Connection failed:
  → Click on the agent name
  → Click "Log" (left sidebar)
  → Read the error message:
    Common issues:
    - SSH authentication failed → Check key and authorized_keys
    - Java not found → Install Java on agent
    - Connection refused → Check Security Group port 22
    - Permission denied on /var/jenkins → Check ownership
```

---

### Step 6: Verify Agent from Command Line

```bash
# From the Jenkins master — verify SSH works
ssh -i ~/.ssh/jenkins_agent_key jenkins@AGENT-IP "java -version"
# Should show Java version without error

# Check agent is running Jenkins agent process
ssh -i ~/.ssh/jenkins_agent_key jenkins@AGENT-IP \
    "ps aux | grep jenkins"
# Should show jenkins agent java process after connection

# View agent log in Jenkins UI:
# Dashboard → Manage Jenkins → Nodes → nexus-agent-01 → Log
```

---

## Part 6: Practical Tasks

### Task 1: Set Up Jenkins Agent (Summary)

```
Complete checklist:

Infrastructure:
  □ EC2 instance launched (Ubuntu 22.04, t3.medium)
  □ Security Group: SSH from Jenkins master IP only

Software on agent:
  □ Java 17 installed and matches master
  □ Docker installed and running
  □ Jenkins user created
  □ Docker group membership for jenkins user
  □ /var/jenkins directory created with correct ownership

SSH:
  □ Master's public key in /home/jenkins/.ssh/authorized_keys
  □ Private key added to Jenkins credentials

Jenkins configuration:
  □ Node created: nexus-agent-01
  □ Labels set: linux docker python ubuntu-22
  □ Remote root: /var/jenkins
  □ Executors: 2
  □ Status: Connected (green dot)
```

---

### Task 2: Run Existing Jobs on the New Agent

Update previous jobs to run on the specific agent:

**For Freestyle jobs:**
```
Job Configuration → General
✅ Restrict where this project can be run
Label Expression: linux docker
   (must match one of your agent's labels)

→ Save → Build Now
→ Build should now show: "Running on nexus-agent-01"
```

**For Pipeline jobs (Jenkinsfile):**
```groovy
// Target the agent by label
pipeline {
    agent {
        label 'linux && docker'    // AND condition
        // or:
        label 'ubuntu-22'          // Match any agent with this label
    }
    stages {
        stage('Verify Agent') {
            steps {
                sh '''
                    echo "Running on: $(hostname)"
                    echo "User: $(whoami)"
                    echo "Workspace: ${WORKSPACE}"
                    java -version
                    docker --version
                '''
            }
        }
    }
}
```

---

### Task 3: Agent Environment Validation Job

Create a dedicated job that verifies the agent environment:

```groovy
// Jenkinsfile — Agent Environment Validation
pipeline {
    agent {
        label 'linux && docker'
    }

    stages {

        stage('System Information') {
            steps {
                sh '''
                    echo "========================================"
                    echo " Agent Environment Validation"
                    echo "========================================"
                    echo ""
                    echo "--- Agent Identity ---"
                    echo "Hostname:   $(hostname)"
                    echo "IP Address: $(hostname -I | awk '{print $1}')"
                    echo "Username:   $(whoami)"
                    echo "Groups:     $(groups)"
                    echo ""
                    echo "--- Operating System ---"
                    cat /etc/os-release | grep -E "^(NAME|VERSION|ID)="
                    uname -a
                '''
            }
        }

        stage('Environment Variables') {
            steps {
                sh '''
                    echo "========================================"
                    echo " All Environment Variables"
                    echo "========================================"
                    env | sort

                    echo ""
                    echo "--- Jenkins-Specific Variables ---"
                    echo "BUILD_NUMBER:  $BUILD_NUMBER"
                    echo "JOB_NAME:      $JOB_NAME"
                    echo "WORKSPACE:     $WORKSPACE"
                    echo "NODE_NAME:     $NODE_NAME"
                    echo "NODE_LABELS:   $NODE_LABELS"
                    echo "EXECUTOR_NUM:  $EXECUTOR_NUMBER"
                '''
            }
        }

        stage('Software Versions') {
            steps {
                sh '''
                    echo "========================================"
                    echo " Installed Software Versions"
                    echo "========================================"
                    echo ""
                    echo "--- Java ---"
                    java -version 2>&1

                    echo ""
                    echo "--- Docker ---"
                    docker --version
                    docker info --format \
                        "  Server: {{.ServerVersion}}\n  Storage: {{.Driver}}"

                    echo ""
                    echo "--- Git ---"
                    git --version

                    echo ""
                    echo "--- Python ---"
                    python3 --version
                    pip3 --version

                    echo ""
                    echo "--- System Resources ---"
                    echo "CPU cores: $(nproc)"
                    echo "Total RAM: $(free -h | awk '/^Mem:/ {print $2}')"
                    echo "Free RAM:  $(free -h | awk '/^Mem:/ {print $7}')"
                    echo "Disk:"
                    df -h / | awk 'NR==2 {print "  Total: "$2"\n  Used:  "$3"\n  Free:  "$4"\n  Use%:  "$5}'
                '''
            }
        }

        stage('Docker Functionality Test') {
            steps {
                sh '''
                    echo "========================================"
                    echo " Docker Functionality Test"
                    echo "========================================"
                    echo ""
                    echo "1. Running hello-world container..."
                    docker run --rm hello-world 2>&1 | grep "Hello from Docker"
                    echo "   ✓ Docker can pull and run containers"

                    echo ""
                    echo "2. Running Python in Docker..."
                    docker run --rm python:3.11-slim python3 -c \
                        "import sys; print(f'  ✓ Python {sys.version} in Docker')"

                    echo ""
                    echo "3. Building a simple image..."
                    echo -e "FROM alpine\nCMD [\"echo\", \"build test\"]" > /tmp/test.dockerfile
                    docker build -f /tmp/test.dockerfile -t agent-test:latest /tmp/
                    docker run --rm agent-test:latest
                    docker rmi agent-test:latest
                    echo "   ✓ Docker build works"
                '''
            }
        }

        stage('Network Connectivity') {
            steps {
                sh '''
                    echo "========================================"
                    echo " Network Connectivity"
                    echo "========================================"
                    echo ""
                    echo "1. DNS Resolution:"
                    nslookup google.com > /dev/null && echo "   ✓ DNS working" || echo "   ✗ DNS failed"

                    echo ""
                    echo "2. Internet connectivity:"
                    curl -s --max-time 5 https://google.com > /dev/null && \
                        echo "   ✓ Internet accessible" || \
                        echo "   ✗ Internet not accessible"

                    echo ""
                    echo "3. Docker Hub connectivity:"
                    curl -s --max-time 5 https://hub.docker.com > /dev/null && \
                        echo "   ✓ Docker Hub accessible" || \
                        echo "   ✗ Docker Hub not accessible"

                    echo ""
                    echo "4. GitHub connectivity:"
                    curl -s --max-time 5 https://github.com > /dev/null && \
                        echo "   ✓ GitHub accessible" || \
                        echo "   ✗ GitHub not accessible"
                '''
            }
        }

        stage('Validation Summary') {
            steps {
                sh '''
                    echo ""
                    echo "========================================"
                    echo " Validation Complete"
                    echo "========================================"
                    echo " Agent: $(hostname) is ready for CI/CD"
                    echo " Labels: linux docker python ubuntu-22"
                    echo "========================================"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Agent validation passed — nexus-agent-01 is ready"
        }
        failure {
            echo "❌ Agent validation failed — check console output"
        }
        always {
            cleanWs()
        }
    }
}
```

---

### Task 4: Parallel Execution Across Agents

Design a pipeline that distributes work across master and agents:

```groovy
// Jenkinsfile — Parallel Multi-Agent Pipeline
pipeline {
    agent none    // No default agent — each stage defines its own

    environment {
        APP_NAME = 'nexus-parallel-demo'
    }

    stages {

        stage('Trigger') {
            agent { label 'built-in' }    // Run on master
            steps {
                echo """
=== NexusCorp Parallel Pipeline ===
Build:    #${BUILD_NUMBER}
Agents:   master + nexus-agent-01
Strategy: Parallel execution
====================================
                """
            }
        }

        stage('Parallel Tests') {
            parallel {

                stage('Unit Tests — Agent') {
                    agent { label 'linux && docker' }
                    steps {
                        sh '''
                            echo "=== Unit Tests on: $(hostname) ==="
                            echo "Agent labels: $NODE_LABELS"
                            pip3 install -q pytest
                            python3 -c "
tests = [
    ('Add', 1 + 1 == 2),
    ('Sub', 5 - 3 == 2),
    ('Mul', 3 * 4 == 12),
    ('Div', 10 / 2 == 5.0),
]
print('Running unit tests...')
for name, result in tests:
    status = 'PASS' if result else 'FAIL'
    print(f'  [{status}] {name}')
failed = sum(1 for _, r in tests if not r)
print(f'Results: {len(tests) - failed}/{len(tests)} passed')
exit(failed)
"
                        '''
                    }
                }

                stage('Docker Build — Agent') {
                    agent { label 'linux && docker' }
                    steps {
                        sh '''
                            echo "=== Docker Build on: $(hostname) ==="
                            echo "Agent labels: $NODE_LABELS"

                            # Build a test image
                            cat > /tmp/Dockerfile.test << 'EOF'
FROM python:3.11-slim
WORKDIR /app
RUN echo "NexusCorp test image" > info.txt
CMD ["cat", "info.txt"]
EOF
                            docker build -f /tmp/Dockerfile.test -t nexus-test:${BUILD_NUMBER} /tmp/
                            docker run --rm nexus-test:${BUILD_NUMBER}
                            docker rmi nexus-test:${BUILD_NUMBER}
                            echo "Docker build stage complete"
                        '''
                    }
                }

                stage('Lint Check — Master') {
                    agent { label 'built-in' }    // Runs on master
                    steps {
                        sh '''
                            echo "=== Lint Check on: $(hostname) (master) ==="
                            echo "Node: $NODE_NAME"

                            python3 -c "
import ast, sys
code = '''
def greet(name):
    return f\"Hello, {name}!\"

def add(a, b):
    return a + b
'''
try:
    ast.parse(code)
    print('  ✓ Syntax check passed')
    print('  ✓ No obvious style issues')
    print('Lint check complete')
except SyntaxError as e:
    print(f'  ✗ Syntax error: {e}')
    sys.exit(1)
"
                        '''
                    }
                }

                stage('Security Scan — Agent') {
                    agent { label 'linux' }
                    steps {
                        sh '''
                            echo "=== Security Scan on: $(hostname) ==="
                            echo "Agent labels: $NODE_LABELS"

                            echo "Scanning for hardcoded secrets..."
                            python3 -c "
import re, sys
patterns = {
    'password': r'password\\s*=\\s*[\'\"]+.+[\'\"]+',
    'api_key':  r'api_key\\s*=\\s*[\'\"]+.+[\'\"]+',
    'secret':   r'secret\\s*=\\s*[\'\"]+.+[\'\"]+',
}
# Simulate scan — no real files to check in demo
print('  Scanning: app.py → No issues found')
print('  Scanning: config.py → No issues found')
print('  Scanning: utils.py → No issues found')
print('Security scan: PASSED')
"
                        '''
                    }
                }
            }
        }

        stage('Results Aggregation') {
            agent { label 'built-in' }    // Collect results on master
            steps {
                sh '''
                    echo "========================================"
                    echo " All Parallel Stages Complete"
                    echo "========================================"
                    echo ""
                    echo "Stages completed in parallel:"
                    echo "  ✅ Unit Tests     (nexus-agent-01)"
                    echo "  ✅ Docker Build   (nexus-agent-01)"
                    echo "  ✅ Lint Check     (jenkins-master)"
                    echo "  ✅ Security Scan  (nexus-agent-01)"
                    echo ""
                    echo "Build #${BUILD_NUMBER} — All checks passed"
                    echo "========================================"
                '''
            }
        }
    }

    post {
        success {
            echo "Parallel pipeline succeeded — all agents performed as expected"
        }
        failure {
            echo "One or more parallel stages failed"
        }
        always {
            echo "Build #${BUILD_NUMBER} complete"
        }
    }
}
```

---

## 🔧 Mini-Project: NexusCorp Agent Fleet Setup

**Scenario:** Set up two specialized agents for NexusCorp's different team needs, then run a pipeline that correctly distributes work.

```bash
# ── Agent 1: Python/Backend Agent ───────────────────────────────
# EC2: t3.medium, Ubuntu 22.04
# Labels: linux python docker backend

# Setup commands (run on agent 1 EC2):
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre docker.io git python3 python3-pip
sudo useradd -m -s /bin/bash jenkins
sudo usermod -aG docker jenkins
sudo mkdir -p /var/jenkins
sudo chown -R jenkins:jenkins /var/jenkins
# Add master SSH key to jenkins authorized_keys


# ── Agent 2: Node.js/Frontend Agent ─────────────────────────────
# EC2: t3.small, Ubuntu 22.04
# Labels: linux node frontend

# Setup commands (run on agent 2 EC2):
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre git

# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Create jenkins user
sudo useradd -m -s /bin/bash jenkins
sudo mkdir -p /var/jenkins
sudo chown -R jenkins:jenkins /var/jenkins
# Add master SSH key to jenkins authorized_keys


# ── Pipeline that uses both agents ──────────────────────────────
```

```groovy
// Jenkinsfile — NexusCorp Multi-Agent Fleet
pipeline {
    agent none

    stages {

        stage('Backend Tests') {
            agent { label 'python && backend' }
            steps {
                sh '''
                    echo "Backend tests on: $(hostname)"
                    python3 --version
                    pip3 install -q pytest
                    python3 -c "
print('Running backend tests...')
assert 2 + 2 == 4
assert 'nexus'.upper() == 'NEXUS'
print('All backend tests passed!')
"
                '''
            }
        }

        stage('Frontend Tests') {
            agent { label 'node && frontend' }
            steps {
                sh '''
                    echo "Frontend tests on: $(hostname)"
                    node --version
                    npm --version
                    node -e "
console.log('Running frontend tests...');
const results = [
  ['DOM render', true],
  ['State management', true],
  ['API call mock', true],
];
results.forEach(([name, passed]) => {
  console.log('  [' + (passed ? 'PASS' : 'FAIL') + '] ' + name);
});
console.log('All frontend tests passed!');
"
                '''
            }
        }

        stage('Docker Build') {
            agent { label 'docker' }
            steps {
                sh '''
                    echo "Docker build on: $(hostname)"
                    docker --version
                    echo "Building NexusCorp image..."
                    echo "Build #${BUILD_NUMBER} complete"
                '''
            }
        }
    }

    post {
        always {
            echo "Fleet pipeline complete — work distributed across agents"
        }
    }
}
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Jenkins Master** | The central Jenkins server that orchestrates builds, serves the UI, and manages configuration |
| **Jenkins Agent** | A worker machine that executes build steps on behalf of the master |
| **Node** | Jenkins' term for any machine in the Jenkins network — master or agent |
| **Executor** | A slot on a node for running one build at a time (agents typically have 2–4 executors) |
| **Label** | A tag on an agent describing its capabilities — jobs target agents by label |
| **Label expression** | A logical expression to select agents: `linux && docker`, `python || node` |
| **SSH agent connection** | The master connects to the agent over SSH and starts the Jenkins agent JAR |
| **JNLP connection** | The agent connects to the master (reverse connection — agent initiates) |
| **`built-in`** | The label for the Jenkins master node itself |
| **Remote root directory** | The directory on the agent where Jenkins stores build workspaces |
| **`/var/jenkins_home`** | Jenkins master's home directory (in Docker: persisted as a volume) |
| **`authorized_keys`** | File containing SSH public keys permitted to connect to a Linux user |
| **`jenkins` user** | The Linux user account on the agent that Jenkins connects as and runs builds as |
| **Zero executors** | Setting master executors to 0 — master runs no jobs, only orchestrates |
| **Permanent agent** | A node that stays connected — as opposed to cloud agents that spin up per build |
| **Ephemeral agent** | An agent that is created for one build and destroyed afterward (Docker, Kubernetes) |
| **Build queue** | Jobs waiting for an available executor on a matching agent |
| **`nproc`** | Linux command showing the number of available processor cores |
| **`t3.medium`** | AWS EC2 instance type — 2 vCPU, 4GB RAM — suitable for build agents |
| **Security Group** | AWS firewall rules controlling inbound/outbound traffic to EC2 instances |
| **`usermod -aG docker jenkins`** | Adds the jenkins user to the docker group — allows Docker commands without sudo |
| **`NODE_NAME`** | Jenkins environment variable containing the name of the agent running the build |
| **`NODE_LABELS`** | Jenkins environment variable listing all labels of the current agent |
| **`EXECUTOR_NUMBER`** | Jenkins environment variable identifying which executor slot is being used |
| **Parallel stages** | Multiple pipeline stages that run simultaneously on potentially different agents |
| **Load distribution** | Spreading builds across multiple agents to reduce queue time |
| **Agent fleet** | A collection of Jenkins agents managed by one master |

---

## 📚 Resources

- [Jenkins Distributed Builds — Official Docs](https://www.jenkins.io/doc/book/using/using-agents/) — official guide to setting up agents
- [Jenkins Nodes Management](https://www.jenkins.io/doc/book/managing/nodes/) — node configuration reference
- [Setting Up Jenkins on EC2 — AWS Tutorial](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/) — AWS-specific setup guide
- [SSH Agent Plugin Documentation](https://plugins.jenkins.io/ssh-slaves/) — SSH-based agent connection
- [Kubernetes Plugin for Jenkins](https://plugins.jenkins.io/kubernetes/) — for Kubernetes-based ephemeral agents
- [Jenkins Pipeline: Parallel Stages](https://www.jenkins.io/doc/book/pipeline/syntax/#parallel) — official parallel pipeline documentation

---

## 🔭 Day 38 Preview: Introduction to AWS Cloud

Today you scaled Jenkins horizontally with agents. Tomorrow you move to the cloud — the infrastructure layer where everything runs at scale.

You will learn:
- What AWS is and how it is organized (regions, availability zones, edge locations)
- The AWS Free Tier — what you can explore without spending money
- Core AWS services overview: EC2, S3, IAM, VPC, RDS, Lambda, ECS
- Setting up the AWS CLI and your first IAM user
- How your Jenkins agents, Docker containers, and applications run on AWS infrastructure

The Jenkins pipelines you've built will deploy to AWS. The Docker images you build will be stored in AWS ECR. The applications will run on EC2 or ECS. AWS is the next essential layer — and it starts tomorrow.

---

> 💬 *"A single Jenkins master is a single point of failure and a single point of capacity. Agents turn Jenkins from a tool into a platform."*
>
> — DevOps Learning by Yukta
