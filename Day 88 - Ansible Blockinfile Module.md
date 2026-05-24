# Ansible Study Material: Installing & Configuring HTTPD Web Server

---

## 📋 Original Task

> The Nautilus DevOps team wants to install and set up a simple `httpd` web server on all app servers in **Stratos DC**. Additionally, they want to deploy a sample web page using **Ansible only**.

### Task Requirements:
1. Using the playbook, install `httpd` web server on all app servers. Make sure its service is **up and running**.
2. Using the `blockinfile` Ansible module, add the following content in `/var/www/html/index.html`:
   ```
   Welcome to XfusionCorp!
   This is Nautilus sample file, created using Ansible!
   Please do not modify this file manually!
   ```
3. The `/var/www/html/index.html` file's **user and group owner** should be `apache` on all app servers.
4. The `/var/www/html/index.html` file's **permissions** should be `0777` on all app servers.

### Notes:
- Validation runs: `ansible-playbook -i inventory playbook.yml`
- Do **not** use any custom or empty `marker` for the `blockinfile` module.

---

## 🖥️ Environment Details

| Item | Value |
|---|---|
| Jump Host | thor@jump-host |
| Ansible Directory | `/home/thor/ansible` |
| Inventory File | `/home/thor/ansible/inventory` |
| Playbook File | `/home/thor/ansible/playbook.yml` |
| Ansible Config | `/home/thor/ansible/ansible.cfg` |

### Inventory File (`/home/thor/ansible/inventory`)
```
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```

### Ansible Config (`/home/thor/ansible/ansible.cfg`)
```ini
[defaults]
host_key_checking = False
```
> `host_key_checking = False` disables SSH fingerprint verification, allowing Ansible to connect to hosts without manual SSH key approval.

---

## 📁 Final Playbook (`playbook.yml`)

```yaml
---
- name: Install and configure httpd on all app servers
  hosts: all
  become: yes

  tasks:

    - name: Install httpd package
      package:
        name: httpd
        state: present

    - name: Ensure httpd service is started and enabled
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Create index.html with blockinfile
      blockinfile:
        path: /var/www/html/index.html
        create: yes
        block: |
          Welcome to XfusionCorp!
          This is Nautilus sample file, created using Ansible!
          Please do not modify this file manually!

    - name: Set ownership of index.html to apache
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0777'
```

---

## 🔍 Playbook Breakdown — Task by Task

### 1. Play Header
```yaml
- name: Install and configure httpd on all app servers
  hosts: all
  become: yes
```
| Option | Meaning |
|---|---|
| `hosts: all` | Run on all hosts defined in the inventory |
| `become: yes` | Use privilege escalation (`sudo`) to run tasks as root |

---

### 2. Task — Install httpd
```yaml
- name: Install httpd package
  package:
    name: httpd
    state: present
```
- `package` module is OS-agnostic (works on RHEL/CentOS/Debian).
- `state: present` → install the package if it's not already installed.

---

### 3. Task — Start & Enable httpd Service
```yaml
- name: Ensure httpd service is started and enabled
  service:
    name: httpd
    state: started
    enabled: yes
```
| Option | Meaning |
|---|---|
| `state: started` | Start the service if it's not running |
| `enabled: yes` | Enable the service to auto-start on system reboot |

---

### 4. Task — Create Web Page using `blockinfile`
```yaml
- name: Create index.html with blockinfile
  blockinfile:
    path: /var/www/html/index.html
    create: yes
    block: |
      Welcome to XfusionCorp!
      This is Nautilus sample file, created using Ansible!
      Please do not modify this file manually!
```
| Option | Meaning |
|---|---|
| `path` | File to insert the block into |
| `create: yes` | Create the file if it doesn't exist |
| `block` | The multi-line content to insert |
| No `marker` specified | Uses default markers (see below) |

**Default Markers added by `blockinfile`:**
```
# BEGIN ANSIBLE MANAGED BLOCK
Welcome to XfusionCorp!
This is Nautilus sample file, created using Ansible!
Please do not modify this file manually!
# END ANSIBLE MANAGED BLOCK
```
> ⚠️ The task says **do not use custom or empty markers** — so we omit the `marker` option entirely and let Ansible use its defaults.

---

### 5. Task — Set File Ownership & Permissions
```yaml
- name: Set ownership of index.html to apache
  file:
    path: /var/www/html/index.html
    owner: apache
    group: apache
    mode: '0777'
```
| Option | Meaning |
|---|---|
| `owner: apache` | Set file owner to the `apache` user |
| `group: apache` | Set file group to the `apache` group |
| `mode: '0777'` | Set permissions to rwxrwxrwx (everyone can read/write/execute) |

---

## 🔐 Understanding File Permissions (`0777`)

Linux file permissions use **octal (numeric) format**:

| Symbol | Value | Meaning |
|---|---|---|
| r | 4 | Read |
| w | 2 | Write |
| x | 1 | Execute |

`0777` = `rwxrwxrwx`

| Who | Permission |
|---|---|
| Owner | rwx (7) |
| Group | rwx (7) |
| Others | rwx (7) |

**Why `0777` here?**
- The `apache` user (which runs the `httpd` web server) needs to **read** the `index.html` file to serve it to visitors.
- `0777` ensures no permission-related errors (like `403 Forbidden`) when the web server tries to access the file.

---

## 🚀 Commands Used

### Syntax Check
```bash
cd /home/thor/ansible && ansible-playbook -i inventory playbook.yml --syntax-check
```
**Expected output:**
```
playbook: playbook.yml
```

### Run the Playbook
```bash
cd /home/thor/ansible && ansible-playbook -i inventory playbook.yml
```

### Verify on App Server
```bash
ssh tony@stapp01 "cat /var/www/html/index.html && ls -la /var/www/html/index.html && systemctl status httpd | grep Active"
```

---

## ✅ Final Verification Output

```
# BEGIN ANSIBLE MANAGED BLOCK
Welcome to XfusionCorp!
This is Nautilus sample file, created using Ansible!
Please do not modify this file manually!
# END ANSIBLE MANAGED BLOCK
-rwxrwxrwx 1 apache apache 176 May 24 16:27 /var/www/html/index.html
Active: active (running) since Sun 2026-05-24 16:27:50 UTC; 1min 7s ago
```

### PLAY RECAP
```
stapp01  : ok=5  changed=4  unreachable=0  failed=0  skipped=0
stapp02  : ok=5  changed=4  unreachable=0  failed=0  skipped=0
stapp03  : ok=5  changed=4  unreachable=0  failed=0  skipped=0
```

---

## 📚 Key Ansible Modules Used — Quick Reference

| Module | Purpose | Key Options |
|---|---|---|
| `package` | Install/remove packages | `name`, `state` |
| `service` | Manage system services | `name`, `state`, `enabled` |
| `blockinfile` | Insert/update a block of text in a file | `path`, `block`, `create`, `marker` |
| `file` | Manage file attributes | `path`, `owner`, `group`, `mode` |

---

## 💡 Key Concepts to Remember

- **`become: yes`** — Required for installing packages and managing services (needs root).
- **`hosts: all`** — Targets every host in the inventory when no groups are defined.
- **`blockinfile` vs `copy`** — Use `blockinfile` when you want to insert managed content that Ansible can update/remove cleanly; use `copy` when replacing the whole file.
- **`create: yes` in blockinfile** — Without this, the task fails if the file doesn't exist yet.
- **`state: started` vs `state: enabled`** — `started` starts it now; `enabled` makes it survive reboots. Always use both together for web servers.
- **Default markers in blockinfile** — Never omit them unless told to use custom ones; they allow Ansible to track and update the block in future runs.
