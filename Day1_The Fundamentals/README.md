# 🐧 Day 1: Linux Fundamentals — Your First Steps into the Terminal

> **Part of the NexusCorp DevOps Transformation Series**
> _"Linux is not just an OS — it's the language the modern infrastructure speaks."_

---

## 📋 Table of Contents

- [Why Linux for DevOps?](#why-linux-for-devops)
- [Learning Objectives](#learning-objectives)
- [What is Linux?](#what-is-linux)
- [The Linux Architecture](#the-linux-architecture)
- [Linux Distributions](#linux-distributions)
- [Getting Your Environment Ready](#getting-your-environment-ready)
- [Core Concepts](#core-concepts)
- [Essential Commands — Day 1 Cheat Sheet](#essential-commands--day-1-cheat-sheet)
- [Hands-On Exercises](#hands-on-exercises)
- [Mini-Project](#mini-project)
- [Glossary](#glossary)
- [Resources](#resources)
- [Day 1 Checklist](#day-1-checklist)

---

## Why Linux for DevOps?

Before NexusCorp could adopt DevOps at scale, their engineers had to master Linux. Here's why it matters:

| Stat | What It Means |
|------|--------------|
| ~96% of the world's top web servers run Linux | Your apps live on Linux |
| All major cloud providers (AWS, GCP, Azure) default to Linux | Your pipelines run on Linux |
| Docker, Kubernetes, Ansible, Terraform — all Linux-native | Your DevOps tools are Linux tools |
| 500+ of the Fortune 500 use Linux in production | Knowing Linux = employability |

> **NexusCorp Connection:** When NexusCorp's systems were siloed and brittle, one of the root causes was that developers had no idea what was happening on the servers their code ran on. Understanding Linux closes that gap.

---

## Learning Objectives

By the end of Day 1, you will be able to:

- [ ] Explain what Linux is and why it dominates the server world
- [ ] Identify the key components of the Linux architecture
- [ ] Name at least 3 Linux distributions and their use cases
- [ ] Set up a working Linux environment on your machine
- [ ] Navigate the filesystem using the terminal
- [ ] Create, read, move, and delete files and directories
- [ ] Understand file permissions at a conceptual level
- [ ] Feel comfortable opening a terminal and running commands

**Time Estimate:** 4–6 hours (reading + exercises + mini-project)

---

## What is Linux?

Linux is a **free, open-source operating system kernel** created by **Linus Torvalds** in 1991. A kernel is the core of an OS — it manages communication between your hardware and software.

What we commonly call "Linux" is more accurately a **Linux distribution (distro)** — the kernel bundled with software, tools, and a package manager to form a complete OS.

### A Very Brief History

```
1969  — Unix created at Bell Labs (the grandfather of Linux)
1983  — GNU Project started by Richard Stallman (free software movement)
1991  — Linus Torvalds releases Linux kernel v0.01 (as a hobby project!)
1994  — Linux kernel v1.0 released
2000s — Linux dominates servers and web hosting
2008  — Android (built on Linux) launches — Linux enters your pocket
2010s — Cloud computing explodes; Linux powers AWS, GCP, Azure
Today — Linux runs 96%+ of the world's servers, all supercomputers, most IoT
```

### Linux vs. Windows vs. macOS

| Feature | Linux | Windows | macOS |
|---------|-------|---------|-------|
| Cost | Free | Paid | Free (with Apple hardware) |
| Open Source | Yes | No | Partially |
| Customizability | Extremely high | Low | Medium |
| Server dominance | ~96% | ~1.8% | ~1.9% |
| DevOps tooling | Native | Via WSL/VM | Good (Unix-based) |
| Package management | `apt`, `yum`, `dnf` | WinGet (limited) | Homebrew |

---

## The Linux Architecture

Linux has a layered architecture. Think of it like an onion:

```
┌─────────────────────────────────────┐
│          USER APPLICATIONS          │  ← The apps you run (vim, git, docker)
├─────────────────────────────────────┤
│              SHELL                  │  ← Your interface to the OS (bash, zsh)
├─────────────────────────────────────┤
│          SYSTEM LIBRARIES           │  ← glibc, system calls
├─────────────────────────────────────┤
│         LINUX KERNEL                │  ← Process, memory, I/O management
├─────────────────────────────────────┤
│             HARDWARE                │  ← CPU, RAM, Disk, Network
└─────────────────────────────────────┘
```

### Key Components Explained

**Kernel**
The heart of Linux. Manages processes, memory, devices, and system calls. You rarely interact with it directly.

**Shell**
The command-line interface that interprets your commands. The most common shell is **Bash** (Bourne Again SHell). Think of it as your translator between you and the kernel.

**Filesystem**
Linux organizes everything in a single tree starting at `/` (root). Unlike Windows (C:, D: drives), there is one unified hierarchy.

**Package Manager**
A tool to install, update, and remove software. Examples: `apt` (Debian/Ubuntu), `yum`/`dnf` (RHEL/CentOS/Fedora), `pacman` (Arch).

---

## Linux Distributions

A **distribution (distro)** is a complete Linux-based OS built around the Linux kernel. There are hundreds of distros, but a DevOps engineer mainly needs to know these:

### The Big Families

```
Linux Kernel
│
├── Debian Family
│   ├── Debian (stable, conservative)
│   ├── Ubuntu (most popular desktop + server)
│   │   └── Ubuntu Server (common in cloud)
│   └── Linux Mint (beginner-friendly desktop)
│
├── Red Hat Family
│   ├── RHEL — Red Hat Enterprise Linux (paid, enterprise)
│   ├── CentOS / AlmaLinux / Rocky Linux (free RHEL alternatives)
│   └── Fedora (cutting-edge, community-driven)
│
├── SUSE Family
│   ├── openSUSE
│   └── SLES — SUSE Linux Enterprise Server
│
└── Arch Family
    ├── Arch Linux (DIY, bleeding-edge)
    └── Manjaro (beginner-friendly Arch)
```

### Which Should You Learn First?

**For DevOps beginners: Ubuntu Server or CentOS/AlmaLinux**

- Ubuntu: Best documentation, huge community, default on AWS EC2 free tier
- CentOS/AlmaLinux: Mirrors RHEL, common in enterprise environments

> **NexusCorp Note:** NexusCorp standardized on Ubuntu 22.04 LTS for their application servers during their DevOps transformation — a common real-world choice.

---

## Getting Your Environment Ready

You have three options. Choose one based on your setup:

### Option A: WSL2 on Windows (Recommended for Windows users)

```powershell
# Open PowerShell as Administrator and run:
wsl --install

# Then launch Ubuntu from the Start menu
# Default credentials: set during first launch
```

### Option B: VirtualBox + Ubuntu (Any OS)

1. Download [VirtualBox](https://www.virtualbox.org/)
2. Download [Ubuntu 22.04 LTS ISO](https://ubuntu.com/download/server)
3. Create a new VM → 2 CPU, 4GB RAM, 20GB disk
4. Mount ISO and install Ubuntu

### Option C: Cloud-Based (No local setup needed)

- [AWS Free Tier EC2](https://aws.amazon.com/free/) — t2.micro instance, Ubuntu AMI
- [Google Cloud Shell](https://shell.cloud.google.com/) — free browser-based Linux terminal
- [Killercoda](https://killercoda.com/) — free Linux playground scenarios

### Verify Your Setup

Once you have access to a Linux terminal, run:

```bash
uname -a          # Print OS info
lsb_release -a    # Print distro info (Debian/Ubuntu)
whoami            # Print current user
pwd               # Print current directory
```

Expected output (Ubuntu example):
```
Linux ubuntu 5.15.0-1040-aws #45-Ubuntu SMP Thu Jun 15 19:17:35 UTC 2023 x86_64 GNU/Linux
```

---

## Core Concepts

### 1. The Linux Filesystem Hierarchy

```
/                   Root — the top of everything
├── bin/            Essential user binaries (ls, cp, mv)
├── sbin/           System binaries (for root/admin use)
├── etc/            Configuration files (the "settings drawer")
├── home/           User home directories (/home/alice, /home/bob)
├── root/           Home directory of the root (admin) user
├── var/            Variable data — logs, caches, databases
│   └── log/        System logs live here
├── tmp/            Temporary files (cleared on reboot)
├── usr/            User programs and utilities
│   ├── bin/        Non-essential user binaries
│   └── local/      Locally installed software
├── proc/           Virtual filesystem: info about running processes
├── dev/            Device files (disks, terminals)
├── mnt/            Mount points for external filesystems
└── opt/            Optional/third-party software
```

> **Remember:** In Linux, _everything is a file_ — even hardware devices, network connections, and processes are represented as files.

### 2. Paths

**Absolute path** — starts from root `/`:
```bash
/home/alice/documents/report.txt
```

**Relative path** — relative to your current location:
```bash
documents/report.txt    # if you're in /home/alice
../bob/notes.txt        # go up one level, then into bob's folder
```

**Special path shortcuts:**
```bash
~        # Your home directory (/home/yourusername)
.        # Current directory
..       # Parent directory
-        # Previous directory (cd -)
```

### 3. File Permissions

Every file in Linux has permissions for three groups:

```
-rwxr-xr--  1  alice  developers  4096  Jun 10 09:00  script.sh
│└──┬──┘└─┬─┘└─┬──┘  └────┬────┘
│   │     │    │           └── File size
│   │     │    └── Group name
│   │     └── Owner name
│   └── Permissions: [owner][group][others]
└── File type: - (file), d (directory), l (symlink)
```

Permission characters:
```
r = read    (4)
w = write   (2)
x = execute (1)
- = none    (0)
```

Example: `rwxr-xr--`
- Owner (alice): `rwx` = read + write + execute
- Group (developers): `r-x` = read + execute
- Others: `r--` = read only

> Day 1 Scope: Just understand the concept. You'll practice `chmod` and `chown` on Day 2.

### 4. Users and Root

Linux is a multi-user system. Every action is tied to a user.

- **Root** (`root`) — the superuser. Has unrestricted access to everything. Use with caution.
- **sudo** — "superuser do." Run a single command as root without logging in as root.
- **Regular users** — limited permissions. Can only affect their own files.

```bash
whoami          # See your username
sudo whoami     # Should output "root"
id              # See user ID, group ID, and group memberships
```

---

## Essential Commands — Day 1 Cheat Sheet

### Navigation

| Command | What It Does | Example |
|---------|-------------|---------|
| `pwd` | Print working directory | `pwd` → `/home/alice` |
| `ls` | List directory contents | `ls -la /etc` |
| `cd` | Change directory | `cd /var/log` |
| `cd ~` | Go to home directory | `cd ~` |
| `cd ..` | Go up one directory | `cd ..` |
| `cd -` | Go to previous directory | `cd -` |
| `tree` | Display directory tree | `tree -L 2 /etc` |

### Working with Files and Directories

| Command | What It Does | Example |
|---------|-------------|---------|
| `touch` | Create empty file | `touch notes.txt` |
| `mkdir` | Create directory | `mkdir -p projects/devops` |
| `cp` | Copy file/directory | `cp file.txt backup.txt` |
| `mv` | Move or rename | `mv old.txt new.txt` |
| `rm` | Remove file | `rm file.txt` |
| `rm -rf` | Remove directory (CAREFUL!) | `rm -rf old_folder/` |
| `cat` | Print file contents | `cat /etc/hostname` |
| `less` | View file with scrolling | `less /var/log/syslog` |
| `head` | First N lines of file | `head -20 file.txt` |
| `tail` | Last N lines of file | `tail -f /var/log/syslog` |

### File Information

| Command | What It Does | Example |
|---------|-------------|---------|
| `ls -l` | Long listing with permissions | `ls -l /home` |
| `ls -a` | Show hidden files | `ls -a ~` |
| `file` | Determine file type | `file image.png` |
| `stat` | Detailed file metadata | `stat script.sh` |
| `wc` | Word/line/byte count | `wc -l file.txt` |
| `du -sh` | Directory size | `du -sh /var/log` |
| `df -h` | Disk usage summary | `df -h` |

### System Information

| Command | What It Does | Example |
|---------|-------------|---------|
| `uname -a` | OS and kernel info | `uname -a` |
| `hostname` | System hostname | `hostname` |
| `uptime` | How long system has been running | `uptime` |
| `whoami` | Current username | `whoami` |
| `id` | User and group IDs | `id` |
| `date` | Current date and time | `date` |
| `cal` | Calendar | `cal` |

### Getting Help

```bash
man ls          # Manual page for 'ls'
ls --help       # Quick help flag (most commands support this)
info bash       # Info pages (more detailed than man)
type cd         # Tell you what kind of command something is
which python3   # Show path to a command's binary
```

> **Pro Tip:** When stuck, `man <command>` is your best friend. Press `q` to quit, `/` to search within the manual.

---

## Hands-On Exercises

Work through these in order. Each builds on the last.

### Exercise 1: Navigate Like a Pro (15 min)

```bash
# 1. Open your terminal and find out where you are
pwd

# 2. List the contents of the root filesystem
ls /

# 3. Explore three key directories
ls /etc
ls /var/log
ls /home

# 4. Navigate using relative paths
cd /var
ls
cd log
pwd
cd ../..
pwd

# 5. Use the shortcut to go home instantly
cd ~
pwd
```

**Questions to answer in your notes:**
- What's the difference between `/bin` and `/usr/bin`?
- What files do you see in `/etc`? What do you think they control?

---

### Exercise 2: Create Your DevOps Workspace (20 min)

```bash
# 1. Create a project structure for this course
mkdir -p ~/devops-journey/day-01/{notes,scripts,practice}

# 2. Verify the structure was created
tree ~/devops-journey     # if tree is installed
ls -R ~/devops-journey    # alternatively

# 3. Create some files
touch ~/devops-journey/day-01/notes/what-i-learned.txt
touch ~/devops-journey/day-01/scripts/hello.sh

# 4. Write something into your notes file
echo "Day 1: Linux is the backbone of DevOps" > ~/devops-journey/day-01/notes/what-i-learned.txt
echo "Everything in Linux is a file" >> ~/devops-journey/day-01/notes/what-i-learned.txt

# 5. Read it back
cat ~/devops-journey/day-01/notes/what-i-learned.txt
```

---

### Exercise 3: File Operations (25 min)

```bash
# 1. Copy a file
cd ~/devops-journey/day-01/practice
cp ../notes/what-i-learned.txt backup-notes.txt

# 2. Rename a file using mv
mv backup-notes.txt day1-backup.txt

# 3. Look at system files (read-only exploration)
cat /etc/hostname
cat /etc/os-release
head -5 /etc/passwd       # First 5 lines of the users file

# 4. Count lines in a file
wc -l /etc/passwd         # How many users/entries?

# 5. View a live log (Ctrl+C to exit)
sudo tail -f /var/log/syslog     # Ubuntu
# or
sudo tail -f /var/log/messages   # CentOS/RHEL

# 6. Check disk and directory sizes
df -h
du -sh ~
du -sh /var/log
```

---

### Exercise 4: Understanding Permissions (20 min)

```bash
# 1. Create a test script
echo '#!/bin/bash' > ~/devops-journey/day-01/scripts/hello.sh
echo 'echo "Hello from NexusCorp DevOps Training!"' >> ~/devops-journey/day-01/scripts/hello.sh

# 2. Try to run it (will fail without execute permission)
./hello.sh     # Permission denied!

# 3. Check its current permissions
ls -l ~/devops-journey/day-01/scripts/hello.sh

# 4. Read the permission string and decode it in your notes
# (We'll grant execute permission on Day 2 with chmod)

# 5. Look at permissions of some system files
ls -l /etc/passwd        # -rw-r--r--  (readable by all, writable only by root)
ls -l /etc/shadow        # -rw-r-----  (only root can read — passwords stored here)
ls -l /bin/bash          # -rwxr-xr-x  (executable by everyone)
```

---

## Mini-Project

### "The Filesystem Explorer" (45–60 min)

**Goal:** Build a personal reference document about the Linux filesystem.

**Tasks:**

1. **Explore** — Visit each of these directories and list at least 2 interesting files inside:
   - `/etc`
   - `/var/log`
   - `/proc`
   - `/usr/bin`

2. **Document** — Create a markdown file at `~/devops-journey/day-01/notes/filesystem-notes.md` with your findings:

```bash
cat > ~/devops-journey/day-01/notes/filesystem-notes.md << 'EOF'
# My Linux Filesystem Notes — Day 1

## /etc — Configuration Files
Files I found:
- 

## /var/log — System Logs
Files I found:
-

## /proc — Process Information
Interesting things:
-

## /usr/bin — User Programs
Commands I recognize:
-

## My Questions
1.
2.
EOF
```

3. **Investigate** these commands and note what each tells you:
```bash
cat /proc/cpuinfo | head -20
cat /proc/meminfo | head -10
cat /proc/version
ls /proc/ | head -20
```

4. **Reflect** — Add a section to your notes file answering:
   - What surprised you most about the Linux filesystem?
   - How is it different from Windows or macOS (if you've used those)?
   - What directory do you think will matter most for a DevOps engineer?

**Deliverable:** Share your `filesystem-notes.md` with a colleague or post a screenshot in your learning community.

---

## Glossary

| Term | Definition |
|------|-----------|
| **Kernel** | The core of the OS that manages hardware resources |
| **Shell** | A command-line interface (CLI) to interact with the OS (e.g., Bash) |
| **Bash** | Bourne Again SHell — the most common Linux shell |
| **Terminal / Console** | The application that provides access to the shell |
| **Distro** | A complete OS built around the Linux kernel |
| **Root** | The superuser with full system privileges; also the top `/` of the filesystem |
| **sudo** | Execute a command with root-level privileges |
| **PATH** | An environment variable listing directories where the shell looks for commands |
| **Absolute path** | A path starting from `/` (e.g., `/home/alice/file.txt`) |
| **Relative path** | A path relative to the current directory (e.g., `./file.txt`) |
| **Package manager** | A tool to install/manage software (e.g., `apt`, `yum`) |
| **Permissions** | Rules controlling who can read, write, or execute a file |
| **Process** | A running program managed by the kernel |
| **stdin / stdout / stderr** | Standard input, output, and error streams |
| **LTS** | Long-Term Support — a version with extended maintenance (e.g., Ubuntu 22.04 LTS) |
| **CLI** | Command-Line Interface — text-based interaction with the OS |
| **Flag / Option** | A modifier for a command (e.g., `-l` in `ls -l`) |

---

## Resources

### Watch (pick at least one)

- 🎥 [The Linux Command Line — Full Course for Beginners (freeCodeCamp)](https://www.youtube.com/watch?v=ZtqBQ68cfJc)
- 🎥 [Linux Tutorial for Beginners (TechWorld with Nana)](https://www.youtube.com/watch?v=wBp0Rb-ZJak)
- 🎥 [How Netflix Became A Master of DevOps — Case Study](https://www.youtube.com/results?search_query=netflix+devops+case+study)

### Read

- 📖 [The Linux Command Line (free book) — William Shotts](https://linuxcommand.org/tlcl.php)
- 📖 [Linux Journey (interactive)](https://linuxjourney.com/)
- 📖 [Explain Shell — paste any command to understand it](https://explainshell.com/)

### Practice

- 💻 [OverTheWire: Bandit — Linux wargame for beginners](https://overthewire.org/wargames/bandit/)
- 💻 [Killercoda Linux Scenarios](https://killercoda.com/learn)
- 💻 [Linux Survival — interactive browser terminal](https://linuxsurvival.com/)

---

## Day 1 Checklist

Before moving to Day 2, confirm you can do all of the following **without looking them up**:

- [ ] Explain what Linux is in one sentence
- [ ] Name 3 Linux distros and one use case for each
- [ ] Open a terminal and identify your current user and directory
- [ ] Navigate to `/etc`, `/var/log`, and back to `~` without using absolute paths
- [ ] Create a nested directory structure with `mkdir -p`
- [ ] Create, copy, move, and delete a file
- [ ] Read a file using `cat`, `head`, `tail`, and `less`
- [ ] Read the permission string on a file (e.g., `-rwxr-xr--`)
- [ ] Get help for any command using `man`
- [ ] Check disk usage with `df -h` and `du -sh`

---

## What's Next — Day 2 Preview

**Day 2: Working with Text, Permissions & Users**

Tomorrow you'll go deeper:

- Text manipulation: `grep`, `sed`, `awk`, `cut`, `sort`, `uniq`
- File permissions in practice: `chmod`, `chown`, `chgrp`
- Users and groups: `useradd`, `usermod`, `passwd`
- Pipes and redirection: `|`, `>`, `>>`, `<`
- Your first shell script

> _"The command line is not a relic of the past. It is the interface of the present — and for DevOps engineers, the superpower of the future."_

---



