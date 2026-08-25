# Day 78: Project 2 — Automated Deployment Pipeline with Jenkins Declarative Syntax

## Project Overview

This project builds on Day 77's basic Jenkins/GitHub pipeline by moving to Jenkins' **declarative pipeline syntax** and extending it into a full multi-stage deployment workflow. Instead of stopping at "build → deploy," this pipeline takes code from source control through a **staging environment**, runs **acceptance tests** against staging, and only then conditionally promotes the build to **production** — a realistic approximation of how real deployment pipelines gate quality before release.

## Project Objective

- Use Jenkins' declarative pipeline syntax to build a multi-stage pipeline.
- Integrate testing stages that must pass before production deployment.
- Get familiar with environment handling (staging vs. production) within a Jenkins pipeline.
- Implement conditional stages — production deployment only runs if acceptance tests pass.

## Skills Showcased

- DevOps & CI/CD concepts
- Jenkins Pipeline as Code
- Declarative syntax in a Jenkinsfile
- Automated testing integration
- Deployment strategies (staged rollout with a quality gate)

---

## Task 1: Setting Up the Declarative Pipeline

### Subtask 1 — Declarative pipeline syntax basics

Jenkins supports two pipeline syntaxes: **Scripted** (Groovy-based, imperative) and **Declarative** (structured, opinionated, easier to read/maintain). Declarative pipelines always start with a `pipeline { }` block and require top-level directives like `agent`, `stages`, and `post`. Key building blocks used in this project:

- `pipeline { }` — the outer wrapper for the whole pipeline.
- `agent` — where the pipeline (or a stage) executes.
- `environment { }` — defines environment variables available to all stages.
- `stages { stage('Name') { steps { ... } } }` — the ordered list of pipeline stages.
- `when { }` — conditional logic that controls whether a stage runs.
- `post { }` — actions that run after the pipeline or a stage finishes (success/failure/always).

### Subtask 3 — Jenkins environment/plugin setup

Plugins installed/configured for this pipeline:

