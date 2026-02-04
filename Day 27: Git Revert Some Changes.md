## Day 27: Git Revert Some Changes

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
