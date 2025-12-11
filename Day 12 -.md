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
## 2. Block port 6300 for everyone except LBR host
### Assume LBR host IP: Usually in KodeKloud tasks, LBR host is load balancer, example IP: 172.16.238.14
(Use the correct LBR IP if different.)
```
sudo iptables -A INPUT -p tcp --dport 6300 -s 172.16.238.14 -j ACCEPT
```
```
sudo iptables -A INPUT -p tcp --dport 6300 -j DROP
```
## 3. Make rules persistent (survive reboot)
```
sudo service iptables save
```
### Verify they are loaded on reboot:

```
sudo systemctl restart iptables
```
```
sudo iptables -L -n
```
### You should see:

`ACCEPT  tcp  --  172.16.238.14  0.0.0.0/0  tcp dpt:6300`

`DROP    tcp  --  0.0.0.0/0      0.0.0.0/0  tcp dpt:6300`
