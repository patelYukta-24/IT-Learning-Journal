# Day 10: Basic Shell Scripting & Task Scheduling

> **DevOps Transformation** | Linux Fundamentals Series

---

## 📋 Overview

This is where everything from the past nine days starts to compound.

Every command you've learned — file operations, permissions, process management, networking, text processing — can now be combined into **scripts** that run automatically, repeatedly, and unattended. This is the foundation of automation, and automation is the foundation of DevOps.

Today you write your first Bash scripts, learn the building blocks of the language, and then schedule those scripts to run without you ever typing a command again.

---

## 🗺️ What You'll Learn Today

| Topic | Concepts / Commands | Why It Matters |
|---|---|---|
| Bash scripting basics | shebang, variables, input | Write reusable, parameterized scripts |
| Control flow | `if`, `elif`, `else`, `case` | Make scripts that make decisions |
| Loops | `for`, `while`, `until` | Make scripts that repeat work |
| Functions | `function`, arguments | Organize and reuse script logic |
| Task scheduling | `cron`, `crontab`, `at` | Run scripts automatically on a schedule |

---

## 🏢 NexusCorp Scenario

> NexusCorp's ops team is tired of manually running the same five commands every morning: check disk space, verify services are running, rotate yesterday's logs, ping the database, and send a status report. Each task takes two minutes by hand. Combined, that's ten minutes every single morning — wasted, error-prone, and non-scalable across their growing server fleet.
>
> Today you build the scripts that eliminate that manual work entirely. By the end of Day 10, the ops team runs nothing manually — cron handles it at 6 AM before anyone is even at their desk.

---

## Part 1: Introduction to Shell Scripting

### What Is a Shell Script?

A **shell script** is a plain text file containing a sequence of shell commands. When executed, the shell reads the file line by line and runs each command in order — exactly as if you had typed them yourself.

The difference between typing commands and writing a script:

| Manual Command | Script Equivalent |
|---|---|
| Run once, by hand | Run anytime, by anyone, automatically |
| Easy to forget steps | Documented, repeatable, consistent |
| No record of what ran | Version-controllable, auditable |
| Breaks if you mistype | Tested once, reliable thereafter |

---

### Why Bash?

**Bash** (**B**ourne **A**gain **Sh**ell) is the default shell on the vast majority of Linux distributions and macOS. It is the most widely used shell for scripting in the DevOps world.

Other shells exist and are worth knowing about:

| Shell | Notes |
|---|---|
| **bash** | Default on Ubuntu, Debian, RHEL, CentOS, macOS — the standard choice |
| **sh** | POSIX-compliant shell — more portable, fewer features than bash |
| **zsh** | Feature-rich, popular for interactive use (default on macOS since Catalina) |
| **fish** | User-friendly, interactive shell — not POSIX-compatible |
| **ksh** | Korn shell — common in enterprise Unix environments |

For DevOps scripting, **bash is the right choice**: it is available everywhere, well-documented, and supports all the features you need.

---

### The Shebang Line

Every bash script starts with a **shebang** — a special first line that tells the operating system which interpreter to use to run the file.

```bash
#!/bin/bash
```

The `#!` is the shebang. `/bin/bash` is the path to the bash interpreter. Without this line, the system might use a different shell to run your script, leading to unexpected behavior.

**Alternative shebang (more portable):**
```bash
#!/usr/bin/env bash
```
This looks up `bash` in the user's `PATH` rather than assuming it lives at `/bin/bash` — preferred when writing scripts that might run on multiple operating systems.

---

### Creating and Running Your First Script

```bash
# Step 1: Create the script file
nano hello.sh

# Step 2: Write the script
#!/bin/bash
echo "Hello, World!"

# Step 3: Save and exit (Ctrl+O, Enter, Ctrl+X in nano)

# Step 4: Make it executable
chmod +x hello.sh

# Step 5: Run it
./hello.sh
# Output: Hello, World!
```

**Three ways to run a bash script:**
```bash
./hello.sh          # Execute directly (requires +x permission)
bash hello.sh       # Pass to bash interpreter (no +x needed)
source hello.sh     # Run in current shell (variables persist after script ends)
```

---

## Part 2: Variables

Variables store values that you can reference and reuse throughout your script.

### Defining and Using Variables

