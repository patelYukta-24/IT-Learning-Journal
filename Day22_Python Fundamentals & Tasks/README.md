# Day 22: Python Fundamentals & Tasks

> **DevOps Transformation** | Python & Automation Series

---

## 📋 Overview

Yesterday you installed Python, understood data types, and ran your first script. Today you build the programming constructs that turn static scripts into dynamic, decision-making automation tools.

**Variables, control structures, loops, and functions** are the four pillars of any program — in any language. Master these in Python and you can build tools that check conditions, repeat operations, handle data collections, and organize logic into reusable blocks. Every DevOps automation script you will ever write uses these four constructs, usually all at once.

By the end of today you will have written scripts that make decisions, iterate over server lists, and call reusable functions — the core pattern of every practical DevOps script.

---

## 🗺️ What You'll Do Today

| Task | Topic | DevOps Application |
|---|---|---|
| Task 1 | Variables | Store hostnames, counts, thresholds |
| Task 2 | Control Structures | Make decisions — if service is down, alert |
| Task 3 | Loops | Repeat checks across all servers |
| Task 4 | Functions | Reusable health-check logic |
| Task 5 | Built-in Functions | `len()`, `type()`, `range()`, `input()` |
| Task 6 | Lists and Tuples | Server lists, fixed config values |
| Task 7 | Python Help System | Navigate docs from the shell |
| Task 8 | Exploring Errors | Read and understand Python exceptions |

---

## 🏢 NexusCorp Scenario

> The NexusCorp ops team needs a script that checks whether each server in a list is above a CPU threshold, prints a warning for those that are, counts how many are critical, and reports a summary. Every single line of that script uses something from today's session — variables to store the data, a loop to iterate the servers, a conditional to check the threshold, a function to format the output, and built-ins to count and format.
>
> That script is this session's mini-project. Build today's fundamentals, and it will make complete sense.

---

## Task 1: Understanding Variables

### What Is a Variable?

A variable is a named container that holds a value. In Python, you do not declare the type — Python determines it automatically based on the value you assign.

```python
# No type declaration needed — Python figures it out
name = "DevOps"
age = 30
salary = 1000.50

print(name)      # DevOps
print(age)       # 30
print(salary)    # 1000.5
```

---

### Variable Naming Rules

```python
# Valid variable names
server_name = "web-01"      # snake_case — preferred in Python
cpuUsage = 73.5             # camelCase — works but not Pythonic
MAX_RETRIES = 3             # ALL_CAPS — convention for constants
_private = "internal"       # Leading underscore — internal use signal

# Invalid variable names
# 2servers = []             # Cannot start with a number
# server-name = ""          # Hyphens not allowed (use underscore)
# class = "web"             # Cannot use Python reserved keywords
```

**Python naming conventions (PEP 8):**

| What | Convention | Example |
|---|---|---|
| Variables and functions | `snake_case` | `server_count`, `check_disk()` |
| Constants | `ALL_CAPS` | `MAX_RETRIES`, `DEFAULT_PORT` |
| Classes | `PascalCase` | `ServerConfig`, `DeploymentPipeline` |
| Private (internal) | `_leading_underscore` | `_internal_flag` |

---

### Variables in a DevOps Context

```python
# Configuration variables
environment = "staging"
max_disk_threshold = 80        # percentage
default_port = 443
retry_count = 3
deploy_timeout = 300           # seconds

# Dynamic values computed from other variables
warning_threshold = max_disk_threshold - 10    # 70
critical_message = f"CRITICAL: Disk above {max_disk_threshold}%"

# Multiple assignment
host = ip = "192.168.1.45"     # Both point to same value
x, y, z = 1, 2, 3             # Unpack multiple values at once

# Swap values (Python makes this elegant)
primary, secondary = "web-01", "web-02"
primary, secondary = secondary, primary
print(primary)    # web-02

# Check a variable's type and value
server_count = 12
print(type(server_count))              # <class 'int'>
print(f"Managing {server_count} servers")
```

---

### Subtask 1

```python
# subtask_variables.py

name = "DevOps"
age = 30
salary = 1000.50

print(f"Name:   {name}")
print(f"Age:    {age}")
print(f"Salary: {salary}")
print(f"Types:  {type(name)}, {type(age)}, {type(salary)}")
```

