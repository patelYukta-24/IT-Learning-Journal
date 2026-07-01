# Day 9: Networking Basics & Text Processing Tools

> **DevOps Transformation** | Linux Fundamentals Series

---

## 📋 Overview

Today covers two groups of tools that show up constantly in real DevOps work — often in the same command.

**Networking commands** let you verify connectivity, inspect interfaces, and diagnose communication failures between servers. **Text processing tools** let you slice through logs, config files, and data streams to find exactly what you need. In practice, you'll frequently pipe the output of a networking command directly into `grep`, `awk`, or `sed` — so learning them together makes sense.

---

## 🗺️ What You'll Learn Today

| Topic | Commands | Why It Matters |
|---|---|---|
| Network reachability | `ping` | Verify a host is alive and measure latency |
| Interface configuration | `ifconfig`, `ip` | View IPs, MAC addresses, interface state |
| Network connections | `netstat`, `ss` | See open ports, active connections, listeners |
| Pattern searching | `grep` | Find matching lines in files or command output |
| Field extraction | `awk` | Process structured text, extract columns |
| Stream editing | `sed` | Find and replace text in files or streams |

---

## 🏢 NexusCorp Scenario

> NexusCorp's application deployment just completed, but the app server can't reach the database. The on-call engineer needs to verify network connectivity, check which ports are listening, inspect interface configuration, and dig through log files — all without leaving the terminal. Every tool covered today plays a direct role in getting the system back up.

---

## Part 1: Networking Basics

### Why Networking Matters in DevOps

Modern infrastructure is distributed. Your application server, database, cache layer, and load balancer are almost certainly different machines — often in different availability zones or regions. Every interaction between them is a network call.

As a DevOps engineer, you need to be able to answer:
- Is that server reachable from here?
- What IP address is this machine using?
- Is the service actually listening on the port it's supposed to?
- Are there unexpected connections open on this host?

The three tools below give you the answers.

---

### `ping` — Test Host Reachability

`ping` sends ICMP echo request packets to a target host and waits for replies. It is the first tool you reach for when troubleshooting connectivity — answering the most fundamental question: *can this machine talk to that one?*

**Syntax:**
```bash
ping [options] hostname_or_IP
```

**Basic usage:**
```bash
# Ping a domain (runs indefinitely — use Ctrl+C to stop)
ping google.com

# Ping an IP address directly
ping 8.8.8.8

# Send a fixed number of packets (-c = count)
ping -c 4 google.com

# Set interval between packets in seconds (-i)
ping -c 4 -i 0.5 google.com

# Set packet size in bytes (-s)
ping -c 4 -s 1024 google.com

# Set timeout in seconds (-W)
ping -c 4 -W 2 google.com

# Quiet output — only show summary (-q)
ping -c 4 -q google.com
```

**Sample output:**
```
PING google.com (142.250.80.46) 56(84) bytes of data.
64 bytes from 142.250.80.46: icmp_seq=1 ttl=116 time=12.4 ms
64 bytes from 142.250.80.46: icmp_seq=2 ttl=116 time=11.9 ms
64 bytes from 142.250.80.46: icmp_seq=3 ttl=116 time=12.1 ms
64 bytes from 142.250.80.46: icmp_seq=4 ttl=116 time=12.3 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 11.9/12.2/12.4/0.2 ms
```

**Reading the output:**

| Field | Meaning |
|---|---|
| `icmp_seq` | Sequence number — gaps indicate dropped packets |
| `ttl` | Time to Live — number of hops remaining; low values suggest many hops |
| `time` | Round-trip latency in milliseconds |
| `packet loss` | Percentage of packets that never received a reply |
| `rtt min/avg/max` | Minimum, average, and maximum round-trip times |

**What ping results tell you:**

