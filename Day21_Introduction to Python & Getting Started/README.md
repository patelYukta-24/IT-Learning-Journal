# Day 21: Introduction to Python & Getting Started

> **DevOps Transformation** | Python & Automation Series

---

## 📋 Overview

Day 20 answered *why* Python matters for DevOps. Today you set up your environment and write your first code.

Python is an **interpreted, high-level, general-purpose programming language**. It was designed with one priority above all others: readability. Python code reads almost like plain English, which is why it has become the dominant language for automation, scripting, cloud tooling, data science, and increasingly, the backbone of DevOps workflows.

By the end of today you will have Python installed, understood its core data types, written and run your first script, and explored the interactive shell — the fastest way to experiment with Python concepts in real time.

---

## 🗺️ What You'll Do Today

| Task | Activity | Goal |
|---|---|---|
| Task 1 | Install Python | Working Python environment on your machine |
| Task 2 | Study data types | Understand the building blocks of every Python program |
| Task 3 | Write your first script | `hello_devops.py` — Python executing your instructions |
| Task 4 | Explore the interactive shell | Experiment with Python in real time |
| Task 5 | Documentation dive | Navigate the official Python docs — a skill you'll use constantly |

---

## 🏢 NexusCorp Scenario

> NexusCorp's new DevOps apprentice is handed a task on their first day: "Write a script that reads a list of server names from a file and prints their status." Before they can do that, they need Python installed, they need to understand how Python handles text and lists, and they need to know how to run a `.py` file. That is exactly what today covers — the prerequisite session before any real automation can happen.

---

## Part 1: What Is Python?

### Language Characteristics

| Property | Meaning | DevOps Implication |
|---|---|---|
| **Interpreted** | Code runs line by line via an interpreter — no compilation step | Run scripts instantly; easy to test and iterate |
| **High-level** | Abstracts away memory management and hardware details | Focus on logic, not low-level system details |
| **General-purpose** | Not designed for one domain — works for web, scripting, data, cloud | One language for automation, API calls, data processing, and more |
| **Dynamically typed** | Variable types are determined at runtime, not declared | Faster to write; type errors show at runtime, not compile time |
| **Garbage collected** | Memory is managed automatically | No manual memory allocation or freeing |

### Why Python Dominates DevOps

```
Infrastructure as Code  →  Ansible (written in Python)
Cloud Automation        →  boto3 (AWS Python SDK)
CI/CD Scripting         →  Python pipeline steps
Monitoring & Alerting   →  Python exporters and scripts
Container Management    →  Docker SDK for Python
Kubernetes              →  Official Python client library
Configuration Mgmt      →  SaltStack (written in Python)
```

Python is not one tool in the DevOps toolbox — it is the thread that connects all the other tools.

---

## Task 1: Install Python

### Check If Python Is Already Installed

Before installing, check whether Python is already available:

```bash
# Check Python 3
python3 --version

# On some systems, 'python' points to Python 3
python --version

# Check where Python is installed
which python3
```

If you see `Python 3.x.x` — you're ready. If not, install it below.

---

### Installation by Operating System

**Ubuntu / Debian:**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv -y

# Verify
python3 --version
pip3 --version
```

**RHEL / CentOS / Amazon Linux:**
```bash
sudo yum install python3 python3-pip -y

# Verify
python3 --version
```

**macOS:**
```bash
# Using Homebrew (recommended)
brew install python3

# Verify
python3 --version
```

**Windows:**
- Download the installer from [python.org/downloads](https://www.python.org/downloads/)
- During installation: ✅ check **"Add Python to PATH"**
- Verify in Command Prompt: `python --version`

---

### Verify Your Installation

```bash
# Check Python version
python3 --version
# Expected: Python 3.11.x or similar (3.8+ is fine for all DevOps work)

# Check pip (Python package installer)
pip3 --version

# Confirm Python runs
python3 -c "print('Python is working!')"
# Output: Python is working!
```

**Python 2 vs Python 3:** Python 2 reached end-of-life in January 2020. Always use Python 3. If your system has both, always use `python3` explicitly to avoid accidentally running Python 2.

---

### Understanding the Python Ecosystem

```
Python Installation
├── python3              ← The interpreter (runs your .py files)
├── pip3                 ← Package manager (installs third-party libraries)
└── venv                 ← Creates isolated virtual environments per project

