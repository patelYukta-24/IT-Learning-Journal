# Day 17: Advanced Rebasing, Git Log, Stash, Reset & More

> **DevOps Transformation** | Git & Collaboration Series

---

## 📋 Overview

Yesterday you learned how to combine branches through merging. Today you learn how to *rewrite*, *inspect*, *undo*, and *selectively apply* history — the advanced Git toolkit that separates confident Git users from beginners who are afraid to touch anything they didn't just commit.

Each tool today solves a real problem that shows up constantly in professional workflows: the half-finished work you need to set aside, the commit you made to the wrong branch, the three "WIP" commits you want to collapse before a code review, the production hotfix that needs to go to three branches at once.

By the end of today, none of those situations will be scary.

---

## 🗺️ What You'll Learn Today

| Topic | Commands | Why It Matters |
|---|---|---|
| Rebasing | `git rebase` | Replay commits on a new base for a linear history |
| Git Log | `git log` and flags | Search, filter, and visualize commit history |
| Stash | `git stash` | Shelve uncommitted work and pick it up later |
| Reset | `git reset` | Move branch pointer backward to undo commits |
| Cherry-Pick | `git cherry-pick` | Apply a single specific commit to another branch |
| Other concepts | `.gitconfig`, `.git/` folder | Understand Git's internals and personalize your setup |

---

## 🏢 NexusCorp Scenario

> The NexusCorp release engineer is preparing a hotfix. She needs to: stash her half-finished feature work, cherry-pick last week's bug fix onto the release branch, rebase her feature branch to avoid a messy merge commit, and use `git log` to prove to the audit team exactly who changed what and when.
>
> Every tool she uses today is one you'll learn in the next hour.

---

## Part 1: Advanced Rebasing

### What Is Rebase?

Rebasing moves or replays a sequence of commits onto a new base commit. Instead of creating a merge commit that joins two diverged histories, rebase **rewrites** the feature branch commits as if they had been made on top of the latest version of the target branch.

**Merge vs Rebase — the core difference:**

```
Starting point (both branches diverged from C2):

  main:       C1 ── C2 ── C3 ── C4
                     \
  feature-x:          D1 ── D2


After git merge main (into feature-x):

  main:       C1 ── C2 ── C3 ── C4
                     \              \
  feature-x:          D1 ── D2 ── M (merge commit)


After git rebase main (onto feature-x):

  main:       C1 ── C2 ── C3 ── C4
                                   \
  feature-x:                        D1' ── D2'
```

Note: After rebase, `D1` and `D2` become `D1'` and `D2'` — they are **new commits** with new SHA hashes, even if their content is identical. The original commits still exist in Git's object store but are no longer referenced by any branch.

---

### Basic Rebase

```bash
# Switch to the branch you want to rebase
git checkout feature-x

# Replay feature-x commits on top of latest main
git rebase main

# If conflicts arise during rebase:
# 1. Resolve them in the conflicted file
# 2. Stage the resolved file
git add resolved_file.txt
# 3. Continue the rebase
git rebase --continue

# To abort the rebase and return to original state
git rebase --abort

# To skip the current conflicting commit (use carefully)
git rebase --skip
```

---

### Interactive Rebase — Rewriting History

Interactive rebase (`-i`) lets you **edit, reorder, squash, rename, or drop** individual commits before replaying them. It is one of the most powerful Git tools for cleaning up history before a code review.

```bash
# Interactively rebase the last 3 commits
git rebase -i HEAD~3

# Interactively rebase back to a specific commit
git rebase -i abc1234
```

Git opens your editor with a list like this:

```
pick a1b2c3d Add login form HTML
pick b2c3d4e Add login form CSS
pick c3d4e5f Add login form validation
pick d4e5f6g Fix typo in login form

# Commands:
# p, pick   = use commit as-is
# r, reword = use commit, but edit the commit message
# e, edit   = use commit, but stop for amending
# s, squash = combine with previous commit (keeps both messages)
# f, fixup  = combine with previous commit (discards this message)
# d, drop   = remove this commit entirely
# reorder lines to reorder commits
```

**Common interactive rebase operations:**

