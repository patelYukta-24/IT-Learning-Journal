# Day 77: Project 1 — CI/CD Pipeline Automation with Jenkins and GitHub

## Project Overview

CI/CD (Continuous Integration / Continuous Delivery-Deployment) automates the build, test, and deployment stages of software development so code changes reach production faster and more reliably. In this project, I set up an end-to-end pipeline where **Jenkins** automates build/test/deploy, and **GitHub** acts as the source control system that triggers Jenkins automatically on every push, via a webhook.

## Project Objective

- Set up a Jenkins pipeline that automates the build, test, and deployment phases of a web application.
- Trigger the pipeline automatically via a GitHub webhook whenever code is pushed.
- Implement clear stages in a `Jenkinsfile` — clean build, test, deploy.
- Configure notifications for build/deployment status.

## Skills Showcased

- DevOps practices (CI/CD workflow design)
- Jenkins for automation
- GitHub for version control and as a CI trigger
- Writing Jenkinsfiles (Pipeline as Code)
- Implementing and configuring webhooks
- Automation testing frameworks (where applicable)

---

## Task 1: Environment Setup

### Subtask 1 — Install and configure Jenkins

```bash
# Install Java (Jenkins requires it)
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre

# Add Jenkins repo and key
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install -y jenkins

# Start and enable the service
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

- Jenkins runs on port `8080` by default. Get the initial admin password from `/var/lib/jenkins/secrets/initialAdminPassword` to unlock the setup wizard.
- Install the suggested plugins during setup, plus **GitHub Integration**, **Pipeline**, and **Git** plugins explicitly (needed for webhook triggers and Jenkinsfile parsing).
- Create an admin user and set the Jenkins URL.

### Subtask 2 — Create a GitHub repository

Create a new repository for the web application (e.g. `cicd-jenkins-demo`), initialized with a `README.md` and a `.gitignore` for the app's language/runtime.

### Subtask 3 — Write a basic web application

A minimal "Hello World" app keeps the focus on the pipeline logic rather than the app itself.

```js
// app.js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World from Jenkins CI/CD Pipeline!');
});

app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});
```

```json
// package.json
{
  "name": "cicd-jenkins-demo",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"Running placeholder tests...\" && exit 0"
  },
  "dependencies": {
    "express": "^4.19.2"
  }
}
```

### Subtask 4 — Initialize the GitHub repository with the app code

```bash
git init
git add .
git commit -m "Initial commit: basic Hello World app"
git branch -M main
git remote add origin https://github.com/<username>/cicd-jenkins-demo.git
git push -u origin main
```

### Subtask 5 — Add a Jenkinsfile with Build / Test / Deploy stages

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'cicd-jenkins-demo'
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
                echo 'Running tests...'
                sh 'npm test'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                  pkill -f "node app.js" || true
                  nohup node app.js > app.log 2>&1 &
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded — build, test, and deploy all passed.'
        }
        failure {
            echo 'Pipeline failed — check the stage logs above.'
        }
    }
}
```

Commit this `Jenkinsfile` to the repository root:

```bash
git add Jenkinsfile
git commit -m "Add Jenkinsfile with Build, Test, Deploy stages"
git push
```

### Subtask 6 — Configure a GitHub webhook to trigger Jenkins on push

1. In the GitHub repo: **Settings → Webhooks → Add webhook**.
2. **Payload URL**: `http://<jenkins-server-ip>:8080/github-webhook/`
3. **Content type**: `application/json`
4. **Trigger on**: "Just the push event."
5. Save the webhook — GitHub shows a green checkmark once Jenkins responds successfully to the test ping.
6. In Jenkins, on the pipeline job's configuration page: enable **"GitHub hook trigger for GITScm polling"** under Build Triggers, and set the repository URL under Pipeline → SCM.

> If Jenkins is running locally and isn't publicly reachable, GitHub's webhook can't reach it directly — a tunnel like `ngrok` solves this: `ngrok http 8080`, then use the generated public URL as the webhook payload URL instead of a local IP. Worth documenting since it's a common gap in local setups.

---

## Configuring Build/Deployment Notifications

Install the **Email Extension Plugin** and configure SMTP settings under **Manage Jenkins → Configure System**. Then add a post-build notification step to the Jenkinsfile:

```groovy
post {
    success {
        mail to: 'your-email@example.com',
             subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "The pipeline completed successfully."
    }
    failure {
        mail to: 'your-email@example.com',
             subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
             body: "The pipeline failed. Check Jenkins console output for details."
    }
}
```

(Slack notifications are an alternative to email, via the Slack Notification plugin.)

---

## Deliverables

- [ ] GitHub repository containing the web application code and `Jenkinsfile` — link: `https://github.com/<username>/cicd-jenkins-demo`
- [ ] Screenshots or a short video of the setup process and the configured webhook in GitHub (Settings → Webhooks page, showing the green checkmark)
- [ ] This documentation — the setup steps and reasoning behind each configuration choice

*(The repo link, screenshots, and video above are placeholders — fill these in with your own actual links/captures once you've run through the setup, so the documentation reflects real, self-verified work rather than a claim of a hosted deployment.)*

---

## Validation of Task Completion

- Push a change to the GitHub repository and confirm it triggers a new build in Jenkins automatically — visible in **Build History** on the job page, timestamped right after the push.
- Confirm the Jenkins dashboard shows all three defined stages — **Build**, **Test**, **Deploy** — in the stage view, each with a green checkmark on success.

---

## Additional Resources

- [Jenkins Installation Documentation](https://www.jenkins.io/doc/book/installing/)
- [Jenkins Pipeline Syntax Reference](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [GitHub Webhooks Documentation](https://docs.github.com/en/webhooks)

---
