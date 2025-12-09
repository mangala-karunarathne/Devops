## Day1: Create User With Shell
### 1. Connect to App Server 2
```
ssh mangala@stapp02
```
### 2. Switch to root (if needed)
```
sudo -i
```
### 3. Create user john with a non-interactive shell
```
useradd -s /sbin/nologin john
```

### (If /sbin/nologin doesn't exist, use /usr/sbin/nologin)

### 4. Verify the user was created
grep john /etc/passwd


### ✅ Expected output:

`john:x:1001:1001::/home/john:/sbin/nologin`