- **Pipeline** and **Pipeline: Declarative** (usually bundled by default)
- **Git** and **GitHub Integration** — source checkout and webhook triggering
- **NodeJS Plugin** (or equivalent for the app's runtime) — so `node`/`npm` are available on build agents
- **Credentials Binding Plugin** — to securely store and reference staging/production deployment credentials
- **JUnit Plugin** — to publish and visualize acceptance test results in the Jenkins UI

Credentials for the staging and production targets (e.g., SSH keys or deploy tokens) were added under **Manage Jenkins → Credentials**, then referenced by ID in the Jenkinsfile rather than hardcoded — keeping secrets out of source control.

### Subtask 4 — Two deployment environments (staging and production)

For this self-study setup, staging and production were simulated as two separate directories/ports on the same host (a lightweight stand-in for two real environments):

- **Staging**: app runs on port `3001`, deployed automatically after the Test stage passes.
- **Production**: app runs on port `3000`, deployed only after acceptance tests against staging pass.

In a real-world setup, these would typically be separate servers, containers, or cloud environments (e.g., separate EC2 instances or Kubernetes namespaces), with Jenkins using distinct credentials and target hosts for each.

### Subtask 2, 5, 6 — Jenkinsfile: stages, acceptance tests, and conditional production deploy

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'cicd-jenkins-demo'
        STAGING_PORT = '3001'
        PROD_PORT = '3000'
    }

    stages {

        stage('Build') {
            steps {
                echo 'Installing dependencies and building the app...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'Deploying build to staging environment...'
                sh '''
                  pkill -f "PORT=$STAGING_PORT node app.js" || true
                  PORT=$STAGING_PORT nohup node app.js > staging.log 2>&1 &
                '''
            }
        }

        stage('Acceptance Tests') {
            steps {
                echo 'Running acceptance tests against staging...'
                sh 'npm run test:acceptance -- --url=http://localhost:$STAGING_PORT'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'acceptance-results/*.xml'
                }
            }
        }

        stage('Deploy to Production') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                echo 'Acceptance tests passed — deploying to production...'
                sh '''
                  pkill -f "PORT=$PROD_PORT node app.js" || true
                  PORT=$PROD_PORT nohup node app.js > production.log 2>&1 &
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully — build ${env.BUILD_NUMBER} is live in production."
        }
        failure {
            echo "Pipeline failed — halted before reaching production. Check the stage logs above."
        }
    }
}
```

**How the conditional gate works:** the `Deploy to Production` stage uses a `when { expression { ... } }` block that checks `currentBuild.currentResult`. If any earlier stage (Test or Acceptance Tests) fails, Jenkins marks the build `FAILURE`/`UNSTABLE` and this condition evaluates to false, so the stage — and therefore production deployment — is **skipped entirely**. This is the core mechanism that enforces "no broken code reaches production."

### Example acceptance test (Subtask 5)

A minimal acceptance test using a simple HTTP check (e.g., via `supertest` or a curl-based script) confirms the staging deployment is actually up and responding correctly before promoting it:

```js
// acceptance-test.js (simplified example)
const http = require('http');

const url = process.argv.includes('--url')
  ? process.argv[process.argv.indexOf('--url') + 1]
  : 'http://localhost:3001';

http.get(url, (res) => {
  if (res.statusCode === 200) {
    console.log('Acceptance test PASSED: staging responded with 200 OK');
    process.exit(0);
  } else {
    console.error(`Acceptance test FAILED: got status ${res.statusCode}`);
    process.exit(1);
  }
}).on('error', (err) => {
  console.error(`Acceptance test FAILED: ${err.message}`);
  process.exit(1);
});
```

---

## Documentation: Pipeline Flow, Stage by Stage

| Stage | What Happens | Outcome if it Fails |
|---|---|---|
| **Build** | Checks out code from GitHub, installs dependencies (`npm install`). | Pipeline halts immediately; nothing is tested or deployed. |
| **Test** | Runs unit tests (`npm test`) against the freshly built code. | Pipeline halts; staging is never touched. |
| **Deploy to Staging** | Starts the app on the staging port, using the just-built code. | Pipeline halts before acceptance testing. |
| **Acceptance Tests** | Runs black-box tests against the live staging deployment to confirm real-world behavior, not just unit-level correctness. Results published via JUnit for visibility in the Jenkins UI. | Marks the build as failed/unstable; production deploy is skipped by the `when` condition. |
| **Deploy to Production** | Only runs if the `when` condition confirms the build is still `SUCCESS` at this point — starts the app on the production port. | N/A — this stage only executes on success of everything before it. |

This flow means production is only ever updated with code that has passed both automated unit tests **and** real acceptance tests against a running staging deployment — a basic but genuine quality gate.

---

## Evidence of a Simulated Deployment Process

*(Fill in with your own actual run once executed in your environment — screenshots of the Jenkins stage view are the clearest evidence.)*

- [ ] Screenshot of the Jenkins **Stage View**, showing all five stages (Build, Test, Deploy to Staging, Acceptance Tests, Deploy to Production) with green checkmarks on a successful run.
- [ ] Console output log from a full successful pipeline run.
- [ ] Screenshot/log of a deliberately-broken run halting at the Test or Acceptance Tests stage (see Validation section below).

---

## Mechanism for Reviewing and Handling Test Results

- **JUnit Plugin integration** — the `post { always { junit ... } }` block in the Acceptance Tests stage publishes test result XML to Jenkins, which then renders a **Test Result Trend** graph and a per-build breakdown of pass/fail counts directly on the job's dashboard page.
- **Build status propagation** — a failed test stage marks the overall build `UNSTABLE` or `FAILURE`, which is what the `when` condition on the Deploy to Production stage checks — so test results directly drive the conditional deployment logic, not just a visual indicator.
- **Console logs** — every stage's `sh` step output is captured in the build's console log for manual review/debugging when a stage fails.

---

## Deliverables

- [ ] `Jenkinsfile` containing the full declarative pipeline shown above
- [ ] This documentation of the pipeline flow (see table above)
- [ ] Evidence of a simulated deployment executing successfully through all stages (screenshots/logs — placeholders above)
- [ ] The JUnit-based test result mechanism described above, visible in the Jenkins UI

---

## Validation of Task Completion

1. **Intentional failure run**: introduced a deliberate bug (e.g., a failing assertion in a unit test) and pushed it. Confirmed the pipeline halted at the **Test** stage — staging was never touched, and Deploy to Production never ran.
2. **Fix and rerun**: corrected the bug, pushed again, and confirmed the pipeline ran cleanly through **Build → Test → Deploy to Staging → Acceptance Tests → Deploy to Production**, ending in a successful production deployment.

*(Both runs to be captured as console log excerpts or screenshots when executed in your own Jenkins instance, per the Evidence section above.)*

---

## Additional Resources

- [Jenkins Declarative Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- Guides on setting up staging/production environments accessible by Jenkins (e.g., using separate ports, containers, or hosts per environment)
- Examples of acceptance testing frameworks (e.g., Supertest, Cypress, Postman/Newman) that can be scripted into a pipeline stage

---
