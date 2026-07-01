# Day 8: Package Management & Software Installation

> **DevOps Transformation** | Linux Fundamentals Series

---

## 📋 Overview

Every Linux system needs software — web servers, monitoring agents, compilers, utilities, database clients. In Windows you download an installer and click through a wizard. In Linux, you use a **package manager** — a command-line tool that handles downloading, installing, configuring, and removing software automatically, including every dependency it needs.

Understanding package management is one of the last pieces of the foundational Linux toolkit. By the end of today, you'll be able to install, update, upgrade, and remove software on any Debian-based or Red Hat-based Linux system — which covers the vast majority of servers you'll encounter in a DevOps role.

---

## 🗺️ What You'll Learn Today

| Topic | Commands | Why It Matters |
|---|---|---|
| What is a Package Manager? | — | Understand the system before using it |
| Debian-based Package Management | `apt` | Ubuntu, Debian — most common for DevOps |
| RPM-based Package Management | `yum`, `dnf` | RHEL, CentOS, Fedora — dominant in enterprise |
| apt vs yum Comparison | both | Choose the right tool for the right system |
| Update vs Upgrade | `update`, `upgrade` | Critical distinction for system maintenance |
| Logs and Verification | `/var/log/` | Audit what changed and when |

---

## 🏢 NexusCorp Scenario

> NexusCorp runs a mixed infrastructure: their developer workstations and CI/CD containers run Ubuntu, while their production application servers run Red Hat Enterprise Linux (RHEL). A new monitoring agent needs to be deployed across both environments, and the ops team needs to ensure all systems are patched before a compliance audit next week.
>
> Today you learn how to handle both environments — and how to do it safely without breaking running services.

---

## Part 1: Introduction to Package Managers

### What Is a Package Manager?

A **package manager** is a system utility that automates the process of installing, upgrading, configuring, and removing software on a Linux system. Software is distributed in **packages** — compressed archives that contain the program's files, metadata, and instructions for installation.

Without a package manager, installing software on Linux would mean:
1. Manually downloading source code or binaries
2. Figuring out every library and dependency the software needs
3. Downloading and installing each dependency manually, in the right order
4. Dealing with version conflicts between different programs

Package managers handle all of this automatically.

---

### What a Package Manager Does

```
┌─────────────────────────────────────────────────────────┐
│               Package Manager Workflow                   │
│                                                         │
│  User Command                                           │
│  (apt install nginx)                                    │
│         │                                               │
│         ▼                                               │
│  Query package repository ──► Find nginx + dependencies │
│         │                                               │
│         ▼                                               │
│  Download packages from repository servers              │
│         │                                               │
│         ▼                                               │
│  Verify integrity (checksums, signatures)               │
│         │                                               │
│         ▼                                               │
│  Install files to correct system locations              │
│         │                                               │
│         ▼                                               │
│  Run post-install scripts (start service, set perms)    │
│         │                                               │
│         ▼                                               │
│  Register package in local package database             │
└─────────────────────────────────────────────────────────┘
```

### Key Benefits

| Benefit | Description |
|---|---|
| **Dependency resolution** | Automatically finds and installs every library the software needs |
| **Centralized repositories** | Software comes from trusted, vetted sources — not random websites |
| **Easy removal** | Uninstalls cleanly, including dependencies no longer needed |
| **System-wide updates** | Update every piece of installed software in one command |
| **Rollback capability** | Some package managers support downgrading to a previous version |
| **Integrity verification** | Packages are signed and checksummed — tampered packages are rejected |

---

### The Two Major Package Manager Families

Linux distributions fall into two main packaging families, each with its own package format and tooling:

| Family | Package Format | Distributions | Package Manager |
|---|---|---|---|
| **Debian-based** | `.deb` | Ubuntu, Debian, Linux Mint, Kali | `apt`, `dpkg` |
| **RPM-based** | `.rpm` | RHEL, CentOS, Fedora, Amazon Linux | `yum`, `dnf`, `rpm` |

**Which one will you use?**
- Ubuntu (most common for personal Linux, containers, CI) → `apt`
- RHEL / CentOS / Amazon Linux (dominant in enterprise production) → `yum` or `dnf`

Knowing both is essential for a DevOps engineer — you'll almost certainly work with both ecosystems.

---

## Part 2: `apt` — Advanced Package Tool (Debian/Ubuntu)