```bash
# Squash last 4 commits into one clean commit
git rebase -i HEAD~4
# Change 'pick' to 's' or 'f' on commits 2, 3, 4
# Write a single clear commit message for the result

# Rename a commit message
git rebase -i HEAD~2
# Change 'pick' to 'r' on the commit to rename

# Drop a commit entirely
git rebase -i HEAD~3
# Change 'pick' to 'd' on the unwanted commit

# Split a commit into two
git rebase -i HEAD~2
# Change 'pick' to 'e' on the commit to split
# When Git pauses: git reset HEAD~1, then add and commit in parts
# Then: git rebase --continue
```

---

### When to Use Rebase vs Merge

| Use Rebase when... | Use Merge when... |
|---|---|
| You want a clean, linear history | You want to preserve the full branching record |
| Cleaning up commits before a PR | Recording that a feature branch existed |
| Incorporating latest `main` changes into your feature branch | Joining long-lived branches (e.g., `develop` into `main`) |
| Working on a private local branch | The branch has already been pushed and shared |

**🚨 The Golden Rule of Rebase:** Never rebase a branch that has been pushed to a shared remote and pulled by other developers. Rebase rewrites commit hashes — anyone who pulled the old commits will have a diverged history. Only rebase local, private branches.

---

## Part 2: Git Log — Inspecting History

`git log` is how you read the story of your repository. Far more than just "show recent commits," it is a powerful query tool for understanding exactly what changed, when, and why.

---

### Core `git log` Usage

```bash
# Full commit log (most recent first)
git log

# Compact one-line view — SHA + message only
git log --oneline

# Show last N commits
git log -5
git log -10 --oneline

# Visual branch graph in the terminal
git log --oneline --graph --all

# Show full diff for each commit
git log -p

# Show stats (files changed, lines added/removed) per commit
git log --stat

# Show a summary of changes without the full diff
git log --shortstat
```

---

### Filtering Commits

```bash
# Filter by author
git log --author="Dev Engineer"
git log --author="@nexuscorp.com"   # Partial match works

# Filter by date range
git log --since="2024-01-01"
git log --until="2024-06-30"
git log --since="2 weeks ago"
git log --since="yesterday"

# Filter by commit message keyword
git log --grep="hotfix"
git log --grep="bug" --grep="fix" --all-match   # Both terms must match

# Filter by file — only show commits that touched a specific file
git log -- src/auth/login.py
git log --follow -- src/auth/login.py   # Follow renames

# Filter commits that changed a specific string in the code
git log -S "database_host"    # Pickaxe search — added or removed this string
git log -G "regex pattern"    # Regex version of pickaxe

# Show commits in branch A not yet in branch B
git log main..feature-x --oneline

# Show commits that are unique to either branch
git log main...feature-x --oneline
```

---

### Formatting Output

```bash
# Custom format
git log --pretty=format:"%h  %an  %ar  %s"
# %h  = short SHA
# %H  = full SHA
# %an = author name
# %ae = author email
# %ar = relative date (e.g., "3 days ago")
# %ad = absolute date
# %s  = subject (first line of commit message)
# %b  = body (rest of commit message)

# Show date in a specific format
git log --date=short --format="%ad %an: %s"

# Combine graph with custom format
git log --graph --pretty=format:"%C(yellow)%h%Creset %s %C(blue)(%ar)%Creset" --all
```

---

### Useful Aliases for Git Log

Add these to `~/.gitconfig` to save typing:

```bash
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.hist "log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short"
git config --global alias.last "log -1 HEAD"

# Then use:
git lg
git hist
git last
```

---

### `git show` — Inspect a Specific Commit

```bash
# Show the full diff and metadata of the most recent commit
git show

# Show a specific commit by hash
git show a1b2c3d

# Show only the files changed (not the full diff)
git show --name-only a1b2c3d

# Show a specific file as it was at a specific commit
git show a1b2c3d:src/app.py
```

---

### `git blame` — Who Changed Each Line?

```bash
# Show who last changed each line of a file
git blame src/auth/login.py

# Show blame for a specific line range
git blame -L 10,25 src/auth/login.py

# Show blame ignoring whitespace changes
git blame -w src/auth/login.py
```

