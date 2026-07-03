# Before Day 14 + Day 14: Git vs GitHub & Getting Started with Git

> **DevOps Transformation** | Git & Collaboration Series

---

## 📋 Overview

Before writing a single Git command, there are three widespread misconceptions about Git and GitHub that trip up almost every beginner. Getting these wrong leads to confusion that follows people for months. Today starts by clearing them up — then moves into actually setting up Git and running your first commands.

By the end of this session you'll have Git configured, a local repository initialized, files staged and committed, and a clear mental model of how the staging workflow operates.

---

## 🗺️ What You'll Learn Today

| Topic | Concepts / Commands | Why It Matters |
|---|---|---|
| Git vs GitHub myths | Debunking 3 common misconceptions | Builds the correct mental model before touching commands |
| Git vs GitHub differences | Nature, functionality, access, independence | Clarifies which tool does what |
| Why the distinction matters | Choosing the right tools, knowing alternatives | Informs decisions beyond just GitHub |
| Configuring Git | `git config` | Identifies who is making changes in a shared repo |
| Core Git workflow | `git init`, `git clone`, `git add`, `git commit`, `git status` | The five commands that underpin everything else in Git |

---

## 🏢 NexusCorp Scenario

> Every new engineer at NexusCorp gets the same orientation before touching the codebase: a 20-minute session on what Git actually is — and what it isn't. The number of support tickets from engineers who thought "push to GitHub" and "commit with Git" were the same operation dropped significantly after this session was introduced.
>
> You're getting that session now.

---

## Part 1: Debunking the Myths

Three myths about Git and GitHub are so common that they are worth addressing explicitly before anything else.

---

### Myth 1: Git and GitHub Are the Same Thing

**Fact:** They are completely separate tools made by different organizations.

**Git** is a version control system — a command-line tool that runs on your local machine and tracks changes to files over time. It was created by Linus Torvalds in 2005 to manage the Linux kernel source code. Git has no inherent connection to the internet or any particular website.

**GitHub** is a cloud-based platform built *around* Git. It provides a hosting service for Git repositories, along with a web interface, pull requests, issue tracking, Actions (CI/CD), project boards, and more. GitHub was founded in 2008 and acquired by Microsoft in 2018.

```
Git                         GitHub
────────────────────        ────────────────────────────
A tool                      A service (website + platform)
Runs locally                Lives in the cloud
Created 2005 (Torvalds)     Founded 2008 (acquired by Microsoft)
Tracks changes              Hosts, shares, and collaborates on repos
No account needed           Requires an account
Works offline               Requires internet (except Enterprise)
Free, open source           Free tier + paid plans
```

**Analogy:** Git is like the engine in a car. GitHub is like a parking garage with amenities — a place you can store the car, share it, and let others use it. The garage depends on the engine existing. The engine works perfectly fine outside the garage.

---

### Myth 2: Git Is Only for Developers

**Fact:** Git is useful for anyone who manages files that change over time.

While Git originated in software development, its core capability — tracking changes to text files — applies far beyond code:

| Use Case | How Git Helps |
|---|---|
| **Technical writers** | Track document revisions, compare drafts, collaborate on docs |
| **Designers** | Version design assets, track iterations, roll back changes |
| **Data scientists** | Version datasets, notebooks, and analysis scripts |
| **DevOps engineers** | Version infrastructure configs, Ansible playbooks, Terraform files |
| **Academic researchers** | Track paper drafts, collaborate with co-authors |
| **System administrators** | Track configuration changes across servers |

In DevOps specifically, Git is used to version everything: application code, Dockerfiles, Kubernetes manifests, CI/CD pipeline definitions, shell scripts, and infrastructure-as-code — not just application source code.

---

### Myth 3: Using GitHub Means My Code Is Open Source

**Fact:** GitHub supports both public and private repositories. The visibility of your code is your choice.

| Repository Type | Who Can See It | Who Can Clone It |
|---|---|---|
| **Public** | Anyone on the internet | Anyone |
| **Private** | Only you and collaborators you explicitly invite | Only invited collaborators |
| **Internal** (GitHub Enterprise) | All members of your organization | Organization members |

Pushing to GitHub does not automatically make your code open source. A repository is private by default when created through the GitHub UI unless you explicitly set it to public. Open source also requires more than just public visibility — it requires a license that grants specific rights to users.