```bash
#!/bin/bash

# Define a variable (no spaces around the = sign)
NAME="NexusCorp"
VERSION=2
PI=3.14159

# Reference a variable with $
echo "Welcome to $NAME"
echo "Version: $VERSION"

# Curly braces make the variable boundary explicit (recommended)
echo "Welcome to ${NAME}!"

# Variable in arithmetic
COUNT=5
TOTAL=$((COUNT * 2))
echo "Total: $TOTAL"
```

**⚠️ No spaces around `=`:** `NAME="value"` is correct. `NAME = "value"` will fail — bash interprets it as a command called `NAME` with arguments `=` and `value`.

---

### Special Variables

Bash provides built-in variables you don't need to define:

| Variable | Meaning |
|---|---|
| `$0` | Name of the script itself |
| `$1`, `$2`, `$3`... | Positional arguments passed to the script |
| `$#` | Number of arguments passed |
| `$@` | All arguments as separate words |
| `$*` | All arguments as a single string |
| `$?` | Exit code of the last command (0 = success, non-zero = error) |
| `$$` | PID of the current script |
| `$USER` | Current username |
| `$HOME` | Current user's home directory |
| `$PWD` | Current working directory |
| `$HOSTNAME` | System hostname |

```bash
#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "All arguments: $@"
echo "Argument count: $#"
echo "Running as: $USER on $HOSTNAME"
echo "Working directory: $PWD"
```

**Running with arguments:**
```bash
./script.sh deploy production v2.1
# $1 = deploy
# $2 = production
# $3 = v2.1
```

---

### User Input

```bash
#!/bin/bash
echo "Enter your name:"
read USER_NAME
echo "Hello, $USER_NAME!"

# Read with a prompt on the same line (-p)
read -p "Enter environment (staging/production): " ENV

# Read without echoing input to screen (-s, for passwords)
read -s -p "Enter password: " PASSWORD
echo ""  # Newline after hidden input
echo "Connecting to $ENV environment..."
```

---

### Command Substitution

Capture the output of a command and store it in a variable:

```bash
#!/bin/bash
# Backtick syntax (older)
DATE=`date +%Y-%m-%d`

# $() syntax (preferred — readable and nestable)
DATE=$(date +%Y-%m-%d)
UPTIME=$(uptime -p)
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')

echo "Date: $DATE"
echo "Uptime: $UPTIME"
echo "Disk usage on /: $DISK_USAGE"
```

---

## Part 3: Conditionals

### `if` / `elif` / `else`

```bash
#!/bin/bash

NUMBER=15

if [ $NUMBER -gt 10 ]; then
    echo "$NUMBER is greater than 10"
elif [ $NUMBER -eq 10 ]; then
    echo "$NUMBER equals 10"
else
    echo "$NUMBER is less than 10"
fi
```

**Spacing matters:** Always include spaces inside `[ ]`. `[ $X -gt 5 ]` is correct. `[$X -gt 5]` will fail.

---

### Comparison Operators

**Numeric comparisons (use with `[ ]`):**

| Operator | Meaning | Example |
|---|---|---|
| `-eq` | Equal to | `[ $A -eq $B ]` |
| `-ne` | Not equal to | `[ $A -ne $B ]` |
| `-lt` | Less than | `[ $A -lt 10 ]` |
| `-le` | Less than or equal | `[ $A -le 10 ]` |
| `-gt` | Greater than | `[ $A -gt 0 ]` |
| `-ge` | Greater than or equal | `[ $A -ge 1 ]` |

**String comparisons:**

| Operator | Meaning | Example |
|---|---|---|
| `=` or `==` | Equal | `[ "$A" = "$B" ]` |
| `!=` | Not equal | `[ "$A" != "$B" ]` |
| `-z` | String is empty | `[ -z "$A" ]` |
| `-n` | String is not empty | `[ -n "$A" ]` |

**File and directory tests:**

| Operator | Meaning | Example |
|---|---|---|
| `-f` | File exists and is a regular file | `[ -f "/etc/hosts" ]` |
| `-d` | Directory exists | `[ -d "/var/log" ]` |
| `-e` | File or directory exists | `[ -e "/tmp/lock" ]` |
| `-r` | File is readable | `[ -r "config.conf" ]` |
| `-w` | File is writable | `[ -w "config.conf" ]` |
| `-x` | File is executable | `[ -x "script.sh" ]` |
| `-s` | File is non-empty | `[ -s "output.log" ]` |

