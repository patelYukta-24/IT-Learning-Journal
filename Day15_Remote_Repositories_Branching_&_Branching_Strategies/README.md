# Day 15: Remote Repositories, Branching & Branching Strategies

> **DevOps Transformation** | Git & Collaboration Series

---

## 📋 Overview

Yesterday you learned how to track changes locally — `init`, `add`, `commit`. Today you take that work out of your machine and into the world.

**Remote repositories** are how your local Git history gets backed up, shared, and collaborated on. **Branching** is how multiple people (or you, across multiple tasks) work simultaneously without breaking each other's work. And **branching strategies** are the agreed-upon rules that make all of that scale across a real team.

These three concepts together define how professional teams use Git every single day.

---

## 🗺️ What You'll Learn Today

| Topic | Commands | Why It Matters |
|---|---|---|
| Remote repositories | `git remote`, `git push`, `git pull`, `git fetch` | Share work, collaborate, and back up your history |
| Branching basics | `git branch`, `git checkout`, `git switch` | Isolate work, develop features safely |
| Merging | `git merge` | Combine completed work back into the main codebase |
| Merge conflicts | Manual resolution | The unavoidable reality of parallel development |
| Branching strategies | Git Flow, GitHub Flow, Trunk-Based | Team-level patterns for managing releases and features |

---

## 🏢 NexusCorp Scenario

> NexusCorp just moved from a single-developer codebase to a five-person engineering team. Three engineers are working on new features, one is fixing a production bug, and one is preparing the next release. Without a clear branching strategy, they would be stepping on each other constantly. With one — and with the remote workflow you'll learn today — each person works independently and merges cleanly when ready.
>
> This is the system you're building toward.

---

## Part 1: Remote Repositories

### What Is a Remote Repository?

A **remote repository** is a version of your repository hosted on a server — typically GitHub, GitLab, or Bitbucket — that your local Git installation knows how to communicate with. It serves three purposes:

| Purpose | Description |
|---|---|
| **Backup** | Your full commit history lives somewhere other than your laptop |
| **Collaboration** | Multiple developers push and pull from the same remote |
| **Deployment** | CI/CD pipelines typically trigger from pushes to specific remote branches |

The relationship between local and remote:

```
Your Machine                        Remote (GitHub/GitLab)
──────────────────────              ──────────────────────
Working Directory
      │ git add
      ▼
  Staging Area
      │ git commit
      ▼
Local Repository  ──── git push ──► Remote Repository
                  ◄─── git pull ───
                  ◄─── git fetch ──  (no merge)
```

---

### `git remote` — Managing Remote Connections

A remote is just a named URL that your local repository points to. The default name for the remote a repo was cloned from is `origin` — a convention, not a requirement.

```bash
# List all remotes (short form)
git remote

# List all remotes with their URLs
git remote -v

# Sample output:
# origin  https://github.com/username/my-project.git (fetch)
# origin  https://github.com/username/my-project.git (push)

# Add a new remote (useful when you didn't clone — e.g., after git init)
git remote add origin https://github.com/username/my-project.git

# Add a second remote (e.g., the upstream original of a fork)
git remote add upstream https://github.com/original-owner/my-project.git

# Rename a remote
git remote rename origin github

# Remove a remote
git remote remove upstream

# Change the URL of an existing remote
git remote set-url origin https://github.com/username/new-name.git

# View detailed information about a remote
git remote show origin
```

**Common remote naming conventions:**

| Name | Convention |
|---|---|
| `origin` | Your fork or personal copy — the remote you push to |
| `upstream` | The original repository your fork was based on |
| `production` | A remote that deploys directly to production (some pipelines) |

---

### `git push` — Send Local Commits to Remote

`git push` uploads commits from your local repository to the remote. Only changes that have been committed locally are pushed — the staging area and working directory are not pushed.

```bash
# Push the current branch to its tracked remote branch
git push

# Push a specific branch to origin
git push origin main

# Push a local branch to a remote branch (explicit mapping)
git push origin feature-login:feature-login

# Push and set the upstream tracking branch (-u = --set-upstream)
# After this, plain 'git push' knows where to go
git push -u origin main

# Push all local branches to origin
git push --all origin

# Push tags (not included by default)
git push origin --tags

# Force push — overwrites remote history (use with extreme caution)
# Only acceptable on your own feature branches, never on shared branches
git push --force-with-lease origin feature-x
```

