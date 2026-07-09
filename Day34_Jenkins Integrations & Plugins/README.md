# Day 34: Jenkins Integrations & Plugins

> **DevOps Transformation** | CI/CD & Automation Series

---

## 📋 Overview

Jenkins out of the box is a powerful automation server — but what makes it truly indispensable in a DevOps pipeline is its **plugin ecosystem**. With over 1,800 community-maintained plugins, Jenkins can integrate with virtually every tool in the modern software delivery stack: Git, Docker, Kubernetes, Slack, AWS, Jira, SonarQube, Terraform, and more.

Today you explore the plugin architecture, install and configure essential plugins, integrate Jenkins with Git, and set up build notifications — transforming Jenkins from a standalone server into the central hub of NexusCorp's CI/CD toolchain.

---

## 🗺️ What You'll Do Today

| Topic | What It Covers |
|---|---|
| Jenkins Plugin Architecture | What plugins are and how they work |
| Installing & Managing Plugins | Plugin Manager UI and CLI |
| Essential Plugin Categories | SCM, Build Tools, Docker, Notifications, UI |
| Git Integration | Connect Jenkins to GitHub with webhooks |
| Slack Notification Setup | Send build status messages to Slack |
| Tasks & Exercises | Plugin exploration, Git integration, notifications |

---

## 🏢 NexusCorp Scenario

> NexusCorp's Jenkins server is running but isolated — it doesn't know about their GitHub repositories, can't build Docker images, and sends build results nowhere. The team needs to integrate Jenkins with their full toolchain. Today you make those connections: Jenkins pulls code from GitHub, builds Docker images, and posts results to the team's Slack channel.

---

## Part 1: Introduction to Jenkins Plugins

### What Are Jenkins Plugins?

Plugins are extensions that add new capabilities to Jenkins. They are installed at runtime — no source code changes needed, no Jenkins restarts (usually). Each plugin provides one or more of:

- New job types or build steps
- Integration with external tools (Git, Docker, Slack)
- New UI elements
- New security features
- New reporting capabilities

```
Jenkins Core (the engine)
      │
      ├── Git Plugin           → Pull code from Git repositories
      ├── Docker Pipeline      → Build/run Docker in pipelines
      ├── Kubernetes Plugin    → Run builds in K8s pods
      ├── Slack Notification   → Post build results to Slack
      ├── SonarQube Scanner    → Run code quality analysis
      ├── Blue Ocean           → Modern pipeline UI
      ├── JUnit Plugin         → Parse and display test results
      ├── Credentials Plugin   → Store secrets safely
      └── ... 1,800+ more
```

**Plugin dependency chain:** Plugins can depend on other plugins. Installing one plugin often installs several dependencies automatically. Jenkins manages this graph for you.

---

### How Plugins Work Internally

```
Plugin Architecture:
─────────────────────────────────────────────────────

Jenkins Core          Plugin JAR
     │                    │
     │   ← API hooks ─────┤
     │                    ├── New build step type
     │                    ├── New post-build action
     │                    ├── New trigger type
     │                    ├── UI forms and views
     │                    └── New pipeline steps

Plugins are stored at:
  $JENKINS_HOME/plugins/PLUGIN_NAME.jpi (or .hpi)

Plugin data and config:
  $JENKINS_HOME/plugins/PLUGIN_NAME/

Docker path:
  /var/jenkins_home/plugins/
```

---

## Part 2: Installing and Managing Plugins

### Plugin Manager — Web UI

```
Navigation:
Dashboard → Manage Jenkins → Plugins

Four tabs:

1. Updates
   → Plugins with newer versions available
   → Always review before updating in production
   → Updates can sometimes break existing jobs

2. Available Plugins
   → All plugins not yet installed
   → Search by name or keyword
   → Check the box → Install (no restart needed for most)

3. Installed
   → All currently installed plugins
   → Shows version, enabled/disabled status
   → Can disable or uninstall from here

4. Advanced Settings
   → Upload a plugin .jpi/.hpi file directly
   → Configure update site URL (proxy settings)
   → Check for updates manually
```