---

## Part 2: Git vs GitHub — The Full Distinction

| Dimension | Git | GitHub |
|---|---|---|
| **Nature** | A tool (software you install) | A service (platform you sign into) |
| **Where it runs** | On your local machine | In the cloud |
| **Primary function** | Track changes in files over time | Host, share, and collaborate on Git repositories |
| **Internet required?** | No — works fully offline | Yes (push/pull require internet) |
| **Account required?** | No | Yes |
| **Independence** | Git works without GitHub | GitHub cannot function without Git |
| **Alternatives** | zsh, fish (different shells) — Git has no real alternative | GitLab, Bitbucket, Gitea, Azure DevOps |
| **Who made it** | Linus Torvalds, 2005 | GitHub Inc., 2008 (Microsoft, 2018) |
| **Cost** | Free, open source | Free tier + paid plans |

---

### Why This Distinction Matters

Understanding that Git and GitHub are separate lets you make better tool decisions:

**1. Right tool for the job:** If all you need is version control on a single machine or within a private network, Git alone is sufficient. You do not need GitHub.

**2. Platform choice:** GitHub is not the only Git hosting platform. GitLab, Bitbucket, Gitea, and Azure DevOps all use Git underneath — and each offers different features, pricing, and self-hosting options. Knowing Git works with any of them prevents lock-in thinking.

**3. Troubleshooting:** Many Git errors and concepts (remotes, push/pull, fetch, origin) only make sense if you understand the Git-local / GitHub-remote boundary. Conflating the two makes these harder to debug.

**4. Security awareness:** Knowing that GitHub and Git are separate means understanding that `git commit` is a local operation with no network activity — and `git push` is the step that sends data to a remote server.

```
Local Machine                    Remote (GitHub/GitLab/etc.)
─────────────────                ───────────────────────────
Working Directory
      │
      │ git add
      ▼
  Staging Area
      │
      │ git commit
      ▼
 Local Repository  ──── git push ────►  Remote Repository
                   ◄─── git pull ────
                   ◄─── git fetch ───
```

Everything to the left of the arrow is Git. The remote on the right is GitHub (or any other hosting platform). The arrow is the network boundary.

---

## Part 3: Setting Up Git

### Install Git

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install git

# RHEL / CentOS
sudo yum install git

# macOS (via Homebrew)
brew install git

# Verify installation
git --version
```

📎 Download for all platforms: [git-scm.com/downloads](https://git-scm.com/downloads)

---

### Configure Git

Before making any commits, configure Git with your identity. This information is embedded in every commit you make — it is how collaborators (and your future self) know who made which change.

```bash
# Set your name (appears in commit history)
git config --global user.name "Your Name"

# Set your email (should match your GitHub account email)
git config --global user.email "you@example.com"

# Set your default text editor (for writing commit messages)
git config --global core.editor nano        # or vim, code, etc.

# Set the default branch name to 'main' (modern standard)
git config --global init.defaultBranch main

# View all current configuration
git config --list

# View a specific setting
git config user.name
```

**Where config is stored:**

| Scope | Flag | Location | Applies To |
|---|---|---|---|
| System | `--system` | `/etc/gitconfig` | Every user on the machine |
| Global | `--global` | `~/.gitconfig` | All repos for your user |
| Local | `--local` | `.git/config` | Only the current repo |

Local overrides global, which overrides system. For everyday use, `--global` is the right choice.

**View your global config file:**
```bash
cat ~/.gitconfig
```

Sample output:
```
[user]
    name = Dev Engineer
    email = dev@nexuscorp.com
[core]
    editor = nano
[init]
    defaultBranch = main
```

---

## Part 4: Core Git Commands

### The Git Workflow

Every Git workflow follows the same three-zone model:

```
┌─────────────────┐    git add     ┌─────────────────┐    git commit    ┌─────────────────┐
│  Working        │ ─────────────► │  Staging Area   │ ───────────────► │ Local           │
│  Directory      │                │  (Index)        │                  │ Repository      │
│                 │ ◄───────────── │                 │                  │ (.git folder)   │
│  Files you      │  git restore   │  Changes queued │                  │ Committed       │
│  edit directly  │                │  for next commit│                  │ history         │
└─────────────────┘                └─────────────────┘                  └────────┬────────┘
                                                                                  │
                                                                             git push
                                                                                  │
                                                                                  ▼
                                                                         ┌─────────────────┐
                                                                         │ Remote Repo     │
                                                                         │ (GitHub/GitLab) │
                                                                         └─────────────────┘
