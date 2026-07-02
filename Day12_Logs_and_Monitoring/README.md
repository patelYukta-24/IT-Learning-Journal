# Day 12 : Logs and Monitoring

> **DevOps Transformation** | Linux Fundamentals Series — Bonus Day

---

## 📋 Overview

Every previous day taught you how to *do* things on a Linux system — manage files, processes, packages, disks. This bonus day teaches you how to know **what already happened**, and **what's happening right now**.

Logs are the system's memory: every service, every login, every failed command leaves a trace somewhere under `/var/log`. Monitoring tools are the system's pulse: real-time visibility into CPU, memory, disk I/O, and what's consuming them. Together, these are the two skills that separate someone who can *use* Linux from someone who can *operate* it in production — diagnosing an outage, catching a resource leak before it causes one, or proving to an auditor exactly what happened and when.

---

## 🗺️ What You'll Learn Today

| Topic | Commands / Concepts | Why It Matters |
|---|---|---|
| System Log Locations | `/var/log/syslog`, `/var/log/auth.log`, `/var/log/dmesg` | Know where to look first when something breaks |
| Logging Daemons | `syslog`, `rsyslog` | Understand what actually writes and routes log messages |
| Custom Logging | `rsyslog` config, facilities, severities | Route specific application logs to their own files |
| Live Process Monitoring | `htop`, `top` | See what's consuming CPU/memory right now |
| Disk I/O Monitoring | `iostat` | Identify disk bottlenecks |
| Memory/CPU Monitoring | `vmstat` | Spot memory pressure and CPU contention over time |

---

## 🏢 NexusCorp Scenario

> A NexusCorp production web server has been intermittently slow for the past hour, and the on-call engineer has no dashboard access — just SSH. Support tickets are piling up, and nobody can say yet whether it's a CPU spike, a memory leak, a disk bottleneck, or a misbehaving service. There's also a routine security review coming up, and the compliance team wants proof that failed login attempts are being logged and retained.
>
> Today you learn to answer both: diagnose a live performance issue with nothing but the command line, and read the logs that prove what happened.

---

## Part 1: System Logs — `/var/log`

### Why Logs Live in `/var/log`

Recall from Day 11's filesystem hierarchy: `/var` holds **variable data** — files that grow and change during normal system operation. Logs are the biggest category of that. Nearly every service on a Linux system writes its own log file here.

### Key Log Files to Know

| Log File | Contains |
|---|---|
| `/var/log/syslog` (Debian/Ubuntu) | General system activity — the default catch-all log |
| `/var/log/messages` (RHEL/CentOS) | Equivalent general system log on RPM-based distros |
| `/var/log/auth.log` (Debian/Ubuntu) | Authentication events — logins, `sudo` usage, SSH attempts |
| `/var/log/secure` (RHEL/CentOS) | Equivalent authentication log on RPM-based distros |
| `/var/log/dmesg` | Kernel ring buffer — boot messages, hardware/driver events |
| `/var/log/kern.log` | Kernel-specific log messages |
| `/var/log/apt/history.log` | Package install/upgrade/remove history (from Day 8) |
| `/var/log/cron` or `/var/log/syslog` (cron entries) | Cron job execution records (from Day 10) |
| `/var/log/nginx/`, `/var/log/apache2/` | Web server access and error logs |
| `/var/log/wtmp`, `/var/log/btmp` | Binary logs of successful/failed login sessions (read via `last`, `lastb`) |

### Reading Logs

```bash
# View the end of a log (most recent entries)
tail -50 /var/log/syslog

# Follow a log live as new entries arrive — essential for real-time debugging
tail -f /var/log/syslog

# Search for a specific term across a log
grep -i "error" /var/log/syslog

# Search for failed SSH login attempts
grep "Failed password" /var/log/auth.log

# View kernel boot messages
dmesg | less

# View recent kernel messages with human-readable timestamps
dmesg -T | tail -30

# See login history (successful logins)
last

# See failed login attempts
lastb
```

### Interpreting a Typical `syslog` Entry

```
Jul  2 09:14:33 nexus-web01 sshd[2841]: Failed password for invalid user admin from 203.0.113.45 port 51422 ssh2
```

