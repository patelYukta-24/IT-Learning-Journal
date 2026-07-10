# Day 36: Jenkins Declarative Pipelines — Deep Dive

> **DevOps Transformation** | CI/CD & Automation Series

---

## 📋 Overview

Over the past four days you have built Jenkins jobs through the UI — Freestyle projects, parameterized builds, Docker integrations. Today you go deeper into the proper way to define pipelines: **Pipeline as Code** using the Declarative Pipeline syntax.

A Declarative Pipeline is a Jenkinsfile stored in your Git repository alongside your application code. It defines every stage, step, and condition in a structured, readable format. When your team reviews a pull request, they review the pipeline too. When the pipeline changes, there is a commit history. When something breaks, you can roll back.

This is how professional DevOps teams use Jenkins — not clicking through UIs, but writing and committing `Jenkinsfile`s.

---

## 🗺️ What You'll Cover Today

| Topic | What It Covers |
|---|---|
| Pipeline as Code | Why Jenkinsfiles beat GUI configuration |
| Declarative vs Scripted | Syntax differences, trade-offs, when to use each |
| Declarative syntax deep dive | All blocks: `pipeline`, `agent`, `stages`, `post`, `when`, `environment`, `options` |
| Parallel stages | Run multiple stages simultaneously |
| Advanced features | Input approval, error handling, shared libraries |
| Tasks | Create, visualize, and deliberately break pipelines |

---

## 🏢 NexusCorp Scenario

> NexusCorp has 12 Jenkins jobs. Each is configured through the UI — nobody knows what they do without clicking into them. When a junior engineer accidentally deleted three jobs, the configurations were gone. The team spent a day recreating them from memory and Slack messages.
>
> The solution: every pipeline must live as a `Jenkinsfile` in the application repository. Configuration is code. Code is version-controlled. Version control means history, rollback, and review. Today you learn how to write those Jenkinsfiles properly.

---

## Part 1: Introduction to Jenkins Pipelines

### What Is a Jenkins Pipeline?

A Jenkins Pipeline is a suite of plugins that supports implementing and integrating continuous delivery pipelines into Jenkins. It provides an extensible set of tools for modeling simple to complex delivery pipelines as code.

The **"as code"** nature means:

```
Traditional Jenkins (GUI):
  Job config lives in: Jenkins internal database
  Version control:     No
  Reviewable:         No (no PR review of pipeline changes)
  Recoverable:        Only if you backed up Jenkins
  Visible:            Only to people with Jenkins access

Pipeline as Code (Jenkinsfile):
  Pipeline lives in:  Your Git repository
  Version control:    Yes — every change is a commit
  Reviewable:         Yes — pipeline changes go through PR review
  Recoverable:        Yes — git revert restores any version
  Visible:            Anyone with repo access can read the pipeline
```

---

### The Jenkinsfile

A `Jenkinsfile` is a plain text file, checked into the root of your repository, that defines the pipeline. Jenkins reads this file when a build starts.

```
Repository structure:
nexus-api/
├── app.py
├── tests/
│   └── test_app.py
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── Jenkinsfile          ← Pipeline definition lives here
```

---

## Part 2: Declarative vs Scripted Pipelines

Both pipeline types are built on **Apache Groovy DSL** (Domain Specific Language) and both run on Jenkins. They differ in syntax, flexibility, and complexity.

### Declarative Pipeline

Uses a structured, opinionated syntax. Every section is predefined. You cannot write arbitrary Groovy code outside of `script {}` blocks.

```groovy
// Declarative Pipeline — starts with 'pipeline' block
pipeline {
    agent any

    environment {
        APP_NAME = 'nexus-api'
        APP_VERSION = '1.0.0'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                sh 'make test'
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'make deploy'
            }
        }
    }

    post {
        success { echo 'Pipeline succeeded!' }
        failure { echo 'Pipeline failed!' }
        always  { cleanWs() }
    }
}
```

---

### Scripted Pipeline

Based on the Groovy scripting language — more flexible, can use full Groovy features, but requires more discipline to keep readable.

