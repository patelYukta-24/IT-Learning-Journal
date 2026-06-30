# Day 6: Permissions & Ownership

> **DevOps Transformation** | Linux Fundamentals Series

---

## 📋 Overview

Yesterday you learned how to create, move, copy, and edit files. Today you learn who gets to *use* them.

Linux's permission system is one of its most powerful security features. Every file and directory has a defined owner, a group, and a precise set of access rules. Get this wrong on a production server and you're either locking out legitimate users or leaving the door open to attackers.

At NexusCorp, a misconfigured deployment script was once left world-writable on a staging server. By the end of today, you'll know exactly why that's dangerous — and how to fix it in seconds.

---

## 🗺️ What You'll Learn Today

| Topic | Commands / Concepts | Why It Matters |
|---|---|---|
| User Roles & Groups | users, groups, root | Foundation of Linux access control |
| File Permissions | `r`, `w`, `x`, `ls -l` | Understand what every permission bit means |
| Changing Permissions | `chmod` | Control who can read, write, or execute |
| Changing Ownership | `chown`, `chgrp` | Control who *owns* files and directories |
| Special Permissions | SUID, SGID, Sticky Bit | Advanced permission flags for shared environments |

---

## 🏢 NexusCorp Scenario

> NexusCorp's DevOps team is onboarding three new developers. Each needs access to shared project directories but must not be able to overwrite each other's work. Meanwhile, the deployment scripts that run as a specific service account must not be editable by the general team. The ops lead has five minutes to configure all of this correctly before the sprint starts.
>
> This is what they did — and what you'll learn today.

---

## Part 1: Understanding User Roles, Groups, and Permissions

### The Linux Identity Model

Every process and file in Linux operates with an **identity**. That identity determines what it can access and what it can change.

```
┌─────────────────────────────────────────────┐
│              Linux Identity Model            │
├───────────────┬─────────────────────────────┤
│  Root User    │ Superuser. Unrestricted      │
│               │ access to everything.        │
├───────────────┼─────────────────────────────┤
│  Regular User │ Named account with its own   │
│               │ home directory and UID.      │
├───────────────┼─────────────────────────────┤
│  Group        │ A named collection of users  │
│               │ sharing permissions.         │
└───────────────┴─────────────────────────────┘
```

**Key identifiers:**
```bash
# See who you are
whoami

# See your user ID, group ID, and group memberships
id

# See all logged-in users
who

# See users on the system
cat /etc/passwd

# See groups on the system
cat /etc/group
```

---

### The Root User

`root` is the superuser — the administrative account with unrestricted access to every file, process, and system setting. There are no permission checks for root.

```bash
# Run a single command as root (safer than switching to root)
sudo chmod 600 /etc/secret.conf

# Switch to root shell (use sparingly)
sudo su -

# Check if you can use sudo
sudo -l
```

**🚨 Root Safety Rule:** Never work as root day-to-day. Use `sudo` only for specific commands that require it. A typo as root can destroy a system; a typo as a regular user is contained.

**💡 NexusCorp Rule:** All deployment scripts are run via `sudo` with specific command whitelisting — not as a full root shell.

---

### The Three Permission Categories

Every file and directory has permissions defined for three groups of people:

| Category | Symbol | Meaning |
|---|---|---|
| **User** (owner) | `u` | The account that owns the file |
| **Group** | `g` | Members of the file's assigned group |
| **Others** | `o` | Everyone else on the system |
| **All** | `a` | User + Group + Others combined |

---

### The Three Permission Types

For each of those three categories, three types of access can be granted or denied:

| Permission | Symbol | Octal | On a **File** | On a **Directory** |
|---|---|---|---|---|
| **Read** | `r` | `4` | View file contents | List directory contents (`ls`) |
| **Write** | `w` | `2` | Modify or delete the file | Create, rename, delete files inside |
| **Execute** | `x` | `1` | Run the file as a program | Enter the directory (`cd`) |
| **None** | `-` | `0` | No access | No access |

> **Important distinction:** Execute on a directory means you can `cd` into it. Without `x` on a directory, even if you have `r`, you cannot access its contents — you can see the filenames but not open them.

