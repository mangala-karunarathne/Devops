# Jenkins Plugin Installation – Study Material
> **Task Reference:** Nautilus DevOps – Git & GitLab Plugin Setup  
> **Date Completed:** April 2026  
> **Status:** ✅ Completed Successfully

---

## 📋 Original Task

> The Nautilus DevOps team has recently setup a Jenkins server, which they want to use for some CI/CD jobs. Before that they want to install some plugins which will be used in most of the jobs. Please find below more details about the task:
>
> 1. Click on the Jenkins button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.
> 2. Once logged in, install the `Git` and `GitLab` plugins. You may need to restart Jenkins to complete the plugin installation; if required, opt to **Restart Jenkins when installation is complete and no jobs are running** on the plugin installation/update page (Update Centre).
>
> **Note:**
> - After restarting Jenkins, wait for the login page to reappear before proceeding.
> - For tasks involving web UI changes, capture screenshots to share for review or consider using screen recording software like loom.com for documentation and sharing.

---

## 🧠 Concepts to Understand

### What is Jenkins?
Jenkins is an open-source **automation server** used to build, test, and deploy software — the backbone of many CI/CD (Continuous Integration / Continuous Delivery) pipelines. It is written in Java and supports hundreds of plugins to extend its capabilities.

### What is a Jenkins Plugin?
A **plugin** is an add-on that extends Jenkins functionality. Plugins allow Jenkins to integrate with external tools like Git, GitLab, Docker, Kubernetes, Slack, etc.

- Plugins are managed via **Manage Jenkins → Plugins**
- They can be installed from the **Jenkins Plugin Index** (loaded in the "Available plugins" tab)
- Some plugins require a **restart** to activate

### What is the Git Plugin?
- Integrates **Git version control** with Jenkins
- Allows Jenkins jobs/pipelines to **clone, fetch, and checkout** code from Git repositories
- Essential for any pipeline that pulls source code from a Git-based repository (GitHub, GitLab, Bitbucket, etc.)
- **Version installed:** 5.10.1

### What is the Git Client Plugin?
- A **utility/dependency plugin** for Git support in Jenkins
- Provides the underlying Git client library used by the Git plugin and others
- Usually installed automatically as a dependency
- **Version installed:** 6.6.0

### What is the GitLab Plugin?
- Allows **GitLab to trigger Jenkins builds** via webhooks
- Displays Jenkins build results back in the **GitLab UI** (merge requests, pipelines)
- Enables tight integration between GitLab and Jenkins for CI/CD workflows
- **Version installed:** 1.9.13

---

## 🪜 Step-by-Step Process

### Step 1: Access Jenkins UI
- Open the Jenkins URL in your browser (via the top bar button in lab environments)
- **Expected:** Login page appears

### Step 2: Log In
| Field    | Value       |
|----------|-------------|
| Username | `admin`     |
| Password | `Adm!n321`  |

- **Expected:** Jenkins Dashboard loads

### Step 3: Navigate to Plugin Manager
```
Manage Jenkins → Plugins → Available plugins tab
```
- **Expected:** A searchable list of installable plugins

### Step 4: Search and Select Git Plugin
1. Type `Git` in the search box
2. Locate **"Git plugin"** — *"This plugin integrates Git with Jenkins"*
3. Check the checkbox ✅

### Step 5: Search and Select GitLab Plugin
1. Clear search, type `GitLab`
2. Locate **"GitLab Plugin"** — *"This plugin allows GitLab to trigger Jenkins builds..."*
3. Check the checkbox ✅

### Step 6: Install Plugins
- Click the **"Install"** button
- **Expected:** Installation progress page (Update Centre) appears

### Step 7: Enable Auto-Restart
- Check: **"Restart Jenkins when installation is complete and no jobs are running"**
- **Expected:** Jenkins restarts automatically after installation

### Step 8: Wait and Re-Login
- Wait ~30–60 seconds for Jenkins to come back up
- Log in again with `admin` / `Adm!n321`

### Step 9: Verify Installation
```
Manage Jenkins → Plugins → Installed plugins → Search "git"
```
- Confirm all three plugins appear with **Enabled** status ✅

---

## ✅ Verification Results (Screenshot Evidence)

| Plugin            | Version | Health Score | Status        |
|-------------------|---------|--------------|---------------|
| Git client plugin | 6.6.0   | 100 ✅        | Enabled       |
| Git plugin        | 5.10.1  | 100 ✅        | Enabled       |
| GitLab Plugin     | 1.9.13  | 97 ✅         | Enabled (Blue toggle) |

---

## 🔁 Jenkins Restart – Key Notes

| Situation | What to Do |
|-----------|------------|
| Plugin needs restart | Check "Restart Jenkins when installation is complete and no jobs are running" |
| Jenkins is restarting | Wait for the **login page** to fully reappear before doing anything |
| After restart | Log back in and verify installed plugins |

> ⚠️ **Never force-close Jenkins mid-installation** — it can corrupt plugin files.

---

## 🔗 Plugin Dependencies

When you install a plugin, Jenkins may automatically install **dependency plugins** alongside it. For example:
- Installing **Git plugin** also pulls in **Git client plugin**
- This is normal and expected behavior

Always check the **Installed plugins** tab after installation — you may see more plugins than you explicitly selected.

---

## 💡 Key Takeaways

1. **Jenkins plugins** extend its functionality — Git and GitLab plugins are among the most commonly used in CI/CD.
2. The **Plugin Manager** is accessed via `Manage Jenkins → Plugins`.
3. Always use the **"Restart when no jobs are running"** option to safely restart Jenkins.
4. After restart, **wait for the login page** before proceeding — Jenkins takes 30–60 seconds.
5. Verify installations under **Installed plugins** tab by searching for plugin names.
6. A **Health Score of 100** means the plugin is fully stable; anything above 80 is generally acceptable.

---

## 📚 Further Reading

- [Jenkins Official Documentation](https://www.jenkins.io/doc/)
- [Jenkins Plugin Index](https://plugins.jenkins.io/)
- [Git Plugin Page](https://plugins.jenkins.io/git/)
- [GitLab Plugin Page](https://plugins.jenkins.io/gitlab-plugin/)
- [Jenkins CI/CD Best Practices](https://www.jenkins.io/doc/book/pipeline/pipeline-best-practices/)

---

*Study material generated based on hands-on task completion. Screenshots were captured during task execution for verification.*