```groovy
// Scripted Pipeline — starts with 'node' block
node {
    try {
        stage('Build') {
            sh 'make build'
        }

        stage('Test') {
            sh 'make test'
        }

        // Full Groovy available
        def branches = ['main', 'develop', 'staging']
        for (branch in branches) {
            if (env.BRANCH_NAME == branch) {
                stage("Deploy to ${branch}") {
                    sh "make deploy-${branch}"
                }
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        cleanWs()
    }
}
```

---

### Declarative vs Scripted — Detailed Comparison

| Feature | Declarative | Scripted |
|---|---|---|
| **Starting block** | `pipeline { }` | `node { }` |
| **Structure** | Rigid, predefined sections | Flexible — any Groovy code |
| **Groovy code** | Only in `script { }` blocks | Anywhere |
| **Error handling** | Built-in `post` section | Manual `try/catch/finally` |
| **Linting** | ✅ Built-in linter | ❌ No linter |
| **Parallel stages** | ✅ `parallel { }` block | ✅ `parallel()` method |
| **`when` conditions** | ✅ Built-in | Must implement manually |
| **Learning curve** | Lower | Higher |
| **UI visualization** | ✅ Better (Blue Ocean) | Partial |
| **Recommended for** | Most CI/CD pipelines | Complex custom workflows |
| **Best for** | Beginners and standard pipelines | Advanced users needing full Groovy |

**Recommendation:** Use **Declarative** for all new pipelines. Use `script {}` blocks inside Declarative when you need Groovy logic. Only use Scripted when Declarative genuinely cannot express what you need.

---

## Part 3: Advantages of Declarative Pipelines

### 1. Readable Structure

Every Declarative Pipeline has the same predictable structure. Any engineer can open a `Jenkinsfile` and understand what it does immediately:

```groovy
pipeline {           // The pipeline
    agent any        // Where to run
    environment { }  // Variables
    options { }      // Pipeline settings
    stages {         // The work
        stage('X') {
            steps { }
        }
    }
    post { }        // After-run actions
}
```

### 2. Built-in Linter

Jenkins provides a linter that validates Declarative Pipeline syntax before a build runs. You can use it via:

```
Method 1: Jenkins UI Linter
Dashboard → New Item → Pipeline
→ Paste your Jenkinsfile in the Pipeline Script box
→ Click "Validate" (below the script box)
→ Jenkins reports syntax errors immediately

Method 2: Pipeline Lint via API
curl -X POST \
    http://localhost:8080/pipeline-model-converter/validate \
    -u admin:TOKEN \
    -F "jenkinsfile=<./Jenkinsfile"

Method 3: VS Code Extension
Install: Jenkins Pipeline Linter Connector
→ Lints your Jenkinsfile locally against your Jenkins server
```

### 3. Stage-Based Visualization

Blue Ocean and the classic Stage View both show Declarative Pipeline stages as visual columns — making it immediately clear which stage passed, which failed, and how long each took:

```
Build        Test         Deploy
  ✅           ✅           ❌
 12s          45s         FAILED

Click a stage → See only that stage's logs
Click the red stage → Jump directly to the failure
```

---

## Part 4: Declarative Pipeline — Complete Syntax Reference

### The Skeleton

```groovy
pipeline {
    agent { }          // Required: where to run
    environment { }    // Optional: env vars
    options { }        // Optional: pipeline settings
    parameters { }     // Optional: input parameters
    triggers { }       // Optional: automated triggers
    tools { }          // Optional: tool installations
    stages {           // Required: the work
        stage('name') {
            agent { }          // Optional: override agent per stage
            environment { }    // Optional: stage-level env vars
            options { }        // Optional: stage-level options
            when { }           // Optional: conditional execution
            input { }          // Optional: pause for human input
            steps { }          // Required: what to do
            post { }           // Optional: stage-level post actions
        }
    }
    post { }           // Optional: pipeline-level post actions
}
```

---

### `agent` — Where to Run

