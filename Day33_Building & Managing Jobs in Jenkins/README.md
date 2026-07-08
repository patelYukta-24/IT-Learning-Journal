# Day 33: Building & Managing Jobs in Jenkins

> **DevOps Transformation** | CI/CD & Automation Series

---

## 📋 Overview

Yesterday you installed Jenkins and understood CI/CD fundamentals. Today you go hands-on with the core unit of Jenkins: the **Job**.

A Jenkins job is a runnable task — build this code, run these tests, deploy this application. Everything Jenkins does is organized into jobs. Today you create multiple types, understand how Jenkins tracks their history, and learn to read the console output that tells you exactly what happened during a build.

By the end of today you will have created a Freestyle job, a Pipeline job, and a Matrix job, analyzed real console output, and built the muscle memory for navigating the Jenkins UI confidently.

---

## 🗺️ What You'll Do Today

| Topic | What It Covers |
|---|---|
| Jenkins Job Types | Freestyle, Pipeline, Matrix, Maven, External |
| Managing Jobs | Build history, workspaces, console output |
| Task 1 | Create three different job types |
| Task 2 | Freestyle pipeline that prints Hello World |
| Task 3 | Analyze and understand console output |

---

## 🏢 NexusCorp Scenario

> NexusCorp's DevOps lead is setting up Jenkins for three different teams: the backend team needs a Pipeline job with multiple stages, the QA team needs a Matrix job to test their Python library against three versions, and a junior engineer just needs a simple Freestyle job to understand how Jenkins works. Today you build all three.

---

## Part 1: Understanding Jenkins Jobs

### What Is a Jenkins Job?

A **job** (also called a **project**) is the fundamental unit of work in Jenkins. It represents a configurable, repeatable task that Jenkins can trigger, execute, monitor, and report on. Every CI/CD action in Jenkins — building code, running tests, deploying containers — is defined as a job.

```
Jenkins Job Lifecycle:
─────────────────────────────────────────────────────────

Trigger → Queue → Execute → Report → Archive
   │         │        │         │        │
   │         │        │         │        └── Store artifacts,
   │         │        │         │            test results
   │         │        │         └── Pass/Fail notification,
   │         │        │             update GitHub status
   │         │        └── Run build steps in workspace
   │         └── Wait for an available agent
   └── Manual click, SCM change, schedule (cron),
       upstream job completion, API call, webhook
```

---

### Jenkins Job Trigger Types

| Trigger Type | When It Fires | Example |
|---|---|---|
| **Manual** | When you click "Build Now" | Any time you want |
| **SCM Polling** | Jenkins checks Git on a schedule | `H/5 * * * *` = every 5 min |
| **Webhook** | Git push sends HTTP request to Jenkins | Every `git push` |
| **Schedule (cron)** | Time-based, like cron | `0 2 * * *` = 2 AM daily |
| **Upstream** | Another job finishes | After "build" job succeeds |
| **API** | HTTP POST to Jenkins API | From a script or CI tool |
| **Parameterized** | Manual with input values | Choose branch, environment |

---

## Part 2: Types of Jenkins Jobs

### 1. Freestyle Project

The simplest and most flexible job type. Configured entirely through the web UI — no code required. Great for learning Jenkins, running simple scripts, and tasks that don't need complex workflow logic.

```
Freestyle Project:
  ┌─────────────────────────────────────┐
  │  General settings                   │
  │  Source Code Management (Git)       │
  │  Build Triggers (when to run)       │
  │  Build Environment (options)        │
  │  Build Steps (what to run)          │
  │    → Execute shell                  │
  │    → Execute Windows batch          │
  │    → Invoke Maven goals             │
  │    → Execute Docker command         │
  │  Post-build Actions (after run)     │
  │    → Archive artifacts              │
  │    → Send email                     │
  │    → Trigger downstream job         │
  └─────────────────────────────────────┘
```

**Best for:** Simple build scripts, learning Jenkins, deploying scripts, one-off automation tasks.

---

