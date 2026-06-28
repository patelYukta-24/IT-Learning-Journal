# 🐧 DAY 4 — Introduction to Linux & Navigating the Filesystem

> **NexusCorp DevOps Transformation Series**
> _"Before you can automate the world, you need to understand the ground you stand on."_

**Skill Level:** Absolute Beginner
**Estimated Time:** 6–8 Hours
**Approach:** Read → Understand → Practice → Build

---

## 📋 Table of Contents

1. [Day 4 Overview & Goals](#1-day-4-overview--goals)
2. [Brief History of Linux & Its Variants](#2-brief-history-of-linux--its-variants)
3. [Linux Distros — The Ecosystem Map](#3-linux-distros--the-ecosystem-map)
4. [Installing Linux on Your System](#4-installing-linux-on-your-system)
5. [VM vs Dual Boot vs Cloud — Which Should You Choose?](#5-vm-vs-dual-boot-vs-cloud--which-should-you-choose)
6. [Navigating the Linux Filesystem](#6-navigating-the-linux-filesystem)
7. [File Types & Structure](#7-file-types--structure)
8. [Absolute vs Relative Paths](#8-absolute-vs-relative-paths)
9. [Special Directories & Shortcuts](#9-special-directories--shortcuts)
10. [Essential Navigation Commands — Deep Dive](#10-essential-navigation-commands--deep-dive)
11. [Practical Exercises](#11-practical-exercises)
12. [Mini-Project: NexusCorp Onboarding Setup](#12-mini-project-nexuscorp-onboarding-setup)
13. [Cheat Sheet](#13-cheat-sheet)
14. [Glossary](#14-glossary)
15. [Resources](#15-resources)
16. [Day 4 Mastery Checklist](#16-day-4-mastery-checklist)
17. [What's Coming — Day 5 Preview](#17-whats-coming--day-5-preview)

---

## 1. Day 4 Overview & Goals

### Why This Day Matters

Imagine being handed the keys to a car you've never seen — with no dashboard labels, no steering wheel labels, nothing. That's Linux without context. Before you can run pipelines, deploy containers, or automate infrastructure, you need two things:

1. **A system to work on** (your Linux installation)
2. **The ability to navigate it** (filesystem fundamentals)

Today covers both. By tonight, you'll have Linux running and you'll be moving around its filesystem with confidence.

### Learning Goals

By the end of Day 4, you will be able to:

- [ ] Explain the history of Linux from UNIX to modern distros
- [ ] Identify at least 6 Linux distributions and their intended use cases
- [ ] Install Linux using VirtualBox (or an equivalent method)
- [ ] Navigate the Linux filesystem using `cd`, `ls`, `pwd`, and related commands
- [ ] Explain the difference between absolute and relative paths
- [ ] Understand what `~`, `.`, and `..` represent
- [ ] Create directories and files from the terminal
- [ ] Identify the 7 Linux file types

### NexusCorp Context

> NexusCorp's developers had zero Linux experience when the DevOps transformation began. Every pipeline, every container, every cloud server they would eventually manage runs on Linux. Day 4 is where their journey — and yours — officially begins.

---

## 2. Brief History of Linux & Its Variants

### 2.1 Where It All Started — UNIX (1969)

```
Bell Labs, Murray Hill, New Jersey — 1969
Ken Thompson + Dennis Ritchie + team
→ Created UNIX: a portable, multi-user, multitasking operating system
→ Written in C (not assembly), making it portable across hardware
→ Introduced concepts still used today: filesystems, pipes, shell, permissions
```

UNIX was revolutionary. It introduced ideas that became the blueprint for nearly every modern operating system: the concept that everything is a file, hierarchical directories, user permissions, and the shell as an interface.

However, UNIX was proprietary. Universities could study it, but could not freely distribute or modify it.

### 2.2 The GNU Project — The Freedom Movement (1983)

```
1983 — Richard Stallman, MIT
→ Starts the GNU Project ("GNU's Not Unix")
→ Goal: Create a fully free UNIX-compatible OS
→ Creates foundational tools: gcc, bash, glibc, make, gzip
→ Missing: A free kernel
```

GNU stood for "GNU's Not Unix" — a recursive acronym. Stallman's vision was an OS where users had the freedom to run, study, modify, and distribute software. He called these the **Four Freedoms**.

By the early 1990s, GNU had almost everything needed for a complete OS — except a stable, free kernel.

### 2.3 The Linux Kernel — Linus Torvalds (1991)

```
1991 — University of Helsinki, Finland
Linus Torvalds, 21 years old
```

On August 25, 1991, Linus Torvalds posted this message to the Minix newsgroup:

```
"Hello everybody out there using minix —
I'm doing a (free) operating system (just a hobby, won't be big and professional
like gnu) for 386(486) AT clones."
```

This "hobby project" became the Linux kernel — the missing piece GNU needed. Combined together:

```
GNU tools + Linux kernel = GNU/Linux
(What people call "Linux" today)
```

### 2.4 Key Milestones Timeline

```
1969  ────── UNIX created at Bell Labs
              │
1983  ────── GNU Project launched (Stallman)
              │
1991  ────── Linux kernel v0.01 (Torvalds, hobby project)
              │  Linux released under GPL (free forever)
1992  ────── GNU + Linux = functional free OS
              │
1993  ────── First distros: Slackware, Debian released
              │
1994  ────── Linux kernel v1.0 released
              │
1995  ────── Red Hat Linux released (enterprise focus)
              │
1996  ────── Tux the penguin becomes Linux mascot
              │  Linux kernel v2.0 — SMP (multiple CPUs)
2000  ────── IBM invests $1 BILLION in Linux
              │
2003  ────── Fedora Project starts (community Red Hat)
              │
2004  ────── Ubuntu released (user-friendly, free)
              │
2005  ────── Git created by Linus Torvalds
              │
2007  ────── CentOS gains major enterprise adoption
              │
2008  ────── Android (Linux-based) launches on smartphones
              │
2010  ────── Linux powers 60%+ of top 500 supercomputers
              │
2013  ────── Docker launches — Linux containers go mainstream
              │
2014  ────── Microsoft: "Microsoft Loves Linux"
              │  Azure runs Linux on more VMs than Windows
2016  ────── Windows Subsystem for Linux (WSL) released
              │
2019  ────── Linux powers 100% of Top 500 supercomputers
              │
2020  ────── WSL2 with full kernel released
              │
Today ────── Linux powers:
              │  • 96%+ of web servers
              │  • 90%+ of cloud infrastructure
              │  • All 500 Fortune 500 companies
              │  • 3+ billion Android devices
              │  • The International Space Station
```

### 2.5 Why "Linux" Won

Three forces combined to make Linux dominant:

**1. The GPL License**
The GNU General Public License (GPL) means: you can use, modify, and distribute Linux freely — but any modifications must also be free. This created a virtuous cycle of contributions.

**2. The Internet**
Linux development was distributed across thousands of developers worldwide, coordinated via email and later Git. No other OS had this scale of collaboration.

**3. The Right Timing**
When the internet and server boom hit in the late 1990s, enterprises needed a stable, free, scalable OS. Linux was there. Windows was too expensive; proprietary UNIX was too locked-in.

---

## 3. Linux Distros — The Ecosystem Map

### 3.1 What is a Distro?

The Linux kernel alone is not an OS. A **distribution (distro)** bundles:

```
Linux Kernel
    +
GNU tools (bash, ls, grep, gcc...)
    +
Package manager (apt, yum, pacman...)
    +
Init system (systemd, SysV...)
    +
Desktop environment (GNOME, KDE, XFCE...) [optional]
    +
Pre-configured software
    =
Linux Distribution
```

### 3.2 The Distro Family Tree

```
                    LINUX KERNEL
                         │
          ┌──────────────┼──────────────┐
          │              │              │
     Debian          Red Hat          Arch
     Family          Family          Family
          │              │              │
    ┌─────┴─────┐  ┌─────┴──────┐  ┌──┴──────────┐
    │           │  │            │  │             │
 Debian      Ubuntu  RHEL    Fedora  Arch      Manjaro
 (stable)  (popular)│       (test)   (DIY)    (easier)
              │     ├─CentOS Stream
           ├─ Ubuntu   ├─AlmaLinux
           │   Server  └─Rocky Linux
           ├─ Xubuntu
           ├─ Lubuntu
           └─ Linux Mint (via Ubuntu)
                              │
                      ┌───────┘
                   Others
             ┌────────┼────────┐
           SUSE     Alpine    Kali
          (enterprise)(containers)(security)
```

### 3.3 Distros by Use Case

#### 🖥️ Desktop / Beginner-Friendly

| Distro | Best For | Package Mgr | Notes |
|--------|---------|-------------|-------|
| **Ubuntu** | General purpose, beginners | `apt` | Most tutorials written for Ubuntu |
| **Linux Mint** | Windows switchers | `apt` | Very familiar interface |
| **Fedora** | Developers, latest features | `dnf` | GNOME desktop, bleeding-edge |
| **Manjaro** | Power users (Arch made easy) | `pacman` | Rolling release |
| **elementary OS** | macOS switchers | `apt` | Beautiful, simple |
| **Pop!_OS** | Developers, gamers | `apt` | By System76, great GPU support |

#### 🖧 Server / Enterprise

| Distro | Best For | Package Mgr | Notes |
|--------|---------|-------------|-------|
| **Ubuntu Server** | Cloud, DevOps, startups | `apt` | Default on AWS, GCP, Azure |
| **RHEL** | Enterprise critical systems | `dnf` | Paid support from Red Hat |
| **AlmaLinux** | RHEL replacement (free) | `dnf` | CentOS successor |
| **Rocky Linux** | RHEL replacement (free) | `dnf` | By CentOS original founder |
| **Debian** | Stability-first servers | `apt` | Very conservative release cycle |
| **CentOS Stream** | Upstream RHEL testing | `dnf` | Rolling, not 1:1 with RHEL |

#### 🔐 Specialized Distros

| Distro | Specialization | Notes |
|--------|--------------|-------|
| **Kali Linux** | Penetration testing & cybersecurity | 600+ security tools pre-installed |
| **Parrot OS** | Security + privacy | Lighter than Kali |
| **Tails** | Privacy & anonymity | Runs from USB, leaves no trace |
| **Alpine Linux** | Containers (Docker images) | Tiny ~5MB base image |
| **Raspberry Pi OS** | Raspberry Pi hardware | ARM-optimized |
| **VyOS** | Network routing | Router/firewall OS |
| **Proxmox VE** | Virtualization | Hypervisor built on Debian |
| **FreeNAS/TrueNAS** | NAS / storage | Built on FreeBSD (Unix, not Linux) |

#### ☁️ Cloud-Optimized

| Distro | Cloud Provider | Notes |
|--------|--------------|-------|
| **Amazon Linux 2** | AWS | Optimized for EC2, free |
| **CoreOS / Flatcar** | K8s, containers | Minimal, auto-updating |
| **Bottlerocket** | AWS + K8s | Container-focused, by Amazon |
| **Google COS** | Google Cloud | Container-optimized OS |

### 3.4 Which Distro for This Course?

> **Recommendation: Ubuntu 22.04 LTS**

Why:
- Largest community = most Stack Overflow answers
- Default on AWS Free Tier (what you'll use in cloud sections)
- Best documentation for beginners
- LTS = Long-Term Support (supported until April 2027)
- Every tool in this course has Ubuntu instructions

---

## 4. Installing Linux on Your System

### 4.1 Method 1: VirtualBox (Recommended for Beginners)

VirtualBox lets you run Linux inside your current OS as a window. Nothing about your current system changes. Perfect for learning.

#### Step-by-Step VirtualBox Installation

**Prerequisites:**
- 8 GB RAM (minimum; 16 GB recommended)
- 30 GB free disk space
- 64-bit processor with virtualization enabled in BIOS/UEFI

---

**STEP 1: Enable Virtualization in BIOS/UEFI**

```
1. Restart your computer
2. Press the BIOS key during startup:
   - Dell:   F2 or F12
   - HP:     F10 or Esc
   - Lenovo: F1 or F2
   - ASUS:   Del or F2
3. Find: "Intel VT-x" or "AMD-V" or "SVM Mode"
4. Set to: ENABLED
5. Save and exit (usually F10)
```

---

**STEP 2: Download VirtualBox**

```
1. Go to: https://www.virtualbox.org/wiki/Downloads
2. Download for your OS:
   → "Windows hosts" (if on Windows)
   → "macOS hosts" (if on Mac)
   → "Linux hosts" (if already on Linux)
3. Run the installer with all defaults
4. Also download: "VirtualBox Extension Pack" from same page
   → Open VirtualBox → File → Preferences → Extensions → Add
```

---

**STEP 3: Download Ubuntu 22.04 LTS ISO**

```
1. Go to: https://ubuntu.com/download/server
   (or https://ubuntu.com/download/desktop for GUI)
2. Click "Download Ubuntu 22.04.X LTS"
3. Save the .iso file (about 1.4 GB)
   DO NOT extract or open it — leave it as .iso
```

---

**STEP 4: Create the Virtual Machine**

```
In VirtualBox:
1. Click "New" (blue star icon)

2. Name and operating system:
   Name:    Ubuntu-22.04
   Folder:  (choose where VM files will be saved)
   ISO Image: Browse → select your .iso file
   ☑ Skip Unattended Installation (for learning; do it manually)
   Click: Next

3. Hardware:
   Base Memory:  4096 MB   (4 GB minimum; 8 GB if you have it)
   Processors:   2         (more if your CPU has more cores)
   Click: Next

4. Virtual Hard Disk:
   ● Create a Virtual Hard Disk Now
   Disk Size: 25.00 GB    (30 GB better for more practice)
   ☑ Pre-allocate Full Size → leave UNCHECKED (dynamic is fine)
   Click: Next → Finish
```

---

**STEP 5: Configure VM Settings (Before First Boot)**

```
Right-click your VM → Settings

● System → Processor:
  ☑ Enable PAE/NX

● Display → Screen:
  Video Memory: 128 MB
  ☑ Enable 3D Acceleration (optional)

● Network → Adapter 1:
  ● Attached to: NAT
  (Gives internet access; safe default)

● Storage:
  Verify ISO is attached under "Controller: IDE"
  If not: click CD icon → "Choose a disk file" → your .iso

Click: OK
```

---

**STEP 6: Install Ubuntu**

```
1. Start the VM (green arrow)
2. Ubuntu installer loads (wait 30-60 seconds)

For Ubuntu Server:
   ● Language: English → Enter
   ● Update installer? → Continue without updating (faster)
   ● Keyboard layout: Detect → or select yours
   ● Installation type: Ubuntu Server → Done
   ● Network: leave default (DHCP) → Done
   ● Proxy: leave blank → Done
   ● Mirror: leave default → Done
   ● Storage layout: Use entire disk → Done → Continue
   ● Profile setup:
       Your name:          Alex DevOps
       Server name:        nexuscorp-dev
       Username:           alex          ← remember this!
       Password:           ••••••••      ← remember this!
       Confirm password:   ••••••••
   ● SSH: ☑ Install OpenSSH server → Done
   ● Featured snaps: skip (space to deselect) → Done
   ● Wait for installation (5–15 minutes)
   ● "Install complete!" → Reboot Now
   ● Press Enter when prompted to remove installer
```

---

**STEP 7: First Boot & Update**

```bash
# Login with your username and password
# You'll see a terminal prompt like:
alex@nexuscorp-dev:~$

# First thing: update all packages
sudo apt update && sudo apt upgrade -y

# Install helpful tools
sudo apt install -y vim curl wget git tree net-tools

# Verify everything works
uname -a
lsb_release -a
whoami
```

---

**STEP 8: Install VirtualBox Guest Additions (Quality of Life)**

```bash
# In VM menu: Devices → Insert Guest Additions CD Image
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
sudo mount /dev/cdrom /mnt
sudo /mnt/VBoxLinuxAdditions.run
sudo reboot
```

After reboot: shared clipboard, drag-and-drop, and better display resolution work.

---

### 4.2 Method 2: VMware Workstation Player

VMware is an alternative to VirtualBox, often considered smoother on Windows:

```
1. Download VMware Workstation Player (free):
   https://www.vmware.com/products/workstation-player.html

2. Install VMware Player

3. Open VMware → "Create a New Virtual Machine"
   → "Installer disc image file (iso)" → Browse to Ubuntu ISO

4. Follow guided setup:
   - Full name: Alex DevOps
   - Username: alex
   - Password: ••••••••
   - VM Name: Ubuntu-22.04-LTS
   - Maximum disk size: 25 GB
   - Store as single file → Finish

5. VMware auto-installs VMware Tools (equivalent to Guest Additions)
   Better integration out of the box than VirtualBox
```

---

### 4.3 Method 3: WSL2 on Windows (Fastest Setup)

Windows Subsystem for Linux 2 runs a real Linux kernel inside Windows. No VM overhead, instant start:

```powershell
# Open PowerShell as Administrator

# Install WSL2 with Ubuntu (one command!)
wsl --install

# Reboot when prompted

# After reboot: Ubuntu launches automatically
# Create username and password

# To open later: search "Ubuntu" in Start menu
# Or: wsl in any terminal

# List available distros
wsl --list --online

# Install a different distro
wsl --install -d Debian
wsl --install -d kali-linux
```

**WSL2 limitations to know:**
- No systemd by default (can enable with `systemd=true` in `/etc/wsl.conf`)
- Some networking commands behave differently
- Perfect for learning; some DevOps tools work differently

---

### 4.4 Method 4: Dual Boot (Advanced)

Dual boot installs Linux alongside Windows on the same physical machine. You choose which OS to boot at startup.

```
WARNING: Incorrectly partitioning your disk can erase your Windows installation.
Back up all important data BEFORE attempting dual boot.
```

**High-level steps:**

```
1. BACKUP YOUR DATA (seriously, do it)

2. Create a bootable USB drive:
   - Download Rufus: https://rufus.ie/
   - Insert USB (8GB+)
   - Rufus → select Ubuntu ISO → Partition scheme: GPT → Start

3. Shrink Windows partition:
   - Windows Key + X → Disk Management
   - Right-click C: drive → Shrink Volume
   - Enter amount to shrink: 30720 MB (30 GB minimum)
   - Apply

4. Boot from USB:
   - Restart → press boot menu key (F12, F8, or Esc)
   - Select USB device

5. Ubuntu installer:
   - "Install Ubuntu alongside Windows Boot Manager"
   - Set partition size: use the free space you created
   - Complete installation

6. GRUB bootloader installs:
   - Every startup: choose Ubuntu or Windows
```

---

### 4.5 Method 5: Cloud Linux (Zero Local Setup)

If installation is not an option, use free cloud terminals:

```
Option A: Google Cloud Shell
→ https://shell.cloud.google.com
→ Free persistent ~5 GB Ubuntu environment
→ No account payment needed (just a Google account)
→ Full terminal, vim, git, Python, Node.js pre-installed

Option B: Killercoda
→ https://killercoda.com/learn
→ Browser-based Ubuntu/Kubernetes scenarios
→ No signup required
→ Resets after session (not persistent)

Option C: AWS Free Tier EC2
→ https://aws.amazon.com/free/
→ Create account → EC2 → Launch Ubuntu t2.micro
→ Connect via browser-based terminal
→750 hours/month free for 12 months

Option D: GitHub Codespaces
→ https://github.com/features/codespaces
→ Ubuntu-based VS Code environment in browser
→ 60 hours/month free
```

---

## 5. VM vs Dual Boot vs Cloud — Which Should You Choose?

### Comparison Matrix

| Factor | VirtualBox VM | Dual Boot | WSL2 | Cloud |
|--------|:---:|:---:|:---:|:---:|
| Difficulty | Medium | Hard | Easy | Easy |
| Risk to current OS | None | Medium | None | None |
| Performance | 70–80% native | 100% native | 90% native | Varies |
| Internet required | No | No | No | Yes |
| Persistence | Yes | Yes | Yes | Mostly |
| Cost | Free | Free | Free | Free tier |
| Good for DevOps tools | ✅ | ✅ | Mostly | ✅ |
| Recommended for beginners | ✅ | ❌ | ✅ | ✅ |

### Decision Flowchart

```
Do you have 8+ GB RAM?
├─ NO  → Use Cloud Shell or WSL2
└─ YES → Do you want to keep Windows?
          ├─ YES, always → VirtualBox VM or WSL2 (safest)
          └─ NO, I want full Linux → Dual Boot
                                      (back up first!)
```

### Pros & Cons Summary

**VirtualBox VM — Best for This Course**
```
✅ Zero risk to your main OS
✅ Easy to snapshot and restore if you break things
✅ Identical to a real Linux server environment
✅ Can run multiple distros simultaneously
❌ Requires RAM (4 GB for VM + your OS)
❌ Slightly slower than native
```

**Dual Boot**
```
✅ Full hardware performance
✅ Feels like a real Linux machine
❌ Risk of data loss if partitioning goes wrong
❌ Inconvenient to switch between OS
❌ Not recommended for complete beginners
```

**WSL2**
```
✅ Instant startup (seconds, not minutes)
✅ Excellent Windows integration
✅ Access Windows files from Linux
❌ Some system tools behave differently
❌ Not a full Linux experience (no desktop GUI by default)
```

**Cloud**
```
✅ No local resources needed
✅ Mirrors real-world DevOps environment
✅ Multiple free options
❌ Requires internet
❌ Some free tiers reset or expire
```

---

## 6. Navigating the Linux Filesystem

### 6.1 The Filesystem as a Map

Think of the Linux filesystem like a city:

```
/  (root) = The City Center — everything starts here

├── /home     = Residential Area — where users live
│                 Each user has their own house (/home/alice)
│
├── /etc      = City Hall — all configuration and rules
│
├── /bin      = The Toolshed — common tools everyone uses
│
├── /var/log  = The Newspaper Archives — record of everything
│
├── /tmp      = The Parking Lot — temporary, gets cleaned
│
└── /proc     = The Control Room — live data about the system
```

Unlike Windows (C:\, D:\ — multiple roots), Linux has ONE single root `/`. Everything — every file, every device, every network socket — hangs off that single tree.

### 6.2 The Complete Directory Structure

```
/                           Root of the entire filesystem
│
├── bin/                    Essential user binaries
│   ├── ls                  The ls command lives here
│   ├── cp, mv, rm          Common file tools
│   └── bash                The shell itself
│
├── boot/                   Boot loader files
│   ├── grub/               GRUB bootloader config
│   └── vmlinuz-*           The compressed Linux kernel
│
├── dev/                    Device files (hardware represented as files)
│   ├── sda                 First hard disk
│   ├── sda1                First partition of sda
│   ├── null                The black hole (discard anything sent here)
│   └── tty                 Terminal devices
│
├── etc/                    System-wide configuration files
│   ├── passwd              User account information
│   ├── shadow              Hashed passwords
│   ├── hosts               Static hostname → IP mapping
│   ├── fstab               Filesystem mount configuration
│   ├── crontab             Scheduled tasks
│   ├── ssh/sshd_config     SSH server configuration
│   └── nginx/nginx.conf    (example) App config files live here
│
├── home/                   Home directories for regular users
│   ├── alice/              Alice's home: /home/alice
│   │   ├── .bashrc         Alice's shell config (hidden file)
│   │   ├── .ssh/           SSH keys
│   │   └── projects/       Alice's project files
│   └── bob/                Bob's home: /home/bob
│
├── lib/                    Shared libraries for /bin and /sbin
├── lib64/                  64-bit libraries
│
├── media/                  Mount points for removable media
│   └── usb/                USB drive mounts here automatically
│
├── mnt/                    Temporary manual mount point
│   └── backup/             (example: manually mounted disk)
│
├── opt/                    Optional third-party applications
│   └── myapp/              Manually installed app
│
├── proc/                   Virtual filesystem: live kernel/process data
│   ├── cpuinfo             CPU information (read to see your CPU)
│   ├── meminfo             RAM information
│   ├── 1234/               Directory for process with PID 1234
│   └── version             Kernel version string
│
├── root/                   Root user's home directory
│   └── (root's files)      NOT /home/root — it's just /root
│
├── run/                    Runtime data (PID files, sockets)
│
├── sbin/                   System binaries (root-only commands)
│   ├── fdisk               Partition manager
│   ├── iptables            Firewall
│   └── useradd             User management
│
├── srv/                    Data for services (web, FTP)
│   └── www/                Sometimes web files go here
│
├── sys/                    Virtual filesystem: hardware/driver interface
│
├── tmp/                    Temporary files (deleted on reboot)
│   └── (app temp files)    Safe to store session data here
│
├── usr/                    User programs (secondary hierarchy)
│   ├── bin/                Most user commands (git, vim, python3)
│   ├── sbin/               Non-essential system commands
│   ├── lib/                Libraries for /usr/bin
│   ├── local/              Locally compiled/installed programs
│   │   ├── bin/            Priority in PATH over /usr/bin
│   │   └── lib/
│   └── share/              Arch-independent data, man pages, icons
│
└── var/                    Variable data (changes during operation)
    ├── log/                System and application logs
    │   ├── syslog          General system log
    │   ├── auth.log        Authentication attempts
    │   └── nginx/          Nginx access and error logs
    ├── cache/              Application cache
    ├── spool/              Print jobs, mail queues
    └── www/html/           Web server document root (common)
```

### 6.3 Key Directories to Memorize First

As a beginner, focus on these seven first:

| Directory | Remember It As | What's Inside |
|-----------|---------------|--------------|
| `/` | The Top | Everything starts here |
| `/home/username` | Your Room | Your personal files, configs, projects |
| `/etc` | City Hall | Config files for everything |
| `/var/log` | The Logbook | Logs for debugging and monitoring |
| `/tmp` | The Whiteboard | Temp files, wiped on reboot |
| `/bin` and `/usr/bin` | The Toolbox | All the commands you run |
| `/proc` | The Dashboard | Live system info |

---

## 7. File Types & Structure

### 7.1 The Seven File Types

Linux has 7 types of files. You can see the type from the first character of `ls -la` output:

```
ls -la /dev/ /var/log/ /tmp/
```

| Symbol | Type | Description | Example |
|--------|------|-------------|---------|
| `-` | Regular file | Text, binaries, images, scripts | `/etc/hosts`, `script.sh` |
| `d` | Directory | A container for other files | `/home/alice/`, `/etc/` |
| `l` | Symbolic link | A pointer/shortcut to another file | `/usr/bin/python → python3.11` |
| `c` | Character device | Hardware that reads/writes character by character | `/dev/tty`, `/dev/null` |
| `b` | Block device | Storage hardware (disk, USB) | `/dev/sda`, `/dev/sdb1` |
| `p` | Named pipe (FIFO) | Inter-process communication channel | Created with `mkfifo` |
| `s` | Socket | Network/IPC communication endpoint | `/run/docker.sock` |

### 7.2 Identifying File Types in Practice

```bash
# See the type character at the start of each line
ls -la /

# Output example:
drwxr-xr-x  18 root root  4096 Jan 10 09:00 etc        ← d = directory
-rw-r--r--   1 root root   220 Jan 10 09:00 profile     ← - = regular file
lrwxrwxrwx   1 root root     7 Jan 10 09:00 bin -> usr/bin  ← l = symlink

# Check file type explicitly
file /etc/passwd          # ASCII text
file /bin/ls              # ELF 64-bit LSB pie executable
file /dev/sda             # block special (0/0)
file /dev/null            # character special (1/3)
file ~/script.sh          # Bourne-Again shell script, ASCII text

# Find files by type
find /dev -type c         # Character devices
find /dev -type b         # Block devices
find /etc -type l         # Symbolic links in /etc
find . -type f            # Only regular files
find . -type d            # Only directories
```

### 7.3 Hidden Files

In Linux, any file starting with `.` (dot) is hidden:

```bash
ls ~                    # Shows visible files
ls -a ~                 # Shows ALL files (including hidden)
ls -la ~                # Long format + all

# Common hidden files in home directory:
.bashrc                 # Bash configuration (runs every new shell)
.bash_profile           # Login shell config
.bash_history           # Command history
.ssh/                   # SSH keys and config
.gitconfig              # Git user configuration
.config/                # Application config directory
.local/                 # User-local applications and data

# Create a hidden file
touch .mysecret
ls                      # Won't show .mysecret
ls -a                   # Shows .mysecret
```

---

## 8. Absolute vs Relative Paths

### 8.1 The Core Concept

Every location in the filesystem can be described in two ways:

**Absolute Path** — The full address from the root `/`. Like a GPS coordinate — always unambiguous.

**Relative Path** — The path from where you currently are. Like saying "turn left at the corner" — depends on where you're standing.

### 8.2 Absolute Paths

```bash
# Always start with /
/home/alice/projects/devops/script.sh
/etc/nginx/nginx.conf
/var/log/syslog
/usr/bin/python3

# Absolute paths work from ANY location
cd /var/log            # Always goes to /var/log, no matter where you are
cat /etc/hosts         # Always reads this exact file
ls /home/alice/        # Always lists this exact directory

# Examples:
pwd                    # Shows your absolute path, e.g., /home/alice
cd /etc                # Goes to /etc (absolute)
ls /var/log            # Lists /var/log (absolute)
```

### 8.3 Relative Paths

```bash
# Do NOT start with /
# They are relative to your CURRENT directory (shown by pwd)

# If you are in /home/alice:
cd projects            # Goes to /home/alice/projects
cd projects/devops     # Goes to /home/alice/projects/devops
cat notes.txt          # Reads /home/alice/notes.txt
./script.sh            # Runs /home/alice/script.sh

# If you are in /home/alice/projects:
cd ..                  # Goes up to /home/alice
cd ../bob              # Goes up to /home, then into bob → /home/bob
cat ../notes.txt       # Reads /home/alice/notes.txt (one level up)

# Visual example:
# Current location: /home/alice/projects
# 
# Relative: documents/report.txt
# Absolute: /home/alice/projects/documents/report.txt
#           └──────────── auto-prepended ─────────────┘
```

### 8.4 Side-by-Side Comparison

```bash
# Scenario: You are in /home/alice

# Navigate to /etc/nginx/
cd /etc/nginx               # Absolute: works from anywhere
cd ../../etc/nginx          # Relative: up 2 levels, then into etc/nginx
                            # (messy; absolute is better for /etc)

# Navigate to projects/
cd /home/alice/projects     # Absolute
cd projects                 # Relative (much simpler here!)

# Read a config file from anywhere
cat /etc/hostname           # Absolute: always correct
# vs.
cd /etc && cat hostname     # Relative after cd: same result, more steps
```

### 8.5 When to Use Which

```
Use ABSOLUTE paths when:
✅ Writing scripts (scripts run from unknown locations)
✅ Navigating to system directories (/etc, /var, /usr)
✅ Sharing commands with others
✅ Configuring services (always use absolute in config files)

Use RELATIVE paths when:
✅ Navigating within your own project directory
✅ Interactive work (you know where you are)
✅ Short trips within the current area
✅ Referencing files next to each other (./sibling.sh)
```

---

## 9. Special Directories & Shortcuts

### 9.1 The Magic Symbols

```bash
~        → Your home directory (/home/yourusername)
.        → Current directory
..       → Parent directory (one level up)
-        → Previous directory (where you just were)
/        → Root directory (the very top)

# Practical use:
cd ~                    # Go home
cd ~/projects           # Go to projects in your home
ls .                    # List current directory (same as just: ls)
ls ..                   # List parent directory
ls ../..                # List grandparent directory
cd -                    # Go back to where you were before
cat ./config.txt        # Read config.txt in current dir
cp ../file.txt .        # Copy file from parent to current dir
```

### 9.2 ~ (Tilde) — Your Home Directory

```bash
# ~ always expands to your home directory
echo ~                  # /home/alice
echo ~alice             # /home/alice (alice's home)
echo ~root              # /root (root's home)

# Common uses:
cd ~                    # Go home (same as: cd with no args)
ls ~/Documents          # List Documents in your home
cat ~/.bashrc           # Read your bash config
vim ~/.ssh/config       # Edit SSH config
cp file.txt ~/backup/   # Copy to your home's backup dir
```

### 9.3 . (Dot) — Current Directory

```bash
# . refers to the current directory
ls .                    # Same as: ls
cp file.txt .           # Copy file TO current directory
cp ../file.txt .        # Copy from parent to here
mv /tmp/script.sh .     # Move script here
./script.sh             # Run script in current directory

# Why ./script.sh and not just script.sh?
# Linux does NOT search the current directory for executables by default
# (unlike Windows). You must explicitly say "in this directory" with ./
./myscript.sh           # ✅ Run script in current directory
myscript.sh             # ❌ "command not found" (unless . is in PATH)
```

### 9.4 .. (Double Dot) — Parent Directory

```bash
# .. means "the directory above"
cd ..                   # Go up one level
cd ../..                # Go up two levels
cd ../../etc            # Up two levels, then into etc
ls ..                   # List parent directory
cat ../config.txt       # Read file in parent directory
cp ../shared/file.txt . # Copy from parent's shared to here

# Example:
# Current: /home/alice/projects/devops/scripts
cd ..                   # → /home/alice/projects/devops
cd ../..                # → /home/alice/projects
cd ../../..             # → /home/alice
cd ../../../..          # → /home
cd ../../../../..       # → /  (root)
```

### 9.5 - (Dash) — Previous Directory

```bash
# - takes you back to wherever you were before the last cd
cd /etc
cd /var/log
cd -                    # Back to /etc (previous)
cd -                    # Back to /var/log
cd -                    # Back to /etc (toggles!)

# Useful for:
cd /very/long/path/to/somewhere
cd /another/long/path
cd -                    # Jump back instantly!

echo $OLDPWD            # See what the previous directory was
```

---

## 10. Essential Navigation Commands — Deep Dive

### 10.1 `pwd` — Print Working Directory

```bash
pwd                     # Print your current location

# Output: /home/alice

# Options:
pwd -L                  # Print logical path (follows symlinks)
pwd -P                  # Print physical path (resolves symlinks)

# Use case: Never get lost again
# When you open a terminal, pwd tells you where you are
# In scripts: use pwd to know where the script is running

# Advanced: store current location
SAVED=$(pwd)
cd /somewhere/else
# ... do work ...
cd "$SAVED"             # Return to saved location
```

### 10.2 `ls` — List Directory Contents

```bash
# Basic usage
ls                      # List current directory
ls /etc                 # List specific directory
ls /home/alice          # List alice's home

# Essential flags
ls -l                   # Long format (permissions, owner, size, date)
ls -a                   # All files (including hidden . files)
ls -la                  # Long + all (most common combo)
ls -lh                  # Long + human-readable sizes (KB, MB, GB)
ls -lt                  # Long + sort by time (newest first)
ls -lS                  # Long + sort by size (largest first)
ls -lr                  # Reverse sort order
ls -lR                  # Recursive (list subdirectories too)
ls -1                   # One file per line
ls -d */                # Only directories

# Combine flags (all equivalent)
ls -l -a -h
ls -lah
ls -la -h

# Understanding ls -la output:
ls -la /home/alice

# drwxr-xr-x  5  alice  alice  4096  Jun 10 09:00  projects
# │└─┬─┘└─┬─┘  │  │      │     │      │             │
# │  │    │    │  │      │     │      │             └── filename
# │  │    │    │  │      │     │      └── last modified date
# │  │    │    │  │      │     └── size in bytes
# │  │    │    │  │      └── group owner
# │  │    │    │  └── user owner
# │  │    │    └── number of hard links
# │  │    └── others' permissions (r-x = read + execute)
# │  └── group's permissions (r-x = read + execute)
# └── type + owner perms (d = dir, rwx = read+write+exec)

# Colorized output (usually default)
ls --color=auto

# Sort options
ls -lt                  # Newest modified first
ls -ltr                 # Oldest modified first (useful for logs)
ls -lS                  # Largest first
ls -lSr                 # Smallest first

# List specific file types
ls *.txt                # All .txt files
ls *.sh                 # All shell scripts
ls -d */                # Only directories in current location
ls /etc/*.conf          # All .conf files in /etc
```

### 10.3 `cd` — Change Directory

```bash
# Navigate to absolute path
cd /etc
cd /var/log
cd /home/alice/projects

# Navigate with relative path
cd projects             # Into projects (must exist in current dir)
cd projects/devops      # Into nested relative path

# Special shortcuts
cd                      # Go home (same as cd ~)
cd ~                    # Go home
cd -                    # Go to previous directory
cd ..                   # Go up one level
cd ../..                # Go up two levels
cd ~/projects           # Absolute using tilde

# Practical patterns
cd /etc && ls           # Go somewhere and immediately list
cd "My Documents"       # Paths with spaces need quotes
cd My\ Documents        # Or escape the space with backslash

# Common beginner mistakes:
cd /home                # ✅ Goes to /home
cd home                 # ❌ "No such file or directory" (if not in /)
cd /home/alice          # ✅ Alice's home (absolute)
cd alice                # ✅ Only works if you're already in /home

# Pro tip: use Tab completion
cd /etc/ngi<Tab>        # Autocompletes to /etc/nginx/
cd ~/proj<Tab>          # Autocompletes to ~/projects/
```

### 10.4 `tree` — Visual Directory Tree

```bash
# Install first
sudo apt install tree

# Basic use
tree                    # Tree of current directory
tree /etc               # Tree of /etc
tree ~                  # Tree of home directory

# Options
tree -L 2               # Limit depth to 2 levels
tree -L 2 /etc          # /etc tree, 2 levels deep
tree -a                 # Include hidden files
tree -d                 # Directories only (no files)
tree -f                 # Full path for each file
tree -h                 # Human-readable file sizes
tree -s                 # File sizes in bytes
tree --dirsfirst        # Directories before files
tree -P "*.conf"        # Only show .conf files
tree -I "*.log"         # Exclude .log files

# Output to file
tree /etc -L 2 > filesystem_map.txt

# Example output:
# /etc
# ├── apt
# │   ├── apt.conf.d
# │   └── sources.list
# ├── hosts
# ├── nginx
# │   ├── nginx.conf
# │   └── sites-enabled
# └── ssh
#     └── sshd_config
```

### 10.5 Other Useful Navigation Commands

```bash
# pushd / popd — directory stack
pushd /etc              # Go to /etc AND save current location
pushd /var/log          # Go to /var/log AND save /etc
dirs                    # Show directory stack: /var/log /etc ~
popd                    # Pop /var/log, return to /etc
popd                    # Pop /etc, return to ~

# Why use pushd/popd?
# Better than cd - when you need to visit multiple dirs and return

# stat — detailed file/directory information
stat /etc/passwd
# Output:
#   File: /etc/passwd
#   Size: 1234       Blocks: 8    IO Block: 4096  regular file
#   Device: 8,1      Inode: 131073   Links: 1
#   Access: (0644/-rw-r--r--)  Uid: (0/root)  Gid: (0/root)
#   Access: 2025-06-10 08:30:00
#   Modify: 2025-05-01 12:00:00
#   Change: 2025-05-01 12:00:00

# file — determine file type
file /etc/passwd        # ASCII text
file /bin/ls            # ELF 64-bit LSB pie executable
file /var/log/syslog    # ASCII text

# du — disk usage of directories
du -sh ~                # Size of your home directory
du -sh /var/log         # Size of log directory
du -sh /var/log/*       # Size of each item in /var/log
du -ah . | sort -rh | head -20  # Top 20 largest items here

# df — disk free (filesystem level)
df -h                   # All filesystems, human-readable
df -h /home             # Just the filesystem containing /home

# wc — count lines, words, bytes (useful for files)
wc -l /etc/passwd       # How many user accounts?
ls /etc | wc -l         # How many items in /etc?
```

---

## 11. Practical Exercises

### Exercise 1: Your First Linux Session (20 min)

Open your terminal and work through these commands one by one. Write down what each outputs.

```bash
# ── PART A: Orient Yourself ──────────────────────────────────

# 1. Find out where you are
pwd

# 2. Find out who you are
whoami
id

# 3. See your system info
uname -a
hostname
uptime

# 4. Go to your home directory
cd ~
pwd              # Confirm you're in /home/yourusername

# ── PART B: Explore the Filesystem ──────────────────────────

# 5. Look at the root of the filesystem
ls /
# Write down: What directories do you see?

# 6. Explore key directories
ls /etc
ls /var/log
ls /home
ls /bin

# 7. Look at some interesting files
cat /etc/hostname
cat /etc/os-release
cat /proc/version
cat /proc/cpuinfo | head -20

# ── PART C: Navigate ─────────────────────────────────────────

# 8. Navigate using absolute paths
cd /etc
pwd
cd /var/log
pwd
cd /
pwd

# 9. Navigate back home
cd ~
pwd

# 10. Navigate using relative paths
cd ..              # Go up one level from home
pwd
cd ..              # Up one more
pwd
cd -               # Go back to where you were
pwd
```

**Write in your notes:**
- Where is your home directory?
- What does `ls /` show you? Name 5 directories.
- What's the difference you noticed between `cd /etc` and `cd etc`?

---

### Exercise 2: Official Day 4 Practical (30 min)

These are the exercises from the course curriculum. Complete all of them:

```bash
# ── TASK 1: Navigate to home directory ───────────────────────
cd ~
pwd
# ✅ Confirm output: /home/yourusername

# ── TASK 2: Create directory Day1 ────────────────────────────
mkdir Day1
ls                          # Confirm Day1 was created
ls -la                      # Long format to see permissions

# ── TASK 3: Navigate into Day1 ───────────────────────────────
cd Day1
pwd                         # Should show /home/yourusername/Day1

# ── TASK 4: Create intro.txt ─────────────────────────────────
touch intro.txt             # Create the file

# ── TASK 5: Write text into intro.txt ────────────────────────
echo "My first day learning Linux" > intro.txt

# Verify the content was written
cat intro.txt
# Expected output: My first day learning Linux

# ── TASK 6: Find which directory you're in ───────────────────
pwd
# Expected output: /home/yourusername/Day1

# ── TASK 7: List the contents of the directory ───────────────
ls
# Expected output: intro.txt

ls -la
# Should show intro.txt with its permissions, owner, size, date
```

**Expected final state:**
```
/home/yourusername/
└── Day1/
    └── intro.txt       (contains: "My first day learning Linux")
```

---

### Exercise 3: Path Mastery (25 min)

Practice until absolute and relative paths feel natural:

```bash
# ── Setup: Create a practice structure ───────────────────────
mkdir -p ~/practice/{documents,scripts,config,logs}
touch ~/practice/documents/report.txt
touch ~/practice/scripts/backup.sh
touch ~/practice/config/settings.conf
touch ~/practice/logs/app.log

# Verify structure
tree ~/practice
# Should show:
# practice/
# ├── config
# │   └── settings.conf
# ├── documents
# │   └── report.txt
# ├── logs
# │   └── app.log
# └── scripts
#     └── backup.sh

# ── Navigate using ONLY absolute paths ───────────────────────
cd /home/$USER/practice/documents
pwd

cd /home/$USER/practice/scripts
pwd

cd /home/$USER/practice/logs
pwd

# ── Navigate using ONLY relative paths ───────────────────────
cd ~/practice             # Start here
cd documents              # Relative: into documents
pwd

cd ..                     # Relative: back to practice
cd scripts                # Relative: into scripts
pwd

cd ../config              # Relative: up to practice, then into config
pwd

cd ../../                 # Relative: up two levels (to home)
pwd

# ── Mix it up ────────────────────────────────────────────────
# From anywhere, use absolute to get to logs
cd /var/log
pwd

# Now use relative to go back to your practice area
cd ~/practice/logs
pwd

# ── Use the - shortcut ───────────────────────────────────────
cd /etc
cd ~/practice/documents
cd -              # Back to /etc
cd -              # Back to ~/practice/documents
```

---

### Exercise 4: ls Options Mastery (20 min)

```bash
# ── Explore ls flags ─────────────────────────────────────────

# Go to /etc (many files)
cd /etc

# List basic
ls

# Long format (see permissions, owner, size, date)
ls -l

# Show hidden files
ls -a

# Long + hidden
ls -la

# Human readable sizes
ls -lh

# Sort by time (newest first)
ls -lt

# Sort by size (largest first)
ls -lS

# Reverse order (oldest/smallest first)
ls -ltr

# Recursive listing
ls -lR /etc/ssh

# Only directories
ls -d */

# ── Practice in your home dir ─────────────────────────────────
cd ~
ls -la           # See hidden .bashrc, .profile, .ssh, etc.
ls -lt           # What did you recently modify?
ls -lS           # What's largest?

# ── Count items ───────────────────────────────────────────────
ls /etc | wc -l           # How many items in /etc?
ls -la ~ | wc -l          # How many items (inc. hidden) in home?
ls -la /bin | wc -l       # How many items in /bin?
```

---

### Exercise 5: File Type Detective (15 min)

```bash
# ── Identify file types with ls and file command ─────────────

# Look at /dev — see character (c) and block (b) devices
ls -la /dev | head -20
# Find lines starting with c (character)
ls -la /dev | grep "^c"
# Find lines starting with b (block)
ls -la /dev | grep "^b"

# Look at symlinks
ls -la /bin
# Lines starting with l are symlinks → shows target

ls -la /usr/bin | grep "^l" | head -10

# Use file command for specifics
file /etc/passwd           # ASCII text
file /etc/os-release       # ASCII text
file /bin/ls               # ELF 64-bit executable
file /dev/null             # character special
file /dev/sda              # block special (may need sudo)
file /usr/bin/python3      # symbolic link (or ELF executable)

# Find all symlinks in /usr/bin
find /usr/bin -type l | head -10

# Find all directories in /etc
find /etc -type d

# Find all config files in /etc
find /etc -name "*.conf" | head -10
```

---

## 12. Mini-Project: NexusCorp Onboarding Setup

**Scenario:**
You've just joined NexusCorp as a junior DevOps engineer. Your first task is to create your personal workspace directory structure on the Linux server. You must use both absolute and relative paths at various steps to prove you understand both.

**Instructions:**

```bash
# ── STEP 1: Create your NexusCorp workspace structure ─────────
# Using absolute path to create top-level
mkdir -p ~/nexuscorp

# Using relative path for everything inside
cd ~/nexuscorp
mkdir -p {projects,scripts,docs,logs,config,backups}
mkdir -p projects/{devops-pipeline,monitoring,automation}
mkdir -p docs/{runbooks,notes,diagrams}
mkdir -p scripts/{deploy,backup,monitoring}

# Verify your structure
tree ~/nexuscorp

# Expected structure:
# nexuscorp/
# ├── backups/
# ├── config/
# ├── docs/
# │   ├── diagrams/
# │   ├── notes/
# │   └── runbooks/
# ├── logs/
# ├── projects/
# │   ├── automation/
# │   ├── devops-pipeline/
# │   └── monitoring/
# └── scripts/
#     ├── backup/
#     ├── deploy/
#     └── monitoring/

# ── STEP 2: Create your onboarding notes file ────────────────
# Use ABSOLUTE path this time
cat > /home/$USER/nexuscorp/docs/notes/day4-notes.md << 'EOF'
# Day 4 Notes — NexusCorp DevOps Onboarding

## What I Learned Today

### Linux History
- Linux was created by Linus Torvalds in 1991 as a hobby project
- It builds on GNU tools created by Richard Stallman (1983)
- The GPL license is why Linux remains free forever
- Linux powers 96%+ of the world's web servers today

### Linux Distros
- Ubuntu is best for beginners and cloud (my choice for this course)
- RHEL/AlmaLinux is common in enterprises
- Kali Linux is specialized for security
- Alpine Linux is used in Docker containers (tiny ~5MB)

### Filesystem Key Directories
- / = root (top of everything)
- /home/username = my personal space
- /etc = all configuration files
- /var/log = all system logs
- /tmp = temporary files (gone on reboot)
- /bin and /usr/bin = where commands live

### Path Types
- Absolute: starts with / → always works from anywhere
- Relative: based on current location → shorter but context-dependent
- ~ = home, . = here, .. = parent, - = previous

### Commands Learned
- pwd   → where am I?
- ls -la → what's here?
- cd    → move around
- tree  → visual map
- file  → what type is this?
- stat  → detailed info

## Questions I Still Have
1. (write yours here)
2.
3.

## Challenges I Faced During Installation
1. (write yours here)
2.

Date: $(date)
EOF

# Verify it was created
cat ~/nexuscorp/docs/notes/day4-notes.md

# ── STEP 3: Create a README for your workspace ───────────────
# Use relative path this time
cd ~/nexuscorp
cat > README.txt << 'EOF'
NexusCorp DevOps Workspace
==========================
Created: Day 4 of DevOps training

Directory Structure:
  projects/     → DevOps pipeline code and IaC
  scripts/      → Automation scripts
  docs/         → Runbooks, notes, diagrams
  logs/         → Local log storage
  config/       → Configuration files
  backups/      → Backup files

Owner: [your name]
Contact: [your email]
EOF

# ── STEP 4: Explore your creation ───────────────────────────
# From home, use absolute path to list
ls -la /home/$USER/nexuscorp/

# From root, use absolute path
ls /home/$USER/nexuscorp/projects/

# From /tmp, navigate using relative path back to your workspace
cd /tmp
ls                           # You're in /tmp
cd ~/nexuscorp               # Home shortcut
pwd                          # Confirm: /home/youruser/nexuscorp
cd docs/notes                # Relative
pwd                          # Confirm: /home/youruser/nexuscorp/docs/notes
cat day4-notes.md            # Read your notes (relative)

# ── STEP 5: Write your installation notes ────────────────────
cat > ~/nexuscorp/docs/notes/installation-notes.txt << 'EOF'
Linux Installation Notes — Day 4
==================================

Method used: [VirtualBox / WSL2 / Cloud / Dual Boot]

Steps I followed:
1.
2.
3.
4.
5.

Challenges I faced:
1.
2.

How I solved them:
1.
2.

Time taken: approximately X minutes

System details:
EOF

# Append actual system info
echo "" >> ~/nexuscorp/docs/notes/installation-notes.txt
uname -a >> ~/nexuscorp/docs/notes/installation-notes.txt
lsb_release -a >> ~/nexuscorp/docs/notes/installation-notes.txt 2>/dev/null

echo ""
echo "✅ NexusCorp workspace created successfully!"
echo "📂 Location: ~/nexuscorp"
echo ""
tree ~/nexuscorp
```

**Deliverable:** Take a screenshot of:
1. Your `tree ~/nexuscorp` output
2. Your `cat ~/nexuscorp/docs/notes/day4-notes.md` output

Share these with your learning partner or in your study group.

---

## 13. Cheat Sheet

### Navigation Quick Reference

```
FINDING YOUR LOCATION
─────────────────────────────────────────────────
pwd                 Where am I right now?
hostname            What machine am I on?
whoami              Who am I logged in as?

MOVING AROUND
─────────────────────────────────────────────────
cd /absolute/path   Go to absolute path
cd relative/path    Go to relative path
cd ~                Go home
cd                  Go home (no arguments)
cd ..               Go up one level
cd ../..            Go up two levels
cd -                Go to previous location
cd ~/projects       Go to projects in home

LISTING CONTENTS
─────────────────────────────────────────────────
ls                  Basic listing
ls -l               Long format (details)
ls -a               All files (inc. hidden)
ls -la              Long + all (use this most)
ls -lh              Long + human-readable sizes
ls -lt              Sorted by time (newest first)
ls -lS              Sorted by size (largest first)
ls -lR              Recursive listing
ls -d */            Only directories

VIEWING FILESYSTEM
─────────────────────────────────────────────────
tree                Visual tree (must install)
tree -L 2           Tree, max 2 levels deep
tree -d             Directories only
df -h               Disk free (all filesystems)
du -sh dir/         Size of directory
stat file           Detailed file info
file filename       What type is this file?

SPECIAL SHORTCUTS
─────────────────────────────────────────────────
~                   Your home (/home/you)
.                   Current directory
..                  Parent directory
-                   Previous directory
/                   Root (top of everything)

PATH TYPES
─────────────────────────────────────────────────
/etc/hosts          ABSOLUTE: starts with /
etc/hosts           RELATIVE: based on where you are
./script.sh         RELATIVE: explicitly "here"
../config.txt       RELATIVE: one level up

FILE TYPES (first char of ls -la)
─────────────────────────────────────────────────
-                   Regular file
d                   Directory
l                   Symbolic link
c                   Character device
b                   Block device
p                   Named pipe
s                   Socket
```

### Key System Files to Know

```
/etc/hostname       → This server's hostname
/etc/hosts          → Local DNS (IP ↔ hostname)
/etc/os-release     → OS name and version
/etc/passwd         → User account list
/proc/version       → Kernel version
/proc/cpuinfo       → CPU details
/proc/meminfo       → Memory details
```

---

## 14. Glossary

| Term | Definition |
|------|-----------|
| **Absolute Path** | The full path from root `/` to a file/directory. Always starts with `/`. Works from anywhere. Example: `/home/alice/file.txt` |
| **Bash** | Bourne Again Shell — the default shell on most Linux systems. Interprets your commands. |
| **CLI** | Command-Line Interface — text-based interaction with the OS via a terminal |
| **Daemon** | A background service process. Name ends with `d`: `sshd`, `httpd`, `cron` |
| **Directory** | Linux's term for a folder. Shown as `d` in `ls -la` |
| **Distro** | A Linux distribution — the kernel bundled with software, package manager, and tools |
| **Filesystem** | The method Linux uses to organize and store files on disk. Everything hangs from `/` |
| **FHS** | Filesystem Hierarchy Standard — the specification defining where things go in Linux |
| **GNU** | GNU's Not Unix — the project that created the free tools (bash, gcc, gzip) combined with Linux kernel |
| **GPL** | GNU General Public License — the license that ensures Linux stays free forever |
| **Hidden file** | A file starting with `.` (dot). Hidden from regular `ls` but visible with `ls -a` |
| **Inode** | The data structure storing file metadata (permissions, owner, size) — not the filename |
| **Kernel** | The core of the OS. Manages CPU, RAM, I/O, and processes |
| **LTS** | Long-Term Support — Ubuntu versions supported for 5 years. Example: 22.04 LTS |
| **Mount** | Attaching a filesystem (disk, USB) to the directory tree at a mount point |
| **Parent directory** | The directory one level above the current one. Represented by `..` |
| **PATH** | An environment variable listing directories searched for commands |
| **Prompt** | The text before your cursor (e.g., `alice@ubuntu:~$`) |
| **Relative Path** | A path relative to the current directory. Does NOT start with `/`. Example: `projects/file.txt` |
| **Root** | Two meanings: (1) The superuser account `root`; (2) The top of the filesystem `/` |
| **Shell** | The program that interprets your typed commands. Most common: `bash` |
| **Shebang** | `#!/bin/bash` — first line of a script specifying which interpreter to use |
| **Symlink** | Symbolic link — a file that points to another file/directory (a shortcut) |
| **Terminal** | The application you type commands into. Provides access to the shell |
| **Tilde (~)** | Shortcut for your home directory (`/home/yourusername`) |
| **UNIX** | The predecessor OS (1969) that Linux is inspired by |
| **VirtualBox** | Free, open-source software for running virtual machines |
| **VM** | Virtual Machine — a software-simulated computer running inside your real computer |
| **WSL2** | Windows Subsystem for Linux 2 — runs a real Linux kernel inside Windows |

---

## 15. Resources

### Cheat Sheets
- 📋 **cheat.sh** — The recommended cheat sheet resource from this course
  ```bash
  # Use directly from terminal
  curl cheat.sh/ls          # Cheat sheet for ls
  curl cheat.sh/cd          # Cheat sheet for cd
  curl cheat.sh/find        # Cheat sheet for find
  curl cheat.sh/tar         # Cheat sheet for tar
  ```
- 📋 [Explain Shell](https://explainshell.com/) — Paste any command to see what each part does
- 📋 [TLDR Pages](https://tldr.sh/) — Simplified man pages
  ```bash
  sudo apt install tldr
  tldr ls
  tldr cd
  ```

### In-Depth Reading
- 📖 [The Linux Kernel — official documentation](https://www.kernel.org/doc/)
- 📖 [Linux From Scratch](https://www.linuxfromscratch.org/) — Build Linux from the ground up
- 📖 [The Linux Command Line (free book)](https://linuxcommand.org/tlcl.php)

### Handwritten Notes
- ✍️ [Kunal Kushwaha Linux Notes](https://github.com/kunal-kushwaha/DevOps-Bootcamp) — The handwritten commands referenced in this course

### Video Resources
- 🎥 [Linux File System Explained (YouTube)](https://www.youtube.com/results?search_query=linux+filesystem+hierarchy+explained)
- 🎥 [How Netflix Became A Master of DevOps — Case Study](https://www.youtube.com/results?search_query=netflix+devops+master+case+study)
- 🎥 [VirtualBox Ubuntu Installation Tutorial](https://www.youtube.com/results?search_query=install+ubuntu+virtualbox+2024)

### Interactive Practice
- 💻 [Linux Journey — Grasshopper section](https://linuxjourney.com/) — Start here for interactive filesystem lessons
- 💻 [OverTheWire: Bandit Level 0](https://overthewire.org/wargames/bandit/bandit0.html) — Practice navigation through a security game
- 💻 [Killercoda Ubuntu Playground](https://killercoda.com/learn) — Instant browser-based Ubuntu

---

## 16. Day 4 Mastery Checklist

Work through this checklist before moving to Day 5. Check off each item only when you can do it **without looking it up**:

### Linux History & Distros
- [ ] I can explain what UNIX is and how Linux relates to it
- [ ] I know who created Linux, when, and why it was originally a "hobby project"
- [ ] I can name at least 3 Debian-family distros and 3 Red Hat-family distros
- [ ] I can explain what LTS means and why it matters
- [ ] I know why Ubuntu is recommended for this course
- [ ] I can name 2 specialized distros and their use cases

### Installation
- [ ] I have a working Linux system (VM, WSL2, cloud, or native)
- [ ] I can log in and see a terminal prompt
- [ ] I can run `sudo apt update` successfully
- [ ] I have written down the steps I took to install Linux and any challenges

### Filesystem Navigation
- [ ] I know the purpose of `/`, `/etc`, `/home`, `/var/log`, `/tmp`, `/bin`
- [ ] I can navigate to any location using `cd` with an absolute path
- [ ] I can navigate using relative paths (`..`, `.`, directory names)
- [ ] I use `pwd` to confirm my location without thinking
- [ ] I can list files with `ls -la` and read the output
- [ ] I understand what `~`, `.`, `..`, and `-` mean in paths
- [ ] I can explain the difference between absolute and relative paths in one sentence

### File Types & Structure
- [ ] I know all 7 Linux file types (-, d, l, c, b, p, s)
- [ ] I can identify file types from `ls -la` output
- [ ] I know what hidden files are and how to see them
- [ ] I can use `file` command to identify a file's type

### Day 4 Exercise
- [ ] I navigated to my home directory using `cd ~`
- [ ] I created a `Day1` directory using `mkdir`
- [ ] I created `intro.txt` inside `Day1`
- [ ] I wrote "My first day learning Linux" into `intro.txt`
- [ ] I confirmed my location with `pwd`
- [ ] I listed the contents of `Day1` with `ls`

### Mini-Project
- [ ] I created the full NexusCorp workspace directory structure
- [ ] I created `day4-notes.md` with my notes
- [ ] I created `installation-notes.txt` with my installation experience
- [ ] I can `tree ~/nexuscorp` and see the full structure

---

## 17. What's Coming — Day 5 Preview

**Day 5: File Operations, Text Manipulation & Permissions**

Tomorrow you go deeper:

```
File Operations:
  cp, mv, rm, touch, mkdir — advanced usage
  Wildcards: *, ?, [abc]

Text Viewing & Editing:
  cat, less, head, tail, grep
  Your first vim session
  echo and redirection: >, >>

File Permissions (hands-on):
  chmod with octal and symbolic modes
  chown and chgrp
  Understanding rwx for files vs directories

Standard I/O:
  stdin, stdout, stderr
  Pipes: |
  Redirection: >, >>, <, 2>
```

> _"The filesystem is the map. Day 4 taught you to read it. Day 5 teaches you to change it."_

---

**Questions to Open Up Minds:**

1. Why do you think Linux uses a single-root filesystem (`/`) instead of Windows-style drive letters (`C:\`, `D:\`)?

2. NexusCorp's original problem was that developers had no idea what happened to their code after it left their laptops. How does understanding the Linux filesystem start to solve that?

3. If you had to recommend one Linux distro to a friend starting their DevOps career, which would you choose and why?

4. The Linux kernel was released under the GPL. How would Linux be different today if Linus Torvalds had released it under a proprietary license instead?

---

*Day 4 of the NexusCorp DevOps Transformation Series*
*Module: Linux Fundamentals | Next: Day 5 — File Operations & Permissions*
