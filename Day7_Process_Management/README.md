# Day 7: Process Management

> **DevOps Transformation** | Linux Fundamentals Series

---

## 📋 Overview

Yesterday you learned who can touch a file. Today you learn what's actually *running* on a system — and how to take control of it.

Every program you run, every background service, every script that fires off a cron job is a **process**. As a DevOps engineer, the ability to see what's running, how much it's consuming, and when to kill it is one of the most-used skills in day-to-day troubleshooting. A server running slow at 2 AM isn't a mystery if you know how to read `top`.

---

## 🗺️ What You'll Learn Today

| Topic | Commands / Concepts | Why It Matters |
|---|---|---|
| Processes & Threads | Process lifecycle, states, PIDs | Understand what's actually happening under the hood |
| Viewing Processes | `ps`, flags `-e`, `-f`, `-u` | Snapshot what's running right now |
| Real-Time Monitoring | `top` | Watch CPU/memory usage live, catch problems as they happen |
| Controlling Processes | `kill`, signals | Stop misbehaving or stuck processes |
| Priority Management | `nice`, `renice` | Control how much CPU time a process gets |

---

## 🏢 NexusCorp Scenario

> It's 2 AM and NexusCorp's monitoring system just paged the on-call engineer: the staging server's CPU is pinned at 100%. There's no dashboard open, no GUI — just an SSH session and a terminal. The engineer needs to figure out what's eating the CPU, decide whether to kill it or let it finish, and document what happened before the next stand-up.
>
> This is exactly the skill set you're building today.

---

## Part 1: Understanding Processes and Threads

### What Is a Process?

A **process** is an instance of a running program. The program itself — say, `/usr/bin/python3` — is just a file sitting on disk. The moment you execute it, the kernel allocates it memory, a process ID, and CPU time, and it becomes a living, running process.

```
Program (on disk)  →  Execution  →  Process (in memory, running)
  /usr/bin/nginx          ↓           PID 4821, using 2% CPU, 15MB RAM
```

**Key distinction:**

| Concept | Definition |
|---|---|
| **Program** | A static file containing instructions, sitting on disk |
| **Process** | A running instance of that program, with its own memory, PID, and resources |
| **Thread** | The smallest unit of execution within a process — a process can have many threads sharing the same memory space |

You can run the same program multiple times and get multiple independent processes. Open three terminal windows running `bash`, and you have three separate `bash` processes — same program, different PIDs, different memory.

---

### Processes vs Threads

A process can contain **one or more threads**. Threads within the same process share memory and resources but execute independently, which makes them lighter-weight than spawning a whole new process.

```
┌─────────────────────────────────────┐
│  Process: nginx (PID 4821)           │
│  ┌─────────┐  ┌─────────┐           │
│  │Thread 1 │  │Thread 2 │  ...      │
│  │(worker) │  │(worker) │           │
│  └─────────┘  └─────────┘           │
│   Shared memory space                │
└─────────────────────────────────────┘
```

**💡 NexusCorp Use Case:** A web server like nginx spawns multiple worker threads to handle simultaneous requests without needing to start an entirely new process for each connection — far more efficient.

---

### The Process Lifecycle and States

Every process moves through a lifecycle, and at any moment it exists in one of several **states**:

| State | Symbol (in `ps`/`top`) | Meaning |
|---|---|---|
| **Running (R)** | `R` | Actively executing on the CPU, or ready to run |
| **Sleeping (S)** | `S` | Waiting for an event (e.g., user input, disk I/O) — interruptible |
| **Uninterruptible Sleep (D)** | `D` | Waiting on I/O that cannot be interrupted (e.g., disk read) |
| **Stopped (T)** | `T` | Paused, usually by a signal (`Ctrl+Z` or `SIGSTOP`) |
| **Zombie (Z)** | `Z` | Finished executing, but its exit status hasn't been collected by its parent yet |

