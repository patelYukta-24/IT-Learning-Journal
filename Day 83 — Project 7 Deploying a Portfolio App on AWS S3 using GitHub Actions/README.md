# Day 83 — Project 7: Deploying a Portfolio App on AWS S3 using GitHub Actions

**Part XII: Project 7 (Day 83)**
**Curriculum:** NexusCorp DevOps Transformation — Shubham Londhe

---

## ❖ Project Overview

This project demonstrates how to automate the deployment of a static portfolio website to Amazon S3 using GitHub Actions for continuous integration and continuous deployment (CI/CD). A workflow was configured that builds and deploys the portfolio directly to an S3 bucket set up for static website hosting, triggered automatically whenever changes are pushed to the repository.

## ❖ Project Objective

- Understand CI/CD pipelines with GitHub Actions.
- Learn to automate the deployment of a static website to AWS S3.
- Gain practical experience with GitHub Actions and AWS services.

## ❖ Skills Showcased

- GitHub Actions for CI/CD
- AWS S3 for Static Website Hosting
- YAML Scripting for Workflow Creation
- AWS CLI for S3 Synchronization

## ❖ Environment

- **AWS services:** S3 (static website hosting), IAM (scoped deploy user)
- **CI/CD:** GitHub Actions
- **App:** Static portfolio site (HTML/CSS/JS build output; a React/Vite-based portfolio was used, producing a `build/`(`dist/`) folder)

---

## ❖ Tasks / Exercises

### Task 1: Automate Deployment of a Static Portfolio Application to AWS S3 Using GitHub Actions

#### Subtask 1 — Source a static portfolio application

Selected an existing personal portfolio project repository (static site with a standard `npm run build` step producing a deployable output folder) and cloned it locally to verify the build worked before wiring up CI/CD:

```bash
git clone https://github.com/<username>/portfolio-app.git
cd portfolio-app
npm install
npm run build
```

Confirmed the build output folder (`build/`) was generated correctly and opened `index.html` locally to sanity-check the site.

#### Subtask 2 — Create the S3 bucket for static website hosting

```bash
aws s3api create-bucket \
  --bucket <portfolio-bucket-name> \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

aws s3 website s3://<portfolio-bucket-name>/ \
  --index-document index.html \
  --error-document index.html
```

Disabled "Block all public access" for this bucket and attached a bucket policy allowing public reads:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<portfolio-bucket-name>/*"
    }
  ]
}
```

```bash
aws s3api put-bucket-policy \
  --bucket <portfolio-bucket-name> \
  --policy file://bucket-policy.json
```

#### Subtask 3 — Set up AWS CLI credentials securely as GitHub repository secrets

Created a dedicated IAM user (`github-actions-deployer`) with a minimal policy scoped only to the objects needed for deployment, rather than reusing a personal admin key:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:DeleteObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::<portfolio-bucket-name>",
        "arn:aws:s3:::<portfolio-bucket-name>/*"
      ]
    }
  ]
}
```

Generated an access key pair for this user, then added them under **GitHub repo → Settings → Secrets and variables → Actions**:

| Secret name             | Value                          |
|--------------------------|---------------------------------|
| `AWS_ACCESS_KEY_ID`      | IAM user's access key           |
| `AWS_SECRET_ACCESS_KEY`  | IAM user's secret key           |
| `AWS_REGION`             | `ap-south-1`                    |
| `S3_BUCKET_NAME`         | `<portfolio-bucket-name>`       |

No AWS credentials were ever committed to the repository — everything runs through GitHub's encrypted secrets store and is injected into the workflow at run time.

#### Subtask 4 — Define the GitHub Actions workflow

Created `.github/workflows/main.yml`:

```yaml
name: Deploy Portfolio to S3

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

      - name: Build project
        run: npm run build

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Sync build output to S3
        run: |
          aws s3 sync build/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

      - name: Deployment summary
        run: echo "Deployed to http://${{ secrets.S3_BUCKET_NAME }}.s3-website.${{ secrets.AWS_REGION }}.amazonaws.com"
```

Key design choices:
- Triggered only on pushes to `main`, so feature branches don't redeploy the live site.
- `--delete` on `aws s3 sync` removes stale files from the bucket that no longer exist in the new build, keeping the bucket an exact mirror of the latest build output.
- `aws-actions/configure-aws-credentials` is the official AWS action for injecting credentials into the runner's environment for the duration of the job, rather than hand-rolling `aws configure` calls.

#### Pushed the change and observed the workflow run

```bash
git add .github/workflows/main.yml
git commit -m "Add GitHub Actions workflow for S3 deployment"
git push origin main
```

Watched the run under the repo's **Actions** tab — all four steps (checkout, install, build, sync) completed successfully with green checkmarks, and the final `aws s3 sync` step logged each uploaded object with `upload:` lines confirming the transfer.

---

## ❖ Validation

- [x] Workflow run completed successfully on push to `main` (visible in the **Actions** tab)
- [x] `aws s3 ls s3://<portfolio-bucket-name>/` shows the built site files
- [x] Public S3 static website endpoint loads the portfolio correctly in a browser
- [x] A follow-up commit (small content change) triggered an automatic redeploy with no manual steps, confirming the CI/CD loop works end-to-end

**Public URL (S3 static website endpoint):**
```
http://<portfolio-bucket-name>.s3-website.ap-south-1.amazonaws.com
```

---

## ❖ Deliverables

- [x] **Working static portfolio website** hosted on AWS S3
- [x] **GitHub Actions workflow file** — `.github/workflows/main.yml`
- [x] **Logs of a successful build/deploy** — Actions tab run history, `aws s3 sync` upload logs
- [x] **Public URL** of the S3-hosted portfolio site

---

## ❖ Notes / Learnings

- Scoping the IAM user's policy to only `PutObject` / `DeleteObject` / `ListBucket` on the specific bucket (rather than broad `s3:*` or account-wide access) follows least-privilege practice — the deploy pipeline can only touch this one bucket even if the secret were ever compromised.
- `aws s3 sync ... --delete` is the piece that makes this a true mirror deployment; without `--delete`, old/removed files from previous builds would silently accumulate in the bucket.
- Static website hosting on S3 serves over plain HTTP by default — a follow-on improvement (not required for this exercise) would be fronting the bucket with CloudFront + ACM for HTTPS and a custom domain, similar to the optional CDN step explored in the Day 79 static hosting project.
- This was completed as a **self-study lab exercise** on a personal AWS account and personal GitHub repository — documented here as practice toward the GitHub Actions CI/CD and S3 hosting skill areas of the curriculum, not a client/production deployment.

---

## ❖ References

- GitHub Actions documentation
- `aws-actions/configure-aws-credentials` (official AWS GitHub Action)
- AWS S3 static website hosting documentation