---

## Part 3: Stash — Set Work Aside Temporarily

`git stash` saves your uncommitted changes (both staged and unstaged) to a temporary stack, leaving your working directory clean. This lets you switch branches, pull updates, or work on something urgent — then come back and restore exactly where you left off.

---

### Core Stash Commands

```bash
# Stash current changes (staged + unstaged)
git stash

# Stash with a descriptive label (strongly recommended)
git stash save "WIP: feature-login form validation"
# Modern syntax:
git stash push -m "WIP: feature-login form validation"

# List all stashed entries
git stash list
# Output:
# stash@{0}: On feature-login: WIP: feature-login form validation
# stash@{1}: On main: WIP: hotfix attempt

# Apply the most recent stash (keeps it in the stash list)
git stash apply

# Apply a specific stash by index
git stash apply stash@{1}

# Apply AND remove from stash list (pop = apply + drop)
git stash pop

# Remove a specific stash
git stash drop stash@{0}

# Remove ALL stashes
git stash clear

# Show what a stash contains
git stash show stash@{0}
git stash show -p stash@{0}   # Full diff
```

---

### Stashing Untracked and Ignored Files

```bash
# By default, stash does NOT include untracked files
# Include untracked files with -u
git stash push -u -m "Include new untracked files too"

# Include ignored files (rare — be careful)
git stash push -a -m "Include ignored files"
```

---

### Creating a Branch from a Stash

```bash
# Apply a stash onto a new branch
# Useful when the stash conflicts with current branch state
git stash branch new-branch-name stash@{0}
```

---

### Real-World Stash Scenarios

| Scenario | Stash Command |
|---|---|
| Urgent bug reported — need to switch branches without committing WIP | `git stash push -m "WIP: feature in progress"` |
| Need to pull latest changes before finishing current work | `git stash` → `git pull` → `git stash pop` |
| Accidentally started work on wrong branch | `git stash` → `git checkout correct-branch` → `git stash pop` |
| Want to test something clean without losing current work | `git stash` → test → `git stash pop` |

---

## Part 4: Reset — Moving the Branch Pointer

`git reset` moves the current branch's `HEAD` pointer to a specified commit. Depending on the flag, it also changes what happens to the staging area and working directory.

---

### The Three Modes of Reset

```
          Working     Staging     Commit
          Directory   Area        History
          ─────────   ───────     ───────
--soft    unchanged   unchanged   pointer moves
--mixed   unchanged   cleared     pointer moves  (default)
--hard    cleared     cleared     pointer moves
```

```bash
# --soft: Move HEAD back, keep staged changes and working directory
# Changes appear as "to be committed" after reset
git reset --soft HEAD~1     # Undo last commit, keep changes staged
git reset --soft HEAD~3     # Undo last 3 commits, keep all changes staged

# --mixed (default): Move HEAD back, unstage changes, keep working directory
# Changes appear as "not staged" after reset
git reset HEAD~1            # Undo last commit, unstage changes (files unchanged)
git reset HEAD~2            # Undo last 2 commits

# --hard: Move HEAD back, discard ALL changes in staging and working directory
# DESTRUCTIVE — changes are gone
git reset --hard HEAD~1     # Undo last commit, discard all changes
git reset --hard HEAD~3     # Undo last 3 commits entirely

# Reset to a specific commit hash
git reset --hard a1b2c3d

# Reset a single file to its state at HEAD (unstage without changing content)
git reset HEAD filename.txt
```

---

### `git reset` vs `git revert`

This is a critical distinction — both undo changes but in fundamentally different ways:

| | `git reset` | `git revert` |
|---|---|---|
| **What it does** | Moves branch pointer backward — rewrites history | Creates a new commit that undoes a previous commit |
| **History** | Removes commits from history | Preserves all history, adds a new undo commit |
| **Safe to use on shared branches?** | **No** — rewrites history others may have | **Yes** — additive, doesn't touch existing commits |
| **Recoverable?** | Yes, via `git reflog` within ~90 days | Always — the original commit still exists |
| **Use case** | Undoing local, unpushed commits | Undoing commits that are already on shared/remote branches |

