## We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. Our security team has raised a concern that right now Apache’s port i.e 6300 is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:

### 1. Install iptables and all its dependencies on each app host.
### 2. Block incoming port 6300 on all apps for everyone except for LBR host.
### 3. Make sure the rules remain, even after system reboot.

## 1. Install iptables and dependencies [On each app server, run:]
```
sudo yum install -y iptables iptables-services
```
## Enable and start iptables service:
```
sudo systemctl enable iptables
```
```
sudo systemctl start iptables
```
### Check ports listening on the Load Balancer server (General)
```
Check ports listening on the Load Balancer server (General)
```

## 2. Block port 6300 for everyone except LBR host
### Assume LBR host IP: Usually in KodeKloud tasks, LBR host is load balancer, example IP: 172.16.238.14
(Use the correct LBR IP if different. 8088 for 3rd try as it changes time to time- scenario port changed.. lol)

## Add firewall rules (ORDER MATTERS)
### Allow established connections
```
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```
### Allow SSH (important)
```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```
### Allow Apache ONLY from Load Balancer
```
sudo iptables -A INPUT -p tcp -s 172.16.238.14 --dport 3004 -j ACCEPT
```
### Block Apache port for everyone else
```
sudo iptables -A INPUT -p tcp --dport 3004 -j DROP
```
### 3. Make rules persistent (survive reboot)
```
sudo service iptables save
```
### Verify they are loaded on reboot:

```
sudo systemctl restart iptables
```

```
sudo iptables -L -n --line-numbers
```
### Verify Apache is running and listening
```
sudo systemctl status httpd
```

```
sudo netstat -tulnp | grep 3004
```
### If not LISTENING, restart Apache:

```
sudo systemctl restart httpd
```

```
sudo systemctl enable httpd
```
### Test port reachability from LB
### From LB host:
```
telnet 172.16.238.x 3004
```
### Test HTTP response
### From LB host:
```
curl -I http://172.16.238.x:3004
```

### Verify network connectivity

### Ping test from LB host to App Server 1:

```
ping 172.16.238.x
```