### 2. Pipeline Project

Defines the build workflow as code in a `Jenkinsfile`. Supports complex multi-stage workflows with parallel execution, conditions, and error handling. The professional standard for CI/CD.

```groovy
// Pipeline job — defined in Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Build')  { steps { sh 'make build' } }
        stage('Test')   { steps { sh 'make test' } }
        stage('Deploy') { steps { sh 'make deploy' } }
    }
}
```

**Best for:** Real CI/CD pipelines, multi-stage workflows, anything production-grade.

---

### 3. Multibranch Pipeline

Automatically scans a Git repository and creates a pipeline for each branch that contains a `Jenkinsfile`. When a new branch is created, Jenkins automatically creates a new job for it.

```
Git Repository:
  main          → Pipeline Job (auto-created)
  develop       → Pipeline Job (auto-created)
  feature/login → Pipeline Job (auto-created)
  feature/api   → Pipeline Job (auto-created)
```

**Best for:** Teams following feature branch workflows, where every branch needs its own CI pipeline.

---

### 4. Matrix Project (Multi-Configuration)

Runs the same build across multiple configurations simultaneously. Useful for testing code against multiple operating systems, language versions, or browsers at the same time.

```
Matrix Job: "test-python-library"

                Python 3.9   Python 3.10   Python 3.11
  Ubuntu:          ✅            ✅             ✅
  CentOS:          ✅            ✅             ✅
  Alpine:          ✅            ✅             ✅

= 9 builds running simultaneously
```

**Best for:** Cross-platform testing, multi-version compatibility, browser matrix testing.

---

### 5. Maven Project

A Freestyle project pre-configured for Java Maven builds. Understands `pom.xml`, automatically runs `mvn` goals, and parses Maven output natively.

**Best for:** Java/Maven projects where you want built-in Maven integration without scripting it.

---

### 6. External Job

Monitors a task running outside of Jenkins — records the result and output of an external process in Jenkins' history. Lets you centralize monitoring even for jobs Jenkins doesn't directly execute.

**Best for:** Legacy scripts, external cron jobs, or processes you want to track centrally.

---

### Job Type Comparison

| Job Type | Complexity | Defined Where | Version Control | Best For |
|---|---|---|---|---|
| Freestyle | Low | Jenkins UI | No (UI config) | Learning, simple tasks |
| Pipeline | Medium-High | `Jenkinsfile` in repo | ✅ Yes | All real CI/CD |
| Multibranch Pipeline | Medium | `Jenkinsfile` per branch | ✅ Yes | Branch-based workflows |
| Matrix | Medium | Jenkins UI | No | Cross-platform testing |
| Maven | Low | Jenkins UI + `pom.xml` | Partial | Java/Maven projects |
| External | Low | Jenkins UI | No | Monitoring external jobs |

---

## Part 3: Managing and Monitoring Jobs

### Build History

Every time a job runs, Jenkins creates a **build record** — a numbered entry containing the full execution log, timing, test results, and artifacts.

```
Job: nexus-api-build
  Build History (left sidebar):
  ┌──────────────────────────────┐
  │ #8  ● (blue)  2 min ago     │  ← Passed
  │ #7  ● (blue)  1 hour ago    │  ← Passed
  │ #6  ● (red)   3 hours ago   │  ← Failed
  │ #5  ● (blue)  5 hours ago   │  ← Passed
  │ #4  ● (blue)  Yesterday     │  ← Passed
  └──────────────────────────────┘

Build colors:
  ● Blue    → Build passed (SUCCESS)
  ● Red     → Build failed (FAILURE)
  ● Yellow  → Build unstable (test failures but no script failure)
  ● Grey    → Build aborted or not yet run
  ● Animated → Build currently running
```

**Accessing build details:**
```
Job page → Build History → Click build number (#8)
  → Console Output     (full log of what ran)
  → Test Results       (if JUnit XML published)
  → Artifacts          (if files were archived)
  → Changes            (Git commits in this build)
  → Parameters         (if parameterized build)
  → Timing             (how long each stage took)
```

