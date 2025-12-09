## Day 4 - Grant executable permissions
### In a bid to automate backup processes, the xFusionCorp Industries sysadmin team has developed a new bash script named xfusioncorp.sh. While the script has been distributed to all necessary servers, it lacks executable permissions on App Server 2 within the Stratos Datacenter. Your task is to grant executable permissions to the `/tmp/xfusioncorp.sh` script on App Server 2. Additionally, ensure that all users have the capability to execute it.

## Step 1: Connect to App Server 2
```
ssh tony@stapp02
```
## Step 2: Check if the file exists

### Now verify that the file /tmp/xfusioncorp.sh exists.

```
ls -l /tmp/xfusioncorp.sh
```

### You might see something like below
`-rw-r--r-- 1 root root  120 Oct 19 06:40 /tmp/xfusioncorp.sh`

### Step 3: Give execute permissions

### We want all users to be able to run it — that means read + execute for everyone.
```
sudo chmod a+x /tmp/xfusioncorp.sh
```
### Breakdown:

### chmod → change mode (permissions)

### a → all users (owner + group + others)

### +x → add execute permission

## Step 4: Verify permissions
```
ls -l /tmp/xfusioncorp.sh
```

### You should now see like below
`-rwxr-xr-x 1 root root 120 Oct 19 06:40 /tmp/xfusioncorp.sh`

## Step 5: (Optional) Test if it runs

### Just to be sure, you can execute it:
```
/tmp/xfusioncorp.sh
```
### If it runs without permission denied errors, you're done!