`apt` is the primary package manager for Debian-based distributions. It is a higher-level frontend that wraps the lower-level `dpkg` tool, adding dependency resolution, repository management, and a cleaner interface.

---

### Repository Configuration

`apt` downloads packages from **repositories** — servers that host catalogued, signed software. Repository sources are defined in:

```
/etc/apt/sources.list
/etc/apt/sources.list.d/   ← Additional repository files go here
```

To view your configured repositories:
```bash
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
```

---

### Core `apt` Commands

#### Installing Software

```bash
# Install a single package
sudo apt install nginx

# Install multiple packages at once
sudo apt install nginx curl wget git

# Install without confirmation prompt (-y = assume yes)
sudo apt install -y nodejs

# Install a specific version
sudo apt install nginx=1.18.0-0ubuntu1
```

When you install a package, `apt` automatically identifies and installs every dependency it needs. You'll see output like:

```
The following additional packages will be installed:
  libgd3 libxpm4 libxslt1.1
The following NEW packages will be installed:
  libgd3 libxpm4 libxslt1.1 nginx
0 upgraded, 4 newly installed, 0 to remove.
```

---

#### Removing Software

```bash
# Remove a package (keeps config files)
sudo apt remove nginx

# Remove a package AND its configuration files
sudo apt purge nginx

# Remove packages that were installed as dependencies
# and are no longer needed by anything
sudo apt autoremove

# Full cleanup — remove + purge orphaned packages
sudo apt autoremove --purge
```

**`remove` vs `purge`:** Use `remove` when you might reinstall the package later and want to keep your configuration. Use `purge` for a clean, complete removal.

---

#### Searching and Inspecting Packages

```bash
# Search for a package by name or description
apt search nginx

# Show detailed information about a package (before or after installing)
apt show nginx

# List all installed packages
apt list --installed

# List all installed packages and filter by name
apt list --installed | grep nginx

# List packages with available upgrades
apt list --upgradable

# Check which package provides a specific file
dpkg -S /usr/bin/python3
```

---

#### Updating Package Lists

```bash
# Fetch the latest package list from all configured repositories
sudo apt update
```

This does **not** install or upgrade anything. It only refreshes the local database of what packages are available and at what versions. You should always run this before installing or upgrading packages.

---

#### Upgrading Packages

```bash
# Upgrade all installed packages to their latest available versions
sudo apt upgrade

# Upgrade a specific package only
sudo apt upgrade nginx

# Full upgrade — may also add or remove packages to resolve dependencies
sudo apt full-upgrade

# Perform update then upgrade in one line (common pattern in scripts)
sudo apt update && sudo apt upgrade -y
```

---

#### Checking Logs

`apt` logs all activity to:

```bash
# Main apt log (high-level: install, upgrade, remove events)
cat /var/log/apt/history.log

# Detailed dpkg log (every file operation during installs/removals)
cat /var/log/dpkg.log

# View the most recent 50 lines
tail -50 /var/log/apt/history.log

# Find all installs in the log
grep "^Install" /var/log/apt/history.log
```

---

### Common `apt` Flag Reference

| Command / Flag | Description |
|---|---|
| `apt install <pkg>` | Install a package |
| `apt remove <pkg>` | Remove a package, keep config |
| `apt purge <pkg>` | Remove a package and its config files |
| `apt autoremove` | Remove unused dependency packages |
| `apt update` | Refresh package lists from repositories |
| `apt upgrade` | Upgrade all upgradable packages |
| `apt full-upgrade` | Upgrade, allowing dependency changes |
| `apt search <term>` | Search for packages by name/description |
| `apt show <pkg>` | Display detailed package information |
| `apt list --installed` | List all installed packages |
| `apt list --upgradable` | List packages with available updates |
| `-y` | Skip confirmation prompts |
| `-q` | Quiet — reduce output verbosity |

---

## Part 3: `yum` / `dnf` — RPM-Based Package Management

`yum` (**Y**ellowdog **U**pdater, **M**odified) is the traditional package manager for Red Hat-based Linux distributions. On modern Fedora and RHEL 8+ systems, it has been succeeded by `dnf` (**D**andified **Y**um), which is faster and has better dependency resolution — but the command syntax is nearly identical. Both are worth knowing.

---

### Repository Configuration

`yum` uses `.repo` files to define software sources:

```
/etc/yum.repos.d/       ← Repository configuration files
/etc/yum.conf           ← Global yum configuration
```