---

### Workspace

Each job has a **workspace** — a directory on the Jenkins agent where the build runs. Jenkins checks out your code here, runs your build commands here, and files written here are accessible during the build.

```
Default workspace path:
  /var/jenkins_home/workspace/JOB_NAME/

Contents after a Git build:
  /var/jenkins_home/workspace/nexus-api-build/
    ├── app.py
    ├── requirements.txt
    ├── tests/
    ├── Dockerfile
    └── Jenkinsfile
```

**Workspace management:**
```
Job Page → Workspace (left sidebar)
  → Browse files in the workspace
  → Download workspace as ZIP

Jenkins UI → Manage Jenkins → Workspaces
  → See all workspaces across all jobs
  → Delete workspaces to free disk space

# Clean workspace at end of build (best practice)
post {
    always { cleanWs() }
}
```

---

### Console Output

The console output is the real-time log of everything that happened during a build. It is the primary debugging tool in Jenkins — when a build fails, the console output tells you exactly why.

```
Console output for build #6 (FAILED):
─────────────────────────────────────────────────────────────────

Started by user admin
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/nexus-api-build
[Pipeline] { (Declarative: Checkout SCM)
Checking out git https://github.com/nexuscorp/api.git
 > git fetch --tags
 > git checkout main
Commit message: "Add payment gateway integration"
[Pipeline] stage
[Pipeline] { (Install Dependencies)
+ pip3 install -r requirements.txt
Successfully installed flask-3.0.0 redis-5.0.1 psycopg2-binary-2.9.9
[Pipeline] stage
[Pipeline] { (Test)
+ pytest tests/ -v
FAILED tests/test_payment.py::test_charge_card - AssertionError: Expected 200, got 500
                                                                        ^
                                                            BUILD FAILS HERE

ERROR: script returned exit code 1
Finished: FAILURE
```

**Reading console output:**
- Every `+` line shows the exact shell command that ran
- Error messages appear in red in the Jenkins UI
- `Finished: SUCCESS` or `Finished: FAILURE` is the final verdict
- Exit codes are shown for shell commands

**Accessing console output:**
```
Method 1: Job page → Build History → Click #N → Console Output
Method 2: Direct URL: http://jenkins:8080/job/JOB_NAME/BUILD_NUMBER/console
Method 3: Blue Ocean UI → Click failed stage → View logs
Method 4: From terminal (requires Jenkins CLI):
          java -jar jenkins-cli.jar -s http://jenkins:8080 \
               console JOB_NAME BUILD_NUMBER
```

---

### Job Configuration Options

```
Each Jenkins job has these configuration sections:

General:
  → Description
  → Discard old builds (keep last N builds to save disk)
  → This project is parameterized (add input parameters)
  → Disable this project

Source Code Management:
  → None / Git / SVN
  → Repository URL, branch, credentials

Build Triggers:
  → Build periodically (cron schedule)
  → GitHub hook trigger
  → Poll SCM (check for changes on schedule)
  → Build after other projects are built (upstream)
  → Trigger builds remotely (via API)

Build Environment:
  → Delete workspace before build starts
  → Abort build if stuck (timeout)
  → Add timestamps to console output
  → Use secret text/files (inject credentials)

Build Steps:
  → Execute shell
  → Execute Windows batch command
  → Invoke Ant / Maven
  → Send files over SSH
  → Docker Build and Publish

Post-build Actions:
  → Archive the artifacts
  → Publish JUnit test results
  → Send email notification
  → Trigger downstream projects
  → Publish HTML reports
  → Slack notifications
```

---

## Part 4: Practical Tasks

### Task 1: Create Different Job Types

#### Step A — Freestyle Job