**Visualizing the lifecycle:**
```
   [Created] → [Running] ⇄ [Sleeping]
                   │
                   ↓
            [Stopped] (optional, via signal)
                   │
                   ↓
              [Terminated] → [Zombie] → [Reaped by parent]
```

**Check process states yourself:**
```bash
ps aux | awk '{print $8}' | sort | uniq -c
```

---

### Parent and Child Processes

Every process (except the very first one, `init`/`systemd` with PID 1) is spawned by another process — its **parent**. The parent can spawn multiple **children**, forming a process tree.

```bash
# View the process tree
ps -ef --forest

# Or use pstree for a cleaner visual
pstree -p
```

**Example output:**
```
systemd(1)─┬─sshd(892)───sshd(1204)───bash(1205)───ps(1310)
           ├─cron(745)
           └─nginx(4821)─┬─nginx(4822)
                          └─nginx(4823)
```

Here, `bash` (1205) is a child of `sshd` (1204), and the `ps` command (1310) is itself a temporary child of `bash`. When a parent process terminates, its children either get reassigned to `init`/`systemd` or are terminated along with it, depending on how the system is configured.

**💡 NexusCorp Use Case:** Understanding parent-child relationships matters when killing processes — kill the wrong parent and you might unexpectedly terminate a whole tree of dependent child processes.

---

## Part 2: `ps` — Viewing Active Processes

`ps` (**p**rocess **s**tatus) takes a snapshot of currently running processes. Unlike `top`, it doesn't update live — it shows you the state of the system at the moment you ran the command.

---

### Basic Usage

```bash
# Show processes running in your current terminal session
ps

# Sample output:
#   PID TTY          TIME CMD
#  1205 pts/0    00:00:00 bash
#  1310 pts/0    00:00:00 ps
```

This default view is intentionally narrow — it only shows processes tied to your current terminal. For anything useful in DevOps work, you need the flags below.

---

### Essential Flags

| Flag | Description |
|---|---|
| `-e` | Show **e**very process running on the system |
| `-f` | **F**ull-format listing — shows PPID, start time, full command |
| `-u [user]` | Show processes owned by a specific **u**ser |
| `-a` | Show processes for all users attached to a terminal |
| `-x` | Include processes not attached to a terminal (daemons/services) |
| `aux` | BSD-style combo: all processes, user-oriented, extended detail (most common) |
| `--sort` | Sort output by a specific field (e.g., `--sort=-%cpu`) |

---

### `ps -e` — List Every Process

```bash
ps -e
```

**Sample output:**
```
    PID TTY          TIME CMD
      1 ?        00:00:02 systemd
    745 ?        00:00:00 cron
    892 ?        00:00:00 sshd
   4821 ?        00:00:01 nginx
   5102 pts/0    00:00:00 bash
```

This lists **every PID** on the system. The `TTY` column shows `?` for processes with no attached terminal (background services/daemons) and a terminal name (`pts/0`) for interactive sessions.

---

### `ps -ef` — Full-Format Listing

```bash
ps -ef
```

**Sample output:**
```
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:00 ?        00:00:02 /sbin/init
root       745     1  0 08:00 ?        00:00:00 /usr/sbin/cron
www-data  4821     1  0 09:15 ?        00:00:01 nginx: master process
yukta     5102  4980  0 10:02 pts/0    00:00:00 -bash
```

**Field breakdown:**

| Column | Meaning |
|---|---|
| `UID` | User who owns the process |
| `PID` | Process ID |
| `PPID` | Parent Process ID |
| `C` | CPU utilization (legacy field, less precise than `top`) |
| `STIME` | Time the process started |
| `TTY` | Terminal associated with the process (`?` = none) |
| `TIME` | Total CPU time consumed since the process started |
| `CMD` | The command that launched the process |

---

### `ps -u [username]` — Filter by User

```bash
ps -u www-data
```

This shows only processes owned by the specified user — extremely useful when troubleshooting "why is this service account using so much CPU" type issues.

---

### `ps aux` — The Most Common Combo

```bash
ps aux
```