---

### Installing Plugins via UI

```
Step-by-step:

1. Dashboard → Manage Jenkins → Plugins
2. Click "Available plugins" tab
3. Search bar: type plugin name
4. Check the box next to the plugin
5. Click "Install" (bottom of page)
   → "Download now and install after restart" (safe)
   → "Install without restart" (most plugins support this)
6. Watch the installation progress
7. Restart Jenkins if prompted:
   ✅ Restart Jenkins when installation is complete and
      no jobs are running
```

---

### Installing Plugins via Jenkins CLI

```bash
# Download the Jenkins CLI jar
curl -O http://localhost:8080/jnlpJars/jenkins-cli.jar

# Install plugins from command line
java -jar jenkins-cli.jar \
    -s http://localhost:8080 \
    -auth admin:YOUR_API_TOKEN \
    install-plugin git docker-workflow slack blueocean

# Restart Jenkins after bulk install
java -jar jenkins-cli.jar \
    -s http://localhost:8080 \
    -auth admin:YOUR_API_TOKEN \
    safe-restart

# List all installed plugins
java -jar jenkins-cli.jar \
    -s http://localhost:8080 \
    -auth admin:YOUR_API_TOKEN \
    list-plugins
```

---

### Installing Plugins via Docker (at Startup)

For Docker-based Jenkins, you can pre-install plugins using a custom image:

```dockerfile
# Dockerfile for Jenkins with pre-installed plugins
FROM jenkins/jenkins:lts-jdk17

# Install plugins during image build
RUN jenkins-plugin-cli --plugins \
    git \
    github \
    docker-workflow \
    docker-commons \
    pipeline-stage-view \
    blueocean \
    slack \
    email-ext \
    credentials-binding \
    workspace-cleanup \
    ansicolor \
    timestamper
```

```bash
# Build and run custom Jenkins image
docker build -t nexuscorp-jenkins .
docker run -d \
    --name jenkins \
    -p 8080:8080 \
    -v jenkins-data:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    nexuscorp-jenkins
```

---

### Managing Plugin Updates Safely

```
Production Plugin Update Strategy:
───────────────────────────────────

1. Test updates in a staging Jenkins first
2. Read the plugin changelog for breaking changes
3. Backup $JENKINS_HOME before updating
4. Update one plugin at a time (isolate issues)
5. Keep a list of pinned plugin versions
6. Never auto-update plugins in production without testing

Backup before updating:
docker cp jenkins:/var/jenkins_home/plugins ./plugins-backup-$(date +%Y%m%d)
```

---

## Part 3: Essential Jenkins Plugins

### Category 1: Source Code Management (SCM)

```
┌─────────────────────────────────────────────────────────┐
│ Git Plugin                                              │
│ → The foundation — enables Jenkins to clone/fetch repos │
│ → Supports: checkout, polling, branch filtering        │
│ → Install: "git"                                       │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ GitHub Integration Plugin                               │
│ → Webhook support from GitHub                          │
│ → GitHub status checks on pull requests                │
│ → Commit status updates (red/green checks in GitHub UI)│
│ → Install: "github"                                    │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ GitLab Plugin                                           │
│ → Same capabilities for GitLab repositories            │
│ → Merge request integration                            │
│ → Install: "gitlab-plugin"                             │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Bitbucket Plugin                                        │
│ → Webhooks from Bitbucket Cloud and Server             │
│ → Install: "bitbucket"                                 │
└─────────────────────────────────────────────────────────┘
```

**Pipeline steps enabled by Git plugin:**
```groovy
// In Jenkinsfile
stage('Checkout') {
    steps {
        git url: 'https://github.com/nexuscorp/api.git',
            branch: 'main',
            credentialsId: 'github-token'

        // Or use the declarative checkout
        checkout scm
    }
}
```

---

### Category 2: Build Tools

