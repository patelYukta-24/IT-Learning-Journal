# Day 16: Merging & Handling Conflicts

> **DevOps Transformation** | Git & Collaboration Series

---

## 📋 Overview

Yesterday you learned how to create branches and push to remotes. Today you face the inevitable consequence of parallel development: **merging**, and the conflicts that come with it.

Merge conflicts are not a sign that something went wrong. They are a normal, expected part of collaborative development — Git's way of saying "two people changed the same thing, and I need a human to decide what the final version should be." Knowing how to read a conflict, reason through it, and resolve it cleanly is one of the most practically valuable Git skills you can have.

By the end of today you will have deliberately created conflicts, resolved them at the command line, and understand exactly what Git is doing at each step.

---

## 🗺️ What You'll Learn Today

| Topic | Commands / Concepts | Why It Matters |
|---|---|---|
| Merging mechanics | `git merge` | Integrate completed work back into shared branches |
| Merge types | Fast-forward, three-way, squash | Choose the right merge strategy for the situation |
| Conflict anatomy | `<<<<<<<`, `=======`, `>>>>>>>` | Read and understand what Git is showing you |
| Conflict resolution | `git add`, `git commit`, `git merge --abort` | Resolve conflicts safely and completely |
| Prevention habits | Workflow practices | Reduce the frequency and complexity of conflicts |

---

## 🏢 NexusCorp Scenario

> Two NexusCorp engineers are working on the same configuration file. Engineer A is updating the database connection string for the new staging environment. Engineer B is adding a new logging section to the same file. Neither knows the other is editing it. When Engineer A tries to merge her branch into `main`, Git stops with a conflict.
>
> This is not a Git failure. This is Git doing exactly what it is supposed to do — catching a problem before it silently breaks the system. Today you learn how to handle it.

---

## Part 1: Merging in Depth

### What Merging Actually Does

When you merge branch B into branch A, Git:

1. Finds the **common ancestor** — the last commit both branches share
2. Collects all changes made in branch B since that point
3. Applies those changes to branch A
4. Creates a result — either by moving a pointer (fast-forward) or creating a new merge commit (three-way)

```
Common Ancestor: the commit where the two branches last agreed

       common
       ancestor
          │
          C1 ── C2 ── C3          ← main
          │
          └── C4 ── C5            ← feature-x

Git compares: C3 vs C5 vs C1
If no lines conflict → auto-merge
If lines conflict   → marks the file and asks you to decide
```

---

### Types of Merges

#### Fast-Forward Merge

Occurs when the target branch (e.g., `main`) has made **no new commits** since the feature branch diverged. Git simply moves the `main` pointer forward — no new commit is created.

```
Before:
  main:      A ── B
                   \
  feature-x:        C ── D

After (fast-forward):
  main:      A ── B ── C ── D
  feature-x: (same — or deleted)
```

```bash
git checkout main
git merge feature-x
# Output: Fast-forward
#         README.md | 2 ++
#         1 file changed, 2 insertions(+)
```

**When to use:** Appropriate for short-lived personal feature branches where a linear history is preferred.

**When to avoid:** When you want to preserve the fact that a feature was developed separately — use `--no-ff` to force a merge commit.

---

#### Three-Way Merge

Occurs when **both branches have made new commits** since they diverged. Git uses three snapshots: the common ancestor, the tip of the current branch, and the tip of the incoming branch. A new **merge commit** is created with two parents.

```
Before:
  main:      A ── B ── C
                   \
  feature-x:        D ── E

After (three-way merge):
  main:      A ── B ── C ── M
                   \       /
  feature-x:        D ── E
```

`M` is the merge commit — it has two parents (`C` and `E`) and records that the two lines of development were joined here.

```bash
git checkout main
git merge feature-x
# Output: Merge made by the 'ort' strategy.
#         feature_notes.txt | 1 +
#         1 file changed, 1 insertion(+)
```

---

#### Squash Merge

Combines all commits from the feature branch into a **single new commit** on the target branch. The feature branch history is compressed — only one clean commit appears on `main`.

```
Before:
  feature-x: D1 ── D2 ── D3 ── D4

After squash merge onto main:
  main: ... ── C ── S    (S contains all changes from D1–D4)
```

```bash
git checkout main
git merge --squash feature-x
git commit -m "Add feature X (squashed)"
```

**When to use:** When the feature branch has many messy "WIP" or "fix typo" commits that you don't want cluttering `main`'s history.