Third-party packages (installed via pip3):
├── boto3                ← AWS automation
├── requests             ← HTTP API calls
├── paramiko             ← SSH connections
├── pyyaml               ← Parse YAML files
└── python-dotenv        ← Load environment variables from .env files
```

---

## Task 2: Study Python Data Types

Every piece of data in Python has a **type** — a classification that tells Python what kind of value it is and what operations can be performed on it. Understanding data types is the foundation of writing any Python program.

---

### Core Built-In Data Types

#### `int` — Integer (whole numbers)

```python
# Integers — no decimal point
server_count = 12
port_number = 8080
exit_code = 0
max_retries = 3

# Arithmetic operations
total_cores = server_count * 4        # 48
available_ports = 65535 - port_number # 57455

# Check the type
type(server_count)    # <class 'int'>
```

---

#### `float` — Floating Point (decimal numbers)

```python
# Floats — have a decimal point
cpu_usage = 73.5
memory_threshold = 80.0
disk_used_gb = 128.4

# Mixed arithmetic (int + float = float)
average = (73.5 + 68.2 + 81.1) / 3   # 74.26666...

type(cpu_usage)    # <class 'float'>
```

---

#### `str` — String (text)

```python
# Strings — any text, enclosed in single or double quotes
hostname = "web-server-01"
environment = 'production'
ip_address = "192.168.1.45"

# Multi-line string (triple quotes)
config_note = """
This server handles incoming web traffic.
Maintained by the NexusCorp ops team.
"""

# String operations
print(hostname.upper())               # WEB-SERVER-01
print(hostname.replace("01", "02"))   # web-server-02
print(len(hostname))                  # 13

# f-strings — embed variables directly in text (most common format)
port = 443
message = f"Connecting to {hostname} on port {port}"
print(message)    # Connecting to web-server-01 on port 443

# String slicing
version = "Python-3.11.4"
print(version[7:])     # 3.11.4
print(version[:6])     # Python

type(hostname)    # <class 'str'>
```

---

#### `bool` — Boolean (True or False)

```python
# Booleans — only two possible values
service_running = True
deployment_failed = False
is_production = True

# Booleans are the result of comparisons
disk_usage = 85
alert_needed = disk_usage > 80      # True
within_limit = disk_usage <= 80     # False

# Used in conditionals
if service_running:
    print("Service is UP")
else:
    print("Service is DOWN")

type(service_running)    # <class 'bool'>
```

---

#### `list` — Ordered, Mutable Sequence

Lists hold multiple values in order. Items can be added, removed, or changed after the list is created.

```python
# Lists — square brackets, comma-separated
servers = ["web-01", "web-02", "db-01", "cache-01"]
ports = [80, 443, 8080, 3306]
mixed = ["web-01", 443, True, 99.5]    # Can mix types

# Accessing items (zero-indexed)
print(servers[0])       # web-01
print(servers[-1])      # cache-01 (last item)
print(servers[1:3])     # ['web-02', 'db-01'] (slice)

# Modifying lists
servers.append("db-02")            # Add to end
servers.insert(0, "lb-01")         # Insert at position
servers.remove("cache-01")         # Remove by value
popped = servers.pop()             # Remove and return last item

# Useful list operations
print(len(servers))                # Number of items
print("web-01" in servers)         # True/False membership check
servers.sort()                     # Sort in place

# Loop over a list
for server in servers:
    print(f"Checking: {server}")

type(servers)    # <class 'list'>
```

---

#### `tuple` — Ordered, Immutable Sequence

Tuples are like lists but **cannot be changed after creation**. Use them for data that should not be modified — configuration constants, fixed collections.

```python
# Tuples — parentheses, comma-separated
valid_environments = ("development", "staging", "production")
db_connection = ("db.nexuscorp.com", 5432, "nexusdb")

# Accessing (same as list)
print(valid_environments[0])    # development
print(db_connection[-1])        # nexusdb

# Unpacking
host, port, dbname = db_connection
print(f"Connecting to {dbname} at {host}:{port}")

# Tuples cannot be modified — this would raise an error:
# valid_environments[0] = "dev"    # TypeError!

type(valid_environments)    # <class 'tuple'>
```

---

#### `dict` — Dictionary (Key-Value Pairs)

Dictionaries store data as key-value pairs. They are the most important data structure for DevOps Python — JSON responses, config files, and server metadata all map naturally to dictionaries.

```python
# Dictionaries — curly braces, key: value pairs
server = {
    "hostname": "web-server-01",
    "ip": "192.168.1.45",
    "port": 80,
    "status": "running",
    "cpu_usage": 45.2,
    "services": ["nginx", "node", "redis"]
}