**⚠️ Force Push Warning:** `git push --force` rewrites the remote branch history. If anyone else has pulled that branch, their history will diverge and their next pull will break. Always prefer `--force-with-lease`, which checks that no one else has pushed since your last fetch.

---

### `git pull` — Fetch and Merge Remote Changes

`git pull` does two things in one command: fetches the latest commits from the remote, then merges them into your current local branch. It is the standard way to bring your local branch up to date with the remote.

```bash
# Pull changes from the tracked remote branch into current branch
git pull

# Pull from a specific remote and branch
git pull origin main

# Pull using rebase instead of merge (cleaner history)
git pull --rebase origin main

# Pull all remote branches (fetch + update tracking branches)
git pull --all
```

**`git pull` = `git fetch` + `git merge`**

```
Before pull:
  Local:   A -- B -- C (main)
  Remote:  A -- B -- C -- D -- E (origin/main)

After git pull:
  Local:   A -- B -- C -- D -- E (main)
```

---

### `git fetch` — Download Without Merging

`git fetch` downloads all new commits and branches from the remote but does **not** modify your local branches. It updates the remote-tracking references (`origin/main`, `origin/feature-x`) without touching your working directory.

```bash
# Fetch all changes from origin
git fetch origin

# Fetch from all remotes
git fetch --all

# Fetch a specific branch
git fetch origin main

# After fetching, compare local to remote
git diff main origin/main

# View what changed on remote without applying it
git log origin/main --oneline
```

**When to use `fetch` instead of `pull`:**

| Use `git fetch` when... | Use `git pull` when... |
|---|---|
| You want to see what changed remotely before deciding to merge | You trust the remote and want to update immediately |
| You're on a branch with uncommitted changes | Your working directory is clean |
| You want to review the diff before merging | You want a quick sync |
| You're in the middle of something and just need a status check | You're starting a new work session |

---

### Forking vs Cloning

These two terms are often confused by beginners:

| | **Fork** | **Clone** |
|---|---|---|
| **What it is** | A copy of a repository on GitHub, under your own account | A copy of a repository on your local machine |
| **Where it lives** | On GitHub (remote) | On your machine (local) |
| **Who owns it** | You (on GitHub) | You (locally) |
| **Git operation?** | No — a GitHub feature | Yes — `git clone` |
| **Connection to original** | Tracked as `upstream` | N/A — set manually |
| **When to use** | Contributing to projects you don't have write access to | Working on projects you own or have been invited to |

**Fork → Clone → Push → Pull Request workflow:**

```
1. Fork the repo on GitHub       → Creates your copy at github.com/you/repo
2. git clone your fork           → Brings it to your machine
3. git remote add upstream ...   → Connects to original repo
4. Make changes on a branch
5. git push to your fork
6. Open a Pull Request           → Proposes merging your branch into the original
```

---

## Part 2: Branching

### What Is a Branch?

A branch is a lightweight, movable pointer to a specific commit. When you create a branch, Git doesn't copy files — it simply creates a new pointer. This is why branching in Git is nearly instant, regardless of project size.

```
main:     A -- B -- C
                     \
feature-x:            D -- E -- F
```

`main` and `feature-x` share history up to commit `C`. After that, they diverge independently. Neither knows about the other's changes until they are merged.

**The `HEAD` pointer:** `HEAD` is a special pointer that always points to the branch you are currently on (or directly to a commit in "detached HEAD" state). When you commit, the current branch pointer moves forward to the new commit — `HEAD` moves with it.

---

### `git branch` — Create, List, and Delete Branches

```bash
# List all local branches (current branch marked with *)
git branch

# List all branches including remote-tracking branches
git branch -a

# List remote branches only
git branch -r

# Create a new branch (does not switch to it)
git branch feature-login

# Delete a branch (safe — refuses if unmerged changes exist)
git branch -d feature-login

# Force delete a branch (even if unmerged)
git branch -D experimental-idea

# Rename the current branch
git branch -m new-branch-name

# See which branches have been merged into current branch
git branch --merged

# See which branches have NOT been merged
git branch --no-merged
```

---

### `git checkout` / `git switch` — Change Branches

```bash
# Switch to an existing branch (classic syntax)
git checkout main

# Create a new branch AND switch to it
git checkout -b feature-x

# Modern syntax (Git 2.23+) — clearer intent
git switch main
git switch -c feature-x        # -c = create

# Switch to the previous branch (like cd -)
git checkout -
git switch -

# Checkout a specific commit (enters "detached HEAD" state)
git checkout a1b2c3d

# Restore a specific file from another branch (doesn't switch branch)
git checkout main -- config/settings.py
```