```bash
# List all configured repositories
yum repolist

# List all repos including disabled ones
yum repolist all

# View a specific repo file
cat /etc/yum.repos.d/redhat.repo
```

---

### Core `yum` Commands

#### Installing Software

```bash
# Install a single package
sudo yum install nginx

# Install multiple packages at once
sudo yum install nginx curl wget git

# Install without confirmation (-y = assume yes)
sudo yum install -y nodejs

# Install a specific version
sudo yum install nginx-1.20.1

# Install a local .rpm file
sudo yum install ./package.rpm
```

---

#### Removing Software

```bash
# Remove a package
sudo yum remove nginx

# Remove a package and its unused dependencies
sudo yum autoremove nginx

# Clean cached package data
sudo yum clean all
```

---

#### Searching and Inspecting Packages

```bash
# Search for a package
yum search nginx

# Show detailed information about a package
yum info nginx

# List all installed packages
yum list installed

# List installed packages matching a pattern
yum list installed | grep nginx

# List packages available to upgrade
yum list updates

# Find which package provides a specific file or command
yum provides /usr/bin/python3
```

---

#### Updating and Upgrading

```bash
# Check for available updates (no install)
yum check-update

# Update a specific package
sudo yum update nginx

# Update all installed packages
sudo yum update

# Update and remove obsolete packages
sudo yum update --obsoletes
```

> **Note on `yum` terminology:** In `yum`, `update` does what `apt` calls `upgrade` — it actually installs newer versions of packages. `yum` does not have a separate "refresh list only" step equivalent to `apt update`; it checks repositories at runtime.

---

#### Checking Logs

```bash
# yum transaction history
yum history

# Full details of a specific transaction (replace N with transaction ID)
yum history info N

# Undo a specific transaction (rollback)
sudo yum history undo N

# Raw yum log file
cat /var/log/yum.log

# On RHEL 8+ / dnf systems
cat /var/log/dnf.log
dnf history
```

---

### `dnf` — The Modern Replacement

On RHEL 8+, Fedora, and CentOS Stream, `dnf` replaces `yum`. The syntax is nearly identical — you can substitute `dnf` for `yum` in almost every command above:

```bash
sudo dnf install nginx
sudo dnf update
sudo dnf remove nginx
dnf list installed
dnf search nginx
dnf history
```

`dnf` improvements over `yum`:
- Faster dependency resolution
- Better error messages
- Cleaner API for plugins
- Native support for modular repositories

---

## Part 4: `apt` vs `yum` — Side-by-Side Comparison

| Aspect | `apt` (Debian/Ubuntu) | `yum` / `dnf` (RHEL/CentOS) |
|---|---|---|
| **Package format** | `.deb` | `.rpm` |
| **Low-level tool** | `dpkg` | `rpm` |
| **Distributions** | Ubuntu, Debian, Mint, Kali | RHEL, CentOS, Fedora, Amazon Linux |
| **Repo config** | `/etc/apt/sources.list` | `/etc/yum.repos.d/*.repo` |
| **Install** | `apt install <pkg>` | `yum install <pkg>` |
| **Remove** | `apt remove <pkg>` | `yum remove <pkg>` |
| **Search** | `apt search <term>` | `yum search <term>` |
| **Package info** | `apt show <pkg>` | `yum info <pkg>` |
| **Refresh repos** | `apt update` (separate step) | Automatic at runtime |
| **Upgrade all** | `apt upgrade` | `yum update` |
| **List installed** | `apt list --installed` | `yum list installed` |
| **Transaction rollback** | Limited | `yum history undo N` |
| **Log file** | `/var/log/apt/history.log` | `/var/log/yum.log` |

---

### When to Choose Which

| Scenario | Use |
|---|---|
| Ubuntu or Debian servers, CI/CD containers | `apt` |
| RHEL, CentOS, Amazon Linux production servers | `yum` / `dnf` |
| Enterprise environments with support contracts | `yum` / `dnf` (RHEL ecosystem) |
| Docker-based development environments | `apt` (most base images are Ubuntu/Debian) |
| Any cloud instance — check the distro first | `cat /etc/os-release` |

**How to identify your distribution:**
```bash
# Works on any systemd-based Linux
cat /etc/os-release

# Legacy alternative
cat /etc/redhat-release   # RHEL/CentOS
lsb_release -a            # Ubuntu/Debian
```

---

## Part 5: Update vs. Upgrade — A Critical Distinction

