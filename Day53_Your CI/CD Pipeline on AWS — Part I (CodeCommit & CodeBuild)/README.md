# Day 53: Your CI/CD Pipeline on AWS — Part I (CodeCommit & CodeBuild)

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. What is CodeCommit?](#1-what-is-codecommit)
- [2. CodeCommit Tasks](#2-codecommit-tasks)
- [3. What is CodeBuild?](#3-what-is-codebuild)
- [4. CodeBuild Tasks](#4-codebuild-tasks)
- [5. Troubleshooting Notes](#5-troubleshooting-notes)
- [6. Key Learnings](#6-key-learnings)
- [7. Resources](#7-resources)

---

## Overview
Day 53 begins a multi-day build toward a full CI/CD pipeline on AWS, using **CodeCommit, CodeBuild, CodeDeploy, CodePipeline, and S3**. Part I covers the first two pieces: **CodeCommit** (source control) and **CodeBuild** (build automation) — setting up a Git repository natively in AWS, then defining a build process for it via a `buildspec.yaml`.

## Topics Covered
- ❖ What is CodeCommit
- ❖ CodeCommit Tasks — repo creation, Git credentials, clone/commit/push
- ❖ What is CodeBuild
- ❖ CodeBuild Tasks — buildspec file, building `index.html` with Nginx

---

## 1. What is CodeCommit?
AWS CodeCommit is a fully managed Git-based source control service. It lets you store, version, and collaborate on source code entirely within AWS, without hosting or managing your own Git servers.

### 1.1 Why Use CodeCommit (vs. GitHub/GitLab)
- Fully integrated with IAM — access control uses the same users/roles/policies as the rest of your AWS account
- No separate hosting or infrastructure to manage
- Native integration with the rest of the CI/CD toolchain (CodeBuild, CodeDeploy, CodePipeline) covered over these next few days
- Encryption at rest, audit logging via CloudTrail — useful where compliance/traceability matters

### 1.2 Authentication Options
CodeCommit supports two ways to authenticate over HTTPS from a local machine:
| Method | Description |
|---|---|
| **Git credentials (IAM-generated)** | A username/password pair generated per IAM user, specifically for Git operations — used in this task |
| **AWS CLI credential helper** | Uses your existing AWS CLI configuration (Access Key/Secret) to authenticate Git operations automatically |

---

## 2. CodeCommit Tasks

### Task 1 — Create Repository + Set Up IAM Git Credentials

**Step 1: Create the CodeCommit repository**
1. CodeCommit → Repositories → Create repository
2. Repository name: `day53-cicd-repo`
3. Description (optional): "Day 53 CI/CD pipeline practice repo"
4. Create

**Step 2: Set up Git credentials in IAM**
1. IAM → Users → select your user → **Security credentials** tab
2. Scroll to **HTTPS Git credentials for AWS CodeCommit**
3. Click **Generate credentials**
4. **Download the `.csv` immediately** — the password is shown only once
5. Ensure your IAM user has the managed policy `AWSCodeCommitPowerUser` (or a scoped equivalent) attached

**Equivalent via AWS CLI:**
```bash
aws codecommit create-repository \
  --repository-name day53-cicd-repo \
  --repository-description "Day 53 CI/CD pipeline practice repo"
```
> Note: Git credentials themselves must be generated via the console — there's no CLI equivalent for HTTPS Git credential generation.

> Status: reference implementation — pending execution + screenshot of the created repo and downloaded Git credentials confirmation.

---

### Task 2 — Clone, Commit, and Push to CodeCommit

**Step 1: Get the clone URL**
- CodeCommit → Repositories → `day53-cicd-repo` → **Clone URL → HTTPS**

**Step 2: Clone using the IAM Git credentials**
```bash
git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/day53-cicd-repo
cd day53-cicd-repo
```
When prompted, enter the **Git username/password** generated in Task 1 (not your AWS console password or access key).

**Step 3: Add a file and commit locally**
```bash
echo "# Day 53 CI/CD Practice" > notes.md
git add notes.md
git commit -m "Add initial notes file"
```

**Step 4: Push to CodeCommit**
```bash
git push origin main
```
> If the repo's default branch differs (e.g., `master`), match that branch name instead.

**Verification:**
- CodeCommit console → `day53-cicd-repo` → confirm `notes.md` appears with the correct commit message and author

> Status: reference implementation — pending execution + screenshot of the pushed commit visible in the CodeCommit console.

---

## 3. What is CodeBuild?
AWS CodeBuild is a fully managed build service — it compiles source code, runs tests, and produces deployable artifacts, without needing to provision or manage your own build servers. It scales automatically to handle concurrent builds and only charges for the compute time actually used.

### 3.1 The `buildspec.yml` File
CodeBuild's behavior is driven by a **buildspec file** (YAML) placed in the root of the source repository. It defines the build in **phases**:

| Phase | Purpose |
|---|---|
| `install` | Install any runtime dependencies/tools needed for the build |
| `pre_build` | Commands to run before the main build (e.g., logging in to a registry) |
| `build` | The main build commands |
| `post_build` | Commands after the build completes (e.g., pushing an image, cleanup) |
| `artifacts` | Defines which files are collected as the build's output |

---

## 4. CodeBuild Tasks

### Task 1 — Create `index.html` in the CodeCommit Repository

**Step 1: Add a simple `index.html`**
```bash
cd day53-cicd-repo
cat <<'EOF' > index.html
<!DOCTYPE html>
<html>
<head><title>Day 53 CI/CD</title></head>
<body>
  <h1>Hello from CodeBuild + CodeCommit — Day 53</h1>
</body>
</html>
EOF

git add index.html
git commit -m "Add index.html for CodeBuild task"
git push origin main
```

> Status: reference implementation — pending execution + screenshot of `index.html` visible in the CodeCommit repo.

---

### Task 2 — Build `index.html` Using Nginx via `buildspec.yaml`

**Objective:** Add a `buildspec.yaml` that runs the build using an Nginx-based container image, and complete a CodeBuild project run against this repo.

**Step 1: Add `buildspec.yaml` to the repo root**
```yaml
version: 0.2

phases:
  install:
    commands:
      - echo "Install phase — no extra dependencies needed for a static file"
  pre_build:
    commands:
      - echo "Pre-build phase started on $(date)"
  build:
    commands:
      - echo "Build phase — validating index.html exists"
      - test -f index.html && echo "index.html found"
      - mkdir -p dist
      - cp index.html dist/
  post_build:
    commands:
      - echo "Build completed on $(date)"

artifacts:
  files:
    - dist/index.html
  discard-paths: no
```

**Step 2: Commit and push the buildspec**
```bash
git add buildspec.yaml
git commit -m "Add buildspec.yaml for CodeBuild"
git push origin main
```

**Step 3: Create the CodeBuild project (console)**
1. CodeBuild → Build projects → Create build project
2. Project name: `day53-nginx-build`
3. Source provider: **AWS CodeCommit**
4. Repository: `day53-cicd-repo`, branch: `main`
5. Environment:
   - Managed image → Operating system: **Amazon Linux 2**
   - Runtime: **Standard**
   - Image: use the standard image, or specify an **Nginx-based custom image** if the build needs Nginx actually installed/running (e.g., `nginx:latest` as a custom container image, if testing serving the file rather than just validating it)
6. Buildspec: **Use a buildspec file** (it will pick up `buildspec.yaml` from the repo root)
7. Artifacts: Amazon S3 → select/create a bucket (e.g., `day53-build-artifacts-<your-id>`)
8. Create build project

**Step 4: Run the build**
- CodeBuild → `day53-nginx-build` → **Start build**
- Watch the phase-by-phase logs (Install → Pre-build → Build → Post-build)
- Once successful, confirm the `dist/index.html` artifact appears in the configured S3 bucket

**Equivalent via AWS CLI:**
```bash
aws codebuild create-project \
  --name day53-nginx-build \
  --source type=CODECOMMIT,location=https://git-codecommit.<region>.amazonaws.com/v1/repos/day53-cicd-repo \
  --artifacts type=S3,location=day53-build-artifacts-<your-id> \
  --environment type=LINUX_CONTAINER,image=aws/codebuild/amazonlinux2-x86_64-standard:5.0,computeType=BUILD_GENERAL1_SMALL \
  --service-role arn:aws:iam::<account-id>:role/codebuild-service-role

aws codebuild start-build --project-name day53-nginx-build
```

**Verification:**
```bash
aws codebuild batch-get-builds --ids <build-id>
aws s3 ls s3://day53-build-artifacts-<your-id>/
```

> Status: reference implementation — pending execution + screenshots of the successful build log and the resulting artifact in S3.

---

## 5. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| `git clone`/`push` prompts fail with 403 | Wrong credentials used (console password/access key instead of Git credentials) | Use the username/password generated under IAM → Security credentials → HTTPS Git credentials |
| `git push` rejected | Local branch doesn't match the repo's default branch name | Check with `git branch -a` and push to the correct branch (`main` vs `master`) |
| CodeBuild project fails at `install`/`build` phase | `buildspec.yaml` syntax error or missing file referenced | Validate YAML indentation; confirm `index.html` exists in the repo root before the build runs |
| CodeBuild can't access the CodeCommit repo | Service role lacks CodeCommit read permissions | Attach `AWSCodeCommitReadOnly` (or broader) to the CodeBuild service role |
| Artifacts don't appear in S3 | Wrong bucket/path configured, or `artifacts` section misconfigured in buildspec | Recheck the `artifacts.files` paths match what the build phase actually produces |

---

## 6. Key Learnings
- CodeCommit's Git credentials are a separate, purpose-built credential type — distinct from both console passwords and AWS Access Keys — specifically scoped for Git HTTPS operations.
- A `buildspec.yaml`'s phase structure (`install` → `pre_build` → `build` → `post_build`) maps closely to how a CI pipeline conceptually works regardless of the specific tool — this pattern will recur with CodePipeline and CodeDeploy over the next few days.
- CodeBuild removes the need to maintain dedicated build servers — a direct parallel to how Fargate (Day 52) removed the need to manage EC2 instances for ECS.
- This is explicitly Part I of a longer arc — CodeCommit + CodeBuild here, CodeDeploy + CodePipeline + S3 to follow, building toward one complete pipeline.

## 7. Resources
- [AWS CodeCommit — User Guide](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html)
- [Setting Up Git Credentials for CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html)
- [AWS CodeBuild — User Guide](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
- [Buildspec Reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)

---
