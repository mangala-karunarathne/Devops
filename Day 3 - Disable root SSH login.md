## Day 3 - Disable root SSH login
### Following security audits, the xFusionCorp Industries security team has rolled out new protocols, including the restriction of direct root SSH login. Your task is to disable direct SSH root login on all app servers within the Stratos Datacenter.
## 1.Open the SSH config file:
```
sudo nano /etc/ssh/sshd_config
```

## 2. Look for this line:
`PermitRootLogin yes`

## 3. Change it to:
```
PermitRootLogin no
```

## 4. Save the file and exit.

## 5.Restart SSH to apply changes:
```
sudo systemctl restart sshd
```
### ✅ Now, root cannot login directly over SSH.
### Optional check:
```
ssh root@your_server_ip
```
### It should fail.
