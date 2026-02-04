## Day 28: Git Cherry Pick

### The Nautilus application development team has been working on a project repository /opt/demo.git. This repo is cloned at /usr/src/kodekloudrepos on storage server in Stratos DC. They recently shared the following requirements with the DevOps team:

### There are two branches in this repository, master and feature. One of the developers is working on the feature branch and their work is still in progress, however they want to merge one of the commits from the feature branch to the master branch, the message for the commit that needs to be merged into master is Update info.txt. Accomplish this task for them, also remember to push your changes eventually.

--- 

## Login into Data storage server
```
ssh natasha@ststor01
```

## Make the user as Sudo User
```
sudo su -
```

## Navigate to given repo path
```
cd /usr/src/kodekloudrepos
```

## check the available files
```
ls
```

## Navigate to given repo( as ls command gives demo file as result)
```
cd demo
```

## check available branches( make sure we are in feature branch or whatever mentioned branch in given task)
```
git branch
```

## Check git logs
```
git log --oneline
```

## Copy the commit ID fo that mentioned commit in given scenario

## Checkout to the Master branch
```
git checkout master
```

## check available branches( make sure we are in master branch)
```
git branch
```

## Merge the mentioned commit or which is copied ( By using the cherry pick action) 
```
git cherry-pick c1e219c
```

## Check the git status (to verify the untracked files)
```
git status
```

## Check the git status (to verify the untracked files)
```
git status
```

## Push the code to master(already in master branch)
```
git push
```

## check the available files( verify the newly added file availability)
```
ls
```

## Check git logs
```
git log --oneline
```
