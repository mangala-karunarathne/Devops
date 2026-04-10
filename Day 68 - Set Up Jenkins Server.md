# Jenkins Setup Study Material
### Task: CI/CD Pipeline Setup for xFusionCorp Industries

---

## 📋 Original Task

> The DevOps team at xFusionCorp Industries is initiating the setup of CI/CD pipelines and has decided to utilize Jenkins as their server. Execute the task according to the provided requirements:
>
> 1. Install `Jenkins` on the jenkins server using the `apt` utility only, and start it using the `service` command.
>    - If you face a timeout issue while starting the Jenkins service, first check the service status with `service jenkins status`
>    - Then review the logs in `/var/log/jenkins/jenkins.log` to identify the cause.
> 2. Jenkins's admin user name should be `theadmin`, password should be `Adm!n321`, full name should be `Mariyam` and email should be `mariyam@jenkins.stratos.xfusioncorp.com`.
>
> **Note:**
> 1. To access the `jenkins` server, connect from the jump host using the `root` user with the password `S3curePass`.
> 2. After Jenkins server installation, click the `Jenkins` button on the top bar to access the Jenkins UI and follow on-screen instructions to create an admin user.

---

## 🔐 Credentials Summary

| Purpose | Username | Password |
|---|---|---|
| SSH into Jenkins server | `root` | `S3curePass` |
| Jenkins Admin Login | `theadmin` | `Adm!n321` |

| Jenkins Admin Field | Value |
|---|---|
| Username | `theadmin` |
| Password | `Adm!n321` |
| Full Name | `Mariyam` |
| Email | `mariyam@jenkins.stratos.xfusioncorp.com` |

---

## 🧠 Key Concepts Before Starting

### What is Jenkins?
Jenkins is an open-source automation server used to build CI/CD (Continuous Integration / Continuous Delivery) pipelines. It automates repetitive tasks like building, testing, and deploying software.

### What is a Jump Host?
A jump host (also called a bastion host) is an intermediary server used to securely access other servers in a network. You SSH into the jump host first, then SSH from there into the target server.

### What is a Hostname?
In lab environments like KodeKloud/Stratos DC, servers are given short names (hostnames) like `jenkins`, `webserver`, `dbserver`. These are mapped to IP addresses in the `/etc/hosts` file on the jump host — so you can use the name instead of the IP.

```
# Example /etc/hosts entry on jump host
172.16.238.10   jenkins
```

---

## 🪜 Step-by-Step Commands

### Step 1: SSH into the Jenkins Server from Jump Host

```bash
ssh root@jenkins
# Password: S3curePass
```

### Step 2: Update Package Lists

```bash
apt-get update -y
```

### Step 3: Install Java (Required Dependency)

Jenkins runs on Java, so Java must be installed first.

```bash
apt-get install -y openjdk-17-jdk
```

Verify installation:

```bash
java -version
```

### Step 4: Add Jenkins Repository Key

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

### Step 5: Add Jenkins Repository

```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Step 6: Update Package Lists Again

```bash
apt-get update -y
```

### Step 7: Install Jenkins using apt

```bash
apt-get install -y jenkins
```

### Step 8: Start Jenkins using service command

```bash
service jenkins start
```

### Step 9: Verify Jenkins is Running

```bash
service jenkins status
```

Expected output:
```
● jenkins.service - Jenkins Continuous Integration Server
   Active: active (running) ...
```

### Step 10: Get Initial Admin Password

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the 32-character string — you will need it to unlock Jenkins in the browser.

---

## 🖥️ Jenkins UI Setup (Browser)

1. Click the **Jenkins button** on the top bar in the lab
2. On the **"Unlock Jenkins"** screen, paste the initial admin password
3. Click **"Install suggested plugins"** and wait for completion
4. Fill in the **Create Admin User** form:
   - Username: `theadmin`
   - Password: `Adm!n321`
   - Full name: `Mariyam`
   - Email: `mariyam@jenkins.stratos.xfusioncorp.com`
5. Click **Save and Continue**
6. On the Instance Configuration screen, leave the URL as default and click **Save and Finish**
7. Click **Start using Jenkins**

---

## 🚨 Troubleshooting

### Timeout when starting Jenkins

```bash
# Check service status
service jenkins status

# Check logs for errors
cat /var/log/jenkins/jenkins.log
```

**Common cause:** Wrong Java version installed. Jenkins requires a specific JDK. Check the log and install the correct version.

---

## 📂 Important File Locations

| File/Path | Purpose |
|---|---|
| `/var/lib/jenkins/secrets/initialAdminPassword` | One-time password to unlock Jenkins UI |
| `/var/log/jenkins/jenkins.log` | Jenkins log file for debugging |
| `/etc/apt/sources.list.d/jenkins.list` | Jenkins apt repository entry |
| `/usr/share/keyrings/jenkins-keyring.asc` | Jenkins GPG key for apt verification |
| `/etc/hosts` | Hostname-to-IP mapping on the jump host |

---

## 🔑 Why We Knew the Credentials

- **SSH password (`S3curePass`)** — provided in the task's Note section
- **Jenkins admin credentials** — provided in Requirement #2 of the task
- **Server name (`jenkins`)** — stated in the task; pre-mapped in `/etc/hosts` on the jump host

---

## 💡 Simple Analogy

| Real World | This Task |
|---|---|
| Entering a building with a key | SSH into jump host |
| Taking an elevator to the right floor | SSH from jump host to Jenkins server |
| Installing an app on your computer | `apt-get install jenkins` |
| Pressing the power button | `service jenkins start` |
| First-time setup wizard on a new device | Jenkins UI initial configuration |
| Creating your account on a new app | Creating the `theadmin` admin user |

---

## ✅ Verification Checklist

- [ ] Jenkins installed via `apt`
- [ ] Jenkins started via `service jenkins start`
- [ ] Jenkins service shows `active (running)`
- [ ] Jenkins UI accessible in browser
- [ ] Admin user created with correct details
- [ ] Successfully logged in as `theadmin`

---

*Study material prepared for xFusionCorp Industries Jenkins CI/CD setup task.*