```
1. Dashboard → New Item
2. Name: nexus-freestyle-demo
3. Select: Freestyle project → OK

4. General:
   Description: "Basic Freestyle job for NexusCorp demo"
   ✅ Discard old builds → Max builds to keep: 10

5. Build Triggers:
   ✅ Build periodically → Schedule: H * * * *
   (Runs once per hour — remove after testing)

6. Build Steps → Add build step → Execute shell:

#!/bin/bash
echo "======================================="
echo " NexusCorp Freestyle Build"
echo "======================================="
echo "Job:        $JOB_NAME"
echo "Build #:    $BUILD_NUMBER"
echo "Workspace:  $WORKSPACE"
echo "Node:       $NODE_NAME"
echo "Timestamp:  $(date '+%Y-%m-%d %H:%M:%S')"
echo "======================================="

echo ""
echo "System Information:"
echo "  OS:     $(uname -s) $(uname -r)"
echo "  User:   $(whoami)"
echo "  Python: $(python3 --version)"
echo "  Docker: $(docker --version 2>/dev/null || echo 'not available')"

echo ""
echo "Simulating build steps..."
sleep 2
echo "  ✓ Dependencies checked"
sleep 1
echo "  ✓ Code compiled"
sleep 1
echo "  ✓ Tests passed"
sleep 1
echo "  ✓ Artifact created"

echo ""
echo "Build complete!"

7. Post-build Actions:
   → (No post-build actions for now)

8. Save → Build Now
9. Click #1 → Console Output to verify
```

---

#### Step B — Matrix (Multi-Configuration) Job

```
1. Dashboard → New Item
2. Name: nexus-matrix-test
3. Select: Multi-configuration project → OK
   (Note: Requires "Matrix Project" plugin — install if not present)
   Manage Jenkins → Plugins → Available → search "Matrix Project" → Install

4. Configuration Matrix:
   → Add Axis → User-defined Axis
     Name: PYTHON_VERSION
     Values: 3.9 3.10 3.11

   → Add Axis → User-defined Axis
     Name: OS_TYPE
     Values: ubuntu alpine

5. Build Steps → Execute shell:

#!/bin/bash
echo "=============================================="
echo " Matrix Build"
echo " Python Version: $PYTHON_VERSION"
echo " OS Type:        $OS_TYPE"
echo " Combination:    $PYTHON_VERSION on $OS_TYPE"
echo "=============================================="

# Simulate testing for this configuration
echo "Setting up $OS_TYPE environment for Python $PYTHON_VERSION..."
sleep 2

echo "Running tests for Python $PYTHON_VERSION on $OS_TYPE..."
sleep 1

# Simulate occasional test failure for demo
if [ "$PYTHON_VERSION" = "3.9" ] && [ "$OS_TYPE" = "alpine" ]; then
    echo "WARNING: Compatibility issue detected"
    echo "Test suite passed with warnings"
else
    echo "All tests passed for Python $PYTHON_VERSION on $OS_TYPE"
fi

echo "Matrix cell complete: $PYTHON_VERSION/$OS_TYPE"

6. Save → Build Now
   Jenkins runs 6 builds simultaneously (2 OS × 3 Python versions)
   View the matrix result grid on the job page
```

---

#### Step C — Pipeline Job with Two Stages