| Result | Interpretation |
|---|---|
| Replies with low latency | Host is reachable and responsive |
| Replies with high latency | Network congestion or many hops between hosts |
| High packet loss | Unstable network path or overloaded host |
| `Request timeout` | Host is unreachable, or ICMP is blocked by a firewall |
| `Name or service not known` | DNS resolution failed — check DNS, not the host |

**Useful flags reference:**

| Flag | Description | Example |
|---|---|---|
| `-c N` | Send exactly N packets | `ping -c 5 host` |
| `-i N` | Wait N seconds between packets | `ping -i 2 host` |
| `-s N` | Set packet size to N bytes | `ping -s 512 host` |
| `-W N` | Timeout of N seconds per packet | `ping -W 3 host` |
| `-q` | Quiet — only print summary | `ping -q -c 10 host` |
| `-4` / `-6` | Force IPv4 or IPv6 | `ping -4 host` |

**💡 NexusCorp Use Case:** Before investigating an application error, the engineer always pings the database server first. If ping fails, the problem is network-level — no need to dig into application logs yet.

---

### `ifconfig` — Display and Configure Network Interfaces

`ifconfig` (**i**nterface **c**on**fig**) displays the configuration of all active network interfaces — IP addresses, MAC addresses, packet statistics, and interface state. It is the quickest way to answer "what IP does this machine have?"

> **Note:** `ifconfig` is part of the older `net-tools` package and may not be installed by default on modern systems. The modern equivalent is `ip addr` (covered below). Both are worth knowing.

**Basic usage:**
```bash
# Display all active interfaces
ifconfig

# Display a specific interface
ifconfig eth0

# Display all interfaces including inactive ones (-a)
ifconfig -a

# Install if not present (Ubuntu/Debian)
sudo apt install net-tools
```

**Sample output:**
```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      inet 192.168.1.45  netmask 255.255.255.0  broadcast 192.168.1.255
      inet6 fe80::215:5dff:fe01:2345  prefixlen 64  scopeid 0x20<link>
      ether 00:15:5d:01:23:45  txqueuelen 1000  (Ethernet)
      RX packets 124821  bytes 98432100 (93.8 MiB)
      TX packets 84312   bytes 12043210 (11.4 MiB)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
      inet 127.0.0.1  netmask 255.0.0.0
      inet6 ::1  prefixlen 128  scopeid 0x10<host>
```

**Reading the output:**

| Field | Meaning |
|---|---|
| `eth0` / `ens3` / `enp0s3` | Interface name (naming varies by system) |
| `flags=...UP...RUNNING` | Interface is active and connected |
| `inet` | IPv4 address assigned to this interface |
| `netmask` | Subnet mask defining the network range |
| `broadcast` | Broadcast address for the subnet |
| `inet6` | IPv6 address |
| `ether` | MAC (hardware) address |
| `RX packets` | Packets received — check for errors and drops |
| `TX packets` | Packets transmitted — check for errors and drops |
| `lo` | Loopback interface (`127.0.0.1`) — local-only traffic |

---

### `ip` — The Modern Alternative to `ifconfig`

On modern Linux systems, `ip` is preferred over `ifconfig`. It is more powerful and is always available:

```bash
# Show all interfaces and their IP addresses
ip addr show
ip a          # shorthand

# Show a specific interface
ip addr show eth0

# Show routing table
ip route show
ip r          # shorthand

# Show network interface statistics
ip -s link show eth0

# Bring an interface up or down (requires sudo)
sudo ip link set eth0 up
sudo ip link set eth0 down

# Add a temporary IP address to an interface
sudo ip addr add 192.168.1.100/24 dev eth0

# Show ARP cache (MAC address to IP mappings)
ip neigh show
```

**💡 NexusCorp Use Case:** On a newly provisioned cloud instance, `ip addr show` confirms the instance has received the correct private IP from the VPC's DHCP server before any further configuration.

---

### `netstat` — Network Connections, Ports, and Statistics

`netstat` prints active network connections, listening ports, routing tables, and interface statistics. It answers: *what is this machine currently connected to, and what ports is it listening on?*