**Detached HEAD:** When you checkout a specific commit (not a branch), HEAD points directly to that commit rather than to a branch. Any commits you make won't belong to any branch and may be lost. To keep work done in detached HEAD state, create a branch from it:
```bash
git checkout -b recovery-branch
```

---

### `git merge` — Combine Branch History

`git merge` integrates the history of one branch into another. Always merge *into* the branch you want to update — switch to the target branch first.

```bash
# Switch to the branch you want to merge INTO
git checkout main

# Merge feature-x into main
git merge feature-x

# Merge with an explicit merge commit (no fast-forward)
git merge --no-ff feature-x

# Abort a merge in progress (if you want to cancel)
git merge --abort
```

**Types of merges Git performs:**

**Fast-forward merge** — happens when the target branch has no new commits since the feature branch diverged. Git simply moves the pointer forward:

```
Before:                          After (fast-forward):
main:    A -- B                  main:    A -- B -- C -- D
                \                                         ↑
feature-x:       C -- D          feature-x (deleted)
```

**Three-way merge** — happens when both branches have diverged. Git finds the common ancestor and creates a new merge commit:

```
Before:                          After (3-way merge):
main:    A -- B -- C             main:    A -- B -- C -- M
                    \                                  ↗   (merge commit)
feature-x:  B -- D -- E         feature-x:    D -- E
```

---

### Merge Conflicts

A merge conflict occurs when two branches have made different changes to the **same lines of the same file**. Git cannot decide automatically which version to keep — it flags the conflict and waits for you to resolve it.

**What a conflict looks like in the file:**

```
<<<<<<< HEAD
This line was changed on the main branch.
=======
This line was changed on the feature-x branch.
>>>>>>> feature-x
```

| Section | Meaning |
|---|---|
| `<<<<<<< HEAD` to `=======` | The content from your current branch (main) |
| `=======` to `>>>>>>> feature-x` | The content from the branch being merged |

**Resolving a conflict:**

```bash
# Step 1: Git will tell you which files have conflicts after a merge
git status
# Both modified: src/app.py

# Step 2: Open the conflicted file and manually edit it
nano src/app.py
# Remove the <<<<<<<, =======, and >>>>>>> markers
# Keep the version you want (or combine both)

# Step 3: Stage the resolved file
git add src/app.py

# Step 4: Complete the merge with a commit
git commit
# Git pre-fills the merge commit message — you can accept it

# If you want to abandon the merge entirely
git merge --abort
```

**Tips for minimizing conflicts:**
- Pull from the main branch frequently into your feature branch to stay current
- Keep feature branches short-lived — the longer they live, the more they diverge
- Communicate with teammates about who is editing which files
- Break large changes into smaller, more focused commits

---

## Part 3: Branching Strategies

A **branching strategy** is a team agreement about how branches are created, named, used, and merged. Without one, even a small team quickly ends up with a chaotic, unmergeable mess of branches. With one, code reviews, releases, and hotfixes all have predictable, repeatable processes.

The three most widely used strategies:

---

### 1. Git Flow

**Best for:** Projects with scheduled releases and a clear version cycle (e.g., mobile apps, packaged software).

**Branch structure:**

```
main         ──────────────────────────────────────────►  (production-ready)
               ↑                              ↑
release/1.0 ───┘                   release/1.1┘
               ↑                              ↑
develop  ──────┼──────────────────────────────┼──────►  (integration)
               ↑          ↑
feature/login  ┘  feature/dashboard
```

**Branch types:**

| Branch | Purpose | Created from | Merges into |
|---|---|---|---|
| `main` | Production-ready code only | — | — |
| `develop` | Integration branch — all features merge here | `main` | `main` (via release) |
| `feature/*` | New features | `develop` | `develop` |
| `release/*` | Release preparation (bug fixes, versioning) | `develop` | `main` + `develop` |
| `hotfix/*` | Emergency production patches | `main` | `main` + `develop` |

**Workflow:**
```bash
# Start a new feature
git checkout develop
git checkout -b feature/user-auth

# ... work, commit ...

# Merge feature back to develop
git checkout develop
git merge --no-ff feature/user-auth
git branch -d feature/user-auth

# Prepare a release
git checkout -b release/1.0 develop
# ... final testing, version bump ...
git checkout main
git merge --no-ff release/1.0
git tag -a v1.0 -m "Release version 1.0"
git checkout develop
git merge --no-ff release/1.0
```

