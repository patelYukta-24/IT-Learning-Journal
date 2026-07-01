# Nessus — Hands-On Learning Notes

> Part of my **IT Learning Journal** — documenting hands-on lab practice and self-study as I build skills toward a DevOps/Cloud/Linux/Security-adjacent role.
>
> Everything in this document reflects **personal lab practice**, using free tools (Nessus Essentials), virtual machines, and official documentation — not professional/employer experience. I'm noting that clearly here so it's never misread as job history.

---

## 📌 What is Nessus?

**Nessus** is a vulnerability scanning tool made by **Tenable**. In simple terms, it scans computers, servers, and network devices to find security weaknesses — like missing patches, weak configurations, or outdated software — before someone with bad intentions finds them first.

Think of it like a health check-up for your computers: instead of a doctor checking your body, Nessus checks a system's security "health" and gives you a report of what's wrong and how serious each issue is.

---

## 🏢 Why Companies Use Nessus

| Reason | Explanation |
|--------|-------------|
| **Find weaknesses early** | Scans catch vulnerabilities before attackers exploit them |
| **Patch verification** | Confirms whether security updates were actually applied |
| **Compliance** | Many industries (finance, healthcare) are required to run regular vulnerability scans |
| **Asset visibility** | Helps IT teams know exactly what's running on their network |
| **Risk prioritization** | Tells teams which issues to fix first, based on severity |

Vulnerability scanning matters because most real-world breaches don't come from advanced hacking — they come from **known, unpatched vulnerabilities** that a tool like Nessus would have flagged.

---

## ⚙️ How Vulnerability Scanning Works (Simple Version)

1. Nessus is pointed at one or more target systems (by IP address or hostname).
2. It checks which ports are open and what services are running.
3. It compares what it finds against a huge database of known vulnerabilities.
4. It generates a report ranking issues by severity.

---

## 🏗️ Basic Architecture

```
        Administrator (Me)
               │
               ▼
        Nessus Scanner
               │
               ▼
        Target Devices
     (Windows VM / Linux VM)
               │
               ▼
           Scan Runs
               │
               ▼
        Report Generated
```

**Explanation of each step:**
- **Administrator** — I configure the scan (targets, policy, credentials).
- **Nessus Scanner** — The engine that performs the actual scanning.
- **Target Devices** — In my lab, this was a Windows VM and a Linux VM on an isolated internal network.
- **Scan Runs** — Nessus checks ports, services, and known vulnerabilities.
- **Report Generated** — Results are shown with severity ratings and remediation suggestions.

---

## 💻 Nessus Installation (Lab Notes)

I installed **Nessus Essentials** (the free version, limited to 16 IP addresses) in a home lab environment.

### Prerequisites
- A VM or machine with enough resources (I used a VM with 4 GB RAM / 2 vCPUs for lab purposes)
- Internet access (to download Nessus and get an activation code)
- A free Tenable account (required to receive the Nessus Essentials activation code by email)

### Installation Steps
1. Registered for a free activation code on Tenable's website.
2. Downloaded the Nessus installer for my OS.
3. Installed it (on Linux, using `dpkg`; on Windows, using the `.msi` installer).
4. Started the Nessus service.
5. Opened the web interface at `https://localhost:8834`.
6. Entered the activation code to unlock Nessus Essentials.
7. Waited for the initial plugin feed to download and compile (took around 15–20 minutes on my setup).

### Verification
```bash
sudo systemctl status nessusd
```
I confirmed the service showed `active (running)` and that the web dashboard loaded successfully with the plugin feed marked "up to date."

---

## 🔧 Basic Configuration (What I Practiced)

| Area | What I Practiced |
|------|-------------------|
| **Scanner Settings** | Basic performance settings (simultaneous hosts/checks) |
| **Plugin Updates** | Manually triggering updates and confirming the feed date |
| **Credentials** | Adding SSH credentials for my Linux VM and local admin credentials for my Windows VM |
| **Network Settings** | Making sure my scanner VM could reach the target VMs on the same internal network |
| **User Accounts** | Explored the admin account setup (single-user lab, so no multi-user setup needed) |
| **Scan Policies** | Created a "Basic Network Scan" policy and a "Credentialed Patch Audit" policy |
| **Schedules** | Practiced setting a scan to run on a recurring schedule vs. on-demand |
| **Notifications** | Reviewed the email notification settings (not fully tested, since my lab had no SMTP server configured) |