```bash
python3 subtask_variables.py
```

---

## Task 2: Control Structures

Control structures make your script respond to conditions. Instead of always doing the same thing, your script can take different paths depending on the data it encounters.

### `if` / `elif` / `else`

```python
if condition:
    # runs when condition is True
elif another_condition:
    # runs when first is False, this is True
else:
    # runs when all above are False
```

**Important syntax rules:**
- The condition is followed by a colon `:`
- The indented block (4 spaces) is the body — Python uses indentation to define blocks, not curly braces
- `elif` and `else` are optional

---

### Comparison and Logical Operators

```python
# Comparison operators
x > y       # Greater than
x < y       # Less than
x >= y      # Greater than or equal
x <= y      # Less than or equal
x == y      # Equal (note: == not =)
x != y      # Not equal

# Logical operators
x > 0 and x < 100    # Both conditions must be True
x < 0 or x > 100     # At least one must be True
not x > 0            # Inverts the condition

# Membership and identity
"nginx" in services          # True if value is in collection
service is None              # True if value is exactly None
service is not None          # True if value is not None
```

---

### Subtask 2: Positive, Negative, or Zero

```python
# subtask_conditions.py

number = float(input("Enter a number: "))

if number > 0:
    print(f"{number} is POSITIVE")
elif number < 0:
    print(f"{number} is NEGATIVE")
else:
    print("The number is ZERO")
```

---

### DevOps Control Structure Examples

```python
# Disk usage alert
disk_usage = 87

if disk_usage >= 90:
    print("CRITICAL: Disk usage above 90% — immediate action required")
elif disk_usage >= 80:
    print("WARNING: Disk usage above 80% — monitor closely")
elif disk_usage >= 70:
    print("NOTICE: Disk usage above 70% — plan for cleanup")
else:
    print(f"OK: Disk usage at {disk_usage}%")


# Service status check
service_status = "stopped"

if service_status == "running":
    print("Service is healthy")
elif service_status == "stopped":
    print("WARNING: Service is stopped — attempting restart")
elif service_status == "failed":
    print("CRITICAL: Service has failed — alerting on-call")
else:
    print(f"UNKNOWN: Unexpected status '{service_status}'")


# Environment guard
environment = "production"
action = "delete_database"

if environment == "production" and action.startswith("delete"):
    print("BLOCKED: Destructive actions not permitted on production")
else:
    print(f"Executing: {action} on {environment}")


# Checking multiple conditions with 'in'
valid_environments = ["development", "staging", "production"]
selected_env = "staging"

if selected_env in valid_environments:
    print(f"Deploying to: {selected_env}")
else:
    print(f"ERROR: '{selected_env}' is not a valid environment")
```

---

## Task 3: Loops

Loops let your script repeat operations — over a list of servers, a range of numbers, or until a condition changes. They are the engine of bulk automation.

### `for` Loop — Iterate Over a Sequence

```python
# Loop over a list
for item in collection:
    # do something with item
```

```python
# Loop over a list of servers
servers = ["web-01", "web-02", "db-01", "cache-01"]

for server in servers:
    print(f"Checking: {server}")


# Loop over a range of numbers
for i in range(1, 11):          # 1 to 10 (11 is excluded)
    print(i)

# range(start, stop, step)
for i in range(0, 100, 10):    # 0, 10, 20, ... 90
    print(i)

for i in range(10, 0, -1):     # Countdown: 10, 9, 8, ... 1
    print(i)


# Loop with index using enumerate()
services = ["nginx", "postgresql", "redis"]
for index, service in enumerate(services):
    print(f"  {index + 1}. {service}")
# Output:
#   1. nginx
#   2. postgresql
#   3. redis


# Loop over a dictionary
server_info = {"hostname": "web-01", "ip": "192.168.1.45", "port": 80}
for key, value in server_info.items():
    print(f"  {key}: {value}")


# Nested loops
environments = ["staging", "production"]
services = ["nginx", "redis"]

for env in environments:
    for service in services:
        print(f"  Checking {service} on {env}")
```

---

### `while` Loop — Repeat Until Condition Changes

