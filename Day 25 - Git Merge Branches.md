## Day 25 - Git Merge Branches

### The Nautilus application development team has been working on a project repository `/opt/cluster.git`. This repo is cloned at `/usr/src/kodekloudrepos` on storage server in Stratos DC. They recently shared the following requirements with DevOps team:

### Create a new branch datacenter in `/usr/src/kodekloudrepos/cluster` repo from master and copy the `/tmp/index.html` file (present on storage server itself) into the repo. Further, add/commit this file in the new branch and merge back that branch into `master` branch. 
 
### Finally, push the changes to the origin for both of the branches.
---
## Login into Data storage server
```
ssh natasha@ststor01
```

## Make the user as Sudo User
```
sudo su-
```

## if sudo not work the install sudo ( will make user as root automatically )
```
sudo -i
```

## Navigate to given repo
```
cd /usr/src/kodekloudrepos/cluster
```

## Check the branch list(* mark will show the branch where we are)
```
git branch --list
```

## Check available files and permission
```
ls -l
```

## create new branch as given in the scenario
```
git checkout -b datacenter
```

## Check the branch list(* mark will show the branch where we are)
```
git branch
```
## copy the given file from the given path to the place wher we are ( into the created branch)
```
cp /tmp/index.html .
```

## Check available files and permission( we can verify that the copied file also available here)
```
ls -l
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

## Checkout to the parent branch (to merege)
```
git checkout master
```

## Check available files and permission( we can verify available files here)
```
ls -l
```

## Merge the child branch into parent branch
```
git merge datacenter master
```

## Check available files and permission( we can verify that the available files here - index file which was earlier on child branch now in this paranet branch also)
```
ls -l
```

## Push the code to the origin( to master branch in repo)
```
git push
```
