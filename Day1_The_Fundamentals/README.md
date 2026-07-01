# 🐧 Complete Linux Fundamentals — The Definitive Guide

> _"To understand DevOps, you must understand Linux. To understand Linux, you must use it."_

**Skill Level:** Absolute Beginner → Confident Intermediate
**Estimated Time:** 7–10 Days (self-paced)
**Approach:** Concept → Command → Practice → Build

---

## 📋 Table of Contents

1. [What is Linux?](#1-what-is-linux)
2. [Linux Architecture](#2-linux-architecture)
3. [Linux Distributions](#3-linux-distributions)
4. [Setting Up Your Environment](#4-setting-up-your-environment)
5. [The Linux Filesystem](#5-the-linux-filesystem)
6. [Shell & Terminal Basics](#6-shell--terminal-basics)
7. [File & Directory Operations](#7-file--directory-operations)
8. [Text Viewing & Manipulation](#8-text-viewing--manipulation)
9. [File Permissions & Ownership](#9-file-permissions--ownership)
10. [Users & Groups Management](#10-users--groups-management)
11. [Process Management](#11-process-management)
12. [Package Management](#12-package-management)
13. [Networking Fundamentals](#13-networking-fundamentals)
14. [Disk & Storage Management](#14-disk--storage-management)
15. [Shell Scripting](#15-shell-scripting)
16. [Environment Variables & Configuration](#16-environment-variables--configuration)
17. [Archiving & Compression](#17-archiving--compression)
18. [System Logs & Monitoring](#18-system-logs--monitoring)
19. [SSH & Remote Access](#19-ssh--remote-access)
20. [Cron Jobs & Scheduling](#20-cron-jobs--scheduling)
21. [Practical Projects](#21-practical-projects)
22. [Complete Command Reference](#22-complete-command-reference)
23. [Glossary](#23-glossary)
24. [Resources](#24-resources)

---

## 1. What is Linux?

### 1.1 Definition

Linux is a **free, open-source operating system kernel** created by **Linus Torvalds** in 1991. It is inspired by UNIX and powers the majority of the world's servers, cloud infrastructure, mobile devices (Android), and embedded systems.

What people call "Linux" is technically a **Linux distribution** — the kernel + GNU tools + a package manager + additional software, packaged as a complete operating system.

### 1.2 Brief History

```
1969  → Unix created at Bell Labs by Ken Thompson & Dennis Ritchie
1983  → Richard Stallman starts the GNU Project (free OS)
1991  → Linus Torvalds releases Linux kernel v0.01 (hobby project, Helsinki)
1992  → Linux licensed under GPL (General Public License)
1993  → Slackware & Debian — first major distros
1994  → Linux kernel v1.0 released
1996  → Tux the penguin becomes the Linux mascot
2000  → IBM invests $1 billion in Linux
2004  → Ubuntu 4.10 released (Warty Warthog)
2008  → Android (Linux-based) releases on first smartphones
2010s → Cloud computing explodes; AWS, GCP, Azure run on Linux
2020s → Linux powers 96% of web servers, 500/500 Fortune 500, all Top 500 supercomputers
Today → 3 billion+ devices run Linux daily
```

### 1.3 Why Linux Dominates

| Reason | Explanation |
|--------|-------------|
| **Free & Open Source** | No licensing costs; anyone can inspect, modify, distribute |
| **Stability** | Servers run for years without reboots |
| **Security** | Minimal attack surface; rapid community patching |
| **Performance** | Low overhead; highly tunable |
| **Flexibility** | Runs on everything: Raspberry Pi to mainframes |
| **Community** | Millions of contributors; vast documentation |
| **DevOps Native** | Docker, K8s, Ansible, Terraform — all built for Linux |

### 1.4 Linux vs. Windows vs. macOS

| Feature | Linux | Windows | macOS |
|---------|-------|---------|-------|
| Cost | Free | $139–$199 | Free (Apple hardware required) |
| Open Source | Fully | No | Partial (Darwin kernel) |
| Server market share | ~96% | ~1.8% | ~1.9% |
| Customizability | Unlimited | Low | Medium |
| Package manager | `apt`, `yum`, `pacman` | WinGet (limited) | Homebrew |
| Shell default | Bash/Zsh | PowerShell/CMD | Zsh |
| File system | ext4, xfs, btrfs | NTFS | APFS |
| DevOps tooling | Native | Via WSL2 | Good (POSIX-compliant) |

---

## 2. Linux Architecture

### 2.1 Layered Architecture

```
╔══════════════════════════════════════════════════════╗
║              USER SPACE                              ║
║  ┌────────────────────────────────────────────────┐  ║
║  │           USER APPLICATIONS                    │  ║
║  │  (vim, firefox, docker, git, python, nginx)    │  ║
║  └────────────────────────────────────────────────┘  ║
║  ┌────────────────────────────────────────────────┐  ║
║  │              SHELL / CLI                       │  ║
║  │  (bash, zsh, sh, fish, dash)                   │  ║
║  └────────────────────────────────────────────────┘  ║
║  ┌────────────────────────────────────────────────┐  ║
║  │           SYSTEM LIBRARIES                     │  ║
║  │  (glibc, libpthread, libssl)                   │  ║
║  └────────────────────────────────────────────────┘  ║
╠══════════════════════════════════════════════════════╣
║              KERNEL SPACE                            ║
║  ┌────────────────────────────────────────────────┐  ║
║  │            LINUX KERNEL                        │  ║
║  │  ┌──────────┐ ┌──────────┐ ┌──────────────┐   │  ║
║  │  │ Process  │ │ Memory   │ │   File       │   │  ║
║  │  │ Mgmt     │ │ Mgmt     │ │   System     │   │  ║
║  │  └──────────┘ └──────────┘ └──────────────┘   │  ║
║  │  ┌──────────┐ ┌──────────┐ ┌──────────────┐   │  ║
║  │  │ Network  │ │ Device   │ │   System     │   │  ║
║  │  │ Stack    │ │ Drivers  │ │   Calls      │   │  ║
║  │  └──────────┘ └──────────┘ └──────────────┘   │  ║
║  └────────────────────────────────────────────────┘  ║
╠══════════════════════════════════════════════════════╣
║                  HARDWARE                            ║
║       CPU │ RAM │ Disk │ NIC │ GPU │ USB            ║
╚══════════════════════════════════════════════════════╝
```

### 2.2 Key Components

**Kernel** — The core of Linux. Manages:
- CPU scheduling (which process runs when)
- Memory allocation (RAM management)
- I/O operations (disk reads/writes)
- Network communications
- Security and permissions

**Shell** — Your command interpreter. Types:
```
bash    → Bourne Again Shell (most common, default on Ubuntu/CentOS)
zsh     → Z Shell (default on macOS, popular with Oh My Zsh)
sh      → POSIX-compliant shell (basic, minimal)
fish    → Friendly Interactive Shell (beginner-friendly)
dash    → Debian Almquist Shell (fast, for scripts)
```

**System Calls** — The bridge between user applications and the kernel. When you run `ls`, it makes system calls like `open()`, `read()`, `getdents()`.

**Daemons** — Background services that start at boot:
```
sshd      → SSH server
httpd     → Apache web server
cron      → Job scheduler
systemd   → Init system and service manager
```

---

## 3. Linux Distributions

### 3.1 Distribution Family Tree

```
LINUX KERNEL
│
├── Debian Family (Package: .deb | Manager: apt)
│   ├── Debian (stable, conservative — great for servers)
│   ├── Ubuntu (most popular desktop & server)
│   │   ├── Ubuntu Server 22.04 LTS ← DevOps standard
│   │   ├── Ubuntu Desktop
│   │   └── Xubuntu, Lubuntu, Kubuntu (desktop variants)
│   ├── Linux Mint (best for beginners switching from Windows)
│   ├── Kali Linux (penetration testing & security)
│   └── Raspberry Pi OS (embedded/IoT)
│
├── Red Hat Family (Package: .rpm | Manager: yum/dnf)
│   ├── RHEL — Red Hat Enterprise Linux (paid, enterprise)
│   ├── CentOS Stream (community, upstream of RHEL)
│   ├── AlmaLinux (free RHEL clone, CentOS replacement)
│   ├── Rocky Linux (free RHEL clone, by CentOS founder)
│   └── Fedora (cutting-edge, Red Hat's test bed)
│
├── SUSE Family (Package: .rpm | Manager: zypper)
│   ├── openSUSE Leap (stable)
│   ├── openSUSE Tumbleweed (rolling release)
│   └── SLES — SUSE Linux Enterprise Server (enterprise)
│
├── Arch Family (Package: .tar.zst | Manager: pacman)
│   ├── Arch Linux (DIY, minimal, bleeding-edge)
│   ├── Manjaro (Arch with easier setup)
│   └── EndeavourOS (beginner-friendly Arch)
│
└── Others
    ├── Alpine Linux (tiny ~5MB, popular in Docker containers)
    ├── Gentoo (source-based, compile everything)
    └── NixOS (declarative, reproducible)
```

### 3.2 Choosing a Distro for DevOps

| Goal | Recommended Distro |
|------|--------------------|
| Learning (beginner) | Ubuntu 22.04 LTS |
| Enterprise servers | RHEL / AlmaLinux / Rocky Linux |
| Docker base images | Alpine Linux |
| Security testing | Kali Linux |
| Cloud VMs (AWS/GCP/Azure) | Ubuntu or Amazon Linux 2 |
| Containers (K8s nodes) | Ubuntu or Bottlerocket |

---

## 4. Setting Up Your Environment

### 4.1 Option A: WSL2 on Windows (Recommended for Windows Users)

```powershell
# Step 1: Open PowerShell as Administrator
wsl --install

# Step 2: Reboot your computer

# Step 3: Launch Ubuntu from Start Menu
# Create a username and password when prompted

# Step 4: Update your system
sudo apt update && sudo apt upgrade -y
```

### 4.2 Option B: VirtualBox + Ubuntu

```
1. Download VirtualBox: https://www.virtualbox.org/
2. Download Ubuntu 22.04 LTS ISO: https://ubuntu.com/download/server
3. Create VM:
   - Memory: 4096 MB (4 GB)
   - CPU: 2 cores
   - Disk: 25 GB (dynamically allocated)
   - Network: NAT (or Bridged for network access)
4. Mount ISO → Install Ubuntu → Remove ISO → Reboot
```

### 4.3 Option C: Cloud (Zero Setup)

```
AWS Free Tier:
  → EC2 → Launch Instance → Ubuntu 22.04 → t2.micro → Launch
  → Connect via SSH or EC2 Instance Connect

Google Cloud Shell:
  → https://shell.cloud.google.com (free, no setup)

Killercoda:
  → https://killercoda.com/learn (browser-based scenarios)

Play with Docker:
  → https://labs.play-with-docker.com (Alpine Linux terminal)
```

### 4.4 Verify Your Setup

```bash
# Run these immediately after setup
uname -a              # Kernel version and architecture
cat /etc/os-release   # Distro name and version
lsb_release -a        # More detailed distro info (Ubuntu/Debian)
hostname              # Machine name
whoami                # Current user
id                    # User ID, group memberships
uptime                # System uptime
free -h               # RAM available
df -h /               # Root disk space
```

---

## 5. The Linux Filesystem

### 5.1 Filesystem Hierarchy Standard (FHS)

```
/                         ← Root: the top of EVERYTHING
├── bin/                  ← Essential user binaries (ls, cp, mv, cat)
├── sbin/                 ← System binaries (for root: fdisk, iptables)
├── boot/                 ← Boot loader files (GRUB, Linux kernel vmlinuz)
├── dev/                  ← Device files (sda, tty, null, zero, random)
├── etc/                  ← System-wide configuration files
│   ├── passwd            ← User account info
│   ├── shadow            ← Hashed passwords (root-only)
│   ├── hosts             ← Static hostname resolution
│   ├── fstab             ← Filesystem mount table
│   ├── crontab           ← System-wide cron jobs
│   └── ssh/              ← SSH server config
├── home/                 ← User home directories
│   ├── alice/            ← Alice's home: /home/alice
│   └── bob/              ← Bob's home: /home/bob
├── lib/                  ← Shared libraries for /bin and /sbin
├── lib64/                ← 64-bit libraries
├── media/                ← Mount point for removable media (USB, CD)
├── mnt/                  ← Temporary mount point for filesystems
├── opt/                  ← Optional third-party software
├── proc/                 ← Virtual FS: kernel & process info (live data)
│   ├── cpuinfo           ← CPU information
│   ├── meminfo           ← Memory information
│   ├── [PID]/            ← Info about each running process
│   └── version           ← Kernel version string
├── root/                 ← Root user's home directory
├── run/                  ← Runtime data (PID files, sockets)
├── srv/                  ← Data served by this system (web, FTP)
├── sys/                  ← Virtual FS: hardware/kernel interface
├── tmp/                  ← Temporary files (CLEARED on reboot)
├── usr/                  ← Secondary hierarchy for user programs
│   ├── bin/              ← Most user commands (python3, git, vim)
│   ├── sbin/             ← Non-essential system binaries
│   ├── lib/              ← Libraries for /usr/bin
│   ├── local/            ← Locally installed software (takes priority)
│   │   ├── bin/
│   │   └── lib/
│   └── share/            ← Architecture-independent data (docs, icons)
└── var/                  ← Variable data that changes at runtime
    ├── log/              ← Log files (syslog, auth.log, messages)
    ├── cache/            ← Application cache data
    ├── spool/            ← Mail, print queues
    ├── lib/              ← Persistent application state
    └── www/              ← Web server document root (common convention)
```

### 5.2 Everything is a File

Linux treats everything as a file — even hardware:

```bash
/dev/sda          # Your first hard drive
/dev/sda1         # First partition on sda
/dev/null         # Blackhole: discard anything written to it
/dev/zero         # Infinite source of null bytes
/dev/random       # Random data generator
/dev/tty          # Current terminal
/dev/stdin        # Standard input (keyboard)
/dev/stdout       # Standard output (screen)
```

### 5.3 Path Types

```bash
# Absolute path — always starts from /
/home/alice/projects/app.py
/etc/nginx/nginx.conf
/var/log/syslog

# Relative path — relative to current directory
./app.py              # Same as: current_dir/app.py
../config/app.conf    # Up one level, then into config/
../../                # Up two levels

# Special shortcuts
~                     # Your home directory (/home/yourusername)
~alice                # Alice's home directory (/home/alice)
.                     # Current directory
..                    # Parent directory
-                     # Previous directory (cd -)
```

### 5.4 Practical Exploration

```bash
# Explore the filesystem structure
ls /
ls /etc | head -20
ls /var/log
ls /proc | head -20

# Find interesting files
cat /etc/hostname          # Your machine's name
cat /etc/hosts             # Local DNS mappings
cat /etc/os-release        # Distro info
cat /proc/version          # Kernel info
cat /proc/cpuinfo          # CPU details
cat /proc/meminfo          # Memory info

# See how much space directories use
du -sh /var/log
du -sh /etc
du -sh /usr

# Find the largest files on your system
du -ah / 2>/dev/null | sort -rh | head -20
```

---

## 6. Shell & Terminal Basics

### 6.1 Understanding the Prompt

```bash
alice@ubuntu:~$
│     │      │ └── $ = regular user  (# = root)
│     │      └──── ~ = current directory (home)
│     └─────────── ubuntu = hostname
└───────────────── alice = username

# When running as root:
root@ubuntu:/etc#
```

### 6.2 Keyboard Shortcuts (Master These!)

```
Ctrl + C          → Kill/interrupt the running command
Ctrl + Z          → Suspend the running command (puts it in background)
Ctrl + D          → Send EOF (end of input) / logout of shell
Ctrl + L          → Clear the terminal (same as 'clear')
Ctrl + A          → Move cursor to beginning of line
Ctrl + E          → Move cursor to end of line
Ctrl + U          → Delete from cursor to beginning of line
Ctrl + K          → Delete from cursor to end of line
Ctrl + W          → Delete the word before cursor
Ctrl + R          → Reverse search through command history
Tab               → Autocomplete command, filename, or path
Tab Tab           → Show all completions when ambiguous
↑ / ↓            → Scroll through command history
!!                → Repeat last command
!$                → Last argument of previous command
!git              → Run last command that started with 'git'
```

### 6.3 Getting Help

```bash
# Manual pages (most comprehensive)
man ls                    # Manual for ls
man 5 passwd              # Section 5: File formats (passwd file)
man -k "search term"      # Search man pages by keyword
man -f ls                 # Show all man pages for 'ls'

# Quick help
ls --help                 # Built-in help flag
help cd                   # Help for shell built-ins (bash)

# Info pages (more detailed than man)
info bash
info coreutils

# Type: find out what kind of command something is
type ls                   # ls is aliased to 'ls --color=auto'
type cd                   # cd is a shell builtin
type python3              # python3 is /usr/bin/python3

# Which: find the location of a command
which git
which python3
which bash

# Whereis: find binary, source, and man pages
whereis nginx
```

### 6.4 Command Structure

```bash
command  [options/flags]  [arguments]

ls       -la              /home/alice
│         │               └── argument: what to act on
│         └───────────────── flags: how to do it
└─────────────────────────── command: what to do

# Options can be:
ls -l               # Short form (single dash, single letter)
ls --long           # Long form (double dash, full word)
ls -l -a            # Multiple short options (separate)
ls -la              # Multiple short options (combined)
ls -l --all         # Mix of short and long
```

### 6.5 Input/Output & Redirection

```bash
# Three standard streams
stdin  (0)  → keyboard input
stdout (1)  → screen output (normal)
stderr (2)  → screen output (errors)

# Redirection operators
command > file.txt          # Redirect stdout to file (OVERWRITE)
command >> file.txt         # Redirect stdout to file (APPEND)
command < file.txt          # Read stdin from file
command 2> errors.txt       # Redirect stderr to file
command 2>&1                # Redirect stderr to same place as stdout
command > output.txt 2>&1   # Both stdout and stderr to file
command &> output.txt       # Shorthand for both (bash 4+)
command > /dev/null 2>&1    # Discard ALL output (silence command)

# Pipes: chain commands
command1 | command2         # stdout of cmd1 → stdin of cmd2
ls -la | grep ".txt"        # List files, filter for .txt
cat file.txt | sort | uniq  # Read, sort, remove duplicates
ps aux | grep nginx | grep -v grep  # Find nginx processes

# Practical examples
echo "Hello World" > greeting.txt     # Create file with content
echo "Second line" >> greeting.txt    # Append to file
cat < greeting.txt                    # Read file as stdin
ls /nonexistent 2>/dev/null           # Suppress error message
find / -name "*.log" 2>/dev/null      # Suppress permission errors
```

### 6.6 Command History

```bash
history                     # Show all recent commands
history 20                  # Show last 20 commands
history | grep "git"        # Search history for git commands
!142                        # Run command #142 from history
!!                          # Repeat last command
sudo !!                     # Repeat last command with sudo

# History config in ~/.bashrc
HISTSIZE=10000              # Commands kept in memory
HISTFILESIZE=20000          # Commands saved to disk
HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S "  # Add timestamps

# Clear history
history -c                  # Clear session history
history -d 142              # Delete entry #142
```

---

## 7. File & Directory Operations

### 7.1 Navigation Commands

```bash
pwd                         # Print Working Directory
cd /var/log                 # Change to absolute path
cd logs                     # Change to relative path
cd ~                        # Go to home directory
cd                          # Go to home directory (same as cd ~)
cd ..                       # Go up one level
cd ../..                    # Go up two levels
cd -                        # Go to PREVIOUS directory
ls                          # List current directory
ls /etc                     # List specific directory
ls -l                       # Long format (permissions, owner, size, date)
ls -a                       # Show ALL files (including hidden .files)
ls -la                      # Long format + all files
ls -lh                      # Long format + human-readable sizes
ls -lt                      # Sort by modification time (newest first)
ls -lS                      # Sort by file size (largest first)
ls -lr                      # Reverse order
ls -R                       # Recursive (list subdirectories too)
ls --color=auto             # Colorized output
```

### 7.2 Creating Files & Directories

```bash
# Create files
touch file.txt              # Create empty file (or update timestamp)
touch file1.txt file2.txt   # Create multiple files at once
touch {a,b,c}.txt           # Brace expansion: creates a.txt b.txt c.txt

# Create directories
mkdir projects              # Create single directory
mkdir -p projects/web/src   # Create nested directories (no error if exists)
mkdir -p project/{src,tests,docs,config}  # Create multiple subdirs

# Create files with content
echo "Hello World" > hello.txt
echo -e "Line 1\nLine 2\nLine 3" > multiline.txt
cat > poem.txt << 'EOF'
Roses are red,
Violets are blue,
Linux is power,
And so are you.
EOF

printf "Name: %s\nAge: %d\n" "Alice" 30 > info.txt
```

### 7.3 Copying Files & Directories

```bash
# Copy file
cp source.txt destination.txt
cp file.txt /backup/file.txt          # Copy to different location
cp file.txt /backup/                  # Copy to directory (keep filename)

# Copy multiple files
cp file1.txt file2.txt /backup/
cp *.txt /backup/                     # Copy all .txt files
cp {file1,file2}.txt /backup/         # Copy specific files

# Copy directory
cp -r projects/ projects_backup/      # -r = recursive (required for dirs)
cp -rp projects/ backup/              # -p = preserve permissions & timestamps
cp -ru source/ destination/           # -u = only copy if source is newer
cp -v source.txt dest.txt             # -v = verbose (show what's being copied)

# Copy with progress (large files)
rsync -ah --progress source/ destination/
```

### 7.4 Moving & Renaming

```bash
# Move file
mv oldname.txt newname.txt            # Rename file
mv file.txt /home/alice/              # Move to directory
mv file.txt /home/alice/newname.txt   # Move and rename

# Move multiple files
mv *.txt /backup/
mv file{1,2,3}.txt /backup/

# Move directory
mv olddir/ newdir/                    # Rename directory
mv projects/ /home/alice/work/        # Move directory

# Safe move (don't overwrite)
mv -n source.txt dest.txt             # -n = no-clobber (won't overwrite)
mv -i source.txt dest.txt             # -i = interactive (ask before overwrite)
mv -v source.txt dest.txt             # -v = verbose
mv -b source.txt dest.txt             # -b = backup existing file before overwrite
```

### 7.5 Deleting Files & Directories

```bash
# Remove files
rm file.txt
rm file1.txt file2.txt
rm *.log                              # Remove all .log files
rm -f file.txt                        # Force (no error if doesn't exist)
rm -i file.txt                        # Interactive (ask before each delete)
rm -v file.txt                        # Verbose

# Remove directories
rmdir emptydir/                       # Remove EMPTY directory only
rm -r projects/                       # Remove directory and all contents
rm -rf projects/                      # Force remove (no prompts) — DANGEROUS

# ⚠️  NEVER RUN THESE:
# rm -rf /          → Destroys the entire system
# rm -rf *          → Destroys current directory (dangerous if run as root)
# rm -rf ./         → Same as above
# Always double-check your path before rm -rf!

# Safer alternative: move to trash first
mv important_file.txt ~/.trash/
# Or use trash-cli package
sudo apt install trash-cli
trash-put file.txt                    # Move to trash
trash-list                            # List trash
trash-restore                         # Restore from trash
```

### 7.6 Finding Files

```bash
# find — the most powerful file search
find /path -name "filename"           # Find by exact name
find / -name "*.conf"                 # Find by pattern
find . -name "*.py" -type f          # Only files, not dirs
find . -type d                        # Only directories
find . -type l                        # Only symbolic links
find /var/log -name "*.log" -mtime -7 # Modified in last 7 days
find . -size +100M                    # Files larger than 100MB
find . -size -1k                      # Files smaller than 1KB
find . -empty                         # Empty files and directories
find . -perm 777                      # Files with specific permissions
find . -user alice                    # Files owned by alice
find . -group developers              # Files owned by group
find . -name "*.txt" -delete          # Find and delete (be careful!)
find . -name "*.log" -exec rm {} \;  # Find and execute command on each
find . -name "*.py" -exec grep -l "import os" {} \;  # Find Python files that import os

# locate — faster but uses a database (update with updatedb)
sudo updatedb
locate nginx.conf
locate -i "readme"                    # Case-insensitive

# which / whereis — find commands
which python3
whereis nginx

# grep — search inside files
grep "error" /var/log/syslog          # Find "error" in file
grep -r "TODO" /home/alice/projects/ # Recursive search in directory
grep -i "Error" file.txt              # Case-insensitive
grep -n "pattern" file.txt            # Show line numbers
grep -v "debug" logfile.txt           # Invert: lines NOT matching
grep -l "pattern" *.txt               # Only show filenames, not matches
grep -c "error" logfile.txt           # Count matching lines
grep -A 3 "error" log.txt            # Show 3 lines AFTER match
grep -B 3 "error" log.txt            # Show 3 lines BEFORE match
grep -C 3 "error" log.txt            # Show 3 lines BEFORE and AFTER
grep -E "err|warn|crit" log.txt      # Extended regex (multiple patterns)
```

### 7.7 Links (Symbolic & Hard)

```bash
# Symbolic links (like shortcuts/aliases)
ln -s /path/to/original /path/to/link
ln -s /usr/local/bin/python3.11 /usr/local/bin/python3
ln -s /var/www/html /home/alice/www   # Link a directory

ls -la                                # Links show as: link -> target
readlink /usr/bin/python3             # Show what link points to
readlink -f /usr/bin/python3          # Show absolute resolved path

# Hard links (another name for the same data)
ln original.txt hardlink.txt
# Hard links share the same inode (same data on disk)
# Deleting original doesn't delete data until ALL hard links are removed
# Cannot hard link across filesystems or directories

# Remove a link
rm linkname                           # Remove symbolic link
unlink linkname                       # Same thing
```

---

## 8. Text Viewing & Manipulation

### 8.1 Viewing Files

```bash
# cat — concatenate and print
cat file.txt                          # Print entire file
cat -n file.txt                       # With line numbers
cat -A file.txt                       # Show special chars ($ at end of lines)
cat file1.txt file2.txt               # Concatenate two files
cat file1.txt file2.txt > combined.txt # Merge files

# less — interactive pager (use this for large files)
less /var/log/syslog
# Inside less:
#   Space / PgDn     → Next page
#   b / PgUp         → Previous page
#   G                → Go to end
#   g                → Go to beginning
#   /pattern         → Search forward
#   ?pattern         → Search backward
#   n                → Next search result
#   N                → Previous search result
#   q                → Quit

# more — simpler pager (less featureful than less)
more file.txt

# head — view beginning of file
head file.txt                         # First 10 lines (default)
head -20 file.txt                     # First 20 lines
head -c 100 file.txt                  # First 100 bytes

# tail — view end of file
tail file.txt                         # Last 10 lines
tail -20 file.txt                     # Last 20 lines
tail -f /var/log/syslog               # Follow file in real-time (watch for new lines)
tail -f -n 50 /var/log/nginx/access.log  # Last 50 lines, then follow

# Watch a file for changes
watch -n 1 tail -n 20 /var/log/syslog  # Refresh every 1 second
```

### 8.2 Text Manipulation Tools

```bash
# grep — search/filter lines
grep "ERROR" app.log
grep -E "^[0-9]{4}-[0-9]{2}" log.txt     # Regex: lines starting with date
grep -v "^#" /etc/ssh/sshd_config         # Exclude comments

# cut — extract columns/fields
cut -d: -f1 /etc/passwd                   # Extract first field (username)
cut -d: -f1,3 /etc/passwd                 # Extract fields 1 and 3
cut -c1-10 file.txt                       # Extract first 10 characters of each line
echo "hello world" | cut -d' ' -f2       # Extract second word

# sort — sort lines
sort file.txt                             # Alphabetical sort
sort -r file.txt                          # Reverse sort
sort -n numbers.txt                       # Numeric sort
sort -k2 -t: /etc/passwd                 # Sort by 2nd field, using : as delimiter
sort -u file.txt                          # Sort and remove duplicates

# uniq — remove or identify duplicates (must be sorted first)
sort file.txt | uniq                      # Remove duplicate lines
sort file.txt | uniq -c                   # Count occurrences of each line
sort file.txt | uniq -d                   # Show only duplicate lines
sort file.txt | uniq -u                   # Show only unique lines

# wc — word/line/character/byte count
wc file.txt                               # Lines, words, bytes
wc -l file.txt                            # Line count only
wc -w file.txt                            # Word count only
wc -c file.txt                            # Byte count only
wc -m file.txt                            # Character count only
ls /etc | wc -l                           # Count files in /etc

# tr — translate or delete characters
echo "hello" | tr 'a-z' 'A-Z'            # Lowercase to uppercase
echo "Hello World" | tr -d ' '           # Delete spaces
echo "hello   world" | tr -s ' '         # Squeeze multiple spaces to one
cat file.txt | tr '\n' ','               # Replace newlines with commas

# sed — stream editor (find and replace)
sed 's/old/new/' file.txt                 # Replace first occurrence per line
sed 's/old/new/g' file.txt               # Replace ALL occurrences per line
sed 's/old/new/gi' file.txt              # Case-insensitive replace all
sed -i 's/old/new/g' file.txt            # Edit FILE IN PLACE (overwrites)
sed -i.bak 's/old/new/g' file.txt        # Edit in place, save backup as .bak
sed '/^#/d' file.txt                      # Delete lines starting with #
sed '/^$/d' file.txt                      # Delete empty lines
sed -n '5,10p' file.txt                   # Print only lines 5-10
sed '1d' file.txt                         # Delete first line
sed '$d' file.txt                         # Delete last line
sed 's/^/  /' file.txt                   # Add 2 spaces indent to every line

# awk — pattern scanning and text processing
awk '{print $1}' file.txt                 # Print first column
awk '{print $1, $3}' file.txt            # Print columns 1 and 3
awk -F: '{print $1}' /etc/passwd         # Use : as delimiter, print first field
awk -F: '{print $1, $3}' /etc/passwd    # Username and UID
awk 'NR==5' file.txt                      # Print line 5
awk 'NR>=5 && NR<=10' file.txt           # Print lines 5-10
awk '/pattern/ {print}' file.txt          # Print lines matching pattern
awk '{sum += $1} END {print sum}' nums.txt  # Sum first column
awk '{print NR": "$0}' file.txt           # Add line numbers
awk -F: '$3 >= 1000 {print $1}' /etc/passwd  # Regular users (UID >= 1000)

# paste — merge lines from files
paste file1.txt file2.txt                 # Side by side (tab-separated)
paste -d, file1.txt file2.txt            # Use comma as delimiter

# diff — compare files
diff file1.txt file2.txt                  # Show differences
diff -u file1.txt file2.txt              # Unified format (easier to read)
diff -r dir1/ dir2/                       # Compare directories recursively

# xargs — build commands from stdin
find . -name "*.log" | xargs rm          # Delete all found .log files
find . -name "*.txt" | xargs wc -l       # Count lines in all .txt files
cat urls.txt | xargs -I {} curl -O {}    # Download each URL
echo "file1 file2 file3" | xargs -n1 touch  # Create each file
```

### 8.3 Text Editors

```bash
# nano — beginner-friendly (key combos shown at bottom)
nano filename.txt
# Ctrl+O → Save (Write Out)
# Ctrl+X → Exit
# Ctrl+W → Search
# Ctrl+K → Cut line
# Ctrl+U → Paste (Uncut)

# vim — powerful, modal editor (industry standard)
vim filename.txt
# Modes:
#   Normal mode  (default) — for navigation and commands
#   Insert mode  (press i) — for typing text
#   Visual mode  (press v) — for selecting text
#   Command mode (press :) — for saving, quitting, etc.

# vim Quick Reference:
# i           → Enter Insert mode at cursor
# a           → Enter Insert mode after cursor
# o           → New line below and enter Insert mode
# ESC         → Return to Normal mode
# :w          → Save file
# :q          → Quit (fails if unsaved changes)
# :wq or ZZ   → Save and quit
# :q!         → Quit without saving (force)
# :wq!        → Save and quit (force)
# dd          → Delete (cut) current line
# yy          → Yank (copy) current line
# p           → Paste after cursor
# P           → Paste before cursor
# u           → Undo
# Ctrl+R      → Redo
# /pattern    → Search forward
# n           → Next match
# :%s/old/new/g → Replace all in file
# gg          → Go to first line
# G           → Go to last line
# :42         → Go to line 42

# Vim crash course commands
vim practice.txt    # Open file
# Press i           → Start typing
# Type: Hello Linux World
# Press ESC         → Exit insert mode
# Type: :wq        → Save and quit
```

---

## 9. File Permissions & Ownership

### 9.1 Understanding Permissions

```
ls -la output:
-rwxr-xr--  2  alice  devops  4096  Jun 10 09:00  script.sh
│└─┬─┘└─┬─┘└─┬─┘  │    │      │      │               │
│  │    │    │    │    │      │      │               └── Filename
│  │    │    │    │    │      │      └── Modification time
│  │    │    │    │    │      └── File size (bytes)
│  │    │    │    │    └── Group owner
│  │    │    │    └── User owner
│  │    │    └── Hard link count
│  │    └── Others' permissions
│  └── Group's permissions
└── File type + Owner's permissions

File types:
-   → Regular file
d   → Directory
l   → Symbolic link
c   → Character device
b   → Block device
p   → Named pipe (FIFO)
s   → Socket

Permission characters:
r (4) → Read
w (2) → Write
x (1) → Execute (for files) / Enter + list (for directories)
- (0) → No permission

For directories:
r → Can list contents (ls)
w → Can create/delete files inside
x → Can enter directory (cd) and access files inside
```

### 9.2 Reading Permissions

```bash
-rwxr-xr--

Position 1:     -          File type (- = regular file)
Positions 2-4:  rwx        Owner: read + write + execute = 7
Positions 5-7:  r-x        Group: read + execute (no write) = 5
Positions 8-10: r--        Others: read only = 4

Octal value: 754

# Common permission patterns
-rw-r--r--  (644)  → Regular files (readable by all, writable by owner)
-rwxr-xr-x  (755)  → Executables, directories (readable/executable by all)
-rw-------  (600)  → Private files (owner only)
-rwx------  (700)  → Private executables/directories
-rw-rw-r--  (664)  → Shared project files (group can write)
-rwxrwxr-x  (775)  → Shared project dirs (group can write)
-rwsrwsr-x  (????) → SetUID/SetGID bits
```

### 9.3 Changing Permissions with chmod

```bash
# Symbolic mode
chmod u+x script.sh          # Add execute for user (owner)
chmod g+w file.txt           # Add write for group
chmod o-r file.txt           # Remove read from others
chmod a+r file.txt           # Add read for all (a = u+g+o)
chmod u+x,g-w file.txt       # Multiple changes at once
chmod ug+rw file.txt         # Add read+write for user and group

# Octal mode (faster once you know it)
chmod 755 script.sh          # rwxr-xr-x
chmod 644 document.txt       # rw-r--r--
chmod 600 private.key        # rw-------
chmod 777 shared_dir/        # rwxrwxrwx (avoid — everyone can do everything)
chmod 700 ~/.ssh             # rwx------ (SSH dir should be private)

# Recursive (apply to dir and all contents)
chmod -R 755 /var/www/html/  # Make web directory accessible

# Common octal values:
# 7 = rwx (4+2+1)
# 6 = rw- (4+2+0)
# 5 = r-x (4+0+1)
# 4 = r-- (4+0+0)
# 0 = --- (0+0+0)

# Quick reference table:
# 000 = ---  400 = r--  600 = rw-  700 = rwx
# 644 = rw-r--r--  (default files)
# 755 = rwxr-xr-x  (default dirs/scripts)
# 777 = rwxrwxrwx  (world-writable, dangerous)
```

### 9.4 Changing Ownership with chown & chgrp

```bash
# chown — change owner
chown alice file.txt               # Change owner to alice
chown alice:devops file.txt        # Change owner to alice, group to devops
chown :devops file.txt             # Change only group to devops
chown -R alice:devops /project/    # Recursive ownership change

# chgrp — change group only
chgrp devops file.txt
chgrp -R developers /shared/

# Check ownership
ls -la file.txt
stat file.txt                      # Detailed file info including ownership
```

### 9.5 Special Permissions

```bash
# SetUID (SUID) — execute as file owner, not the user running it
chmod u+s script.sh         # Set SUID
chmod 4755 script.sh        # Octal: 4xxx
# Example: /usr/bin/passwd has SUID (allows users to change their own password)
ls -la /usr/bin/passwd      # Shows -rwsr-xr-x (s in owner execute position)

# SetGID (SGID) — on dirs: new files inherit group of directory
chmod g+s /shared/          # Set SGID on directory
chmod 2755 /shared/         # Octal: 2xxx
# All files created in /shared will inherit its group

# Sticky Bit — on dirs: only file owner can delete their own files
chmod +t /tmp               # Set sticky bit
chmod 1777 /tmp             # Octal: 1xxx (typical for /tmp)
ls -la /tmp                 # Shows drwxrwxrwt (t at end)
# Even if permissions allow it, only the owner can delete their files

# View special permissions
ls -la /usr/bin/passwd      # SUID:   -rwsr-xr-x
ls -la /usr/bin/wall        # SGID:   -rwxr-sr-x
ls -la /tmp                 # Sticky: drwxrwxrwt
```

### 9.6 umask — Default Permission Mask

```bash
# umask sets the REMOVAL mask for new file/directory permissions
umask                       # Show current umask (e.g., 0022)

# Default creation permissions:
# Files:       666 (rw-rw-rw-)
# Directories: 777 (rwxrwxrwx)

# With umask 022:
# New files:   666 - 022 = 644 (rw-r--r--)
# New dirs:    777 - 022 = 755 (rwxr-xr-x)

# Set a new umask (current session)
umask 027                   # New files: 640, New dirs: 750

# Set permanently in ~/.bashrc:
echo "umask 027" >> ~/.bashrc
```

---

## 10. Users & Groups Management

### 10.1 User Information

```bash
# Who am I?
whoami                      # Print current username
id                          # Print UID, GID, and all group memberships
id alice                    # Info about specific user
id -u                       # Just the UID
id -g                       # Just the primary GID
id -G                       # All group IDs
id -nG                      # All group names

# Who else is logged in?
who                         # Users currently logged in
w                           # Logged in users + what they're doing
last                        # Login history
last alice                  # Login history for alice
lastlog                     # Last login for all users
finger alice                # Detailed user info (if installed)

# User account files
cat /etc/passwd             # All user accounts
# Format: username:x:UID:GID:comment:home:shell
cat /etc/shadow             # Hashed passwords (root only)
cat /etc/group              # All groups
# Format: groupname:x:GID:member1,member2
```

### 10.2 Managing Users

```bash
# Create user
sudo useradd alice                        # Create user (basic)
sudo useradd -m alice                     # Create with home directory
sudo useradd -m -s /bin/bash alice        # With home dir and bash shell
sudo useradd -m -s /bin/bash -G sudo,docker alice  # With groups
sudo useradd -m -c "Alice Smith" -s /bin/bash alice  # With comment/full name
sudo useradd -u 1500 alice               # Set specific UID

# Set password
sudo passwd alice                         # Set alice's password
sudo passwd                               # Change root's password

# Modify user
sudo usermod -s /bin/zsh alice           # Change shell
sudo usermod -aG docker alice            # Add to group (a = append, G = groups)
sudo usermod -aG sudo alice              # Add to sudo group (give admin)
sudo usermod -L alice                    # Lock account (disable login)
sudo usermod -U alice                    # Unlock account
sudo usermod -e 2025-12-31 alice         # Set account expiry date
sudo usermod -d /new/home alice          # Change home directory

# Delete user
sudo userdel alice                        # Delete user (keep home dir)
sudo userdel -r alice                     # Delete user AND home directory

# Switch users
su alice                                  # Switch to alice (keeps environment)
su - alice                               # Switch to alice (full login shell)
sudo -u alice command                    # Run single command as alice
sudo -i                                  # Open root shell
sudo su                                  # Switch to root
exit                                     # Return to previous user
```

### 10.3 Managing Groups

```bash
# Create group
sudo groupadd devops
sudo groupadd -g 2000 devops             # With specific GID

# Modify group
sudo groupmod -n newname oldname         # Rename group
sudo groupmod -g 2500 devops             # Change GID

# Add user to group
sudo usermod -aG devops alice            # Add alice to devops
sudo gpasswd -a alice devops             # Same result

# Remove user from group
sudo gpasswd -d alice devops

# Delete group
sudo groupdel devops

# View group membership
groups alice                             # Groups alice belongs to
getent group devops                      # Members of devops group
cat /etc/group | grep devops
```

### 10.4 sudo — Privilege Escalation

```bash
# Run commands as root
sudo apt update                          # Run apt as root
sudo systemctl restart nginx             # Restart service as root

# Edit sudoers file (ALWAYS use visudo, never edit directly)
sudo visudo

# sudoers syntax:
# user  ALL=(ALL:ALL) ALL
# │      │    │   │   └── Commands allowed
# │      │    │   └── Group to run as
# │      │    └── User to run as
# │      └── From which hosts
# └── Who this applies to

# Common entries to add in /etc/sudoers.d/alice:
alice ALL=(ALL:ALL) ALL                  # Full sudo access
alice ALL=(ALL:ALL) NOPASSWD: ALL        # No password required
alice ALL=(ALL:ALL) /usr/bin/apt         # Only allow specific command
%devops ALL=(ALL:ALL) ALL               # All members of devops group
```

---

## 11. Process Management

### 11.1 Viewing Processes

```bash
# ps — snapshot of current processes
ps                          # Your processes (current terminal)
ps aux                      # All processes, all users (BSD format)
ps -ef                      # All processes (UNIX format)
ps aux | grep nginx         # Find nginx processes
ps -u alice                 # Processes owned by alice
ps --sort=-%cpu | head      # Top CPU consumers
ps --sort=-%mem | head      # Top memory consumers

# ps output columns (ps aux):
# USER  PID  %CPU  %MEM  VSZ   RSS   TTY  STAT  START  TIME  COMMAND
# VSZ = Virtual memory size (total allocated)
# RSS = Resident Set Size (RAM actually used)
# STAT: R=Running, S=Sleeping, D=Uninterruptible sleep, Z=Zombie, T=Stopped

# top — interactive process viewer
top
# Inside top:
#   q     → Quit
#   k     → Kill process (enter PID)
#   r     → Renice (change priority)
#   M     → Sort by memory
#   P     → Sort by CPU
#   1     → Show individual CPUs
#   h     → Help

# htop — better top (install first: sudo apt install htop)
htop

# pgrep — find process by name
pgrep nginx                 # Print PIDs of processes named nginx
pgrep -l nginx              # Print PID and name
pgrep -u alice              # Processes owned by alice

# pstree — visual process tree
pstree
pstree alice                # Processes for specific user
pstree -p                   # Show PIDs
```

### 11.2 Signals & Killing Processes

```bash
# Signals
kill -l                     # List all signals
# Common signals:
# SIGTERM (15) — Graceful termination (please stop when ready)
# SIGKILL (9)  — Forceful kill (cannot be caught or ignored)
# SIGHUP  (1)  — Hang up / reload config
# SIGSTOP (19) — Pause process
# SIGCONT (18) — Resume paused process

# Kill by PID
kill 1234                   # Send SIGTERM (graceful)
kill -15 1234               # Explicit SIGTERM
kill -9 1234                # SIGKILL (force, last resort)
kill -1 1234                # SIGHUP (reload config)

# Kill by name
pkill nginx                 # Kill all processes named nginx
pkill -u alice              # Kill all of alice's processes
killall nginx               # Kill all nginx processes

# Background & foreground jobs
command &                   # Run in background
command > /dev/null 2>&1 &  # Background with no output
Ctrl + Z                    # Suspend foreground job
bg                          # Resume suspended job in background
fg                          # Bring background job to foreground
fg %2                       # Bring job #2 to foreground
jobs                        # List background jobs
nohup command &             # Run immune to hangup (survives logout)
disown %1                   # Disown job (won't be killed on logout)
```

### 11.3 Process Priority (nice & renice)

```bash
# Nice values: -20 (highest priority) to 19 (lowest)
# Default is 0

nice -n 10 ./script.sh               # Run with lower priority
nice -n -5 ./important_script.sh     # Run with higher priority (root only for negative)
sudo nice -n -20 ./critical.sh       # Highest priority

renice -n 5 -p 1234                  # Change priority of running process
renice -n 5 -u alice                 # Change priority of all alice's processes
```

### 11.4 System Resource Monitoring

```bash
# CPU and memory
free -h                     # Memory usage (human-readable)
vmstat 1 5                  # VM stats every 1 second, 5 times
mpstat 1 5                  # CPU stats per processor

# Disk I/O
iostat                      # I/O stats (install sysstat)
iostat -x 1                 # Extended stats, every 1 second
iotop                       # Real-time I/O by process (like top for I/O)

# System load
uptime                      # Load averages (1, 5, 15 minutes)
# Load average of 1.0 on a single-core = 100% busy
# Load average of 2.0 on a dual-core = 100% busy

cat /proc/loadavg
cat /proc/cpuinfo | grep "model name" | head -1
nproc                       # Number of CPU cores
lscpu                       # Detailed CPU info

# Overall system health
dmesg | tail -20            # Kernel ring buffer (hardware messages)
dmesg -T | grep -i error    # Timestamped kernel errors
```

---

## 12. Package Management

### 12.1 APT (Debian / Ubuntu)

```bash
# Update package list and upgrade
sudo apt update                         # Refresh package index
sudo apt upgrade                        # Upgrade all packages
sudo apt full-upgrade                   # Upgrade + resolve deps
sudo apt dist-upgrade                   # Major version upgrades

# Install packages
sudo apt install nginx                  # Install single package
sudo apt install nginx curl git vim     # Install multiple
sudo apt install -y nginx               # Auto-yes (non-interactive)
sudo apt install --no-install-recommends nginx  # Minimal install
sudo apt install ./package.deb         # Install local .deb file

# Remove packages
sudo apt remove nginx                   # Remove (keep configs)
sudo apt purge nginx                    # Remove + delete configs
sudo apt autoremove                     # Remove unused dependencies
sudo apt clean                          # Clear package cache

# Search & info
apt search nginx                        # Search packages
apt show nginx                          # Detailed package info
apt list --installed                    # List installed packages
apt list --installed | grep nginx       # Check if specific package installed
apt depends nginx                       # Show dependencies

# Common useful packages for DevOps
sudo apt install -y \
  curl wget git vim nano htop \
  tree net-tools netcat \
  unzip tar gzip \
  build-essential \
  python3 python3-pip \
  docker.io \
  openssh-server
```

### 12.2 YUM / DNF (Red Hat / CentOS / Fedora)

```bash
# DNF (modern, replaces yum on RHEL 8+, Fedora)
sudo dnf update                         # Update all packages
sudo dnf install nginx                  # Install package
sudo dnf remove nginx                   # Remove package
sudo dnf search nginx                   # Search
sudo dnf info nginx                     # Package info
sudo dnf list installed                 # Installed packages
sudo dnf provides /usr/bin/vim          # Which package owns this file
sudo dnf history                        # Transaction history
sudo dnf history undo last              # Undo last transaction

# YUM (older RHEL 7 / CentOS 7)
sudo yum update
sudo yum install nginx
sudo yum remove nginx
sudo yum search nginx
```

### 12.3 Snap & Flatpak

```bash
# Snap (Ubuntu)
snap find vscode                        # Search
sudo snap install code --classic        # Install
snap list                               # List installed snaps
sudo snap remove code                   # Remove

# Flatpak (universal)
flatpak search gimp
flatpak install flathub org.gimp.GIMP
flatpak list
flatpak uninstall org.gimp.GIMP
```

### 12.4 Compiling from Source

```bash
# General steps for compiling from source:
# 1. Install build tools
sudo apt install build-essential

# 2. Download and extract source
wget https://example.com/package-1.0.tar.gz
tar -xzf package-1.0.tar.gz
cd package-1.0/

# 3. Configure
./configure --prefix=/usr/local

# 4. Compile
make -j$(nproc)             # -j = parallel jobs, nproc = number of CPUs

# 5. Install
sudo make install

# 6. Verify
which newcommand
newcommand --version
```

---

## 13. Networking Fundamentals

### 13.1 Network Information

```bash
# IP addresses and interfaces
ip addr show                # Show all interfaces and IP addresses
ip addr show eth0           # Show specific interface
ip -4 addr                  # IPv4 only
ip -6 addr                  # IPv6 only
hostname -I                 # Print all IP addresses
hostname -i                 # Print main IP

# Legacy (but still widely used)
ifconfig                    # Show interfaces (install net-tools)
ifconfig eth0               # Specific interface

# Routing
ip route show               # Routing table
ip route get 8.8.8.8        # How to reach specific IP
route -n                    # Numeric routing table (legacy)

# DNS
cat /etc/resolv.conf        # DNS servers configured
cat /etc/hosts              # Local hostname resolution
nslookup google.com         # DNS lookup
dig google.com              # Detailed DNS lookup
dig google.com MX           # MX records (mail)
dig @8.8.8.8 google.com    # Use specific DNS server
host google.com             # Simple DNS lookup
```

### 13.2 Connectivity Testing

```bash
# ping — test connectivity
ping google.com             # Continuous ping
ping -c 4 google.com        # Send 4 packets
ping -i 0.5 google.com      # Ping every 0.5 seconds
ping 192.168.1.1            # Ping specific IP

# traceroute — trace route to destination
traceroute google.com
tracepath google.com        # Similar, no root required

# netstat — network connections
netstat -tuln               # TCP/UDP listening ports (no DNS)
netstat -tunlp              # With process IDs (requires root)
netstat -an | grep ESTABLISHED  # Active connections

# ss — modern replacement for netstat
ss -tuln                    # TCP/UDP listening
ss -tulnp                   # With processes
ss -s                       # Summary statistics
ss -t state established     # Only established connections

# Port scanning (localhost)
nmap localhost
nmap -sV localhost           # With service version
nmap -p 22,80,443 server    # Specific ports
```

### 13.3 Data Transfer

```bash
# curl — transfer data from/to URLs
curl https://api.example.com                    # GET request
curl -o file.html https://example.com           # Download to file
curl -O https://example.com/file.tar.gz         # Download (keep filename)
curl -L https://example.com                     # Follow redirects
curl -I https://example.com                     # Headers only
curl -d '{"key":"value"}' -H "Content-Type: application/json" \
     https://api.example.com/endpoint           # POST JSON
curl -u username:password https://example.com   # Basic auth
curl -H "Authorization: Bearer TOKEN" \
     https://api.example.com                    # Bearer token
curl --proxy http://proxy:8080 https://example.com  # Via proxy
curl -v https://example.com                     # Verbose (see request/response)

# wget — download files
wget https://example.com/file.tar.gz            # Simple download
wget -c https://example.com/largefile.tar.gz    # Resume interrupted download
wget -O newname.tar.gz https://example.com/file.tar.gz  # Custom filename
wget -r -np https://example.com/docs/           # Recursive download
wget -q --spider https://example.com            # Check if URL exists (quiet)

# scp — secure copy (over SSH)
scp file.txt alice@server:/home/alice/          # Copy to remote
scp alice@server:/home/alice/file.txt .         # Copy from remote
scp -r local_dir/ alice@server:/remote/path/   # Copy directory
scp -P 2222 file.txt alice@server:/path/        # Custom SSH port

# rsync — efficient sync/backup
rsync -av source/ destination/                  # Sync (verbose, archive)
rsync -avz source/ user@remote:/path/           # With compression
rsync -avz --progress source/ remote:/path/     # Show progress
rsync -avz --delete source/ destination/        # Delete files not in source
rsync -avz --exclude="*.log" src/ dst/          # Exclude logs
rsync --dry-run -av source/ destination/        # Preview without changes
```

### 13.4 Firewall Basics (ufw — Ubuntu)

```bash
# UFW (Uncomplicated Firewall) — Ubuntu/Debian
sudo ufw status                         # Check firewall status
sudo ufw enable                         # Enable firewall
sudo ufw disable                        # Disable firewall

sudo ufw allow 22                       # Allow SSH
sudo ufw allow 22/tcp                   # Allow SSH (TCP only)
sudo ufw allow 80                       # Allow HTTP
sudo ufw allow 443                      # Allow HTTPS
sudo ufw allow 8080/tcp                 # Custom port
sudo ufw allow from 192.168.1.0/24     # Allow subnet
sudo ufw allow from 192.168.1.0/24 to any port 22  # Specific source + port

sudo ufw deny 23                        # Deny telnet
sudo ufw delete allow 80                # Remove rule

sudo ufw logging on                     # Enable firewall logging
sudo ufw logging medium                 # Set log level

# firewalld (CentOS/RHEL)
sudo systemctl start firewalld
sudo firewall-cmd --state
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

---

## 14. Disk & Storage Management

### 14.1 Viewing Disk Usage

```bash
# Disk space (filesystem level)
df -h                               # Human-readable disk usage
df -hT                              # Include filesystem type
df -h /home                         # Specific mount point
df -i                               # Inode usage

# Directory sizes
du -sh /var/log                     # Size of a directory
du -sh /*                           # Size of each directory in /
du -sh * | sort -rh | head -20     # Top 20 largest items in current dir
du -sh /var/log/* | sort -rh       # Subdirectory sizes sorted
du -ah . | sort -rh | head -20     # Including files
du --max-depth=2 /                  # Limit depth

# ncdu — interactive disk usage (install first)
sudo apt install ncdu
ncdu /var                           # Interactive browser
```

### 14.2 Block Devices & Partitions

```bash
# List block devices
lsblk                               # Tree view of block devices
lsblk -f                            # Include filesystem type and UUID
blkid                               # Block device UUIDs and types (root)
fdisk -l                            # List partition tables (root)
parted -l                           # Modern partition tool

# Typical device naming:
# /dev/sda     → First SATA/SCSI disk
# /dev/sda1    → First partition on sda
# /dev/sdb     → Second disk
# /dev/nvme0n1 → First NVMe SSD
# /dev/vda     → Virtual disk (cloud VMs)
```

### 14.3 Mounting & Unmounting

```bash
# Mount a filesystem
sudo mount /dev/sdb1 /mnt/backup
sudo mount -t ext4 /dev/sdb1 /mnt/backup  # Specify type
sudo mount -o ro /dev/sdb1 /mnt/           # Mount read-only

# Mount ISO
sudo mount -o loop image.iso /mnt/cdrom

# Unmount
sudo umount /mnt/backup
sudo umount -f /mnt/backup              # Force unmount
sudo umount -l /mnt/backup              # Lazy unmount (unmount when not busy)

# View mounts
mount                                   # All mounted filesystems
cat /proc/mounts                        # Same info from kernel
mount | grep sdb                        # Filter specific device

# fstab — permanent mounts (auto-mount at boot)
# /etc/fstab format:
# device  mount_point  filesystem  options  dump  pass
# UUID=xxx  /data  ext4  defaults  0  2

# Find UUID for fstab
blkid /dev/sdb1
```

### 14.4 Creating Filesystems

```bash
# Format a partition (WARNING: destroys all data!)
sudo mkfs.ext4 /dev/sdb1             # Create ext4 filesystem
sudo mkfs.xfs /dev/sdb1              # Create XFS filesystem
sudo mkfs.vfat /dev/sdb1             # Create FAT32 (USB drives)

# Check filesystem
sudo fsck /dev/sdb1                  # Check and repair (unmounted)
sudo e2fsck -f /dev/sdb1             # ext2/3/4 check (force)

# Swap space
sudo mkswap /dev/sdb2                # Format as swap
sudo swapon /dev/sdb2                # Enable swap
sudo swapoff /dev/sdb2               # Disable swap
swapon --show                        # Show swap usage
free -h                              # Memory including swap
```

---

## 15. Shell Scripting

### 15.1 Script Basics

```bash
#!/bin/bash
# This is a comment
# First line (shebang) tells OS which interpreter to use

# Other shebangs:
#!/bin/sh          # POSIX shell
#!/usr/bin/env python3  # Use python3 from PATH
#!/usr/bin/env node     # Node.js

# Make script executable
chmod +x script.sh

# Run a script
./script.sh                 # Run from current directory
bash script.sh              # Run with bash explicitly
source script.sh            # Run in current shell (. script.sh)
```

### 15.2 Variables

```bash
#!/bin/bash

# Variable assignment (NO spaces around =)
name="Alice"
age=30
is_admin=true
GREETING="Hello"

# Access variables with $
echo $name
echo "Hello, $name! You are $age years old."
echo "Hello, ${name}!"                # Curly braces (good practice)

# Readonly variables
readonly MAX_RETRIES=3

# Unset variable
unset name

# Special variables
echo $0             # Script name
echo $1             # First argument
echo $2             # Second argument
echo $@             # All arguments as separate words
echo $*             # All arguments as single string
echo $#             # Number of arguments
echo $?             # Exit code of last command (0=success)
echo $$             # PID of current script
echo $!             # PID of last background command

# Variable substitution
echo ${name:-"default"}         # Use "default" if name is empty
echo ${name:="default"}         # Set name to "default" if empty
echo ${name:+"has value"}       # Output "has value" if name is set
echo ${#name}                   # Length of name
echo ${name^^}                  # Uppercase
echo ${name,,}                  # Lowercase
echo ${name:0:3}                # Substring (first 3 chars)
echo ${name/old/new}            # Replace first match
echo ${name//old/new}           # Replace all matches
```

### 15.3 Input & Output

```bash
#!/bin/bash

# Read user input
read -p "Enter your name: " name
echo "Hello, $name!"

read -p "Enter your name: " -t 10 name   # Timeout after 10 seconds
read -s -p "Enter password: " password   # Silent (no echo)
echo ""                                  # New line after silent input

# Read into array
read -a fruits <<< "apple banana cherry"
echo ${fruits[0]}         # apple
echo ${fruits[@]}         # all

# Output
echo "Simple output"
echo -e "Line 1\nLine 2"  # Enable escape sequences
echo -n "No newline"       # Suppress newline
printf "Name: %-10s Age: %d\n" "Alice" 30  # Formatted output

# Colors in output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'           # No Color (reset)

echo -e "${RED}Error: Something went wrong${NC}"
echo -e "${GREEN}Success: Operation completed${NC}"
echo -e "${YELLOW}Warning: Check configuration${NC}"
```

### 15.4 Conditionals

```bash
#!/bin/bash

# if / elif / else
if [ "$name" = "Alice" ]; then
    echo "Hello, Alice!"
elif [ "$name" = "Bob" ]; then
    echo "Hello, Bob!"
else
    echo "Hello, stranger!"
fi

# Test operators for strings
[ "$str" = "value" ]      # String equals
[ "$str" != "value" ]     # String not equals
[ -z "$str" ]             # String is empty
[ -n "$str" ]             # String is not empty
[[ "$str" == *pattern* ]] # Pattern matching (bash only)
[[ "$str" =~ regex ]]     # Regex matching (bash only)

# Test operators for numbers
[ $a -eq $b ]             # Equal
[ $a -ne $b ]             # Not equal
[ $a -lt $b ]             # Less than
[ $a -le $b ]             # Less than or equal
[ $a -gt $b ]             # Greater than
[ $a -ge $b ]             # Greater than or equal

# Arithmetic comparison (alternative)
(( $a > $b ))
(( $a == $b ))

# Test operators for files
[ -f "$file" ]            # Exists and is a regular file
[ -d "$dir" ]             # Exists and is a directory
[ -e "$path" ]            # Exists (any type)
[ -r "$file" ]            # Readable
[ -w "$file" ]            # Writable
[ -x "$file" ]            # Executable
[ -s "$file" ]            # Exists and is not empty
[ -L "$link" ]            # Is a symbolic link
[ "$f1" -nt "$f2" ]       # f1 is newer than f2
[ "$f1" -ot "$f2" ]       # f1 is older than f2

# Logical operators
[ condition1 ] && [ condition2 ]    # AND
[ condition1 ] || [ condition2 ]    # OR
[[ condition1 && condition2 ]]      # AND (bash)
[[ condition1 || condition2 ]]      # OR (bash)
[ ! condition ]                     # NOT

# case statement
case "$day" in
    Mon|Tue|Wed|Thu|Fri)
        echo "Weekday"
        ;;
    Sat|Sun)
        echo "Weekend"
        ;;
    *)
        echo "Unknown"
        ;;
esac
```

### 15.5 Loops

```bash
#!/bin/bash

# for loop — iterate over list
for name in Alice Bob Charlie; do
    echo "Hello, $name!"
done

# for loop — C-style
for (( i=1; i<=10; i++ )); do
    echo "Iteration: $i"
done

# for loop — range
for i in {1..10}; do
    echo $i
done

for i in {0..100..10}; do    # 0 to 100 in steps of 10
    echo $i
done

# for loop — iterate over files
for file in *.txt; do
    echo "Processing: $file"
    wc -l "$file"
done

# for loop — iterate over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# while loop
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# while read loop (read file line by line)
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts

# Process command output line by line
ps aux | while read -r line; do
    echo "$line"
done

# until loop (run UNTIL condition is true)
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Loop control
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue        # Skip iteration 5
    fi
    if [ $i -eq 8 ]; then
        break           # Stop at 8
    fi
    echo $i
done
```

### 15.6 Functions

```bash
#!/bin/bash

# Define function
greet() {
    local name=$1                    # local = only exists in function
    echo "Hello, $name!"
}

# Call function
greet "Alice"
greet "Bob"

# Function with return value
add() {
    local result=$(( $1 + $2 ))
    echo $result                     # Use echo to "return" values
}

sum=$(add 5 3)
echo "5 + 3 = $sum"

# Function with exit code
file_exists() {
    if [ -f "$1" ]; then
        return 0    # 0 = success = true
    else
        return 1    # non-zero = failure = false
    fi
}

if file_exists "/etc/passwd"; then
    echo "File exists"
fi

# Error handling in functions
check_command() {
    if ! command -v "$1" &> /dev/null; then
        echo "Error: $1 is not installed"
        return 1
    fi
    return 0
}
```

### 15.7 Error Handling

```bash
#!/bin/bash

# Exit on first error
set -e

# Exit on unset variable
set -u

# Fail on pipe errors
set -o pipefail

# Enable all three (common practice)
set -euo pipefail

# Check exit code manually
if ! cp source.txt dest.txt; then
    echo "Copy failed!"
    exit 1
fi

# Error function
error() {
    echo "[ERROR] $1" >&2
    exit 1
}

info() {
    echo "[INFO] $1"
}

# Trap for cleanup on exit
cleanup() {
    rm -f /tmp/myapp_$$*.tmp
    echo "Cleanup done"
}
trap cleanup EXIT               # Run cleanup when script exits
trap cleanup INT TERM           # Also on Ctrl+C or kill

# Check required tools
for cmd in git docker kubectl; do
    command -v $cmd >/dev/null 2>&1 || error "$cmd is required but not installed"
done
```

### 15.8 Complete Script Example

```bash
#!/bin/bash
# backup.sh — Automated backup script
# Usage: ./backup.sh <source_dir> <backup_dir>

set -euo pipefail

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Logging
log() { echo -e "[$(date '+%Y-%m-%d %H:%M:%S')] $1"; }
info() { log "${GREEN}[INFO]${NC} $1"; }
warn() { log "${YELLOW}[WARN]${NC} $1"; }
error() { log "${RED}[ERROR]${NC} $1" >&2; exit 1; }

# Validate arguments
[ $# -ne 2 ] && error "Usage: $0 <source_dir> <backup_dir>"

SOURCE_DIR="$1"
BACKUP_DIR="$2"
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
BACKUP_NAME="backup_${TIMESTAMP}.tar.gz"

# Validation
[ ! -d "$SOURCE_DIR" ] && error "Source directory does not exist: $SOURCE_DIR"
mkdir -p "$BACKUP_DIR" || error "Cannot create backup directory: $BACKUP_DIR"

# Check available space
REQUIRED=$(du -sb "$SOURCE_DIR" | awk '{print $1}')
AVAILABLE=$(df -B1 "$BACKUP_DIR" | awk 'NR==2 {print $4}')

if [ "$REQUIRED" -gt "$AVAILABLE" ]; then
    error "Insufficient space. Required: $REQUIRED bytes, Available: $AVAILABLE bytes"
fi

# Perform backup
info "Starting backup of $SOURCE_DIR"
info "Destination: $BACKUP_DIR/$BACKUP_NAME"

if tar -czf "$BACKUP_DIR/$BACKUP_NAME" -C "$(dirname "$SOURCE_DIR")" \
        "$(basename "$SOURCE_DIR")" 2>/dev/null; then
    BACKUP_SIZE=$(du -sh "$BACKUP_DIR/$BACKUP_NAME" | awk '{print $1}')
    info "Backup completed: $BACKUP_NAME ($BACKUP_SIZE)"
else
    error "Backup failed!"
fi

# Keep only last 7 backups
info "Cleaning old backups (keeping last 7)..."
ls -t "$BACKUP_DIR"/backup_*.tar.gz | tail -n +8 | xargs -r rm -v

info "Done!"
```

---

## 16. Environment Variables & Configuration

### 16.1 Environment Variables

```bash
# View variables
env                             # All environment variables
printenv                        # Same as env
printenv PATH                   # Specific variable
printenv HOME USER SHELL        # Multiple
echo $PATH                      # Access in shell
echo $HOME                      # Home directory
echo $USER                      # Current user
echo $SHELL                     # Current shell
echo $LANG                      # System language
echo $TERM                      # Terminal type
echo $EDITOR                    # Default editor
echo $HISTFILE                  # History file location

# Set variable (current session only)
export MY_VAR="hello"
export PATH="$PATH:/usr/local/bin"    # Add to PATH

# Common important variables
PATH            → Directories searched for commands
HOME            → Current user's home directory
USER / LOGNAME  → Current username
SHELL           → Path to current shell
TERM            → Terminal type
LANG / LC_ALL   → Locale/language settings
EDITOR          → Default text editor
VISUAL          → Visual editor (used by some programs)
PAGER           → Default pager (less, more)
PS1             → Shell prompt string
HISTSIZE        → Number of commands in history
HISTFILE        → Location of history file
```

### 16.2 Making Variables Permanent

```bash
# ~/.bashrc — runs on every new INTERACTIVE bash shell
# Add to end of file:
echo 'export MY_VAR="value"' >> ~/.bashrc
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc

# Reload without logging out
source ~/.bashrc
. ~/.bashrc                     # Same thing

# ~/.bash_profile or ~/.profile — runs on LOGIN shells
# Good for environment setup

# /etc/environment — system-wide (all users, all shells)
sudo echo 'MY_GLOBAL_VAR="value"' >> /etc/environment

# /etc/profile.d/ — drop-in scripts for all users
sudo nano /etc/profile.d/myapp.sh
# Content:
#!/bin/bash
export MYAPP_HOME=/opt/myapp
export PATH="$PATH:$MYAPP_HOME/bin"
```

### 16.3 Customizing Your Shell

```bash
# ~/.bashrc customizations

# Better history
HISTSIZE=10000
HISTFILESIZE=20000
HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S  "
HISTCONTROL=ignoredups:erasedups
shopt -s histappend                     # Append instead of overwrite

# Useful aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias grep='grep --color=auto'
alias df='df -h'
alias du='du -h'
alias mkdir='mkdir -pv'
alias cp='cp -iv'
alias mv='mv -iv'
alias rm='rm -I'

# Custom prompt (PS1)
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
# Colors: \u=user, \h=host, \w=working dir

# Functions in .bashrc
mkcd() { mkdir -p "$1" && cd "$1"; }    # mkdir and cd in one
extract() {                             # Universal extract
    case $1 in
        *.tar.gz)  tar -xzf "$1" ;;
        *.tar.bz2) tar -xjf "$1" ;;
        *.zip)     unzip "$1" ;;
        *.gz)      gunzip "$1" ;;
        *.rar)     unrar x "$1" ;;
        *)         echo "Unknown archive type: $1" ;;
    esac
}
```

---

## 17. Archiving & Compression

### 17.1 tar — Tape Archive

```bash
# tar — bundles files, does NOT compress by default

# Create archive
tar -cf archive.tar file1.txt file2.txt    # Create archive
tar -cf archive.tar directory/              # Archive directory
tar -czf archive.tar.gz directory/         # Create + gzip compress
tar -cjf archive.tar.bz2 directory/        # Create + bzip2 compress
tar -cJf archive.tar.xz directory/         # Create + xz compress

# Extract archive
tar -xf archive.tar                         # Extract
tar -xzf archive.tar.gz                     # Extract gzip
tar -xjf archive.tar.bz2                    # Extract bzip2
tar -xJf archive.tar.xz                     # Extract xz
tar -xf archive.tar -C /tmp/               # Extract to specific directory

# View contents without extracting
tar -tf archive.tar
tar -tzf archive.tar.gz

# Add files to existing archive
tar -rf archive.tar newfile.txt

# Useful options
tar -czf backup.tar.gz dir/ --exclude="*.log"   # Exclude files
tar -czf backup.tar.gz dir/ --exclude-vcs        # Exclude .git etc.
tar -czfv archive.tar.gz dir/                    # Verbose (show files)
tar -czf - dir/ | ssh user@remote "cat > remote.tar.gz"  # Pipe to remote

# Flags summary:
# c = create, x = extract, t = list
# f = filename, v = verbose
# z = gzip, j = bzip2, J = xz
```

### 17.2 Compression Tools

```bash
# gzip — compress/decompress (.gz)
gzip file.txt               # Compresses file.txt → file.txt.gz (original removed)
gzip -k file.txt            # Keep original
gzip -9 file.txt            # Maximum compression
gunzip file.txt.gz          # Decompress
gzip -d file.txt.gz         # Same as gunzip
zcat file.txt.gz            # View compressed file without extracting

# bzip2 — better compression than gzip (.bz2)
bzip2 file.txt              # Compress → file.txt.bz2
bunzip2 file.txt.bz2        # Decompress
bzcat file.txt.bz2          # View without extracting

# xz — best compression (.xz)
xz file.txt                 # Compress → file.txt.xz
unxz file.txt.xz            # Decompress
xzcat file.txt.xz           # View without extracting

# zip/unzip — cross-platform (.zip)
zip archive.zip file1.txt file2.txt
zip -r archive.zip directory/         # Include directory
unzip archive.zip                     # Extract
unzip archive.zip -d /tmp/target/     # Extract to specific dir
unzip -l archive.zip                  # List contents

# Compression speed vs ratio comparison:
# gzip:  Fastest,  medium compression (~3:1)
# bzip2: Medium,   better compression (~4:1)
# xz:    Slowest,  best compression (~5:1)
# zip:   Fast,     medium, but Windows-compatible
```

---

## 18. System Logs & Monitoring

### 18.1 Log Files

```bash
# Key log locations
/var/log/syslog           # General system logs (Debian/Ubuntu)
/var/log/messages         # General system logs (RHEL/CentOS)
/var/log/auth.log         # Authentication logs (Debian/Ubuntu)
/var/log/secure           # Authentication logs (RHEL/CentOS)
/var/log/kern.log         # Kernel logs
/var/log/dmesg            # Boot messages / hardware
/var/log/nginx/           # Nginx web server logs
/var/log/apache2/         # Apache web server logs
/var/log/mysql/           # MySQL logs
/var/log/cron             # Cron job logs
/var/log/mail.log         # Mail server logs
/var/log/dpkg.log         # Package install/remove logs (Debian)
/var/log/journal/         # systemd journal (binary format)

# Viewing logs
tail -f /var/log/syslog                     # Follow in real-time
tail -100 /var/log/auth.log                 # Last 100 lines
grep "Failed password" /var/log/auth.log    # Find failed SSH attempts
grep "error" /var/log/nginx/error.log       # Nginx errors
cat /var/log/syslog | grep "$(date '+%b %d')" # Today's logs

# journalctl — systemd journal (modern systems)
journalctl                                  # All logs
journalctl -f                               # Follow (like tail -f)
journalctl -u nginx                         # Logs for nginx service
journalctl -u nginx -f                      # Follow nginx logs
journalctl --since "2025-01-01"             # Since date
journalctl --since "1 hour ago"             # Recent
journalctl -p err                           # Only errors
journalctl -p err -b                        # Errors since last boot
journalctl -b                               # Current boot
journalctl -b -1                            # Previous boot
journalctl -n 50                            # Last 50 lines
journalctl --disk-usage                     # How much disk logs use
journalctl --vacuum-size=500M               # Keep only 500MB of logs
```

### 18.2 System Monitoring

```bash
# Real-time monitoring
top                         # Process viewer (built-in)
htop                        # Better top (sudo apt install htop)
btop                        # Beautiful top (sudo apt install btop)
iotop                       # Disk I/O by process (sudo apt install iotop)
nethogs                     # Network by process (sudo apt install nethogs)
iftop                       # Network interface traffic (sudo apt install iftop)
nload                       # Network bandwidth (sudo apt install nload)

# System statistics
vmstat 1                    # VM stats every 1 second
vmstat -s                   # One-shot summary
iostat -x 1                 # Extended I/O stats
sar -u 1 5                  # CPU stats (sysstat package)
sar -r                      # Memory stats
sar -n DEV 1 5              # Network stats

# Snapshot commands
cat /proc/cpuinfo            # CPU information
cat /proc/meminfo            # Memory information
cat /proc/diskstats          # Disk statistics
cat /proc/net/dev            # Network stats
lsof                         # List open files (ALL processes)
lsof -u alice                # Open files by user
lsof -p 1234                 # Open files by PID
lsof -i :80                  # Processes using port 80
lsof -i TCP                  # All TCP connections
strace -p 1234               # Trace system calls of running process
```

---

## 19. SSH & Remote Access

### 19.1 SSH Basics

```bash
# Connect to remote server
ssh user@hostname
ssh alice@192.168.1.100
ssh alice@server.example.com
ssh -p 2222 alice@server          # Custom port
ssh -i ~/.ssh/id_rsa alice@server # Specific private key
ssh -v alice@server               # Verbose (debug connection issues)
ssh -X alice@server               # X11 forwarding (GUI apps)

# First connection — verify fingerprint
# "The authenticity of host 'server' can't be established..."
# Compare fingerprint with server admin or known_hosts

# Execute single command remotely
ssh alice@server "ls -la /var/log"
ssh alice@server "sudo systemctl status nginx"
ssh alice@server "cat /etc/hostname && uptime"

# Copy files
scp file.txt alice@server:/remote/path/
scp alice@server:/remote/file.txt .
scp -r dir/ alice@server:/remote/

# SSH tunneling
ssh -L 8080:localhost:80 alice@server    # Local port forwarding
# Access server's port 80 via localhost:8080

ssh -R 9090:localhost:3000 alice@server  # Remote port forwarding
# Share your local port 3000 with the server

ssh -D 1080 alice@server                 # Dynamic (SOCKS proxy)
```

### 19.2 SSH Key Management

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "alice@example.com"
ssh-keygen -t ed25519 -C "alice@example.com"    # More modern, recommended

# Key files created:
# ~/.ssh/id_ed25519      → Private key (NEVER share this!)
# ~/.ssh/id_ed25519.pub  → Public key (safe to share)

# Copy public key to remote server
ssh-copy-id alice@server
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@server    # Specific key

# Manual alternative
cat ~/.ssh/id_ed25519.pub | ssh alice@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Permissions (MUST be correct)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/id_ed25519           # Private key MUST be 600

# SSH config file (~/.ssh/config)
cat > ~/.ssh/config << 'EOF'
Host myserver
    HostName 192.168.1.100
    User alice
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60

Host jump-host
    HostName jump.example.com
    User alice

Host production
    HostName prod.example.com
    User deploy
    ProxyJump jump-host
    IdentityFile ~/.ssh/prod_key
EOF

chmod 600 ~/.ssh/config

# Now connect simply:
ssh myserver
ssh production
```

### 19.3 SSH Server Configuration

```bash
# Edit SSH server config (requires root)
sudo nano /etc/ssh/sshd_config

# Important settings to review/change:
Port 22                          # Change for security
PermitRootLogin no               # Never allow direct root login
PasswordAuthentication no        # Force key-based auth
PubkeyAuthentication yes         # Allow key auth
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3                   # Limit login attempts
LoginGraceTime 20                # Timeout for login
AllowUsers alice bob             # Only allow specific users

# Restart SSH after changes
sudo systemctl restart sshd      # or: sudo service ssh restart

# Test config before restarting (saves you from locking yourself out)
sudo sshd -t
```

---

## 20. Cron Jobs & Scheduling

### 20.1 Cron Syntax

```
# Cron expression format:
# ┌─── minute (0-59)
# │ ┌─── hour (0-23)
# │ │ ┌─── day of month (1-31)
# │ │ │ ┌─── month (1-12 or Jan-Dec)
# │ │ │ │ ┌─── day of week (0-7 or Sun-Sat; 0 and 7 = Sunday)
# │ │ │ │ │
# * * * * * command_to_run

# Special values:
# *    = every
# ,    = list (e.g., 1,3,5)
# -    = range (e.g., 1-5)
# /    = step (e.g., */5 = every 5)

# Examples:
* * * * *           # Every minute
0 * * * *           # Every hour (at :00)
0 6 * * *           # Every day at 6:00 AM
0 6 * * 1           # Every Monday at 6:00 AM
0 6 1 * *           # First of every month at 6:00 AM
0 6 1 1 *           # January 1st at 6:00 AM
*/5 * * * *         # Every 5 minutes
0 0,12 * * *        # Midnight and noon every day
0 9-17 * * 1-5      # 9 AM to 5 PM, Monday–Friday
30 23 * * 5         # 11:30 PM every Friday
@reboot             # Once on startup
@daily              # Same as 0 0 * * *
@hourly             # Same as 0 * * * *
@weekly             # Same as 0 0 * * 0
@monthly            # Same as 0 0 1 * *
@yearly             # Same as 0 0 1 1 *
```

### 20.2 Managing Crontab

```bash
# User crontab
crontab -e                          # Edit your crontab
crontab -l                          # List your cron jobs
crontab -r                          # Remove ALL your cron jobs (careful!)
crontab -u alice -l                 # List alice's crontab (root only)

# System-wide cron
cat /etc/crontab                    # System crontab (has user column)
ls /etc/cron.d/                     # Additional system cron files
ls /etc/cron.daily/                 # Scripts run daily
ls /etc/cron.hourly/                # Scripts run hourly
ls /etc/cron.weekly/                # Scripts run weekly
ls /etc/cron.monthly/               # Scripts run monthly

# Example crontab entries:
# Backup every day at 2 AM
0 2 * * * /home/alice/backup.sh >> /var/log/backup.log 2>&1

# Disk usage report every Monday at 8 AM
0 8 * * 1 df -h | mail -s "Disk Report" admin@example.com

# Clear temp files every Sunday at midnight
0 0 * * 0 find /tmp -mtime +7 -delete

# Run script every 5 minutes
*/5 * * * * /scripts/health_check.sh

# Best practices for cron:
# 1. Use full paths: /usr/bin/python3 not python3
# 2. Redirect output: >> /var/log/myjob.log 2>&1
# 3. Set environment variables at top of crontab:
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@example.com
```

### 20.3 systemd Timers (Modern Alternative)

```bash
# Create a timer unit (modern systemd approach)
# 1. Create service file
sudo nano /etc/systemd/system/backup.service
# [Unit]
# Description=Daily Backup
# [Service]
# Type=oneshot
# ExecStart=/home/alice/backup.sh

# 2. Create timer file
sudo nano /etc/systemd/system/backup.timer
# [Unit]
# Description=Daily Backup Timer
# [Timer]
# OnCalendar=daily
# Persistent=true
# [Install]
# WantedBy=timers.target

# 3. Enable and start timer
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
systemctl list-timers                   # List all timers
systemctl status backup.timer
```

---

## 21. Practical Projects

### Project 1: System Health Dashboard Script

```bash
#!/bin/bash
# system_health.sh — Comprehensive system health report

set -euo pipefail

RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'
BLUE='\033[0;34m'; BOLD='\033[1m'; NC='\033[0m'

divider() { echo -e "${BLUE}$(printf '=%.0s' {1..60})${NC}"; }
section() { echo -e "\n${BOLD}${GREEN}▶ $1${NC}"; divider; }

echo -e "${BOLD}${BLUE}"
echo "  ╔══════════════════════════════════════╗"
echo "  ║     SYSTEM HEALTH DASHBOARD          ║"
echo "  ║     $(date '+%Y-%m-%d %H:%M:%S')         ║"
echo "  ╚══════════════════════════════════════╝"
echo -e "${NC}"

section "SYSTEM INFORMATION"
echo -e "Hostname:    ${YELLOW}$(hostname)${NC}"
echo -e "OS:          $(cat /etc/os-release | grep PRETTY_NAME | cut -d= -f2 | tr -d '"')"
echo -e "Kernel:      $(uname -r)"
echo -e "Uptime:      $(uptime -p)"
echo -e "Boot Time:   $(who -b | awk '{print $3, $4}')"

section "CPU USAGE"
cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
cpu_cores=$(nproc)
load=$(uptime | awk -F'load average:' '{print $2}')
echo -e "Cores:       ${cpu_cores}"
echo -e "CPU Usage:   ${cpu_usage}%"
echo -e "Load Avg:    ${load}"

section "MEMORY USAGE"
free -h | awk 'NR==1 {printf "%-12s %8s %8s %8s\n", $1, $2, $3, $4}
               NR==2 {printf "%-12s %8s %8s %8s\n", $1, $2, $3, $4}
               NR==3 {printf "%-12s %8s %8s %8s\n", $1, $2, $3, $4}'

section "DISK USAGE"
df -h | grep -E "^/dev|^Filesystem" | awk '{printf "%-20s %8s %8s %8s %8s\n", $1, $2, $3, $4, $5}'
echo ""
# Alert on high disk usage
df -h | grep -E "^/dev" | awk '{gsub(/%/,"",$5); if ($5+0 > 80) print}' | while read line; do
    echo -e "${RED}⚠ HIGH DISK USAGE: $line${NC}"
done

section "NETWORK INTERFACES"
ip -4 addr show | grep -E "^[0-9]|inet " | awk '
    /^[0-9]/ {split($2,a,":"); iface=a[2]}
    /inet / {split($2,b,"/"); print iface": "b[1]}'

section "TOP 5 CPU PROCESSES"
ps aux --sort=-%cpu | head -6 | awk '{printf "%-30s %6s %6s\n", $11, $3, $4}' | head -6

section "TOP 5 MEMORY PROCESSES"
ps aux --sort=-%mem | head -6 | awk '{printf "%-30s %6s %6s\n", $11, $4, $3}' | head -6

section "LISTENING PORTS"
ss -tuln | grep LISTEN | awk '{print $5}' | sort -u

section "RECENT AUTH FAILURES"
auth_log="/var/log/auth.log"
[ -f "$auth_log" ] && grep -c "Failed password" "$auth_log" 2>/dev/null && \
    echo "recent failures:" && grep "Failed password" "$auth_log" | tail -5 || \
    echo "Auth log not accessible or no failures found"

echo -e "\n${GREEN}Health check complete: $(date)${NC}\n"
```

---

### Project 2: User Onboarding Automation Script

```bash
#!/bin/bash
# onboard_user.sh — Create and set up a new user account

set -euo pipefail

[ "$EUID" -ne 0 ] && { echo "Run as root or with sudo"; exit 1; }
[ $# -lt 2 ] && { echo "Usage: $0 <username> <fullname> [groups]"; exit 1; }

USERNAME="$1"
FULLNAME="$2"
EXTRA_GROUPS="${3:-}"
DEFAULT_GROUPS="sudo,docker"

RED='\033[0;31m'; GREEN='\033[0;32m'; NC='\033[0m'
log() { echo -e "${GREEN}[✓]${NC} $1"; }
err() { echo -e "${RED}[✗]${NC} $1" >&2; exit 1; }

# Check user doesn't already exist
id "$USERNAME" &>/dev/null && err "User $USERNAME already exists"

# Create user
log "Creating user: $USERNAME ($FULLNAME)"
useradd -m -c "$FULLNAME" -s /bin/bash "$USERNAME"

# Set groups
ALL_GROUPS="$DEFAULT_GROUPS"
[ -n "$EXTRA_GROUPS" ] && ALL_GROUPS="$ALL_GROUPS,$EXTRA_GROUPS"
log "Adding to groups: $ALL_GROUPS"
usermod -aG "$ALL_GROUPS" "$USERNAME"

# Generate temporary password
TEMP_PASS=$(openssl rand -base64 12)
echo "$USERNAME:$TEMP_PASS" | chpasswd
passwd --expire "$USERNAME"

# Set up SSH directory
log "Setting up SSH directory"
mkdir -p "/home/$USERNAME/.ssh"
touch "/home/$USERNAME/.ssh/authorized_keys"
chmod 700 "/home/$USERNAME/.ssh"
chmod 600 "/home/$USERNAME/.ssh/authorized_keys"
chown -R "$USERNAME:$USERNAME" "/home/$USERNAME/.ssh"

# Copy bash config skeleton
cp /etc/skel/.bashrc "/home/$USERNAME/.bashrc"
chown "$USERNAME:$USERNAME" "/home/$USERNAME/.bashrc"

# Create personal directories
log "Creating default directories"
for dir in projects scripts docs; do
    mkdir -p "/home/$USERNAME/$dir"
    chown "$USERNAME:$USERNAME" "/home/$USERNAME/$dir"
done

echo ""
echo "═══════════════════════════════════════════"
echo "  User Created Successfully"
echo "═══════════════════════════════════════════"
echo "  Username:  $USERNAME"
echo "  Full Name: $FULLNAME"
echo "  Home:      /home/$USERNAME"
echo "  Groups:    $(id -nG $USERNAME)"
echo "  Temp Pass: $TEMP_PASS"
echo "  (User must change on first login)"
echo "═══════════════════════════════════════════"
```

---

### Project 3: Log Analyzer

```bash
#!/bin/bash
# log_analyzer.sh — Analyze web server access logs

LOG_FILE="${1:-/var/log/nginx/access.log}"
[ ! -f "$LOG_FILE" ] && { echo "Log file not found: $LOG_FILE"; exit 1; }

echo "╔════════════════════════════════════════════╗"
echo "║           LOG ANALYSIS REPORT              ║"
echo "║  File: $LOG_FILE"
echo "╚════════════════════════════════════════════╝"
echo ""

echo "📊 SUMMARY"
echo "Total requests: $(wc -l < "$LOG_FILE")"
echo "Unique IPs:     $(awk '{print $1}' "$LOG_FILE" | sort -u | wc -l)"
echo ""

echo "🔥 TOP 10 IP ADDRESSES"
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10 | \
    awk '{printf "  %6s requests  %s\n", $1, $2}'
echo ""

echo "📄 TOP 10 REQUESTED PAGES"
awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10 | \
    awk '{printf "  %6s times  %s\n", $1, $2}'
echo ""

echo "⚠️  HTTP STATUS CODE BREAKDOWN"
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn | \
    awk '{printf "  %6s  HTTP %s\n", $1, $2}'
echo ""

echo "❌ TOP 404 PAGES"
awk '$9 == 404 {print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10 | \
    awk '{printf "  %6s times  %s\n", $1, $2}'
echo ""

echo "🌐 TOP BROWSERS/USER AGENTS"
awk -F'"' '{print $6}' "$LOG_FILE" | cut -d' ' -f1 | sort | uniq -c | sort -rn | head -5
```

---

### Project 4: Development Environment Setup Script

```bash
#!/bin/bash
# dev_setup.sh — Set up a complete developer environment

set -euo pipefail

echo "🚀 Setting up Developer Environment..."

# Update system
sudo apt update && sudo apt upgrade -y

# Core tools
sudo apt install -y \
    curl wget git vim nano htop tree \
    build-essential make cmake \
    net-tools netcat-openbsd nmap \
    unzip zip tar jq \
    tmux screen

# Languages
sudo apt install -y \
    python3 python3-pip python3-venv \
    nodejs npm \
    golang

# Docker
if ! command -v docker &>/dev/null; then
    curl -fsSL https://get.docker.com | sh
    sudo usermod -aG docker "$USER"
    echo "✅ Docker installed"
fi

# Git configuration
read -p "Git username: " git_name
read -p "Git email: " git_email
git config --global user.name "$git_name"
git config --global user.email "$git_email"
git config --global init.defaultBranch main
git config --global core.editor vim
echo "✅ Git configured"

# SSH key
if [ ! -f "$HOME/.ssh/id_ed25519" ]; then
    ssh-keygen -t ed25519 -C "$git_email" -f "$HOME/.ssh/id_ed25519" -N ""
    echo "✅ SSH key generated"
    echo "📋 Add this key to GitHub/GitLab:"
    cat "$HOME/.ssh/id_ed25519.pub"
fi

# Useful aliases in .bashrc
cat >> "$HOME/.bashrc" << 'BASHRC'

# DevOps aliases
alias ll='ls -alF --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
alias gs='git status'
alias gp='git pull'
alias gph='git push'
alias gcm='git commit -m'
alias dps='docker ps'
alias dpa='docker ps -a'

# Functions
mkcd() { mkdir -p "$1" && cd "$1"; }
BASHRC

echo ""
echo "✅ Development environment setup complete!"
echo "   Run: source ~/.bashrc to apply aliases"
echo "   Logout and back in for Docker group to take effect"
```

---

## 22. Complete Command Reference

### Navigation & Files

| Command | Description | Example |
|---------|-------------|---------|
| `pwd` | Print working directory | `pwd` |
| `ls -la` | List all with details | `ls -la /etc` |
| `cd` | Change directory | `cd /var/log` |
| `find` | Find files | `find / -name "*.conf" 2>/dev/null` |
| `locate` | Fast file search | `locate nginx.conf` |
| `touch` | Create empty file | `touch file.txt` |
| `mkdir -p` | Create nested dirs | `mkdir -p a/b/c` |
| `cp -r` | Copy directory | `cp -r src/ dst/` |
| `mv` | Move/rename | `mv old.txt new.txt` |
| `rm -rf` | Remove recursively | `rm -rf dir/ (CAREFUL)` |
| `ln -s` | Symbolic link | `ln -s /usr/bin/python3 /usr/bin/py` |

### Text Operations

| Command | Description | Example |
|---------|-------------|---------|
| `cat` | Print file | `cat /etc/hosts` |
| `less` | Pager | `less /var/log/syslog` |
| `head -n` | First N lines | `head -20 file.txt` |
| `tail -f` | Follow file | `tail -f /var/log/syslog` |
| `grep -r` | Recursive search | `grep -r "TODO" ./src` |
| `sed 's/a/b/g'` | Replace text | `sed -i 's/foo/bar/g' file` |
| `awk '{print $1}'` | Column extract | `awk -F: '{print $1}' /etc/passwd` |
| `sort -n` | Numeric sort | `sort -rn numbers.txt` |
| `uniq -c` | Count duplicates | `sort file \| uniq -c` |
| `wc -l` | Count lines | `wc -l /etc/passwd` |
| `cut -d: -f1` | Cut by delimiter | `cut -d: -f1 /etc/passwd` |
| `tr 'a-z' 'A-Z'` | Translate chars | `echo hello \| tr 'a-z' 'A-Z'` |
| `diff` | Compare files | `diff file1 file2` |
| `tee` | Write to file and stdout | `command \| tee output.txt` |

### System & Process

| Command | Description | Example |
|---------|-------------|---------|
| `ps aux` | All processes | `ps aux \| grep nginx` |
| `top` / `htop` | Process monitor | `htop` |
| `kill -9` | Force kill process | `kill -9 1234` |
| `pkill` | Kill by name | `pkill nginx` |
| `jobs` | Background jobs | `jobs` |
| `fg` / `bg` | Foreground/background | `fg %1` |
| `nohup` | Survive logout | `nohup ./script.sh &` |
| `systemctl` | Manage services | `sudo systemctl restart nginx` |
| `journalctl -f` | Follow system logs | `journalctl -u nginx -f` |
| `uname -a` | Kernel info | `uname -a` |
| `uptime` | System uptime | `uptime` |
| `dmesg` | Kernel messages | `dmesg -T \| tail -20` |

### Users & Permissions

| Command | Description | Example |
|---------|-------------|---------|
| `whoami` | Current user | `whoami` |
| `id` | User/group IDs | `id alice` |
| `sudo` | Run as root | `sudo apt update` |
| `su -` | Switch user | `su - alice` |
| `useradd -m` | Create user | `sudo useradd -m alice` |
| `passwd` | Change password | `sudo passwd alice` |
| `usermod -aG` | Add to group | `sudo usermod -aG docker alice` |
| `chmod 755` | Change permissions | `chmod +x script.sh` |
| `chown user:group` | Change owner | `chown -R alice:dev /project` |
| `groups` | List groups | `groups alice` |

### Networking

| Command | Description | Example |
|---------|-------------|---------|
| `ip addr` | IP addresses | `ip addr show` |
| `ping` | Test connectivity | `ping -c 4 google.com` |
| `ss -tuln` | Listening ports | `ss -tulnp` |
| `curl` | HTTP requests | `curl -L https://example.com` |
| `wget` | Download files | `wget https://example.com/file.zip` |
| `scp` | Secure copy | `scp file user@host:/path/` |
| `rsync` | Sync files | `rsync -avz src/ user@host:/dst/` |
| `ssh` | Remote shell | `ssh -i key.pem user@host` |
| `netstat -tuln` | Open ports | `netstat -tulnp` |
| `traceroute` | Trace route | `traceroute google.com` |
| `dig` | DNS lookup | `dig google.com MX` |
| `nmap` | Port scan | `nmap -sV localhost` |

### Disk & Storage

| Command | Description | Example |
|---------|-------------|---------|
| `df -h` | Disk free | `df -h` |
| `du -sh` | Directory size | `du -sh /var/log` |
| `lsblk` | Block devices | `lsblk -f` |
| `mount` | Mount filesystem | `sudo mount /dev/sdb1 /mnt` |
| `fdisk -l` | Partition list | `sudo fdisk -l` |
| `mkfs.ext4` | Format partition | `sudo mkfs.ext4 /dev/sdb1` |

### Archives & Compression

| Command | Description | Example |
|---------|-------------|---------|
| `tar -czf` | Create .tar.gz | `tar -czf archive.tar.gz dir/` |
| `tar -xzf` | Extract .tar.gz | `tar -xzf archive.tar.gz` |
| `tar -tf` | List archive | `tar -tf archive.tar.gz` |
| `gzip` | Compress file | `gzip -k file.txt` |
| `gunzip` | Decompress | `gunzip file.txt.gz` |
| `zip -r` | Create .zip | `zip -r archive.zip dir/` |
| `unzip` | Extract .zip | `unzip archive.zip -d /tmp/` |

### Useful One-Liners

```bash
# Find all files larger than 100MB
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Find and replace in multiple files
grep -rl "oldtext" . | xargs sed -i 's/oldtext/newtext/g'

# Show top 10 memory-consuming processes
ps aux --sort=-%mem | head -11

# Monitor log file with grep filter
tail -f /var/log/syslog | grep -i error

# Get external IP address
curl -s https://api.ipify.org

# Download and extract in one step
curl -sL https://example.com/file.tar.gz | tar -xz

# Find files modified in the last 24 hours
find . -mtime -1 -type f

# Count occurrences of each word in a file
cat file.txt | tr ' ' '\n' | sort | uniq -c | sort -rn | head -20

# Check if a port is open
nc -zv google.com 443 2>&1

# Create a quick HTTP server (Python)
python3 -m http.server 8080

# Watch a directory for file changes
watch -n 2 'ls -lat /var/log | head -10'

# Disk usage of each subdirectory, sorted
du -sh /var/log/* 2>/dev/null | sort -rh | head -10

# Kill all processes matching a name
ps aux | grep 'processname' | grep -v grep | awk '{print $2}' | xargs kill

# SSH key fingerprint
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# Bulk rename files
for f in *.txt; do mv "$f" "${f%.txt}.md"; done

# Check if a command exists
command -v docker &>/dev/null && echo "Docker installed" || echo "Not found"

# Monitor CPU temperature
watch -n 1 'cat /sys/class/thermal/thermal_zone*/temp | awk "{print \$1/1000 \" °C\"}"'

# Extract specific columns from CSV
awk -F, 'NR>1 {print $1, $3}' data.csv

# Run multiple commands in parallel
{ command1 & command2 & command3 & wait; }

# Find duplicate files by MD5
find . -type f -exec md5sum {} + | sort | awk 'BEGIN{last=""} $1==last{print} {last=$1}'
```

---

## 23. Glossary

| Term | Definition |
|------|-----------|
| **Bash** | Bourne Again Shell — the most common Linux shell |
| **Block device** | Storage device that reads/writes in blocks (disks) |
| **Boot loader** | Software that starts the OS (GRUB on Linux) |
| **CLI** | Command-Line Interface |
| **Daemon** | Background service process (sshd, httpd, cron) |
| **Distro** | A complete OS built around the Linux kernel |
| **EOF** | End of File marker |
| **Filesystem** | Method for organizing and storing files on storage |
| **FOSS** | Free and Open Source Software |
| **GRUB** | GRand Unified Bootloader — common Linux boot loader |
| **Hard link** | Another directory entry pointing to the same inode |
| **Inode** | Data structure storing file metadata (not the filename) |
| **Kernel** | Core of the OS managing hardware resources |
| **LTS** | Long-Term Support — OS version with extended maintenance |
| **MAN page** | Manual page for a command (`man ls`) |
| **Mount** | Attach a filesystem to the directory tree |
| **Package** | Bundled software with metadata for package managers |
| **PATH** | Environment variable listing directories for command lookup |
| **PID** | Process ID — unique identifier for a running process |
| **Pipe** | `\|` — connects stdout of one command to stdin of another |
| **POSIX** | Portable Operating System Interface — Unix standards |
| **Process** | A running program instance |
| **Prompt** | The text before your cursor in the shell |
| **Redirect** | `>`, `>>`, `<` — send output/input to/from files |
| **Root** | The superuser account; also the top `/` of filesystem |
| **Shell** | Program that interprets commands (bash, zsh, sh) |
| **shebang** | `#!/bin/bash` — first line of a script specifying interpreter |
| **Signal** | OS message sent to a process (SIGTERM, SIGKILL) |
| **Soft link** | Symbolic link — a file that points to another path |
| **SSH** | Secure Shell — encrypted remote access protocol |
| **stdin/stdout/stderr** | Standard input/output/error streams (0, 1, 2) |
| **Superuser** | Root user — has unlimited system privileges |
| **sudo** | Run a command with root privileges |
| **Symlink** | Symbolic link — shortcut to another file/directory |
| **Terminal** | Application providing access to the shell |
| **TTY** | Teletypewriter — refers to terminal sessions |
| **umask** | Default permission removal mask for new files |
| **UID / GID** | User ID / Group ID — numeric identifiers |
| **Virtual FS** | /proc, /sys — expose kernel info as files |
| **Wildcard** | `*`, `?`, `[]` — pattern matching in filenames |

---

## 24. Resources

### Official Documentation
- 📖 [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- 📖 [Bash Reference Manual](https://www.gnu.org/software/bash/manual/)
- 📖 [Linux man-pages Project](https://man7.org/linux/man-pages/)
- 📖 [The Linux Documentation Project (TLDP)](https://tldp.org/)

### Free Books & Courses
- 📚 [The Linux Command Line — William Shotts (Free Online)](https://linuxcommand.org/tlcl.php)
- 📚 [Linux Bible (Wiley)](https://www.wiley.com/en-us/Linux+Bible-p-9781119578888)
- 🎓 [Linux Foundation Free Courses](https://training.linuxfoundation.org/resources/free-courses/)

### Interactive Learning
- 💻 [Linux Journey](https://linuxjourney.com/) — Structured interactive lessons
- 💻 [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/) — Learn by hacking
- 💻 [Killercoda](https://killercoda.com/learn) — Browser-based Linux scenarios
- 💻 [Explain Shell](https://explainshell.com/) — Understand any command
- 💻 [Linux Survival](https://linuxsurvival.com/) — Interactive command trainer
- 💻 [Vim Adventures](https://vim-adventures.com/) — Learn Vim through a game

### YouTube Channels
- 🎥 [TechWorld with Nana](https://www.youtube.com/@TechWorldwithNana) — DevOps & Linux
- 🎥 [NetworkChuck](https://www.youtube.com/@NetworkChuck) — Beginner-friendly Linux
- 🎥 [Chris Titus Tech](https://www.youtube.com/@ChrisTitusTech) — Linux tips & tricks
- 🎥 [The Linux Foundation](https://www.youtube.com/@LinuxfoundationOrg)

### Certifications
- 🏆 [LPIC-1](https://www.lpi.org/our-certifications/lpic-1-overview) — Linux Professional Institute
- 🏆 [CompTIA Linux+](https://www.comptia.org/certifications/linux) — Vendor-neutral
- 🏆 [RHCSA](https://www.redhat.com/en/services/certification/rhcsa) — Red Hat Certified System Administrator

### Cheat Sheets
- 📋 [tldr-pages](https://tldr.sh/) — Simplified man pages (`sudo apt install tldr`)
- 📋 [cheat.sh](https://cheat.sh/) — Community cheat sheets (`curl cheat.sh/tar`)
- 📋 [Linux Command Library](https://linuxcommandlibrary.com/)

---

## ✅ Complete Mastery Checklist

### Foundations
- [ ] Explain what Linux is, its history, and why it dominates servers
- [ ] Name and compare 5+ Linux distributions
- [ ] Describe the Linux architecture (kernel, shell, filesystem)
- [ ] Set up a Linux environment (WSL2, VM, or cloud)

### Navigation & Files
- [ ] Navigate the filesystem without thinking about commands
- [ ] Explain the purpose of every top-level directory (`/etc`, `/var`, `/proc`, etc.)
- [ ] Create complex directory structures with brace expansion
- [ ] Find files using `find` with multiple criteria
- [ ] Search file contents with `grep` using regex

### Text & Editing
- [ ] Comfortably use `vim` to edit files
- [ ] Use `sed` and `awk` for text processing
- [ ] Chain commands with pipes to solve real problems
- [ ] Redirect stdin/stdout/stderr appropriately

### Permissions & Users
- [ ] Read any permission string instantly
- [ ] Use `chmod` in both symbolic and octal mode
- [ ] Manage users and groups from the command line
- [ ] Configure `sudo` safely

### Process & System
- [ ] Monitor and manage processes with `ps`, `top`, `htop`
- [ ] Send appropriate signals to processes
- [ ] Understand and manage foreground/background jobs
- [ ] Interpret system resource usage

### Networking
- [ ] Configure network interfaces
- [ ] Troubleshoot connectivity with `ping`, `traceroute`, `ss`
- [ ] Transfer files with `scp` and `rsync`
- [ ] Set up and use SSH key authentication

### Scripting
- [ ] Write a bash script with variables, conditionals, and loops
- [ ] Implement error handling in scripts
- [ ] Schedule tasks with cron
- [ ] Write reusable functions

### Practical Skills
- [ ] Install and manage packages
- [ ] Read and analyze log files
- [ ] Set up a basic firewall with `ufw`
- [ ] Create and extract archives
- [ ] Monitor disk, memory, and CPU

---

> _"The strength of Linux is its simplicity: everything is a file, and every tool does one thing well. Master these building blocks and you can build anything."_

---

*Complete Linux Fundamentals Guide | DevOps Learning Series*
*Covers: RHEL 8+, CentOS Stream, Ubuntu 20.04/22.04 LTS, Debian 11+*