```
1. Dashboard → New Item
2. Name: nexus-pipeline-demo
3. Select: Pipeline → OK

4. Pipeline section:
   Definition: Pipeline script (paste directly for now)

5. Paste this Jenkinsfile:

pipeline {
    agent any

    environment {
        APP_NAME = 'nexus-demo-app'
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d_%H%M%S',
                              returnStdout: true).trim()
    }

    stages {

        stage('Prepare') {
            steps {
                echo "======================================"
                echo " Stage 1: Prepare"
                echo "======================================"
                echo "App:       ${APP_NAME}"
                echo "Build #:   ${BUILD_NUMBER}"
                echo "Timestamp: ${BUILD_TIMESTAMP}"

                sh '''
                    echo "Checking environment..."
                    python3 --version
                    pip3 --version
                    echo "Environment ready."
                '''
            }
        }

        stage('Build & Test') {
            steps {
                echo "======================================"
                echo " Stage 2: Build and Test"
                echo "======================================"

                sh '''
                    echo "Simulating dependency install..."
                    sleep 2

                    echo "Running unit tests..."
                    python3 -c "
import sys
tests = [
    ('Addition test',       1 + 1 == 2),
    ('String test',         'nexus'.upper() == 'NEXUS'),
    ('List test',           len([1,2,3]) == 3),
    ('Dictionary test',     {'k': 'v'}['k'] == 'v'),
]
passed = 0
failed = 0
for name, result in tests:
    status = 'PASS' if result else 'FAIL'
    print(f'  [{status}] {name}')
    if result:
        passed += 1
    else:
        failed += 1
print(f'\\nResults: {passed} passed, {failed} failed')
sys.exit(0 if failed == 0 else 1)
"
                    echo "All tests passed!"
                '''
            }
        }

        stage('Package') {
            steps {
                echo "======================================"
                echo " Stage 3: Package"
                echo "======================================"

                sh '''
                    echo "Creating build artifact..."
                    mkdir -p build
                    echo "Build: ${BUILD_NUMBER}" > build/manifest.txt
                    echo "App: ${APP_NAME}" >> build/manifest.txt
                    echo "Status: Ready to deploy" >> build/manifest.txt
                    cat build/manifest.txt
                    echo "Artifact created: build/manifest.txt"
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCEEDED — Build #${BUILD_NUMBER} is ready"
        }
        failure {
            echo "Pipeline FAILED — Check console output for details"
        }
        always {
            echo "Pipeline finished with status: ${currentBuild.result}"
        }
    }
}

6. Save → Build Now
7. View in Blue Ocean for the visual stage-by-stage view
```

---

### Task 2: Freestyle Pipeline to Print "Hello World!"

```
1. Dashboard → New Item
2. Name: hello-world
3. Select: Freestyle project → OK

4. Build Steps → Add build step:
   → Execute shell (Linux/macOS)
   → OR Execute Windows batch command (Windows)

5. Enter the command:

# Linux/macOS — Execute shell
echo "Hello World!"
echo "From Jenkins Build #${BUILD_NUMBER}"
echo "Running on: $(hostname)"
echo "At: $(date)"

# Windows — Execute Windows batch command
echo Hello World!
echo From Jenkins Build #%BUILD_NUMBER%

6. Save
7. Click "Build Now"
8. Click #1 → Console Output

Expected console output:
─────────────────────────────────────────────────────────
Started by user admin
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/hello-world
[hello-world] $ /bin/sh -xe /tmp/jenkins-script123.sh
+ echo Hello World!
Hello World!
+ echo From Jenkins Build #1
From Jenkins Build #1
+ echo Running on: 3f2b1c4d5e6a
Running on: 3f2b1c4d5e6a
+ echo At: Mon Jun 10 14:32:01 UTC 2024
At: Mon Jun 10 14:32:01 UTC 2024
Finished: SUCCESS
─────────────────────────────────────────────────────────

9. Build it multiple times:
   → Each gets a new number (#1, #2, #3...)
   → Build history shows all runs
   → Compare output between builds
```

---

### Task 3: Analyze Console Output

Run builds and practice reading the output. Here is what every section means:

```
Console Output Anatomy:
─────────────────────────────────────────────────────────

Started by user admin                    ← Who triggered this build
Running as SYSTEM                        ← Jenkins system user
Building in workspace /var/jenkins_home/ ← Where the build runs
workspace/hello-world

[hello-world] $ /bin/sh -xe /tmp/...    ← Jenkins creates a temp script

The -xe flags mean:
  -x = Print each command before executing (the + lines)
  -e = Exit immediately if any command fails

+ echo Hello World!                      ← The + means: this command ran
Hello World!                             ← This is the command's output

+ python3 -m pytest tests/              ← Another command that ran
FAILED tests/test_api.py::test_login    ← A test failure
ERROR: script returned exit code 1      ← Exit code shows failure

Finished: FAILURE                        ← Final status
─────────────────────────────────────────────────────────

Key things to look for when debugging:
  1. The last + command before the failure = what failed
  2. Exit code: 0 = success, anything else = failure
  3. "ERROR:" lines = explicit errors
  4. "FAILED" = test failure
  5. "Finished: FAILURE" = the build did not succeed
```

