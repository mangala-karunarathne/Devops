# Ansible Study Material — Installing a Package on App Servers

---

## Original Task

> The Nautilus Application development team wanted to test some applications on app servers in **Stratos Datacenter**. They shared some pre-requisites with the DevOps team, and packages need to be installed on app servers. Since we are already using Ansible for automating such tasks, please perform this task using Ansible as per details mentioned below:
>
> 1. Create an inventory file `/home/thor/playbook/inventory` on **jump host** and add all app servers in it.
> 2. Create an Ansible playbook `/home/thor/playbook/playbook.yml` to install `sqlite` package on **all app servers** using Ansible `yum` module.
> 3. Make sure user `thor` should be able to run the playbook on **jump host**.
>
> **Note:** Validation will try to run playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way, without passing any extra arguments.

---

## What is Ansible?

Ansible is an open-source **IT automation tool** that lets you:
- Configure systems
- Deploy software
- Run tasks on multiple servers **simultaneously**

Instead of SSHing into each server one by one, Ansible connects to all of them at once and runs the same task everywhere — from a single command on your **control node** (jump host in this case).

### Key concepts

| Term | What it means |
|---|---|
| Control node | The machine where you run Ansible (jump host here) |
| Managed nodes | The servers Ansible connects to (stapp01, stapp02, stapp03) |
| Inventory | A file listing all managed nodes and how to connect to them |
| Playbook | A YAML file describing what tasks to run on which servers |
| Module | A built-in Ansible function (e.g. `yum`, `apt`, `copy`, `service`) |
| Task | A single action in a playbook (e.g. install a package) |
| Play | A group of tasks targeting a set of hosts |

---

## Stratos Datacenter — App Server Reference

| Server | Hostname | IP Address | SSH User | Password |
|---|---|---|---|---|
| App Server 1 | stapp01 | 172.16.238.10 | tony | Ir0nM@n |
| App Server 2 | stapp02 | 172.16.238.11 | steve | Am3ric@ |
| App Server 3 | stapp03 | 172.16.238.12 | banner | BigGr33n |

---

## Step-by-Step Solution

### Step 1 — Create the working directory

```bash
mkdir -p /home/thor/playbook
cd /home/thor/playbook
```

No output expected. This just creates the folder where all our Ansible files will live.

---

### Step 2 — Create the inventory file

**Path:** `/home/thor/playbook/inventory`

```ini
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_pass=Ir0nM@n
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_ssh_pass=Am3ric@
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_ssh_pass=BigGr33n
```

**What each field means:**

| Field | Purpose |
|---|---|
| `stapp01` | Alias/name Ansible uses to refer to this host |
| `ansible_host` | The actual IP address to connect to |
| `ansible_user` | The SSH username |
| `ansible_ssh_pass` | The SSH password |

> **Tip:** In production environments, you'd use SSH keys instead of passwords for better security.

---

### Step 3 — Create the playbook

**Path:** `/home/thor/playbook/playbook.yml`

```yaml
---
- name: Install sqlite on all app servers
  hosts: all
  become: yes
  tasks:
    - name: Install sqlite package
      yum:
        name: sqlite
        state: present
```

**Breaking it down line by line:**

| Line | Meaning |
|---|---|
| `---` | YAML document start marker |
| `name:` | Human-readable description of the play |
| `hosts: all` | Run this play on ALL hosts in the inventory |
| `become: yes` | Use sudo (privilege escalation) to run as root |
| `tasks:` | List of tasks to execute |
| `yum:` | Use the Ansible yum module (for RedHat/CentOS systems) |
| `name: sqlite` | The package to install |
| `state: present` | Ensure the package is installed (idempotent) |

> **What is idempotent?**
> Running the playbook multiple times gives the same result. If sqlite is already installed, Ansible skips it (`ok`) instead of installing again (`changed`). This is safe to run repeatedly.

---

### Step 4 — Create ansible.cfg

**Path:** `/home/thor/playbook/ansible.cfg`

```ini
[defaults]
host_key_checking = False
```

This disables SSH host key verification, so Ansible doesn't pause and ask *"Do you trust this host?"* the first time it connects. Without this, automation gets blocked waiting for manual input.