> **Note:** Like `ifconfig`, `netstat` is part of `net-tools` and is being replaced by `ss` on modern systems. Both are documented here.

**Install if needed:**
```bash
sudo apt install net-tools     # Debian/Ubuntu
sudo yum install net-tools     # RHEL/CentOS
```

**Common usage:**
```bash
# List all active connections
netstat

# Show all listening and established connections
netstat -a

# Show only listening ports
netstat -l

# Show TCP connections only
netstat -t

# Show UDP connections only
netstat -u

# Show process name and PID associated with each connection (-p)
sudo netstat -p

# Show numerical addresses instead of resolving hostnames (-n)
netstat -n

# The most useful combination: all TCP/UDP, numeric, with PIDs
sudo netstat -tulnp

# Show the routing table
netstat -r

# Show interface statistics
netstat -i
```

**Sample output of `sudo netstat -tulnp`:**
```
Proto Recv-Q Send-Q Local Address    Foreign Address  State     PID/Program
tcp        0      0 0.0.0.0:22       0.0.0.0:*        LISTEN    892/sshd
tcp        0      0 0.0.0.0:80       0.0.0.0:*        LISTEN    4821/nginx
tcp        0      0 127.0.0.1:5432   0.0.0.0:*        LISTEN    3201/postgres
tcp        0      0 192.168.1.45:22  192.168.1.2:54320 ESTABLISHED 1205/sshd
```

**Reading the output:**

| Column | Meaning |
|---|---|
| `Proto` | Protocol — `tcp` or `udp` |
| `Local Address` | IP and port this machine is using |
| `Foreign Address` | Remote IP and port (`*` = any) |
| `State` | `LISTEN` = waiting for connections; `ESTABLISHED` = active connection |
| `PID/Program` | Process ID and name using this socket |

**Flag reference:**

| Flag | Description |
|---|---|
| `-t` | Show TCP connections |
| `-u` | Show UDP connections |
| `-l` | Show only listening sockets |
| `-n` | Show numeric addresses (faster, no DNS lookup) |
| `-p` | Show PID and program name (requires sudo) |
| `-r` | Show routing table |
| `-i` | Show interface statistics |

---

### `ss` — The Modern Replacement for `netstat`

`ss` (**s**ocket **s**tatistics) is faster than `netstat` and available by default on all modern Linux systems. The flags are nearly identical:

```bash
# All listening TCP/UDP sockets with PIDs (equivalent to netstat -tulnp)
sudo ss -tulnp

# Show all established TCP connections
ss -t state established

# Show connections on a specific port
ss -t sport = :80

# Show all sockets with detailed info
ss -s
```

---

### Networking Troubleshooting Workflow

When a network issue is reported, work through it systematically:

```bash
# Step 1: Is my own network interface up?
ip addr show

# Step 2: Can I reach my default gateway?
ip route show
ping -c 3 $(ip route | grep default | awk '{print $3}')

# Step 3: Can I reach an external IP? (bypasses DNS)
ping -c 3 8.8.8.8

# Step 4: Does DNS resolution work?
ping -c 3 google.com

# Step 5: Is the target service listening on the expected port?
sudo ss -tulnp | grep :80

# Step 6: Can I reach a specific port on a remote host?
nc -zv 192.168.1.10 5432   # Test TCP connection to postgres port
```

---

### 🛠️ Practical Exercise 1: Networking

```bash
# Step 1: Check connectivity to a domain
ping -c 4 google.com

# Review: latency, packet loss, round-trip time

# Step 2: Display network interface configuration
ifconfig
# or on modern systems:
ip addr show

# Identify: your primary interface name, IP address, subnet mask

# Step 3: List all network connections and listening ports
sudo netstat -tulnp
# or:
sudo ss -tulnp

# Identify: which services are listening, on which ports, with which PIDs

# Step 4: Check your routing table
ip route show
# or:
netstat -r

# Identify: your default gateway
```