```

Understanding this three-zone model is the key to understanding Git. Every command operates on one or more of these zones.

---

### `git init` — Initialize a New Repository

Creates a new Git repository in the current directory. Git adds a hidden `.git/` folder that contains all version history, configuration, and metadata.

```bash
# Create a new directory and initialize it as a Git repo
mkdir my_git_project
cd my_git_project
git init

# Output:
# Initialized empty Git repository in /home/user/my_git_project/.git/

# View the hidden .git folder
ls -la
# drwxr-xr-x  .git/

# Initialize with a specific branch name
git init -b main
```

**What's inside `.git/`:**
```bash
ls .git/
# HEAD        ← Points to the current branch
# config      ← Local repo configuration
# objects/    ← All file content and commits (compressed)
# refs/       ← Branch and tag pointers
```

You should never manually edit anything inside `.git/`. Git manages it entirely.

---

### `git clone` — Copy an Existing Repository

Downloads a complete copy of a remote repository — including all files, all branches, and the entire commit history.

```bash
# Clone a public repository
git clone https://github.com/username/repository.git

# Clone into a specific directory name
git clone https://github.com/username/repository.git my_local_name

# Clone only the latest snapshot (no full history — faster for large repos)
git clone --depth 1 https://github.com/username/repository.git

# Clone a specific branch
git clone -b develop https://github.com/username/repository.git
```

After cloning:
- The remote is automatically named `origin`
- You are on the default branch (`main` or `master`)
- All remote branches are available to check out locally

```bash
cd repository
git remote -v
# origin  https://github.com/username/repository.git (fetch)
# origin  https://github.com/username/repository.git (push)
```

---

### `git status` — Inspect the Working Tree

Shows the state of your working directory and staging area. This is the command you run most often — before and after every operation to understand what Git sees.

```bash
git status
```

**Possible states Git reports:**

| Status | Meaning |
|---|---|
| `Untracked files` | New files Git has never seen before — not yet added |
| `Changes not staged for commit` | Files Git tracks that have been modified but not yet staged |
| `Changes to be committed` | Files in the staging area, ready for the next commit |
| `nothing to commit, working tree clean` | Working directory matches the last commit exactly |

**Sample output at different workflow stages:**

```bash
# After creating a new file:
$ git status
On branch main
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        readme.md
nothing added to commit but untracked files present

# After git add:
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   readme.md

# After git commit:
$ git status
On branch main
nothing to commit, working tree clean
```

---

### `git add` — Stage Changes

Moves changes from the working directory into the staging area. Only staged changes are included in the next commit. This gives you precise control over what goes into each commit.

```bash
# Stage a specific file
git add readme.md

# Stage multiple specific files
git add file1.txt file2.txt

# Stage all changes in the current directory (and subdirectories)
git add .

# Stage all changes to tracked files (not new untracked files)
git add -u

# Stage parts of a file interactively (patch mode)
git add -p readme.md

# Stage an entire directory
git add src/
```

**Why the staging area exists:**

The staging area lets you craft focused, logical commits. Imagine you fixed a bug and also reorganized some imports in the same session. With the staging area, you can commit just the bug fix first, then commit the reorganization separately — even though you changed both sets of files at the same time. Clean commits make history readable and rollbacks safer.

---

### `git commit` — Save a Snapshot

Records everything in the staging area as a permanent snapshot in the repository's history. Each commit has a unique SHA hash, an author, a timestamp, and a message.

```bash
# Commit with an inline message (-m)
git commit -m "Add project README with setup instructions"

# Open your configured editor to write a detailed multi-line message
git commit

# Stage all tracked modified files AND commit in one step (-a skips git add)
# Note: does NOT include brand new untracked files
git commit -am "Fix typo in welcome message"

# Amend the most recent commit (fix message or add forgotten files)
# Only do this before pushing — never amend commits others have pulled
git commit --amend -m "Corrected: Add project README with setup instructions"
```

**Writing good commit messages:**

A commit message is a note to your future self and your teammates. The industry-standard format:

```
Short summary line (50 characters or fewer, imperative mood)