**Trade-off:** The individual commit history of the feature branch is lost. The feature branch itself is not automatically deleted and cannot be fast-forwarded later.

---

#### `--no-ff` — Always Create a Merge Commit

Forces a merge commit even when a fast-forward is possible. Preserves the record that a feature branch existed.

```bash
git merge --no-ff feature-x -m "Merge feature-x: add user authentication"
```

**When to use:** In team workflows where you want `git log --graph` to clearly show which commits belonged to which feature. Most pull request merges on GitHub use this behavior by default.

---

### Core Merge Commands

```bash
# Merge a branch into the current branch
git merge feature-x

# Merge with an explicit commit message
git merge feature-x -m "Merge feature-x into main"

# Force a merge commit (no fast-forward)
git merge --no-ff feature-x

# Squash all feature branch commits into one
git merge --squash feature-x

# Preview what a merge would change (dry run)
git diff main...feature-x

# Abort a merge in progress (return to pre-merge state)
git merge --abort

# After resolving conflicts, complete the merge
git add resolved_file.txt
git commit
```

---

## Part 2: Merge Conflicts

### What Causes a Conflict?

A conflict arises when:
- Branch A and Branch B have **both modified the same lines** of the same file
- One branch **deleted a file** that the other branch modified
- Both branches **added a file with the same name** but different content

Git can automatically merge changes to **different lines** of the same file — that is not a conflict. Only changes to the **same lines** require human resolution.

```
Branch main edits line 7 of config.yml
Branch feature-x also edits line 7 of config.yml
→ CONFLICT

Branch main edits line 7
Branch feature-x edits line 15
→ NO CONFLICT — Git merges automatically
```

---

### Anatomy of a Conflict Marker

When Git encounters a conflict, it edits the file directly and inserts **conflict markers** to show you exactly what the disagreement is:

```
<<<<<<< HEAD
database_host = "db-staging.nexuscorp.internal"
=======
database_host = "db-prod.nexuscorp.com"
>>>>>>> feature/update-db-config
```

| Section | Meaning |
|---|---|
| `<<<<<<< HEAD` | Start of your current branch's version |
| Content between `<<<<<<<` and `=======` | What **your current branch** (HEAD) has on this line |
| `=======` | Divider between the two conflicting versions |
| Content between `=======` and `>>>>>>>` | What the **incoming branch** has on this line |
| `>>>>>>> branch-name` | End of the incoming branch's version |

The markers and everything between them replace the original line(s) in the file. Your job is to delete all the markers and leave only the version the final file should have.

---

### A More Complex Conflict

Conflicts can span multiple lines:

```
<<<<<<< HEAD
[database]
host = db-staging.nexuscorp.internal
port = 5432
ssl = false
=======
[database]
host = db-prod.nexuscorp.com
port = 5432
ssl = true
max_connections = 100
>>>>>>> feature/prod-db-config
```

Here you might decide to keep the production host and SSL setting from the feature branch, but discuss `max_connections` before committing either value. The resolution is whatever the correct final content should be — Git does not dictate the outcome, only flags the disagreement.

---

### Step-by-Step Conflict Resolution

```bash
# Step 1: Attempt the merge
git merge feature/update-db-config
# Output:
# Auto-merging config/database.conf
# CONFLICT (content): Merge conflict in config/database.conf
# Automatic merge failed; fix conflicts and then commit the result.

# Step 2: See which files have conflicts
git status
# You will see:
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#         both modified:   config/database.conf

# Step 3: Open and inspect the conflicted file
nano config/database.conf
# or:
cat config/database.conf
# Find all <<<<<<< markers

# Step 4: Edit the file to the desired final state
# - Remove ALL conflict markers (<<<<<<, =======, >>>>>>>)
# - Keep the correct content
# - Save the file

# Step 5: Verify no conflict markers remain
grep -n "<<<<<<\|=======\|>>>>>>>" config/database.conf
# This should return nothing — if it does, you missed a marker

# Step 6: Stage the resolved file
git add config/database.conf

# Step 7: If there were multiple conflicted files, repeat Steps 3–6 for each
git status   # Confirm all conflicts are staged

# Step 8: Complete the merge
git commit
# Git pre-fills a merge commit message — review it and save
# Or supply your own:
git commit -m "Merge feature/update-db-config: resolve database host conflict"

# Step 9: Verify the final state
git log --oneline --graph
cat config/database.conf
```

---

### Aborting a Merge

If you realize mid-conflict that you need more information before resolving — or that the merge was a mistake — you can completely abandon it:

```bash
git merge --abort
```

This restores your working directory and staging area to exactly the state they were in before you ran `git merge`. It is safe to run at any point during a conflict, as long as you have not yet run `git commit`.

---

### Checking for Conflicts Before Merging

```bash
# See what differs between branches before merging
git diff main..feature-x

# See only which files differ (not the full diff)
git diff --name-only main..feature-x

# Show commits in feature-x not yet in main
git log main..feature-x --oneline

# Simulate the merge to preview conflicts (then abort)
git merge --no-commit --no-ff feature-x
git diff --cached    # Review what would be committed
git merge --abort    # Abort — nothing was actually committed
```

---

### Resolving Conflicts with a Merge Tool

For complex conflicts, a visual merge tool is faster than editing raw text:

```bash
# Configure a merge tool
git config --global merge.tool vimdiff     # Terminal visual diff
git config --global merge.tool vscode      # VS Code

# Open the merge tool for all conflicted files
git mergetool

# VS Code merge tool configuration (add to ~/.gitconfig)
# [merge]
#     tool = vscode
# [mergetool "vscode"]
#     cmd = code --wait $MERGED
```

In VS Code, a conflict presents four buttons above each conflict block:
- **Accept Current Change** — keep HEAD version
- **Accept Incoming Change** — keep the merging branch's version
- **Accept Both Changes** — keep both (appended)
- **Compare Changes** — side-by-side view

---

## Part 3: Why Conflict Resolution Matters

Merge conflicts are not just a technical inconvenience — they represent a coordination problem in the team. How your team handles them has direct consequences:

| If conflicts are ignored or handled poorly | If conflicts are resolved carefully |
|---|---|
| Code remains broken and undeployable | Integration happens smoothly |
| One developer's work silently overwrites another's | Both sets of changes are intentionally preserved or discarded |
| Bugs are introduced by partial or incorrect merges | The final state is reviewed and intentional |
| Team trust erodes | Team workflow stays productive |
| Release timelines slip | Deployments happen on schedule |

**Conflict resolution is a communication tool as much as a technical one.** When you resolve a conflict, you are making an architectural decision: which version of the truth should the codebase hold? That decision sometimes requires a conversation with a teammate, not just a command.

---

### Habits That Reduce Conflict Frequency

| Practice | How It Helps |
|---|---|
| **Pull from main frequently** | Keeps your feature branch close to current main — less divergence = fewer conflicts |
| **Keep branches short-lived** | The longer a branch lives, the more `main` moves while it's open |
| **Make small, focused commits** | Easier to identify what changed and why when resolving |
| **Communicate with teammates** | "I'm working on `config/database.conf` this sprint" prevents two people editing the same file |
| **Review diffs before merging** | `git diff main..feature-x` previews what will merge |
| **Use `git rebase`** | Replays your commits on top of the latest `main` — keeps history linear and reduces conflict surface |
| **Define file ownership** | In large codebases, agree informally who "owns" which parts |

---

## 🛠️ Practical Exercises

### Exercise 1: Simulate and Resolve a Merge Conflict

```bash
# Step 1: Create a fresh repo
mkdir conflict_practice
cd conflict_practice
git init -b main

# Step 2: Create an initial file and commit it on main
cat > config.conf << 'EOF'
[app]
name = NexusCorp App
version = 1.0

[database]
host = localhost
port = 5432
EOF

git add config.conf
git commit -m "Initial commit: app config"

# Step 3: Create and switch to a new branch
git checkout -b feature/update-db-host

# Step 4: Edit the database host on the feature branch
sed -i 's/host = localhost/host = db-staging.nexuscorp.internal/' config.conf
git add config.conf
git commit -m "Update database host for staging environment"

# Step 5: Switch to main and make a DIFFERENT change to the same line
git checkout main
sed -i 's/host = localhost/host = db-prod.nexuscorp.com/' config.conf
git add config.conf
git commit -m "Update database host for production"

# Step 6: Attempt to merge — observe the conflict
git merge feature/update-db-host
# CONFLICT (content): Merge conflict in config.conf

# Step 7: Inspect the conflict markers
cat config.conf

# Step 8: Resolve — choose the staging host (or write a comment noting both)
cat > config.conf << 'EOF'
[app]
name = NexusCorp App
version = 1.0

[database]
host = db-staging.nexuscorp.internal
port = 5432
EOF

# Step 9: Verify no markers remain
grep -c "<<<<<<" config.conf   # Should output 0

# Step 10: Stage and complete the merge
git add config.conf
git commit -m "Merge feature/update-db-host: use staging database host"

# Step 11: Review the final log
git log --oneline --graph
cat config.conf
```

