# Ansible – Complete Notes (Theory + Practical)
**Source:** Ajay Nikam | DevOps Journey!

---

## PART 1 — THEORY

---

### 1. What is Ansible?

Ansible is an **open-source, agentless IT automation tool** used for:
- Configuration Management
- Application Deployment
- Infrastructure Provisioning
- Task Automation (across 1 to 1000+ servers)

It was developed by Red Hat and uses **YAML** (human-readable language) to define automation tasks called **Playbooks**.

> **Simple Definition:** Ansible lets you control and configure many remote servers from one central machine — without installing any software on those servers.

---

### 2. Why Ansible? (Key Features)

| Feature | Description |
|---------|-------------|
| **Agentless** | No software/agent needed on target servers — uses SSH |
| **Simple Syntax** | Playbooks written in YAML — easy to read and write |
| **Idempotent** | Running a playbook multiple times gives the same result — safe to re-run |
| **Push-Based** | Control node *pushes* configs to managed nodes (unlike Puppet/Chef which pull) |
| **Scalable** | Manage 1 to thousands of servers from one place |
| **Open Source** | Free to use, large community |

---

### 3. Ansible Architecture

```
┌─────────────────────────────────────────┐
│           CONTROL NODE                  │
│  (Ansible installed here)               │
│  - Inventory File                       │
│  - Playbooks / Ad-Hoc Commands          │
│  - Ansible Engine                       │
└──────────┬──────────────────────────────┘
           │  SSH (Passwordless)
     ┌─────▼──────┐   ┌────────────┐   ┌────────────┐
     │  Managed   │   │  Managed   │   │  Managed   │
     │  Node 1    │   │  Node 2    │   │  Node 3    │
     │ (No Agent) │   │ (No Agent) │   │ (No Agent) │
     └────────────┘   └────────────┘   └────────────┘
```

---

### 4. Core Components of Ansible

#### 4.1 Control Node
- The machine where Ansible is **installed**
- From here, you run all commands and playbooks
- Can be your laptop, a VM, or a dedicated server
- **Note:** Ansible does NOT run on Windows as a control node (use Linux/Mac)

#### 4.2 Managed Nodes (Hosts)
- Target servers that Ansible manages/configures
- **No Ansible installation required** on these
- Ansible connects to them via **SSH** (Linux) or **WinRM** (Windows)

#### 4.3 Inventory
- A file listing all managed nodes (IPs or hostnames)
- Supports **grouping** (e.g., webservers, dbservers)
- Default path: `/etc/ansible/hosts`

#### 4.4 Modules
- **Pre-built mini-programs** that Ansible pushes to managed nodes to perform tasks
- Examples: `apt`, `yum`, `copy`, `service`, `shell`, `file`
- Ansible has **700+ built-in modules**
- Modules are **idempotent** — they check current state before making changes

#### 4.5 Playbooks
- YAML files that define **what tasks to run** on which servers
- The core of Ansible automation
- A Playbook = one or more **Plays**; a Play = one or more **Tasks**

#### 4.6 Roles
- A way to **organize and reuse** Playbook content
- Has a predefined folder structure: `tasks/`, `handlers/`, `vars/`, `templates/`, etc.
- Used in large projects for clean, maintainable code

#### 4.7 Plugins
- Extend Ansible's core functionality
- Types: Connection plugins, Callback plugins, Lookup plugins
- Run on the **Control Node** (unlike modules which run on managed nodes)

---

### 5. How Ansible Works — Step by Step

1. You write an **Inventory** file with target server IPs
2. You write a **Playbook** (YAML) defining the desired state
3. You run: `ansible-playbook -i inventory playbook.yaml`
4. Ansible **SSHes into each target server**
5. Ansible **pushes modules** (small scripts) to the target
6. Modules **execute locally** on the target and return results
7. Results are displayed back on the Control Node
8. Temporary files are **cleaned up** after execution

---

### 6. Push-Based vs Pull-Based (Interview Concept)