```
┌─────────────────────────────────────────────────────────┐
│ Maven Integration Plugin                               │
│ → Run Maven goals (compile, test, package)             │
│ → Parse Maven output for test results                  │
│ → Install: "maven-plugin"                              │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Gradle Plugin                                           │
│ → Run Gradle builds for Java/Android projects          │
│ → Install: "gradle"                                    │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ NodeJS Plugin                                           │
│ → Manage multiple Node.js versions                     │
│ → Run npm, yarn commands                               │
│ → Install: "nodejs"                                    │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Pipeline Utility Steps                                  │
│ → readJSON, writeJSON, readYaml, findFiles             │
│ → Useful for parsing config files in pipelines         │
│ → Install: "pipeline-utility-steps"                    │
└─────────────────────────────────────────────────────────┘
```

---

### Category 3: Containerization & Deployment

```
┌─────────────────────────────────────────────────────────┐
│ Docker Pipeline Plugin                                  │
│ → Build Docker images in pipeline                      │
│ → Push to registries                                   │
│ → Run containers as build environments                 │
│ → Install: "docker-workflow"                           │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Docker Commons Plugin                                   │
│ → Shared Docker utilities (required by docker-workflow)│
│ → Install: "docker-commons"                            │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Kubernetes Plugin                                       │
│ → Run Jenkins agents as Kubernetes pods                │
│ → Dynamic scaling — pods spin up and down per build    │
│ → Install: "kubernetes"                                │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ SSH Agent Plugin                                        │
│ → Use SSH keys in pipeline builds                      │
│ → Deploy via SSH to remote servers                     │
│ → Install: "ssh-agent"                                 │
└─────────────────────────────────────────────────────────┘
```

**Docker Pipeline example:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build("nexuscorp/api:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com',
                                        'dockerhub-credentials') {
                        docker.image("nexuscorp/api:${env.BUILD_NUMBER}").push()
                        docker.image("nexuscorp/api:${env.BUILD_NUMBER}").push('latest')
                    }
                }
            }
        }
    }
}
```

---

### Category 4: Notification Plugins

```
┌─────────────────────────────────────────────────────────┐
│ Slack Notification Plugin                               │
│ → Post build results to Slack channels                 │
│ → Custom messages with build details                   │
│ → Install: "slack"                                     │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Email Extension Plugin                                  │
│ → Configurable email notifications                     │
│ → HTML templates, attachments, per-recipient rules     │
│ → Install: "email-ext"                                 │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Microsoft Teams Notification                            │
│ → Send build status to Microsoft Teams                 │
│ → Install: "Office-365-Connector"                      │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ PagerDuty Plugin                                        │
│ → Trigger PagerDuty incidents on build failures        │
│ → Install: "pagerduty"                                 │
└─────────────────────────────────────────────────────────┘
```

---

### Category 5: UI/UX Plugins

```
┌─────────────────────────────────────────────────────────┐
│ Blue Ocean                                              │
│ → Modern visual pipeline UI                            │
│ → Visual stage-by-stage pipeline view                  │
│ → Better log viewing with collapsible stages           │
│ → Install: "blueocean"                                 │
│ → Access: http://jenkins:8080/blue                     │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Pipeline Stage View                                     │
│ → Classic UI — shows pipeline stages as columns        │
│ → Lighter than Blue Ocean                              │
│ → Install: "pipeline-stage-view"                       │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ AnsiColor Plugin                                        │
│ → Colorizes console output (makes logs readable)       │
│ → Install: "ansicolor"                                 │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Timestamper                                             │
│ → Adds timestamps to every console output line        │
│ → Install: "timestamper"                               │
└─────────────────────────────────────────────────────────┘
```

---

### Category 6: Security & Credentials

```
┌─────────────────────────────────────────────────────────┐
│ Credentials Plugin (usually pre-installed)              │
│ → Secure storage for passwords, SSH keys, tokens       │
│ → Referenced by ID in Jenkinsfiles                     │
│ → Install: "credentials"                               │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Credentials Binding Plugin                              │
│ → Inject stored credentials into build environment     │
│ → Use secrets as env vars in shell steps               │
│ → Install: "credentials-binding"                       │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│ Role-Based Authorization Strategy                       │
│ → Fine-grained access control                          │
│ → Define roles with specific permissions               │
│ → Install: "role-strategy"                             │
└─────────────────────────────────────────────────────────┘
```

---

### Recommended Plugin Installation List

Install all of these for a production-ready Jenkins:

```
# Copy-paste into Jenkins Plugin Manager search, one by one:

