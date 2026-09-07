# WSO2 Linux Challenge — Prep Guide

**Exam:** September 20, 10:00–11:00 AM
**Program:** Linux Systems Administration & DevOps Engineering
**Time to prepare:** ~13 days

---

## How to think about this exam

A "Linux Challenge" for a sysadmin/DevOps shortlist is almost always testing **hands-on command-line fluency**, not theory. Expect tasks like: navigate a filesystem, manage users/permissions, troubleshoot a broken service, inspect logs, manage processes, configure networking, and maybe write a small shell script. One hour usually means 15–25 short practical tasks or a handful of scenario-based problems — speed and muscle memory matter as much as knowledge.

Since you already have Docker, Kubernetes, Terraform, and GitHub Actions exposure, you likely have decent Linux exposure already — but exams like this probe the fundamentals people skip when they live inside containers (permissions, systemd, process signals, disk management). Prioritize those gaps.

---

## Core topics to master (ranked by likely weight)

### 1. Filesystem & permissions (high priority)
- Navigation: `pwd`, `cd`, `ls -la`, `find`, `locate`, `tree`
- File ops: `cp`, `mv`, `rm`, `ln -s` vs `ln` (hard vs symbolic links)
- Permissions: `chmod` (numeric AND symbolic: `u+x`, `go-w`), `chown`, `chgrp`
- Special permissions: SUID, SGID, sticky bit
- `umask` and default permissions
- Filesystem hierarchy standard (`/etc`, `/var`, `/opt`, `/proc`, `/sys`, `/tmp`)

### 2. Users, groups & access control (high priority)
- `useradd`, `usermod`, `userdel`, `groupadd`, `passwd`
- `/etc/passwd`, `/etc/shadow`, `/etc/group` structure
- `sudo` and `/etc/sudoers` (visudo)
- `su` vs `sudo`

### 3. Process & service management (high priority)
- `ps aux`, `top`/`htop`, `kill`, `killall`, signals (`SIGTERM` vs `SIGKILL`, `kill -9` vs `kill -15`)
- Job control: `&`, `nohup`, `jobs`, `fg`, `bg`, `disown`
- **systemd**: `systemctl start/stop/restart/status/enable/disable`, `journalctl -u <service>`, writing a basic `.service` unit file
- Boot process basics (what systemd does at boot, runlevels/targets)

### 4. Networking (medium-high priority)
- `ip a` / `ifconfig`, `ip route`, `ping`, `traceroute`
- `netstat -tulnp` / `ss -tulnp` (what's listening on what port)
- `curl`, `wget`, `dig`, `nslookup`
- `/etc/hosts`, `/etc/resolv.conf`
- Firewall basics: `ufw` or `iptables`/`firewalld` fundamentals
- SSH: key-based auth, `~/.ssh/authorized_keys`, `ssh-keygen`, `scp`, `rsync`

### 5. Package & software management (medium priority)
- `apt`/`dpkg` (Debian/Ubuntu) and `yum`/`dnf`/`rpm` (RHEL/CentOS) — know both families
- Adding repos, checking installed packages, dependency resolution

### 6. Disk & storage (medium priority)
- `df -h`, `du -sh`, `lsblk`, `fdisk`/`parted`
- Mounting: `mount`, `umount`, `/etc/fstab`
- Basic LVM concepts (physical/volume/logical groups) — know the vocabulary even if not hands-on

### 7. Text processing & shell scripting (high priority — often the differentiator)
- `grep`, `sed`, `awk`, `cut`, `sort`, `uniq`, `wc`, `tr`
- Pipes and redirection: `|`, `>`, `>>`, `2>&1`
- Basic bash scripting: variables, `if`/`for`/`while`, `$1`/`$@`, exit codes, `#!/bin/bash`
- `cron` and `crontab -e` syntax

### 8. Logs & troubleshooting (high priority)
- `/var/log/syslog`, `/var/log/auth.log`, `journalctl`
- `tail -f`, `less`, `grep` on logs
- A classic scenario: "service won't start — find out why" (check status → check logs → check config → check permissions/ports)

### 9. Environment & shell config
- `.bashrc`, `.bash_profile`, `/etc/environment`, `PATH`
- Environment variables: `export`, `env`

---

## 13-Day Study Plan

| Days | Focus | Goal |
|---|---|---|
| 1–2 | Filesystem, permissions, users/groups | Comfortable with `chmod`/`chown` numeric+symbolic, sudoers, user lifecycle |
| 3–4 | Process & service management (systemd deep dive) | Can start/stop/enable services, read `journalctl`, write a simple unit file |
| 5–6 | Networking + SSH | Comfortable diagnosing "what's listening", SSH key setup, basic firewall rules |
| 7–8 | Text processing & bash scripting | Can write a 10–15 line script solving a real task under time pressure |
| 9 | Package management + disk/storage | Know both apt and yum syntax; can read `df`/`du`/`lsblk` output |
| 10 | Logs & troubleshooting scenarios | Practice the "diagnose a broken service" workflow end-to-end |
| 11 | Timed mock drills | Simulate exam pressure (see below) |
| 12 | Weak-spot review | Revisit whatever felt slow on day 11 |
| 13 | Light review only | Skim notes, sleep well — no new material the day before |

---

## How to practice (not just read)

Reading won't build the speed this exam rewards. Set up a free VM or container and force yourself to solve problems blind:

- **Spin up an Ubuntu VM** (VirtualBox, or a cloud free-tier instance) — practicing in a real shell beats reading command references.
- **OverTheWire: Bandit** — a well-known free wargame that builds exactly this kind of command-line fluency through puzzles, level by level.
- **Break things on purpose**: revoke a permission, kill a service's config file, misconfigure a firewall rule — then fix it using only the diagnostic workflow (status → logs → config → permissions).
- **Time yourself**: once you're comfortable, do 10 random tasks in 15 minutes to build exam-pressure speed.

---

## Exam-day tips

- Arrive with a clear head — skim your notes 30 min before, don't cram new material.
- If it's a practical/CLI test, work efficiently: use `history` and tab-completion, don't second-guess simple commands.
- If you get stuck on a task, note it and move on — partial completion across many tasks usually scores better than perfecting one.
- Read each question fully before typing — CLI exams often penalize rushing into the wrong command.

---

*Given your existing Docker/K8s/Terraform background, you're well-positioned — this guide is weighted toward the raw Linux fundamentals that container/IaC work sometimes lets people skip.*