**Reflection Questions:**
1. What does `0.0.0.0` in the Local Address column of `netstat` mean, compared to `127.0.0.1`?
2. If `ping 8.8.8.8` succeeds but `ping google.com` fails, what does that tell you?
3. Why might a port show as `LISTEN` but the service still be unreachable from another machine?

---

## Part 2: Text Processing Tools

### Why Text Processing Is a Core DevOps Skill

In Linux, almost everything is a text file — logs, configs, manifests, data exports, command output. The ability to search through, extract from, and transform that text without opening an editor is what separates slow, manual work from fast, scriptable automation.

`grep`, `awk`, and `sed` are the three fundamental tools. They are often used individually, but their real power comes from **piping them together** into a single command.

---

### `grep` — Search for Patterns in Text

`grep` (**G**lobal **R**egular **E**xpression **P**rint) reads input line by line and prints every line that matches a given pattern. It is the fastest way to find something in a file or in command output.

**Syntax:**
```bash
grep [options] "pattern" file(s)
```

**Basic usage:**
```bash
# Search for a string in a file
grep "error" app.log

# Case-insensitive search (-i)
grep -i "error" app.log

# Search and show line numbers (-n)
grep -n "error" app.log

# Show lines that do NOT match (-v = invert)
grep -v "DEBUG" app.log

# Search recursively through all files in a directory (-r)
grep -r "NexusCorp" /etc/

# Show N lines of context around each match (-A = after, -B = before, -C = both)
grep -A 2 -B 2 "CRITICAL" app.log
grep -C 3 "CRITICAL" app.log

# Count matching lines (-c)
grep -c "error" app.log

# Show only the matching part of the line, not the whole line (-o)
grep -o "192\.168\.[0-9]*\.[0-9]*" access.log

# Search for whole words only (-w)
grep -w "fail" app.log      # matches "fail" but not "failure"

# Use extended regular expressions (-E)
grep -E "error|warning|critical" app.log

# Show filenames that contain a match (-l)
grep -rl "password" /etc/

# Suppress filenames in output (-h)
grep -h "error" *.log
```

**Useful patterns:**
```bash
# Find all lines starting with a word
grep "^ERROR" app.log

# Find all lines ending with a pattern
grep "failed$" app.log

# Find IP addresses in a log
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log

# Search command output (pipe into grep)
ps aux | grep nginx
cat /etc/passwd | grep -v "nologin"
dmesg | grep -i "error"
```

**Flag reference:**

| Flag | Description |
|---|---|
| `-i` | Case-insensitive match |
| `-n` | Show line numbers |
| `-v` | Invert — show non-matching lines |
| `-r` | Recursive search through directories |
| `-l` | List filenames with matches only |
| `-c` | Count matching lines |
| `-w` | Match whole words only |
| `-A N` | Show N lines after match |
| `-B N` | Show N lines before match |
| `-C N` | Show N lines before and after match |
| `-E` | Extended regex (enables `+`, `?`, `\|`, `{}`) |
| `-o` | Print only the matching portion |
| `-q` | Quiet — return exit code only, no output |

**💡 NexusCorp Use Case:** Pulling only ERROR lines from a 500MB log file in seconds:
```bash
grep -n "ERROR" /var/log/nexuscorp/app.log | tail -20
```

---

### `awk` — Field Extraction and Text Processing

`awk` is a complete text-processing language built around the concept of **records** (lines) and **fields** (columns separated by a delimiter). It scans each line of input, matches it against patterns, and executes actions.

**Syntax:**
```bash
awk 'pattern { action }' file
```

**Core concepts:**
- `$0` — the entire line
- `$1`, `$2`, `$3`... — individual fields (columns), split by the field separator
- `NR` — current record (line) number
- `NF` — number of fields in the current record
- `FS` — field separator (default: whitespace)

