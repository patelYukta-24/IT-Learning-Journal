# Day 79: Project 3 — Hosting a Static Website on AWS S3

## Project Overview

This project deploys a static website using **Amazon S3**, demonstrating serverless web hosting on cloud object storage — reliable, scalable, and inexpensive, with no web server to provision or maintain. The project covers configuring an S3 bucket for static website hosting, applying the correct public-access permissions, optionally fronting the site with **CloudFront** for CDN performance, and pointing a custom domain at the deployment via DNS.

## Project Objective

- Configure an AWS S3 bucket for static website hosting.
- Apply the necessary bucket policy/permissions for public read access.
- Follow best practices for hosting static assets on S3.
- Attach a custom domain name to the bucket (or CloudFront distribution) with proper DNS configuration.

## Skills Showcased

- Cloud Services (AWS S3)
- DNS and Domain Configuration
- Cloud Security and IAM (bucket policies, public access blocks)
- Static Website Deployment

---

## Task 1: Host the Static Website on S3

### Subtask 1 — Register or choose a domain name

Chose/registered a domain (e.g., via Route 53, GoDaddy, or Namecheap) — e.g. `www.example-portfolio.com`. Note: the S3 bucket name must **exactly match** the domain name for website-endpoint hosting to work (e.g., bucket named `www.example-portfolio.com`).

### Subtask 2 — Create and configure the S3 bucket for static hosting

```bash
# Create the bucket (name must match the domain exactly)
aws s3api create-bucket \
  --bucket www.example-portfolio.com \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
```

Enable static website hosting on the bucket:

```bash
aws s3 website s3://www.example-portfolio.com/ \
  --index-document index.html \
  --error-document error.html
```

This can also be done via the console: **S3 → bucket → Properties → Static website hosting → Enable**, specifying `index.html` as the index document and `error.html` as the error document.

### Subtask 3 — Bucket policy for public read access

By default, S3 buckets block all public access. To serve a public website:

1. Under **Permissions → Block public access**, uncheck "Block all public access" for this bucket (scoped to just this bucket, not account-wide).
2. Attach a bucket policy granting public `GetObject` only:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::www.example-portfolio.com/*"
    }
  ]
}
```

**Security note**: this policy only grants read (`GetObject`) access to objects — it does **not** grant listing, write, or delete permissions to the public, which is the key best practice here: public visibility of the *content*, not the *bucket*.

### Subtask 4 — Upload website files

```bash
aws s3 sync ./website-files/ s3://www.example-portfolio.com/ \
  --acl public-read
```

Includes `index.html`, `error.html`, CSS/JS assets, and images. Content-type headers are set automatically by `aws s3 sync` based on file extensions, so `.html`, `.css`, and `.js` files serve with correct MIME types.

### Subtask 5 — CloudFront CDN (optional, for performance)

Creating a CloudFront distribution in front of the S3 bucket adds edge caching (lower latency worldwide), free HTTPS via **AWS Certificate Manager (ACM)**, and the ability to enforce HTTPS-only access (S3 website endpoints are HTTP-only on their own).

```bash
aws cloudfront create-distribution \
  --origin-domain-name www.example-portfolio.com.s3-website-ap-south-1.amazonaws.com \
  --default-root-object index.html
```

Key settings configured in the CloudFront console:
- **Origin**: the S3 static website endpoint (not the bucket's REST endpoint — the *website* endpoint is required for `index.html`/`error.html` routing to work correctly).
- **Viewer protocol policy**: "Redirect HTTP to HTTPS."
- **Alternate domain name (CNAME)**: `www.example-portfolio.com`.
- **Custom SSL certificate**: an ACM certificate issued for the domain (must be requested in `us-east-1` regardless of the bucket's region, since CloudFront only reads ACM certs from that region).

### Subtask 6 — DNS configuration

Updated DNS records at the domain registrar (or Route 53) to point the domain at the hosted site:

- **Without CloudFront** (S3 website endpoint only): create a **CNAME** record pointing `www.example-portfolio.com` → the S3 static website endpoint (e.g., `www.example-portfolio.com.s3-website-ap-south-1.amazonaws.com`). Note: a bare/apex domain (no `www`) can't use a CNAME under standard DNS rules — that needs Route 53's **Alias** record type instead.
- **With CloudFront**: create a CNAME (or Route 53 Alias) pointing the domain to the CloudFront distribution's domain name (e.g., `d123456abcdef8.cloudfront.net`), which is the recommended approach since it also gets HTTPS working properly.

---

## Best Practices Applied

- **Least-privilege bucket policy** — public access limited strictly to `s3:GetObject`, nothing else.
- **Separate index/error documents** — a proper `error.html` avoids exposing raw S3 XML error responses to visitors.
- **HTTPS enforced via CloudFront + ACM** rather than serving the site over plain HTTP from the S3 website endpoint.
- **Bucket naming matches the domain exactly** — required for the S3-website-endpoint-to-domain mapping to function without extra redirection tricks.
- **Versioning considered** (optional) — enabling S3 bucket versioning protects against accidental overwrites/deletions of site content during future updates.

---

## Deliverables

- [ ] **Live URL** of the static website — either the CloudFront domain or the custom domain once DNS propagates (e.g., `https://www.example-portfolio.com`)
- [ ] Screenshots/documentation of the S3 bucket configuration (Properties → Static website hosting panel, Permissions → bucket policy)
- [ ] This summary of the steps taken to secure and expose the website (see Subtask 3 and Best Practices above)
- [ ] Evidence of domain configuration — screenshot of the DNS records (CNAME/Alias) at the registrar or Route 53, pointing to the S3/CloudFront endpoint

*(Live URL and screenshots are placeholders — add your own once deployed to your actual AWS account and domain, so this stays honest, verifiable self-study documentation.)*

---

## Validation of Task Completion

- Uploaded/confirmed `index.html` renders correctly at the S3 website endpoint before adding DNS/CloudFront on top, isolating hosting issues from DNS/CDN issues.
- Verified the **bucket policy and public access settings** are correct by accessing the site anonymously (private/incognito browser window) rather than while logged into the AWS account.
- Tested the **error document** by requesting a non-existent path (e.g., `/doesnotexist`) and confirming `error.html` is returned instead of a raw AWS XML error.
- Once DNS propagated, confirmed the custom domain resolves to the site and (if CloudFront is used) loads over **HTTPS** with a valid certificate.
- If using CloudFront, confirmed cache invalidation works (`aws cloudfront create-invalidation --distribution-id <id> --paths "/*"`) after pushing an update to the S3 bucket, since CloudFront otherwise serves cached content until TTL expiry.

---

## Additional Resources

- [AWS Documentation: Hosting a Static Website on S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- Guides on DNS configuration for S3 static hosting (registrar-specific: Route 53, GoDaddy, Namecheap, etc.)
- AWS best practices for security and scalability in static website hosting (bucket policies, CloudFront, ACM)

---