```bash
# Safely undo a specific commit that's already been pushed
git revert a1b2c3d
# Creates a new commit that reverses the changes from a1b2c3d

# Revert without immediately committing (review first)
git revert --no-commit a1b2c3d
git revert --continue   # After reviewing staged changes
```

---

### Recovering from a Hard Reset — `git reflog`

`git reflog` records every position `HEAD` has been — including commits that are no longer on any branch after a reset. It is your safety net.

```bash
# View reflog — all recent HEAD positions
git reflog

# Output example:
# a1b2c3d HEAD@{0}: reset: moving to HEAD~2
# d4e5f6g HEAD@{1}: commit: Add payment gateway
# e5f6g7h HEAD@{2}: commit: Update cart logic

# Recover commits after an accidental hard reset
git reset --hard e5f6g7h   # Go back to the commit before the reset
# Or create a branch at the lost commit:
git branch recovery-branch e5f6g7h
```

**Reflog entries expire after 90 days by default** — this is your window for recovery.

---

## Part 5: Cherry-Picking — Applying Specific Commits

`git cherry-pick` applies the changes introduced by one or more specific commits onto the current branch, creating new commits with the same content but new hashes.

```bash
# Apply a single commit to the current branch
git cherry-pick a1b2c3d

# Apply multiple specific commits
git cherry-pick a1b2c3d b2c3d4e c3d4e5f

# Apply a range of commits
git cherry-pick a1b2c3d..e5f6g7h   # Exclusive of first commit
git cherry-pick a1b2c3d^..e5f6g7h  # Inclusive of first commit

# Cherry-pick without immediately committing (stage only)
git cherry-pick --no-commit a1b2c3d

# Continue after resolving cherry-pick conflicts
git cherry-pick --continue

# Abort a cherry-pick in progress
git cherry-pick --abort
```

---

### Real-World Cherry-Pick Scenarios

| Scenario | How Cherry-Pick Helps |
|---|---|
| A bug fix is on `develop` but production (`main`) needs it now | Cherry-pick the fix commit onto `main` without merging all of `develop` |
| A commit was accidentally made on the wrong branch | Cherry-pick it to the correct branch, then reset/drop from the wrong one |
| Backporting a feature to an older release branch | Cherry-pick the specific commit(s) to the maintenance branch |
| Extracting one piece of a large, mixed commit | Cherry-pick with `--no-commit`, then stage only the parts you want |

---

### Cherry-Pick vs Merge vs Rebase

| | Cherry-Pick | Merge | Rebase |
|---|---|---|---|
| **Scope** | Single commit(s) | Entire branch | Entire branch |
| **Creates new commit?** | Yes | Only for three-way | Yes (replayed commits) |
| **Rewrites history?** | No (adds new commits) | No | Yes |
| **Preserves context?** | No — commit is detached from its branch | Yes | Yes |
| **Best for** | Targeted hotfix backports | Joining branches | Linearizing history |

---

## Part 6: Other Useful Concepts

### `.gitconfig` — Git Configuration File

`~/.gitconfig` is where your global Git settings live. You can set user info, define aliases, configure tools, and change default behaviors.

```bash
# View the full config file
cat ~/.gitconfig

# Sample ~/.gitconfig:
[user]
    name = Dev Engineer
    email = dev@nexuscorp.com

[core]
    editor = nano
    autocrlf = input       # Normalize line endings on commit

[init]
    defaultBranch = main

[pull]
    rebase = false         # Use merge strategy for git pull

[alias]
    st = status
    co = checkout
    br = branch
    lg = log --oneline --graph --all
    undo = reset --soft HEAD~1
    save = stash push -m
    last = log -1 HEAD

[merge]
    tool = vscode

[diff]
    tool = vscode
```

**Add aliases directly from the command line:**
```bash
git config --global alias.st status
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.lg "log --oneline --graph --all"
```

---

### The `.git/` Folder — Git's Internal Database

Every Git repository contains a hidden `.git/` directory at its root. This folder is Git's entire brain — delete it and the directory is no longer a repository.

```bash
ls -la .git/
```