| Field | Value | Meaning |
|---|---|---|
| Timestamp | `Jul 2 09:14:33` | When the event occurred |
| Hostname | `nexus-web01` | Which machine logged it |
| Process[PID] | `sshd[2841]` | Which service and process ID generated the entry |
| Message | `Failed password...` | The actual event detail |

### Log Rotation

Logs would grow forever without management. `logrotate` handles this automatically — compressing, archiving, and deleting old logs on a schedule.

```bash
# View the logrotate configuration
cat /etc/logrotate.conf
ls /etc/logrotate.d/

# Manually trigger rotation (for testing)
sudo logrotate -f /etc/logrotate.conf
```

---

## Part 2: `syslog` and `rsyslog`

### What Is `syslog`?

**Syslog** is both a *standard* (a protocol for logging messages, defined in RFC 5424) and a general term for the logging system built on that standard. It defines a common format for log messages, including two key classifications:

| Concept | Examples |
|---|---|
| **Facility** — what subsystem generated the message | `auth`, `cron`, `daemon`, `kern`, `mail`, `user`, `local0`–`local7` |
| **Severity** — how serious the message is | `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug` |

### What Is `rsyslog`?

**`rsyslog`** ("rocket-fast syslog") is the modern daemon that actually implements syslog on most Linux distributions today. It listens for log messages from the kernel and applications, then routes them to files, remote servers, or databases based on rules you define.

```bash
# Check whether rsyslog is running
sudo systemctl status rsyslog

# Main configuration file
cat /etc/rsyslog.conf

# Additional modular config files
ls /etc/rsyslog.d/
```

### Anatomy of an `rsyslog` Rule

Rules follow the pattern: `facility.severity    destination`

```
# Example rules from /etc/rsyslog.conf

# Log all authentication messages to auth.log
auth,authpriv.*                /var/log/auth.log

# Log everything except auth/mail/cron to syslog
*.*;auth,authpriv.none         -/var/log/syslog

# Log all kernel messages to their own file
kern.*                         -/var/log/kern.log

# Log all cron activity separately
cron.*                         /var/log/cron.log
```

### Configuring Custom Logging

**Scenario:** NexusCorp wants a custom application ("nexusapp") to log to its own file instead of getting mixed into `syslog`.

```bash
# Step 1: Create a new rsyslog config file
sudo nano /etc/rsyslog.d/30-nexusapp.conf

# Step 2: Add a rule routing the local0 facility to a dedicated file
local0.*    /var/log/nexusapp.log

# Step 3: Restart rsyslog to apply the change
sudo systemctl restart rsyslog

# Step 4: Test it — send a test message using the logger command
logger -p local0.info "NexusApp test message: service started"

# Step 5: Verify it landed in the right file
tail /var/log/nexusapp.log
```

`logger` is the standard way to send a message into the syslog system manually — useful for testing rules or having your own scripts write structured log entries.

```bash
# Send a message with a specific facility and severity
logger -p local0.warning "Custom warning message"

# Send a message with a tag for easier filtering
logger -t nexusapp "Deployment completed successfully"
```

---

## Part 3: Monitoring Tools

### `top` — Classic Real-Time Process Monitor

```bash
top
```

Key columns:

| Column | Meaning |
|---|---|
| `PID` | Process ID |
| `USER` | Owner of the process |
| `%CPU` | Percentage of CPU currently used |
| `%MEM` | Percentage of RAM currently used |
| `TIME+` | Total CPU time consumed since start |
| `COMMAND` | The process name |

Inside `top`: press `P` to sort by CPU, `M` to sort by memory, `k` to kill a process by PID, `q` to quit.

### `htop` — Improved, Interactive Process Monitor

`htop` is a more visual, easier-to-navigate alternative to `top` — color-coded bars for CPU/memory, mouse support, and simpler process management.

```bash
# Install if not already present
sudo apt install htop      # Debian/Ubuntu
sudo yum install htop      # RHEL/CentOS

# Launch
htop
```

Inside `htop`: arrow keys to navigate, `F9` to kill a selected process, `F6` to sort by a column, `F5` for tree view (shows parent/child process relationships).

