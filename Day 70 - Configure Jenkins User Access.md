# Jenkins CI/CD User Access Configuration — Study Material

---

## 📋 Original Task

> **The Nautilus team is integrating Jenkins into their CI/CD pipelines. After setting up a new Jenkins server, they're now configuring user access for the development team. Follow these steps:**
>
> 1. Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login with username `admin` and password `Adm!n321`.
> 2. Create a Jenkins user named `rose` with the password `B4zNgHA7Ya`. Their full name should match `Rose`.
> 3. Utilize the `Project-based Matrix Authorization Strategy` to assign `overall read` permission to the `rose` user.
> 4. Remove all permissions for `Anonymous` users (if any), ensuring that the `admin` user retains overall `Administer` permissions.
> 5. For the existing job, grant `rose` user only `read` permissions, disregarding other permissions such as Agent, SCM, etc.
>
> **Notes:**
> - You may need to install plugins and restart Jenkins service. After plugin installation, select `Restart Jenkins when installation is complete and no jobs are running`.
> - After restarting the Jenkins service, wait for the Jenkins login page to reappear before proceeding. Avoid clicking `Finish` immediately after restarting the service.
> - Capture screenshots of your configuration for review purposes.

---

## 🧠 Key Concepts

### What is Jenkins?
Jenkins is an open-source automation server used to implement **CI/CD (Continuous Integration / Continuous Delivery)** pipelines. It automates building, testing, and deploying applications.

### What is CI/CD?
| Term | Meaning |
|------|---------|
| **CI (Continuous Integration)** | Automatically build and test code on every commit |
| **CD (Continuous Delivery)** | Automatically prepare tested code for deployment |
| **CD (Continuous Deployment)** | Automatically deploy code to production after tests pass |

### What is the Matrix Authorization Strategy?
A Jenkins security model where permissions are granted to users or groups in a **matrix (table) format**. Each row is a user/group, and each column is a specific permission (read, build, configure, delete, etc.).

- **Global Matrix Authorization** — applies permissions system-wide
- **Project-based Matrix Authorization** — allows setting permissions at the individual job/project level in addition to global settings

---

## 🔌 Plugin: Matrix Authorization Strategy

### Why is it needed?
Jenkins by default offers limited authorization options. The **Matrix Authorization Strategy** plugin unlocks fine-grained, per-user, per-permission control.

### How to Install
1. Go to **Manage Jenkins** → **Manage Plugins**
2. Click the **Available plugins** tab
3. Search for `Matrix Authorization Strategy`
4. Check the box and click **Install without restart**
5. On the progress page, check ✅ **"Restart Jenkins when installation is complete and no jobs are running"**
6. Wait for Jenkins to restart and the login page to reappear

---

## 👤 User Management in Jenkins

### Creating a New User
- Path: **Manage Jenkins** → **Manage Users** → **Create User**
- Fields: Username, Password, Full Name, Email Address
- Users are stored in Jenkins' internal user database (by default)

### User Credentials Used in This Task
| Field | Value |
|-------|-------|
| Username | `rose` |
| Password | `B4zNgHA7Ya` |
| Full Name | `Rose` |

---

## 🔐 Security Configuration

### Accessing Security Settings
**Manage Jenkins** → **Configure Global Security**

### Authorization Options in Jenkins
| Option | Description |
|--------|-------------|
| Anyone can do anything | No security — not recommended |
| Legacy mode | Admins have full control, others read-only |
| Logged-in users can do anything | All authenticated users have full access |
| Matrix-based security | Fine-grained global permissions per user |
| **Project-based Matrix Authorization Strategy** | Fine-grained global + per-job permissions per user ✅ |

---

## 🗂️ Permission Matrix Reference

### Overall (Global) Permissions
| Permission | Description |
|------------|-------------|
| **Administer** | Full system access — manage plugins, users, settings |
| **Read** | Can log in and view Jenkins dashboard |
| **RunScripts** | Can run Groovy scripts in Script Console |
| **UploadPlugins** | Can upload plugins manually |
| **ConfigureUpdateCenter** | Can manage update center settings |

### Job-Level Permissions
| Permission | Description |
|------------|-------------|
| **Read** | Can view the job and its build history |
| **Build** | Can trigger builds |
| **Configure** | Can modify job settings |
| **Delete** | Can delete the job |
| **Cancel** | Can abort running builds |
| **Discover** | Redirects anonymous users to login if job exists |
| **Move** | Can move the job to another folder |
| **Workspace** | Can view the job's workspace files |

### Other Permission Categories
- **Agent** — manage build agents/nodes
- **SCM** — source control management tag operations
- **View** — manage dashboard views
- **Credentials** — manage stored credentials
- **Lockable Resources** — manage resource locks

---

## ✅ Configuration Summary for This Task