**Pros:** Clear structure, great for versioned software, well-documented.
**Cons:** Complex, overhead for small teams, can slow down continuous delivery.

---

### 2. GitHub Flow

**Best for:** Teams practicing continuous deployment — shipping to production frequently, sometimes multiple times per day.

**Branch structure:**

```
main     ──────────────────────────────────────────────► (always deployable)
               ↑          ↑          ↑
feature/x ─────┘  fix/y ──┘  docs/z ─┘
```

**Rules:**
1. `main` is always deployable — never commit broken code directly to it
2. Create a descriptive branch for every change, no matter how small
3. Commit and push frequently — opens early for discussion
4. Open a pull request when ready for review
5. Merge only after review approval and passing CI
6. Deploy to production immediately after merging

**Workflow:**
```bash
# Start any new change
git checkout main
git pull origin main
git checkout -b feature/add-payment-gateway

# ... work, commit often ...
git push -u origin feature/add-payment-gateway

# Open a Pull Request on GitHub for review
# After approval and CI pass:
# Merge via GitHub UI or:
git checkout main
git merge --no-ff feature/add-payment-gateway
git push origin main
git branch -d feature/add-payment-gateway
```

**Pros:** Simple, fast, fits CI/CD, low overhead.
**Cons:** Requires strong test coverage; not ideal for supporting multiple live versions simultaneously.

---

### 3. Trunk-Based Development

**Best for:** High-performing DevOps teams with strong automated testing and feature flag infrastructure.

**Branch structure:**

```
main (trunk)  ─────────────────────────────────────────► (everyone commits here)
                  ↑    ↑    ↑    ↑    ↑
short-lived       │    │    │    │    │
feature branches──┘    ┘    ┘    ┘    ┘   (hours or 1-2 days max)
```

**Rules:**
1. Everyone commits to `main` (trunk) at least once per day
2. Feature branches exist for at most a day or two — often just hours
3. **Feature flags** control whether new features are active in production, even before they're complete
4. Every commit must pass automated tests before merging
5. CI pipeline runs on every push

**Pros:** Eliminates merge conflicts almost entirely, fastest path to production, forces small incremental changes.
**Cons:** Requires mature CI/CD and test coverage, requires feature flag infrastructure, difficult for less experienced teams.

---

### Choosing a Strategy

| Factor | Git Flow | GitHub Flow | Trunk-Based |
|---|---|---|---|
| **Team size** | Medium–Large | Any | Medium–Large |
| **Release cadence** | Scheduled (weeks/months) | Continuous | Continuous |
| **Multiple live versions?** | Yes | No | No |
| **CI/CD maturity needed** | Low–Medium | Medium | High |
| **Complexity** | High | Low | Medium |
| **Merge conflicts** | Common | Occasional | Rare |
| **Best for** | Mobile apps, packaged software | SaaS, web apps | High-performing teams |

---

## 🛠️ Practical Exercises

### Exercise 1: Pushing to a Remote

```bash
# Step 1: Fork a public repository on GitHub
# Go to github.com, find an interesting repo, click "Fork"
# This creates your personal copy under github.com/your-username/repo-name

# Step 2: Clone YOUR fork (not the original)
git clone https://github.com/YOUR-USERNAME/REPO-NAME.git
cd REPO-NAME

# Step 3: Verify the remote is set to your fork
git remote -v
# origin  https://github.com/YOUR-USERNAME/REPO-NAME.git

# Step 4: Add upstream to track the original
git remote add upstream https://github.com/ORIGINAL-OWNER/REPO-NAME.git
git remote -v

# Step 5: Create a branch for your change
git checkout -b my-first-contribution

# Step 6: Make a change (e.g., edit README.md)
echo "" >> README.md
echo "<!-- Explored by: a DevOps learner -->" >> README.md

# Step 7: Stage and commit
git add README.md
git commit -m "Add exploration comment to README"

# Step 8: Push to YOUR fork
git push -u origin my-first-contribution

# Step 9: Verify on GitHub — visit your fork and see the new branch
```

---

### Exercise 2: Branch Creation and Switching