Optional longer description after a blank line. Explain WHY this
change was made, not just what was changed. The code shows what;
the message should explain why.

- Use bullet points for multiple changes if helpful
- Reference issue numbers: Fixes #42
```

**Examples:**

```bash
# Good — clear, imperative, specific
git commit -m "Fix login redirect loop when session expires"
git commit -m "Add Dockerfile for staging environment"
git commit -m "Remove deprecated API endpoints from v1 routes"

# Poor — vague, not actionable
git commit -m "fix stuff"
git commit -m "changes"
git commit -m "WIP"
```

---

### Viewing History with `git log`

```bash
# Full commit history
git log

# Compact one-line view
git log --oneline

# Visual branch graph
git log --oneline --graph --all

# Show last 5 commits
git log -5

# Show commits by a specific author
git log --author="Dev Engineer"

# Show what changed in each commit
git log -p

# Show stats (files changed, lines added/removed)
git log --stat
```

---

## 🛠️ Practical Exercises

### Exercise 1: Initialize and Commit

```bash
# Step 1: Create and enter the project directory
mkdir my_git_project
cd my_git_project

# Step 2: Initialize the repository
git init

# Step 3: Confirm the repo was created
ls -la   # You should see .git/

# Step 4: Create readme.md with a brief introduction
nano readme.md
# Write a few lines — project name, purpose, your role

# Step 5: Check status before staging
git status
# readme.md should show as "Untracked files"

# Step 6: Stage the file
git add readme.md

# Step 7: Check status again — confirm it's staged
git status
# readme.md should now show under "Changes to be committed"

# Step 8: Commit with a clear message
git commit -m "Initial commit: add project README"

# Step 9: Verify the commit was recorded
git log --oneline
```

---

### Exercise 2: Clone and Explore

```bash
# Step 1: Clone an interesting public repository
# Suggestion: replace with any public repo you find interesting
git clone https://github.com/torvalds/linux.git     # Linux kernel (large)
git clone https://github.com/django/django.git       # Django framework
git clone https://github.com/microsoft/vscode.git   # VS Code editor

# Step 2: Enter the cloned directory
cd django   # (or whichever you cloned)

# Step 3: View the commit history (most recent first)
git log --oneline | head -20

# Step 4: View the full history with stats
git log --stat | head -50

# Step 5: Read a few commit messages in full
git log -5

# Step 6: See all branches
git branch -a | head -20

# Step 7: Examine the repo structure
ls -la

# Questions to reflect on:
# - How often are commits made?
# - What does a well-written commit message look like here?
# - How many contributors are active?
# - What branch naming conventions do they use?
```

---

### Exercise 3: Staging Area Workflow

```bash
# Continuing inside my_git_project from Exercise 1

# Step 1: Create notes.txt
nano notes.txt
# Add a few lines about what you learned — Git vs GitHub,
# the three-zone model, what staging area means, etc.

# Step 2: Check status — notes.txt should be untracked
git status

# Step 3: Stage notes.txt
git add notes.txt

# Step 4: Check status again — confirm it moved to staging
git status

# Step 5: Commit with a descriptive message
git commit -m "Add learning notes from Day 14"

# Step 6: View the full history so far
git log --oneline

# Bonus: Make a change to readme.md and notes.txt,
# then stage only one of them and commit.
# Observe how git status shows the difference
# between staged and unstaged changes simultaneously.
echo "Updated by: $(whoami)" >> readme.md
echo "Additional notes added." >> notes.txt

git add readme.md        # Stage only readme.md
git status               # notes.txt should still show as "not staged"
git commit -m "Update README with author information"
git add notes.txt
git commit -m "Add additional notes to notes.txt"
git log --oneline
```

---

## 🔧 Mini-Project: My First Git Repository

Build a properly structured personal project repository from scratch, applying everything from today.

```bash
# Step 1: Create the project
mkdir nexus_git_starter
cd nexus_git_starter
git init -b main

# Step 2: Create a .gitignore to exclude common junk files
cat > .gitignore << 'EOF'
# OS files
.DS_Store
Thumbs.db

# Editor files
.vscode/
*.swp
*~

# Logs
*.log

# Environment files (never commit credentials)
.env
.env.*
EOF

# Step 3: Create a README.md
cat > README.md << 'EOF'
# NexusCorp Git Starter