```python
# Repeat while a condition is True
while condition:
    # do something
    # eventually condition must become False, or use break
```

```python
# Count from 1 to 10
count = 1
while count <= 10:
    print(count)
    count += 1    # count = count + 1


# Retry loop — common pattern in DevOps scripts
import time

max_retries = 5
attempt = 0
connected = False

while attempt < max_retries and not connected:
    attempt += 1
    print(f"Connection attempt {attempt} of {max_retries}...")
    # Simulate connection (in real code, try the actual connection here)
    if attempt == 3:          # Simulate success on attempt 3
        connected = True
    else:
        time.sleep(1)         # Wait before retrying

if connected:
    print("Connection successful!")
else:
    print(f"Failed after {max_retries} attempts")
```

---

### Loop Control: `break`, `continue`, `pass`

```python
# break — exit the loop immediately
servers = ["web-01", "web-02", "OFFLINE", "web-03"]

for server in servers:
    if server == "OFFLINE":
        print(f"Stopping at offline server: {server}")
        break
    print(f"Processing: {server}")
# Output:
# Processing: web-01
# Processing: web-02
# Stopping at offline server: OFFLINE


# continue — skip the rest of this iteration, move to next
services = ["nginx", None, "redis", None, "postgresql"]

for service in services:
    if service is None:
        continue            # Skip None values
    print(f"Starting: {service}")


# pass — do nothing (placeholder for empty blocks)
for server in servers:
    pass    # Will fill in this logic later
```

---

### Subtask 3: Numbers 1 to 10 — Both Loop Types

```python
# subtask_loops.py

print("--- for loop ---")
for i in range(1, 11):
    print(i)

print("\n--- while loop ---")
count = 1
while count <= 10:
    print(count)
    count += 1
```

---

## Task 4: Functions

Functions let you group a block of code under a name and reuse it throughout your script. Instead of copying the same disk-check logic to ten different places, you write it once as a function and call it ten times.

### Defining and Calling Functions

```python
# Define a function
def function_name(parameter1, parameter2):
    """Docstring — describes what this function does."""
    # function body
    return result    # optional — functions can return values or None


# Call a function
result = function_name(value1, value2)
```

---

### Subtask 4: The `greet` Function

```python
# subtask_functions.py

def greet(name):
    """Print a greeting for the given name."""
    print(f"Hello, {name}!")

greet("DevOps")
greet("NexusCorp")
greet("World")
```

---

### Functions in Depth

```python
# Function with a return value
def add_numbers(a, b):
    """Return the sum of a and b."""
    return a + b

total = add_numbers(10, 25)
print(f"Total: {total}")    # Total: 35


# Function with a default parameter value
def check_disk(usage, threshold=80):
    """Return the status of disk usage against a threshold."""
    if usage >= threshold:
        return f"WARNING: Disk at {usage}% (threshold: {threshold}%)"
    return f"OK: Disk at {usage}%"

print(check_disk(87))           # Uses default threshold of 80
print(check_disk(87, 90))       # Override threshold to 90
print(check_disk(usage=65))     # Keyword argument


# Function with multiple return values
def get_server_stats(hostname):
    """Return hostname, status, and a mock CPU value."""
    status = "running"
    cpu = 45.2
    return hostname, status, cpu

host, status, cpu = get_server_stats("web-01")
print(f"{host}: {status}, CPU: {cpu}%")


# Function accepting variable number of arguments
def check_all_services(*services):
    """Check a variable number of services."""
    print(f"Checking {len(services)} services:")
    for service in services:
        print(f"  - {service}: OK")

check_all_services("nginx", "redis", "postgresql")


# Function with keyword arguments (**kwargs)
def create_server_record(**details):
    """Create a server record from keyword arguments."""
    for key, value in details.items():
        print(f"  {key}: {value}")

create_server_record(hostname="web-01", ip="192.168.1.45", env="staging")
```

---

### DevOps Function Patterns