```bash
# Step 1: Confirm which branch you're on
git branch
git status

# Step 2: Create a new branch
git checkout -b feature-x
# Or: git switch -c feature-x

# Step 3: Confirm you're on the new branch
git branch    # feature-x should have *

# Step 4: Practice switching
git switch main
git switch feature-x
git switch main
git switch feature-x
# Get comfortable — switching is fast and safe when working tree is clean

# Step 5: Make and commit a change on feature-x
echo "Feature X is being developed here." > feature_x_notes.txt
git add feature_x_notes.txt
git commit -m "Add Feature X development notes"

# Step 6: Switch to main — notice the file is gone (it belongs to feature-x)
git switch main
ls    # feature_x_notes.txt will not be here

# Step 7: Switch back — it reappears
git switch feature-x
ls    # feature_x_notes.txt is back
```

---

### Exercise 3: Simulating a Merge Conflict

```bash
# Setup: ensure you're in a test repo with both branches from Exercise 2

# Step 1: On feature-x, edit a file
git switch feature-x
echo "This line was written on feature-x." > shared.txt
git add shared.txt
git commit -m "Add shared.txt from feature-x perspective"

# Step 2: Switch to main, edit the SAME file differently
git switch main
echo "This line was written on main." > shared.txt
git add shared.txt
git commit -m "Add shared.txt from main perspective"

# Step 3: Attempt to merge feature-x into main
git merge feature-x
# Git will report a conflict in shared.txt

# Step 4: Check status — identify conflicted files
git status
# Both modified: shared.txt

# Step 5: Open the conflicted file
cat shared.txt
# You'll see:
# <<<<<<< HEAD
# This line was written on main.
# =======
# This line was written on feature-x.
# >>>>>>> feature-x

# Step 6: Resolve — edit the file to the desired final state
cat > shared.txt << 'EOF'
This file reflects the merged work from main and feature-x.
Both branches contributed to the final version.
EOF

# Step 7: Stage the resolved file
git add shared.txt

# Step 8: Complete the merge commit
git commit
# Accept the pre-filled merge message or write your own

# Step 9: Verify the merge
git log --oneline --graph
```

---

### Exercise 4: Implement a Branching Strategy

```bash
# Simulate a simplified GitHub Flow on a new repository

# Step 1: Initialize a project repo
mkdir nexus_branching_demo
cd nexus_branching_demo
git init -b main

# Create initial project structure
mkdir -p src docs
echo "# NexusCorp App" > README.md
echo "print('NexusCorp App v0.1')" > src/app.py
git add .
git commit -m "Initial commit: project scaffold"

# Step 2: Feature branch — user authentication
git checkout -b feature/user-auth
echo "def authenticate(user, password): pass" >> src/app.py
git add src/app.py
git commit -m "Add authentication function placeholder"
echo "## User Authentication" > docs/auth.md
git add docs/auth.md
git commit -m "Add auth documentation"

# Step 3: Merge feature into main (simulate PR merge)
git checkout main
git merge --no-ff feature/user-auth -m "Merge feature/user-auth into main"
git branch -d feature/user-auth

# Step 4: Feature branch — dashboard
git checkout -b feature/dashboard
echo "def render_dashboard(): pass" >> src/app.py
git add src/app.py
git commit -m "Add dashboard render function"

# Step 5: Hotfix — critical bug on main (while dashboard is in progress)
git checkout main
git checkout -b hotfix/fix-startup-error
echo "# startup fix applied" >> src/app.py
git add src/app.py
git commit -m "Fix critical startup error"
git checkout main
git merge --no-ff hotfix/fix-startup-error -m "Merge hotfix/fix-startup-error"
git branch -d hotfix/fix-startup-error

# Step 6: Merge dashboard
git checkout feature/dashboard
git merge main   # Bring in the hotfix first
git checkout main
git merge --no-ff feature/dashboard -m "Merge feature/dashboard into main"
git branch -d feature/dashboard

# Step 7: View the full branching history
git log --oneline --graph --all
```

---

## 🔧 Mini-Project: NexusCorp Collaborative Workflow

**Scenario:** Simulate the full NexusCorp team workflow — fork, branch, change, push, and prepare a pull request.