| | Ansible (Push) | Puppet / Chef (Pull) |
|--|--|--|
| How it works | Control node pushes configs to targets | Target agents pull configs from server |
| Agent needed? | ❌ No | ✅ Yes |
| Setup complexity | Simple | Complex |
| Speed | Immediate | Periodic (scheduled) |

> **Memory Trick:** Push = server *pushes* config to you. Pull = you *pull* config from server.

---

### 7. Ansible vs Other Tools

| Tool | Type | Language | Agent |
|------|------|----------|-------|
| Ansible | Push | YAML | No |
| Terraform | Declarative IaC | HCL | No |
| Puppet | Pull | DSL/Ruby | Yes |
| Chef | Pull | Ruby | Yes |

---

### 8. Use Cases of Ansible in DevOps

- **Configuration Management** – Install packages, manage config files across all servers
- **Application Deployment** – Deploy code from Git to multiple servers
- **Infrastructure Provisioning** – Spin up AWS EC2 instances, VPCs
- **CI/CD Integration** – Trigger Ansible playbooks from Jenkins pipelines
- **Patch Management** – Apply OS updates to hundreds of servers at once

---
---

## PART 2 — PRACTICAL

---

### 1. Installation

```bash
sudo apt update
sudo apt install ansible -y
ansible --version      # verify installation
```

Other methods:
```bash
pip install ansible              # via Python pip
brew install ansible             # Mac only
```

---

### 2. Passwordless SSH Authentication Setup

> Mandatory before running any Ansible command — Ansible uses SSH to connect.

**Step 1 — On Control Node (generate SSH keys):**
```bash
ssh-keygen
# Press Enter 3 times (accept defaults)
# Keys saved at: ~/.ssh/id_rsa (private) and ~/.ssh/id_rsa.pub (public)
```

**Step 2 — Copy public key to Target Server:**
```bash
cat ~/.ssh/id_rsa.pub
# Copy the output
```

**Step 3 — On Target Server (paste public key):**
```bash
nano ~/.ssh/authorized_keys
# Paste the public key here and save
```

**Step 4 — Test connection:**
```bash
ssh <target-server-IP>
# Should connect WITHOUT asking for a password ✅
```

---

### 3. Inventory File

Create a file named `inventory` (no extension needed):

```ini
# Simple list
192.168.1.10
192.168.1.11

# Grouped servers
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

**Verify connectivity:**
```bash
ansible -i inventory all -m ping
```

---

### 4. Ad-Hoc Commands

> For quick one-off tasks — no playbook needed.

**Syntax:**
```bash
ansible -i <inventory> <target> -m <module> -a "<arguments>"
```

**Examples:**
```bash
# Create a file on all servers
ansible -i inventory all -m shell -a "touch devopsclass"

# Check disk usage
ansible -i inventory all -m shell -a "df -h"

# Install nginx (Ubuntu)
ansible -i inventory all -m shell -a "sudo apt install nginx -y"

# Target only webservers group
ansible -i inventory webservers -m shell -a "uptime"
```

**Common Flags:**
| Flag | Meaning |
|------|---------|
| `-i` | Path to inventory file |
| `-m` | Module name |
| `-a` | Arguments to pass to module |
| `all` | Target all servers in inventory |
| `--become` | Run as sudo/root |

---

### 5. Writing an Ansible Playbook

**File: `first-playbook.yml`**

```yaml
---
- name: Install and Start Nginx          # Play name
  hosts: all                             # Target all servers
  become: true                           # Run as sudo/root

  tasks:

    - name: Install nginx                # Task 1
      apt:
        name: nginx
        state: present                   # present = install, absent = remove

    - name: Start nginx service          # Task 2
      service:
        name: nginx
        state: started                   # started = start service
