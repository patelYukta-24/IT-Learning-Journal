# Day 11 : Filesystem Hierarchy and Disk Management

> **DevOps Transformation** | Linux Fundamentals Series — Bonus Day

---

## 📋 Overview

The core 10-day series covered navigating, using, and scripting a Linux system. But every one of those days assumed the disk was already partitioned, formatted, and mounted correctly. This bonus day goes one layer deeper: **how Linux organizes storage itself.**

Understanding the Filesystem Hierarchy Standard (FHS) tells you *where* things live and *why* — critical when you're troubleshooting a full disk, debugging a broken service, or writing a script that needs to know where logs, configs, or binaries are supposed to be. Disk partitioning and filesystem management tell you *how* raw storage becomes something Linux can actually use.

This is exactly the knowledge a DevOps or Cloud engineer reaches for when provisioning a new server, resizing storage on a cloud instance, or diagnosing a "disk full" alert at 3 AM.

---

## 🗺️ What You'll Learn Today

| Topic | Commands / Concepts | Why It Matters |
|---|---|---|
| Filesystem Hierarchy Standard (FHS) | `/etc`, `/var`, `/usr`, `/home`, etc. | Know where every type of file belongs, on any Linux system |
| Disk Partitioning | MBR vs GPT, primary/extended/logical | Understand how a disk gets divided before it can be used |
| Partitioning Tools | `fdisk`, `parted`, `gdisk` | Create, view, and modify partitions |
| Filesystem Creation | `mkfs`, filesystem types (ext4, xfs) | Format a partition so Linux can store files on it |
| Mounting | `mount`, `umount`, `/etc/fstab` | Attach a filesystem to the directory tree so it's usable |
| Disk Usage Monitoring | `df`, `du` | Answer "what's using my disk space?" quickly and accurately |

---

## 🏢 NexusCorp Scenario

> NexusCorp is provisioning a new staging server. The base OS is installed, but the ops team has attached an additional 50GB block storage volume that needs to be partitioned, formatted, and mounted before the application team can use it for log storage. Meanwhile, a production alert just came in: `/var` is at 95% capacity and nobody knows why.
>
> Today you learn both sides of this: setting up new storage correctly, and diagnosing where existing storage went.

---

## Part 1: The Linux Filesystem Hierarchy Standard (FHS)

### Why a Standard Exists

Unlike Windows (`C:\Program Files`, `C:\Users`), Linux doesn't organize files by drive letter — everything lives under a single root directory, `/`. The **Filesystem Hierarchy Standard (FHS)** defines what each top-level directory is *for*, so that software, scripts, and administrators can predict where things live on any distribution.

```
/
├── bin/     → Essential user command binaries (ls, cp, cat)
├── sbin/    → Essential system binaries (fdisk, reboot, iptables)
├── etc/     → System-wide configuration files
├── var/     → Variable data: logs, caches, spool files, databases
├── usr/     → User-installed software, libraries, documentation
├── home/    → Personal directories for each user
├── root/    → Home directory for the root user (not /root of the tree — the user's home)
├── opt/     → Optional/third-party software packages
├── tmp/     → Temporary files, cleared on reboot
├── mnt/     → Temporary mount point for manually mounted filesystems
├── media/   → Auto-mount point for removable media (USB, CD)
├── dev/     → Device files (disks, terminals, USB devices)
├── proc/    → Virtual filesystem exposing kernel/process info
├── boot/    → Kernel, bootloader (GRUB) files
└── lib/     → Shared libraries needed by /bin and /sbin binaries
```

### Directory-by-Directory Breakdown

| Directory | Purpose | Real Example |
|---|---|---|
| `/etc` | Configuration files for the system and installed software | `/etc/passwd`, `/etc/ssh/sshd_config`, `/etc/fstab` |
| `/var` | Files that change frequently during normal operation | `/var/log/syslog`, `/var/log/apt/history.log`, `/var/www` |
| `/usr` | The bulk of installed software and its supporting files | `/usr/bin`, `/usr/lib`, `/usr/share/doc` |
| `/home` | Each user's personal files and settings | `/home/yukta/` |
| `/root` | Root user's home directory (kept separate from `/home` for boot-time safety) | `/root/.bashrc` |
| `/tmp` | Scratch space for any process; typically wiped on reboot | Temp files created by installers, scripts |
| `/opt` | Self-contained third-party applications | `/opt/google/chrome/` |
| `/dev` | Every hardware device represented as a file | `/dev/sda` (a disk), `/dev/null`, `/dev/tty` |
| `/proc` | Live kernel and process data, not real files on disk | `/proc/cpuinfo`, `/proc/meminfo`, `/proc/<pid>/` |
| `/boot` | Files needed to boot the system before the OS is fully loaded | `/boot/vmlinuz`, `/boot/grub/` |
| `/mnt`, `/media` | Attachment points for extra storage | Mounted USB drives, extra disks |