---

## ⚠️ Common Configuration Mistakes (Things I Ran Into)

| Problem | Possible Cause | How to Fix | Verification |
|---------|-----------------|------------|----------------|
| Wrong IP range entered | Typo or misunderstanding CIDR notation | Double-check IP range format (e.g., `192.168.1.0/24`) | Re-run scan and confirm target count matches expected hosts |
| Wrong credentials | Typo, or account locked | Re-enter credentials, confirm account isn't locked | Check "Authentication" status in scan results |
| Plugin update failure | No internet access, or license/activation issue | Check network connectivity, confirm activation status | Run manual plugin update and check the feed date |
| Firewall blocking scanner | Host firewall dropping scan traffic | Allow scanner IP through target firewall | Re-scan and check if more ports show as open |
| DNS resolution problems | Hostname used instead of IP, DNS not resolving | Use IP address instead, or fix DNS config | `ping <hostname>` to confirm resolution |
| License activation issues | Wrong/expired activation code | Re-request activation code from Tenable | Check license status in Nessus settings |
| Time sync problems | VM clock drift causing SSL/auth errors | Sync VM clock with NTP | Compare VM time to host time |
| Incorrect scan policy | Using a template that doesn't match the goal (e.g., using "Basic" instead of "Credentialed Patch Audit") | Choose the right policy template for the goal | Confirm scan results match expected depth |
| Authentication failure (credentialed scan) | Wrong protocol enabled, or insufficient privileges | Verify SSH/SMB port is reachable and account has admin/sudo rights | Check "Authentication" field in scan summary |

---

## 🐞 Common Errors I Worked Through (Lab Troubleshooting)

### 1. Plugin Update Failed
- **Symptoms:** Feed date not updating; warning banner in console
- **Reason:** My lab VM temporarily lost internet access
- **Resolution:** Restored network connectivity, then ran a manual update
- **Command:** `sudo /opt/nessus/sbin/nessuscli update`
- **Verification:** Checked the plugin feed date in the console — it updated to the current date

### 2. Scan Not Starting
- **Symptoms:** Scan status stuck at "Launching"
- **Reason:** Nessus service had not fully finished initializing after a restart
- **Resolution:** Waited a few minutes, then relaunched the scan
- **Verification:** Progress bar began moving normally

### 3. Host Not Reachable
- **Symptoms:** Scan completed almost instantly with zero findings
- **Reason:** Target VM was powered off / on a different virtual network
- **Resolution:** Powered on the VM, confirmed both VMs were on the same internal network
- **Verification:** `ping <target-ip>` succeeded from the scanner VM

### 4. Credentialed Scan Failed
- **Symptoms:** Scan completed but showed "Authentication failed" for the target
- **Reason:** SSH key wasn't set up correctly on my Linux VM (I initially tried key-based auth)
- **Resolution:** Switched to password-based SSH auth for the lab test, and later fixed the SSH key permissions
- **Verification:** Authentication status showed "Success" in the next scan

### 5. Scanner Offline
- **Symptoms:** Web console unreachable
- **Reason:** Nessus service had stopped
- **Resolution:** Restarted the service
- **Command:** `sudo systemctl restart nessusd`
- **Verification:** `sudo systemctl status nessusd` showed `active (running)`

### 6. Port Connectivity Issue
- **Symptoms:** Couldn't reach the web console remotely from another lab machine
- **Reason:** Local firewall on the Nessus host was blocking port 8834
- **Resolution:** Allowed inbound traffic on port 8834
- **Verification:** Accessed `https://<scanner-ip>:8834` successfully from the second machine