---

### Step 5 — Verify file ownership

```bash
ls -la /home/thor/playbook/
```

Expected output:
```
-rw-r--r-- 1 thor thor ... ansible.cfg
-rw-r--r-- 1 thor thor ... inventory
-rw-r--r-- 1 thor thor ... playbook.yml
```

Files must be owned by `thor` since that's the user running the playbook. If they show `root` as owner, fix with:

```bash
sudo chown thor:thor /home/thor/playbook/*
```

> If you created the files while logged in as `thor` (normal case), they are already owned by `thor` and this step can be skipped.

---

### Step 6 — Run the playbook

```bash
cd /home/thor/playbook
ansible-playbook -i inventory playbook.yml
```

**Expected output:**

```
PLAY [Install sqlite on all app servers] *****************************

TASK [Gathering Facts] ***********************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Install sqlite package] ****************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ***********************************************************
stapp01 : ok=2  changed=1  unreachable=0  failed=0
stapp02 : ok=2  changed=1  unreachable=0  failed=0
stapp03 : ok=2  changed=1  unreachable=0  failed=0
```

---

## Understanding the Output

### Task statuses

| Status | Meaning |
|---|---|
| `ok` | Task ran, no change needed (already in desired state) |
| `changed` | Task ran and made a change (e.g. installed a package) |
| `failed` | Task failed — check error message |
| `unreachable` | Could not connect to the host |
| `skipped` | Task was conditionally skipped |

### PLAY RECAP fields

| Field | Meaning |
|---|---|
| `ok` | Number of tasks that succeeded (including no-change tasks) |
| `changed` | Number of tasks that made actual changes |
| `unreachable` | Hosts Ansible could not connect to |
| `failed` | Tasks that failed |

> A successful run has `failed=0` and `unreachable=0` on all hosts.

---

## Why Ansible Instead of Manual SSH?

### Manual approach (without Ansible)

```bash
ssh tony@172.16.238.10
sudo yum install sqlite -y
exit

ssh steve@172.16.238.11
sudo yum install sqlite -y
exit

ssh banner@172.16.238.12
sudo yum install sqlite -y
exit
```

3 logins, 3 commands, done one at a time.

### Ansible approach

```bash
ansible-playbook -i inventory playbook.yml
```

One command. All 3 servers. Done in parallel.

### At scale

| Servers | Manual SSH | Ansible |
|---|---|---|
| 3 | Annoying but doable | 1 command |
| 30 | Very painful | Still 1 command |
| 300 | Practically impossible | Still 1 command |

---

## File Structure Summary

```
/home/thor/playbook/
├── ansible.cfg      # Ansible configuration (disables host key checking)
├── inventory        # List of servers with connection details
└── playbook.yml     # Tasks to run on those servers
```

---

## Common Mistakes to Avoid

| Mistake | What goes wrong | Fix |
|---|---|---|
| Wrong indentation in YAML | Playbook fails to parse | YAML uses 2-space indentation strictly |
| Missing `become: yes` | Package install fails (permission denied) | Add `become: yes` to the play |
| Running from wrong directory | Inventory/playbook not found | Always `cd` into the playbook directory first |
| `host_key_checking` not disabled | Ansible hangs waiting for input | Add `ansible.cfg` with `host_key_checking = False` |
| Wrong `state` value | Unexpected behaviour | Use `present` to install, `absent` to remove, `latest` to upgrade |

---

## Quick Reference — yum Module States

```yaml
# Install a package
yum:
  name: sqlite
  state: present

# Remove a package
yum:
  name: sqlite
  state: absent

# Install latest version
yum:
  name: sqlite
  state: latest

# Install multiple packages
yum:
  name:
    - sqlite
    - git
    - wget
  state: present
```

---

## Key Takeaways

1. **Inventory** tells Ansible *where* to connect
2. **Playbook** tells Ansible *what* to do
3. **ansible.cfg** controls *how* Ansible behaves
4. `become: yes` = run as root via sudo
5. `hosts: all` = run on every host in inventory
6. `state: present` = idempotent install (safe to re-run)
7. Ansible runs tasks on all hosts **in parallel**, not one by one
