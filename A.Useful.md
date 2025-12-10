### App Server 1 - tony
```
stapp01
```
```
tony
```
```
Ir0nM@n	
```
```
ssh tony@stapp01
```
### App Server 2 - steve

```
stapp02
```
```
steve
```
```
Am3ric@
```
```
ssh steve@stapp02
```
### App Server 3 - banner
```
stapp03
```
```
banner
```
```
BigGr33n
```
```
ssh banner@stapp03
```
### jump_host Server - thor

```
jump_host
```
```
thor
```
```
mjolnir123
```
```
ssh thor@jump_host
```

### Port Running Check:
```
telnet stapp01 8082
```
## Checking Service Status on a Linux System (systemd)

### To check the status of any service on a Linux system that uses systemd, use:

```
sudo systemctl status <service-name>
```
### Common Examples
### Apache (httpd)
```
sudo systemctl status httpd
```
### Nginx
```
sudo systemctl status nginx
```
### MySQL / MariaDB
```
sudo systemctl status mysqld
```
```
sudo systemctl status mariadb
```
### Start the service

```
sudo systemctl start sendmail
```

### Restart it

```
sudo systemctl restart sendmail
```

### Check its status

```
sudo systemctl status sendmail
```
## ip address
### ip address with more details 
```
ip a
```
### Show only IPv4:

```
ip -4 a
```

### Show only IPv6:

```
ip -6 a
```

### Show routing:

```
ip r
```
