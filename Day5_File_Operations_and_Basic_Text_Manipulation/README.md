# Day 5: File Operations & Basic Text Manipulation

> **DevOps Transformation** | Linux Fundamentals Series

---

## 📋 Overview

At NexusCorp, the ops team spends a huge chunk of their day doing things that sound deceptively simple: creating directories for new deployments, copying config files, editing server scripts, and cleaning up stale logs. Today you learn the commands that make all of that possible — the **basic verbs of Linux**.

By the end of Day 5, you will be able to confidently navigate, create, organize, and edit files and directories from the command line — without ever touching a GUI.

---

## 🗺️ What You'll Learn Today

| Topic | Commands | Why It Matters |
|---|---|---|
| File Operations | `touch`, `mkdir`, `cp`, `mv`, `rm` | Create, organize, copy, rename, and delete files |
| Basic Text Manipulation | `cat`, `echo`, `nano`, `vi` | Read, write, and edit plain-text files |

---

## 🏢 NexusCorp Scenario

> NexusCorp's infrastructure team just received a new deployment checklist. They need to set up directory structures for a staging environment, copy config templates, and edit a startup script — all over SSH. No file managers, no drag-and-drop. Just the terminal and a deadline.
>
> Today you learn how they do it.

---

## Part 1: File Operations

### The Big 5 Commands

These five commands are your daily toolkit. Learn them deeply — they appear in almost every script and workflow you'll ever write.

---

### 1. `touch` — Create Empty Files / Update Timestamps

`touch` creates a new empty file if it doesn't exist. If the file already exists, it updates the file's access and modification timestamps without changing the content.

**Syntax:**
```bash
touch [options] filename(s)
```

**Examples:**
```bash
# Create a single empty file
touch notes.txt

# Create multiple files at once
touch file1.txt file2.txt file3.txt

# Create files in a specific directory
touch /tmp/logs/error.log

# Update the timestamp of an existing file (useful in build systems)
touch Makefile
```

**Useful Flags:**

| Flag | Description | Example |
|---|---|---|
| `-a` | Change only the access time | `touch -a file.txt` |
| `-m` | Change only the modification time | `touch -m file.txt` |
| `-t` | Set a specific timestamp | `touch -t 202401011200 file.txt` |

**💡 NexusCorp Use Case:** Creating placeholder lock files (`.lock`) to signal that a deployment process is running.

---

### 2. `mkdir` — Make Directories

`mkdir` creates one or more directories. The most important flag to know is `-p` (parents), which lets you create nested directories in one command.

**Syntax:**
```bash
mkdir [options] directory_name(s)
```

**Examples:**
```bash
# Create a single directory
mkdir deployments

# Create multiple directories at once
mkdir staging production backup

# Create nested directories (without -p this would fail if 'projects' doesn't exist)
mkdir -p projects/nexuscorp/staging/v2

# Create a directory and see what was created (-v = verbose)
mkdir -v new_logs
```

**Useful Flags:**

| Flag | Description | Example |
|---|---|---|
| `-p` | Create parent directories as needed | `mkdir -p a/b/c/d` |
| `-v` | Verbose — print each directory created | `mkdir -v mydir` |
| `-m` | Set permissions at creation time | `mkdir -m 755 public_dir` |

**💡 NexusCorp Use Case:** Scaffolding an entire project directory tree in a single command:
```bash
mkdir -p nexuscorp/{src,config,logs,backups}
```
This creates four subdirectories under `nexuscorp` at once using **brace expansion**.

---

### 3. `cp` — Copy Files and Directories

`cp` copies files or entire directories from one location to another. The original remains untouched.

**Syntax:**
```bash
cp [options] source destination
```

**Examples:**
```bash
# Copy a file to the same directory with a new name
cp server.conf server.conf.backup

# Copy a file to a different directory
cp server.conf /etc/nginx/

# Copy multiple files into a directory
cp file1.txt file2.txt file3.txt /backup/

# Copy a directory and all its contents (-r = recursive)
cp -r config/ config_backup/

# Copy and show what's being copied
cp -rv config/ config_backup/

# Copy only if source is newer than destination (-u = update)
cp -u reports/*.csv /shared/reports/
```

**Useful Flags:**

