# Day 54: Your CI/CD Pipeline on AWS — Part II (CodeDeploy & CodePipeline)

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. What is CodeDeploy?](#1-what-is-codedeploy)
- [2. CodeDeploy Tasks](#2-codedeploy-tasks)
- [3. What is CodePipeline?](#3-what-is-codepipeline)
- [4. CodePipeline Tasks](#4-codepipeline-tasks)
- [5. Troubleshooting Notes](#5-troubleshooting-notes)
- [6. Key Learnings](#6-key-learnings)
- [7. Resources](#7-resources)

---

## Overview
Day 54 completes the CI/CD pipeline arc started on Day 53 (CodeCommit + CodeBuild). Today adds **CodeDeploy** (automated deployment to EC2) and **CodePipeline** (the orchestrator that ties CodeCommit → CodeBuild → CodeDeploy into one automated release process triggered by every code change). By the end of this day, a single pipeline should take a commit all the way to a running Nginx page on EC2.

## Topics Covered
- ❖ What is CodeDeploy
- ❖ CodeDeploy Tasks — CodeDeploy agent, `appspec.yml`, deploying `index.html` via Nginx
- ❖ What is CodePipeline
- ❖ CodePipeline Tasks — Deployment Group, full CodeCommit → CodeBuild → CodeDeploy pipeline

---

## 1. What is CodeDeploy?
AWS CodeDeploy automates application deployments to EC2 instances, on-premises servers, Lambda functions, or ECS services. It can pull deployment content from S3, GitHub, Bitbucket, or (as used here) directly from a CodeCommit repository, without requiring changes to existing application code.

### 1.1 Core CodeDeploy Concepts
| Concept | Description |
|---|---|
| **Application** | A logical container/name representing what's being deployed |
| **Deployment Group** | The set of target instances (identified by tags) a deployment goes to |
| **CodeDeploy Agent** | Software installed on each EC2 instance that receives and executes deployment instructions |
| **`appspec.yml`** | A file at the repo root defining what to deploy and which lifecycle hooks (scripts) to run at each stage |
| **Deployment** | A single execution of pushing a specific application revision to a deployment group |

### 1.2 The `appspec.yml` File
`appspec.yml` is to CodeDeploy what `buildspec.yaml` is to CodeBuild — a YAML file that tells CodeDeploy exactly what to do. For an EC2/on-premises deployment, it defines:
- **`files`** — which files from the deployment package go where on the target instance
- **`hooks`** — lifecycle event scripts (e.g., `BeforeInstall`, `AfterInstall`, `ApplicationStart`, `ValidateService`) that run at defined points during the deployment

---

## 2. CodeDeploy Tasks

### Task 1 — Install CodeDeploy Agent, Read `appspec.yml`, Deploy `index.html` via Nginx

**Step 1: Prepare the target EC2 instance**
1. Launch (or reuse) an EC2 instance — Amazon Linux 2, `t2.micro`
2. **Tag it** meaningfully, since CodeDeploy targets instances by tag (e.g., `Key: Name`, `Value: day54-deploy-target`)
3. **Attach an IAM Role** to the instance with the managed policy `AmazonEC2RoleforAWSCodeDeploy` — this lets the agent communicate with the CodeDeploy service

**Step 2: Install the CodeDeploy agent (Amazon Linux 2)**
```bash
sudo yum update -y
sudo yum install -y ruby wget
cd /home/ec2-user
wget https://aws-codedeploy-<region>.s3.<region>.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo systemctl start codedeploy-agent
sudo systemctl enable codedeploy-agent
sudo systemctl status codedeploy-agent
```
> Replace `<region>` with your actual region, e.g. `ap-south-1`.

**Step 3: Install Nginx on the same instance** (the deployment target needs Nginx already present to serve the deployed file)
```bash
sudo amazon-linux-extras install -y nginx1
sudo systemctl enable nginx
sudo systemctl start nginx
```

**Step 4: Create the CodeDeploy Application**
1. CodeDeploy → Applications → Create application
2. Application name: `day54-nginx-app`
3. Compute platform: **EC2/On-premises**
4. Create

**Step 5: Understand `appspec.yml` (to be added to the repo in Task 2)**
```yaml
version: 0.0
os: linux
files:
  - source: /index.html
    destination: /usr/share/nginx/html
hooks:
  AfterInstall:
    - location: scripts/restart_nginx.sh
      timeout: 180
      runas: root
```

**`scripts/restart_nginx.sh`:**
```bash
#!/bin/bash
systemctl restart nginx
```

> Status: reference implementation — pending execution + screenshot of the CodeDeploy agent's `active (running)` status on EC2.

---

### Task 2 — Add `appspec.yml` to CodeCommit and Complete the Deployment

**Step 1: Add `appspec.yml` and the hook script to the Day 53 repo**
```bash
cd day53-cicd-repo
mkdir -p scripts
cat <<'EOF' > appspec.yml
version: 0.0
os: linux
files:
  - source: /index.html
    destination: /usr/share/nginx/html
hooks:
  AfterInstall:
    - location: scripts/restart_nginx.sh
      timeout: 180
      runas: root
EOF

cat <<'EOF' > scripts/restart_nginx.sh
#!/bin/bash
systemctl restart nginx
EOF
chmod +x scripts/restart_nginx.sh

git add appspec.yml scripts/restart_nginx.sh
git commit -m "Add appspec.yml and Nginx restart hook for CodeDeploy"
git push origin main
```

**Step 2: Create a Deployment Group**
1. CodeDeploy → Applications → `day54-nginx-app` → Create deployment group
2. Deployment group name: `day54-deployment-group`
3. Service role: an IAM role with `AWSCodeDeployRole` policy, trusted for `codedeploy.amazonaws.com`
4. Deployment type: **In-place**
5. Environment configuration: **Amazon EC2 instances** → tag key/value matching the instance from Task 1 (e.g., `Name: day54-deploy-target`)
6. Deployment settings: `CodeDeployDefault.AllAtOnce` (fine for a single/small instance count)
7. Load balancer: disable for this simple setup
8. Create deployment group

**Step 3: Create and run a deployment**
1. CodeDeploy → `day54-nginx-app` → Create deployment
2. Deployment group: `day54-deployment-group`
3. Revision location: **My application is stored in AWS CodeCommit** → select the repo and branch/commit
4. Deploy

**Equivalent via AWS CLI:**
```bash
# Create application
aws deploy create-application --application-name day54-nginx-app --compute-platform Server

# Create deployment group
aws deploy create-deployment-group \
  --application-name day54-nginx-app \
  --deployment-group-name day54-deployment-group \
  --service-role-arn arn:aws:iam::<account-id>:role/CodeDeployServiceRole \
  --ec2-tag-filters Key=Name,Value=day54-deploy-target,Type=KEY_AND_VALUE

# Trigger a deployment from CodeCommit
aws deploy create-deployment \
  --application-name day54-nginx-app \
  --deployment-group-name day54-deployment-group \
  --revision revisionType=CodeCommit,gitLocation="{repository=day53-cicd-repo,commitId=<commit-id>}"
```

**Verification:**
1. Browse to `http://<ec2-public-ip>` — should show the Day 53 `index.html` content, now served from `/usr/share/nginx/html`
2. `aws deploy get-deployment --deployment-id <id>` — confirm status `Succeeded`

> Status: reference implementation — pending execution + screenshot of a successful deployment and the deployed page loading in-browser.

---

## 3. What is CodePipeline?
AWS CodePipeline is a continuous delivery orchestration service — it automatically builds, tests, and deploys code every time a defined trigger (like a commit) occurs, based on a release process you define as a series of **stages**. It's the glue that connects CodeCommit, CodeBuild, and CodeDeploy into one automated flow instead of running each service manually in sequence.

### 3.1 Pipeline Stages (for this setup)
```
Source (CodeCommit) ──► Build (CodeBuild) ──► Deploy (CodeDeploy → EC2 Deployment Group)
```
Each stage only proceeds once the previous one succeeds — a failed build, for instance, stops the pipeline before anything reaches the deployment stage.

---

## 4. CodePipeline Tasks

### Task — Deployment Group (already created above) + Full CodeCommit → CodeBuild → CodeDeploy Pipeline

**Step 1: Confirm prerequisites exist**
- CodeCommit repo `day53-cicd-repo` with `buildspec.yaml`, `index.html`, `appspec.yml`, and `scripts/restart_nginx.sh` all committed
- CodeBuild project `day53-nginx-build` (from Day 53)
- CodeDeploy application `day54-nginx-app` and deployment group `day54-deployment-group` (from this day)

**Step 2: Create the Pipeline (console)**
1. CodePipeline → Create pipeline
2. Pipeline name: `day54-cicd-pipeline`
3. Service role: new or existing role with permissions across CodeCommit, CodeBuild, CodeDeploy, and S3 (for artifacts)
4. **Source stage:**
   - Source provider: **AWS CodeCommit**
   - Repository: `day53-cicd-repo`, branch: `main`
   - Change detection: Amazon CloudWatch Events (default — triggers automatically on push)
5. **Build stage:**
   - Build provider: **AWS CodeBuild**
   - Project: `day53-nginx-build`
6. **Deploy stage:**
   - Deploy provider: **AWS CodeDeploy**
   - Application name: `day54-nginx-app`
   - Deployment group: `day54-deployment-group`
7. Review → Create pipeline (this also triggers the first run automatically)

**Equivalent via AWS CLI** (pipeline definitions are more naturally managed as JSON given their nested structure):
```bash
aws codepipeline create-pipeline --cli-input-json file://day54-pipeline.json
```

**`day54-pipeline.json`:**
```json
{
  "pipeline": {
    "name": "day54-cicd-pipeline",
    "roleArn": "arn:aws:iam::<account-id>:role/CodePipelineServiceRole",
    "artifactStore": {
      "type": "S3",
      "location": "day54-pipeline-artifacts-<your-id>"
    },
    "stages": [
      {
        "name": "Source",
        "actions": [{
          "name": "Source",
          "actionTypeId": {"category": "Source", "owner": "AWS", "provider": "CodeCommit", "version": "1"},
          "outputArtifacts": [{"name": "SourceOutput"}],
          "configuration": {"RepositoryName": "day53-cicd-repo", "BranchName": "main"}
        }]
      },
      {
        "name": "Build",
        "actions": [{
          "name": "Build",
          "actionTypeId": {"category": "Build", "owner": "AWS", "provider": "CodeBuild", "version": "1"},
          "inputArtifacts": [{"name": "SourceOutput"}],
          "outputArtifacts": [{"name": "BuildOutput"}],
          "configuration": {"ProjectName": "day53-nginx-build"}
        }]
      },
      {
        "name": "Deploy",
        "actions": [{
          "name": "Deploy",
          "actionTypeId": {"category": "Deploy", "owner": "AWS", "provider": "CodeDeploy", "version": "1"},
          "inputArtifacts": [{"name": "BuildOutput"}],
          "configuration": {"ApplicationName": "day54-nginx-app", "DeploymentGroupName": "day54-deployment-group"}
        }]
      }
    ]
  }
}
```

**Verification:**
1. CodePipeline console → `day54-cicd-pipeline` → confirm all three stages show green/**Succeeded**
2. Push a small change to `index.html` in the repo → confirm the pipeline automatically triggers and re-runs end to end
3. Browse to `http://<ec2-public-ip>` → confirm the updated content appears after the pipeline completes

> Status: reference implementation — pending execution + screenshot of all three pipeline stages succeeding, and of an automatic re-trigger after a fresh commit.

---

## 5. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| Deployment fails at `AfterInstall` | `scripts/restart_nginx.sh` not executable, or wrong path in `appspec.yml` | `chmod +x` the script before committing; verify `hooks` paths match the repo structure |
| CodeDeploy agent not showing as registered | Agent not installed/started, or IAM role missing on the instance | Re-run agent install steps; confirm `AmazonEC2RoleforAWSCodeDeploy` is attached |
| Deployment group finds 0 instances | Tag key/value mismatch between the instance and the deployment group config | Confirm exact tag key/value match, case-sensitive |
| Pipeline's Deploy stage fails immediately | CodeDeploy application/deployment group name typo in pipeline config | Recheck names exactly match what was created in CodeDeploy |
| Pipeline doesn't trigger on new commits | Change detection method not configured, or wrong branch | Confirm CloudWatch Events (or polling) is enabled and branch name matches |
| Nginx doesn't serve the new file after deploy | `destination` path in `appspec.yml` incorrect, or Nginx wasn't restarted | Confirm destination is `/usr/share/nginx/html` and the `AfterInstall` hook actually restarts Nginx |

---

## 6. Key Learnings
- `appspec.yml` and `buildspec.yaml` follow the same underlying pattern — a YAML file defining lifecycle phases/hooks — just for different stages of the pipeline (deploy vs. build).
- CodeDeploy targets EC2 instances by **tags**, not instance IDs directly — this is what makes a deployment group reusable as instances are replaced or scaled.
- CodePipeline is purely an orchestrator — it doesn't do the source control, building, or deploying itself; it coordinates CodeCommit, CodeBuild, and CodeDeploy and automatically re-triggers the whole flow on every new commit.
- With this pipeline complete, the full loop — commit → build → deploy → live on EC2 — now happens automatically, which is the entire point of CI/CD: removing manual steps between "code changed" and "code is live."

## 7. Resources
- [AWS CodeDeploy — User Guide](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)
- [AppSpec File Reference (EC2/On-Premises)](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)
- [Installing the CodeDeploy Agent](https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install.html)
- [AWS CodePipeline — User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)

---
