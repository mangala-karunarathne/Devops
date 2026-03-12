# Jenkins Task: Install Packages via SSH
## Original Task Requirements

Some new requirements have come up to install and configure some packages on the Nautilus infrastructure under Stratos Datacenter. The Nautilus DevOps team installed and configured a new Jenkins server so they wanted to create a Jenkins job to automate this task.

1. Access the Jenkins UI by clicking on the `Jenkins` button in the top bar. Log in using the credentials: username `admin` and password `Adm!n321`.
2. Create a new Jenkins job named `install-packages` and configure it with the following specifications:
   - Add a string parameter named `PACKAGE`.
   - Configure the job to install a package specified in the `$PACKAGE` parameter on the `storage server` (Stratos Datacenter).
   - Build the job at least once (e.g. with parameter `PACKAGE=vim-enhanced`) so the package is installed on the Storage server and can be verified.

**Notes:**
- Ensure to install any required plugins and restart the Jenkins service if necessary. Opt for `Restart Jenkins when installation is complete and no jobs are running` on the plugin installation/update page.
- Verify that the Jenkins job runs successfully on repeated executions to ensure reliability.

---

## Steps Followed to Complete the Task

---

## 1. Login to Jenkins
- URL: Jenkins button in top bar
- Username: `admin`
- Password: `Adm!n321`

---

## 2. Install "Publish Over SSH" Plugin
1. Go to **Manage Jenkins** → **Plugins** → **Available plugins**
2. Search for **Publish Over SSH**
3. Check the box → Click **Install**
4. Select **"Restart Jenkins when installation is complete and no jobs are running"**
5. Refresh the page and log back in

---

## 3. Configure the Storage Server SSH Connection
1. Go to **Manage Jenkins** → **System**
2. Scroll to **Publish Over SSH** section
3. Click **Add** under SSH Servers and fill in:

```
Name:        storage-server
Hostname:    ststor01.stratos.xfusioncorp.com
Username:    natasha
Remote Dir:  /tmp
```

4. Click **Advanced** → check **"Use password authentication"** → enter password
5. Click **Test Configuration** → should return **Success**
6. Click **Save**

---

## 4. Create the Jenkins Job
1. From dashboard → Click **"New Item"**
2. Name: `install-packages`
3. Select **Freestyle project**
4. Click **OK**

---

## 5. Configure the Job

### Add String Parameter
1. Check **"This project is parameterized"**
2. Click **Add Parameter** → **String Parameter**
3. Fill in:
   - **Name:** `PACKAGE`
   - **Default Value:** *(leave blank)*
   - **Description:** `Package name to install on storage server`

### Add Build Step
1. Scroll to **Build Steps**
2. Click **"Add build step"** → **"Send files or execute commands over SSH"**
3. Configure:
   - **SSH Server Name:** `storage-server` *(from dropdown)*
   - **Exec command:**
```bash
sudo yum install -y $PACKAGE
```
4. Click **Save**

---

## 6. Build the Job
1. Click **"Build with Parameters"**
2. Enter `PACKAGE` value: `vim-enhanced`
3. Click **Build**
4. Click build number → **Console Output**

---

## 7. Successful Console Output
```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/install-packages
SSH: Connecting from host [jenkins.stratos.xfusioncorp.com]
SSH: Connecting with configuration [storage-server] ...
SSH: EXEC: completed after 7,404 ms
SSH: Disconnecting configuration [storage-server] ...
SSH: Transferred 0 file(s)
Build step 'Send files or execute commands over SSH' changed build result to SUCCESS
Finished: SUCCESS
```

---

## Notes
- Running `yum install -y` twice is **not an issue** — it is idempotent (safe to repeat)
- If the package is already installed, yum exits cleanly and the build still shows **SUCCESS**
- Two successful builds also satisfies the **reliability check** requirement