**Sample output:**
```
USER     PID  %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
root       1   0.0  0.1  16828  1200 ?    Ss   08:00   0:02 /sbin/init
www-data 4821   1.2  2.1 102400 21500 ?    S    09:15   0:01 nginx: master
yukta    5102   0.0  0.3   8120  3100 pts/0 Ss  10:02   0:00 -bash
```

| Column | Meaning |
|---|---|
| `%CPU` | Percentage of CPU currently used |
| `%MEM` | Percentage of physical RAM used |
| `VSZ` | Virtual memory size (KB) |
| `RSS` | Resident Set Size — actual physical memory used (KB) |
| `STAT` | Process state code (`R`, `S`, `D`, `T`, `Z`, often with modifiers like `+` or `s`) |

---

### Real-World Troubleshooting Patterns

```bash
# Find the top 5 CPU-consuming processes
ps aux --sort=-%cpu | head -6

# Find the top 5 memory-consuming processes
ps aux --sort=-%mem | head -6

# Search for a specific process by name
ps aux | grep nginx

# Get just the PID of a process by name (useful in scripts)
pgrep nginx

# Count how many processes a user is running
ps -u www-data | wc -l

# Check if a specific PID is still running
ps -p 4821
```

**💡 NexusCorp Use Case:** Before restarting a hung service, the on-call engineer runs `ps aux | grep <service_name>` to confirm exactly which PID is stuck and whether multiple zombie instances have piled up.

---

### 🛠️ Practical Exercise 1: `ps`

```bash
# Step 1: List all processes on the system and review the PIDs
ps -e

# Step 2: Identify a few PIDs and note what they correspond to
ps -ef | head -10

# Step 3: View processes started by a specific user
# Replace 'www-data' with any valid user on your system
ps -u www-data

# Step 4: Combine with grep to find a specific service
ps -ef | grep ssh

# Step 5: Find the 3 most CPU-intensive processes right now
ps aux --sort=-%cpu | head -4
```

**Reflection Questions:**
1. Why does plain `ps` (no flags) show so few processes compared to `ps -e`?
2. What's the difference between `%CPU` and `TIME` in `ps aux` output?
3. If a process's `PPID` is `1`, what does that tell you about how it started?

---

## Part 3: `top` — Real-Time Process Monitoring

While `ps` gives you a snapshot, `top` gives you a **live, continuously updating view** of system activity — CPU, memory, load average, and per-process resource usage, refreshed every few seconds.

```bash
top
```

---

### Reading the `top` Header

```
top - 14:32:01 up 3 days,  4:12,  2 users,  load average: 0.45, 0.38, 0.31
Tasks: 187 total,   1 running, 185 sleeping,   0 stopped,   1 zombie
%Cpu(s):  3.2 us,  1.1 sy,  0.0 ni, 95.4 id,  0.2 wa,  0.0 hi,  0.1 si,  0.0 st
MiB Mem :  16038.6 total,   4821.3 free,   6102.8 used,   5114.5 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   8742.1 avail Mem
```

| Field | Meaning |
|---|---|
| `up 3 days, 4:12` | System uptime |
| `load average` | System load over the last 1, 5, and 15 minutes |
| `Tasks` | Total processes and their states (running/sleeping/stopped/zombie) |
| `%Cpu(s)` | CPU breakdown: `us`=user, `sy`=system, `id`=idle, `wa`=I/O wait |
| `Mem` | Total, free, used, and cached memory |
| `Swap` | Swap space usage — high swap use often signals memory pressure |

**Understanding Load Average:** The three numbers represent average system load over 1, 5, and 15 minutes. As a rule of thumb, a load average roughly equal to your CPU core count means the system is fully utilized but not overwhelmed; significantly higher means processes are queued and waiting.

---

### Reading the Process List

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 4821 www-data  20   0  102400  21500   8200 S   1.2   2.1   0:01.45 nginx
 5102 yukta     20   0    8120   3100   2800 S   0.0   0.3   0:00.12 bash
 6203 yukta     20   0  892400 154200  42100 R  45.8   9.7   2:14.88 python3
