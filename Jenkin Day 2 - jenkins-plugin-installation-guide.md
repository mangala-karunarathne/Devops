# 🔧 Jenkins Plugin Installation – Study Notes
> KodeKloud DevOps Learning | Topic: Jenkins CI/CD Basics

---

## 📋 Original Task (KodeKloud)

> The Nautilus DevOps team has recently setup a Jenkins server, which they want to use for some CI/CD jobs. Before that they want to install some plugins which will be used in most of the jobs.

**Task Requirements:**
1. Access the Jenkins UI using the Jenkins button on the top bar
2. Login using username `admin` and password `Adm!n321`
3. Install the **`Git`** and **`GitLab`** plugins
4. If required, restart Jenkins when installation is complete and no jobs are running (via Update Centre)
5. After restarting, wait for the login page to reappear before proceeding

**Notes from task:**
- For web UI tasks, capture screenshots for review/documentation
- Consider using screen recording tools like [loom.com](https://loom.com) for documentation

---

## 📋 Original Task (KodeKloud)

> **Scenario:** The Nautilus DevOps team has recently set up a Jenkins server, which they want to use for some CI/CD jobs. Before that, they want to install some plugins which will be used in most of the jobs.

### Task Requirements:
1. Access the Jenkins UI using the Jenkins button on the top bar
   - **Username:** `admin`
   - **Password:** `Adm!n321`
2. Once logged in, install the following plugins:
   - ✅ **Git**
   - ✅ **GitLab**
3. If a restart is required, opt to **"Restart Jenkins when installation is complete and no jobs are running"** on the Update Centre page
4. After restarting, **wait for the login page to reappear** before proceeding

### Task Outcome:
- Both **Git plugin** and **GitLab Plugin** successfully installed and enabled ✅
- Verified in **Manage Jenkins → Plugins → Installed plugins**

---

## 📌 What is Jenkins?

Jenkins is an open-source **automation server** used to build, test, and deploy software — the backbone of most CI/CD pipelines.

- Written in Java
- Highly extensible via **plugins** (1800+ available)
- Can integrate with Git, GitLab, GitHub, Docker, Kubernetes, and more

---

## 🧩 What are Jenkins Plugins?

Plugins extend Jenkins functionality. Almost every integration in Jenkins (Git, Docker, Slack, etc.) is powered by a plugin.

| Plugin | Purpose |
|--------|---------|
| **Git plugin** | Allows Jenkins to clone/pull from Git repositories |
| **GitLab plugin** | Allows GitLab to trigger Jenkins builds and show build results in GitLab UI |
| **Git client plugin** | Utility/dependency used by the Git plugin |
| **Pipeline** | Enables Jenkinsfile-based pipeline jobs |
| **Blue Ocean** | Modern UI for Jenkins pipelines |

---

## 🔐 Default Credentials (KodeKloud Lab)

| Field | Value |
|-------|-------|
| Username | `admin` |
| Password | `Adm!n321` |

> ⚠️ Never use default credentials in production environments!

---

## 🛠️ How to Install Plugins – Step by Step

### Step 1: Log In
- Open Jenkins UI
- Enter username and password → Click **Sign In**

### Step 2: Go to Manage Jenkins
- Click **"Manage Jenkins"** from the left sidebar or dashboard

### Step 3: Open Plugin Manager
- Click **"Plugins"** (puzzle piece icon 🧩)
- Click the **"Available plugins"** tab

### Step 4: Search and Select Plugins
- Search for **`Git`** → ✅ Check the box next to **Git plugin**
- Search for **`GitLab`** → ✅ Check the box next to **GitLab Plugin**

### Step 5: Install
- Click the **"Install"** button
- Watch the progress on the **Update Centre** page

### Step 6: Restart Jenkins (if required)
- Check the option: **"Restart Jenkins when installation is complete and no jobs are running"**
- ⏳ Wait for the login page to fully reload before proceeding

### Step 7: Verify Installation
- Go to **Manage Jenkins → Plugins → Installed plugins**
- Search for `Git` and `GitLab`
- Both should show as **Enabled** ✅

---

## ✅ Verification Checklist

- [ ] Logged into Jenkins successfully
- [ ] Navigated to Manage Jenkins → Plugins
- [ ] Found "Git plugin" in Available plugins
- [ ] Found "GitLab Plugin" in Available plugins
- [ ] Both plugins installed successfully
- [ ] Jenkins restarted (if prompted)
- [ ] Both plugins visible in Installed tab with Enabled toggle ON

---

## 📊 Plugin Health Scores (from lab)

| Plugin | Version | Health Score | Status |
|--------|---------|--------------|--------|
| Git client plugin | 6.5.0 | 100 | ✅ Enabled |
| Git plugin | 5.10.0 | 100 | ✅ Enabled |
| GitLab Plugin | 1.9.13 | 97 | ✅ Enabled |

> 💡 Health score of **100** = no known issues. Score of **97** = minor known issues but still fully functional.

---

## 🔄 Plugin Manager Tabs Explained

| Tab | What it shows |
|-----|--------------|
| **Available plugins** | Plugins not yet installed, available to download |
| **Installed plugins** | All currently installed plugins |
| **Updates** | Installed plugins with newer versions available |
| **Advanced** | Upload plugins manually (.hpi files), configure update sites |

---

## 💡 Key Concepts to Remember

### CI/CD Pipeline Flow
```
Code Push → Git Repo → Jenkins Triggered → Build → Test → Deploy
```

### Jenkins + GitLab Integration
- **GitLab plugin** enables webhooks so that a `git push` to GitLab **automatically triggers** a Jenkins job
- Jenkins then runs your pipeline and reports pass/fail **back to GitLab**

### Plugin Dependencies
- Some plugins depend on others (e.g., Git plugin requires Git client plugin)
- Jenkins automatically installs dependencies during plugin installation

---

## ⚠️ Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| Plugin not appearing in Available tab | Click "Check now" to refresh plugin list |
| Installation stuck | Check internet connectivity on Jenkins server |
| Jenkins not restarting | Manually restart via `http://<jenkins-url>/restart` |
| Plugin shows but not working | Check if it's enabled in Installed tab |
| Dependency conflict | Update all plugins first, then install new ones |

---

## 🖥️ Useful Jenkins URLs

| URL | Purpose |
|-----|---------|
| `/manage` | Manage Jenkins page |
| `/pluginManager` | Plugin Manager |
| `/pluginManager/available` | Available plugins tab |
| `/pluginManager/installed` | Installed plugins tab |
| `/restart` | Restart Jenkins |
| `/safeRestart` | Restart when no jobs are running |

---

## 📚 Further Learning Resources

- 🌐 [Jenkins Official Docs](https://www.jenkins.io/doc/)
- 🧩 [Jenkins Plugin Index](https://plugins.jenkins.io/)
- 🎓 [KodeKloud Jenkins Course](https://kodekloud.com)
- 📖 [Git Plugin Docs](https://plugins.jenkins.io/git/)
- 📖 [GitLab Plugin Docs](https://plugins.jenkins.io/gitlab-plugin/)

---

*Notes created during KodeKloud Jenkins Lab — Plugin Installation Task*