```python
def check_threshold(value, threshold, metric_name):
    """Generic threshold checker — returns status string."""
    if value >= threshold:
        return f"ALERT: {metric_name} is {value}% (threshold: {threshold}%)"
    return f"OK: {metric_name} is {value}%"


def format_server_report(servers):
    """Format a list of server dicts into a report string."""
    lines = ["Server Status Report", "=" * 30]
    for server in servers:
        lines.append(f"  {server['name']}: {server['status']}")
    return "\n".join(lines)


def is_valid_environment(env, valid_envs=None):
    """Return True if env is in the list of valid environments."""
    if valid_envs is None:
        valid_envs = ["development", "staging", "production"]
    return env in valid_envs


# Use them together
print(check_threshold(87, 80, "CPU"))
print(check_threshold(45, 80, "Memory"))
print(is_valid_environment("staging"))     # True
print(is_valid_environment("unknown"))     # False
```

---

## Task 5: Python's Built-in Functions

Python ships with dozens of built-in functions — no import required. These are the ones you will use constantly in DevOps scripts.

### Essential Built-ins

```python
# len() — length of a sequence
servers = ["web-01", "web-02", "db-01"]
print(len(servers))           # 3
print(len("nexuscorp.com"))   # 13

# type() — check the type of a value
print(type(42))               # <class 'int'>
print(type("hello"))          # <class 'str'>
print(type([1, 2, 3]))        # <class 'list'>
print(type({"a": 1}))         # <class 'dict'>

# range() — generate a sequence of numbers
print(list(range(5)))         # [0, 1, 2, 3, 4]
print(list(range(1, 6)))      # [1, 2, 3, 4, 5]
print(list(range(0, 20, 5)))  # [0, 5, 10, 15]

# int(), float(), str() — type conversion
print(int("42"))              # 42
print(float("3.14"))          # 3.14
print(str(100))               # "100"

# input() — get input from the user
# server = input("Enter server name: ")   # Pauses for input

# print() — output with formatting options
print("a", "b", "c")                    # a b c (space separator)
print("a", "b", "c", sep=", ")         # a, b, c
print("Loading", end="...")             # Loading... (no newline)
print()                                 # Newline

# abs() — absolute value
print(abs(-42))               # 42

# max(), min(), sum() — work on collections
values = [73.5, 88.1, 45.2, 91.0]
print(max(values))            # 91.0
print(min(values))            # 45.2
print(sum(values))            # 297.8
print(sum(values) / len(values))  # Average: 74.45

# round() — round a float
print(round(73.567, 2))       # 73.57
print(round(73.567))          # 74

# sorted() — return a sorted copy (does not modify original)
servers = ["web-02", "db-01", "web-01", "cache-01"]
print(sorted(servers))        # ['cache-01', 'db-01', 'web-01', 'web-02']
print(sorted(values, reverse=True))   # Descending

# enumerate() — add index to iteration
for i, server in enumerate(servers, start=1):
    print(f"  {i}. {server}")

# zip() — pair up two collections
names = ["web-01", "db-01", "cache-01"]
statuses = ["running", "stopped", "running"]
for name, status in zip(names, statuses):
    print(f"  {name}: {status}")

# any() and all() — test collections of booleans
checks = [True, True, False, True]
print(any(checks))    # True — at least one is True
print(all(checks))    # False — not all are True

# isinstance() — check type safely
value = 42
if isinstance(value, int):
    print("It's an integer")

# open() — file operations (used constantly in DevOps scripts)
# with open("servers.txt", "r") as f:
#     content = f.read()
```

---

### Subtask 5: `len()` and `type()`

```python
# subtask_builtins.py

name = "DevOps"
age = 30
salary = 1000.50
servers = ["web-01", "web-02", "db-01"]

print(f"Length of name '{name}': {len(name)}")
print(f"Length of servers list: {len(servers)}")

print(f"\nType of name:    {type(name)}")
print(f"Type of age:     {type(age)}")
print(f"Type of salary:  {type(salary)}")
print(f"Type of servers: {type(servers)}")
```

---

## Task 6: Lists and Tuples

### Lists — Mutable, Ordered Collections