# Accessing values
print(server["hostname"])      # web-server-01
print(server.get("port"))      # 80
print(server.get("region", "unknown"))  # unknown (safe default)

# Modifying dictionaries
server["status"] = "maintenance"      # Update existing key
server["region"] = "ap-south-1"       # Add new key
del server["cpu_usage"]               # Remove a key

# Looping over a dictionary
for key, value in server.items():
    print(f"{key}: {value}")

# Useful methods
print(server.keys())      # All keys
print(server.values())    # All values
print("ip" in server)     # True — check if key exists

type(server)    # <class 'dict'>
```

---

#### `set` — Unordered Collection of Unique Values

```python
# Sets — no duplicates, no guaranteed order
active_ports = {80, 443, 8080, 443, 80}   # Duplicates removed
print(active_ports)    # {80, 8080, 443}

# Set operations
required_ports = {80, 443, 22}
open_ports = {80, 443, 8080, 3306}

missing = required_ports - open_ports      # Ports not open
print(missing)    # {22}

type(active_ports)    # <class 'set'>
```

---

### Type Conversion

```python
# Convert between types
port_str = "8080"
port_int = int(port_str)         # "8080" → 8080
cpu_float = float("73.5")        # "73.5" → 73.5
count_str = str(12)              # 12 → "12"
as_list = list("hello")          # ['h', 'e', 'l', 'l', 'o']

# Check type
print(type(port_int))            # <class 'int'>
print(isinstance(port_int, int)) # True
```

---

### Data Types Quick Reference

| Type | Example | Mutable? | DevOps Use Case |
|---|---|---|---|
| `int` | `8080` | N/A | Port numbers, counts, exit codes |
| `float` | `73.5` | N/A | CPU/memory percentages, latency |
| `str` | `"web-01"` | No | Hostnames, IPs, commands, log messages |
| `bool` | `True` | N/A | Service status, flag conditions |
| `list` | `["a", "b"]` | Yes | Server lists, command output lines |
| `tuple` | `("a", "b")` | No | Fixed config values, DB connection params |
| `dict` | `{"k": "v"}` | Yes | Server metadata, API responses, configs |
| `set` | `{1, 2, 3}` | Yes | Unique values, port sets, tag collections |

---

## Task 3: Write Your First Python Script

### Creating `hello_devops.py`

```bash
# Step 1: Create the file
nano hello_devops.py
# Or: touch hello_devops.py, then open in your editor
```

```python
# hello_devops.py
print("Hello, DevOps!")
```

```bash
# Step 2: Run the script
python3 hello_devops.py
# Output: Hello, DevOps!
```

---

### Extend It — Make It More DevOps

Once the basic script works, extend it step by step:

```python
# hello_devops.py — extended version

import datetime

# Variables
environment = "staging"
server_count = 12
python_version = "3.11"

# Print with f-strings
print("Hello, DevOps!")
print(f"Environment : {environment}")
print(f"Servers managed: {server_count}")
print(f"Python version: {python_version}")
print(f"Script run at: {datetime.datetime.now()}")

# A simple list
services = ["nginx", "postgresql", "redis"]
print(f"\nMonitored services:")
for service in services:
    print(f"  - {service}")
```

```bash
python3 hello_devops.py
```

**Expected output:**
```
Hello, DevOps!
Environment : staging
Servers managed: 12
Python version: 3.11
Script run at: 2024-06-10 14:32:01.234567

Monitored services:
  - nginx
  - postgresql
  - redis
```

---

### How Python Executes Your Script

```
python3 hello_devops.py
     │          │
     │          └── Your source file (plain text)
     │
     └── Python interpreter:
         1. Reads hello_devops.py line by line
         2. Converts each line to bytecode
         3. Executes the bytecode
         4. Outputs the result
