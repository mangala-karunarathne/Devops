## Scenario 

### Nautilus developers are actively working on one of the project repositories, `/usr/src/kodekloudrepos/demo`. Recently, they decided to implement some new features in the application, and they want to  maintain those new changes in a separate branch. Below are the requirements that have been shared with the DevOps team:


### On Storage server in Stratos DC create a new branch xfusioncorp_demo from master branch in `/usr/src/kodekloudrepos/demo` git repo.

### Please do not try to make any changes in the code.
---
# Data Storage Server – Git Setup & Branch Creation Notes

## 1. Login to Data Storage Server
```bash
ssh natasha@ststor01
```

## 2. Make the User a Sudo User

### Switch to sudo (if already permitted)
```bash
sudo su -
```

### If sudo does not work, install / switch to root
```bash
sudo -i
```

## 3. Navigate to the Given Repository
```bash
cd /usr/src/kodekloudrepos/demo/
```

## 4. Check Available Files
```bash
ls
```

## 5. Check Existing Git Branches
```bash
git branch --list
```

## 6. Fix Git Safe Directory Issue (If Git Commands Fail)
If Git throws a safe.directory error, run:
```bash
git config --global --add safe.directory /usr/src/kodekloudrepos/apps
```

## 7. Switch to Master Branch
(New branch must be created from master)
```bash
git checkout master
```

## 8. Create a New Branch as per Scenario
```bash
git checkout -b xfusioncorp_cluster
```

## 9. Verify Branch Creation
```bash
git branch --list
```

✅ Branch `xfusioncorp_cluster` successfully created from `master`.