```groovy
// Any available agent
agent any

// No agent globally (define per stage)
agent none

// Specific agent by label
agent { label 'linux-docker' }

// Run in a Docker container
agent {
    docker {
        image 'python:3.11-slim'
        label 'docker-capable'
        args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
}

// Build a Dockerfile and use it as the agent
agent {
    dockerfile {
        filename 'Dockerfile.ci'
        dir 'ci'
        additionalBuildArgs '--build-arg VERSION=1.0'
    }
}

// Kubernetes pod agent
agent {
    kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: python
    image: python:3.11-slim
    command: ['sleep', 'infinity']
'''
    }
}
```

---

### `environment` — Variables

```groovy
environment {
    // Static values
    APP_NAME    = 'nexus-api'
    VERSION     = '1.0.0'
    DEPLOY_ENV  = 'staging'

    // From credential store
    DOCKER_CREDS = credentials('dockerhub-credentials')
    // Creates: DOCKER_CREDS_USR (username) and DOCKER_CREDS_PSW (password)

    // Computed values
    IMAGE_TAG   = "${BUILD_NUMBER}-${GIT_COMMIT[0..7]}"
    FULL_IMAGE  = "nexuscorp/${APP_NAME}:${IMAGE_TAG}"
}
```

---

### `options` — Pipeline Settings

```groovy
options {
    // Abort the build if it runs longer than N minutes
    timeout(time: 30, unit: 'MINUTES')

    // Keep only last N builds
    buildDiscarder(logRotator(numToKeepStr: '10'))

    // Add timestamps to all console output
    timestamps()

    // Colorize console output (requires AnsiColor plugin)
    ansiColor('xterm')

    // Don't allow concurrent builds of this job
    disableConcurrentBuilds()

    // Retry the entire pipeline on failure
    retry(2)

    // Skip the default checkout (do it manually in a stage)
    skipDefaultCheckout()

    // Only build one branch at a time (for Multibranch)
    throttleJobProperty(
        categories: [],
        limitOneJobWithMatchingParams: false,
        maxConcurrentPerNode: 1,
        maxConcurrentTotal: 1,
        paramsToUseForLimit: '',
        throttleEnabled: true,
        throttleOption: 'project'
    )
}
```

---

### `parameters` — Build Parameters

```groovy
parameters {
    string(
        name: 'DEPLOY_ENV',
        defaultValue: 'staging',
        description: 'Target deployment environment'
    )
    choice(
        name: 'LOG_LEVEL',
        choices: ['INFO', 'DEBUG', 'WARNING', 'ERROR'],
        description: 'Application log level'
    )
    booleanParam(
        name: 'RUN_INTEGRATION_TESTS',
        defaultValue: true,
        description: 'Whether to run integration tests'
    )
    password(
        name: 'DEPLOY_TOKEN',
        defaultValue: '',
        description: 'Deployment authorization token'
    )
    text(
        name: 'CHANGELOG',
        defaultValue: '',
        description: 'Optional release notes for this build'
    )
}

// Access in steps:
// ${params.DEPLOY_ENV}
// ${params.LOG_LEVEL}
// ${params.RUN_INTEGRATION_TESTS}
```

---

### `triggers` — Automated Triggers

```groovy
triggers {
    // Poll SCM every 5 minutes
    pollSCM('H/5 * * * *')

    // Cron schedule — run every day at 2 AM
    cron('H 2 * * *')

    // GitHub hook (requires GitHub Integration plugin + webhook configured)
    githubPush()

    // Trigger when upstream job completes
    upstream(upstreamProjects: 'nexus-build', threshold: hudson.model.Result.SUCCESS)
}
```

---

### `when` — Conditional Stage Execution