| Path | Contents |
|---|---|
| `.git/HEAD` | Points to the current branch (e.g., `ref: refs/heads/main`) |
| `.git/config` | Local repository configuration (overrides `~/.gitconfig`) |
| `.git/objects/` | All content — every commit, file, tree — stored as compressed objects |
| `.git/refs/heads/` | One file per local branch, containing its tip commit SHA |
| `.git/refs/remotes/` | Remote-tracking branch references (`origin/main`, etc.) |
| `.git/refs/tags/` | Tag references |
| `.git/COMMIT_EDITMSG` | The message from the last commit (used by editor when committing) |
| `.git/MERGE_HEAD` | Present during a merge — contains the SHA of the branch being merged |
| `.git/logs/` | Reflog data — the history of where HEAD and each branch has pointed |
| `.git/index` | The staging area — a binary file tracking what will go in the next commit |

**You should never manually edit anything inside `.git/`** — Git manages it entirely. Knowing what's there helps you understand what Git commands actually do under the hood.

---

## 🛠️ Practical Exercises

### Exercise 1: Rebase Playground

```bash
# Step 1: Create a fresh repo
mkdir rebase_practice
cd rebase_practice
git init -b main

# Step 2: Create initial commits on main
echo "Project start" > project.txt
git add project.txt && git commit -m "Initial commit"

echo "Main: update 1" >> project.txt
git commit -am "Main update 1"

echo "Main: update 2" >> project.txt
git commit -am "Main update 2"

# Step 3: Create a feature branch from the first commit
git checkout -b feature-x HEAD~2

echo "Feature: work A" > feature.txt
git add feature.txt && git commit -m "Feature work A"

echo "Feature: work B" >> feature.txt
git commit -am "Feature work B"

# Step 4: See the diverged history
git log --oneline --graph --all

# Step 5: Rebase feature-x onto main
git rebase main

# Step 6: Observe the linearized history
git log --oneline --graph --all
# feature-x should now sit on top of main's latest commit

# Bonus: Try an interactive rebase to squash the two feature commits
git rebase -i HEAD~2
# Change second 'pick' to 'f' (fixup)
# Save and exit

git log --oneline --graph --all
```

---

### Exercise 2: Stashing Challenge

```bash
# Step 1: Start some work on a branch (do NOT commit)
git checkout -b feature-payment
echo "half-finished payment code" > payment.py

# Step 2: An urgent bug is reported — you need to switch branches
# But you can't commit unfinished work
git stash push -m "WIP: payment module — half finished"

# Step 3: Verify working directory is clean
git status   # Should be clean

# Step 4: Switch to main and do the urgent work
git checkout main
echo "urgent fix applied" >> project.txt
git commit -am "Hotfix: apply urgent production fix"

# Step 5: Return to feature branch and restore stashed work
git checkout feature-payment
git stash list                    # Confirm the stash is there
git stash pop                     # Restore and remove from stash

# Step 6: Verify your work is back
cat payment.py
git status
```

---

### Exercise 3: Reset Game

```bash
# Step 1: Make three sequential commits
echo "Change 1" > reset_test.txt
git add reset_test.txt && git commit -m "Commit 1: first change"

echo "Change 2" >> reset_test.txt
git commit -am "Commit 2: second change"

echo "Change 3" >> reset_test.txt
git commit -am "Commit 3: third change"

# Step 2: View the current history
git log --oneline
# Commit 3 is HEAD

# Step 3: Soft reset — go back 2 commits, keep changes staged
git reset --soft HEAD~2
git log --oneline    # Now at Commit 1
git status           # Changes from Commit 2 and 3 are staged

# Step 4: Restore for next test
git commit -m "Re-commit: combined changes 2 and 3"

# Step 5: Mixed reset (default) — unstage changes but keep files
git reset HEAD~1
git status           # Changes are unstaged
cat reset_test.txt   # File still has all content

# Step 6: Hard reset — discard everything
git add . && git commit -m "Recommit before hard reset"
git reset --hard HEAD~1
cat reset_test.txt   # The last change is gone
git status           # Clean

# Step 7: Use reflog to recover
git reflog
# Find the commit hash before the hard reset and recover it
```

---

### Exercise 4: Cherry-Picking Exercise