**Exercise — trigger and analyze builds:**

```bash
# Trigger builds in different states:

# 1. Successful build — analyze what SUCCESS looks like
# (Build any passing job)

# 2. Failing build — make a job fail on purpose
# In Execute shell, add:
exit 1    # Forces an exit with failure code

# 3. Unstable build — publish test results with failures
# (Requires JUnit XML output — skip for now)

# 4. Aborted build — click "Abort" while a build is running
# Add to shell: sleep 60
# Then click the red X to abort

# For each build, note:
# - What color is the build status ball?
# - Where exactly in the console did it fail?
# - What was the exit code?
# - How long did the build take?
```

---

## 🔧 Mini-Project: NexusCorp Jenkins Job Suite

Build three linked jobs that simulate a real CI workflow:

### Job 1: Code Quality Check

```
Name: nexus-quality-check
Type: Freestyle

Execute shell:
#!/bin/bash
echo "=== NexusCorp Code Quality Check ==="

# Simulate linting
echo "Running linter..."
sleep 2
python3 -c "
issues = []
files = ['app.py', 'models.py', 'utils.py']
for f in files:
    print(f'  Checking {f}... OK')
print(f'Lint complete: 0 issues found in {len(files)} files')
"

# Simulate complexity check
echo ""
echo "Checking code complexity..."
sleep 1
echo "  Average complexity: 3.2 (threshold: 10)"
echo "  All functions within acceptable complexity"

echo ""
echo "Quality check PASSED"
```

---

### Job 2: Unit Tests

```
Name: nexus-unit-tests
Type: Freestyle
Build Triggers: Build after other projects are built
  → Projects to watch: nexus-quality-check
  → Trigger: Trigger only if build is stable

Execute shell:
#!/bin/bash
echo "=== NexusCorp Unit Tests ==="
echo "Triggered by upstream: nexus-quality-check"
echo ""

python3 -c "
import time, random

test_suites = {
    'API Tests': 12,
    'Model Tests': 8,
    'Utility Tests': 15,
    'Integration Tests': 5,
}

total_passed = 0
total_failed = 0

for suite, count in test_suites.items():
    print(f'Running {suite}...')
    time.sleep(1)
    passed = count
    failed = 0
    total_passed += passed
    total_failed += failed
    print(f'  {passed}/{count} passed')

print()
print(f'=== Results: {total_passed} passed, {total_failed} failed ===')
if total_failed > 0:
    exit(1)
"
```

---

### Job 3: Build and Package

```
Name: nexus-build-package
Type: Freestyle
Build Triggers: Build after other projects are built
  → Projects to watch: nexus-unit-tests
  → Trigger: Trigger only if build is stable

Execute shell:
#!/bin/bash
echo "=== NexusCorp Build and Package ==="
echo "Triggered by: nexus-unit-tests"
echo ""

VERSION="1.0.${BUILD_NUMBER}"
ARTIFACT="nexus-app-${VERSION}.tar.gz"

echo "Building version: $VERSION"
sleep 2

# Create a mock artifact
mkdir -p dist
echo "NexusCorp App" > dist/app.py
echo "Version: $VERSION" > dist/version.txt
echo "Build: #$BUILD_NUMBER" >> dist/version.txt
echo "Date: $(date)" >> dist/version.txt

tar czf $ARTIFACT -C dist .
echo ""
echo "Artifact created: $ARTIFACT"
ls -lh $ARTIFACT
echo ""
echo "Build complete! Ready to deploy version $VERSION"

Post-build Actions:
  → Archive the artifacts: *.tar.gz
```