```python
# Create a list of DevOps tools
devops_tools = ["Docker", "Kubernetes", "Terraform", "Ansible", "Git"]

# Access items (zero-indexed)
print(devops_tools[0])          # Docker
print(devops_tools[-1])         # Git (last item)
print(devops_tools[1:3])        # ['Kubernetes', 'Terraform']

# Add items
devops_tools.append("Jenkins")          # Add to end
devops_tools.insert(2, "Helm")          # Insert at index 2
devops_tools.extend(["Prometheus", "Grafana"])  # Add multiple

# Remove items
devops_tools.remove("Git")              # Remove by value
removed = devops_tools.pop()            # Remove and return last
removed_at = devops_tools.pop(0)        # Remove and return at index

# Search and check
print("Docker" in devops_tools)         # True
print(devops_tools.index("Helm"))       # Position of "Helm"
print(devops_tools.count("Docker"))     # How many times it appears

# Sort and reverse
devops_tools.sort()                     # Sort in place (alphabetical)
devops_tools.sort(reverse=True)         # Sort descending
devops_tools.reverse()                  # Reverse order in place

# Other useful operations
print(len(devops_tools))                # Count of items
copy = devops_tools.copy()              # Shallow copy
devops_tools.clear()                    # Remove all items
```

---

### Tuples — Immutable, Ordered Collections

```python
# Create a tuple
valid_environments = ("development", "staging", "production")
db_config = ("db.nexuscorp.com", 5432, "nexusdb", "readonly")

# Access items (same as list)
print(valid_environments[0])      # development
print(db_config[-1])              # readonly
print(valid_environments[1:])     # ('staging', 'production')

# Unpack tuple values
host, port, dbname, mode = db_config
print(f"Connecting to {dbname} at {host}:{port} in {mode} mode")

# Membership check
current_env = "staging"
if current_env in valid_environments:
    print(f"Valid environment: {current_env}")

# Count occurrences (tuples support count and index)
print(valid_environments.count("staging"))    # 1
print(valid_environments.index("production")) # 2

# Tuples cannot be modified — these would raise errors:
# valid_environments[0] = "dev"    # TypeError
# valid_environments.append("qa")  # AttributeError

# Convert between list and tuple
as_list = list(valid_environments)
as_tuple = tuple(as_list)
```

---

### List vs Tuple — When to Use Which

| | List | Tuple |
|---|---|---|
| **Syntax** | `[item1, item2]` | `(item1, item2)` |
| **Mutable?** | Yes — add, remove, change | No — fixed after creation |
| **Performance** | Slightly slower | Slightly faster |
| **Use for** | Collections that change | Fixed configuration values |
| **DevOps examples** | Server list, log lines, results | DB config, valid environments, port ranges |
| **Can be dict key?** | No (mutable) | Yes (immutable) |

---

### Subtask 6

```python
# subtask_lists_tuples.py

# List of DevOps tools
devops_tools = ["Docker", "Kubernetes", "Terraform", "Ansible", "Git"]
print("Initial list:", devops_tools)

# Add a tool
devops_tools.append("Jenkins")
print("After append:", devops_tools)

# Remove a tool
devops_tools.remove("Git")
print("After remove:", devops_tools)

# Access items
print("First tool:", devops_tools[0])
print("Last tool:", devops_tools[-1])
print("Tools 2-4:", devops_tools[1:4])

# Tuple — immutable
valid_envs = ("development", "staging", "production")
print("\nValid environments:", valid_envs)
print("First:", valid_envs[0])
print("Can we modify it?")
try:
    valid_envs[0] = "dev"
except TypeError as e:
    print(f"  TypeError: {e}")
```

---

## Task 7: Python's Interactive Help System

Python has a built-in help system accessible directly from the interactive shell. It is one of the most underused features by beginners — knowing how to use it means you can explore any module or function without leaving the terminal.

### Using `help()`

```python
# Enter the Python shell
python3

# Get help on a type — shows all methods and attributes
>>> help(str)           # String methods (very long — press q to exit)
>>> help(list)          # List methods
>>> help(dict)          # Dictionary methods

# Get help on a specific method
>>> help(str.split)
>>> help(list.append)
>>> help(dict.get)

# Get help on a built-in function
>>> help(print)
>>> help(len)
>>> help(sorted)

# Get help on a module (after importing it)
>>> import os
>>> help(os)
>>> help(os.path)

>>> exit()
```

### Using `dir()` — List Available Methods

`dir()` shows everything available on an object — methods, attributes, and special names.

