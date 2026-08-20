# Day 56: Understanding Ad-hoc Commands in Ansible and Hands-On

## 📋 Table of Contents
- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [1. Introduction to Ad-hoc Commands](#1-introduction-to-ad-hoc-commands)
- [2. Ad-hoc Command Tasks](#2-ad-hoc-command-tasks)
- [3. Ansible Video Blog Task](#3-ansible-video-blog-task)
- [4. Troubleshooting Notes](#4-troubleshooting-notes)
- [5. Key Learnings](#5-key-learnings)
- [6. Resources](#6-resources)

---

## Overview
Day 56 builds directly on Day 55's Ansible master/node setup by introducing **ad-hoc commands** — one-off Ansible commands run straight from the CLI for quick tasks, without writing a full playbook. The day also includes a reflective task: writing up a blog explanation after watching a video walkthrough of Ansible.

## Topics Covered
- ❖ Introduction to Ad-hoc Commands
- ❖ Tasks — ping 3 servers, check uptime, explore ad-hoc examples with screenshots/blog
- ❖ Ansible video → blog explanation task

---

## 1. Introduction to Ad-hoc Commands
Ansible **ad-hoc commands** are one-liner commands used to perform a specific, quick task across one or more managed hosts — without the overhead of writing a playbook. The relationship between the two:

| | Ad-hoc Command | Playbook |
|---|---|---|
| **Analogy** | A single shell command | A shell script (multiple commands + logic) |
| **Use case** | Quick, one-off tasks — check uptime, restart a service, copy a file | Repeatable, structured, multi-step automation with conditionals/loops/handlers |
| **Reusability** | Not saved/reused by default (unless scripted separately) | Version-controlled, reusable, shareable |

### 1.1 Ad-hoc Command Syntax
```bash
ansible <host-pattern> -m <module> -a "<module-arguments>"
```
- `<host-pattern>` — a host, group (from inventory), or `all`
- `-m` — the module to use (e.g., `ping`, `shell`, `command`, `copy`, `apt`)
- `-a` — arguments passed to that module

---

## 2. Ad-hoc Command Tasks

### Task 1 — Ping 3 Servers from the Inventory File

**Prerequisite:** the inventory file (`/etc/ansible/hosts`, from Day 55) should list at least 3 nodes. Extending the Day 55 setup with a third node:
```ini
[nodes]
node1 ansible_host=<node1-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/my-keypair.pem
node2 ansible_host=<node2-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/my-keypair.pem
node3 ansible_host=<node3-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/my-keypair.pem
```

**Ad-hoc ping command:**
```bash
ansible nodes -m ping
```
Or, to explicitly target all three by name:
```bash
ansible node1,node2,node3 -m ping
```

**Expected output (per node):**
```json
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

> Status: reference implementation — pending execution + screenshot of `SUCCESS`/`pong` from all 3 nodes.

---

### Task 2 — Check Uptime via Ad-hoc Command

**Command:**
```bash
ansible nodes -m command -a "uptime"
```
Or using the `shell` module (allows shell features like pipes, though `command` is safer/preferred when not needed):
```bash
ansible nodes -a "uptime"
```
> Note: if no `-m` module is specified, Ansible defaults to the `command` module — so `ansible nodes -a "uptime"` and `ansible nodes -m command -a "uptime"` are equivalent.

**Expected output (per node):**
```
node1 | CHANGED | rc=0 >>
 14:32:10 up 2 days,  3:15,  1 user,  load average: 0.00, 0.01, 0.05
```

> Status: reference implementation — pending execution + screenshot of uptime output from all nodes.

---

### Task 3 — Explore Additional Ad-hoc Command Examples

**Objective:** Try out a range of common ad-hoc commands, screenshot the results, and write a short blog explaining each.

**A curated set of useful ad-hoc examples to try and document:**

| Purpose | Command |
|---|---|
| Check disk space | `ansible nodes -m shell -a "df -h"` |
| Check free memory | `ansible nodes -m shell -a "free -m"` |
| List running processes | `ansible nodes -a "ps aux"` |
| Copy a file to all nodes | `ansible nodes -m copy -a "src=/home/ubuntu/notes.txt dest=/tmp/notes.txt"` |
| Install a package (apt) | `ansible nodes -m apt -a "name=nginx state=present" --become` |
| Restart a service | `ansible nodes -m service -a "name=nginx state=restarted" --become` |
| Create a user | `ansible nodes -m user -a "name=devopsuser state=present" --become` |
| Reboot all nodes | `ansible nodes -m reboot --become` |
| Gather host facts | `ansible nodes -m setup` |
| Check a specific fact (e.g., OS) | `ansible nodes -m setup -a "filter=ansible_distribution*"` |

**Suggested blog structure for this task:**
1. Brief intro — what ad-hoc commands are and when to reach for them over a playbook
2. For each command tried: the command itself, a screenshot of the output, and 1–2 sentences on what it does / why it's useful
3. A short closing reflection — which command felt most immediately useful for real day-to-day server management

> Status: pending — commands to be run against the Day 55 nodes, screenshots collected, and blog post drafted/published.

---

## 3. Ansible Video Blog Task

**Objective:** Watch the referenced Ansible video and write a blog post explaining what it covered, in your own words.

**Suggested structure for this blog post:**
1. Video title/source and a one-line summary of its scope
2. Key concepts explained in the video — e.g., control node vs. managed node, agentless architecture, inventory, ad-hoc commands vs. playbooks (tying back to Days 55–56's hands-on work)
3. Any new insight or "aha moment" that wasn't obvious from just doing the hands-on tasks
4. How the video's explanation compares with/reinforces what was practiced hands-on in the last two days
5. Closing takeaway — what to explore next (e.g., writing actual playbooks, roles, handlers)

> Status: pending — to be drafted after watching the referenced video; link to the published post can be added here once live.

---

## 4. Troubleshooting Notes
| Issue | Likely Cause | Fix |
|---|---|---|
| `ansible nodes -a "uptime"` returns `UNREACHABLE` | Node unreachable via SSH — stopped instance, wrong IP, or SG blocking | Confirm instance is running, IP in inventory is current (EC2 public/private IPs can change on stop/start), and SG allows SSH from master |
| `apt`/`service`/`user` module commands fail with permission errors | Missing privilege escalation | Add `--become` (and `--ask-become-pass` if a sudo password is required) to run as root |
| `copy` module fails, "Destination directory does not exist" | Target path on the remote host doesn't exist | Create the directory first, or adjust `dest` to an existing path |
| Ad-hoc command runs on only 1 host instead of all 3 | Host pattern typo, or inventory group doesn't include all 3 | Double check group name and that all 3 hosts are listed under it in `/etc/ansible/hosts` |
| `ansible nodes -m setup` output is overwhelming | `setup` module returns a large fact dictionary by default | Use the `filter=` argument to narrow to specific facts, as shown in the table above |

---

## 5. Key Learnings
- Ad-hoc commands are the right tool for "I need to do this one thing, right now, on a few machines" — playbooks are the right tool once that task needs to be repeatable, shared, or combined with logic.
- The `command`/`shell` module distinction matters: `command` is safer (no shell interpretation, so no risk of injection via pipes/redirects), while `shell` is needed when actual shell features (pipes, wildcards) are required.
- `--become` is Ansible's equivalent of `sudo` — most system-changing modules (`apt`, `service`, `user`) need it unless already running as an elevated user.
- The `setup` module's fact-gathering is genuinely useful beyond ad-hoc exploration — it's the same fact data playbooks can reference for conditional logic later.
- Documenting hands-on command output (screenshots + explanation) as a blog forces a clearer understanding than just running commands and moving on — worth keeping as a habit going into playbooks next.

## 6. Resources
- [Ansible Ad-hoc Commands — Official Guide](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)
- [Ansible Modules Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)
- [Ansible `setup` Module Reference](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html)

---