| Flag | Description | Example |
|---|---|---|
| `-r` or `-R` | Copy directories recursively | `cp -r mydir/ mydir_copy/` |
| `-v` | Verbose output | `cp -v file.txt /tmp/` |
| `-i` | Prompt before overwriting | `cp -i file.txt dest/` |
| `-u` | Only copy if source is newer | `cp -u *.log /archive/` |
| `-p` | Preserve timestamps and permissions | `cp -p config.yml config.yml.bak` |
| `-n` | Do not overwrite existing files | `cp -n template.conf new.conf` |

**⚠️ Common Mistake:** Forgetting `-r` when copying directories. Without it, `cp` will fail on directories.

**💡 NexusCorp Use Case:** Before editing any config file, always make a backup:
```bash
cp -p /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

---

### 4. `mv` — Move or Rename Files and Directories

`mv` moves files or directories from one location to another. When source and destination are in the same directory, it effectively **renames** the file. There is no separate rename command in Linux.

**Syntax:**
```bash
mv [options] source destination
```

**Examples:**
```bash
# Rename a file
mv oldname.txt newname.txt

# Move a file to a different directory
mv report.csv /home/user/reports/

# Move and rename at the same time
mv draft_v1.txt /archive/final_report.txt

# Move multiple files into a directory
mv *.log /var/log/archive/

# Rename a directory
mv old_project/ new_project/

# Prompt before overwriting (-i = interactive)
mv -i important.txt /backup/
```

**Useful Flags:**

| Flag | Description | Example |
|---|---|---|
| `-i` | Prompt before overwriting | `mv -i file.txt dest/` |
| `-v` | Verbose — show what's being moved | `mv -v *.txt archive/` |
| `-n` | Do not overwrite existing files | `mv -n file.txt backup/` |
| `-u` | Move only if source is newer | `mv -u *.csv data/` |

**💡 NexusCorp Use Case:** Rotating log files by renaming them with a timestamp:
```bash
mv app.log app_$(date +%Y%m%d).log
```

---

### 5. `rm` — Remove Files and Directories

`rm` permanently deletes files and directories. **There is no recycle bin.** Deleted files cannot be recovered without special tools.

**Syntax:**
```bash
rm [options] file(s)
```

**Examples:**
```bash
# Delete a single file
rm temp.txt

# Delete multiple files
rm file1.txt file2.txt file3.txt

# Delete all .log files in current directory
rm *.log

# Delete a directory and all its contents (-r = recursive)
rm -r old_project/

# Force delete without prompts (use with extreme caution!)
rm -rf /tmp/cache/