```python
>>> dir(str)       # All string methods
>>> dir([])        # All list methods
>>> dir({})        # All dictionary methods

# Filter to non-special methods (remove dunders)
>>> [m for m in dir(str) if not m.startswith("_")]
# Shows only usable methods: ['capitalize', 'center', 'count', 'encode', ...]
```

### Subtask 7

```python
# In the Python interactive shell:

python3

>>> help(str)
# Read through the available string methods
# Note: upper(), lower(), split(), strip(), replace(), startswith(), endswith()
# Press q to exit help

>>> "hello devops".upper()
>>> "  trim me  ".strip()
>>> "web-01,web-02,db-01".split(",")

>>> exit()
```

---

## Task 8: Exploring Errors

Python raises **exceptions** when something goes wrong. Learning to read error messages is one of the most important debugging skills — Python's error output tells you exactly what went wrong and where.

### Common Exception Types

| Exception | Caused By | Example |
|---|---|---|
| `SyntaxError` | Invalid Python syntax | Missing colon, unclosed bracket |
| `NameError` | Using a variable before defining it | `print(undefined_var)` |
| `TypeError` | Wrong type for an operation | `"hello" + 5` |
| `ValueError` | Right type, wrong value | `int("hello")` |
| `IndexError` | Accessing a list index that doesn't exist | `servers[99]` on a 3-item list |
| `KeyError` | Accessing a dict key that doesn't exist | `server["missing_key"]` |
| `ZeroDivisionError` | Dividing by zero | `10 / 0` |
| `FileNotFoundError` | Opening a file that doesn't exist | `open("missing.txt")` |
| `AttributeError` | Calling a method that doesn't exist on a type | `(42).upper()` |

---

### Reading a Python Traceback

When an error occurs, Python prints a **traceback** — a full description of what went wrong and where.

```
Traceback (most recent call last):
  File "check_servers.py", line 14, in check_health
    result = servers[index]
IndexError: list index out of range
```

**Reading it:**
- **`Traceback (most recent call last)`** — read from the bottom up; the last line is the actual error
- **`File "check_servers.py", line 14`** — which file and which line
- **`in check_health`** — which function
- **`result = servers[index]`** — the exact line that failed
- **`IndexError: list index out of range`** — the exception type and message

---

### Subtask 8: Deliberately Trigger Errors

```python
# Open the Python shell and trigger each error type:
python3

# SyntaxError — missing closing parenthesis
>>> print("Hello"
  File "<stdin>", line 1
    print("Hello"
                 ^
SyntaxError: '(' was never closed

# NameError — variable not defined
>>> print(undefined_variable)
NameError: name 'undefined_variable' is not defined

# TypeError — incompatible types
>>> "Server count: " + 42
TypeError: can only concatenate str (not "int") to str
# Fix: "Server count: " + str(42)   or   f"Server count: {42}"

# IndexError — list index out of range
>>> servers = ["web-01", "web-02"]
>>> servers[10]
IndexError: list index out of range

# KeyError — dict key does not exist
>>> config = {"host": "localhost"}
>>> config["port"]
KeyError: 'port'
# Fix: config.get("port", 5432)   — returns default if key missing

# ValueError — wrong value for the type
>>> int("not-a-number")
ValueError: invalid literal for int() with base 10: 'not-a-number'

# ZeroDivisionError
>>> 100 / 0
ZeroDivisionError: division by zero

>>> exit()
```

---

### Handling Errors Gracefully with `try/except`

Once you understand what errors look like, you can handle them so your script doesn't crash:

```python
# Without error handling — crashes on bad input
disk_usage = int(input("Enter disk usage: "))   # crashes if user types "abc"

# With error handling — recovers gracefully
try:
    disk_usage = int(input("Enter disk usage percentage: "))
    if disk_usage > 80:
        print("WARNING: High disk usage")
    else:
        print(f"OK: Disk at {disk_usage}%")
except ValueError:
    print("ERROR: Please enter a valid number")
except Exception as e:
    print(f"Unexpected error: {e}")
finally:
    print("Check complete.")    # Always runs
```

---

## 🔧 Mini-Project: NexusCorp Server Health Checker

Combine everything from today into one practical script.