**Logical operators:**

```bash
# AND — both conditions must be true
if [ $AGE -ge 18 ] && [ $AGE -le 65 ]; then

# OR — at least one condition must be true
if [ "$ENV" = "staging" ] || [ "$ENV" = "production" ]; then

# NOT
if ! [ -f "/tmp/lock" ]; then
```

---

### Real-World Conditional Example

```bash
#!/bin/bash

DISK_THRESHOLD=80
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')

if [ "$DISK_USAGE" -ge "$DISK_THRESHOLD" ]; then
    echo "WARNING: Disk usage is at ${DISK_USAGE}% — threshold is ${DISK_THRESHOLD}%"
    echo "Consider cleaning up /var/log or /tmp"
else
    echo "OK: Disk usage is at ${DISK_USAGE}%"
fi
```

---

### `case` Statement

`case` is cleaner than a long `if/elif` chain when checking one variable against multiple values:

```bash
#!/bin/bash
read -p "Enter environment (dev/staging/production): " ENV

case $ENV in
    dev)
        echo "Connecting to development server: dev.nexuscorp.internal"
        ;;
    staging)
        echo "Connecting to staging server: staging.nexuscorp.internal"
        ;;
    production)
        echo "Connecting to PRODUCTION: prod.nexuscorp.com"
        echo "WARNING: Changes will affect live users."
        ;;
    *)
        echo "Unknown environment: $ENV"
        echo "Valid options: dev, staging, production"
        exit 1
        ;;
esac
```

---

## Part 4: Loops

### `for` Loop

```bash
#!/bin/bash

# Loop over a range of numbers
for i in {1..10}; do
    echo "Number: $i"
done

# Loop with a step (every 2)
for i in {0..20..2}; do
    echo "$i"
done

# C-style for loop
for (( i=1; i<=5; i++ )); do
    echo "Step $i"
done

# Loop over a list of values
for SERVER in web01 web02 db01 cache01; do
    echo "Checking: $SERVER"
    ping -c 1 -q "$SERVER" > /dev/null 2>&1 && echo "  $SERVER: UP" || echo "  $SERVER: DOWN"
done

# Loop over files in a directory
for FILE in /var/log/*.log; do
    echo "Processing: $FILE"
    wc -l "$FILE"
done

# Loop over command output
for USER in $(cat /etc/passwd | awk -F: '{print $1}'); do
    echo "User: $USER"
done
```

---

### `while` Loop

```bash
#!/bin/bash

# Basic while loop
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Read a file line by line
while IFS= read -r LINE; do
    echo "Processing: $LINE"
done < /etc/hosts

# Infinite loop with a break condition
while true; do
    LOAD=$(uptime | awk '{print $NF}')
    echo "Current load: $LOAD"
    sleep 5
    # In a real script, add a break condition here
    break
done
```

---

### `until` Loop

`until` is the opposite of `while` — it runs until the condition becomes true:

```bash
#!/bin/bash

RETRIES=0
MAX_RETRIES=5

until ping -c 1 -q db.nexuscorp.internal > /dev/null 2>&1; do
    RETRIES=$((RETRIES + 1))
    echo "Database unreachable — retry $RETRIES of $MAX_RETRIES"
    if [ $RETRIES -ge $MAX_RETRIES ]; then
        echo "ERROR: Could not reach database after $MAX_RETRIES attempts."
        exit 1
    fi
    sleep 10
done

echo "Database is reachable. Proceeding with deployment."
```

---

## Part 5: Functions

Functions let you define reusable blocks of logic and call them by name.

```bash
#!/bin/bash

# Define a function
greet() {
    echo "Hello, $1!"   # $1 is the first argument to the function
}

# Call the function
greet "NexusCorp"
greet "DevOps Team"


# Function with return value (exit code)
is_running() {
    SERVICE=$1
    systemctl is-active --quiet "$SERVICE"
    return $?   # Returns 0 if running, non-zero if not
}

# Use the function in a conditional
if is_running nginx; then
    echo "nginx is running"
else
    echo "nginx is NOT running"
fi


# Function with local variables
check_disk() {
    local PATH_TO_CHECK=$1
    local THRESHOLD=$2
    local USAGE=$(df "$PATH_TO_CHECK" | awk 'NR==2 {print $5}' | sed 's/%//')

    if [ "$USAGE" -ge "$THRESHOLD" ]; then
        echo "ALERT: $PATH_TO_CHECK is at ${USAGE}% (threshold: ${THRESHOLD}%)"
        return 1
    else
        echo "OK: $PATH_TO_CHECK is at ${USAGE}%"
        return 0
    fi
}

check_disk "/" 80
check_disk "/var" 70
```