This is one of the most common sources of confusion for Linux newcomers, and an important concept to understand clearly before touching a production system.

---

### In `apt` (Debian/Ubuntu)

| Command | What It Does | Installs Anything? |
|---|---|---|
| `sudo apt update` | Downloads the latest package list from repositories | **No** |
| `sudo apt upgrade` | Installs newer versions of already-installed packages | **Yes** |
| `sudo apt full-upgrade` | Like upgrade, but can also add/remove packages if needed | **Yes** |

Think of it as a two-step process:

```
Step 1: apt update
        └── "Let me check what's new in the repos"
            Downloads: package names, versions, checksums
            Installs:  nothing

Step 2: apt upgrade
        └── "Now install the newer versions I just found out about"
            Compares: installed versions vs latest in updated list
            Installs:  new versions of packages with available updates
```

You **must** run `apt update` before `apt upgrade` — otherwise, `apt` will upgrade packages against a potentially stale list and might miss recently released updates.

---

### In `yum` (RHEL/CentOS)

`yum` combines the two steps: `yum update` both checks the repositories and installs available updates in one command. There is no separate "refresh list only" step in `yum`.

| Command | What It Does |
|---|---|
| `yum check-update` | Check for updates without installing |
| `sudo yum update` | Download updated package lists AND install updates |

---

### Why This Matters in Production

The update/upgrade distinction matters most when managing production systems:

| Risk | Description |
|---|---|
| **Breaking changes** | A major version upgrade might change a config file format or remove a deprecated flag your scripts use |
| **Service restarts** | Some upgrades require restarting services — plan for downtime or rolling restarts |
| **Dependency conflicts** | Upgrading one package might pull in a new version of a shared library that breaks another package |
| **Kernel updates** | Upgrading the kernel requires a reboot — not something to do on a live production system without coordination |

**Best practice for production:**

```bash
# 1. Check what would be upgraded BEFORE committing to it
apt list --upgradable
# or
yum check-update

# 2. Upgrade a specific package (not everything at once)
sudo apt upgrade nginx
# or
sudo yum update nginx

# 3. If upgrading everything, do it in a maintenance window
sudo apt update && sudo apt upgrade

# 4. Verify afterward
apt list --installed | grep nginx
```

---

## 🛠️ Practical Exercises

### Exercise 1: Install and Explore a Package

```bash
# On Ubuntu/Debian:

# Step 1: Update the package list first
sudo apt update

# Step 2: Install 'tree' (a useful directory visualizer)
sudo apt install tree

# Step 3: Verify the install
tree --version
which tree

# Step 4: Explore what the package contains
dpkg -L tree

# Step 5: List all installed packages
apt list --installed

# Step 6: Filter to confirm 'tree' is in the list
apt list --installed | grep tree
```

```bash
# On RHEL/CentOS:

sudo yum install tree
tree --version
yum list installed | grep tree
```

---

### Exercise 2: Update → Upgrade Workflow

```bash
# On Ubuntu/Debian:

# Step 1: Refresh the package list
sudo apt update

# Check output carefully — it tells you how many packages can be upgraded

# Step 2: See what can be upgraded
apt list --upgradable

# Step 3: Upgrade a single package safely
sudo apt upgrade curl

# Step 4: Perform a full system upgrade
sudo apt upgrade

# Step 5: Verify the logs
tail -30 /var/log/apt/history.log
grep "^Upgrade" /var/log/apt/history.log | tail -10
```

```bash
# On RHEL/CentOS:

# Step 1: Check for available updates
yum check-update

# Step 2: Upgrade a specific package
sudo yum update curl

# Step 3: Perform a full update
sudo yum update

# Step 4: Check history
yum history
yum history info 1
```

---

### Exercise 3: Remove and Clean Up

```bash
# On Ubuntu/Debian:

# Step 1: Remove the 'tree' package
sudo apt remove tree

# Verify it's gone
apt list --installed | grep tree
which tree   # Should return nothing

# Step 2: Check for orphaned dependencies
sudo apt autoremove

# Step 3: For a full purge (removes config files too)
sudo apt purge tree
```

---

## 🔧 Mini-Project: NexusCorp System Readiness Check

**Scenario:** Before a compliance audit, the NexusCorp ops team needs to verify all their Ubuntu staging servers are up to date, install a required monitoring utility, and document what changed.