```python
#!/usr/bin/env python3
"""
health_checker.py
NexusCorp Server Health Checker
Demonstrates: variables, control structures, loops, functions,
              built-ins, lists, tuples, and error handling.
"""

# ── Constants (tuple — these should not change) ────────────────
VALID_ENVIRONMENTS = ("development", "staging", "production")
CPU_WARNING_THRESHOLD = 75
CPU_CRITICAL_THRESHOLD = 90
DISK_WARNING_THRESHOLD = 80

# ── Sample data (list of dicts — real scripts fetch this via API/SSH)
SERVERS = [
    {"name": "web-01", "env": "production", "cpu": 45.2, "disk": 62},
    {"name": "web-02", "env": "production", "cpu": 88.7, "disk": 78},
    {"name": "db-01",  "env": "production", "cpu": 92.1, "disk": 91},
    {"name": "cache-01","env": "staging",   "cpu": 33.4, "disk": 45},
    {"name": "dev-01", "env": "development","cpu": 12.0, "disk": 30},
]

# ── Functions ──────────────────────────────────────────────────
def check_cpu(server_name, cpu_usage):
    """Return a status string based on CPU thresholds."""
    if cpu_usage >= CPU_CRITICAL_THRESHOLD:
        return f"CRITICAL"
    elif cpu_usage >= CPU_WARNING_THRESHOLD:
        return f"WARNING"
    else:
        return "OK"


def check_disk(server_name, disk_usage):
    """Return a status string based on disk threshold."""
    if disk_usage >= DISK_WARNING_THRESHOLD:
        return "WARNING"
    return "OK"


def is_valid_environment(env):
    """Return True if environment is in the valid list."""
    return env in VALID_ENVIRONMENTS


def generate_report(servers):
    """Generate and return a health report for all servers."""
    report_lines = []
    alerts = []

    report_lines.append("=" * 55)
    report_lines.append("  NexusCorp Server Health Report")
    report_lines.append("=" * 55)

    for server in servers:
        name = server["name"]
        env = server["env"]
        cpu = server["cpu"]
        disk = server["disk"]

        # Validate environment
        if not is_valid_environment(env):
            report_lines.append(f"  {name}: UNKNOWN ENVIRONMENT ({env})")
            continue

        cpu_status = check_cpu(name, cpu)
        disk_status = check_disk(name, disk)

        line = (f"  {name:<12} | {env:<12} | "
                f"CPU: {cpu:>5.1f}% [{cpu_status:<8}] | "
                f"Disk: {disk:>3}% [{disk_status}]")
        report_lines.append(line)

        if cpu_status != "OK" or disk_status != "OK":
            alerts.append(f"  ⚠  {name}: CPU={cpu_status}, DISK={disk_status}")

    return report_lines, alerts


# ── Main ───────────────────────────────────────────────────────
def main():
    report_lines, alerts = generate_report(SERVERS)

    for line in report_lines:
        print(line)

    print("=" * 55)
    print(f"  Total servers checked : {len(SERVERS)}")

    cpu_values = [s["cpu"] for s in SERVERS]
    print(f"  Average CPU usage     : {round(sum(cpu_values) / len(cpu_values), 1)}%")
    print(f"  Highest CPU           : {max(cpu_values)}%")
    print(f"  Alerts raised         : {len(alerts)}")
    print("=" * 55)

    if alerts:
        print("\n  ALERTS:")
        for alert in alerts:
            print(alert)
    else:
        print("\n  All systems healthy.")


if __name__ == "__main__":
    main()
```

```bash
python3 health_checker.py
```

**Expected output:**
```
=======================================================
  NexusCorp Server Health Report
=======================================================
  web-01       | production   | CPU:  45.2% [OK      ] | Disk:  62% [OK]
  web-02       | production   | CPU:  88.7% [WARNING ] | Disk:  78% [OK]
  db-01        | production   | CPU:  92.1% [CRITICAL] | Disk:  91% [WARNING]
  cache-01     | staging      | CPU:  33.4% [OK      ] | Disk:  45% [OK]
  dev-01       | development  | CPU:  12.0% [OK      ] | Disk:  30% [OK]
=======================================================
  Total servers checked : 5
  Average CPU usage     : 54.3%
  Highest CPU           : 92.1%
  Alerts raised         : 2
=======================================================

  ALERTS:
  ⚠  web-02: CPU=WARNING, DISK=OK
  ⚠  db-01: CPU=CRITICAL, DISK=WARNING
```