```bash
# Step 1: On main, create a commit you want to copy
git checkout main
echo "Critical bug fix" > bugfix.txt
git add bugfix.txt && git commit -m "Fix: resolve critical authentication bug"

# Note the commit hash
BUG_FIX_HASH=$(git log --oneline -1 | awk '{print $1}')
echo "Bug fix commit hash: $BUG_FIX_HASH"

# Step 2: Switch to a release branch that also needs this fix
git checkout -b release/v1.1
git log --oneline   # The bugfix is NOT here yet

# Step 3: Cherry-pick the fix onto the release branch
git cherry-pick $BUG_FIX_HASH

# Step 4: Verify the fix is now on the release branch
git log --oneline
cat bugfix.txt

# Step 5: Confirm main still has its own copy of the commit
git checkout main
git log --oneline
```

---

## 💼 Interview Questions & Answers

These questions come up frequently in DevOps and developer interviews. Work through your own answers, then compare.

---

**Q1: What is the difference between `git merge` and `git rebase`?**

Both integrate changes from one branch into another, but differently:

- `git merge` creates a **merge commit** with two parents, preserving the full branching history. The commit graph shows that two lines of development joined at a point in time.
- `git rebase` **replays** commits from the feature branch on top of the target branch, producing a linear history with no merge commit. The commits are rewritten with new SHAs.

Use **merge** when you want to preserve the record of how branches evolved. Use **rebase** to produce a clean, linear history — typically for local feature branches before a pull request. Never rebase shared/published branches.

---

**Q2: How would you undo the last commit?**

It depends on whether you want to keep the changes:

```bash
# Keep changes staged (soft reset)
git reset --soft HEAD~1

# Keep changes but unstaged (mixed reset — default)
git reset HEAD~1

# Discard all changes entirely (hard reset — destructive)
git reset --hard HEAD~1

# If the commit has already been pushed to a shared branch — use revert:
git revert HEAD    # Creates a new commit that undoes the last one
```

---

**Q3: Describe a scenario where you might use `git stash`.**

You're in the middle of developing a new feature (files modified, not committed) when your manager reports a critical production bug that needs an immediate fix. You can't commit half-finished code, but you also can't leave your working directory dirty when switching branches.

Solution:
```bash
git stash push -m "WIP: feature development in progress"
git checkout main
git checkout -b hotfix/critical-bug
# ... fix the bug, commit, push, merge ...
git checkout feature-branch
git stash pop    # Your work is exactly as you left it
```

---

**Q4: What is the difference between `git reset` and `git revert`?**

| | `git reset` | `git revert` |
|---|---|---|
| **Mechanism** | Moves branch pointer backward — removes commits from history | Creates a new commit that reverses the changes |
| **History** | Rewrites history | Preserves history |
| **Safe on shared branches?** | No — breaks others who have pulled those commits | Yes — purely additive |
| **Recovery** | Via `git reflog` within ~90 days | Always recoverable — original commit still exists |
| **When to use** | Undoing local, unpushed commits | Undoing commits already on shared or remote branches |

---

## 🔧 Mini-Project: Full Advanced Workflow

**Scenario:** Bring together all of today's tools in a realistic sequence.

```bash
# Setup
mkdir nexus_advanced_git
cd nexus_advanced_git
git init -b main

echo "# NexusCorp Advanced Git Demo" > README.md
git add . && git commit -m "Initial commit"

# ── 1. Stash: interrupted mid-work ────────────────────
git checkout -b feature/dashboard
echo "dashboard in progress" > dashboard.py
git stash push -m "WIP: dashboard — interrupted"
git checkout main

# ── 2. Cherry-pick: hotfix needs to go to two branches ─
echo "def fix_auth(): pass" > auth_fix.py
git add auth_fix.py
git commit -m "Hotfix: resolve auth bypass vulnerability"
HOTFIX_SHA=$(git log --oneline -1 | awk '{print $1}')

git checkout -b release/v1.0
git cherry-pick $HOTFIX_SHA
git log --oneline   # Hotfix is now on release branch

# ── 3. Rebase: clean up feature branch ─────────────────
git checkout main
echo "Main: base update" >> README.md
git commit -am "Update README on main"

git checkout feature/dashboard
git stash pop    # Restore WIP work
git add . && git commit -m "WIP: dashboard initial draft"
echo "dashboard v2" >> dashboard.py
git commit -am "Dashboard: add v2 layout"

# Rebase feature branch onto latest main
git rebase main
git log --oneline --graph --all

# ── 4. Interactive rebase: squash WIP commits ───────────
git rebase -i HEAD~2
# Squash second commit into first
# Write one clean commit message: "Add dashboard module"

# ── 5. Inspect full history ─────────────────────────────
git log --oneline --graph --all
git log --stat -3
git log --author="$(git config user.name)" --oneline
```