```

| Column | Meaning |
|---|---|
| `PR` | Priority (lower number = higher priority) |
| `NI` | Nice value (affects scheduling priority, more in Part 4) |
| `VIRT` | Total virtual memory used |
| `RES` | Physical (resident) memory used |
| `SHR` | Shared memory |
| `S` | Process state (same codes as `ps`) |
| `%CPU` / `%MEM` | Live resource consumption |
| `TIME+` | Cumulative CPU time, to hundredths of a second |

---

### Why `top` Matters for DevOps

`top` is often the **first command** run when a server is reported as slow or unresponsive. In seconds, it tells you:
- Is the bottleneck CPU, memory, or I/O wait?
- Which specific process is responsible?
- Is the system under genuine load, or is something stuck/looping?

This makes it indispensable for live troubleshooting — far faster than digging through logs when a server is actively degrading.

---

### Useful Interactive Keys Inside `top`

| Key | Action |
|---|---|
| `h` | Display help — list of all available commands inside `top` |
| `q` | Quit `top` |
| `k` | Kill a process (prompts for PID, then signal) |
| `r` | Renice a process (change its priority) |
| `M` | Sort processes by memory usage |
| `P` | Sort processes by CPU usage (default) |
| `1` | Toggle display of individual CPU cores |
| `u` | Filter to show only one user's processes |
| `c` | Toggle full command path display |
| `Space` | Force an immediate screen refresh |

---

### 🛠️ Practical Exercise 2: `top`

```bash
# Step 1: Open top
top

# Step 2: Once inside, identify the top 3 CPU-consuming processes
#         (they're listed at the top by default, sorted by %CPU)

# Step 3: Press 'h' to view the in-app help menu — browse the available commands

# Step 4: Press 'q' to return to help, then 'q' again to exit help

# Step 5: Try sorting by memory instead of CPU
# Press 'M' while top is running

# Step 6: Exit top
# Press 'q'
```

**Reflection Questions:**
1. What's the difference between `%CPU` in `top` and `load average`?
2. If `%wa` (I/O wait) is unusually high, what kind of bottleneck does that suggest?
3. Why might `top`'s top 3 processes change between two consecutive refreshes?

---

## Part 4: `kill` and `nice` — Managing Process Execution

### `kill` — Sending Signals to Processes

A common misconception: `kill` doesn't just "kill" a process — it **sends a signal** to it. What the process does with that signal depends on the signal type and how the program is written to handle it.

**Syntax:**
```bash
kill [signal] PID
```

---

### Common Signals

| Signal | Number | Name | Behavior |
|---|---|---|---|
| `SIGHUP` | 1 | Hangup | Often used to tell a process to reload its config |
| `SIGINT` | 2 | Interrupt | Same as pressing `Ctrl+C` |
| `SIGKILL` | 9 | Kill | Immediately terminates the process — cannot be ignored or caught |
| `SIGTERM` | 15 | Terminate | Default signal; politely asks the process to shut down gracefully |
| `SIGSTOP` | 19 | Stop | Pauses the process (cannot be ignored) |
| `SIGCONT` | 18 | Continue | Resumes a stopped process |

**View all available signals:**
```bash
kill -l
```

---

### Using `kill`

```bash
# Send the default signal (SIGTERM, 15) — graceful shutdown request
kill 4821

# Force kill — immediate termination, no cleanup
kill -9 4821
kill -SIGKILL 4821

# Send SIGHUP — common way to make a daemon reload its config
kill -1 4821
kill -SIGHUP 4821

# Kill by process name instead of PID
pkill nginx

# Kill all processes matching a pattern, prompting for confirmation
pkill -i python3

