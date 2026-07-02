# SCCM (Microsoft Configuration Manager) — Hands-On Learning Notes

> Part of my **IT Learning Journal** — documenting hands-on lab practice and self-study as I build skills toward a DevOps/Cloud/Linux/Security-adjacent role.
>
> Everything here reflects **personal lab practice** — building SCCM in a home-lab environment using virtual machines and official Microsoft documentation — not professional/employer experience. I'm noting that clearly so it's never misread as job history.
>
> ⚠️ **Please edit the specific exercises below to match exactly what you did** in your own lab build.

---

## 📌 What is SCCM?

**SCCM** (System Center Configuration Manager), now officially called **Microsoft Configuration Manager (MECM/MCM)**, is a Microsoft tool used to manage large numbers of Windows devices in a company — installing software, deploying updates, enforcing configurations, and tracking hardware/software inventory, all from one central console.

In simple terms: instead of an IT technician walking to every employee's desk to install software or run updates, SCCM lets them push those changes to hundreds or thousands of devices at once, remotely.

---

## 🏢 Why Companies Use SCCM

| Reason | Explanation |
|--------|-------------|
| **Software deployment at scale** | Install applications on hundreds/thousands of PCs without manual visits |
| **Patch management** | Push Windows updates and third-party patches company-wide |
| **OS deployment** | Image and deploy new operating systems to new/replacement machines |
| **Inventory management** | Track exactly what hardware and software exists across the company |
| **Compliance enforcement** | Ensure devices meet required security/configuration baselines |
| **Reduced IT workload** | Automates repetitive, manual desktop support tasks |

This matters because manually managing software and updates across a large company simply doesn't scale — SCCM turns a task that would take weeks of manual work into something that can be scheduled and pushed centrally.

---

## ⚙️ How It Works (Simple Version)

1. SCCM discovers devices on the network (via Active Directory, IP ranges, or other discovery methods).
2. An SCCM client agent is installed on each managed device.
3. Administrators create **deployments** (software packages, updates, configuration baselines).
4. The SCCM client checks in with the server, receives the deployment, and applies it.
5. Results (success/failure) are reported back to the central console.

---

## 🏗️ Basic Architecture

```
              Administrator (Me)
                      │
                      ▼
            SCCM Console (Admin UI)
                      │
                      ▼
             SCCM Site Server
          (with SQL Server Database)
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
 Distribution      Management    Software Update
   Point            Point            Point
        │             │             │
        └─────────────┼─────────────┘
                      ▼
             Managed Client Devices
           (Windows PCs / Servers)
```

**Explanation of each part:**
- **SCCM Console** — the admin interface used to create deployments and view reports.
- **Site Server** — the core SCCM server, backed by a SQL Server database that stores all configuration and inventory data.
- **Distribution Point** — stores and delivers actual software packages/content to clients.
- **Management Point** — the point clients communicate with to get policy and deployment instructions.
- **Software Update Point** — integrates with WSUS to manage Windows Updates.
- **Client Devices** — end machines running the SCCM client agent that receive and apply deployments.

---

## 💻 Building the Lab (What I Practiced)

Setting up SCCM is a much heavier build than a single-tool install, so my lab environment involved multiple VMs:

- Built a small lab with **Hyper-V/VirtualBox**, including:
  - A **Windows Server** VM (Active Directory Domain Services + DNS)
  - A **Windows Server** VM for **SQL Server** (SCCM database)
  - A **Windows Server** VM for the **SCCM Site Server** itself
  - A **Windows 10/11** client VM to act as a managed endpoint
- Joined all VMs to the same lab Active Directory domain
- Installed prerequisites (WSUS, IIS, .NET features) before installing SCCM
- Ran the SCCM setup wizard to install the primary site
- Configured **boundaries and boundary groups** so SCCM knew which IP range/subnet the client belonged to
- Pushed the SCCM client agent to the Windows 10/11 VM
- Created a basic **application deployment** (a simple free app) and deployed it to the test client
- Configured a **Software Update Point** and tested deploying a Windows update
- Reviewed **client health/status** in the console to confirm the deployment succeeded