---

### Exercise 2: Multi-File Conflict Resolution

```bash
# Continuing in conflict_practice repo

# Step 1: Create two more files
echo "LOG_LEVEL=INFO" > app.env
echo "MAX_WORKERS=4" > worker.conf
git add app.env worker.conf
git commit -m "Add app environment and worker config"

# Step 2: Create a branch and edit both files
git checkout -b feature/performance-tuning
sed -i 's/LOG_LEVEL=INFO/LOG_LEVEL=DEBUG/' app.env
sed -i 's/MAX_WORKERS=4/MAX_WORKERS=8/' worker.conf
git add app.env worker.conf
git commit -m "Tune logging and worker count for performance testing"

# Step 3: Edit the same lines differently on main
git checkout main
sed -i 's/LOG_LEVEL=INFO/LOG_LEVEL=WARNING/' app.env
sed -i 's/MAX_WORKERS=4/MAX_WORKERS=2/' worker.conf
git add app.env worker.conf
git commit -m "Reduce log verbosity and workers for production"

# Step 4: Merge and handle both conflicts
git merge feature/performance-tuning
# Two files will conflict

git status   # See both listed as "both modified"

# Resolve app.env
echo "LOG_LEVEL=INFO" > app.env     # Agreed middle ground
git add app.env

# Resolve worker.conf
echo "MAX_WORKERS=4" > worker.conf  # Agreed middle ground
git add worker.conf

# Complete the merge
git commit -m "Merge feature/performance-tuning: resolved config conflicts"

git log --oneline --graph
```

---

### Exercise 3: Collaborative Merge Challenge

Work through this with a classmate, colleague, or by simulating two "users" across two directories.

```bash
# ── Simulating two contributors locally ──────────────────────

# Setup: create a "shared" bare repository (simulates GitHub remote)
mkdir /tmp/shared_remote
git init --bare /tmp/shared_remote

# Contributor 1 clones and makes changes
cd /tmp
git clone /tmp/shared_remote contributor1
cd contributor1
echo "# NexusCorp Project" > README.md
echo "Initial setup by Contributor 1" >> README.md
git add README.md
git commit -m "Initial commit by Contributor 1"
git push origin main

# Contributor 2 clones and makes DIFFERENT changes to same line
cd /tmp
git clone /tmp/shared_remote contributor2
cd contributor2
# Edit line 2 of README.md differently
echo "# NexusCorp Project" > README.md
echo "Initial setup by Contributor 2 (different version)" >> README.md
git add README.md
git commit -m "Initial commit by Contributor 2"

# Contributor 2 tries to push — will be rejected (behind remote)
git push origin main
# rejected: Updates were rejected because the remote contains work you do not have

# Contributor 2 must pull first, then resolve the conflict
git pull origin main
# CONFLICT (content): Merge conflict in README.md

cat README.md   # Inspect conflict markers

# Resolve: combine both contributors' names
cat > README.md << 'EOF'
# NexusCorp Project

Initial setup — joint contribution by Contributors 1 and 2.
EOF

git add README.md
git commit -m "Merge: resolve README conflict between contributors"
git push origin main

# Contributor 1 pulls the resolved state
cd /tmp/contributor1
git pull origin main
cat README.md   # Should show the resolved version
```

---

## 🔧 Advanced Challenge: Intentional Conflict Gauntlet

**Scenario:** Deliberately engineer multiple conflicts across different file types, then resolve them all before completing a single merge.

