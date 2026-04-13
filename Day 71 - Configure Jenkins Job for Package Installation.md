# Jenkins Job: Install Packages on Stratos Datacenter Storage Server
> Study Material & Task Walkthrough

---

## 📋 Original Task

**Task Requirements:**

1. Access the Jenkins UI and log in using:
   - Username: `admin`
   - Password: `Adm!n321`

2. Create a new Jenkins job named `install-packages` with the following specifications:
   - Add a string parameter named `PACKAGE`
   - Configure the job to install a package specified in the `$PACKAGE` parameter on the **storage server** (Stratos Datacenter)
   - Build the job at least once (e.g. with parameter `PACKAGE=vim-enhanced`) so the package is installed on the Storage server and can be verified

**Notes from task:**
- Ensure to install any required plugins and restart the Jenkins service if necessary
- Opt for `Restart Jenkins when installation is complete and no jobs are running` on the plugin installation/update page
- Verify that the Jenkins job runs successfully on repeated executions to ensure reliability

---

## 🏗️ Stratos Datacenter Infrastructure Reference

| Server | Hostname | IP Address |
|--------|----------|------------|
| Jumphost | `jumphost.stratos.xfusioncorp.com` | `10.244.81.25` |
| Jenkins | `jenkins.stratos.xfusioncorp.com` | `10.244.13.46` |
| Storage | `ststor01.stratos.xfusioncorp.com` | `10.244.97.192` |
| App01 | `stapp01.stratos.xfusioncorp.com` | `10.244.195.61` |
| App02 | `stapp02.stratos.xfusioncorp.com` | `10.244.164.25` |
| App03 | `stapp03.stratos.xfusioncorp.com` | `10.244.240.173` |

> ⚠️ **Important:** IPs may vary per lab session. Always verify with `cat /etc/hosts` on the jumphost before configuring SSH connections.

### Storage Server Credentials
- **User:** `natasha`
- **Password:** `Bl@kW`
- **Sudo:** Yes (with password)

---

## 🔧 Step-by-Step Solution

### Step 1: Log into Jenkins UI

- Navigate to Jenkins URL and log in
- **Username:** `admin`
- **Password:** `Adm!n321`

---

### Step 2: Install the "Publish Over SSH" Plugin

This plugin allows Jenkins to execute commands on remote servers over SSH.

1. Go to **Manage Jenkins → Manage Plugins**
2. Click the **Available** tab
3. Search for `Publish Over SSH`
4. Check the checkbox and click **Install**
5. Select **"Restart Jenkins when installation is complete and no jobs are running"**
6. Wait for restart, then refresh the page and log back in

**Verify:** Plugin should appear under the **Installed** tab.

---

### Step 3: Configure SSH Connection to Storage Server

1. Go to **Manage Jenkins → Configure System**
2. Scroll to the **Publish over SSH** section
3. Click **Add** under SSH Servers
4. Fill in the details:

| Field | Value |
|-------|-------|
| Name | `storage-server` |
| Hostname | `10.244.97.192` *(verify from /etc/hosts)* |
| Username | `natasha` |
| Remote Directory | `/tmp` |

5. Click **Advanced** → check **Use password authentication**
   - Password: `Bl@kW`
6. Click **Test Configuration**
7. Click **Save**

> ⚠️ **504 Gateway Timeout on Test Configuration** is a known nginx proxy timeout issue in this lab — it does NOT mean your config is wrong. Proceed with Save anyway; the actual build job will work.

---

### Step 4: Create the Jenkins Job

1. From Dashboard → click **New Item**
2. Enter name: `install-packages`
3. Select **Freestyle project**
4. Click **OK**

---

### Step 5: Add String Parameter

1. Check **"This project is parameterized"**
2. Click **Add Parameter → String Parameter**
3. Configure:

| Field | Value |
|-------|-------|
| Name | `PACKAGE` |
| Default Value | *(leave empty or `vim-enhanced`)* |
| Description | `Package name to install on storage server` |

---

### Step 6: Add Build Step

1. Scroll to **Build** section
2. Click **Add build step → Send files or execute commands over SSH**
3. Configure:

| Field | Value |
|-------|-------|
| SSH Server Name | `storage-server` |
| Exec command | `sudo yum install -y $PACKAGE` |

4. Click **Save**

---

### Step 7: Build the Job

1. Go to the `install-packages` job page
2. Click **Build with Parameters**
3. Enter `PACKAGE = vim-enhanced`
4. Click **Build**

---

### Step 8: Verify Console Output

Click **Build #1 → Console Output**

**Expected successful output:**
```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/install-packages
SSH: Connecting from host [jenkins.stratos.xfusioncorp.com]
SSH: Connecting with configuration [storage-server] ...
SSH: EXEC: completed after 8,204 ms
SSH: Disconnecting configuration [storage-server] ...
SSH: Transferred 0 file(s)
Build step 'Send files or execute commands over SSH' changed build result to SUCCESS
Finished: SUCCESS
```

---

### Step 9: Verify Package on Storage Server

From the jumphost terminal:

```bash
ssh natasha@10.244.97.192 "rpm -qa | grep vim-enhanced"
```

**Expected output:**
```
vim-enhanced-7.4.629-8.el7_9.x86_64
```

---

## 🐛 Troubleshooting

### Issue 1: Connection Timed Out (`Finished: UNSTABLE`)

```
ERROR: Failed to connect and initialize SSH connection.
Message: [java.net.ConnectException: Connection timed out]
```

**Cause:** Wrong IP address for the storage server.

**Fix:**
1. On jumphost, run: `cat /etc/hosts`
2. Find the correct IP for `ststor01.stratos.xfusioncorp.com`
3. Update **Manage Jenkins → Configure System → Publish Over SSH** with the correct IP
4. Re-run the job

**Verify connectivity before configuring:**
```bash
nc -zv 10.244.97.192 22
# Expected: Ncat: Connected to 10.244.97.192:22
```

---

### Issue 2: 504 Gateway Timeout on "Test Configuration"

**Cause:** nginx reverse proxy timeout — not a Jenkins or SSH config error.

**Fix:** Ignore it. Click **Save** and proceed. The actual build will work fine.

---

### Issue 3: Remote Directory Permission Error

**Cause:** Using `/` as Remote Directory causes permission issues.

**Fix:** Change Remote Directory to `/tmp` in the SSH server configuration.

---

## 📚 Key Concepts

### What is "Publish Over SSH" Plugin?
A Jenkins plugin that allows you to:
- Execute shell commands on remote servers over SSH
- Transfer files to remote servers
- Use parameterized values (like `$PACKAGE`) in remote commands

### What is a Parameterized Jenkins Job?
A job that accepts input values at build time. Instead of hardcoding values, you define parameters (like `PACKAGE`) that can be passed in each time the job runs — making the job reusable for different inputs.

### Why `/tmp` as Remote Directory?
- The remote directory is the working directory Jenkins uses on the remote server
- `/tmp` is writable by all users, avoiding permission issues
- `/` (root) may not be writable by the SSH user

### How `sudo yum install` Works Without Interactive Password
In this lab, the `natasha` user has `sudo` privileges configured to allow package installation. In production environments, you would configure `/etc/sudoers` with `NOPASSWD` for specific commands.

---

## ✅ Task Completion Checklist

- [ ] Logged into Jenkins UI
- [ ] Installed `Publish Over SSH` plugin
- [ ] Jenkins restarted after plugin install
- [ ] SSH server `storage-server` configured with correct IP
- [ ] Jenkins job `install-packages` created as Freestyle project
- [ ] String parameter `PACKAGE` added
- [ ] Build step configured with SSH exec command `sudo yum install -y $PACKAGE`
- [ ] Job built with `PACKAGE=vim-enhanced`
- [ ] Console Output shows `Finished: SUCCESS`
- [ ] Package verified on storage server with `rpm -qa | grep vim-enhanced`
- [ ] Job run a second time to verify reliability

---

## 🔁 Re-running for Different Packages

The job is reusable. To install any other package:

1. Go to `install-packages` job
2. Click **Build with Parameters**
3. Enter any package name (e.g., `httpd`, `git`, `wget`)
4. Click **Build**

---

*Study material generated from hands-on lab walkthrough — Stratos Datacenter / KodeKloud Jenkins task*
