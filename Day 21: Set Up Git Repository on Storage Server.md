## Git Bare Repository Setup on Storage Server (Stratos DC)

---
##📘 Scenario

### The Nautilus development team has provided requirements to the DevOps team for a new application development project.
As part of the setup, they requested the establishment of a Git repository on the Storage Server within the Stratos DC environment.
The task involves installing Git using yum, creating a bare Git repository with a specific name and location, and verifying the setup to ensure it is ready for use by development teams.
---

## 📋 Requirements

Install Git on the Storage Server using yum
Create a bare Git repository
Repository path must be exactly:
/opt/beta.git


## Verify Git installation and repository creation


## 🖥️ Environment Details

Server Name: Storage Server (Stratos DC)
Hostname: ststor01
User: natasha
Privileges Required: Root (sudo)
Git Repository Type: Bare repository

---
## 🚀 Implementation Steps
### 1️⃣ Login to the Storage Server
```
Shellssh natasha@ststor01``Show more lines
```

## 2️⃣ Switch to Root User
```
sudo su -
```

## 3️⃣ Install Git Using yum
```
yum install git -y
```

## 4️⃣ Verify Git Installation
```
git --versionShow more lines
```

### ✅ This confirms that Git has been installed successfully on the server.

## 5️⃣ Create the Bare Git Repository
### Initialize a bare Git repository at the required path:
```
git init --bare /opt/beta.git
```

## ℹ️ Note:
### A bare repository does not have a working tree and is commonly used as a central repository for sharing code via git push and git pull.


## 6️⃣ Verify Repository Creation
```
ls -l /opt/beta.git
```