**Basic usage:**
```bash
# Print the first field (column) of each line
awk '{print $1}' file.txt

# Print multiple fields
awk '{print $1, $3}' file.txt

# Use a specific delimiter (-F)
awk -F: '{print $1}' /etc/passwd       # Colon-delimited
awk -F, '{print $1, $3}' data.csv      # CSV

# Print lines where a field matches a condition
awk '$3 > 100 {print $0}' data.txt

# Print line number alongside content
awk '{print NR, $0}' file.txt

# Print only the last field of each line
awk '{print $NF}' file.txt

# Print lines containing a pattern
awk '/error/ {print}' app.log

# Sum a numeric column
awk '{sum += $2} END {print "Total:", sum}' data.txt

# Print lines between two patterns
awk '/START/,/END/ {print}' file.txt

# Count lines matching a pattern
awk '/ERROR/ {count++} END {print count}' app.log
```

**Real-world examples:**
```bash
# Extract usernames from /etc/passwd
awk -F: '{print $1}' /etc/passwd

# Show process name and CPU from ps output
ps aux | awk '{print $1, $3, $11}'

# Extract IPs from an access log (field 1)
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Print only lines where memory usage > 50%
ps aux | awk '$4 > 50 {print $0}'

# Process a CSV — print name (col 1) and score (col 3) where score > 80
awk -F, '$3 > 80 {print $1, $3}' scores.csv

# Calculate average response time from a log
awk '{sum += $NF; count++} END {print "Average:", sum/count "ms"}' timings.log
```

**`awk` program structure:**
```
BEGIN { ... }    # Runs once before processing any lines (setup)
/pattern/ { ... } # Runs for each matching line
END { ... }      # Runs once after all lines are processed (summary)
```

**💡 NexusCorp Use Case:** Monitoring which IP addresses are hitting the web server most frequently:
```bash
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10
```

---

### `sed` — Stream Editor for Find, Replace, and Transform

`sed` (**s**tream **ed**itor) reads text line by line, applies editing commands, and writes the result. It is most commonly used for **find and replace**, but it can also delete lines, insert text, and transform content.

**Syntax:**
```bash
sed [options] 'command' file
```

**The substitution command — `s/pattern/replacement/flags`:**
```bash
# Replace first occurrence on each line
sed 's/old/new/' file.txt

# Replace ALL occurrences on each line (g = global)
sed 's/old/new/g' file.txt

# Case-insensitive replace (I flag)
sed 's/error/ERROR/gi' file.txt

# Replace and edit the file in-place (-i)
sed -i 's/old/new/g' file.txt

# In-place edit with a backup file (-i.bak)
sed -i.bak 's/old/new/g' file.txt

# Preview without modifying (no -i)
sed 's/localhost/db.nexuscorp.com/g' app.conf
```

**Working with specific lines:**
```bash
# Only replace on line 5
sed '5s/old/new/' file.txt

# Only replace on lines 2 through 4
sed '2,4s/old/new/g' file.txt

# Replace only on lines matching a pattern
sed '/ERROR/s/unknown/IDENTIFIED/g' app.log
```

**Deleting lines:**
```bash
# Delete line 3
sed '3d' file.txt

# Delete lines 5 through 10
sed '5,10d' file.txt

# Delete blank lines
sed '/^$/d' file.txt

# Delete lines containing a pattern
sed '/DEBUG/d' app.log

# Delete comments (lines starting with #)
sed '/^#/d' config.conf
```

**Inserting and appending text:**
```bash
# Insert a line BEFORE line 3 (i = insert)
sed '3i\New line before line 3' file.txt

# Append a line AFTER line 3 (a = append)
sed '3a\New line after line 3' file.txt

# Add a header line before the first line
sed '1i\# Auto-generated config' config.conf
```

**Printing specific lines:**
```bash
# Print only line 5 (-n suppresses default output, p prints the match)
sed -n '5p' file.txt

# Print lines 3 through 7
sed -n '3,7p' file.txt

# Print lines matching a pattern
sed -n '/ERROR/p' app.log
```

