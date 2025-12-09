## Day 5 - Disabling SELinux step by step
### Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 1 in the Stratos Datacenter: Install the required SELinux packages. Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes. No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight. Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.

## Summary of what we’ll do
### 1. Check the OS so we run the right package commands.
### 2. Install SELinux packages (so the utilities and policies are present).
### 3. Permanently disable SELinux by editing the config file (so after the reboot it will be disabled).
### 4. (Optional) show how to immediately disable it for the current boot (explained, but you don’t need it because you said to disregard CLI status).
### 5. Verify current state and show the commands to check after the scheduled reboot.

## 0) Connect to App server 1
```
ssh steve@172.16.238.10
```
## 1) Detect OS / package manager (important so we use the right install commands)
```
cat /etc/os-release
uname -r
```
### If you see ID="centos" or ID="rhel" or ID="fedora" use yum or dnf.

### If you see ID="ubuntu" or ID="debian" use apt.

### (You only need to run these once to know which branch below to follow.)

## 2) Install SELinux packages
### For RHEL / CentOS / Rocky / Alma (yum/dnf)
```
# update package metadata
sudo yum makecache fast   # or: sudo dnf makecache

# install typical SELinux utilities and targeted policy
sudo yum install -y policycoreutils selinux-policy-targeted selinux-policy

# optional helpful tools
sudo yum install -y setools-console setroubleshoot
```

### For Debian/Ubuntu (apt)
```
sudo apt update
sudo apt install -y selinux-basics selinux-utils selinux-policy-default
```
## 3) Permanently disable SELinux (this makes the change survive the reboot)
```
sudo cp /etc/selinux/config /etc/selinux/config.bak.$(date +%F_%T)
```

### Then change SELINUX= to disabled:
```
# safe inline edit
sudo sed -i 's/^\s*SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

# show the file to confirm
grep -i '^SELINUX=' /etc/selinux/config && cat /etc/selinux/config | sed -n '1,20p'
```
### Why this is sufficient: /etc/selinux/config controls the persistent SELinux mode at boot. Setting SELINUX=disabled means after the next reboot the kernel will start with SELinux disabled.

## 6) Verify installed packages and config (run now)
### Check packages:
```
# RHEL-like
rpm -qa | grep -i selinux || echo "No rpm packages found matching selinux"

# Debian-like
dpkg -l | grep -i selinux || echo "No dpkg packages found matching selinux"
```
### Check config:
```
grep -i '^SELINUX=' /etc/selinux/config
# should show: SELINUX=disabled
```