---

## Part 6: Script Best Practices

```bash
#!/bin/bash
# ============================================================
# Script: nexus_health_check.sh
# Description: Basic server health check for NexusCorp
# Usage: ./nexus_health_check.sh [environment]
# ============================================================

# Exit immediately if any command fails
set -e

# Treat unset variables as errors
set -u

# Fail if any command in a pipe fails (not just the last)
set -o pipefail

# Define constants at the top
readonly LOG_FILE="/var/log/nexus_health.log"
readonly DISK_THRESHOLD=80
readonly ENVIRONMENT=${1:-"staging"}   # Default to staging if no arg

# Logging function
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

log "Starting health check for: $ENVIRONMENT"
log "Running as: $USER on $HOSTNAME"
```

**Script best practices summary:**

| Practice | Why |
|---|---|
| `set -e` | Stop on first error — don't silently continue after a failure |
| `set -u` | Catch typos in variable names — unset variables error out |
| `set -o pipefail` | Catch failures inside pipes, not just the final command |
| Quote all variables (`"$VAR"`) | Prevents word splitting on values with spaces |
| Use `local` in functions | Prevents variables from leaking into the global scope |
| Add comments and a usage block | Makes scripts maintainable by others (and future you) |
| Always test on non-production first | Scripts that work in staging can still break in production |

---

## 🛠️ Practical Exercises: Shell Scripting

### Exercise 1: Hello World and Variables

```bash
#!/bin/bash
# hello.sh — Exercise 1

# Part A: Basic Hello World
echo "Hello, World!"

# Part B: Use a variable for the greeting
GREETING="Hello"
TARGET="World"
echo "${GREETING}, ${TARGET}!"

# Part C: Make it take a command-line argument
NAME=${1:-"World"}   # Default to "World" if no argument given
echo "Hello, $NAME!"
```

```bash
chmod +x hello.sh
./hello.sh
./hello.sh NexusCorp
```

---

### Exercise 2: Loop Through Numbers 1 to 10

```bash
#!/bin/bash
# count.sh — Exercise 2

echo "Counting from 1 to 10:"
for i in {1..10}; do
    echo "  $i"
done

echo ""
echo "Even numbers from 1 to 10:"
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        echo "  $i"
    fi
done
```

---

### Exercise 3: Even or Odd Checker

```bash
#!/bin/bash
# even_odd.sh — Exercise 3

read -p "Enter a number: " NUMBER

# Validate input is a number
if ! [[ "$NUMBER" =~ ^-?[0-9]+$ ]]; then
    echo "Error: '$NUMBER' is not a valid integer."
    exit 1
fi

REMAINDER=$((NUMBER % 2))

if [ $REMAINDER -eq 0 ]; then
    echo "$NUMBER is EVEN"
else
    echo "$NUMBER is ODD"
fi
```

---

### Exercise 4: NexusCorp Server Health Check Script

```bash
#!/bin/bash
# nexus_health.sh — a complete practical script combining everything

set -uo pipefail

# ── Configuration ────────────────────────────────────
DISK_THRESHOLD=80
SERVICES=("ssh" "cron")    # Adjust based on what's running on your system
LOG_FILE="/tmp/nexus_health_$(date +%Y%m%d).log"

# ── Helper Functions ──────────────────────────────────
log() {
    echo "[$(date '+%H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

check_disk() {
    local USAGE
    USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
    if [ "$USAGE" -ge "$DISK_THRESHOLD" ]; then
        log "WARNING: Disk usage at ${USAGE}% (threshold: ${DISK_THRESHOLD}%)"
    else
        log "OK: Disk usage at ${USAGE}%"
    fi
}

check_services() {
    for SERVICE in "${SERVICES[@]}"; do
        if systemctl is-active --quiet "$SERVICE" 2>/dev/null; then
            log "OK: $SERVICE is running"
        else
            log "WARNING: $SERVICE is NOT running"
        fi
    done
}

check_memory() {
    local FREE_MB
    FREE_MB=$(free -m | awk 'NR==2 {print $4}')
    log "Free memory: ${FREE_MB}MB"
}

# ── Main ──────────────────────────────────────────────
log "=== NexusCorp Health Check Started ==="
log "Host: $HOSTNAME | User: $USER"
log ""

log "--- Disk Check ---"
check_disk

log ""
log "--- Service Check ---"
check_services

log ""
log "--- Memory Check ---"
check_memory

log ""
log "=== Health Check Complete. Log: $LOG_FILE ==="
```