---

### Reading Permission Strings with `ls -l`

```bash
ls -l
```

**Sample output:**
```
-rwxr-xr-- 1 yukta devops 2048 Jun 10 14:32 deploy.sh
drwxr-x--- 2 yukta devops 4096 Jun 10 14:30 configs/
```

**Breaking down `-rwxr-xr--`:**

```
- rwx r-x r--
│ │   │   │
│ │   │   └── Others:  read only
│ │   └────── Group:   read + execute
│ └────────── User:    read + write + execute
└──────────── File type: - = file, d = directory, l = symlink
```

**The full field breakdown:**
```
-rwxr-xr-- 1   yukta   devops  2048  Jun 10 14:32  deploy.sh
│           │   │       │       │     │              │
│           │   │       │       │     │              └── Filename
│           │   │       │       │     └── Last modified
│           │   │       │       └── File size (bytes)
│           │   │       └── Group owner
│           │   └── User owner
│           └── Number of hard links
└── Permission string (10 characters)
```

**Quick examples:**
```
-rw-r--r--   Standard file: owner can read/write; group and others read-only
-rwxrwxrwx   Fully open: everyone can read, write, execute (dangerous!)
drwxr-xr-x   Standard directory: owner full access; group/others can list and enter
-rw-------   Private file: only owner can read/write
----------   No access for anyone (including owner!)
```

---

## Part 2: `chmod` — Changing Permissions

`chmod` (**ch**ange **mod**e) modifies the permission bits of a file or directory. There are two ways to specify permissions: **symbolic** and **octal**.

---

### Method 1: Symbolic Format

Symbolic format is readable and great for targeted changes.

**Syntax:**
```bash
chmod [who][operator][permissions] file
```

| Who | Operator | Permission |
|---|---|---|
| `u` = user | `+` = add | `r` = read |
| `g` = group | `-` = remove | `w` = write |
| `o` = others | `=` = set exactly | `x` = execute |
| `a` = all | | |

**Examples:**
```bash
# Add execute permission for the owner
chmod u+x script.sh

# Remove write permission from group
chmod g-w config.conf

# Give read and execute to others
chmod o+rx public_dir/

# Set group permissions to read-only (exactly, overwriting current)
chmod g=r file.txt

# Remove all permissions from others
chmod o-rwx private.txt

# Add execute for everyone
chmod a+x deploy.sh

# Multiple changes at once (comma-separated)
chmod u+x,g-w,o-rwx sensitive.sh
```

---

### Method 2: Octal (Numeric) Format

Octal format uses numbers. Each permission is a power of 2, and the three digits represent user, group, and others respectively.

**Permission values:**
```
r = 4
w = 2
x = 1
- = 0
```

**Calculate by adding the values:**
```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0
```

**Common permission patterns:**

| Octal | Symbolic | Meaning | Typical Use |
|---|---|---|---|
| `777` | `rwxrwxrwx` | Full access for all | ⚠️ Almost never appropriate |
| `755` | `rwxr-xr-x` | Owner: full; others: read+execute | Executables, public directories |
| `750` | `rwxr-x---` | Owner: full; group: read+execute; others: none | Team scripts |
| `644` | `rw-r--r--` | Owner: read+write; others: read | Config files, HTML files |
| `640` | `rw-r-----` | Owner: read+write; group: read; others: none | Sensitive configs |
| `600` | `rw-------` | Owner: read+write only | Private keys, passwords |
| `400` | `r--------` | Owner: read only | Read-only credentials |
| `000` | `----------` | No access for anyone | Effectively locked |

**Examples:**
```bash
# Standard script (owner executes, group/others read+execute)
chmod 755 deploy.sh

# Private config file (owner read+write only)
chmod 600 .env

# Public web file (owner read+write, everyone read)
chmod 644 index.html

# Locked down private key (SSH requirement — 600 or stricter)
chmod 600 ~/.ssh/id_rsa

# Full access (almost never use this in production)
chmod 777 script.sh
```

---

### Recursive Permission Changes