**Common real-world patterns:**
```bash
# Update a config value in place (safe — creates backup)
sed -i.bak 's/^port=.*/port=8080/' app.conf

# Remove all blank lines from a file
sed -i '/^$/d' messy_file.txt

# Strip trailing whitespace from every line
sed -i 's/[[:space:]]*$//' file.txt

# Replace a hostname across an entire config file
sed -i 's/localhost/db.nexuscorp.internal/g' database.conf

# Extract lines between two markers
sed -n '/BEGIN CERTIFICATE/,/END CERTIFICATE/p' cert.pem

# Comment out a specific line
sed -i 's/^PasswordAuthentication yes/#PasswordAuthentication yes/' /etc/ssh/sshd_config
```

**Flag reference:**

| Option/Flag | Description |
|---|---|
| `-n` | Suppress default output (use with `p` to print selectively) |
| `-i` | Edit the file in place |
| `-i.bak` | Edit in place, creating a `.bak` backup first |
| `-e` | Chain multiple commands: `sed -e 's/a/b/' -e 's/c/d/' file` |
| `s/old/new/` | Substitute first occurrence |
| `s/old/new/g` | Substitute all occurrences (global) |
| `s/old/new/i` | Case-insensitive substitute |
| `/pattern/d` | Delete lines matching pattern |
| `Nd` | Delete line N |
| `-n '/pat/p'` | Print only lines matching pattern |

**💡 NexusCorp Use Case:** Updating a database hostname across all config files in a deployment directory without opening each one:
```bash
# Preview changes first
sed 's/db-old.nexuscorp.com/db-new.nexuscorp.com/g' configs/database.conf

# Apply in-place with backup
sed -i.bak 's/db-old.nexuscorp.com/db-new.nexuscorp.com/g' configs/*.conf
```

---

### Combining Tools — The Real Power

These tools become exponentially more useful when combined with pipes. Each tool does one thing well; you compose them to handle complex tasks.

```bash
# Who are the top 5 IP addresses hitting the server right now?
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -5

# Find all ERRORs in logs from today, show surrounding context
grep -i "error" /var/log/app/*.log | grep "$(date +%Y-%m-%d)" | tail -20

# Extract all unique HTTP status codes from an access log
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Count how many times each user ran sudo today
grep "sudo" /var/log/auth.log | grep "$(date +%b %e)" | awk '{print $6}' | sort | uniq -c

# From a CSV of servers, extract hostnames and ping them
awk -F, '{print $1}' servers.csv | while read host; do ping -c 1 -q "$host" && echo "$host: UP" || echo "$host: DOWN"; done

# Find config lines that are not comments and not blank
grep -v "^#" /etc/nginx/nginx.conf | grep -v "^$" | sed 's/^[[:space:]]*//'

# Replace sensitive values in a config before committing to git
sed 's/password=.*/password=REDACTED/g' app.conf
```

---

### 🛠️ Practical Exercise 2: Text Processing

