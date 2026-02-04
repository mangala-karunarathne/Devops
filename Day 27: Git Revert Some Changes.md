## Day 27: Git Revert Some Changes

### The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/official` present on Storage server in Stratos DC. However, they reported an issue with the recent commits being pushed to this repo. They have asked the DevOps team to revert repo HEAD to last commit. Below are more details about the task:

### In /usr/src/kodekloudrepos/official git repository, revert the latest commit (HEAD) to the previous commit (JFYI the previous commit hash should be with initial commit message).

### Use revert official message (please use all small letters for commit message) for the new revert commit.
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

## Check the available files and permision
```
ls -l
```

## just check the git logs to make sure the logs on comiit history
```
git log
```

## Reveert the latest commit( Head alway point the latest commit)
```
git revert HEAD
```

## Enter

## :x

## Escape

## just check the git logs to make sure the logs on comiit history(Revert commit also should visible there)
```
git log
```

## Check the git status (to verify the untracked files)
```
git status
```
## In status we can see the untracked file. Add that file
```
git add <index.html or whatever file name>
```

## Check the git status (to verify the tracked files)
```
git status
```

## Add commit message( make sure to add same given commit message as scenario asked to do so)
```
git commit -m "<whatever givin commit message >"
```

## just check the git logs to make sure the logs on comiit history(Revert commit also should visible there)
```
git log
```

## it doesn't ask anything to push.. So no need to push...