### `vmstat` — Virtual Memory Statistics

`vmstat` gives a snapshot of overall system performance: processes, memory, swap, I/O, and CPU — in one compact table, ideal for spotting trends over time.

```bash
# One-time snapshot
vmstat

# Repeat every 2 seconds, 5 times
vmstat 2 5
```

Key fields:

| Field | Meaning |
|---|---|
| `r` | Number of processes waiting for CPU time (high = CPU bottleneck) |
| `b` | Processes blocked, waiting on I/O |
| `swpd` | Amount of virtual memory used (swap) |
| `free` | Idle memory |
| `si` / `so` | Memory swapped in/out per second (any non-zero value under load = memory pressure) |
| `us` | % CPU time spent in user space |
| `sy` | % CPU time spent in kernel/system space |
| `id` | % CPU time idle |
| `wa` | % CPU time waiting on I/O (high = disk bottleneck) |

### `iostat` — Disk I/O Statistics

`iostat` reports how busy your disks are — essential for confirming whether slowness is a disk problem.

```bash
# Install if not already present (part of the sysstat package)
sudo apt install sysstat      # Debian/Ubuntu
sudo yum install sysstat      # RHEL/CentOS

# One-time snapshot, human-readable
iostat -h

# Repeat every 2 seconds, 5 times, extended stats
iostat -x 2 5
```

Key fields:

| Field | Meaning |
|---|---|
| `%util` | Percentage of time the disk was busy servicing requests (near 100% = saturated) |
| `await` | Average time (ms) for I/O requests to complete — high values indicate a slow/overloaded disk |
| `r/s`, `w/s` | Read/write operations per second |

### Choosing the Right Tool

| Question | Tool |
|---|---|
| "Which process is eating CPU/memory right now?" | `htop` or `top` |
| "Is the system under general CPU/memory/swap pressure?" | `vmstat` |
| "Is the disk the bottleneck?" | `iostat` |
| "What happened in the past (not right now)?" | Logs (`/var/log/`) |

---

## 🛠️ Practical Exercises

### Exercise 1: Browse and Interpret Logs

```bash
# View recent system activity
tail -50 /var/log/syslog

# Find all authentication-related entries from today
grep "$(date '+%b %e')" /var/log/auth.log

# Check for any failed login attempts
grep "Failed password" /var/log/auth.log

# Review kernel boot messages
dmesg | head -30

# Identify the largest log file currently on the system
sudo du -ah /var/log | sort -rh | head -5
```

### Exercise 2: Configure Custom `rsyslog` Logging

```bash
# Step 1: Create a custom rule for a fictional "labapp"
sudo nano /etc/rsyslog.d/30-labapp.conf
# Add: local1.*    /var/log/labapp.log

# Step 2: Restart rsyslog
sudo systemctl restart rsyslog

# Step 3: Send test messages at different severities
logger -p local1.info "Lab app: informational test message"
logger -p local1.err "Lab app: simulated error message"

# Step 4: Confirm both landed correctly
cat /var/log/labapp.log
```

### Exercise 3: Identify a Bottleneck with Monitoring Tools

```bash
# Step 1: Launch htop and observe for 1 minute — note the top CPU/memory consumers
htop

# Step 2: Check for memory pressure or CPU contention
vmstat 2 5

# Step 3: Check for disk bottlenecks
iostat -x 2 5

# Step 4: Simulate load to see monitoring tools respond (safe on a test VM only)
yes > /dev/null &
htop   # Observe the new process consuming a full CPU core

# Step 5: Clean up the test process
kill %1
```

---

## 🔧 Mini-Project: NexusCorp Incident Diagnosis

**Scenario:** The web server slowdown from today's opening scenario needs a real diagnosis, and the compliance team needs an authentication log summary.