### Why This Matters in Practice

- **Troubleshooting a full disk** — you check `/var/log` first, because logs are the most common silent disk-filler.
- **Writing scripts** — a well-behaved script writes temp files to `/tmp`, not `/home` or `/etc`.
- **Reading documentation** — man pages, `/usr/share/doc`, and config file locations all follow FHS, so once you know the standard, you can navigate *any* distro.

### Quick Inspection Commands

```bash
# See the top-level hierarchy
ls -l /

# See how much space each top-level directory consumes (single level deep)
du -sh /* 2>/dev/null

# See what's inside /var — usually the biggest offender in a "disk full" situation
sudo du -sh /var/* 2>/dev/null | sort -rh | head -10
```

---

## Part 2: Disk Partitioning

### What Is a Partition?

A **partition** divides a physical (or virtual) disk into separate, independently-manageable sections. Each partition can hold its own filesystem, be mounted separately, and be resized or reformatted without touching the others.

Reasons to partition a disk:
- Separate the OS from data (so a full `/home` doesn't crash the whole system)
- Isolate `/var` or `/tmp` so runaway logs can't fill the root filesystem
- Support multiple operating systems on one disk
- Apply different filesystem types or mount options to different types of data

### Partitioning Schemes: MBR vs GPT

| Aspect | MBR (Master Boot Record) | GPT (GUID Partition Table) |
|---|---|---|
| Max partitions | 4 primary (or 3 primary + 1 extended with logical partitions inside) | 128 by default |
| Max disk size | 2 TB | 9.4 ZB (effectively unlimited today) |
| Redundancy | Single copy of partition table — no backup | Partition table stored at both start and end of disk |
| Modern default | Legacy systems, older BIOS | Standard for modern systems (UEFI) |

**In practice:** almost all modern cloud instances and new hardware use GPT. You'll still encounter MBR on older or legacy VMs.

### Primary, Extended, and Logical Partitions (MBR only)

```
MBR Disk (max 4 primary partitions)
│
├── Primary Partition 1
├── Primary Partition 2
├── Primary Partition 3
└── Extended Partition 4
       ├── Logical Partition 5
       ├── Logical Partition 6
       └── Logical Partition 7   ← Extended acts as a container for more partitions
```

GPT removes this complexity entirely — there's no primary/extended/logical distinction, just up to 128 partitions directly.

### Viewing Existing Partitions

```bash
# List all disks and their partitions
lsblk

# Detailed partition table for a specific disk
sudo fdisk -l /dev/sda

# Alternative, works well with both MBR and GPT
sudo parted -l

# Show disk UUID and filesystem type for every partition
sudo blkid
```

### Creating a Partition with `fdisk`

> ⚠️ **Safety note:** Partitioning is destructive if done on the wrong disk. Always confirm the device name with `lsblk` first, and practice only on a spare virtual disk in a VM — never on a disk containing data you need.

```bash
# Open fdisk on the target disk (example: a second disk, sdb)
sudo fdisk /dev/sdb

# Inside the interactive fdisk prompt:
#   n   → create a new partition
#   p   → primary partition
#   1   → partition number
#   [Enter] [Enter]  → accept default start/end sectors (uses full disk)
#   w   → write changes to disk and exit

# Verify the new partition exists
lsblk
sudo fdisk -l /dev/sdb
```

### Creating a GPT Partition with `parted`

```bash
sudo parted /dev/sdb

# Inside parted:
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) print
(parted) quit
```

---

## Part 3: Filesystem Management

### Creating a Filesystem with `mkfs`

A partition is just empty space until it's **formatted** with a filesystem — the structure that lets Linux actually organize and retrieve files on it.

| Filesystem | Common Use Case |
|---|---|
| `ext4` | Default for most Linux distributions — stable, mature, widely supported |
| `xfs` | Default on RHEL/CentOS — strong performance with large files |
| `vfat` / `fat32` | Compatibility with Windows, USB drives |
| `ntfs` | Reading/writing Windows-formatted drives |

```bash
# Format a partition as ext4
sudo mkfs.ext4 /dev/sdb1

# Format as xfs
sudo mkfs.xfs /dev/sdb1

# Check the result
sudo blkid /dev/sdb1
```

### Mounting a Filesystem

Formatting alone doesn't make a filesystem usable — it must be **mounted** to a directory (a "mount point") before files can be read or written to it.

```bash
# Create a mount point
sudo mkdir -p /mnt/data

# Mount the partition to that directory
sudo mount /dev/sdb1 /mnt/data

# Confirm it's mounted
df -h /mnt/data
mount | grep sdb1

# Unmount when done
sudo umount /mnt/data
```

### Making a Mount Permanent with `/etc/fstab`

A manual `mount` doesn't survive a reboot. To mount a filesystem automatically at boot, add an entry to `/etc/fstab`:

```bash
# Find the partition's UUID (safer than using /dev/sdb1, which can change)
sudo blkid /dev/sdb1

# Edit fstab
sudo nano /etc/fstab

# Add a line like this:
UUID=xxxx-xxxx-xxxx-xxxx  /mnt/data  ext4  defaults  0  2
```

| fstab Field | Meaning |
|---|---|
| Device/UUID | Which partition to mount |
| Mount point | Where to mount it |
| Filesystem type | `ext4`, `xfs`, etc. |
| Options | `defaults`, `ro`, `noexec`, etc. |
| Dump | Backup flag (0 = skip, legacy) |
| fsck order | Filesystem check order at boot (0 = skip, 1 = root, 2 = others) |

```bash
# Test an fstab entry without rebooting
sudo mount -a

# If it mounts without errors, the entry is valid
```

> ⚠️ A broken `/etc/fstab` entry can prevent a system from booting properly. Always test with `mount -a` before rebooting, and keep a backup of the original file (`sudo cp /etc/fstab /etc/fstab.bak`) before editing.

---

## Part 4: Monitoring Disk Usage — `df` and `du`

These are the two commands you'll use constantly to answer "how much space is left" and "what's actually using it."

### `df` — Disk Free (filesystem-level view)

```bash
# Human-readable disk space per mounted filesystem
df -h

# Show filesystem type as well
df -hT

# Check a specific mount point
df -h /var
```

Example output interpretation:
```
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/sda1        50G   38G    10G   80%  /
/dev/sdb1        50G    2G    48G    4%  /mnt/data
```

### `du` — Disk Usage (directory/file-level view)

```bash
# Total size of a directory, human-readable
du -sh /var/log

# Size of every subdirectory, one level deep, sorted largest first
du -h --max-depth=1 /var | sort -rh

# Find the 10 largest directories under /var
sudo du -ah /var | sort -rh | head -10
```

### `df` vs `du` — When to Use Which

| Question | Command |
|---|---|
| "Is this disk/partition full?" | `df -h` |
| "What inside this directory is taking up space?" | `du -sh` |
| "Which subdirectory is the culprit?" | `du -h --max-depth=1 | sort -rh` |

**Common gotcha:** `df` and `du` can disagree if a file is deleted while a process still has it open — the space isn't released until the process closes it. This shows up as `df` reporting a nearly-full disk while `du` can't account for all the space. Check with:

```bash
sudo lsof +L1
```

---

## 🛠️ Practical Exercises

> Run all partitioning and formatting exercises on a **spare virtual disk in a VM** — never on your primary system disk.

### Exercise 1: Explore the Filesystem Hierarchy

```bash
# List the top-level directories
ls -l /

# Find the 5 largest top-level directories
du -sh /* 2>/dev/null | sort -rh | head -5

# Find the 5 largest items inside /var specifically
sudo du -sh /var/* 2>/dev/null | sort -rh | head -5

# Identify what filesystem type your root partition uses
df -hT /
```

### Exercise 2: Add and Partition a Virtual Disk

1. In your VM software (VirtualBox/VMware), attach a new virtual disk (e.g., 10GB).
2. Boot the VM and identify the new disk:
   ```bash
   lsblk
   ```
3. Partition it with `fdisk`, format it with `ext4`, and mount it to `/mnt/data`.
4. Confirm with `df -h` and `lsblk`.

### Exercise 3: Monitor Disk Usage

```bash
# Create some test files to fill space
sudo dd if=/dev/zero of=/mnt/data/testfile bs=1M count=500

# Confirm the space was used
df -h /mnt/data
du -sh /mnt/data

# Clean up
sudo rm /mnt/data/testfile
df -h /mnt/data
```

---

## 🔧 Mini-Project: NexusCorp Storage Provisioning + Diagnosis

**Scenario:** NexusCorp needs the new 50GB volume set up for log storage, and the ops team needs to figure out why `/var` is at 95% on the existing production server.

```bash
# --- Part A: Provision the new volume ---

# Step 1: Identify the new disk
lsblk

# Step 2: Partition it (GPT, single partition)
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 0% 100%

# Step 3: Format it
sudo mkfs.ext4 /dev/sdb1

# Step 4: Create the mount point and mount it
sudo mkdir -p /mnt/nexuscorp-logs
sudo mount /dev/sdb1 /mnt/nexuscorp-logs

# Step 5: Make it permanent
sudo blkid /dev/sdb1
echo "UUID=<paste-uuid-here>  /mnt/nexuscorp-logs  ext4  defaults  0  2" | sudo tee -a /etc/fstab
sudo mount -a

# Step 6: Verify
df -h /mnt/nexuscorp-logs

# --- Part B: Diagnose the full /var on the existing server ---

# Step 1: Confirm the problem
df -h /var

# Step 2: Find the biggest offenders
sudo du -h --max-depth=1 /var | sort -rh

# Step 3: Drill into the worst offender (commonly /var/log)
sudo du -ah /var/log | sort -rh | head -10

# Step 4: Document findings before taking action
echo "=== /var Disk Audit ===" > /tmp/nexus_var_audit.txt
df -h /var >> /tmp/nexus_var_audit.txt
sudo du -h --max-depth=1 /var | sort -rh >> /tmp/nexus_var_audit.txt
cat /tmp/nexus_var_audit.txt
```

**Bonus Challenge:** Write a one-line command that reports the top 5 largest files anywhere under `/var/log`, regardless of subdirectory depth.
<details>
<summary>Hint</summary>

```bash
sudo find /var/log -type f -exec du -h {} + | sort -rh | head -5
```
</details>

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **FHS** | Filesystem Hierarchy Standard — defines the purpose of each top-level Linux directory |
| **Partition** | A logically separated section of a physical or virtual disk |
| **MBR** | Master Boot Record — legacy partitioning scheme, max 4 primary partitions, 2TB limit |
| **GPT** | GUID Partition Table — modern partitioning scheme, up to 128 partitions, no practical size limit |
| **Filesystem** | The structure that organizes how data is stored and retrieved on a partition (ext4, xfs, etc.) |
| **`mkfs`** | Command used to create ("format") a filesystem on a partition |
| **Mount** | The act of attaching a filesystem to a directory so it becomes accessible |
| **Mount point** | The directory a filesystem is attached to |
| **`/etc/fstab`** | Configuration file defining filesystems to mount automatically at boot |
| **UUID** | A unique identifier assigned to a filesystem, used in `/etc/fstab` instead of a device name that can change |
| **`df`** | Reports disk space usage at the filesystem/partition level |
| **`du`** | Reports disk space usage at the file/directory level |

---

## 📚 Resources

- [Filesystem Hierarchy Standard — official spec](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html) — the formal FHS 3.0 definition
- [Arch Wiki — Partitioning](https://wiki.archlinux.org/title/Partitioning) — deep, distro-agnostic partitioning reference
- [Arch Wiki — fstab](https://wiki.archlinux.org/title/Fstab) — detailed fstab field reference
- [DigitalOcean — Understanding df and du](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-filesystems) — practical disk usage walkthrough
- [RHEL Documentation — Managing File Systems](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_file_systems/index) — enterprise-grade reference for mkfs, mount, and fstab

---

## 🏁 Bonus Day Complete

This optional day extends the **NexusCorp Linux Fundamentals Series** beyond its original 10 days:

| Day | Topic |
|---|---|
| Day 1 | Linux History, Distributions, and Filesystem Navigation |
| Day 4 | File Operations: `touch`, `mkdir`, `cp`, `mv`, `rm` |
| Day 5 | Text Manipulation: `cat`, `echo`, `nano`, `vi` |
| Day 6 | Permissions & Ownership: `chmod`, `chown`, SUID, SGID, Sticky Bit |
| Day 7 | Process Management: `ps`, `top`, `kill`, `nice` |
| Day 8 | Package Management: `apt`, `yum` |
| Day 9 | Networking & Text Processing: `ping`, `ifconfig`, `grep`, `awk`, `sed` |
| Day 10 | Shell Scripting & Task Scheduling: Bash, `cron`, `at` |
| **Day 11** | **Filesystem Hierarchy & Disk Management: FHS, `fdisk`, `mkfs`, `df`, `du`** |

**What comes next:** AWS Cloud Fundamentals, Git & GitHub, and Shell Scripting deep-dives — building on this Linux foundation toward a complete DevOps skill set.

---

> 💬 *"You can't optimize storage you don't understand. Know your hierarchy, know your disks."*
>
> — DevOps Learning by Yukta
