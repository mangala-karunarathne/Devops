## Day 26 - Git Manage Remotes

### The xFusionCorp development team added updates to the project that is maintained under /opt/media.git repo and cloned under /usr/src/kodekloudrepos/media. Recently some changes were made on Git server that is hosted on Storage server in Stratos DC. The DevOps team added some new Git remotes, so we need to update remote on /usr/src/kodekloudrepos/media repository as per details mentioned below:

### a. In /usr/src/kodekloudrepos/media repo add a new remote dev_media and point it to /opt/xfusioncorp_media.git repository.
### b. There is a file /tmp/index.html on same server; copy this file to the repo and add/commit to master branch.
### c. Finally push master branch to this new remote origin.
---
## Login into Data storage server
```
ssh natasha@ststor01
```
## Make the user as Sudo User
```
sudo su -
```
## Navigate to given repo
```
cd /usr/src/kodekloudrepos/media
```
## Check the branch list(* mark will show the branch where we are)
```
git branch --list
```
## Check available remote origins
```
git remote
```

## Just to check the remote repositeries connected to local repositary
```
git remote -v
```
## Check the files and permision
```
ls -l
```
## Add new remote named as given and locate in the given path 
```
git remote add dev_media /opt/xfusioncorp_media.git
```
## Check available remote origins
```
git remote
```
## Just to check the remote repositeries connected to local repositary
```
git remote -v
```
## Check the files and permision
```
ls -l
```
## Check the git configuration
```
cat .git/config
```

## copy the given file in to the repo(same repo as we are in now) 
```
cp /tmp/index.html .
```
## Check the git status (to verify the available untracked files)
```
git status
```

## Add the index.html file
```
git add index.html
```

## Check the git status (to verify the tracked files)
```
git status
```
## Add commit message
```
git commit -m "Update index.html file"
```
## Check the git status (to verify the tracked files)
```
git status
```

## Push the file into created new origin
```
git push -u dev_media
```

## Check the git status (to verify that nothing to commit)
```
git status
```