---

## 📖 Glossary

| Term | Definition |
|---|---|
| **Rebase** | Replays commits from one branch on top of another, producing a linear history |
| **Interactive rebase (`-i`)** | Rebase mode that lets you edit, squash, reorder, rename, or drop individual commits |
| **`git rebase --continue`** | Resumes a paused rebase after resolving conflicts |
| **`git rebase --abort`** | Cancels an in-progress rebase and returns to the original state |
| **Squash** | Combining multiple commits into one — commonly done via interactive rebase |
| **`git log`** | Displays the commit history of the current branch |
| **`--oneline`** | Flag showing one commit per line (short SHA + message) |
| **`--graph`** | Flag showing a visual ASCII representation of the branch topology |
| **Pickaxe (`-S`)** | `git log` flag that finds commits where a specific string was added or removed |
| **`git blame`** | Shows which commit and author last modified each line of a file |
| **`git show`** | Displays the diff and metadata of a specific commit |
| **`git stash`** | Temporarily saves uncommitted changes to a stack, leaving the working directory clean |
| **`git stash pop`** | Applies the most recent stash and removes it from the stash list |
| **`git stash apply`** | Applies the most recent stash but keeps it in the stash list |
| **`git reset --soft`** | Moves HEAD backward; keeps changes staged |
| **`git reset --mixed`** | Moves HEAD backward; unstages changes but keeps files (default behavior) |
| **`git reset --hard`** | Moves HEAD backward; discards all changes in staging and working directory |
| **`git revert`** | Creates a new commit that undoes a previous commit — safe for shared branches |
| **`git reflog`** | Records all positions HEAD has been — used to recover from destructive resets |
| **`git cherry-pick`** | Applies the changes of a specific commit onto the current branch |
| **`.gitconfig`** | The configuration file where Git user info, aliases, and settings are stored |
| **`.git/`** | The hidden directory containing Git's full internal database for a repository |
| **`git stash branch`** | Creates a new branch and applies a stash onto it |

---

## 📚 Resources

- [Pro Git — Git Tools: Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) — deep dive into rebase, interactive rebase, and amend
- [Pro Git — Git Tools: Stashing and Cleaning](https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning) — complete stash reference
- [Atlassian — Git Reset](https://www.atlassian.com/git/tutorials/undoing-changes/git-reset) — visual guide to soft, mixed, and hard reset
- [Atlassian — Git Cherry Pick](https://www.atlassian.com/git/tutorials/cherry-pick) — practical cherry-pick guide with diagrams
- [Atlassian — Git Rebase](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase) — interactive rebase and golden rule explained
- [devhints.io — Git Cheat Sheet](https://devhints.io/git-log) — quick reference for `git log` format strings

---

## 🔭 What's Next: Git in a DevOps Pipeline

The Git skills from Days 13–17 are the foundation. The next step is connecting them to the DevOps workflow:

- **GitHub Actions / GitLab CI** — trigger pipelines on push, PR, or merge to specific branches
- **Branch protection rules** — enforce code review and passing CI before any merge to `main`
- **Semantic versioning with Git tags** — `git tag -a v1.2.0 -m "Release 1.2.0"`
- **GitOps** — using Git as the single source of truth for infrastructure state

Every CI/CD pipeline you'll ever work with starts with a `git push`. The history you've built over Days 13–17 is what makes the rest of the DevOps toolchain possible.

---

> 💬 *"Git's advanced tools are not about rewriting the past. They are about making the past clean enough to be useful in the future."*
>
> — DevOps Learning by Yukta