# Ask for confirmation before each deletion
rm -i *.txt
```

**Useful Flags:**

| Flag | Description | Example |
|---|---|---|
| `-r` or `-R` | Remove directories recursively | `rm -r mydir/` |
| `-f` | Force — ignore nonexistent files, no prompts | `rm -f file.txt` |
| `-i` | Prompt before every deletion | `rm -i *.log` |
| `-v` | Show what's being deleted | `rm -v *.tmp` |

**🚨 Critical Warning — Never Run This:**
```bash
rm -rf /          # Destroys the entire filesystem
rm -rf /*         # Same disaster, different syntax
rm -rf ./         # Deletes current directory recursively
```
Always double-check your paths before using `rm -rf`. A common safe habit: use `ls` first to preview what you're about to delete.

**💡 NexusCorp Use Case:** Cleaning up old deployment artifacts:
```bash
# Preview first
ls /deploy/builds/old_*

# Then delete
rm -rv /deploy/builds/old_*
```

---

### Putting It All Together: Common Patterns

These five commands constantly work together. Here are real-world workflows:

**Pattern 1: Set Up a Project Structure**
```bash
mkdir -p nexus_deploy/{configs,scripts,logs,backups}
touch nexus_deploy/configs/app.conf
touch nexus_deploy/scripts/deploy.sh
```

**Pattern 2: Backup Before Editing**
```bash
cp -p /etc/app/settings.conf /etc/app/settings.conf.bak
```

**Pattern 3: Archive and Clean Up Logs**
```bash
mkdir -p /var/log/archive/$(date +%Y%m)
mv /var/log/app/*.log /var/log/archive/$(date +%Y%m)/
```

**Pattern 4: Deploy a New Config**
```bash
cp template.conf production.conf
mv production.conf /etc/nginx/sites-available/nexuscorp
```

---

### 🛠️ Practical Exercise 1: File Operations

Work through each step in order and confirm the result before moving on.

```bash
# Step 1: Create the working directory
mkdir Day2Files
cd Day2Files

# Step 2: Create three empty files
touch file1.txt file2.txt file3.txt

# Verify
ls -l

# Step 3: Rename file3.txt
mv file3.txt file3_renamed.txt

# Verify
ls -l

# Step 4: Delete file2.txt
rm file2.txt

# Verify
ls -l

# Step 5: Create a backup directory and copy file1.txt into it
cd ..
mkdir Day2FilesBackup
cp Day2Files/file1.txt Day2FilesBackup/

# Verify
ls Day2FilesBackup/
```

**Expected final state:**
```
Day2Files/
├── file1.txt
└── file3_renamed.txt

Day2FilesBackup/
└── file1.txt
```

**Reflection Questions:**
1. What happens if you run `mkdir Day2Files` a second time? How do you fix this?
2. What's the difference between `cp file1.txt Day2FilesBackup/` and `cp file1.txt Day2FilesBackup/file1_copy.txt`?
3. Why is it risky to run `rm -rf` without `ls`-ing the path first?

---

## Part 2: Basic Text Manipulation

Working with text files is a constant in Linux — from reading logs to editing config files to writing shell scripts. These four tools cover reading, writing, and editing text at different levels of complexity.

---

### 1. `cat` — Concatenate and Display Files

`cat` (short for **concatenate**) reads one or more files and prints them to standard output. It's most often used to quickly view file contents, but it can do more.

**Syntax:**
```bash
cat [options] file(s)
```

**Examples:**
```bash
# Display a file's contents
cat notes.txt

# Display multiple files in sequence
cat header.txt body.txt footer.txt

# Concatenate files and save to a new file
cat part1.txt part2.txt > full_doc.txt

# Display with line numbers (-n)
cat -n script.sh

# Display non-printing characters (-A)
cat -A config.conf

# Create a small file inline (Ctrl+D to end input)
cat > quicknote.txt
This is a quick note.
Ctrl+D
```

**Useful Flags:**

| Flag | Description | Example |
|---|---|---|
| `-n` | Number all output lines | `cat -n script.sh` |
| `-b` | Number non-blank lines only | `cat -b file.txt` |
| `-A` | Show special characters (tabs as `^I`, etc.) | `cat -A config.conf` |
| `-s` | Suppress repeated empty lines | `cat -s file.txt` |

**💡 NexusCorp Use Case:** Quickly reviewing a service config without opening an editor:
```bash
cat /etc/nginx/nginx.conf
```

**🔗 Power Pattern — Combining with `grep`:**
```bash
cat app.log | grep "ERROR"
```

---

### 2. `echo` — Print Text to Output or Files

`echo` prints text to the terminal or redirects it into a file. It's used constantly in scripts to display messages, set file contents, or append lines.

**Syntax:**
```bash
echo [options] "text"
```

**Examples:**
```bash
# Print a message to the terminal
echo "Hello, NexusCorp!"

# Write text to a file (overwrites existing content)
echo "Linux is great" > opinion.txt

# Append text to a file (does NOT overwrite)
echo "And so is the terminal." >> opinion.txt

# Print a variable value
NAME="Yukta"
echo "Welcome, $NAME"

# Print without a trailing newline (-n)
echo -n "Loading..."

# Interpret escape sequences (-e)
echo -e "Line 1\nLine 2\nLine 3"

# Echo a blank line
echo ""
```

**Redirection Operators:**

| Operator | Behavior | Example |
|---|---|---|
| `>` | Write to file (overwrites) | `echo "new" > file.txt` |
| `>>` | Append to file (preserves existing content) | `echo "more" >> file.txt` |

**⚠️ Common Mistake:** Using `>` when you meant `>>`. Overwriting a config file you wanted to append to is a painful lesson.

**💡 NexusCorp Use Case:** Appending deployment events to a log file in a script:
```bash
echo "[$(date)] Deployment started." >> /var/log/deploy.log
```

---

### 3. `nano` — Beginner-Friendly Text Editor

`nano` is a simple, straightforward terminal text editor. Unlike `vi`, `nano` shows its keyboard shortcuts at the bottom of the screen, making it beginner-friendly. It's not available on every system by default, but it's common on Ubuntu/Debian systems.

**Open a file:**
```bash
nano opinion.txt
nano /etc/hosts
```

**Essential Keyboard Shortcuts:**

| Shortcut | Action |
|---|---|
| `Ctrl + O` | Save (Write Out) the file |
| `Enter` | Confirm the filename when saving |
| `Ctrl + X` | Exit nano |
| `Ctrl + K` | Cut the current line |
| `Ctrl + U` | Paste (Uncut) |
| `Ctrl + W` | Search (Where is) |
| `Ctrl + \` | Search and Replace |
| `Ctrl + G` | Open Help |
| `Ctrl + C` | Show current cursor position |
| `Alt + U` | Undo |

**Workflow for editing a file:**
```
1. nano filename.txt       ← Open file
2. Make your edits          ← Type normally
3. Ctrl + O                 ← Save
4. Enter                    ← Confirm filename
5. Ctrl + X                 ← Exit
```

**💡 NexusCorp Use Case:** Quickly editing a config file on a remote server over SSH where you want simplicity and a safety net.

---

### 4. `vi` / `vim` — The Universal Power Editor

`vi` (and its modern successor `vim` — **Vi IMproved**) is available on virtually every Unix/Linux system by default. It is more complex than nano, but it is far more powerful and is the editor you will find on any server you SSH into — even minimal Docker containers.

`vim` operates in **modes**, which is the key concept to understand first.

**The Three Core Modes:**

| Mode | How to Enter | What You Can Do |
|---|---|---|
| **Normal Mode** | Default mode / press `Esc` | Navigate, copy, delete, search |
| **Insert Mode** | Press `i` (insert) or `a` (append) | Type and edit text |
| **Command Mode** | Press `:` from Normal Mode | Save, quit, search/replace |

**Essential Commands — Normal Mode:**

| Command | Action |
|---|---|
| `h` `j` `k` `l` | Move left / down / up / right |
| `w` | Jump forward one word |
| `b` | Jump backward one word |
| `0` | Go to beginning of line |
| `$` | Go to end of line |
| `gg` | Go to top of file |
| `G` | Go to bottom of file |
| `dd` | Delete (cut) current line |
| `yy` | Copy (yank) current line |
| `p` | Paste after cursor |
| `u` | Undo |
| `Ctrl + r` | Redo |
| `/searchterm` | Search forward |
| `n` | Next search result |

**Essential Commands — Insert Mode:**

| Command | Action |
|---|---|
| `i` | Insert before cursor |
| `a` | Append after cursor |
| `o` | Open new line below |
| `O` | Open new line above |
| `Esc` | Return to Normal Mode |

**Essential Commands — Command Mode (`:`):**

| Command | Action |
|---|---|
| `:w` | Save the file |
| `:q` | Quit (fails if unsaved changes exist) |
| `:wq` | Save and quit |
| `:q!` | Quit without saving (force) |
| `:wq!` | Save and quit (force) |
| `:%s/old/new/g` | Replace all occurrences of "old" with "new" |
| `:set number` | Show line numbers |
| `:syntax on` | Enable syntax highlighting |

**The Minimal Survival Sequence:**
```
1. vim filename.txt         ← Open file
2. i                         ← Enter Insert Mode
3. (make your edits)
4. Esc                       ← Return to Normal Mode
5. :wq                       ← Save and quit
```

**If you're ever stuck in vim:** Press `Esc` several times, then type `:q!` and press `Enter` to exit without saving.

**📎 Resource:** [Vim Cheat Sheet](https://vim.rtorr.com/) — a comprehensive reference for those wanting to go deeper with vim.

**💡 NexusCorp Use Case:** Editing startup scripts and cron jobs on production servers where only `vi` is guaranteed to be installed.

---

### nano vs vi — At a Glance

| Feature | nano | vi / vim |
|---|---|---|
| Availability | Common on Ubuntu/Debian | Available on **all** Unix/Linux |
| Learning curve | Very low | Moderate — modal editing |
| Shortcuts visible | Yes (shown at bottom) | No — must be memorized |
| Power/features | Basic | Extremely powerful |
| Best for | Quick config edits, beginners | Production servers, power users, scripting |
| Recommendation | Start here | Learn this eventually |

---

### 🛠️ Practical Exercise 2: Text Manipulation

```bash
# Step 1: Create opinion.txt using echo
echo "Linux is great" > opinion.txt

# Step 2: Display its contents
cat opinion.txt

# Step 3: Append a second line
echo "The terminal is powerful." >> opinion.txt

# Verify both lines are there
cat opinion.txt

# Step 4: Open with nano and add a third line
nano opinion.txt
# → Type: "And DevOps makes it even better."
# → Ctrl+O, Enter, Ctrl+X

# Step 5: Open with vi and make a change
vi opinion.txt
# → Press i to enter Insert Mode
# → Navigate to the end, add: "NexusCorp agrees."
# → Press Esc
# → Type :wq and press Enter

# Step 6: Final display
cat -n opinion.txt
```

**Expected output (with line numbers):**
```
     1	Linux is great
     2	The terminal is powerful.
     3	And DevOps makes it even better.
     4	NexusCorp agrees.
```

**Reflection Questions:**
1. What's the difference between `echo "text" > file.txt` and `echo "text" >> file.txt`?
2. Why might a system admin prefer `vi` over `nano` in production?
3. How would you use `cat` to combine `file1.txt` and `file2.txt` into a `combined.txt`?

---

## 🔧 Mini-Project: NexusCorp Deployment Prep

**Scenario:** You're preparing a new staging environment for NexusCorp's web application. Set up the directory structure, create config placeholders, and document the setup.

```bash
# 1. Create the full project structure
mkdir -p nexuscorp_staging/{configs,scripts,logs,backups}

# 2. Create placeholder config files
touch nexuscorp_staging/configs/app.conf
touch nexuscorp_staging/configs/db.conf
touch nexuscorp_staging/configs/nginx.conf

# 3. Create a deployment log
echo "[$(date)] Staging environment initialized." > nexuscorp_staging/logs/deploy.log

# 4. Create a setup README
echo "NexusCorp Staging Environment" > nexuscorp_staging/README.txt
echo "Created: $(date)" >> nexuscorp_staging/README.txt
echo "Owner: DevOps Team" >> nexuscorp_staging/README.txt

# 5. Write a startup script note using vi or nano
vi nexuscorp_staging/scripts/start.sh
# Add: #!/bin/bash
# Add: echo "Starting NexusCorp Staging..."
# Save and exit (:wq)

# 6. Backup the configs directory
cp -r nexuscorp_staging/configs/ nexuscorp_staging/backups/configs_initial/

# 7. Verify the full structure
find nexuscorp_staging/ -type f

# 8. Display the README and deploy log
cat nexuscorp_staging/README.txt
cat nexuscorp_staging/logs/deploy.log
```

**Bonus Challenge:**
- Rename the `logs` directory to `audit_logs`
- Add a second entry to `deploy.log` using `echo`
- Delete the original `nexuscorp_staging/configs/nginx.conf` and re-create it with a note inside using `nano`

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Flags** | Additional options added to commands to modify their behavior (e.g., `-r`, `-v`, `-f`) |
| **Concatenate** | To link multiple strings or files together in sequence; what `cat` does with multiple files |
| **Text Editor** | Software used to view and edit plain text files; `nano` and `vi` are terminal-based examples |
| **Redirect** | Sending command output to a file instead of the screen; `>` overwrites, `>>` appends |
| **Recursive** | An operation that applies to a directory and all its subdirectories and files (flag: `-r`) |
| **Standard Output (stdout)** | The default destination for command output — usually the terminal screen |
| **Interactive Flag (`-i`)** | A flag that prompts the user before taking a potentially destructive action |
| **Normal Mode** | The default mode in `vi`/`vim` — used for navigation and commands, not typing |
| **Insert Mode** | The `vi`/`vim` mode where you can type and edit text |
| **Brace Expansion** | A shell feature that generates multiple strings from a pattern, e.g., `{a,b,c}` |

---

## 📚 Resources

- [GNU Coreutils Documentation](https://www.gnu.org/software/coreutils/manual/) — official docs for `cp`, `mv`, `rm`, `touch`, `mkdir`
- [Vim Cheat Sheet](https://vim.rtorr.com/) — comprehensive keyboard shortcut reference for vim
- [Nano Editor Documentation](https://www.nano-editor.org/docs.php) — official nano manual
- [Linux Command Line (Book)](https://linuxcommand.org/tlcl.php) — free online book covering these topics in depth
- [explainshell.com](https://explainshell.com) — paste any command to get a breakdown of what every part does

---

## 🔭 Day 6 Preview: Permissions & Ownership

Tomorrow you tackle one of Linux's most important security concepts: **file permissions**.

You'll learn:
- How Linux permission bits (`rwx`) work for user, group, and others
- `chmod` — change file permissions
- `chown` — change file ownership
- `chgrp` — change group ownership
- Numeric (octal) permission notation: `755`, `644`, `600`
- How NexusCorp secures its deployment scripts and config files

Understanding permissions is what separates a Linux user from a Linux administrator. See you there.

---

> 💬 *"The command line is a conversation with your system. File operations are the vocabulary. Text manipulation is the grammar. Master both, and you can say anything."*
>
> — DevOps Training & Learning By Yukta