*(Adjust this list to match exactly what you built and tested in your own lab.)*

---

## 🔧 Core Concepts I Studied

| Concept | What It Means |
|---------|----------------|
| **Site Server** | The central SCCM server managing the environment |
| **Distribution Point** | Where software content is stored and delivered from |
| **Boundaries / Boundary Groups** | Define which network locations belong to which SCCM site |
| **Collections** | Groups of devices or users used to target deployments |
| **Deployments** | The actual push of software, updates, or configurations to a collection |
| **Client Agent** | Software installed on managed devices that communicates with SCCM |
| **Software Update Point (SUP)** | Integrates with WSUS to manage patch deployment |
| **Task Sequences** | Automated, multi-step processes (e.g., for OS deployment/imaging) |
| **Compliance Baselines** | Rules used to check/enforce device configuration standards |

---

## ⚠️ Common Configuration Mistakes (Things I Ran Into)

| Problem | Possible Cause | How to Fix | Verification |
|---------|-----------------|------------|----------------|
| Client not appearing in console | Client push failed, or boundary not configured | Manually install client, verify boundary group includes the client's subnet | Check client status shows "Active" in console |
| Deployment stuck at "In Progress" | Client hasn't checked in recently, or content wasn't distributed properly | Force a client policy retrieval, confirm content distributed to the Distribution Point | Check deployment status monitoring in console |
| SQL Server connection errors | SQL service not running, or wrong permissions during setup | Verify SQL service status, confirm the SCCM service account has correct DB permissions | Re-run SCCM prerequisite checker |
| WSUS/Software Update Point sync failing | WSUS not properly configured before SUP setup | Reconfigure WSUS, re-run SUP configuration wizard | Check "wsyncmgr.log" for sync success |
| Boundary misconfiguration | Wrong subnet/IP range entered | Correct the boundary definition to match actual client subnet | Client shows correct site assignment |
| Content not distributing to Distribution Point | Network/permission issue between site server and DP | Check DP status and distribution job in console, verify service account permissions | Distribution status shows "Success" |

---

## 🐞 Common Errors I Worked Through (Lab Troubleshooting)

### 1. Client Push Failed
- **Symptoms:** Client didn't appear as installed after push installation
- **Reason:** Windows Firewall on the client VM was blocking the install
- **Resolution:** Adjusted firewall rules (or disabled firewall temporarily in the lab) and re-pushed the client
- **Verification:** Client showed as "Active" in the console within a few minutes

### 2. Deployment Not Reaching the Client
- **Symptoms:** Application deployment showed "Unknown" status on the client
- **Reason:** Client hadn't retrieved new machine policy yet
- **Resolution:** Manually triggered "Machine Policy Retrieval & Evaluation Cycle" from the client's Configuration Manager control panel applet
- **Verification:** Deployment status updated to "Success" after the cycle ran

### 3. Software Update Point Sync Failure
- **Symptoms:** No updates appearing to deploy
- **Reason:** WSUS wasn't fully configured before enabling the SUP role
- **Resolution:** Reviewed WSUS configuration, corrected the sync source, and re-ran synchronization
- **Verification:** Update catalog populated with available updates in the console

### 4. Distribution Point Content Failure
- **Symptoms:** Package showed "Failed" status when distributing content
- **Reason:** Service account used by SCCM lacked permissions on the DP server
- **Resolution:** Corrected the service account permissions and redistributed content
- **Verification:** Content status showed "Success" on the Distribution Point

---

## 🔄 Typical Deployment Workflow

```
Create Collection (target devices/users)
             ↓
Import/Create Application or Update Package
             ↓
Distribute Content to Distribution Point
             ↓
Create Deployment (assign package to collection)
             ↓
Client Checks In and Receives Policy
             ↓
Client Downloads and Installs Package
             ↓
Status Reported Back to Console
             ↓
Review Deployment Success/Failure Reports
```

---

## ✅ Best Practices (What I Learned)