```groovy
stage('Deploy to Production') {
    when {
        // Only on main branch
        branch 'main'
    }
    steps { sh 'deploy to prod' }
}

stage('Run Integration Tests') {
    when {
        // Only when parameter is true
        expression { return params.RUN_INTEGRATION_TESTS == true }
    }
    steps { sh 'pytest integration/' }
}

stage('Security Scan') {
    when {
        // Only on pull requests
        changeRequest()
    }
    steps { sh 'trivy image myapp:latest' }
}

stage('Release') {
    when {
        // Only on tags matching v*
        tag "v*"
    }
    steps { sh 'make release' }
}

stage('Deploy') {
    when {
        // AND condition — both must be true
        allOf {
            branch 'main'
            expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
        }
    }
    steps { sh 'deploy' }
}

stage('Notify') {
    when {
        // OR condition — at least one must be true
        anyOf {
            branch 'main'
            branch 'develop'
        }
    }
    steps { sh 'notify team' }
}

stage('Skip if Already Built') {
    when {
        // NOT condition
        not { changelog '.*\\[skip ci\\].*' }
    }
    steps { sh 'run build' }
}
```

---

### `post` — Post-Build Actions

```groovy
post {
    // Always runs — cleanup, notifications
    always {
        cleanWs()
        echo "Build #${BUILD_NUMBER} completed"
    }

    // Only on success
    success {
        echo "Build passed!"
        slackSend color: 'good', message: "✅ Build #${BUILD_NUMBER} passed"
        archiveArtifacts artifacts: 'dist/**', fingerprint: true
    }

    // Only on failure
    failure {
        echo "Build FAILED"
        slackSend color: 'danger', message: "❌ Build #${BUILD_NUMBER} FAILED"
        emailext to: 'team@nexuscorp.com',
            subject: "FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
            body: "See: ${BUILD_URL}"
    }

    // Only when result changed (e.g., from failure back to success)
    changed {
        echo "Build result changed from previous run"
        slackSend message: "Build result changed: ${currentBuild.result}"
    }

    // Only when tests fail but pipeline doesn't exit
    unstable {
        echo "Tests have failures"
    }

    // Only when build was aborted
    aborted {
        echo "Build was manually aborted"
    }
}
```

---

### Parallel Stages

```groovy
stage('Test') {
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'pytest tests/unit -v'
            }
        }
        stage('Integration Tests') {
            steps {
                sh 'pytest tests/integration -v'
            }
        }
        stage('Security Scan') {
            steps {
                sh 'trivy fs . --exit-code 0'
            }
        }
        stage('Lint') {
            steps {
                sh 'flake8 . --max-line-length=100'
            }
        }
    }
}

// All four stages run simultaneously — total time = slowest single stage
```

---

### `input` — Human Approval Gates

```groovy
stage('Deploy to Production') {
    when { branch 'main' }
    input {
        message "Deploy build #${BUILD_NUMBER} to PRODUCTION?"
        ok "Yes, deploy now"
        submitter "tech-lead,devops-team"    // Only these users/groups can approve
        parameters {
            choice(
                name: 'DEPLOY_REGION',
                choices: ['us-east-1', 'ap-south-1', 'eu-west-1'],
                description: 'Which region to deploy to?'
            )
        }
    }
    steps {
        echo "Deploying to ${DEPLOY_REGION}..."
        sh "deploy.sh --region ${DEPLOY_REGION}"
    }
}

// Build pauses at this stage until an authorized user clicks "Approve"
// If nobody approves within the timeout, the build times out
```

---

### `script` — Groovy Code in Declarative

When you need Groovy logic inside a Declarative pipeline, use a `script {}` block:

```groovy
stage('Dynamic Logic') {
    steps {
        script {
            // Full Groovy available inside script block
            def environments = ['dev', 'staging', 'prod']
            def targetEnv = environments.find { it == params.DEPLOY_ENV }

            if (!targetEnv) {
                error("Invalid environment: ${params.DEPLOY_ENV}")
            }

            // Loop
            for (int i = 0; i < 3; i++) {
                echo "Attempt ${i + 1}"
            }

            // Map
            def config = [
                'dev':     'http://dev.nexuscorp.com',
                'staging': 'http://staging.nexuscorp.com',
                'prod':    'https://nexuscorp.com'
            ]

            env.DEPLOY_URL = config[params.DEPLOY_ENV]
        }
        echo "Deploying to: ${env.DEPLOY_URL}"
    }
}
```

---

## Part 5: Creating Your First Declarative Pipeline

### Task 1: Basic Declarative Pipeline