### 7. License Error
- **Symptoms:** "Invalid activation code" message during setup
- **Reason:** Copy-paste error when entering the code
- **Resolution:** Carefully re-typed the code from the activation email
- **Verification:** Nessus Essentials unlocked successfully

### 8. Service Not Running
- **Symptoms:** Couldn't connect to the web UI at all
- **Reason:** Service hadn't auto-started after a VM reboot
- **Resolution:** Started the service manually and enabled it on boot
- **Commands:**
```bash
sudo systemctl start nessusd
sudo systemctl enable nessusd
```

### 9. Scan Timeout / Slow Scan
- **Symptoms:** Scan took much longer than expected on my low-resource lab VM
- **Reason:** VM had limited CPU/RAM allocated, and I was scanning too many hosts simultaneously for the available resources
- **Resolution:** Reduced "max simultaneous hosts" in the scan policy performance settings
- **Verification:** Follow-up scan completed noticeably faster

---

## 🔐 Credentialed vs. Non-Credentialed Scans

| Type | What It Means | What I Observed |
|------|----------------|-------------------|
| **Non-Credentialed** | Nessus scans the target from the outside, without logging in — similar to an attacker's view | Fewer, less detailed findings; mostly open ports and banner-based guesses |
| **Credentialed** | Nessus logs into the target using SSH (Linux) or SMB/WinRM (Windows) | Much more accurate results — exact missing patches, installed software versions, and configuration issues |

**Lab takeaway:** Running both scan types against the same VM showed a clear difference — the credentialed scan found far more (and more accurate) issues than the non-credentialed one.

---

## 🟥 Vulnerability Severity Levels

| Severity | Simple Meaning |
|----------|------------------|
| **Critical** | Extremely dangerous — could allow full system takeover; fix immediately |
| **High** | Serious risk — needs to be fixed soon |
| **Medium** | Moderate risk — should be fixed, but less urgent |
| **Low** | Minor risk — good to fix, low real-world impact |
| **Info** | Not a vulnerability itself — just useful information (e.g., "SSH is running") |

---

## 🧩 CVE and CVSS

**What is a CVE?**
CVE stands for **Common Vulnerabilities and Exposures**. It's a unique ID given to a specific, publicly known vulnerability — for example, `CVE-2021-44228` (the famous "Log4Shell" vulnerability). Think of it as a case number for a specific security bug.

**What is CVSS?**
CVSS stands for **Common Vulnerability Scoring System**. It's a score from 0 to 10 that tells you how severe a vulnerability is — the higher the number, the more dangerous it is.

**Why are they important?**
They give everyone (security teams, vendors, researchers) a shared, standardized way to talk about and rank vulnerabilities, instead of everyone using their own subjective descriptions.

**How does Nessus use them?**
Every vulnerability Nessus finds is linked to a CVE (when applicable) and given a CVSS score, so I can quickly understand both *what* the issue is and *how serious* it is.

---

## 🔄 Scan Workflow (What I Practiced End-to-End)

```
Create Scan
     ↓
Configure Scan (target, policy, credentials)
     ↓
Run Scan
     ↓
Collect Results
     ↓
Analyze Vulnerabilities (check severity, CVE, CVSS)
     ↓
Note Remediation Recommendations
     ↓
(In a real job) Assign to IT Team for Patching
     ↓
Re-Scan to Confirm Fix
```

In my lab, I completed this full cycle on my Windows and Linux VMs: I intentionally left a few services outdated, scanned, reviewed the findings, then patched the VM and re-scanned to confirm the vulnerability was resolved.

---

## ✅ Best Practices (What I Learned)

**Installation**
- Give the Nessus VM enough resources before installing — a slow install/first-scan experience is often just a resource problem.

**Configuration**
- Always double-check target IP ranges before launching a scan — a wrong range wastes time and gives no useful data.

**Plugin Updates**
- Keep plugins updated regularly; an outdated feed means missing new vulnerabilities.

**Credential Management**
- Use a dedicated low-privilege scanning account rather than a "root"/"administrator" account, even in a lab — it's a habit worth building early.

**Scan Scheduling**
- Schedule scans during low-usage times if scanning real production-like systems, since scans do use CPU/network resources on the target.

