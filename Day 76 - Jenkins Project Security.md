# Jenkins Project-Based Permissions — Study Guide

---

## 📋 Original Task

> **xFusionCorp Industries** has recruited some new developers. There are already some existing jobs on Jenkins and two of these new developers need permissions to access those jobs.
>
> **Requirements:**
>
> 1. There is an existing Jenkins job named `Packages`, and two existing Jenkins users:
>    - `sam` (password: `sam@pass12345`)
>    - `rohan` (password: `rohan@pass12345`)
>
> 2. Grant permissions to these users to access the `Packages` job as per the details below:
>    - **Inheritance Strategy:** Select `Inherit permissions from parent ACL`
>    - **sam:** `build`, `configure`, `read`
>    - **rohan:** `build`, `cancel`, `configure`, `read`, `update`, `tag`
>
> **Notes:**
> - Do not modify/alter any other existing job configuration.
> - You may need to install plugins and restart Jenkins.
> - Use `Restart Jenkins when installation is complete and no jobs are running` on the Update Centre page.

---

## 🔑 Key Concepts

### 1. Jenkins Authorization Strategies

Jenkins supports multiple authorization strategies. The relevant ones here:

| Strategy | Description |
|---|---|
| **Anyone can do anything** | No access control (unsafe) |
| **Logged-in users can do anything** | All logged-in users have full access |
| **Matrix-based security** | Global matrix of users/groups vs permissions |
| **Project-based Matrix Authorization Strategy** | Per-job permission matrix (used in this task) |

### 2. Two Levels of Permissions (Critical Concept)

This is the most important concept in this task:

```
Level 1: GLOBAL Security Matrix
  └── Controls who can log in to Jenkins at all
  └── Minimum requirement: Overall/Read

Level 2: PROJECT-LEVEL Security Matrix (per job)
  └── Controls what a user can do within a specific job
  └── Requires project-based security to be enabled on the job
```

> ⚠️ **Common Mistake:** Granting only job-level permissions without giving the user `Overall/Read` globally causes an **"Access Denied — missing Overall/Read permission"** error on login.

---

## 🔌 Required Plugin

**Plugin Name:** `Matrix Authorization Strategy`

- Provides both **Matrix-based security** and **Project-based Matrix Authorization Strategy**
- Install via: **Manage Jenkins → Manage Plugins → Available → search "Matrix Authorization Strategy"**
- After install: check ✅ **"Restart Jenkins when installation is complete and no jobs are running"**

---

## 🛠️ Step-by-Step Solution

### Step 1: Login to Jenkins
- URL: Jenkins button on the top bar
- Username: `admin`
- Password: `Adm!n321`

---

### Step 2: Install the Plugin (if not already installed)

1. **Manage Jenkins** → **Manage Plugins**
2. Check **Installed** tab → search `Matrix Authorization Strategy`
3. If not found → go to **Available** tab → search and install
4. On Update Centre page: ✅ **"Restart Jenkins when installation is complete and no jobs are running"**
5. Refresh the page and log back in after restart

---

### Step 3: Enable Project-Based Matrix Authorization (Global Security)

1. **Manage Jenkins** → **Configure Global Security**
2. Under **Authorization**, select:
   `Project-based Matrix Authorization Strategy`
3. **Add `admin` user with ALL permissions** (critical — prevents lockout):
   - Click **Add user...** → type `admin`
   - Check **every checkbox** in the admin row (Overall/Administer covers all)
4. **Add `sam` and `rohan` with ONLY `Overall/Read`**:
   - Click **Add user...** → type `sam` → check only `Overall > Read`
   - Click **Add user...** → type `rohan` → check only `Overall > Read`
5. Click **Save**

> 🔒 **Why add sam/rohan here?** Without `Overall/Read` globally, Jenkins blocks their login entirely before even checking job-level permissions.

---

### Step 4: Configure Project-Level Permissions on the `Packages` Job

1. From the Dashboard, click on the **`Packages`** job
2. Click **Configure** (left sidebar)
3. Scroll down to find **"Enable project-based security"** → check ✅ this option
4. Under **Inheritance Strategy**, select:
   **`Inherit permissions from parent ACL`**
5. Add users and assign permissions:

**Add `sam`:**
- Click **Add user...** → type `sam`
- Check: ✅ Build | ✅ Configure | ✅ Read

**Add `rohan`:**
- Click **Add user...** → type `rohan`
- Check: ✅ Build | ✅ Cancel | ✅ Configure | ✅ Read | ✅ Update | ✅ Tag

6. Click **Save**

---

## 📊 Permission Summary Tables

### Global Security Matrix

| User/Group | Overall/Administer | Overall/Read | All others |
|---|---|---|---|
| admin | ✅ | ✅ | ✅ All |
| sam | ❌ | ✅ | ❌ |
| rohan | ❌ | ✅ | ❌ |

### Packages Job — Project-Level Matrix

| Permission | sam | rohan |
|---|---|---|
| Build | ✅ | ✅ |
| Cancel | ❌ | ✅ |
| Configure | ✅ | ✅ |
| Delete | ❌ | ❌ |
| Discover | ❌ | ❌ |
| Read | ✅ | ✅ |
| Workspace | ❌ | ❌ |
| Run/Delete | ❌ | ❌ |
| Run/Update | ❌ | ✅ |
| SCM/Tag | ❌ | ✅ |

---

## ✅ Verification Steps

1. Open a **private/incognito browser window**
2. Go to the Jenkins URL
3. Login as `sam` / `sam@pass12345`
   - Sam should see the `Packages` job
   - Sam should be able to Build and Configure it
4. Repeat with `rohan` / `rohan@pass12345`
   - Rohan should see the `Packages` job
   - Rohan should be able to Build, Cancel, Configure, Update, and Tag

---

## ⚠️ Common Errors & Fixes

### Error: "X is missing the Overall/Read permission"

**Cause:** The user was added to the job-level matrix but NOT to the global security matrix with `Overall/Read`.

**Fix:** Go to **Manage Jenkins → Configure Global Security** → add the user with `Overall > Read` checked.

---

### Error: Locked out of Jenkins after changing Authorization Strategy

**Cause:** Admin user was not added to the new matrix before saving.

**Fix (if locked out):**
1. SSH into the Jenkins server
2. Edit `/var/lib/jenkins/config.xml`
3. Change `<useSecurity>true</useSecurity>` to `<useSecurity>false</useSecurity>`
4. Restart Jenkins: `sudo systemctl restart jenkins`
5. Re-configure security properly with admin user first

---

### Jenkins UI stuck after restart

**Fix:** Simply refresh the browser page and log back in. Jenkins restarts its web server, so the UI may appear frozen briefly.

---

## 🧠 Key Takeaways

1. **Always add admin to the global matrix first** before switching authorization strategies.
2. **Two-level permissions model:** Global (login access) + Project (job-level actions).
3. `Overall/Read` is the **minimum global permission** required for any user to access Jenkins.
4. **Inheritance Strategy** `"Inherit permissions from parent ACL"` means the job also respects global permissions in addition to its own.
5. The **Matrix Authorization Strategy plugin** must be installed for project-based permissions to work.
6. Always **verify in incognito mode** after configuration to confirm permissions work as expected.

---

## 📎 Reference

- Jenkins Documentation: https://www.jenkins.io/doc/book/security/access-control/
- Matrix Authorization Strategy Plugin: https://plugins.jenkins.io/matrix-auth/