**Step 1: Create the Jenkinsfile**

```groovy
// Jenkinsfile — NexusCorp Basic Declarative Pipeline
pipeline {
    agent any

    environment {
        APP_NAME    = 'nexus-demo'
        BUILD_TIME  = sh(script: 'date "+%Y-%m-%d %H:%M:%S"', returnStdout: true).trim()
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 15, unit: 'MINUTES')
    }

    stages {

        stage('Initialize') {
            steps {
                echo """
╔══════════════════════════════════════╗
║   NexusCorp Declarative Pipeline     ║
╠══════════════════════════════════════╣
║ App:     ${APP_NAME}
║ Build:   #${BUILD_NUMBER}
║ Branch:  ${env.GIT_BRANCH ?: 'local'}
║ Time:    ${BUILD_TIME}
╚══════════════════════════════════════╝
                """
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "=== System Information ==="
                    echo "  OS:        $(uname -s) $(uname -r)"
                    echo "  Hostname:  $(hostname)"
                    echo "  User:      $(whoami)"
                    echo "  Python:    $(python3 --version)"
                    echo "  Docker:    $(docker --version 2>/dev/null || echo 'not installed')"
                    echo "  Workspace: ${WORKSPACE}"
                '''
            }
        }

        stage('Checkout Code') {
            steps {
                echo "Checking out source code..."
                // For jobs linked to SCM, use: checkout scm
                // For demo without SCM:
                sh '''
                    echo "Creating sample project structure..."
                    mkdir -p src tests
                    cat > src/app.py << 'EOF'
def greet(name):
    return f"Hello, {name}! Welcome to NexusCorp."

def calculate(a, b, operation="add"):
    ops = {
        "add": a + b,
        "subtract": a - b,
        "multiply": a * b,
    }
    return ops.get(operation, 0)

if __name__ == "__main__":
    print(greet("World"))
    print(f"2 + 3 = {calculate(2, 3)}")
EOF
                    cat > tests/test_app.py << 'EOF'
import sys
sys.path.insert(0, 'src')
from app import greet, calculate

def test_greet():
    result = greet("NexusCorp")
    assert "Hello" in result
    assert "NexusCorp" in result

def test_calculate_add():
    assert calculate(2, 3) == 5

def test_calculate_subtract():
    assert calculate(10, 4, "subtract") == 6

def test_calculate_multiply():
    assert calculate(3, 4, "multiply") == 12
EOF
                    echo "Project structure created:"
                    find . -name "*.py" | sort
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    echo "Installing dependencies..."
                    pip3 install -q pytest
                    echo "Dependencies installed."
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    echo "Running test suite..."
                    python3 -m pytest tests/ -v --tb=short
                '''
            }
        }

        stage('Build Artifact') {
            steps {
                sh '''
                    echo "Building artifact..."
                    mkdir -p dist
                    cp src/app.py dist/
                    echo "Version: ${BUILD_NUMBER}" > dist/version.txt
                    echo "Built: ${BUILD_TIME}" >> dist/version.txt
                    tar czf nexus-app-${BUILD_NUMBER}.tar.gz -C dist .
                    echo "Artifact created: nexus-app-${BUILD_NUMBER}.tar.gz"
                    ls -lh nexus-app-${BUILD_NUMBER}.tar.gz
                '''
            }
        }

        stage('Report') {
            steps {
                sh '''
                    echo ""
                    echo "=== Build Summary ==="
                    echo "  Status:   PASSED"
                    echo "  Artifact: nexus-app-${BUILD_NUMBER}.tar.gz"
                    echo "  Build:    #${BUILD_NUMBER}"
                    echo "  Time:     ${BUILD_TIME}"
                    echo "===================="
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded — Build #${BUILD_NUMBER}"
            archiveArtifacts artifacts: '*.tar.gz', fingerprint: true
        }
        failure {
            echo "❌ Pipeline failed — Build #${BUILD_NUMBER}"
        }
        always {
            echo "Pipeline finished. Cleaning workspace..."
            cleanWs()
        }
    }
}
```