**Report Review**
- Don't just look at severity — read the plugin description and remediation advice to actually understand *why* something is a risk.

**Troubleshooting**
- Check the Nessus service status and logs first — most issues I ran into were either a stopped service, a network/firewall block, or a credentials problem.

---

## 🎤 Interview Questions (Beginner Level)

**1. What is Nessus used for?**
It's a vulnerability scanning tool that finds security weaknesses like missing patches and misconfigurations on computers and network devices.

**2. Who makes Nessus?**
Tenable, Inc.

**3. What is the difference between a credentialed and non-credentialed scan?**
A credentialed scan logs into the target with valid credentials for deeper, more accurate results. A non-credentialed scan checks the target from the outside without logging in.

**4. What port does the Nessus web interface use by default?**
Port 8834 (HTTPS).

**5. What is a CVE?**
A unique ID for a specific, publicly known vulnerability.

**6. What is CVSS?**
A 0–10 scoring system that shows how severe a vulnerability is.

**7. What does "Critical" severity mean in a scan report?**
It means the issue is extremely dangerous and should be fixed immediately, as it could allow serious compromise of the system.

**8. What is a scan policy?**
A saved set of scan settings — which checks to run, which ports to scan, and which credentials to use.

**9. Why would a scan show fewer results without credentials?**
Because without login access, Nessus can only see what's visible from the outside (open ports, banners) — not the actual installed software or patch levels.

**10. What's the first thing you'd check if the Nessus web console won't load?**
Whether the Nessus service is actually running, using something like `systemctl status nessusd`.

**11. What would you check if a credentialed scan shows "authentication failed"?**
The credentials themselves, whether the required port (SSH/SMB) is reachable, and whether the account has sufficient privileges.

**12. Why is it important to keep Nessus plugins updated?**
New vulnerabilities are discovered constantly — outdated plugins mean the scanner might miss newly disclosed issues.

**13. What's the difference between Nessus Essentials and Nessus Professional?**
Essentials is free but limited to scanning 16 IPs, meant for learning. Professional is paid and removes that limit, meant for real organizational use.

**14. What should you check if a target host shows as "unreachable" in a scan?**
Whether the host is actually powered on, on the same network, and not blocked by a firewall.

**15. Why do companies run vulnerability scans regularly instead of just once?**
Because new vulnerabilities are discovered all the time, and systems change (new software installed, configs changed) — a one-time scan only reflects that single moment.

**16. What is a false positive in vulnerability scanning?**
A finding that Nessus reports, but that doesn't actually exist or apply to the target system.

**17. What's a basic step to take before scanning a fragile or old system?**
Use a "safe checks" option/scan policy to reduce the risk of the scan crashing the system.

**18. What information does a vulnerability report usually include?**
The vulnerability name, severity, affected host, CVE reference, CVSS score, and remediation recommendation.

**19. Why might a scan take a long time to complete?**
It could be scanning many hosts/ports at once, the network could be slow, or the scanning machine might not have enough resources.

**20. What would you do after receiving a scan report showing several vulnerabilities?**
Review the severity and remediation guidance, then work with the responsible team to patch or fix the highest-priority issues first, and re-scan afterward to confirm the fix worked.

---

## 📝 My Learning Summary

Working through Nessus hands-on gave me a much clearer, practical understanding of how vulnerability scanning actually works — not just the theory. Setting up my own lab with Windows and Linux VMs let me experience the full cycle: installing and configuring the scanner, creating scan policies, running both credentialed and non-credentialed scans, and comparing the difference in results between them.

I also ran into (and worked through) real configuration issues — plugin update failures, unreachable hosts, authentication problems, and a stopped service — which taught me troubleshooting habits that go beyond just clicking "scan." Learning to read a report properly, understanding CVE/CVSS, and connecting severity levels to real remediation priorities gave me a solid foundation in vulnerability management basics.

This was self-directed lab practice, not professional job experience, but it reflects genuine hands-on time with the tool and a real understanding of how it's used in enterprise environments.