# Kill every process owned by a specific user
pkill -u www-data
```

**⚠️ `SIGTERM` vs `SIGKILL` — Why It Matters:**

`SIGTERM` (15) gives a process the chance to clean up — close open files, finish writing to disk, release locks — before exiting. `SIGKILL` (9) gives it **no chance at all**; the kernel terminates it immediately.

**Best practice:** Always try `kill <PID>` (SIGTERM) first. Only escalate to `kill -9` if the process refuses to terminate after a reasonable wait — for example, a hung process unresponsive to graceful shutdown signals.

```bash
# Best practice sequence
kill 4821          # Ask nicely first (SIGTERM)
sleep 5
ps -p 4821          # Check if it's still running
kill -9 4821        # Force it only if necessary
```

**💡 NexusCorp Use Case:** A deployment script always sends `SIGTERM` to the old application instance and waits up to 10 seconds before falling back to `SIGKILL` — this ensures in-flight requests can finish instead of being abruptly cut off.

---

### `nice` and `renice` — Managing Process Priority

Every process has a **niceness value** ranging from `-20` (highest priority) to `19` (lowest priority). The default is `0`. A "nicer" process (higher value) is more willing to yield CPU time to others — hence the name.

```
-20 ──────────────────────────────────► 19
High Priority                  Low Priority
(less nice,                    (more nice,
 grabs more CPU)                yields CPU)
```

---

### Starting a Process with `nice`

```bash
# Start a process with a lower priority (higher niceness, value = 10)
nice -n 10 ./backup_script.sh

# Start a process with the highest possible niceness (lowest priority)
nice -n 19 tar -czf archive.tar.gz /data/

# Start a process with elevated priority (requires sudo for negative values)
sudo nice -n -10 ./critical_task.sh
```

**⚠️ Important:** Only `root` (via `sudo`) can assign **negative** nice values (higher priority than default). Regular users can only lower a process's priority, not raise it — this prevents users from monopolizing CPU resources for their own processes.

---

### Changing Priority of a Running Process with `renice`

```bash
# Change the niceness of an already-running process by PID
renice -n 15 -p 4821

# Change niceness for all processes owned by a user
renice -n 10 -u www-data

# Verify the change
ps -o pid,ni,comm -p 4821
```

---

### Real-World Scenario: Why Priority Matters

Imagine a nightly backup job and a live web server running on the same machine. Without intervention, the backup job competes equally for CPU, potentially slowing down requests to the live site.

```bash
# Run the backup with low priority so it yields to the web server
nice -n 19 ./nightly_backup.sh
```

The backup will still complete — just more slowly, and only using CPU cycles the system isn't actively needing elsewhere.

---

### 🛠️ Practical Exercise 3: `kill` and `nice`

```bash
# --- Part A: kill ---

# Step 1: Start a long-running command in the background
sleep 300 &

# Step 2: Capture its PID
echo "PID of sleep command: $!"

# Or find it manually:
ps aux | grep "sleep 300"

# Step 3: Terminate it gracefully
kill $!

# Step 4: Verify it's gone
ps -p $!
# Should report no such process


# --- Part B: nice ---

# Step 1: Start a CPU-intensive task with low priority
nice -n 19 yes > /dev/null &
echo "Low-priority PID: $!"

# Step 2: Start a second instance with default priority for comparison
yes > /dev/null &
echo "Default-priority PID: $!"

# Step 3: Observe both in top
top
# Look at the NI column and compare %CPU between the two 'yes' processes

# Step 4: Clean up — stop both background processes
kill %1 %2
# Or use pkill
pkill yes
```

**Reflection Questions:**
1. Why might `kill -9` leave behind orphaned resources (like lock files or temp files) that `kill -15` would have cleaned up?
2. If two processes have the same niceness value, how does the scheduler decide which gets more CPU time?
3. In what scenario would you want to `renice` a running process rather than restart it with `nice`?

---

## 🔧 Mini-Project: NexusCorp Incident Response Drill

**Scenario:** A simulated incident — a runaway script is consuming excessive CPU on a NexusCorp staging server. Use everything from today to identify it, manage it, and document the response.

```bash
# Step 1: Simulate the "runaway" process
# (In a real incident this would already be running; here we create one to investigate)
yes > /dev/null &
RUNAWAY_PID=$!
echo "Simulated runaway process started with PID: $RUNAWAY_PID"