**Step 2: Create Pipeline job in Jenkins**

```
Dashboard → New Item
Name: nexus-declarative-demo
Type: Pipeline → OK

Pipeline:
  Definition: Pipeline script
  Script: [paste the Jenkinsfile above]

→ Save → Build Now
```

---

### Task 2: Pipeline Visualization with Blue Ocean

```
Step 1: Open Blue Ocean
  Dashboard → Open Blue Ocean (left sidebar)
  Or: http://localhost:8080/blue

Step 2: Find your job
  → nexus-declarative-demo
  → Click the latest build

Step 3: Observe the visual pipeline
  ┌──────────┐  ┌─────────────┐  ┌───────┐  ┌──────────┐  ┌────────┐  ┌────────┐
  │Initialize│→ │ Env Check   │→ │Checkout│→ │ Install  │→ │  Test  │→ │  Build │
  │  ✅ 2s  │  │  ✅ 3s      │  │ ✅ 5s │  │  ✅ 8s  │  │ ✅ 12s │  │ ✅ 5s │
  └──────────┘  └─────────────┘  └───────┘  └──────────┘  └────────┘  └────────┘

Step 4: Click any stage
  → See only that stage's log output
  → Much cleaner than full console output

Step 5: Click the Tests tab (if JUnit results published)
  → See individual test pass/fail results
```

---

### Task 3: Introduce Errors and Use the Linter

**Exercise 3a: Syntax errors the linter catches**

```groovy
// BROKEN Jenkinsfile — missing 'steps' block
pipeline {
    agent any
    stages {
        stage('Build') {
            // ERROR: 'steps' block is missing
            sh 'echo hello'   // Can't put steps directly in stage
        }
    }
}

// Use the linter:
// Job → Pipeline section → Validate → Shows the error immediately
// Error: "steps" block is required inside a stage
```

```groovy
// BROKEN — invalid block inside pipeline
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo hello'
            }
        }
    }
    // ERROR: random Groovy code not allowed outside script blocks
    def x = 5    // This is NOT allowed in Declarative
    echo x
}
```

```groovy
// BROKEN — wrong nesting
pipeline {
    stages {     // ERROR: 'agent' is required before 'stages'
        stage('Build') {
            steps {
                sh 'echo hello'
            }
        }
    }
}
```

**Exercise 3b: Logic errors the linter won't catch**

```groovy
// VALID SYNTAX — but logic error (test will fail)
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh '''
                    python3 -c "
assert 1 + 1 == 3, 'Math is broken!'  # This will fail
print('Tests passed!')
"
                '''
            }
        }
    }
}

// Pipeline will start but fail at the Test stage
// Console output shows: AssertionError: Math is broken!
```

```groovy
// VALID SYNTAX — but missing command
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'nonexistent_command --version'  // Command not found
            }
        }
    }
}

// Linter passes, but runtime fails with:
// /bin/sh: nonexistent_command: command not found
// exit code 127
```

---

## Part 6: Advanced Declarative Pipeline Patterns

### Complete Production Pipeline