### Global Security Matrix
| User | Administer | Overall Read |
|------|-----------|-------------|
| `admin` | ✅ | ✅ (implied) |
| `rose` | ❌ | ✅ |
| `anonymous` | ❌ | ❌ |

### Job-Level Permission for `rose`
| Permission Category | Permission | Granted to `rose` |
|--------------------|------------|-------------------|
| Job | Read | ✅ |
| Job | Build | ❌ |
| Job | Configure | ❌ |
| Job | Delete | ❌ |
| Agent | All | ❌ |
| SCM | All | ❌ |
| Others | All | ❌ |

---

## 🪜 Step-by-Step Configuration Guide

### Step 1 — Login to Jenkins
1. Open the Jenkins URL
2. Enter username: `admin`, password: `Adm!n321`
3. Click **Sign In**

### Step 2 — Install Matrix Authorization Strategy Plugin
1. **Manage Jenkins** → **Manage Plugins** → **Available plugins**
2. Search `Matrix Authorization Strategy`, check it, click **Install without restart**
3. Check ✅ **"Restart Jenkins when installation is complete and no jobs are running"**
4. Wait for the login page to reappear, then log back in

### Step 3 — Create User `rose`
1. **Manage Jenkins** → **Manage Users** → **Create User**
2. Fill in: username `rose`, password `B4zNgHA7Ya`, full name `Rose`
3. Click **Create User**

### Step 4 — Configure Global Security
1. **Manage Jenkins** → **Configure Global Security**
2. Under **Authorization**, select **Project-based Matrix Authorization Strategy**
3. Add `admin` → check **Administer**
4. Add `rose` → check only **Overall > Read**
5. For `Anonymous` row → uncheck ALL permissions
6. Click **Save**

### Step 5 — Configure Job-Level Permissions for `rose`
1. Open the existing job from the dashboard
2. Click **Configure** in the left sidebar
3. Scroll down → check ✅ **Enable project-based security**
4. Click **Add user or group** → enter `rose`
5. Under **Job** column → check only **Read**
6. Leave all other permissions unchecked
7. Click **Save**

### Step 6 — Verify
1. Open a private/incognito browser window
2. Log in as `rose` / `B4zNgHA7Ya`
3. Confirm rose can see the dashboard and the job
4. Confirm rose does NOT see Build, Configure, or Delete options
5. Log out → confirm redirect to login page (anonymous access blocked)

---

## ⚠️ Common Mistakes to Avoid

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Clicking **Finish** immediately after plugin install | Jenkins may not restart properly | Wait for the login page to reappear |
| Not enabling **"Enable project-based security"** on the job | Job-level matrix won't appear | Scroll down in job config and check the box |
| Leaving Anonymous user with **Read** access | Unauthenticated users can view Jenkins | Uncheck all Anonymous permissions |
| Forgetting to add `admin` to the matrix with **Administer** | Admin gets locked out | Always add admin with Administer before saving |
| Granting `rose` too many job permissions | Security misconfiguration | Grant only Job > Read at the job level |

---

## 🔑 Security Best Practices

1. **Principle of Least Privilege** — Give users only the permissions they need, nothing more
2. **Never leave Anonymous access enabled** in production environments
3. **Always verify admin access** before saving security changes to avoid lockouts
4. **Use Project-based Matrix** when different jobs require different access levels
5. **Regularly audit user permissions** as team members join or leave
6. **Use strong passwords** for all Jenkins user accounts
7. **Keep plugins updated** — security vulnerabilities are often patched in updates

---

## 📚 Useful Jenkins URLs (Reference)

| Page | Path |
|------|------|
| Dashboard | `/` |
| Manage Jenkins | `/manage` |
| Manage Users | `/manage/securityRealm/` |
| Configure Global Security | `/manage/configureSecurity/` |
| Plugin Manager | `/manage/pluginManager/` |
| Installed Plugins | `/manage/pluginManager/installed` |
| System Log | `/manage/log/` |

---

## 🧩 Glossary

| Term | Definition |
|------|------------|
| **Jenkins** | Open-source CI/CD automation server |
| **CI/CD** | Continuous Integration / Continuous Delivery — automated build, test, deploy pipelines |
| **Plugin** | Extension that adds features to Jenkins |
| **Matrix Authorization** | Permission model using a user × permission table |
| **Project-based Security** | Per-job permission overrides on top of global permissions |
| **Anonymous User** | An unauthenticated (not logged in) visitor |
| **Administer** | Highest Jenkins permission — full system control |
| **Overall Read** | Minimum permission to log in and view the dashboard |
| **Job Read** | Permission to view a specific job and its builds |
| **Agent** | A machine (node) that runs Jenkins build jobs |
| **SCM** | Source Control Management (e.g., Git) |

---

*Study material prepared based on Jenkins CI/CD User Access Configuration task — Nautilus Team.*