A personal learning repository tracking progress through the
NexusCorp DevOps Transformation curriculum.

## Contents

- `notes/` — Daily learning notes
- `scripts/` — Practice shell scripts

## Status

Day 14: Git fundamentals complete.
EOF

# Step 4: Create a notes directory and first entry
mkdir notes
cat > notes/day14.md << 'EOF'
# Day 14 Notes: Git Basics

## Key Concepts
- Git is a tool; GitHub is a platform built on Git
- Three zones: Working Directory → Staging Area → Repository
- git add stages changes; git commit records them permanently

## Commands Practiced
- git init, git clone, git add, git commit, git status, git log

## What I Found Challenging

## What I Want to Explore Next
EOF

# Step 5: Stage everything
git add .

# Step 6: Check what you're about to commit
git status

# Step 7: Make the initial commit
git commit -m "Initial commit: project structure, README, and Day 14 notes"

# Step 8: View the repository history
git log --oneline

# Step 9: Add a practice script
mkdir scripts
cat > scripts/hello.sh << 'EOF'
#!/bin/bash
echo "Hello from NexusCorp Git Starter!"
echo "Current branch: $(git branch --show-current)"
echo "Last commit: $(git log --oneline -1)"
EOF
chmod +x scripts/hello.sh

git add scripts/hello.sh
git commit -m "Add hello.sh practice script"

# Step 10: View final history
git log --oneline --graph
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Git** | A distributed version control system that tracks changes to files locally |
| **GitHub** | A cloud-based platform for hosting, sharing, and collaborating on Git repositories |
| **Repository (repo)** | A directory tracked by Git, containing all project files and the complete change history |
| **`git init`** | Initializes a new Git repository in the current directory, creating a `.git/` folder |
| **`git clone`** | Creates a complete local copy of a remote repository, including all history |
| **`git add`** | Moves changes from the working directory to the staging area |
| **`git commit`** | Records a permanent snapshot of staged changes into the local repository |
| **`git status`** | Shows the current state of the working directory and staging area |
| **`git log`** | Displays the commit history of the repository |
| **Working Directory** | The folder where you directly edit files — changes here are not yet tracked by Git |
| **Staging Area (Index)** | An intermediate zone holding changes that will be included in the next commit |
| **Local Repository** | The `.git/` folder on your machine — stores all committed history |
| **Remote Repository** | A version of the repository hosted on a server (GitHub, GitLab, etc.) |
| **Commit** | A permanent, timestamped snapshot of staged changes with an author and message |
| **SHA / Hash** | A unique 40-character identifier automatically assigned to each commit |
| **`.gitignore`** | A file listing patterns of files and directories Git should never track |
| **`origin`** | The default name Git gives to the remote a repo was cloned from |
| **`git config`** | Sets Git configuration values — user identity, default editor, default branch name |
| **Untracked** | A file Git has never seen before — not yet added to the staging area |
| **Staged** | A change that has been `git add`-ed and is queued for the next commit |
| **Open Source** | Code released under a license granting others rights to use, modify, and distribute it |

---

## 📚 Resources

- [Download Git](https://git-scm.com/downloads) — official installers for all platforms
- [Pro Git Book (free)](https://git-scm.com/book/en/v2) — the definitive Git reference, freely available online
- [GitHub Docs — Getting Started](https://docs.github.com/en/get-started) — official GitHub onboarding documentation
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials) — practical, well-illustrated guides on Git workflows
- [Learn Git Branching (Interactive)](https://learngitbranching.js.org) — visual, browser-based Git practice
- [Quick Class on Git & GitHub — YouTube](https://www.youtube.com/results?search_query=git+github+beginner+tutorial) — video walkthrough for visual learners

---

## 🔭 Day 15 Preview: Branching & Merging

Now that your first commits are in place, Day 15 adds the capability that makes Git genuinely powerful for team workflows: branching.

You'll learn:
- `git branch` — create, list, and delete branches
- `git checkout` / `git switch` — move between branches
- `git merge` — combine branches back together
- Resolving merge conflicts — what they are and how to fix them
- Feature branch workflow — the pattern used by most professional teams

Commits are how Git remembers. Branches are how Git lets multiple lines of development coexist without interfering with each other.

---

> 💬 *"Git is the tool. GitHub is the garage. Knowing which is which is the first step to using both well."*
>
> — DevOps Learning by Yukta