```groovy
// Jenkinsfile — NexusCorp Production Pipeline
// Full example combining all Declarative features

pipeline {
    agent none    // Define agent per stage for flexibility

    environment {
        APP_NAME      = 'nexus-api'
        DOCKER_REPO   = 'nexuscorp/nexus-api'
        SLACK_CHANNEL = '#devops-alerts'
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['staging', 'production'],
            description: 'Target environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip tests (emergency deploys only)'
        )
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {

        stage('Checkout') {
            agent any
            steps {
                checkout scm
                script {
                    env.GIT_SHORT_SHA = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = "${BUILD_NUMBER}-${GIT_SHORT_SHA}"
                }
                echo "Branch: ${GIT_BRANCH} | Commit: ${GIT_SHORT_SHA}"
            }
        }

        stage('Parallel Quality Gates') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker { image 'python:3.11-slim' }
                    }
                    steps {
                        sh '''
                            pip install -q pytest
                            pytest tests/unit -v --tb=short
                        '''
                    }
                }

                stage('Lint') {
                    agent {
                        docker { image 'python:3.11-slim' }
                    }
                    steps {
                        sh '''
                            pip install -q flake8
                            flake8 . --max-line-length=100 --exclude=.git
                        '''
                    }
                }

                stage('Security Scan') {
                    agent any
                    steps {
                        sh 'echo "Security scan complete (trivy would run here)"'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            agent any
            steps {
                sh """
                    docker build \
                        --label git.commit=${GIT_SHORT_SHA} \
                        --label build.number=${BUILD_NUMBER} \
                        -t ${DOCKER_REPO}:${IMAGE_TAG} \
                        -t ${DOCKER_REPO}:latest \
                        .
                """
                echo "Built: ${DOCKER_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Integration Tests') {
            agent any
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            steps {
                sh """
                    docker run -d \
                        --name nexus-integration-${BUILD_NUMBER} \
                        -p 5099:5000 \
                        ${DOCKER_REPO}:${IMAGE_TAG}

                    sleep 8

                    curl -f http://localhost:5099/health || exit 1
                    echo "Integration tests passed!"
                """
            }
            post {
                always {
                    sh "docker rm -f nexus-integration-${BUILD_NUMBER} 2>/dev/null || true"
                }
            }
        }

        stage('Push to Registry') {
            agent any
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}
                        docker push ${DOCKER_REPO}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            agent any
            when {
                allOf {
                    branch 'main'
                    expression { return params.DEPLOY_ENV == 'staging' ||
                                        params.DEPLOY_ENV == 'production' }
                }
            }
            steps {
                sh """
                    docker stop nexus-staging 2>/dev/null || true
                    docker rm nexus-staging 2>/dev/null || true
                    docker run -d \
                        --name nexus-staging \
                        --restart unless-stopped \
                        -p 5000:5000 \
                        -e APP_ENV=staging \
                        ${DOCKER_REPO}:${IMAGE_TAG}
                    sleep 8
                    curl -f http://localhost:5000/health
                    echo "Staging deployment complete!"
                """
            }
        }

        stage('Approval: Deploy to Production') {
            when {
                allOf {
                    branch 'main'
                    expression { return params.DEPLOY_ENV == 'production' }
                }
            }
            input {
                message "Deploy ${IMAGE_TAG} to PRODUCTION?"
                ok "Approve and Deploy"
                submitter "tech-lead,devops-lead"
            }
            steps {
                echo "Production deployment approved!"
            }
        }

        stage('Deploy to Production') {
            agent any
            when {
                allOf {
                    branch 'main'
                    expression { return params.DEPLOY_ENV == 'production' }
                }
            }
            steps {
                sh """
                    echo "Deploying ${IMAGE_TAG} to production..."
                    docker stop nexus-production 2>/dev/null || true
                    docker rm nexus-production 2>/dev/null || true
                    docker run -d \
                        --name nexus-production \
                        --restart always \
                        -p 80:5000 \
                        -e APP_ENV=production \
                        ${DOCKER_REPO}:${IMAGE_TAG}
                    sleep 10
                    curl -f http://localhost:80/health
                    echo "Production deployment complete!"
                """
            }
        }
    }

    post {
        success {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: 'good',
                message: "✅ *${APP_NAME}* #${BUILD_NUMBER} deployed to *${params.DEPLOY_ENV}*\n`${DOCKER_REPO}:${env.IMAGE_TAG}`"
            )
        }
        failure {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: 'danger',
                message: "❌ *${APP_NAME}* #${BUILD_NUMBER} FAILED on `${GIT_BRANCH}`\n${BUILD_URL}"
            )
        }
        always {
            sh 'docker image prune -f || true'
            cleanWs()
        }
    }
}
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Declarative Pipeline** | A structured Jenkinsfile syntax using a `pipeline {}` block with predefined sections |
| **Scripted Pipeline** | A flexible Groovy-based Jenkinsfile using a `node {}` block |
| **Groovy DSL** | Domain Specific Language based on Groovy used to write both pipeline types |
| **`pipeline {}`** | The root block of a Declarative Pipeline |
| **`agent`** | Specifies where the pipeline or stage runs (any, specific label, Docker, Kubernetes) |
| **`stages {}`** | Contains the ordered list of stages in the pipeline |
| **`stage('name')`** | A named logical group of steps — appears as a column in UI |
| **`steps {}`** | The actual commands and actions executed within a stage |
| **`environment {}`** | Defines environment variables available in the pipeline or stage |
| **`options {}`** | Configuration for pipeline behavior (timeout, log rotation, etc.) |
| **`parameters {}`** | Defines input parameters that can be provided before a build |
| **`triggers {}`** | Defines automated build triggers (cron, pollSCM, GitHub push) |
| **`when {}`** | Conditional block — stage only runs when condition is true |
| **`post {}`** | Actions that run after the pipeline or stage completes |
| **`parallel {}`** | Runs multiple stages simultaneously |
| **`input {}`** | Pauses pipeline execution awaiting human approval |
| **`script {}`** | Allows arbitrary Groovy code inside a Declarative Pipeline |
| **`always`** | Post condition that runs regardless of build result |
| **`success`** | Post condition that only runs when build succeeded |
| **`failure`** | Post condition that only runs when build failed |
| **`changed`** | Post condition that runs when result differs from previous build |
| **`unstable`** | Post condition for builds with test failures but no script failures |
| **`branch 'name'`** | `when` condition — runs only on a specific branch |
| **`expression {}`** | `when` condition using a Groovy boolean expression |
| **`allOf {}`** | `when` condition — all nested conditions must be true |
| **`anyOf {}`** | `when` condition — at least one nested condition must be true |
| **`not {}`** | `when` condition — inverts the nested condition |
| **`cleanWs()`** | Deletes the workspace at the end of a build |
| **`archiveArtifacts`** | Saves build outputs in Jenkins for download later |
| **Linter** | Built-in Jenkins tool that validates Declarative Pipeline syntax before running |
| **Blue Ocean** | Modern Jenkins UI plugin that visualizes pipelines as stage columns |
| **`buildDiscarder`** | Option to limit how many old builds Jenkins retains |
| **`disableConcurrentBuilds`** | Prevents multiple builds of the same job from running simultaneously |

---

## 📚 Resources

- [Jenkins Declarative Pipeline Syntax — Official Docs](https://www.jenkins.io/doc/book/pipeline/syntax/) — complete reference for all Declarative Pipeline directives
- [Jenkins Pipeline Steps Reference](https://www.jenkins.io/doc/pipeline/steps/) — every available step documented
- [Declarative vs Scripted Pipeline — Official Docs](https://www.jenkins.io/doc/book/pipeline/#declarative-versus-scripted-pipeline-syntax) — official comparison
- [Apache Groovy Documentation](https://groovy-lang.org/documentation.html) — Groovy language reference for `script {}` blocks
- [Jenkins Declarative CI/CD Pipeline — YouTube](https://www.youtube.com/results?search_query=Jenkins+Declarative+Pipeline+DevOps+Live+Project) — video walkthrough referenced in curriculum
- [Blue Ocean Documentation](https://www.jenkins.io/projects/blueocean/) — modern pipeline visualization guide

---

## 🔭 Day 37 Preview: Introduction to AWS Cloud

Today you mastered Pipeline as Code with Jenkins. The next chapter moves from CI/CD tooling into cloud infrastructure.

You will learn:
- What AWS is and how it is structured (regions, availability zones, services)
- The AWS Free Tier and what you can explore without cost
- Core AWS services overview: EC2, S3, IAM, VPC, RDS, Lambda
- Setting up your AWS account and CLI
- How everything you've built — Linux, Docker, Jenkins — runs on AWS infrastructure

The Jenkins pipelines you write going forward will deploy to AWS. The Docker images you build will be stored in AWS ECR. Understanding AWS is the next essential layer.

---

> 💬 *"A Declarative Pipeline is a living document. It defines not just what the pipeline does, but what the team agreed it should do. Commit it, review it, and improve it like any other code."*
>
> — DevOps Learning By Yukta
