# Day 84 — Project 8: Deploying a React Application on AWS Elastic Beanstalk using GitHub Actions

**Part XII: Project 8 (Day 84)**
**Curriculum:** NexusCorp DevOps Transformation — Shubham Londhe

---

## ❖ Project Overview

Deploying a React application to AWS Elastic Beanstalk using GitHub Actions involves setting up a CI/CD pipeline that automatically builds and deploys the application to AWS whenever changes are pushed to the codebase. AWS Elastic Beanstalk simplifies the deployment and scaling of applications (provisioning the underlying EC2 instances, load balancer, and environment health monitoring), while GitHub Actions automates the build and deploy workflow around it.

## ❖ Project Objective

- Set up a CI/CD pipeline using GitHub Actions.
- Learn to deploy a React application using AWS Elastic Beanstalk.
- Gain hands-on experience with GitHub repository integration for deployment processes.

## ❖ Skills Showcased

- GitHub Actions for CI/CD Automation
- AWS Elastic Beanstalk for Application Deployment and Management
- React Application Build and Deployment Processes
- YAML Scripting for GitHub Actions Workflow

## ❖ Environment

- **AWS services:** Elastic Beanstalk (Node.js platform, running on EC2), IAM (scoped deploy user), S3 (used internally by Beanstalk for versioned deploy bundles)
- **CI/CD:** GitHub Actions
- **App:** React application, built with `npm run build`, served via Elastic Beanstalk's Node.js platform

---

## ❖ Tasks / Exercises

### Task 1: Deploy a React Application to AWS Elastic Beanstalk Using GitHub Actions

#### Subtask 1 — Obtain the React application source code

Cloned a React application repository locally to confirm the build worked before wiring up the pipeline:

```bash
git clone https://github.com/<username>/react-app.git
cd react-app
npm install
npm run build
```

Confirmed the `build/` folder was generated correctly.

#### Subtask 2 — Set up and configure an Elastic Beanstalk environment

Used the Elastic Beanstalk CLI (`eb`) locally to initialize and create the environment once, so it existed ahead of the automated pipeline:

```bash
pip install awsebcli --upgrade --user
eb init react-app --platform "Node.js 18" --region ap-south-1
eb create react-app-env --single --instance-type t2.micro
```

Notes on configuration choices:
- `--single` runs a single-instance environment (no load balancer) to stay within free-tier bounds for this exercise; a load-balanced environment would be the production-grade choice.
- Platform selected as **Node.js** rather than a static-file platform, since Elastic Beanstalk needs a lightweight Node/Express server to serve the React build output (a plain `server.js` + `express.static` setup was added to the repo for this purpose) — Beanstalk's Node.js platform expects an app entry point, not just static files.

Added a minimal Express server to serve the built React app:

```js
// server.js
const express = require('express');
const path = require('path');
const app = express();
const PORT = process.env.PORT || 8080;

app.use(express.static(path.join(__dirname, 'build')));
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

Verified the environment came up healthy from the initial manual deploy:

```bash
eb status
eb health
eb open
```

`eb status` reported `Health: Green` and `eb open` launched the running app in a browser — confirming the environment itself was correctly provisioned before automating deployments to it.

#### Subtask 3 — Configure a GitHub Actions workflow

Created an IAM user (`github-actions-eb-deployer`) scoped to Elastic Beanstalk and the S3 bucket Beanstalk uses for application versions, and added its keys as GitHub repository secrets (**Settings → Secrets and variables → Actions**):

| Secret name              | Value                     |
|----------------------------|----------------------------|
| `AWS_ACCESS_KEY_ID`        | IAM user's access key      |
| `AWS_SECRET_ACCESS_KEY`    | IAM user's secret key      |
| `AWS_REGION`                | `ap-south-1`               |
| `EB_APPLICATION_NAME`       | `react-app`                 |
| `EB_ENVIRONMENT_NAME`       | `react-app-env`             |

Created `.github/workflows/deploy.yml`:

```yaml
name: Deploy React App to Elastic Beanstalk

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm install

      - name: Build React application
        run: npm run build

      - name: Zip application bundle
        run: zip -r deploy.zip build server.js package.json package-lock.json .ebextensions -x "node_modules/*"

      - name: Deploy to Elastic Beanstalk
        uses: einaregilsson/beanstalk-deploy@v22
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          application_name: ${{ secrets.EB_APPLICATION_NAME }}
          environment_name: ${{ secrets.EB_ENVIRONMENT_NAME }}
          version_label: "v${{ github.run_number }}"
          region: ${{ secrets.AWS_REGION }}
          deployment_package: deploy.zip