**Planning**
- Always test deployments on a small pilot collection before rolling out to a larger group.

**Boundaries**
- Get boundary groups right early — a lot of "client not showing up" issues trace back to boundary misconfiguration.

**Content Distribution**
- Confirm content successfully reaches Distribution Points before creating the deployment, not after.

**Patch Management**
- Stagger update deployments (test group → pilot group → full rollout) rather than pushing to everyone at once.

**Monitoring**
- Use the built-in deployment status monitoring regularly — don't assume success just because no error popped up.

**Documentation**
- Keep notes on service account permissions and prerequisite steps — SCCM has many moving parts, and small misconfigurations are the most common source of issues.

---

## 🎤 Interview Questions (Beginner Level)

**1. What is SCCM used for?**
Managing large numbers of Windows devices centrally — deploying software, patches, operating systems, and enforcing configuration compliance.

**2. What is the current official name for SCCM?**
Microsoft Configuration Manager (MECM/MCM) — SCCM is still the commonly used legacy name.

**3. What is a Distribution Point?**
A server role that stores and delivers software content to client devices during a deployment.

**4. What is a Collection in SCCM?**
A grouping of devices or users used to target specific deployments.

**5. What is a boundary in SCCM?**
A network location (like an IP range or subnet) that SCCM uses to determine which site a client belongs to.

**6. What is the Management Point used for?**
It's the primary point clients communicate with to receive policy and deployment instructions.

**7. What is the Software Update Point (SUP)?**
An SCCM role that integrates with WSUS to manage and deploy Windows updates.

**8. What database does SCCM rely on?**
Microsoft SQL Server.

**9. What is a Task Sequence used for?**
Automating multi-step processes, most commonly operating system deployment/imaging.

**10. How does a client typically get the SCCM agent installed?**
Through client push installation, software update-based installation, group policy, or manual installation.

**11. What would you check first if a device isn't showing up in the SCCM console?**
Whether the client is properly installed, and whether the device's IP falls within a configured boundary group.

**12. What is a Compliance Baseline?**
A set of configuration rules used to check whether devices meet a required standard, and optionally remediate them if they don't.

**13. Why would you use a pilot/test collection before a full deployment?**
To catch issues (compatibility, failures) on a small group of devices before impacting the entire organization.

**14. What log would you check if the Software Update Point isn't syncing?**
The `wsyncmgr.log` file, which tracks WSUS synchronization activity.

**15. What's the difference between an Application and a Package in SCCM?**
Applications use more advanced detection logic (checking if something is already installed) and support dependencies/requirements; Packages are a simpler, older method of deploying content with a defined program to run.

**16. Why might a deployment show "Unknown" status on a client?**
The client likely hasn't checked in or retrieved the latest policy yet — often fixed by triggering a policy retrieval cycle.

**17. What's the purpose of the SCCM console?**
It's the central administrative interface used to configure the environment, create deployments, and monitor status/reports.

**18. What role does Active Directory play in an SCCM environment?**
It's commonly used for device/user discovery and organizing systems that SCCM will manage.

**19. Why is content distribution to a Distribution Point a required step before deployment?**
Because clients pull the actual software/update files from the Distribution Point — if content isn't there, the deployment will fail even if policy is received.

**20. What's a good first troubleshooting step for a failed deployment?**
Check the deployment status monitoring in the console for the specific error, then review the relevant client-side or server-side log file for more detail.

---

## 📝 My Learning Summary

Building SCCM in a home lab gave me a genuine appreciation for how much infrastructure sits behind "just pushing an app to 500 computers." Standing up Active Directory, SQL Server, and the SCCM site server as separate VMs, then getting boundaries, distribution points, and the client agent all working together, taught me how many moving parts have to align correctly for something as simple-sounding as a software deployment to actually succeed.

Troubleshooting client push failures, deployment status issues, and Software Update Point sync problems gave me hands-on experience with the kind of day-to-day issues an SCCM administrator deals with — even at a small lab scale.

This reflects self-directed lab practice, not professional job experience managing a production SCCM environment.