```bash
# Step 1: Set up the project
mkdir /tmp/conflict_gauntlet
cd /tmp/conflict_gauntlet
git init -b main

cat > app.py << 'EOF'
# NexusCorp Application
APP_NAME = "nexuscorp"
VERSION = "1.0.0"
DEBUG = False

def start():
    print(f"Starting {APP_NAME} v{VERSION}")

def stop():
    print("Shutting down.")
EOF

cat > requirements.txt << 'EOF'
flask==2.3.0
requests==2.31.0
EOF

cat > README.md << 'EOF'
# NexusCorp App
A demo application for conflict resolution practice.
EOF

git add .
git commit -m "Initial project scaffold"

# Step 2: Make changes on a feature branch
git checkout -b feature/v2-upgrade
sed -i 's/VERSION = "1.0.0"/VERSION = "2.0.0"/' app.py
sed -i 's/DEBUG = False/DEBUG = True/' app.py
sed -i 's/flask==2.3.0/flask==3.0.0/' requirements.txt
echo "pytest==7.4.0" >> requirements.txt
echo "Version 2.0.0 introduces debug mode." >> README.md
git add .
git commit -m "Upgrade to v2.0.0 with debug mode"

# Step 3: Make CONFLICTING changes on main
git checkout main
sed -i 's/VERSION = "1.0.0"/VERSION = "1.1.0"/' app.py
sed -i 's/DEBUG = False/DEBUG = False  # Never enable in prod/' app.py
sed -i 's/flask==2.3.0/flask==2.3.3/' requirements.txt
echo "## Changelog" >> README.md
echo "- v1.1.0: Patch release" >> README.md
git add .
git commit -m "Patch release 1.1.0 with production safety comment"

# Step 4: Attempt the merge — observe all three conflicts
git merge feature/v2-upgrade

# Step 5: List all conflicted files
git status

# Step 6: Resolve each file one by one
# (Your decisions — document why you chose each resolution)

# Step 7: Verify no markers remain across all files
grep -rn "<<<<<<\|=======\|>>>>>>>" .

# Step 8: Stage all resolved files and complete the merge
git add .
git commit -m "Merge feature/v2-upgrade: resolved version, debug, and dependency conflicts"

# Step 9: Final audit
git log --oneline --graph
git show HEAD   # Review the merge commit content
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Merge** | The process of integrating the history of one branch into another |
| **`git merge`** | Command that combines a specified branch into the currently checked-out branch |
| **Fast-forward merge** | A merge where the target branch has no new commits — Git moves the pointer forward without a new commit |
| **Three-way merge** | A merge that creates a new merge commit because both branches have diverged from their common ancestor |
| **Squash merge** | Combines all commits from a feature branch into a single commit on the target branch |
| **Merge commit** | A commit with two parents, recording the point where two branches were joined |
| **`--no-ff`** | Flag that forces a merge commit even when a fast-forward is possible |
| **Common ancestor** | The last commit shared between two branches before they diverged |
| **Merge conflict** | A state where Git cannot automatically resolve differences because both branches changed the same lines |
| **Conflict marker** | Special lines (`<<<<<<<`, `=======`, `>>>>>>>`) Git inserts into conflicted files to show both versions |
| **`<<<<<<< HEAD`** | Marks the beginning of the current branch's version in a conflict |
| **`=======`** | Divides the two conflicting versions in a conflict marker block |
| **`>>>>>>> branch`** | Marks the end of the incoming branch's version in a conflict |
| **`git merge --abort`** | Cancels a merge in progress and returns to the pre-merge state |
| **`git mergetool`** | Opens a configured visual tool to assist with resolving conflicts |
| **Unmerged path** | Git's term for a file that has an unresolved conflict |
| **`git diff main..feature`** | Shows the difference between two branches before merging |
| **Rebase** | An alternative to merging that replays commits on top of another branch for a linear history |
| **Pull Request (PR)** | A proposal to merge one branch into another, typically including review and discussion |

---

## 📚 Resources

- [Resolving a Merge Conflict Using the Command Line — GitHub Docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line) — official step-by-step guide
- [Atlassian — Git Merge](https://www.atlassian.com/git/tutorials/using-branches/git-merge) — deep dive into merge types with diagrams
- [Atlassian — Resolve Merge Conflicts](https://www.atlassian.com/git/tutorials/using-branches/merge-conflicts) — practical conflict resolution guide
- [Pro Git — Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging) — the definitive free reference

---

## 🔭 Day 17 Preview: Advanced Git — Rebase, Stash, Reset & More

Tomorrow you go beyond everyday Git into the tools that give you fine-grained control over your repository's history.

You'll learn:
- `git rebase` — rewrite and linearize commit history
- `git stash` — shelve uncommitted work temporarily
- `git reset` — move the branch pointer backward (soft, mixed, hard)
- `git revert` — safely undo a commit without rewriting history
- `git cherry-pick` — apply a single commit from one branch to another
- `git log` advanced flags — search, filter, and visualize history

These tools handle the situations that basic Git can't: the commit you made to the wrong branch, the half-finished work you need to set aside, the single bug fix you need to backport.

---

> 💬 *"A merge conflict is not Git failing — it is Git succeeding at the one thing no tool can do automatically: asking a human to make a decision."*
>
> — DevOps Learning by Yukta