**Test the full chain:**
```
Dashboard → nexus-quality-check → Build Now
  ↓ (if passes, automatically triggers)
nexus-unit-tests
  ↓ (if passes, automatically triggers)
nexus-build-package
  ↓
Archived artifact available for download
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Job / Project** | The fundamental unit of work in Jenkins — a configured, runnable task |
| **Build** | A single execution of a Jenkins job |
| **Build Number** | A sequential integer assigned to each build of a job (starts at #1) |
| **Build History** | The list of all past builds for a job, with their status and timing |
| **Workspace** | The directory on the Jenkins agent where a build executes |
| **Console Output** | The real-time log of everything that happened during a build |
| **Freestyle Project** | A simple, GUI-configured Jenkins job type |
| **Pipeline Project** | A Jenkins job that defines its workflow as code in a Jenkinsfile |
| **Multibranch Pipeline** | Automatically creates pipeline jobs for each branch in a repository |
| **Matrix Project** | Runs the same build across multiple configurations simultaneously |
| **Maven Project** | A Freestyle job pre-configured for Java Maven builds |
| **External Job** | A Jenkins job that monitors a task running outside of Jenkins |
| **Build Trigger** | The condition that causes a job to run (manual, webhook, schedule, upstream) |
| **Upstream Job** | A job whose completion triggers another job |
| **Downstream Job** | A job triggered by the completion of another job |
| **Post-build Action** | An action that runs after a build completes (archive, notify, trigger) |
| **Artifact** | A file produced by a build (JAR, Docker image, ZIP) that can be archived and shared |
| **Exit Code** | The numeric value returned by a shell command (0 = success, non-zero = failure) |
| **Build Status** | The result of a build: SUCCESS, FAILURE, UNSTABLE, ABORTED |
| **Blue ball** | Jenkins icon indicating a successful build |
| **Red ball** | Jenkins icon indicating a failed build |
| **Yellow ball** | Jenkins icon indicating an unstable build (e.g., test failures) |
| **SCM Polling** | Jenkins periodically checking a Git repository for new commits |
| **Axis (Matrix)** | A dimension in a Matrix job — each value becomes a build configuration |
| **`cleanWs()`** | Jenkinsfile step that deletes the workspace after a build |
| **Blue Ocean** | A modern visual pipeline UI plugin for Jenkins |
| **Parameterized Build** | A job that accepts input values before running |
| **`currentBuild.result`** | A Jenkinsfile variable containing the current build's status |

---

## 📚 Resources

- [Jenkins Getting Started — Official Docs](https://www.jenkins.io/doc/book/getting-started/) — official Jenkins guide
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/) — complete Pipeline reference
- [Blue Ocean Plugin](https://www.jenkins.io/projects/blueocean/) — modern Jenkins UI for pipelines
- [Jenkins Matrix Project Plugin](https://plugins.jenkins.io/matrix-project/) — multi-configuration builds
- [Jenkins Job DSL Plugin](https://plugins.jenkins.io/job-dsl/) — define jobs as Groovy code
- [Easiest Jenkins CI/CD Pipeline Tutorial — YouTube](https://www.youtube.com/results?search_query=Jenkins+CICD+Pipeline+Tutorial+DevOps) — video walkthrough referenced in curriculum

---

## 🔭 Day 34 Preview: Jenkins Pipelines — Connecting Git, Docker & Deployment

Today you built jobs manually through the UI. Tomorrow you define everything as code and wire it all together.

You will learn:
- Writing complete `Jenkinsfile`s for real application builds
- Connecting Jenkins to GitHub with webhooks — auto-trigger on `git push`
- Storing secrets safely in Jenkins Credentials store
- Building Docker images and pushing to Docker Hub from a pipeline
- Deploying a container automatically after a successful build
- The complete cycle: `git push` → webhook fires → Jenkins builds → Docker image pushed → container deployed

This is the pipeline that makes CI/CD real — and it uses every skill from Days 1 to 33.

---

> 💬 *"A Jenkins job is a promise: given this code, these steps will always run in this order. The console output is the receipt."*
>
> — DevOps Learning by Yukta