```

No compilation step, no build process — write code, run immediately. This is one of Python's core advantages for scripting and automation.

---

## Task 4: Explore the Python Interactive Shell

The **Python interactive shell** (also called the REPL — Read, Evaluate, Print, Loop) lets you type Python expressions and see results instantly. It is the fastest way to test ideas, explore data types, and experiment with new concepts.

```bash
# Enter the interactive shell
python3
```

You will see:
```
Python 3.11.4 (main, Jul 25 2023, 22:00:00)
[GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

The `>>>` prompt means Python is ready for input.

---

### Things to Try in the Shell

```python
# Basic arithmetic
>>> 2 + 2
4
>>> 10 / 3
3.3333333333333335
>>> 10 // 3       # Integer division
3
>>> 10 % 3        # Modulo (remainder)
1
>>> 2 ** 8        # Exponentiation
256

# Variables
>>> hostname = "web-server-01"
>>> port = 8080
>>> print(f"{hostname}:{port}")
web-server-01:8080

# String operations
>>> "hello".upper()
'HELLO'
>>> "192.168.1.45".split(".")
['192', '168', '1', '45']
>>> len("nexuscorp")
9

# Lists
>>> servers = ["web-01", "web-02", "db-01"]
>>> servers.append("cache-01")
>>> servers
['web-01', 'web-02', 'db-01', 'cache-01']
>>> len(servers)
4

# Dictionary
>>> config = {"host": "localhost", "port": 5432}
>>> config["host"]
'localhost'
>>> config.keys()
dict_keys(['host', 'port'])

# Type checking
>>> type(8080)
<class 'int'>
>>> type("hello")
<class 'str'>
>>> type([1, 2, 3])
<class 'list'>
>>> isinstance(8080, int)
True

# Boolean operations
>>> 80 > 443
False
>>> "web-01" in ["web-01", "web-02"]
True
>>> not False
True

# Exit the shell
>>> exit()
```

---

### Shell vs Script — When to Use Each

| Use the Interactive Shell | Use a Script (`.py` file) |
|---|---|
| Testing a single idea or expression | Any logic you want to save and rerun |
| Checking what a function returns | Tasks with more than 3–4 lines of code |
| Quickly exploring a data type | Anything that will run in production |
| Debugging a small piece of logic | Automation that runs on a schedule |
| Learning how an operator or method works | Code you want to version-control |

---

## Task 5: Documentation Dive

### Navigating the Official Python Documentation

[docs.python.org](https://docs.python.org/3/) is your long-term companion. Knowing how to navigate it efficiently is a skill worth investing in now.

**Start here:**

| Section | URL Path | What's There |
|---|---|---|
| Beginner's Guide | `/3/tutorial/index.html` | Official getting-started tutorial |
| Built-in Types | `/3/library/stdtypes.html` | Complete reference for all data types |
| Built-in Functions | `/3/library/functions.html` | `print()`, `len()`, `type()`, `range()`, and all others |
| Standard Library | `/3/library/index.html` | Every built-in module — `os`, `sys`, `json`, `datetime`, etc. |
| Glossary | `/3/glossary.html` | Definitions of Python-specific terms |

**Navigation tips:**
- Use `Ctrl+F` to search within a page
- The search box at the top of docs.python.org searches the full documentation
- Each function and method page has examples — always read the examples first
- The "Source" link on each page shows the actual Python implementation

**Specific pages to read today:**

```
1. https://docs.python.org/3/tutorial/introduction.html
   → Numbers, strings, and lists introduction

2. https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str
   → String methods reference (you'll use these constantly)

3. https://docs.python.org/3/library/stdtypes.html#list
   → List methods reference

4. https://docs.python.org/3/library/stdtypes.html#mapping-types-dict
   → Dictionary methods reference
```

---

## 🔧 Mini-Project: System Info Reporter

Combine everything from today — variables, data types, f-strings, lists, and dictionaries — into a practical script.

```python
#!/usr/bin/env python3
"""
system_info.py
A basic system information reporter using Python built-ins.
No external libraries required.
"""

import datetime
import platform
import os

# ── Configuration ────────────────────────────────────────────
ENVIRONMENT = "staging"
TEAM = "NexusCorp DevOps"
MONITORED_SERVICES = ["nginx", "postgresql", "redis", "node"]

# ── Collect system information ────────────────────────────────
hostname = platform.node()
operating_system = platform.system()
os_version = platform.version()
python_version = platform.python_version()
current_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
current_user = os.environ.get("USER", "unknown")

# ── Server metadata (as a dictionary) ────────────────────────
server_info = {
    "hostname": hostname,
    "environment": ENVIRONMENT,
    "os": operating_system,
    "os_version": os_version,
    "python_version": python_version,
    "current_user": current_user,
    "report_time": current_time,
    "monitored_services": MONITORED_SERVICES,
    "service_count": len(MONITORED_SERVICES),
}

# ── Generate the report ───────────────────────────────────────
print("=" * 50)
print(f"  {TEAM} — System Info Report")
print("=" * 50)

for key, value in server_info.items():
    if isinstance(value, list):
        print(f"\n  {key.upper()}:")
        for item in value:
            print(f"    - {item}")
    else:
        print(f"  {key:<20}: {value}")

print("=" * 50)
print(f"  Report complete. {server_info['service_count']} services monitored.")
print("=" * 50)
```

```bash
python3 system_info.py
```

**Expected output:**
```
==================================================
  NexusCorp DevOps — System Info Report
==================================================
  hostname            : your-machine-name
  environment         : staging
  os                  : Linux
  os_version          : #1 SMP ...
  python_version      : 3.11.4
  current_user        : devops
  report_time         : 2024-06-10 14:32:01

  MONITORED_SERVICES:
    - nginx
    - postgresql
    - redis
    - node

  service_count       : 4
==================================================
  Report complete. 4 services monitored.
==================================================
```

**Extend it yourself:**
- Add a `tuple` for valid environments and check if `ENVIRONMENT` is one of them
- Add a `set` of critical vs non-critical services
- Add a `bool` flag `IS_PRODUCTION` and print a warning if it is `True`

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Interpreter** | A program that reads and executes Python code line by line without a prior compilation step |
| **REPL** | Read, Evaluate, Print, Loop — the interactive Python shell where expressions are evaluated immediately |
| **Variable** | A named storage location for a value — `hostname = "web-01"` |
| **Data type** | A classification of a value that determines what operations can be performed on it |
| **`int`** | Integer data type — whole numbers without a decimal point |
| **`float`** | Floating-point data type — numbers with a decimal point |
| **`str`** | String data type — sequences of text characters |
| **`bool`** | Boolean data type — only `True` or `False` |
| **`list`** | Ordered, mutable sequence of values — items can be added, removed, or changed |
| **`tuple`** | Ordered, immutable sequence of values — cannot be changed after creation |
| **`dict`** | Dictionary — unordered collection of key-value pairs |
| **`set`** | Unordered collection of unique values — no duplicates |
| **f-string** | A string prefixed with `f` that allows embedding variables: `f"Hello, {name}!"` |
| **`print()`** | Built-in function that outputs text to the terminal |
| **`type()`** | Built-in function that returns the type of a value |
| **`len()`** | Built-in function that returns the number of items in a sequence |
| **`isinstance()`** | Built-in function that checks if a value is an instance of a given type |
| **`pip3`** | Python's package installer — used to install third-party libraries |
| **Script** | A `.py` file containing Python code to be executed by the interpreter |
| **Module** | A Python file that can be imported to provide additional functions and variables |
| **`import`** | Statement used to load a module: `import datetime` |
| **Index** | The position of an item in a sequence — Python uses zero-based indexing (first item is index `0`) |
| **Slice** | A subset of a sequence: `servers[1:3]` returns items at index 1 and 2 |

---

## 📚 Resources

- [Python Official Documentation](https://docs.python.org/3/) — the definitive reference
- [Python Beginner's Guide](https://docs.python.org/3/tutorial/index.html) — official getting-started tutorial
- [Automate the Boring Stuff with Python (free)](https://automatetheboringstuff.com/) — practical Python for automation tasks
- [Python Built-in Types Reference](https://docs.python.org/3/library/stdtypes.html) — complete data type documentation
- [Real Python — Python Basics](https://realpython.com/python-basics/) — well-structured beginner tutorials
- [Python Cheat Sheet — devhints.io](https://devhints.io/python) — quick syntax reference
- [Exercism Python Track](https://exercism.org/tracks/python) — free, mentor-reviewed Python exercises

---

## 🔭 Day 22 Preview: Python Control Flow & Functions

Now that you have Python installed and understand data types, Day 22 introduces the logic structures that make scripts actually *do* something based on conditions.

You will learn:
- `if`, `elif`, `else` — conditional execution
- `for` and `while` loops — repeating operations
- `break`, `continue`, `pass` — controlling loop behavior
- Defining and calling functions — reusable, organized code
- Practical DevOps scripts: a disk space checker, a service status verifier, and a log line counter

With data types from today and control flow from Day 22, you will be able to write scripts that make real decisions — not just print static output.

---

> 💬 *"Every Python expert started by typing `print('Hello, World!')` and running it. The only difference between then and now is the number of times they typed the next line."*
>
> — DevOps Learning by Yukta