```

#### Subtask 4 — Ensure the workflow installs, builds, and deploys

The workflow above covers all three stages end to end:
1. **Install dependencies** — `npm install`
2. **Build the React application** — `npm run build`
3. **Deploy build artifacts to Elastic Beanstalk** — zipping the build output together with the Express server and `package.json`, then using the `einaregilsson/beanstalk-deploy` action to create a new application version and deploy it to the existing environment

Pushed the workflow file and watched it run:

```bash
git add .github/workflows/deploy.yml server.js .ebextensions
git commit -m "Add GitHub Actions workflow for Elastic Beanstalk deployment"
git push origin main
```

All steps completed successfully in the **Actions** tab. The `beanstalk-deploy` step logged the new application version being uploaded and the environment transitioning through `Updating` back to `Ready`/`Green`.

---

## ❖ Validation

- [x] Workflow run completed successfully on push to `main`
- [x] `eb status` / AWS Console shows environment health `Green` after deployment
- [x] New application version (e.g. `v3`, matching the GitHub Actions run number) visible under **Elastic Beanstalk → Application versions**
- [x] Elastic Beanstalk environment URL loads the deployed React app in a browser
- [x] A follow-up commit (small UI text change) triggered an automatic rebuild and redeploy with no manual steps, confirming the CI/CD loop works end-to-end

**Live application URL (Elastic Beanstalk environment):**
```
http://react-app-env.<random-id>.ap-south-1.elasticbeanstalk.com
```

---

## ❖ Deliverables

- [x] **Live React application** running on AWS Elastic Beanstalk
- [x] **GitHub Actions workflow file** — `.github/workflows/deploy.yml`
- [x] **Evidence of successful build/deployment** — GitHub Actions run logs, Elastic Beanstalk environment health and version history
- [x] **URL to the deployed application** on Elastic Beanstalk

---

## ❖ Notes / Learnings

- Elastic Beanstalk's Node.js platform expects a running server process, not raw static files — a small Express wrapper around `express.static` was the missing piece to serve a React `build/` folder correctly under this platform (as opposed to S3 static hosting in Day 79/83, which serves the files directly).
- Scoping the deploy IAM user to Elastic Beanstalk actions (and the S3 bucket Beanstalk manages internally for versions) rather than broad access follows the same least-privilege approach used for the S3/GitHub Actions pipeline on Day 83.
- `--single` instance environments are convenient for learning/cost control, but a real production setup would use a load-balanced, auto-scaled environment — a natural next step building on the ASG/ALB concepts from Days 48–49.
- Using `${{ github.run_number }}` as the version label keeps each deployed version traceable back to the exact CI run that produced it, which made it easy to confirm in the Beanstalk console that a new push actually created a new version rather than silently reusing an old one.
- This was completed as a **self-study lab exercise** on a personal AWS account, following the workflow pattern from the referenced "Effortless Deployment of React App to AWS Elastic Beanstalk" resource — documented here as practice toward the Elastic Beanstalk and GitHub Actions CI/CD skill areas of the curriculum, not a production deployment.

---

## ❖ Cleanup

To avoid ongoing charges after validation:

```bash
eb terminate react-app-env
```

## ❖ Resources

- Effortless Deployment of React App to AWS Elastic Beanstalk
- https://github.com/sitchatt/AWS_Elastic_BeanStalk_On_EC2
- AWS Elastic Beanstalk documentation
- `einaregilsson/beanstalk-deploy` GitHub Action
