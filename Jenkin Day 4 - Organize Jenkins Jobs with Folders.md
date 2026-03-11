# Jenkins – Organize Jobs into Folders

---

## 📋 Original Task

**xFusionCorp Industries** DevOps team aims to streamline the management of Jenkins jobs by organizing them into distinct folders based on their purpose.

**Requirements:**
1. Access the Jenkins UI — login with username `admin` and password `Adm!n321`
2. Create a new folder named `Apache` within the Jenkins UI
3. Move the existing jobs `httpd-php` and `services` under the newly created `Apache` folder

> **Note:** Install any required plugins and restart Jenkins if necessary. Select _"Restart Jenkins when installation is complete and no jobs are running"_ on the plugin page.

---

## 🪜 Steps to Complete

### 1. Login
- URL: `http://<host>:8080`
- Username: `admin` | Password: `Adm!n321`

### 2. Install Folders Plugin
1. Manage Jenkins → Manage Plugins → **Available** tab
2. Search: `CloudBees Folders` → Check it → Click **Install**
3. Select **"Restart Jenkins when installation is complete and no jobs are running"**
4. Refresh and log back in

### 3. Create a Folder
1. Dashboard → **New Item**
2. Enter name: `Apache`
3. Select **Folder** → Click **OK** → **Save**

### 4. Move Jobs into the Folder
Repeat for both `httpd-php` and `services`:

1. Click the job name from the Dashboard
2. Left sidebar → Click **Move**
3. Select **Apache** from the dropdown
4. Click **Move**

### 5. Verify
- Go to Dashboard → Click **Apache** folder
- Both `httpd-php` and `services` should be listed inside ✅

---

## ✅ Completed Screenshot

![Jenkins Apache Folder - Completed](./completed_screenshot.png)

> Both `httpd-php` and `services` jobs successfully moved into the `Apache` folder.