---

## Part 7: Task Scheduling with `cron` and `at`

### What Is Task Scheduling?

Even the best script is only as useful as how reliably it runs. **Task scheduling** lets you define when a script executes — whether that's every minute, every night at midnight, or once at a specific time in the future.

In DevOps, scheduled tasks handle:
- Nightly backups
- Log rotation and cleanup
- Health checks and alerting
- Certificate renewal (e.g., Let's Encrypt)
- Database maintenance jobs
- Periodic report generation

---

### `cron` — Recurring Scheduled Tasks

`cron` is a daemon that runs continuously in the background, checking every minute whether any scheduled job needs to run. Jobs are defined in a **crontab** (cron table) file.

**Accessing your crontab:**
```bash
# Open your crontab for editing
crontab -e

# List your current cron jobs
crontab -l

# Remove all your cron jobs (careful!)
crontab -r

# Edit another user's crontab (requires root)
sudo crontab -u www-data -e
```

---

### Crontab Syntax

Each line in a crontab defines one scheduled job:

```
┌─────────────── Minute        (0–59)
│ ┌───────────── Hour          (0–23)
│ │ ┌─────────── Day of Month  (1–31)
│ │ │ ┌───────── Month         (1–12 or JAN–DEC)
│ │ │ │ ┌─────── Day of Week   (0–7, 0 and 7 = Sunday, or SUN–SAT)
│ │ │ │ │
* * * * *  command_to_run
```

**Special characters:**

| Character | Meaning | Example |
|---|---|---|
| `*` | Any / every value | `* * * * *` = every minute |
| `,` | List of values | `0,30 * * * *` = at :00 and :30 |
| `-` | Range of values | `9-17 * * * *` = every hour from 9 AM to 5 PM |
| `/` | Step / interval | `*/15 * * * *` = every 15 minutes |

---

### Common Cron Expressions

| Expression | Meaning |
|---|---|
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour, on the hour |
| `0 6 * * *` | Every day at 6:00 AM |
| `0 6 * * 1` | Every Monday at 6:00 AM |
| `0 6 * * 1-5` | Every weekday at 6:00 AM |
| `0 0 * * 0` | Every Sunday at midnight |
| `0 2 1 * *` | First day of every month at 2:00 AM |
| `0 2 1 1 *` | January 1st at 2:00 AM (New Year's Day) |
| `30 23 * * *` | Every day at 11:30 PM |
| `0 */6 * * *` | Every 6 hours |
| `@reboot` | Once, when the system boots |
| `@daily` | Once a day (same as `0 0 * * *`) |
| `@weekly` | Once a week (same as `0 0 * * 0`) |
| `@monthly` | Once a month (same as `0 0 1 * *`) |

**📎 Resource:** [crontab.guru](https://crontab.guru) — paste any cron expression to see a plain-English description of when it runs.

---

### Writing Cron Jobs

```bash
# Open crontab
crontab -e

# Example entries:

# Run health check every minute
* * * * * /home/devops/scripts/nexus_health.sh >> /var/log/nexus_health_cron.log 2>&1

# Run backup every day at 2 AM
0 2 * * * /home/devops/scripts/backup.sh >> /var/log/backup.log 2>&1

# Run system update every Sunday at 3 AM
0 3 * * 0 sudo apt update && sudo apt upgrade -y >> /var/log/weekly_update.log 2>&1

# Clear temp files every day at midnight
0 0 * * * find /tmp -type f -mtime +7 -delete

# Run a script on every system reboot
@reboot /home/devops/scripts/startup_checks.sh
```

**The `2>&1` pattern:** Redirects both standard output and standard error to the log file. Without this, errors from your cron job disappear silently — a common source of "my cron job isn't working" confusion.

---

### System-Wide Cron Directories

Beyond user crontabs, the system provides drop-in directories for system-level scheduled tasks:

```
/etc/cron.d/         ← Individual cron files (like a crontab, with username field)
/etc/cron.hourly/    ← Scripts dropped here run every hour
/etc/cron.daily/     ← Scripts dropped here run every day
/etc/cron.weekly/    ← Scripts dropped here run every week
/etc/cron.monthly/   ← Scripts dropped here run every month
```

```bash
# Place a script in cron.daily to run it every day without editing crontab
sudo cp nexus_health.sh /etc/cron.daily/nexus_health
sudo chmod +x /etc/cron.daily/nexus_health
```

---

### Verifying and Debugging Cron Jobs

```bash
# Check if cron daemon is running
systemctl status cron       # Debian/Ubuntu
systemctl status crond      # RHEL/CentOS

# View system cron logs
grep CRON /var/log/syslog | tail -20        # Debian/Ubuntu
grep CRON /var/log/cron | tail -20          # RHEL/CentOS
journalctl -u cron --since "1 hour ago"     # systemd-based

# Common cron job debugging checklist:
# 1. Does the script have a full path? Cron has a minimal PATH.
# 2. Is the script executable? (chmod +x)
# 3. Is 2>&1 capturing errors in the log?
# 4. Does the script work when run manually as the same user?
# 5. Are any required env variables set? (Cron doesn't load .bashrc)
```

**⚠️ Cron PATH Warning:** Cron runs with a very minimal `PATH` — usually just `/usr/bin:/bin`. If your script uses commands like `python3`, `npm`, or custom tools, always use their full paths in the script or crontab:

```bash
# Instead of:
python3 /scripts/check.py

# Use:
/usr/bin/python3 /scripts/check.py
```

---

### `at` — One-Time Scheduled Tasks

While `cron` is for recurring jobs, `at` schedules a command or script to run **once** at a specified future time — without needing to set up and remove a cron entry.

**Install if needed:**
```bash
sudo apt install at       # Debian/Ubuntu
sudo yum install at       # RHEL/CentOS
sudo systemctl enable --now atd
```

**Scheduling with `at`:**
```bash
# Run a command at a specific time
at 14:30
> echo "Deployment starting" >> /tmp/deploy.log
> /home/devops/scripts/deploy.sh
> Ctrl+D          # Press Ctrl+D to save and schedule

# Shorthand: pipe a command directly to at
echo "/home/devops/scripts/deploy.sh" | at 14:30

# Run at a time relative to now
at now + 10 minutes
at now + 2 hours
at now + 1 day

# Run at a specific date and time
at 09:00 AM Jul 20
at midnight tomorrow
at noon next Monday

# Run a script file at a specific time
at -f /home/devops/scripts/report.sh 23:59

# List all pending at jobs
atq
# or
at -l

# Remove a pending job (replace 3 with the job ID from atq)
atrm 3
at -d 3
```

**Sample `at` session:**
```
$ at now + 5 minutes
warning: commands will be executed using /bin/sh
at> echo "5 minutes have passed" >> /tmp/at_test.log
at> Ctrl+D
job 4 at Wed Jun 10 14:37:00 2024
```

**`at` vs `cron` — when to use each:**

| Need | Use |
|---|---|
| Run a script every day at 6 AM | `cron` |
| Run a one-time deployment in 2 hours | `at` |
| Clear logs every Sunday at midnight | `cron` |
| Restart a service after a maintenance window | `at` |
| Generate a report every first of the month | `cron` |
| Send a one-time reminder notification | `at` |

---

## 🛠️ Practical Exercises: Task Scheduling

```bash
# Exercise 1: Schedule a script to run every minute with cron

# First, create a simple test script
cat > /tmp/cron_test.sh << 'EOF'
#!/bin/bash
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Cron job executed on $HOSTNAME" >> /tmp/cron_test.log
EOF
chmod +x /tmp/cron_test.sh

# Open crontab and add the job
crontab -e
# Add this line:
# * * * * * /tmp/cron_test.sh

# Wait 2 minutes, then check the log
cat /tmp/cron_test.log

# Remove the test job afterward
crontab -e   # Delete the line you added


# Exercise 2: Schedule a one-time script with at

echo "bash /tmp/cron_test.sh" | at now + 2 minutes

# Check it was scheduled
atq

# After it runs, verify the log
cat /tmp/cron_test.log


# Exercise 3: Create a cron job that runs daily at a specific time

crontab -e
# Add a job to run the health check every day at 7 AM:
# 0 7 * * * /tmp/cron_test.sh >> /tmp/daily_health.log 2>&1

# Verify the crontab
crontab -l
```

---

## 🔧 Mini-Project: NexusCorp Morning Automation Suite

**Scenario:** Replace the ops team's manual morning routine with a fully automated script that runs at 6 AM every weekday.

```bash
#!/bin/bash
# morning_suite.sh — NexusCorp Daily Automation
# Scheduled via cron: 0 6 * * 1-5

set -uo pipefail

# ── Configuration ────────────────────────────────────────────
readonly REPORT_DIR="/tmp/nexus_reports"
readonly DATE=$(date +%Y-%m-%d)
readonly REPORT_FILE="${REPORT_DIR}/morning_report_${DATE}.txt"
readonly DISK_THRESHOLD=80

mkdir -p "$REPORT_DIR"

# ── Helper ────────────────────────────────────────────────────
log() {
    echo "[$(date '+%H:%M:%S')] $1" | tee -a "$REPORT_FILE"
}

separator() {
    echo "────────────────────────────────────" | tee -a "$REPORT_FILE"
}

# ── Header ────────────────────────────────────────────────────
echo "" | tee "$REPORT_FILE"
log "NexusCorp Morning Report — $DATE"
log "Host: $HOSTNAME | Generated by: $USER"
separator

# ── 1. Disk Check ─────────────────────────────────────────────
log "DISK USAGE"
df -h | grep -v "tmpfs" | tee -a "$REPORT_FILE"
DISK_PERCENT=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
if [ "$DISK_PERCENT" -ge "$DISK_THRESHOLD" ]; then
    log "*** WARNING: Root disk at ${DISK_PERCENT}% ***"
fi
separator

# ── 2. Memory Check ──────────────────────────────────────────
log "MEMORY USAGE"
free -h | tee -a "$REPORT_FILE"
separator

# ── 3. Top CPU Processes ─────────────────────────────────────
log "TOP 5 CPU-CONSUMING PROCESSES"
ps aux --sort=-%cpu | head -6 | tee -a "$REPORT_FILE"
separator

# ── 4. Connectivity Check ────────────────────────────────────
log "CONNECTIVITY CHECKS"
TARGETS=("8.8.8.8" "google.com")
for TARGET in "${TARGETS[@]}"; do
    if ping -c 1 -q "$TARGET" > /dev/null 2>&1; then
        log "OK:   $TARGET is reachable"
    else
        log "FAIL: $TARGET is NOT reachable"
    fi
done
separator

# ── 5. Listening Ports ───────────────────────────────────────
log "LISTENING PORTS"
ss -tulnp 2>/dev/null | tee -a "$REPORT_FILE"
separator

# ── Footer ───────────────────────────────────────────────────
log "Report complete: $REPORT_FILE"
echo ""

# ── Display the report ───────────────────────────────────────
cat "$REPORT_FILE"
```

**Schedule it:**
```bash
chmod +x morning_suite.sh

crontab -e
# Add:
# 0 6 * * 1-5 /home/devops/scripts/morning_suite.sh >> /var/log/nexus_morning.log 2>&1
```

**Bonus Challenges:**
- Add a function that checks if specific services are running and logs their status
- Add a loop that checks connectivity to a list of servers read from a file (`servers.txt`)
- Extend the report to include uptime (`uptime -p`) and the last 5 lines of the system log (`journalctl -n 5`)

---

## 🎯 Supplementary Challenges

### Challenge 1: Pomodoro Timer for the Terminal

A Pomodoro timer helps you work in focused 25-minute sessions. Build one in bash:

```bash
#!/bin/bash
# pomodoro.sh — Terminal Pomodoro Timer

WORK_MINS=25
BREAK_MINS=5

countdown() {
    local MINS=$1
    local LABEL=$2
    local TOTAL=$((MINS * 60))

    echo ""
    echo "⏱  $LABEL — ${MINS} minutes"
    echo "────────────────────────"

    while [ $TOTAL -gt 0 ]; do
        local M=$((TOTAL / 60))
        local S=$((TOTAL % 60))
        printf "\r  Remaining: %02d:%02d " $M $S
        sleep 1
        TOTAL=$((TOTAL - 1))
    done

    echo ""
    echo "✓ $LABEL complete!"
}

ROUND=1
while true; do
    echo ""
    echo "=== Pomodoro Round $ROUND ==="
    countdown $WORK_MINS "Work Session"

    # Optional: system bell
    echo -e "\007"

    read -p "  Start break? (y/n): " ANSWER
    if [ "$ANSWER" = "y" ]; then
        countdown $BREAK_MINS "Break"
    fi

    ROUND=$((ROUND + 1))
    read -p "  Start next round? (y/n): " NEXT
    [ "$NEXT" != "y" ] && break
done

echo ""
echo "Session complete. Rounds completed: $((ROUND - 1))"
```

📎 More inspiration: [Pomodoro Timer on dev.to](https://dev.to)

---

### Challenge 2: BashBlaze — 7 Days of Bash Scripting

BashBlaze is a structured 7-day scripting challenge that builds from beginner to intermediate bash skills, one problem per day.

Topics covered across the 7 days include: file backup systems, process monitoring, log analyzers, user account management, and more.

📎 Repository: [BashBlaze on GitHub](https://github.com/PristineRamar/BashBlaze-7-Days-of-Bash-Scripting-Challenge)

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Shell Script** | A text file containing a sequence of shell commands, executed by the shell interpreter |
| **Bash** | Bourne Again Shell — the most widely used shell for Linux scripting |
| **Shebang (`#!`)** | The first line of a script specifying which interpreter to use (e.g., `#!/bin/bash`) |
| **Variable** | A named storage location holding a value used within a script |
| **Positional Parameter** | Arguments passed to a script (`$1`, `$2`, etc.) |
| **Conditional** | A control structure (`if`/`elif`/`else`) that runs code only when a condition is true |
| **Loop** | A control structure (`for`, `while`, `until`) that repeats code until a condition changes |
| **Function** | A named, reusable block of code within a script |
| **Exit Code** | A numeric value returned by a command — `0` means success, non-zero means failure |
| **cron** | A daemon that runs scheduled tasks (cron jobs) at defined intervals |
| **crontab** | The file defining a user's cron jobs; also the command used to manage it |
| **cron expression** | The five-field time specification in a crontab line (minute, hour, day, month, weekday) |
| **at** | A command for scheduling a one-time task to run at a specific future time |
| **atd** | The daemon that runs jobs scheduled with `at` |
| **`set -e`** | Bash option that causes the script to exit immediately on any command error |
| **`set -u`** | Bash option that treats unset variables as an error |
| **Pipe (`\|`)** | Passes the output of one command as input to the next |
| **Redirection (`>`, `>>`)** | Sends command output to a file instead of the terminal |
| **`2>&1`** | Redirects stderr (file descriptor 2) to the same destination as stdout (1) |
| **Command Substitution** | Captures command output into a variable using `$(command)` syntax |

---

## 📚 Resources

- [A Beginner's Guide to Cron Jobs — OSTechNix](https://www.ostechnix.com/a-beginners-guide-to-cron-jobs/) — practical cron walkthrough
- [crontab.guru](https://crontab.guru) — interactive cron expression builder and explainer
- [Bash Cheat Sheet — devhints.io](https://devhints.io/bash) — quick reference for bash syntax
- [Learn Bash in Y Minutes](https://learnxinyminutes.com/docs/bash/) — rapid bash syntax tour
- [Linux Shell Scripting Playlist — YouTube](https://www.youtube.com/results?search_query=linux+shell+scripting+tutorial) — video tutorials for visual learners
- [jq Official Site](https://jqlang.github.io/jq/) — command-line JSON processor, essential for scripting with APIs and config files
- [BashBlaze — 7 Days of Bash Scripting](https://github.com/PristineRamar/BashBlaze-7-Days-of-Bash-Scripting-Challenge) — structured scripting challenge
- [Pomodoro Timer on dev.to](https://dev.to/search?q=pomodoro+bash) — community bash Pomodoro implementations

---

## 🏁 Linux Fundamentals — Series Complete

Congratulations on completing the **NexusCorp Linux Fundamentals Series**.

Here's what you've built across 10 days:

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

**What comes next:** AWS Cloud Fundamentals, Git & GitHub, and Shell Scripting deep-dives — building on this Linux foundation toward a complete DevOps skill set.

---

> 💬 *"A script that runs itself is worth a hundred commands you'll forget to type."*
>
> — DevOps Learning by Yukta
