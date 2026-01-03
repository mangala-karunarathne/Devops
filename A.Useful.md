### App Server 1 - tony
```
ssh tony@stapp01
```
```
Ir0nM@n
```
### App Server 2 - steve
```
ssh steve@stapp02
```
```
Am3ric@
```

### App Server 3 - banner
```
ssh banner@stapp03
```
```
BigGr33n
```

### jump_host Server - thor
```
ssh thor@jump_host
```
```
mjolnir123
```
### ststor01 Server - natasha
```
ssh natasha@ststor01
```
```
Bl@kW
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
## IP List
````
| Hostname    | IP Address       |
|------------|------------------|
| stapp01    | 172.16.238.10    |
| stapp02    | 172.16.238.11    |
| stapp03    | 172.16.238.12    |
| stlb01     | 172.16.238.14    |
| stdb01     | 172.16.239.10    |
| ststor01   | 172.16.238.15    |
| stbkp01    | 172.16.238.16    |
| stmail01   | 172.16.238.17    |
| jump_host  | Dynamic          |
| jenkins    | 172.16.238.19    |

````