SCM:
  git
  github
  gitlab-plugin

Build Tools:
  pipeline-utility-steps
  workspace-cleanup

Docker:
  docker-workflow
  docker-commons

Notifications:
  slack
  email-ext

UI:
  blueocean
  pipeline-stage-view
  ansicolor
  timestamper

Security:
  credentials-binding
  role-strategy

Testing:
  junit
  htmlpublisher
```

---

## Part 4: Integrating Jenkins with Git

### Setting Up Git Credentials in Jenkins

Before Jenkins can pull from a private repository, it needs credentials:

```
Step 1: Generate a GitHub Personal Access Token (PAT)
─────────────────────────────────────────────────────
GitHub.com → Settings → Developer Settings
→ Personal access tokens → Tokens (classic)
→ Generate new token (classic)
→ Name: "Jenkins CI"
→ Expiration: 90 days (or No expiration for testing)
→ Scopes:
  ✅ repo (full repository access)
  ✅ admin:repo_hook (manage webhooks)
→ Generate token
→ COPY THE TOKEN NOW (you can't see it again)


Step 2: Add Credentials to Jenkins
────────────────────────────────────
Dashboard → Manage Jenkins → Credentials
→ System → Global credentials → Add Credentials

Kind: Username with password
Scope: Global
Username: your-github-username
Password: ghp_xxxx (your Personal Access Token)
ID: github-credentials          ← This ID is used in Jenkinsfile
Description: GitHub PAT for Jenkins CI

→ Create
```

---

### Create a Jenkins Job with Git Integration

```
1. Dashboard → New Item
2. Name: nexus-git-integration
3. Type: Freestyle project → OK

4. Source Code Management:
   ✅ Git
   Repository URL: https://github.com/YOUR-USERNAME/YOUR-REPO.git
   Credentials: github-credentials (select from dropdown)
   Branch: */main

5. Build Triggers:
   ✅ GitHub hook trigger for GITScm polling
   (OR: Poll SCM for repos without webhooks)
   Schedule: H/5 * * * *

6. Build Steps → Execute shell:

#!/bin/bash
echo "=== Git Integration Test ==="
echo "Repository: $GIT_URL"
echo "Branch:     $GIT_BRANCH"
echo "Commit:     $GIT_COMMIT"
echo "Committer:  $GIT_COMMITTER_NAME <$GIT_COMMITTER_EMAIL>"
echo ""
echo "Files in repository:"
ls -la
echo ""
echo "Git log (last 3 commits):"
git log --oneline -3
echo ""
echo "Repository structure:"
find . -not -path './.git/*' -type f | head -20

7. Save → Build Now
```

---

### Setting Up GitHub Webhooks

Webhooks make Jenkins build automatically on every `git push` — no polling required:

```
Step 1: Configure Jenkins to accept webhooks
─────────────────────────────────────────────
Your Jenkins must be publicly accessible:
  → If running locally, use ngrok for a public URL:
    ngrok http 8080
    → Gives you: https://a1b2c3.ngrok.io

  → On EC2: use your public IP:
    http://YOUR-EC2-IP:8080
    (Make sure port 8080 is open in Security Group)

Manage Jenkins → Configure System
→ GitHub
→ Add GitHub Server:
  API URL: https://api.github.com
  Credentials: Add → Secret text
    Secret: YOUR-GITHUB-PAT
    ID: github-api-token
→ Test connection → "Credentials verified"


Step 2: Add webhook on GitHub
──────────────────────────────
GitHub Repository → Settings → Webhooks
→ Add webhook:
  Payload URL: http://YOUR-JENKINS-URL:8080/github-webhook/
  Content type: application/json
  Secret: (leave empty for testing)
  Which events:
    ✅ Just the push event
→ Add webhook

Test: GitHub sends a test ping
→ Green checkmark = webhook is connected


Step 3: Verify — push code and watch Jenkins build
────────────────────────────────────────────────────
# On your local machine:
echo "# Test commit" >> README.md
git add README.md
git commit -m "Test Jenkins webhook"
git push origin main

# Watch Jenkins:
# Dashboard → nexus-git-integration → build starts automatically!
```

---

### Using Git Variables in Pipelines

When Jenkins clones a repository, it sets these environment variables:

```groovy
pipeline {
    agent any
    stages {
        stage('Git Info') {
            steps {
                sh '''
                    echo "URL:        $GIT_URL"
                    echo "Branch:     $GIT_BRANCH"
                    echo "Commit:     $GIT_COMMIT"
                    echo "Short SHA:  ${GIT_COMMIT:0:8}"
                    echo "Author:     $GIT_AUTHOR_NAME"
                    echo "Email:      $GIT_AUTHOR_EMAIL"
                    echo "Committer:  $GIT_COMMITTER_NAME"
                '''
                // Tag the Docker image with the commit SHA
                sh "docker build -t nexuscorp/api:${GIT_COMMIT[0..7]} ."
            }
        }
    }
}
```

---

## Part 5: Setting Up Slack Notifications

### Step 1: Create a Slack App and Get a Webhook URL

```
1. Go to: https://api.slack.com/apps
2. Click "Create New App" → "From scratch"
3. App Name: "Jenkins CI"
4. Pick your workspace → Create App

5. Left sidebar → "Incoming Webhooks"
6. Toggle "Activate Incoming Webhooks" → ON
7. Click "Add New Webhook to Workspace"
8. Choose channel: #devops-alerts (or create it)
9. Click "Allow"
10. Copy the Webhook URL:
    https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXX
```

---

### Step 2: Install Slack Plugin in Jenkins

```
Dashboard → Manage Jenkins → Plugins
→ Available plugins
→ Search: "Slack Notification"
→ Check ✅ → Install
→ Restart Jenkins when complete
```

---

### Step 3: Configure Slack in Jenkins

```
Dashboard → Manage Jenkins → Configure System
→ Scroll to "Slack" section

Workspace: your-workspace-name
Default channel: #devops-alerts

Credential:
→ Add → Jenkins
→ Kind: Secret text
→ Secret: https://hooks.slack.com/services/T.../B.../XXXX
→ ID: slack-webhook
→ Add

→ Test Connection
→ "Jenkins Slack Plugin: build 1 started"
→ Check your Slack channel — message should appear

→ Save
```

---

### Step 4: Add Slack Notifications to a Freestyle Job

```
Job Configuration → Post-build Actions
→ Add post-build action → Slack Notifications

Notify on:
  ✅ Start Build
  ✅ Success
  ✅ Failure
  ✅ Back to Normal (after a failure that now passes)
  ✅ Unstable

Channel: #devops-alerts (leave blank to use default)

→ Save → Build Now

Check Slack: you should receive build status messages
```

---

### Step 5: Add Slack to a Jenkinsfile (Pipeline)

```groovy
pipeline {
    agent any

    environment {
        SLACK_CHANNEL = '#devops-alerts'
        APP_NAME = 'nexus-api'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: '#439FE0',
                    message: """
:building_construction: *Build Started*
*Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}
*Branch:* ${env.GIT_BRANCH}
*Commit:* ${env.GIT_COMMIT?.take(8)}
*Author:* ${env.GIT_AUTHOR_NAME}
<${env.BUILD_URL}|View in Jenkins>
                    """.stripIndent()
                )
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m pytest tests/ -v'
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh "docker run -d --name ${APP_NAME} ${APP_NAME}:${BUILD_NUMBER}"
            }
        }
    }

    post {
        success {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: '#2EB67D',   // Green
                message: """
:white_check_mark: *Build Succeeded*
*Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}
*Duration:* ${currentBuild.durationString}
*Branch:* ${env.GIT_BRANCH}
<${env.BUILD_URL}|View in Jenkins>
                """.stripIndent()
            )
        }

        failure {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: '#E01E5A',   // Red
                message: """
:x: *Build FAILED*
*Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}
*Failed Stage:* ${env.STAGE_NAME}
*Branch:* ${env.GIT_BRANCH}
<${env.BUILD_URL}/console|View Console Output>
@here Please investigate!
                """.stripIndent()
            )
        }

        unstable {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: '#ECB22E',   // Yellow
                message: """
:warning: *Build Unstable* (test failures)
*Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}
<${env.BUILD_URL}testReport|View Test Results>
                """.stripIndent()
            )
        }
    }
}
```

---

## Part 6: Other Essential Integrations

### Email Notifications

```
Step 1: Install Email Extension Plugin
  → Search: "Email Extension Plugin" → Install