```bash
# ── On GitHub ──────────────────────────────────────────────
# 1. Fork any public repository to your GitHub account

# ── Locally ────────────────────────────────────────────────
# 2. Clone your fork
git clone https://github.com/YOUR-USERNAME/REPO.git
cd REPO

# 3. Configure upstream
git remote add upstream https://github.com/ORIGINAL/REPO.git
git remote -v

# 4. Sync with upstream before starting work
git fetch upstream
git checkout main
git merge upstream/main

# 5. Create a feature branch following a naming convention
git checkout -b feature/YOUR-FEATURE-NAME

# 6. Make meaningful changes
#    (edit files, add content, fix something)

# 7. Commit with a clear, present-tense message
git add .
git commit -m "Add: brief description of what this does"

# 8. Push to YOUR fork
git push -u origin feature/YOUR-FEATURE-NAME

# 9. On GitHub:
#    - Go to your fork
#    - Click "Compare & pull request"
#    - Write a clear PR description:
#      * What does this change?
#      * Why was it needed?
#      * How was it tested?
#    - Submit the PR

# 10. View the audit trail
git log --oneline --graph --all
```

**Bonus Group Activity:**

Organize with a classmate or colleague:
1. One person creates a shared repository on GitHub and adds the other as a collaborator
2. Both clone the same repository
3. Each creates a feature branch and edits the same file differently
4. Both push their branches
5. Merge them one at a time and observe/resolve conflicts
6. Discuss: What conflict resolution approach did you choose, and why?

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Remote** | A named reference to a repository hosted on a server (e.g., `origin`, `upstream`) |
| **`git remote`** | Command to view, add, rename, or remove remote connections |
| **`git push`** | Uploads commits from the local branch to its corresponding remote branch |
| **`git pull`** | Fetches changes from the remote and merges them into the current local branch |
| **`git fetch`** | Downloads changes from the remote without merging them into local branches |
| **`origin`** | Default name Git assigns to the remote a repo was cloned from |
| **`upstream`** | Convention for naming the original repository a fork was based on |
| **Fork** | A personal copy of someone else's repository on GitHub, under your own account |
| **Branch** | A lightweight, movable pointer to a specific commit — an independent line of development |
| **`git branch`** | Lists, creates, renames, or deletes branches |
| **`git checkout`** | Switches to a different branch or restores working tree files |
| **`git switch`** | Modern Git 2.23+ command for switching branches (clearer intent than `checkout`) |
| **`git merge`** | Combines the history of one branch into another |
| **`HEAD`** | A special pointer indicating the branch (or commit) you are currently on |
| **Fast-forward merge** | A merge where the target branch has no new commits — Git simply moves the pointer |
| **Three-way merge** | A merge that creates a new merge commit because both branches have diverged |
| **Merge conflict** | Occurs when two branches have changed the same lines of a file differently |
| **Pull Request (PR)** | A GitHub feature for proposing and reviewing changes before merging |
| **Branching strategy** | A team agreement on how branches are created, named, used, and merged |
| **Git Flow** | A branching strategy with `main`, `develop`, `feature`, `release`, and `hotfix` branches |
| **GitHub Flow** | A simple branching strategy: branch from `main`, PR, merge, deploy |
| **Trunk-Based Development** | Everyone commits frequently to a single main branch; features gated by flags |
| **Feature flag** | A configuration toggle that enables or disables a feature in production without deploying new code |
| **Detached HEAD** | A state where HEAD points directly to a commit rather than a branch |
| **`git tag`** | A permanent, named reference to a specific commit — typically used to mark releases |

---

## 📚 Resources

- [GitKraken's Guide to Git Branching Strategies](https://www.gitkraken.com/learn/git/best-practices/git-branch-strategy) — comprehensive comparison of branching strategies with diagrams
- [Atlassian — Git Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) — practical guide to branch-based collaboration
- [Atlassian — Git Flow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) — detailed Git Flow reference
- [GitHub Docs — About Pull Requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) — official PR documentation
- [Learn Git Branching (Interactive)](https://learngitbranching.js.org) — visual, hands-on branch and merge practice
- [Trunk Based Development](https://trunkbaseddevelopment.com) — the definitive reference for trunk-based development

---

## 🔭 Day 16 Preview: Advanced Git — Rebase, Stash & Undoing Changes

Tomorrow you go beyond the everyday workflow into Git's more powerful (and sometimes dangerous) tools.

You'll learn:
- `git rebase` — rewrite history for a cleaner, linear commit log
- `git stash` — temporarily shelve uncommitted changes so you can switch tasks
- `git revert` — safely undo a commit without rewriting history
- `git reset` — move the branch pointer backward (with varying levels of destructiveness)
- `git cherry-pick` — apply a single specific commit from one branch to another

These are the tools that separate a basic Git user from someone who can confidently manage complex repository states.

---

> 💬 *"A branch costs nothing to create and everything to skip. Always branch."*
>
> — DevOps Learning by Yukta
