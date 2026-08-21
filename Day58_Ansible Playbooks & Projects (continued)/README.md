# Day 58 — Ansible Playbooks & Projects (continued)

> **Note:** The Day 58 curriculum text is identical to Day 57 — same three playbook exercises, the same best-practices blog task, and the same "deploy a simple web app" project brief. Rather than duplicate Day 57's content verbatim, this entry treats Day 58 as reinforcement/practice: the same exercise types are solved with different scenarios, and the write-up focuses on what changed or was reinforced from a second pass.

## 📌 Overview

Day 58 revisits Ansible Playbooks & Projects from Day 57. The goal on a repeat day like this is not to re-type the same playbooks, but to practice the same patterns against slightly different requirements — reinforcing muscle memory for playbook structure, modules, and running against inventory groups.

---

## Part 1: Ansible Playbooks (practice round)

### Tasks / Exercises

#### 1. Playbook: Create a file on a different server (variant — with content and permissions)

**`create_file_v2.yml`**
```yaml
---
- name: Create a file with specific content and permissions
  hosts: node1
  become: yes
  tasks:
    - name: Create a notes file with content
      copy:
        content: "Day 58 practice — file created via Ansible playbook.\n"
        dest: /home/ubuntu/day58_notes.txt
        owner: ubuntu
        group: ubuntu
        mode: '0640'
```

```bash
ansible-playbook -i /etc/ansible/hosts create_file_v2.yml
```

#### 2. Playbook: Create a new user (variant — with sudo access)

**`create_user_v2.yml`**
```yaml
---
- name: Create a new sudo-enabled user
  hosts: node1
  become: yes
  tasks:
    - name: Add the user 'deployer'
      user:
        name: deployer
        state: present
        shell: /bin/bash
        create_home: yes
        groups: sudo
        append: yes

    - name: Allow deployer passwordless sudo
      copy:
        content: "deployer ALL=(ALL) NOPASSWD:ALL\n"
        dest: /etc/sudoers.d/deployer
        mode: '0440'
        validate: 'visudo -cf %s'
```

```bash
ansible-playbook -i /etc/ansible/hosts create_user_v2.yml
```

#### 3. Playbook: Install Docker on a group of servers (variant — with Docker Compose)

**`install_docker_v2.yml`**
```yaml
---
- name: Install Docker and Docker Compose on all web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Update apt package index
      apt:
        update_cache: yes

    - name: Install Docker
      apt:
        name: docker.io
        state: present

    - name: Install Docker Compose plugin
      apt:
        name: docker-compose-plugin
        state: present

    - name: Ensure Docker service is running and enabled
      service:
        name: docker
        state: started
        enabled: yes

    - name: Confirm Docker Compose is available
      command: docker compose version
      register: compose_version
      changed_when: false

    - name: Show Compose version
      debug:
        var: compose_version.stdout
```

```bash
ansible-playbook -i /etc/ansible/hosts install_docker_v2.yml
```

📺 Reference: same *Ansible Playbooks* explainer video from the curriculum, re-watched to reinforce concepts (roles, variables, handlers).

#### 4. Blog Task: Ansible Playbook Best Practices (round 2 notes)

Added to the Day 57 blog draft with a few more practices that came up during the repeat exercises:
- Use `register` + `changed_when: false` for informational commands so they don't falsely report "changed" state
- Validate config files before deploying them (e.g. `validate: 'visudo -cf %s'` for sudoers)
- Separate "create" and "configure" tasks so a playbook re-run doesn't silently re-apply destructive changes
- Tag tasks (`tags:`) so specific parts of a large playbook can be run in isolation with `--tags`
- Store secrets (passwords, keys) with **Ansible Vault** rather than in plaintext playbooks or inventory

---

## Part 2: Mini Project — Web App Deployment (reinforcement)

### Introduction

Same premise as Day 57: deploy a simple web app using Ansible. On this pass, the deployment was re-run against the same 3-EC2 environment to confirm idempotency — running the playbooks a second time should report **no changes** where nothing changed, which is the real test of a well-written playbook.

### Tasks / Exercises

- Re-ran `install_nginx.yml` and `deploy_webpage.yml` from Day 57 against the same `webservers` group
- Confirmed idempotency: second run showed `changed=0` for tasks that had already applied cleanly, and only the intentionally-changed webpage content task reported a change
- Practiced targeting a single node instead of the whole group using `--limit`:
  ```bash
  ansible-playbook -i /etc/ansible/hosts deploy_webpage.yml --limit node1
  ```
- Practiced a dry run before applying changes:
  ```bash
  ansible-playbook -i /etc/ansible/hosts deploy_webpage.yml --check
  ```

📖 Reference: same blog by Sandeep Singh on Ansible web app deployment, revisited for the idempotency and `--check`/`--limit` flag details.

---

## 🧠 Key Learnings

- Repeating an exercise with varied requirements (permissions, sudo access, Compose) is a good way to test whether the underlying pattern was actually learned, not just copied
- Idempotency is the real measure of a good playbook — a second run should change nothing it doesn't need to
- `--limit` and `--check` are essential day-to-day flags for safely testing playbooks against subsets of infrastructure
- `register` + `changed_when` and Ansible Vault are the next layer of playbook maturity beyond the Day 57 basics

## ⚠️ Challenges Faced

- Noticed the curriculum content for Day 58 duplicates Day 57 verbatim — treated it as a practice/reinforcement day rather than re-submitting identical work
- Getting `validate: 'visudo -cf %s'` right for the sudoers file took a couple of attempts to get the syntax correct

## 🔗 Resources

- Ansible Playbooks documentation
- Ansible Playbooks explainer video (curriculum link)
- Blog: Deploying a web app with Ansible — by Sandeep Singh (curriculum link)
- Ansible Vault documentation (for secrets management, explored as a follow-on)

---
