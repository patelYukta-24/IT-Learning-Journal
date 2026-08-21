# Day 57 — Ansible Playbooks & Projects

## 📌 Overview

Day 57 moves from ad-hoc Ansible commands (Day 56) into **Ansible Playbooks** — YAML files that define multiple tasks, roles, and configurations to run against one or more managed servers in a repeatable, version-controlled way. The day is split into two parts: writing and testing individual playbooks against local/managed servers, and a hands-on mini-project deploying a sample web app across real EC2 instances using Ansible.

---

## Part 1: Ansible Playbooks

### Introduction

Ansible playbooks run multiple tasks, assign roles, and define configurations, deployment steps, and variables. When managing multiple servers, playbooks organize the steps across the assembled machines and get them into the state the user needs — think of a playbook as an instruction manual for your infrastructure.

### Tasks / Exercises

#### 1. Playbook: Create a file on a different (managed) server

**`create_file.yml`**
```yaml
---
- name: Create a file on the managed node
  hosts: node1
  become: yes
  tasks:
    - name: Create an empty file
      file:
        path: /home/ubuntu/day57_test_file.txt
        state: touch
        mode: '0644'
```

Run it:
```bash
ansible-playbook -i /etc/ansible/hosts create_file.yml
```

Verify on the managed node:
```bash
ls -l /home/ubuntu/day57_test_file.txt
```

#### 2. Playbook: Create a new user

**`create_user.yml`**
```yaml
---
- name: Create a new user on managed nodes
  hosts: node1
  become: yes
  tasks:
    - name: Add the user 'devopsuser'
      user:
        name: devopsuser
        state: present
        shell: /bin/bash
        create_home: yes
```

Run it:
```bash
ansible-playbook -i /etc/ansible/hosts create_user.yml
```

Verify:
```bash
id devopsuser
```

#### 3. Playbook: Install Docker on a group of servers

**`install_docker.yml`**
```yaml
---
- name: Install Docker on all web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Update apt package index
      apt:
        update_cache: yes

    - name: Install prerequisite packages
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: present

    - name: Install Docker
      apt:
        name: docker.io
        state: present

    - name: Ensure Docker service is running and enabled
      service:
        name: docker
        state: started
        enabled: yes

    - name: Add ubuntu user to the docker group
      user:
        name: ubuntu
        groups: docker
        append: yes
```

Run it:
```bash
ansible-playbook -i /etc/ansible/hosts install_docker.yml
```

Verify:
```bash
ansible webservers -i /etc/ansible/hosts -a "docker --version"
```

📺 Reference: *Ansible Playbooks* explainer video (linked in the curriculum) for the underlying concepts.

#### 4. Blog Task: Writing Ansible Playbooks with Best Practices

Wrote a blog post (`blog_ansible_playbooks_best_practices.md`) covering:
- Keep playbooks idempotent — always re-runnable without side effects
- Use `become: yes` only where privilege escalation is actually needed
- Prefer modules (`apt`, `user`, `file`, `service`) over raw `shell`/`command` calls
- Organize inventory into groups (`webservers`, `dbservers`) rather than targeting hosts individually
- Use variables and `group_vars` / `host_vars` instead of hardcoding values
- Name every task clearly — task names show up in playbook output and make debugging much faster
- Use handlers for actions that should only run on change (e.g., restart a service after a config file changes)
- Run `ansible-playbook --check` (dry run) before applying against production
- Keep playbooks small and composable; break large playbooks into roles as complexity grows

---

## Part 2: Mini Project — Deploy a Simple Web App with Ansible

### Introduction

Ansible playbooks are amazing, as learned in Part 1. The natural next step: deploy a simple web app using Ansible across real infrastructure.

### Tasks / Exercises

#### 1. Create 3 EC2 instances with the same key pair

- Launched 3 EC2 instances (Ubuntu) from the AWS Console
- All three created using the **same key pair** so a single private key can reach all managed nodes
- One instance designated as the **Ansible control/host server**; the other two as **managed nodes**

#### 2. Install Ansible on the host server

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
ansible --version
```

#### 3. Copy the private key from local machine to the host server

From the local machine:
```bash
scp -i mykey.pem mykey.pem ubuntu@<ansible_host_public_ip>:/home/ubuntu/.ssh/
```

On the Ansible host server, secure the key:
```bash
chmod 400 /home/ubuntu/.ssh/mykey.pem
```

#### 4. Configure the inventory file

```bash
sudo vim /etc/ansible/hosts
```

```ini
[webservers]
node1 ansible_host=<managed_node_1_private_ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/mykey.pem
node2 ansible_host=<managed_node_2_private_ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/mykey.pem
```

Test connectivity:
```bash
ansible webservers -i /etc/ansible/hosts -m ping
```

#### 5. Create a playbook to install Nginx

**`install_nginx.yml`**
```yaml
---
- name: Install and start Nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

```bash
ansible-playbook -i /etc/ansible/hosts install_nginx.yml
```

#### 6. Deploy a sample webpage using the playbook

**`deploy_webpage.yml`**
```yaml
---
- name: Deploy sample web page
  hosts: webservers
  become: yes
  tasks:
    - name: Copy sample index.html to the web root
      copy:
        content: |
          <!DOCTYPE html>
          <html>
          <head><title>Day 57 - Ansible Deployed App</title></head>
          <body>
            <h1>Deployed with Ansible 🚀</h1>
            <p>This page was pushed to all web servers using a single Ansible playbook.</p>
          </body>
          </html>
        dest: /var/www/html/index.html
        mode: '0644'

    - name: Restart Nginx to serve the new page
      service:
        name: nginx
        state: restarted
```

```bash
ansible-playbook -i /etc/ansible/hosts deploy_webpage.yml
```

Verified by visiting `http://<managed_node_public_ip>/` in a browser — the sample page loaded on both managed nodes from a single playbook run.

📖 Reference: *Blog on Ansible web app deployment* by Sandeep Singh (linked in the curriculum) for additional context.

---

## 🧠 Key Learnings

- Playbooks turn repeatable, error-prone manual steps into version-controlled, idempotent automation
- Grouping hosts in the inventory (`webservers`) lets one playbook target many machines at once
- Modules (`apt`, `user`, `file`, `service`, `copy`) are safer and more idempotent than raw shell commands
- The `become: yes` directive is Ansible's equivalent of `sudo` for privilege escalation on managed nodes
- A shared key pair across EC2 instances simplifies SSH-based orchestration from a single control node
- End-to-end flow — provision infra → install Ansible → configure inventory → write playbooks → deploy — mirrors how real infrastructure teams manage fleets of servers

## ⚠️ Challenges Faced

- Ensuring the private key permissions (`chmod 400`) were correct on the host server before SSH connections from Ansible would succeed
- Getting the inventory groups and `ansible_ssh_private_key_file` path right so `ansible-playbook` could reach both managed nodes without prompting for a password

## 🔗 Resources

- Ansible Playbooks documentation
- Ansible Playbooks explainer video (curriculum link)
- Blog: Deploying a web app with Ansible — by Sandeep Singh (curriculum link)

---
