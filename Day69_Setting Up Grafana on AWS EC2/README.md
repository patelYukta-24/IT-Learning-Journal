# Day 69: Setting Up Grafana on AWS EC2

## Table of Contents
- [Introduction](#introduction)
- [Task 1: Launch an EC2 Instance](#task-1-launch-an-ec2-instance)
- [Task 2: Install Grafana](#task-2-install-grafana)
- [Task 3: Access Grafana](#task-3-access-grafana)
- [Task 4: Secure Your Grafana Instance](#task-4-secure-your-grafana-instance)
- [Verifying the Setup](#verifying-the-setup)
- [Common Issues & Troubleshooting](#common-issues--troubleshooting)
- [Resources](#resources)
- [Key Takeaways](#key-takeaways)

---

## Introduction

With the theory behind Prometheus and Grafana in place, today's focus shifts to hands-on deployment: standing up a real Grafana instance on an AWS EC2 server. This mirrors a common real-world DevOps task — provisioning a small cloud VM, installing a monitoring tool on it, exposing it securely, and locking it down — end to end.

---

## Task 1: Launch an EC2 Instance

**Goal:** Launch an EC2 instance suitable for running Grafana, with the right AMI, instance type, storage, and security group rules.

**Steps:**

1. Sign in to the **AWS Management Console** and navigate to **EC2 → Instances → Launch instance**.
2. **Choose an AMI:** Select **Ubuntu Server 22.04 LTS** (a widely supported, well-documented AMI for Grafana's official install instructions).
3. **Choose an instance type:** Select **t2.micro** — sufficient for a first Grafana setup and eligible under the AWS Free Tier for testing/learning purposes.
4. **Configure instance details:** Leave networking defaults (default VPC/subnet) unless a specific setup is required; ensure **Auto-assign Public IP** is enabled so the instance is reachable from a browser.
5. **Add storage:** The default 8 GB gp2/gp3 root volume is enough for a basic Grafana install; increase it later if extra dashboards, plugins, or data retention are needed.
6. **Configure the security group** — this is the most important step for reachability. Add the following inbound rules:

   | Type | Protocol | Port | Source | Purpose |
   |---|---|---|---|---|
   | SSH | TCP | 22 | My IP (recommended) | Remote administration |
   | HTTP | TCP | 80 | 0.0.0.0/0 | Web traffic (used later for reverse proxy / Let's Encrypt) |
   | HTTPS | TCP | 443 | 0.0.0.0/0 | Encrypted web traffic (used later for reverse proxy) |
   | Custom TCP | TCP | 3000 | 0.0.0.0/0 (or restrict later) | Grafana's default web UI port |

   > Restricting SSH to "My IP" rather than `0.0.0.0/0` is good practice to avoid exposing the instance to brute-force attempts from the whole internet.

7. **Key pair:** Create or select an existing key pair (`.pem` file) for SSH access, and keep it stored securely.
8. Review and **Launch instance**.

---

## Task 2: Install Grafana

**Goal:** Connect to the running instance via SSH and install Grafana following the official installation process.

**Steps:**

1. **Connect via SSH** from a local terminal:

   ```bash
   chmod 400 my-key.pem
   ssh -i my-key.pem ubuntu@<EC2_PUBLIC_IP>
   ```

2. **Update the package manager:**

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install prerequisite packages** needed to add Grafana's repository securely:

   ```bash
   sudo apt install -y software-properties-common wget apt-transport-https
   ```

4. **Add Grafana's GPG key** — this verifies the authenticity of packages coming from Grafana's repository:

   ```bash
   sudo mkdir -p /etc/apt/keyrings/
   wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
   ```

5. **Add Grafana's repository** to the package manager's sources list:

   ```bash
   echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
   ```

6. **Update package lists again** (so the newly added repo is picked up) and **install Grafana**:

   ```bash
   sudo apt update
   sudo apt install -y grafana
   ```

7. **Start the Grafana server** and enable it so it launches automatically on every boot:

   ```bash
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```

8. **Verify the service is running:**

   ```bash
   sudo systemctl status grafana-server
   ```

   A healthy service shows `active (running)`.

---

## Task 3: Access Grafana

**Goal:** Reach the Grafana web UI from a browser and log in for the first time.

**Steps:**

1. In a web browser, navigate to:

   ```
   http://<EC2_PUBLIC_IP>:3000
   ```

2. The **Grafana login page** should load.
3. Log in with the **default credentials**:
   - Username: `admin`
   - Password: `admin`
4. Grafana immediately prompts for a **new password** on first login — this default-credential prompt is a deliberate security measure so the well-known default password is never left active.

> If the page doesn't load, double-check the EC2 security group allows inbound TCP on port 3000, and confirm `grafana-server` is actually running (`sudo systemctl status grafana-server`).

---

## Task 4: Secure Your Grafana Instance

**Goal:** Move beyond the default setup toward something closer to a production-appropriate, secured deployment.

**Steps:**

1. **Change the default password** as prompted at first login — never leave `admin`/`admin` active beyond that first login.

2. **Set up an SSL/TLS certificate** so traffic to the dashboard is encrypted rather than sent in plaintext:
   - **Let's Encrypt**, via the **Certbot** tool, is a good free option for obtaining a trusted certificate for a domain pointed at the EC2 instance's public IP.
   - A certificate requires a real domain name (or subdomain) pointed at the instance — a bare IP address cannot be issued a publicly trusted certificate.

3. **Put a reverse proxy in front of Grafana** (Nginx or Apache) rather than exposing Grafana's own port 3000 directly to the internet. This is considered best practice because it:
   - Terminates SSL/TLS at the proxy, so Grafana itself doesn't need to manage certificates directly.
   - Lets the dashboard be reached over the standard HTTPS port (443) instead of a nonstandard port like 3000.
   - Adds a layer that can apply additional protections (rate limiting, access restrictions, request filtering) in front of the application.

   **Example minimal Nginx reverse proxy config** (`/etc/nginx/sites-available/grafana`):

   ```nginx
   server {
       listen 80;
       server_name your-domain.example.com;

       location / {
           proxy_pass http://localhost:3000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

   ```bash
   sudo apt install -y nginx
   sudo ln -s /etc/nginx/sites-available/grafana /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

   Once this is in place, **Certbot** (`sudo apt install certbot python3-certbot-nginx`) can automatically obtain and wire in a Let's Encrypt certificate for the configured domain, upgrading the site to HTTPS.

4. **Tighten the security group** after the reverse proxy is in place: since Nginx now serves the dashboard over 80/443, the inbound rule for port 3000 can be restricted (or removed entirely) so Grafana is only reachable through the proxy, not directly.

---

## Verifying the Setup

A quick end-to-end checklist to confirm everything is working correctly:

- [ ] `sudo systemctl status grafana-server` shows `active (running)`
- [ ] Grafana UI loads at `http://<EC2_PUBLIC_IP>:3000` (or via the domain once the reverse proxy is set up)
- [ ] Default `admin`/`admin` login triggers a forced password change, and the new password has been set
- [ ] (After reverse proxy setup) the dashboard is reachable over `https://your-domain.example.com` with a valid certificate (padlock shown in the browser)
- [ ] Direct access to port 3000 from the public internet is restricted once the reverse proxy is confirmed working

---

## Common Issues & Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Browser can't reach `<IP>:3000` | Security group missing port 3000 rule | Add an inbound rule for TCP 3000 (or restrict to the proxy once set up) |
| SSH connection refused | Security group missing port 22 rule, or wrong key/user | Confirm inbound rule for port 22 and that you're using `ubuntu@<IP>` with the correct `.pem` |
| `grafana-server` fails to start | Port conflict or misconfigured `grafana.ini` | Check `sudo journalctl -u grafana-server -n 50` for the specific error |
| Certbot fails to issue certificate | Domain not yet pointed at the instance's public IP, or port 80 blocked | Confirm DNS propagation and that port 80 is open in the security group |

---

## Resources

- Official Grafana installation documentation (Debian/Ubuntu instructions used as the basis for Task 2 above)
- Chetan Rakhra's LinkedIn post walking through Grafana-on-EC2 setup, referenced as a supplementary step-by-step guide
- Let's Encrypt / Certbot documentation for obtaining and renewing free TLS certificates

---

## Key Takeaways

- Deploying Grafana on EC2 mirrors a very common real-world DevOps task: provision a VM, install the monitoring tool, expose it deliberately (not accidentally), and lock it down.
- Security group rules are the first gatekeeper — SSH for admin access, and Grafana's port (3000) or the reverse proxy's ports (80/443) for the dashboard itself.
- Grafana's install process (repo + GPG key + package install) exists specifically so packages can be verified as authentic before being trusted and installed.
- The forced password change on first login exists because `admin`/`admin` is public knowledge — leaving it unchanged is one of the most common ways monitoring dashboards get compromised.
- A reverse proxy (Nginx/Apache) with Let's Encrypt is the standard way to put a trusted, encrypted front door on an internal tool like Grafana instead of exposing its raw application port directly to the internet.