```bash
# Apply permissions to a directory and everything inside it
chmod -R 755 /var/www/html/

# Apply to directories only (using find)
find /var/www -type d -exec chmod 755 {} \;

# Apply to files only (using find)
find /var/www -type f -exec chmod 644 {} \;
```

**⚠️ Warning:** `chmod -R 777` on any system directory is one of the most dangerous commands you can run. Always be specific about what you're changing recursively.

---

### Permissions: Too Loose vs Too Strict

| Scenario | Problem |
|---|---|
| `777` on a config file | Any user can overwrite it — attackers or accidents can modify system behavior |
| `777` on a script | Any user can modify what the script does before it runs |
| `000` on your own file | You lock yourself out — need root or `chmod` again to recover |
| `644` on a script | Script won't execute (no `x` bit) — common mistake when deploying |
| `600` on a shared config | Team members can't read it — legitimate work is blocked |

**📎 Resource:** [chmod Calculator](https://chmod-calculator.com) — interactively see how permission settings map between octal and symbolic formats.

---

### 🛠️ Practical Exercise 1: chmod

```bash
# --- Part A: private.txt ---

# Step 1: Create the file
touch private.txt

# Step 2: Check current permissions
ls -l private.txt

# Step 3: Remove all permissions for group and others
chmod go-rwx private.txt
# Or equivalently:
chmod 700 private.txt

# Verify
ls -l private.txt
# Expected: -rwx------

# Step 4: Make it fully open
chmod 777 private.txt
ls -l private.txt
# Expected: -rwxrwxrwx

# Step 5: Restrict to read-only for the user
chmod 400 private.txt
ls -l private.txt
# Expected: -r--------


# --- Part B: shared_folder ---

# Step 1: Create the directory
mkdir shared_folder

# Step 2: Give read and execute to group and others
chmod 755 shared_folder
# Or equivalently:
chmod a+rx shared_folder

# Verify
ls -ld shared_folder
# Expected: drwxr-xr-x
```

**Reflection Questions:**
1. After setting `private.txt` to `400`, can you delete it? Why or why not?
2. What's the difference between `chmod 755 mydir` and `chmod -R 755 mydir`?
3. Why does a shell script need the `x` bit to run, but a Python file run with `python3 script.py` doesn't?

---

## Part 3: `chown` — Changing Ownership

`chown` (**ch**ange **own**er) changes who owns a file or directory. Ownership determines which user and group the permission bits apply to.

**Syntax:**
```bash
chown [options] user[:group] file(s)
```

**Examples:**
```bash
# Change the owner to 'alice'
chown alice file.txt

# Change the owner to 'alice' and group to 'devops'
chown alice:devops file.txt

# Change only the group (note the colon prefix)
chown :devops file.txt

# Change ownership recursively (entire directory tree)
chown -R alice:devops /var/www/nexuscorp/

# Change ownership and report each change (-v)
chown -v bob:ops config.conf

# Reference another file's ownership (copy from)
chown --reference=reference_file.txt target_file.txt
```

**Useful Flags:**

| Flag | Description | Example |
|---|---|---|
| `-R` | Recursive — change ownership of all files in a directory | `chown -R www-data:www-data /var/www/` |
| `-v` | Verbose — report each file changed | `chown -v alice file.txt` |
| `--reference=FILE` | Use another file's ownership as the template | `chown --reference=template.conf new.conf` |

**⚠️ Requires Privileges:** Changing ownership almost always requires `sudo`. Regular users can only transfer ownership away from themselves — and only if the target user also allows it. In practice, always use `sudo chown`.

```bash
sudo chown alice:devops /etc/nexuscorp/app.conf
```

---

### `chgrp` — Changing Group Ownership

`chgrp` is a dedicated command for changing just the group (equivalent to `chown :group`):

```bash
# Change group to 'devops'
chgrp devops project/

# Recursive group change
chgrp -R devops /opt/nexuscorp/

# Verify
ls -l project/
```

---

### 🛠️ Practical Exercise 2: chown

```bash
# Step 1: Create ownership.txt
touch ownership.txt
ls -l ownership.txt

# Step 2: Change its owner (replace 'anotheruser' with a real user on your system)
# List available users first:
cut -d: -f1 /etc/passwd

# Change ownership (requires sudo)
sudo chown anotheruser ownership.txt
ls -l ownership.txt

# Step 3: Create ownership_folder
mkdir ownership_folder

# Step 4: Change its group ownership
sudo chown :devops ownership_folder
# If 'devops' group doesn't exist, create it first:
# sudo groupadd devops

ls -ld ownership_folder
```

**Reflection Questions:**
1. What happens when you try to `chown` a file without `sudo`?
2. Why would a web server's files be owned by `www-data:www-data`?
3. When would you use `chown -R` versus running `chown` on individual files?

---

## Part 4: Special Permissions — SUID, SGID, and Sticky Bit

Beyond the standard `rwx` bits, Linux has three special permission flags for advanced access control scenarios. These are powerful — and potentially dangerous if misapplied.

---

### The Fourth Octal Digit

Standard permissions use three octal digits (e.g., `755`). Special permissions add a **fourth digit** at the front:

```
  4   7   5   5
  │   │   │   │
  │   │   │   └── Others permissions
  │   │   └────── Group permissions
  │   └────────── User permissions
  └────────────── Special bits: SUID=4, SGID=2, Sticky=1
```

---

### 1. SUID — Set User ID Upon Execution

When the SUID bit is set on an **executable file**, the program runs with the **file owner's privileges**, not the calling user's privileges.

**How to identify:** An `s` appears in the user execute position.
```
-rwsr-xr-x   ← SUID set (lowercase 's' = execute also set)
-rwSr-xr-x   ← SUID set but execute NOT set (uppercase 'S' = warning sign)
```

**Real-world example — `/usr/bin/passwd`:**
```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root 59640 ... /usr/bin/passwd
```
Regular users need to change their own password, which requires writing to `/etc/shadow` (owned by root). SUID lets `passwd` run as root temporarily — only for that specific operation.

**Setting SUID:**
```bash
# Symbolic
chmod u+s script.sh

# Octal (4 prefix)
chmod 4755 script.sh

# Verify
ls -l script.sh
# -rwsr-xr-x
```

**🚨 SUID Danger:** A SUID binary that runs as root and has a vulnerability can be exploited to gain root access. Never set SUID on scripts (shell, Python, etc.) — only compiled binaries, and only when absolutely necessary.

---

### 2. SGID — Set Group ID Upon Execution

SGID behaves differently depending on whether it's applied to a **file** or a **directory**:

**On a file:** The program runs with the **group owner's privileges** instead of the calling user's group.

**On a directory (more commonly used):** Any **new files created inside** inherit the **directory's group** automatically, rather than the creating user's primary group.

**How to identify:** An `s` appears in the group execute position.
```
drwxrwsr-x   ← SGID on directory (lowercase 's')
-rwxr-sr-x   ← SGID on file
```

**Real-world example — shared project directory:**
```bash
# Create a shared directory for the devops team
mkdir /opt/nexuscorp/shared
chown :devops /opt/nexuscorp/shared
chmod 2775 /opt/nexuscorp/shared    # 2 = SGID

# Now any file created inside inherits the 'devops' group
# regardless of which team member creates it
```

**Setting SGID:**
```bash
# Symbolic
chmod g+s shared_dir/

# Octal (2 prefix)
chmod 2755 shared_dir/

# Verify
ls -ld shared_dir/
# drwxr-sr-x
```

---

### 3. Sticky Bit — Protecting Shared Directories

The Sticky Bit, when set on a **directory**, restricts file deletion: **only the file owner, the directory owner, or root** can delete files inside — even if others have write permission on the directory.

**How to identify:** A `t` appears in the others execute position.
```
drwxrwxrwt   ← Sticky bit set (lowercase 't' = execute also set)
drwxrwxrwT   ← Sticky bit set but execute NOT set (uppercase 'T')
```

**Real-world example — `/tmp`:**
```bash
ls -ld /tmp
# drwxrwxrwt 1 root root 4096 Jun 10 14:00 /tmp
```
Everyone can write to `/tmp`, but you can't delete another user's files there. The sticky bit makes this safe.

**Setting the Sticky Bit:**
```bash
# Symbolic
chmod o+t sticky_folder/

# Octal (1 prefix)
chmod 1777 sticky_folder/

# Verify
ls -ld sticky_folder/
# drwxrwxrwt
```

---

### Special Permissions Summary

| Special Bit | Octal | Applies To | Effect |
|---|---|---|---|
| **SUID** | `4xxx` | Executable files | Runs as the file's **owner** |
| **SGID** | `2xxx` | Files | Runs as the file's **group** |
| **SGID** | `2xxx` | Directories | New files inherit directory's **group** |
| **Sticky Bit** | `1xxx` | Directories | Only owner can delete their own files |

**Combined examples:**
```bash
chmod 4755 binary      # SUID + standard executable
chmod 2775 shared_dir  # SGID + group-writable directory
chmod 1777 public_dir  # Sticky + world-writable
chmod 6755 binary      # SUID + SGID combined
```

---

### Where Special Bits Are Used in the Wild

| Location | Special Bit | Why |
|---|---|---|
| `/usr/bin/passwd` | SUID | Needs root access to modify `/etc/shadow` |
| `/usr/bin/sudo` | SUID | Must run as root to grant elevated privileges |
| `/tmp` | Sticky Bit | World-writable but protected from cross-user deletion |
| `/var/tmp` | Sticky Bit | Same as `/tmp` but persists across reboots |
| Shared project dirs | SGID | All team members' files share the same group |

---

### 🛠️ Practical Exercise 3: Special Permissions

```bash
# --- Sticky Bit ---

# Step 1: Create sticky_folder
mkdir sticky_folder

# Step 2: Make it world-writable and set the sticky bit
chmod 1777 sticky_folder

# Step 3: Verify
ls -ld sticky_folder
# Expected: drwxrwxrwt


# --- SGID on a Directory ---

# Step 1: Create a shared directory
mkdir sgid_dir

# Step 2: Set group ownership and SGID
sudo chown :devops sgid_dir   # Replace 'devops' with an existing group
chmod 2775 sgid_dir

# Step 3: Create a file inside
touch sgid_dir/testfile.txt

# Step 4: Observe the group ownership of the new file
ls -l sgid_dir/testfile.txt
# The group should be 'devops' (inherited from directory)


# --- SUID on a Script ---

# Step 1: Create a test script
cat > special_script.sh << 'EOF'
#!/bin/bash
echo "Running as user: $(whoami)"
echo "Effective user ID: $(id -u)"
EOF

# Step 2: Make it executable
chmod 755 special_script.sh

# Step 3: Run it normally
./special_script.sh

# Step 4: Set SUID
chmod u+s special_script.sh
ls -l special_script.sh
# -rwsr-xr-x

# Step 5: Note — SUID on shell scripts is ignored by Linux kernel for security
# To observe SUID behavior, it must be on a compiled binary
# This exercise demonstrates the syntax; real SUID testing requires compiled C programs
```

**Reflection Questions:**
1. Why doesn't SUID work on shell scripts in Linux?
2. If `/tmp` didn't have the Sticky Bit, what could a malicious user do?
3. When would SGID on a directory be preferable to managing group permissions manually on each file?

---

## 🔧 Mini-Project: NexusCorp Permission Lockdown

**Scenario:** NexusCorp's staging server has a permission audit coming up. The ops lead has flagged three problems:
1. The deployment script is world-writable
2. The shared config directory doesn't enforce group file ownership
3. The `/tmp/nexus_builds` working directory allows cross-user file deletion

Fix all three.

```bash
# ─── Problem 1: Lock down the deployment script ───

# Simulate a badly configured script
touch deploy.sh
chmod 777 deploy.sh
echo "#!/bin/bash" >> deploy.sh
echo "echo 'Deploying NexusCorp...'" >> deploy.sh

# Audit current state
ls -l deploy.sh
# -rwxrwxrwx  ← Anyone can modify this!

# Fix: owner executes, group reads+executes, others have no access
chmod 750 deploy.sh
ls -l deploy.sh
# -rwxr-x---  ✓


# ─── Problem 2: SGID on shared config directory ───

mkdir shared_configs
sudo chown :devops shared_configs   # Adjust group as needed
chmod 2775 shared_configs

# Create a file as current user — it should inherit 'devops' group
touch shared_configs/app.conf
ls -l shared_configs/app.conf
# Group should show 'devops'


# ─── Problem 3: Sticky bit on build workspace ───

mkdir -p /tmp/nexus_builds
chmod 1777 /tmp/nexus_builds

ls -ld /tmp/nexus_builds
# drwxrwxrwt  ✓ World-writable but deletion-protected


# ─── Final Audit Report ───

echo "=== NexusCorp Permission Audit ==="
echo ""
echo "deploy.sh:"
ls -l deploy.sh

echo ""
echo "shared_configs/:"
ls -ld shared_configs/
ls -l shared_configs/app.conf

echo ""
echo "/tmp/nexus_builds/:"
ls -ld /tmp/nexus_builds/
```

**Bonus Challenges:**
- Use `find` to locate any world-writable files in your home directory: `find ~ -perm -o+w`
- Create a private SSH key directory with correct permissions: `mkdir -m 700 ~/.ssh_test`
- Check which SUID binaries exist on your system: `find / -perm -4000 -type f 2>/dev/null`

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Root User** | The superuser account with unrestricted access to everything on the system |
| **User** | A named Linux account with its own UID, home directory, and permission scope |
| **Group** | A named collection of users who share a set of permissions |
| **Read (`r`)** | Permission to view file contents, or list directory contents |
| **Write (`w`)** | Permission to modify a file, or create/delete files inside a directory |
| **Execute (`x`)** | Permission to run a file as a program, or `cd` into a directory |
| **Octal Format** | A three-digit (or four-digit) numeric system for expressing permissions (e.g., `755`) |
| **Symbolic Format** | A letter-based system for expressing permissions (e.g., `u+x`, `go-rwx`) |
| **SUID** | Set User ID — causes an executable to run as its owner's identity, not the caller's |
| **SGID** | Set Group ID — on files: runs as group owner; on directories: new files inherit group |
| **Sticky Bit** | When set on a directory, only the file owner can delete their own files |
| **`chmod`** | Command to change file or directory permission bits |
| **`chown`** | Command to change the user and/or group owner of a file or directory |
| **`chgrp`** | Command to change only the group owner of a file or directory |
| **UID** | User ID — the numeric identifier assigned to each user |
| **GID** | Group ID — the numeric identifier assigned to each group |
| **`sudo`** | Execute a command as another user (typically root) with logged authorization |
| **World-Writable** | A file or directory where others (`o`) have write permission — a security risk |
| **Umask** | A mask that determines default permissions for newly created files and directories |

---

## 📚 Resources

- [chmod Calculator](https://chmod-calculator.com) — interactive tool to convert between octal and symbolic permissions
- [chmod Command Guide (Linux Man Page)](https://man7.org/linux/man-pages/man1/chmod.1.html) — official reference with all flags and examples
- [Linux File Permissions Explained — Red Hat](https://www.redhat.com/sysadmin/linux-file-permissions-explained) — clear walkthrough with diagrams
- [Understanding SUID, SGID, and Sticky Bit](https://www.redhat.com/sysadmin/suid-sgid-sticky-bit) — Red Hat deep-dive on special permissions
- [Linux Users and Groups — Arch Wiki](https://wiki.archlinux.org/title/Users_and_groups) — comprehensive reference

---

## 🔭 Day 7 Preview: Processes & System Monitoring

Tomorrow you learn how to see what's actually running on your system and how to control it.

You'll learn:
- `ps`, `top`, `htop` — view and monitor running processes
- `kill`, `pkill` — stop processes gracefully or forcefully
- `jobs`, `bg`, `fg` — manage foreground and background processes
- Understanding process states, PIDs, and signals
- How NexusCorp's on-call team detects and kills runaway processes before they take down a service

Permissions told the system *who* can do *what*. Process management tells you *what's actually happening* right now.

---

> 💬 *"On a Linux system, permissions are promises. `chmod` is how you keep them."*
>
> — DevOps learning by Yukta
