# Jenkins Installation & Setup — Study Material
> Based on KodeKloud DevOps Course | CentOS 9

---

## 📌 What is Jenkins?
- Jenkins is an open-source **CI/CD automation server**
- Built on **Java** — runs as a `.war` (Web Application Archive) file inside a JVM
- Used to automate building, testing, and deploying applications
- Accessed via a **web UI** on port `8080` by default

---

## 🔧 Prerequisites

### 1. Verify OS Before Anything
```bash
cat /etc/os-release
# or
cat /etc/redhat-release
```

### 2. Java is Mandatory
Jenkins cannot run without Java. Choose the right version:

| OS | Java Command |
|----|-------------|
| CentOS/RHEL 7 | `yum install -y java-11-openjdk` |
| CentOS/RHEL 8+ | `yum install -y java-17-openjdk` |
| CentOS 9 | `yum install -y java-17-openjdk` ✅ |

Verify:
```bash
java -version
```

---

## 📦 Installation Steps (CentOS 9)

### Step 1: Add Jenkins Repository (Manual Method - Recommended)
> ⚠️ Using `curl` to download the `.repo` file can result in an HTML redirect being saved instead of the actual file. Always create it manually:

```bash
cat > /etc/yum.repos.d/jenkins.repo << 'EOF'
[jenkins]
name=Jenkins
baseurl=https://pkg.jenkins.io/redhat-stable
gpgcheck=1
gpgkey=https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
enabled=1
EOF
```

Verify the file:
```bash
cat /etc/yum.repos.d/jenkins.repo
```

### Step 2: Import GPG Key
```bash
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

### Step 3: Install Jenkins
```bash
yum clean all
yum install -y jenkins
```

> ⚠️ If you get **GPG check FAILED**, use:
> ```bash
> yum install -y jenkins --nogpgcheck
> ```
> This is acceptable in lab/course environments.

### Step 4: Enable & Start Jenkins
```bash
systemctl enable jenkins
systemctl start jenkins
systemctl status jenkins
```

> ⚠️ If you face a **timeout** issue starting Jenkins:
> ```bash
> systemctl edit jenkins
> # Add:
> [Service]
> TimeoutStartSec=300
> ```

---

## 🔑 Initial Setup via Web UI

### Step 1: Get Initial Admin Password
```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```
> This is a **one-time unlock key** — required even if you already know your desired credentials.

### Step 2: Access Jenkins UI
- Click the **Jenkins button** on the KodeKloud top bar
- Paste the initial admin password
- Click **"Install suggested plugins"**

### Step 3: Create Admin User
Fill in your credentials on the setup screen:

| Field | Value (example) |
|-------|----------------|
| Username | `theadmin` |
| Password | `Adm!n321` |
| Full Name | `Yousuf` |
| Email | `yousuf@jenkins.stratos.xfusioncorp.com` |

- Click **Save and Continue**
- Keep default Jenkins URL → **Save and Finish**
- Click **Start using Jenkins** ✅

---

## 🛠️ Useful Commands Cheatsheet

```bash
# Check OS
cat /etc/os-release

# Check Java
java -version

# Jenkins service commands
systemctl start jenkins
systemctl stop jenkins
systemctl restart jenkins
systemctl status jenkins
systemctl enable jenkins      # auto-start on boot

# Check Jenkins is running on port 8080
ss -tlnp | grep 8080

# View Jenkins logs
journalctl -u jenkins -f

# Get initial admin password
cat /var/lib/jenkins/secrets/initialAdminPassword

# Clean yum cache
yum clean all
```

---

## 🐛 Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| `wget: command not found` | wget not pre-installed on CentOS 9 | Use `curl -o` instead |
| `.repo` file contains HTML | curl followed redirect, saved HTML | Create `.repo` file manually |
| `No match for argument: jenkins` | Bad/cached repo | Fix repo file + `yum clean all` |
| `GPG check FAILED` | Key mismatch | `yum install -y jenkins --nogpgcheck` |
| Jenkins service timeout | Slow system startup | Edit service: `TimeoutStartSec=300` |

---

## 💡 Key Concepts to Remember

### Why `yum clean all`?
- Clears cached repo metadata
- Forces yum to re-read updated `.repo` files
- Prevents stale cache from causing "package not found" errors

### Why Initial Admin Password?
- Jenkins security mechanism on first boot
- Proves you have physical/SSH access to the server
- One-time use only — create your permanent admin after unlocking

### curl vs wget
| Tool | Pre-installed on CentOS 9 | Install Command |
|------|--------------------------|-----------------|
| `curl` | ✅ Yes | — |
| `wget` | ❌ No | `yum install -y wget` |

### Jenkins Min Java Requirements
| Jenkins Version | Min Java |
|----------------|----------|
| Jenkins 2.357+ | Java 11+ |
| Jenkins 2.426+ | Java 17+ |
| Older Jenkins  | Java 8   |

---

## 📁 Important File Locations

| File/Directory | Purpose |
|---------------|---------|
| `/etc/yum.repos.d/jenkins.repo` | Jenkins yum repository config |
| `/var/lib/jenkins/` | Jenkins home directory |
| `/var/lib/jenkins/secrets/initialAdminPassword` | One-time unlock password |
| `/var/log/jenkins/jenkins.log` | Jenkins log file |
| `/usr/share/java/jenkins.war` | Jenkins application file |

---

*Study material based on KodeKloud CI/CD with Jenkins course — CentOS 9 environment*
