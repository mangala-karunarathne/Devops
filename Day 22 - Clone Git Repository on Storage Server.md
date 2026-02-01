# Clone a Git Repository on the Storage Server

This note documents the steps to clone an unused Git repository from a local path on the **Storage Server (Stratos DC)** without making any modifications to the repository.

---

## Scenario
### The DevOps team established a new Git repository last week, which remains unused at present. However, the Nautilus application development team now requires a copy of this repository on the Storage Server in the Stratos DC. Follow the provided details to clone the repository:


### The repository to be cloned is located at `/opt/apps.git`


### Clone this Git repository to the `/usr/src/kodekloudrepos directory.` Ensure no modifications are made to the repository during the cloning process.

## Objective

* Clone the existing Git repository located at:

  ```
  /opt/apps.git
  ```
* Destination directory:

  ```
  /usr/src/kodekloudrepos
  ```
* Ensure the repository is **only cloned** (no changes, commits, or modifications).

---

## Step-by-Step Instructions

### 1. Login to the Storage Server

```bash
ssh natasha@ststor01
```

---

### 2. Switch to Root User (if required)

```bash
sudo su -
```

---

### 3. Switch Back to `natasha` User

(Required if repository ownership or permissions expect this user)

```bash
su - natasha
```

---

### 4. Navigate to the Destination Directory

```bash
cd /usr/src/kodekloudrepos
```

---

### 5. Check Existing Files

```bash
ls -l
```

---

### 6. Clone the Repository

```bash
git clone /opt/apps.git
```

---

### 7. (Optional) Fix Git Ownership / Safety Issue

If you encounter a *safe directory* or ownership-related error, run:

```bash
git config --global --add safe.directory /opt/apps.git
```

Then retry cloning:

```bash
git clone /opt/apps.git
```

---

### 8. Verify Cloned Repository

List all files including hidden ones:

```bash
ls -la
```

Navigate into the cloned repository:

```bash
cd apps
```

---

### 9. Verify Git Repository Status

Check Git logs:

```bash
git log
```

> Note: If the repository has no commits yet, the directory may appear empty and `git log` may not show any entries. This is expected behavior.

---

## Notes

* No changes were made to the repository during this process.
* This procedure strictly performs a **read-only clone**.
* Useful for initializing development or deployment workflows later.

---

✅ **Clone completed and verified successfully**