```bash
# Setup — create a sample log file to work with
cat > sample.log << 'EOF'
2024-06-10 08:01:22 INFO  Service started successfully
2024-06-10 08:01:45 DEBUG Database connection pool initialized
2024-06-10 08:03:10 INFO  Request received from 192.168.1.10
2024-06-10 08:03:11 ERROR Failed to authenticate user: admin
2024-06-10 08:04:00 INFO  Request received from 192.168.1.22
2024-06-10 08:04:01 WARN  Response time exceeded threshold: 320ms
2024-06-10 08:04:55 ERROR Database timeout after 30s
2024-06-10 08:05:30 INFO  Request received from 192.168.1.10
2024-06-10 08:06:00 DEBUG Cache cleared
2024-06-10 08:06:45 ERROR Connection refused: 192.168.1.99:5432
EOF

# Setup — create a sample CSV file
cat > team.csv << 'EOF'
name,role,experience_years,location
Alice,DevOps Engineer,5,Mumbai
Bob,SRE,3,Bangalore
Carol,Cloud Architect,8,Hyderabad
Dave,Linux Admin,2,Pune
Eve,DevOps Engineer,6,Chennai
EOF


# --- grep exercises ---

# Exercise 1a: Find all ERROR lines
grep "ERROR" sample.log

# Exercise 1b: Case-insensitive search for "warn" or "error"
grep -iE "warn|error" sample.log

# Exercise 1c: Show ERROR lines with 1 line of context before each
grep -B 1 "ERROR" sample.log

# Exercise 1d: Count how many ERROR lines exist
grep -c "ERROR" sample.log

# Exercise 1e: Find all lines NOT containing DEBUG
grep -v "DEBUG" sample.log


# --- awk exercises ---

# Exercise 2a: Print only the log level (column 3) from each line
awk '{print $3}' sample.log

# Exercise 2b: Print timestamp and message for ERROR lines only
awk '/ERROR/ {print $1, $2, $4, $5, $6, $7}' sample.log

# Exercise 2c: From the CSV, print name and role
awk -F, 'NR > 1 {print $1, "-", $2}' team.csv

# Exercise 2d: From the CSV, print only DevOps Engineers
awk -F, '$2 == "DevOps Engineer" {print $1, $3, "years"}' team.csv

# Exercise 2e: Count lines by log level
awk '{print $3}' sample.log | sort | uniq -c | sort -rn


# --- sed exercises ---

# Exercise 3a: Replace "ERROR" with "** ERROR **" for visibility
sed 's/ERROR/\*\* ERROR \*\*/g' sample.log

# Exercise 3b: Remove all DEBUG lines
sed '/DEBUG/d' sample.log

# Exercise 3c: Remove blank lines
sed '/^$/d' sample.log

# Exercise 3d: Replace IP address in the log
sed 's/192\.168\.1\.10/10.0.0.45/g' sample.log

# Exercise 3e: In-place update on the CSV (with backup)
sed -i.bak 's/DevOps Engineer/DevOps SRE/g' team.csv
cat team.csv
# Restore from backup
mv team.csv.bak team.csv
```

---

## 🔧 Mini-Project: NexusCorp Network & Log Audit

**Scenario:** The security team has asked for a quick audit of the staging server — what network services are exposed, and what errors appear in the application logs from the last hour.

```bash
# ─── Section 1: Network Audit ───

echo "=== NexusCorp Network Audit ===" > /tmp/nexus_net_audit.txt
echo "Date: $(date)" >> /tmp/nexus_net_audit.txt
echo "" >> /tmp/nexus_net_audit.txt

# Capture interface information
echo "--- Network Interfaces ---" >> /tmp/nexus_net_audit.txt
ip addr show >> /tmp/nexus_net_audit.txt

# Capture listening ports
echo "" >> /tmp/nexus_net_audit.txt
echo "--- Listening Ports ---" >> /tmp/nexus_net_audit.txt
sudo ss -tulnp >> /tmp/nexus_net_audit.txt

# Check connectivity to key services
echo "" >> /tmp/nexus_net_audit.txt
echo "--- Connectivity Checks ---" >> /tmp/nexus_net_audit.txt
ping -c 2 -q 8.8.8.8 >> /tmp/nexus_net_audit.txt 2>&1


# ─── Section 2: Log Audit ───

echo "" >> /tmp/nexus_net_audit.txt
echo "--- Recent Errors in Sample Log ---" >> /tmp/nexus_net_audit.txt

# Use grep + awk + sed together
grep "ERROR" sample.log \
  | awk '{print $1, $2, $3, $4, $5, $6, $7, $8}' \
  | sed 's/ERROR/[ERROR]/g' \
  >> /tmp/nexus_net_audit.txt

# Error summary count
echo "" >> /tmp/nexus_net_audit.txt
echo "--- Log Level Summary ---" >> /tmp/nexus_net_audit.txt
awk '{print $3}' sample.log | sort | uniq -c | sort -rn >> /tmp/nexus_net_audit.txt


# ─── Display the report ───
cat /tmp/nexus_net_audit.txt
```