# Step 2: Confirm it in the process list
ps -ef | grep yes

# Step 3: Check its live resource usage
top -p $RUNAWAY_PID
# (press q to exit)

# Step 4: Identify top CPU consumers system-wide
ps aux --sort=-%cpu | head -5

# Step 5: Attempt a graceful shutdown first
kill -15 $RUNAWAY_PID
sleep 2

# Step 6: Verify whether it terminated
ps -p $RUNAWAY_PID && echo "Still running — escalating" || echo "Terminated cleanly"

# Step 7: Force kill only if Step 6 shows it's still alive
kill -9 $RUNAWAY_PID 2>/dev/null

# Step 8: Write an incident summary
cat > incident_summary.txt << 'EOF'
NexusCorp Incident Response Summary
====================================
Issue: High CPU process detected on staging server
Detection method: ps aux --sort=-%cpu
Resolution: Sent SIGTERM, escalated to SIGKILL after no response
Recommendation: Investigate root cause of runaway process before next deploy
EOF

cat incident_summary.txt
```

**Bonus Challenges:**
- Start three background processes with different `nice` values and compare their `%CPU` usage in `top` over a few minutes
- Use `pgrep -l` to list all processes matching a pattern along with their names
- Research and explain what `SIGSTOP` and `SIGCONT` are used for in process debugging

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Process** | An instance of a running program, with its own PID, memory space, and resources |
| **Thread** | The smallest unit of CPU execution, subordinate to a process; multiple threads can share one process's memory |
| **PID** | Process ID — a unique numeric identifier assigned to each running process |
| **PPID** | Parent Process ID — the PID of the process that spawned the current one |
| **Daemon** | A background process with no attached terminal, typically providing a system service |
| **Zombie Process** | A terminated process whose exit status hasn't yet been collected by its parent |
| **Load Average** | A measure of system demand over 1, 5, and 15 minutes |
| **Signal** | A message sent to a process to instruct it to perform an action (terminate, reload, pause, etc.) |
| **SIGTERM** | Signal 15 — politely requests a process to terminate, allowing cleanup |
| **SIGKILL** | Signal 9 — forcibly and immediately terminates a process with no cleanup |
| **Niceness** | A value from -20 to 19 controlling a process's scheduling priority; higher = lower priority |
| **Foreground Process** | A process that has control of the terminal and blocks further input until it finishes |
| **Background Process** | A process running independently of the terminal's foreground, started with `&` |

---

## 📚 Resources

- [Linux Performance Monitoring — TecMint](https://www.tecmint.com/linux-performance-monitoring-and-file-system-statistics-reporting-tools/) — practical guide to monitoring tools beyond `top`
- [Red Hat Enterprise Linux Deployment Guide — Monitoring and Automation](https://access.redhat.com/documentation/) — enterprise-level monitoring and automation reference
- [`ps` Man Page](https://man7.org/linux/man-pages/man1/ps.1.html) — full official flag reference
- [`top` Man Page](https://man7.org/linux/man-pages/man1/top.1.html) — full official reference
- [Signal Man Page (signal(7))](https://man7.org/linux/man-pages/man7/signal.7.html) — complete list of Linux signals and their behavior

---

## 🔭 Day 8 Preview: Disk Usage & Storage Management

Tomorrow you shift from CPU and processes to **disk** — how Linux organizes storage, and how to find out what's eating your disk space before a server runs out entirely.

You'll learn:
- `df` — check available disk space across mounted filesystems
- `du` — measure how much space specific files and directories consume
- Understanding mount points, filesystems, and partitions
- Finding and cleaning up large files safely
- How NexusCorp prevents "disk full" incidents from taking down production

You now know how to manage what's *running*. Next, you'll manage what's *stored*.

---

> 💬 *"You can't fix what you can't see. `ps` and `top` are how you see; `kill` and `nice` are how you fix."*
>
> — DevOps Learning by Yukta