Step 2: Configure SMTP
  Dashboard → Manage Jenkins → Configure System
  → Extended E-mail Notification
    SMTP server: smtp.gmail.com
    SMTP port: 465
    ✅ Use SSL
    SMTP Username: your-email@gmail.com
    SMTP Password: your-app-password  (not your Gmail password)

  → Default Subject: Build ${BUILD_STATUS}: ${PROJECT_NAME} #${BUILD_NUMBER}
  → Default Content:
    Job: ${PROJECT_NAME}
    Build: #${BUILD_NUMBER}
    Status: ${BUILD_STATUS}
    URL: ${BUILD_URL}
    Console: ${BUILD_URL}console

Step 3: Use in Jenkinsfile
```

```groovy
post {
    failure {
        emailext(
            to: 'devops@nexuscorp.com',
            subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
Build Details:
Job:      ${env.JOB_NAME}
Build:    #${env.BUILD_NUMBER}
Status:   FAILED
Branch:   ${env.GIT_BRANCH}
URL:      ${env.BUILD_URL}

Console Output:
${currentBuild.rawBuild.getLog(50).join('\n')}
            """,
            attachLog: true
        )
    }
}
```

---

### SonarQube Integration (Code Quality)

```groovy
// After installing SonarQube Scanner plugin
stage('Code Quality') {
    steps {
        withSonarQubeEnv('nexus-sonarqube') {
            sh 'sonar-scanner \
                -Dsonar.projectKey=nexus-api \
                -Dsonar.sources=. \
                -Dsonar.python.coverage.reportPaths=coverage.xml'
        }
    }
}

stage('Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

---

## Part 7: Practical Tasks

### Task 1: Plugin Exploration

```
Exercise:
─────────
1. Dashboard → Manage Jenkins → Plugins → Available plugins

2. Search and read the description for each of these:
   □ Multibranch Scan Webhook Trigger
   □ AWS Steps Plugin
   □ Terraform Plugin
   □ HashiCorp Vault Plugin
   □ JIRA Plugin
   □ SonarQube Scanner
   □ Trivy Scanner (container security)
   □ JaCoCo Plugin (Java code coverage)

3. For each plugin, note:
   → What problem does it solve?
   → Which stage of the CI/CD pipeline does it serve?
   → Would NexusCorp use this?

4. Install the plugins you'll use in today's exercises:
   □ git (likely already installed)
   □ github
   □ slack
   □ blueocean
   □ ansicolor
   □ timestamper
   □ workspace-cleanup
   □ credentials-binding
```

---

### Task 2: Integrate Jenkins with Git

```bash
# If you don't have a repo, create one:
mkdir nexus-jenkins-demo
cd nexus-jenkins-demo
git init

# Create a simple app
cat > app.py << 'EOF'
def greet(name):
    return f"Hello, {name}! From NexusCorp."

def add(a, b):
    return a + b

if __name__ == "__main__":
    print(greet("World"))
    print(f"2 + 2 = {add(2, 2)}")
EOF

cat > test_app.py << 'EOF'
from app import greet, add

def test_greet():
    assert "Hello" in greet("Test")
    assert "NexusCorp" in greet("Test")

def test_add():
    assert add(2, 2) == 4
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
EOF

cat > Jenkinsfile << 'EOF'
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Checked out: ${env.GIT_BRANCH} @ ${env.GIT_COMMIT?.take(8)}"
            }
        }
        stage('Test') {
            steps {
                sh 'pip3 install pytest -q'
                sh 'python3 -m pytest test_app.py -v'
            }
        }
    }
    post {
        always {
            echo "Build #${BUILD_NUMBER} finished: ${currentBuild.result}"
        }
    }
}
EOF

git add .
git commit -m "Initial NexusCorp Jenkins demo"

# Push to GitHub
git remote add origin https://github.com/YOUR-USERNAME/nexus-jenkins-demo.git
git push -u origin main
```

**Then in Jenkins:**
```
1. New Item → nexus-jenkins-demo → Pipeline → OK
2. Pipeline → Definition: Pipeline script from SCM
3. SCM: Git
4. Repository URL: https://github.com/YOUR-USERNAME/nexus-jenkins-demo.git
5. Credentials: github-credentials
6. Branch: */main
7. Script Path: Jenkinsfile
8. Save → Build Now
```

---

### Task 3: Notification Setup

```groovy
// After setting up Slack (see Part 5):
// Add this to your nexus-jenkins-demo Jenkinsfile:

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Test') {
            steps {
                sh 'pip3 install pytest -q'
                sh 'python3 -m pytest test_app.py -v'
            }
        }
    }

    post {
        success {
            slackSend(
                color: 'good',
                message: "✅ *${env.JOB_NAME}* #${env.BUILD_NUMBER} passed — ${env.GIT_BRANCH}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "❌ *${env.JOB_NAME}* #${env.BUILD_NUMBER} FAILED — ${env.GIT_BRANCH}\n${env.BUILD_URL}"
            )
        }
    }
}

// Push this update and watch Jenkins build + notify Slack
git add Jenkinsfile
git commit -m "Add Slack notifications to pipeline"
git push
// → Webhook fires → Jenkins builds → Slack message sent
```

---

## 🔧 Mini-Project: NexusCorp Fully Integrated Pipeline

```groovy
// Jenkinsfile — NexusCorp Full Integration Demo
// Requires: Git plugin, Docker Pipeline, Slack, Credentials Binding

pipeline {
    agent any

    environment {
        APP_NAME        = 'nexus-api'
        DOCKER_REGISTRY = 'nexuscorp'
        GIT_SHORT_SHA   = "${env.GIT_COMMIT?.take(8)}"
        IMAGE_TAG       = "${GIT_SHORT_SHA ?: env.BUILD_NUMBER}"
        SLACK_CHANNEL   = '#devops-alerts'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                slackSend channel: env.SLACK_CHANNEL,
                    color: '#439FE0',
                    message: ":building_construction: Build *#${BUILD_NUMBER}* started — `${env.GIT_BRANCH}`"
            }
        }

        stage('Install & Lint') {
            steps {
                sh '''
                    pip3 install -q pytest flake8
                    echo "Running linter..."
                    flake8 . --count --max-line-length=100 \
                           --exclude=.git,__pycache__ || true
                '''
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m pytest test_app.py -v'
            }
            post {
                failure {
                    slackSend channel: env.SLACK_CHANNEL,
                        color: 'danger',
                        message: ":x: Tests failed on build *#${BUILD_NUMBER}*\n${env.BUILD_URL}console"
                }
            }
        }

        stage('Build Docker Image') {
            when { branch 'main' }
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG} ${DOCKER_REGISTRY}/${APP_NAME}:latest"
            }
        }

        stage('Push to Registry') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                        docker push ${DOCKER_REGISTRY}/${APP_NAME}:latest
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            slackSend channel: env.SLACK_CHANNEL,
                color: 'good',
                message: """:white_check_mark: *Build Succeeded*
*Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}
*Duration:* ${currentBuild.durationString}
*Image:* `${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}`"""
        }
        failure {
            slackSend channel: env.SLACK_CHANNEL,
                color: 'danger',
                message: """:x: *Build FAILED*
*Job:* ${env.JOB_NAME} #${env.BUILD_NUMBER}
*Stage:* ${env.STAGE_NAME}
<${env.BUILD_URL}console|View Console> @here"""
        }
        always {
            cleanWs()
        }
    }
}
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Plugin** | An extension that adds new capabilities to Jenkins |
| **Plugin Manager** | Jenkins UI for browsing, installing, updating, and removing plugins |
| **`.jpi` / `.hpi`** | Jenkins plugin file format (Jenkins Plugin Index / Hudson Plugin) |
| **Plugin dependency** | A plugin required by another plugin — installed automatically |
| **Personal Access Token (PAT)** | A GitHub-generated token used instead of a password for API/CI access |
| **Webhook** | An HTTP callback sent by GitHub/GitLab when events occur (e.g., push) |
| **Credential** | A secret (password, token, SSH key) stored securely in Jenkins |
| **Credentials ID** | The reference name used in Jenkinsfiles to use a stored credential |
| **`withCredentials()`** | Jenkinsfile step that injects credentials as environment variables |
| **`checkout scm`** | Declarative step that clones the repo configured in the job's SCM settings |
| **`slackSend()`** | Pipeline step provided by the Slack plugin to post messages |
| **Incoming Webhook** | A Slack feature that accepts HTTP POST requests to post messages |
| **Blue Ocean** | A modern Jenkins UI plugin focused on pipeline visualization |
| **Pipeline Stage View** | Plugin showing pipeline stages as columns in the classic Jenkins UI |
| **AnsiColor** | Plugin that renders ANSI color codes in Jenkins console output |
| **Timestamper** | Plugin that prepends timestamps to every console output line |
| **Email Extension** | Plugin providing configurable email notifications with templates |
| **SonarQube Scanner** | Plugin that runs SonarQube code quality analysis in pipelines |
| **SCM Polling** | Jenkins periodically checking a repository for new commits |
| **`GIT_COMMIT`** | Environment variable containing the full SHA of the current commit |
| **`GIT_BRANCH`** | Environment variable containing the current branch name |
| **Upstream job** | A job whose completion triggers another job |
| **`cleanWs()`** | Pipeline step that removes the workspace at the end of a build |
| **SMTP** | Simple Mail Transfer Protocol — used to send email notifications |
| **App Password** | A Google-generated password for apps that don't support OAuth |
| **`ngrok`** | A tool that creates a public HTTPS tunnel to a local server |

---

## 📚 Resources

- [Jenkins Plugin Index](https://plugins.jenkins.io) — browse all 1,800+ Jenkins plugins
- [Jenkins Managing Plugins — Official Guide](https://www.jenkins.io/doc/book/managing/plugins/) — official plugin management documentation
- [Slack App Setup](https://api.slack.com/messaging/webhooks) — official Slack incoming webhook guide
- [GitHub — Managing Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) — creating GitHub PATs
- [Jenkins Credentials Plugin Docs](https://plugins.jenkins.io/credentials/) — secure credential management
- [Blue Ocean Documentation](https://www.jenkins.io/projects/blueocean/) — modern Jenkins UI guide

---

## 🔭 Day 35 Preview: Jenkins Pipeline as Code — Jenkinsfile Deep Dive

Today you wired Jenkins to the outside world. Tomorrow you write the pipelines that define exactly what Jenkins does when triggered.

You will learn:
- Complete Jenkinsfile syntax — declarative pipeline in depth
- Parallel stages — run tests across multiple environments simultaneously
- Input and approval steps — human gates in automated pipelines
- Shared libraries — reuse pipeline code across multiple repos
- `when` conditions — run stages only on specific branches or events
- Building a complete NexusCorp CI/CD pipeline: test → build → push → deploy

This is the session where Jenkins becomes genuinely production-grade.

---

> 💬 *"Jenkins without plugins is a car without wheels. The plugin ecosystem is what makes Jenkins go anywhere."*
>
> — DevOps Learning By Yukta