**Extend it:**
- Add a `memory` field to each server and a `check_memory()` function
- Add user input to filter by environment: `which environment to check?`
- Sort the output by CPU usage descending using `sorted()`

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Variable** | A named container holding a value that can be referenced and changed |
| **Assignment** | The act of giving a variable a value using `=` |
| **`snake_case`** | Python naming convention — words separated by underscores |
| **Control structure** | A statement that directs the flow of execution: `if`, `for`, `while` |
| **`if` / `elif` / `else`** | Conditional branching — runs different code depending on whether conditions are True |
| **Comparison operator** | Operators that compare values: `==`, `!=`, `>`, `<`, `>=`, `<=` |
| **Logical operator** | `and`, `or`, `not` — combine or invert boolean conditions |
| **`for` loop** | Iterates over each item in a sequence |
| **`while` loop** | Repeats while a condition is True |
| **`break`** | Exits the current loop immediately |
| **`continue`** | Skips the rest of the current iteration and moves to the next |
| **`pass`** | Does nothing — used as a placeholder in empty code blocks |
| **Function** | A named, reusable block of code defined with `def` |
| **Parameter** | A variable in the function definition that receives an argument |
| **Argument** | The actual value passed to a function when calling it |
| **`return`** | Sends a value back from a function to the caller |
| **Default parameter** | A parameter with a pre-set value used when no argument is supplied |
| **`*args`** | Allows a function to accept any number of positional arguments |
| **`**kwargs`** | Allows a function to accept any number of keyword arguments |
| **Built-in function** | A function provided by Python itself — `len()`, `print()`, `type()`, `range()` |
| **`help()`** | Built-in function that displays documentation for any object, module, or function |
| **`dir()`** | Returns a list of all attributes and methods available on an object |
| **Exception** | An error raised during program execution that interrupts the normal flow |
| **Traceback** | The error output Python prints, showing the chain of calls that led to the exception |
| **`try/except`** | Error handling block — `try` runs code, `except` catches exceptions |
| **`finally`** | Block in a try/except that always runs, regardless of whether an exception occurred |
| **`enumerate()`** | Built-in that adds an index counter to an iterable |
| **`zip()`** | Built-in that pairs items from two or more iterables |
| **`sorted()`** | Returns a new sorted list without modifying the original |

---

## 📚 Resources

- [Python Official Tutorial — Control Flow](https://docs.python.org/3/tutorial/controlflow.html) — official guide to if, for, while, and functions
- [Python Built-in Functions Reference](https://docs.python.org/3/library/functions.html) — complete list with examples
- [Automate the Boring Stuff — Chapter 2: Flow Control](https://automatetheboringstuff.com/2e/chapter2/) — free, practical control flow guide
- [Real Python — Python for Loops](https://realpython.com/python-for-loop/) — in-depth loop reference
- [Real Python — Defining Functions](https://realpython.com/defining-your-own-python-function/) — comprehensive functions guide
- [PEP 8 — Python Style Guide](https://peps.python.org/pep-0008/) — official naming and formatting conventions

---

## 🔭 Day 23 Preview: Python File I/O, Modules & Working with Data

With control flow and functions established, Day 23 moves into how Python interacts with the real world — reading and writing files, working with JSON and YAML, and importing modules.

You will learn:
- Reading and writing files with `open()` and `with`
- Parsing JSON — the format of almost every API response and config file
- Parsing YAML — the format of Kubernetes manifests, Ansible playbooks, and Docker Compose
- Importing and using standard library modules: `os`, `sys`, `json`, `datetime`, `pathlib`
- A practical project: reading a server inventory from a JSON file and generating a formatted report

Every DevOps script that does real work reads from or writes to files, APIs, or both. Day 23 is where your scripts start interacting with real data.

---

> 💬 *"Variables are memory. Control structures are decisions. Loops are repetition. Functions are reuse. Together, they are everything any program has ever done."*
>
> — DevOps Learning by Yukta