```

**Playbook Keywords Explained:**
| Keyword | Purpose |
|---------|---------|
| `---` | Marks start of YAML file |
| `name` | Human-readable label for play/task |
| `hosts` | Which servers to target (all / group name) |
| `become: true` | Run tasks with sudo privileges |
| `tasks` | List of actions to perform |
| `state: present` | Desired state — install if not present |
| `state: started` | Desired state — start if not running |

---

### 6. Running the Playbook

```bash
ansible-playbook -i inventory first-playbook.yaml
```

**What happens when you run it:**
1. **Gather Facts** – Ansible collects info about target servers (OS, IP, memory, etc.)
2. **Execute Tasks** – Runs each task in order
3. **Report** – Shows OK / Changed / Failed for each task

**With Debugging:**
```bash
ansible-playbook -i inventory first-playbook.yaml -v     # basic verbose
ansible-playbook -i inventory first-playbook.yaml -vv    # more detail
ansible-playbook -i inventory first-playbook.yaml -vvv   # full debug
```

**Output Colors:**
| Color | Meaning |
|-------|---------|
| Green (ok) | Task ran, no changes needed |
| Yellow (changed) | Task ran and made a change |
| Red (failed) | Task failed |

---

### 7. Key Ansible Modules Reference

| Module | Use Case | Example |
|--------|----------|---------|
| `shell` | Run shell commands | `shell: df -h` |
| `command` | Run commands (safer than shell) | `command: uptime` |
| `apt` | Install packages (Ubuntu/Debian) | `apt: name=nginx state=present` |
| `yum` | Install packages (RHEL/CentOS) | `yum: name=httpd state=present` |
| `service` | Manage services | `service: name=nginx state=started` |
| `copy` | Copy files to remote | `copy: src=file.txt dest=/tmp/` |
| `file` | Create/delete files, dirs | `file: path=/tmp/test state=touch` |
| `user` | Manage user accounts | `user: name=devops state=present` |

---

### 8. Complete Real-World Playbook Example

**Install Nginx + Deploy a Custom HTML Page:**

```yaml
---
- name: Deploy Web Server
  hosts: webservers
  become: true

  tasks:

    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy custom index.html
      copy:
        src: ./index.html
        dest: /var/www/html/index.html

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes            # starts nginx on server reboot too
```

---

## PART 3 — INTERVIEW Q&A

| Question | Answer |
|----------|--------|
| What is Ansible? | Open-source agentless IT automation tool using SSH and YAML playbooks |
| Why is Ansible agentless? | It uses SSH to connect — no software needs to be installed on managed nodes |
| What is idempotency? | Running a playbook multiple times produces the same result — no duplicate changes |
| Push vs Pull model? | Ansible is push-based; Puppet/Chef are pull-based |
| Ad-hoc vs Playbook? | Ad-hoc: 1–2 quick tasks; Playbook: multiple structured, reusable tasks |
| What is an inventory file? | A file listing target server IPs/hostnames, optionally grouped |
| What is `become: true`? | Enables privilege escalation — runs tasks as root/sudo |
| What is Gather Facts? | First step in any playbook — Ansible collects system info from target nodes |
| What is a module? | A pre-built script that performs a specific task (install, copy, start service, etc.) |
| How to target specific servers? | Define groups in inventory with `[groupname]` and reference in playbook `hosts:` |
| Difference: `shell` vs `apt` module? | `shell` runs raw commands; `apt` is idempotent and cross-platform safe |
| What is a Role in Ansible? | A structured way to organize and reuse playbook content across projects |

---

## Quick Revision Checklist

- [ ] Define Ansible and explain its key features (agentless, idempotent, push-based)
- [ ] Draw/explain Ansible architecture (Control Node → SSH → Managed Nodes)
- [ ] Know the difference: Push (Ansible) vs Pull (Puppet/Chef)
- [ ] Install Ansible and verify version
- [ ] Set up passwordless SSH between control and target
- [ ] Create an inventory file with grouped servers
- [ ] Run an ad-hoc command using `-m shell`
- [ ] Write a Playbook with `apt` and `service` modules
- [ ] Run a Playbook with `-vv` verbosity
- [ ] Explain Gather Facts, idempotency, and `become: true`

---

## Useful Resources

- Ansible Official Examples: https://github.com/ansible/ansible-examples
- Ansible Modules Docs: https://docs.ansible.com/ansible/latest/collections/index_module.html
- Ansible Interview Q&A (Abhishek): https://www.youtube.com/watch?v=j5PgN0J3d7M