**Bonus Challenges:**
- Use `grep -E` to extract all IP addresses from `sample.log`
- Use `awk` to calculate the average experience from `team.csv`
- Use `sed` to add a comment header (`# Reviewed: <date>`) to the top of `team.csv`
- Chain `ping`, `grep`, and `awk` to test 3 hosts and report only the average round-trip time for each

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **ping** | A command-line utility that sends ICMP echo requests to test host reachability and measure latency |
| **ICMP** | Internet Control Message Protocol — the protocol used by `ping` |
| **ifconfig** | A command-line utility for displaying and configuring network interfaces (legacy; replaced by `ip`) |
| **ip** | The modern command-line tool for network interface configuration, routing, and more |
| **netstat** | Displays network connections, routing tables, and interface statistics (legacy; replaced by `ss`) |
| **ss** | Socket statistics — the modern, faster replacement for `netstat` |
| **Network Interface** | A hardware or virtual component connecting a machine to a network (e.g., `eth0`, `lo`) |
| **Loopback (`lo`)** | A virtual interface (`127.0.0.1`) for local-only communication within the same machine |
| **MAC Address** | Hardware address of a network interface — unique per device |
| **Listening Port** | A port on which a service is waiting for incoming connections |
| **grep** | A text-processing utility for searching lines in text that match a pattern |
| **awk** | A text-processing utility for scanning and parsing structured text by fields; a complete scripting language |
| **sed** | A stream editor for filtering and transforming text — commonly used for find-and-replace |
| **Regular Expression (regex)** | A pattern language used by `grep`, `awk`, and `sed` to match text |
| **Pipe (`\|`)** | Sends the output of one command as the input to the next |
| **Field Separator** | The character `awk` uses to split a line into fields (default: whitespace; change with `-F`) |
| **In-place Edit (`-i`)** | `sed` flag that modifies the original file directly rather than printing to stdout |
| **Stream** | A sequence of text data flowing through stdin, stdout, or a pipe |
| **TTL** | Time to Live — in `ping`, the number of network hops a packet can traverse before being discarded |
| **Packet Loss** | The percentage of ping packets that received no reply — indicates network instability |

---

## 📚 Resources

- [Linux Networking Commands — TecMint](https://www.tecmint.com/linux-network-configuration-and-troubleshooting-commands/) — practical guide to networking commands
- [Understanding sed and awk — TecMint](https://www.tecmint.com/sed-command-to-create-edit-and-manipulate-files-in-linux/) — practical sed and awk walkthrough
- [`grep` Man Page](https://man7.org/linux/man-pages/man1/grep.1.html) — full official grep reference
- [`awk` Manual (GNU)](https://www.gnu.org/software/gawk/manual/gawk.html) — comprehensive awk language reference
- [`sed` Manual (GNU)](https://www.gnu.org/software/sed/manual/sed.html) — official sed reference
- [`ss` Man Page](https://man7.org/linux/man-pages/man8/ss.8.html) — modern socket statistics reference
- [Regex101](https://regex101.com) — interactive regular expression tester with explanations

---

## 🔭 Day 10 Preview: Shell Scripting Fundamentals

Tomorrow everything you've learned comes together. Instead of running commands one at a time, you'll write **shell scripts** — files containing sequences of commands that execute automatically.

You'll learn:
- Script structure: shebang, comments, variables
- Conditionals: `if`, `elif`, `else`
- Loops: `for`, `while`
- Functions and arguments
- How NexusCorp automates repetitive server checks with a single script instead of 20 manual commands

Every command from Days 1 through 9 becomes a building block. Tomorrow you start building with them.

---

> 💬 *"Networking tells you where the data goes. Text processing tells you what it says when it gets there."*
>
> — DevOps Learning by Yukta