```bash
# --- Part A: Diagnose the slowdown ---

echo "=== NexusCorp Incident Report: $(date) ===" > /tmp/nexus_incident.txt

echo "--- Top CPU/Memory Consumers ---" >> /tmp/nexus_incident.txt
ps aux --sort=-%cpu | head -6 >> /tmp/nexus_incident.txt

echo "--- System Performance Snapshot ---" >> /tmp/nexus_incident.txt
vmstat 2 3 >> /tmp/nexus_incident.txt

echo "--- Disk I/O Snapshot ---" >> /tmp/nexus_incident.txt
iostat -x 2 3 >> /tmp/nexus_incident.txt

echo "--- Recent System Errors ---" >> /tmp/nexus_incident.txt
grep -i "error\|fail" /var/log/syslog | tail -20 >> /tmp/nexus_incident.txt

cat /tmp/nexus_incident.txt

# --- Part B: Authentication audit for compliance ---

echo "=== Authentication Audit: $(date) ===" > /tmp/nexus_auth_audit.txt

echo "--- Failed Login Attempts ---" >> /tmp/nexus_auth_audit.txt
grep "Failed password" /var/log/auth.log >> /tmp/nexus_auth_audit.txt

echo "--- Successful Logins ---" >> /tmp/nexus_auth_audit.txt
last -20 >> /tmp/nexus_auth_audit.txt

echo "--- Sudo Usage ---" >> /tmp/nexus_auth_audit.txt
grep "sudo" /var/log/auth.log | tail -20 >> /tmp/nexus_auth_audit.txt

cat /tmp/nexus_auth_audit.txt
```

**Bonus Challenge:** Write a one-line command that counts failed SSH login attempts per source IP address, sorted from most to least frequent (a common first step in spotting a brute-force attempt).
<details>
<summary>Hint</summary>

```bash
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
```
</details>

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Log** | A recorded entry documenting an event that occurred on the system |
| **`/var/log`** | The standard directory where Linux system and application logs are stored |
| **syslog** | Both the standard protocol for system logging and the general logging subsystem it describes |
| **rsyslog** | The modern daemon that implements syslog on most Linux distributions today |
| **Facility** | A syslog classification describing which subsystem generated a message (`auth`, `cron`, `kern`, etc.) |
| **Severity** | A syslog classification describing how serious a message is (`emerg` through `debug`) |
| **`logger`** | Command-line tool used to manually send a message into the syslog system |
| **logrotate** | Utility that automatically compresses, archives, and deletes old log files on a schedule |
| **`top`** | Classic real-time process monitor showing CPU/memory usage per process |
| **`htop`** | Interactive, color-coded alternative to `top` |
| **`vmstat`** | Reports system-wide CPU, memory, and swap statistics over time |
| **`iostat`** | Reports disk I/O statistics, used to identify disk bottlenecks |
| **`%util`** | In `iostat`, the percentage of time a disk was busy servicing requests |
| **Bottleneck** | The specific resource (CPU, memory, disk, network) limiting overall system performance |

---

## 📚 Resources

- [Arch Wiki — rsyslog](https://wiki.archlinux.org/title/Rsyslog) — configuration reference and rule syntax
- [DigitalOcean — How To Use Logging and Log Rotation](https://www.digitalocean.com/community/tutorials/how-to-configure-rsyslog-on-ubuntu) — practical rsyslog and logrotate walkthrough
- [Linux Journal — Understanding vmstat and iostat](https://www.linuxjournal.com/) — background on interpreting resource stats
- [htop Official Site](https://htop.dev/) — official documentation and feature reference
- [RFC 5424 — The Syslog Protocol](https://www.rfc-editor.org/rfc/rfc5424) — the formal syslog specification, for reference

---

## 🏁 Bonus Day Complete

This optional day extends the **NexusCorp Linux Fundamentals Series** further:

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
| Day 11 | Filesystem Hierarchy & Disk Management: FHS, `fdisk`, `mkfs`, `df`, `du` |
| **Day 12** | **Logs & Monitoring: `/var/log`, `rsyslog`, `htop`, `vmstat`, `iostat`** |

**What comes next:** AWS Cloud Fundamentals, Git & GitHub, and Shell Scripting deep-dives — building on this Linux foundation toward a complete DevOps skill set.

---

> 💬 *"When the dashboard is down, the logs are the truth. Learn to read them before you need to."*
>
> — DevOps Learning by Yukta