```bash
# Step 1: Record the system state before changes
echo "=== Pre-Update Package Count ===" > /tmp/nexus_audit.txt
apt list --installed 2>/dev/null | wc -l >> /tmp/nexus_audit.txt

# Step 2: Identify your distribution
cat /etc/os-release | grep PRETTY_NAME

# Step 3: Refresh package sources
sudo apt update 2>&1 | tee -a /tmp/nexus_audit.txt

# Step 4: Check what can be upgraded
echo "=== Upgradable Packages ===" >> /tmp/nexus_audit.txt
apt list --upgradable 2>/dev/null >> /tmp/nexus_audit.txt

# Step 5: Install the required monitoring utility
sudo apt install -y htop

# Step 6: Perform system upgrade
sudo apt upgrade -y

# Step 7: Clean up unused packages
sudo apt autoremove -y

# Step 8: Record post-update state
echo "=== Post-Update Package Count ===" >> /tmp/nexus_audit.txt
apt list --installed 2>/dev/null | wc -l >> /tmp/nexus_audit.txt

# Step 9: Review the log
echo "=== Recent apt History ===" >> /tmp/nexus_audit.txt
tail -20 /var/log/apt/history.log >> /tmp/nexus_audit.txt

# Step 10: Display the full audit report
cat /tmp/nexus_audit.txt
```

**Bonus Challenges:**
- Find out which package provides the `curl` binary: `dpkg -S $(which curl)` on Debian, or `yum provides curl` on RHEL
- Search for all packages related to "python3" using `apt search python3 | head -20`
- Look up the size of an installed package before removal: `apt show tree | grep Size`

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Package Manager** | A utility that automates installing, updating, configuring, and removing software on a Linux system |
| **Package** | A compressed archive containing software files, metadata, and installation instructions |
| **Repository (Repo)** | A server that hosts a collection of signed, catalogued packages available for download |
| **apt** | Advanced Package Tool — the primary package manager for Debian and Ubuntu |
| **yum** | Yellowdog Updater, Modified — the traditional package manager for Red Hat-based distributions |
| **dnf** | Dandified Yum — the modern replacement for `yum` on RHEL 8+, Fedora, and CentOS Stream |
| **dpkg** | Low-level Debian package tool — `apt` wraps this for everyday use |
| **rpm** | Low-level RPM package tool — `yum`/`dnf` wrap this for everyday use |
| **`.deb`** | Package file format used by Debian-based distributions |
| **`.rpm`** | Package file format used by Red Hat-based distributions |
| **Dependencies** | Other packages that a given package requires in order to function correctly |
| **Update** | Refreshing the local package list from repositories — installs nothing |
| **Upgrade** | Installing newer versions of already-installed packages |
| **`apt update`** | Downloads the latest package metadata from configured repositories |
| **`apt upgrade`** | Installs newer versions of installed packages based on the refreshed package list |
| **`purge`** | Removes a package along with its configuration files |
| **`autoremove`** | Removes packages that were installed as dependencies but are no longer required |
| **Transaction History** | A log of package manager operations — useful for auditing and rollbacks |

---

## 📚 Resources

- [Tecmint — Linux Package Managers Guide](https://www.tecmint.com/linux-package-management/) — practical overview of package management across distributions
- [apt Man Page](https://manpages.debian.org/bullseye/apt/apt.8.en.html) — official Debian apt reference
- [yum Man Page](https://man7.org/linux/man-pages/man8/yum.8.html) — official yum command reference
- [dnf Documentation](https://dnf.readthedocs.io/en/latest/) — official dnf reference for RHEL 8+ / Fedora
- [Understanding apt vs apt-get](https://itsfoss.com/apt-vs-apt-get-difference/) — clarifies the relationship between `apt` and the older `apt-get`

---

## 🔭 Day 9 Preview: Networking Basics

Tomorrow you move from managing software on a single machine to understanding how Linux communicates across a network.

You'll learn:
- `ip`, `ifconfig` — view and configure network interfaces
- `ping`, `traceroute` — test connectivity and trace network paths
- `ss`, `netstat` — see what's listening on which ports
- `curl`, `wget` — make HTTP requests and download files from the command line
- How NexusCorp diagnoses network issues between its application and database servers

You now know how to get software onto a system. Next, you'll make those systems talk to each other.

---

> 💬 *"A package manager is your system's supply chain. Keep it updated, keep it audited, keep it clean."*
>
> — DevOps learing by Yukta
