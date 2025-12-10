## Day 12 - Linux Network Services.md
### Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port 8082 (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue. Use tools like telnet, netstat, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings. Once fixed, you can test the same using command curl http://stapp01:8082 command from jump host. Note: Please do not try to alter the existing index.html code, as it will lead to task failure.`curl -v http://stapp01:3000/`

### Coonect to server 
`ssh tony@stapp01`

### Check port : Anything running 
`telnet stapp01 3000`

### Check Server Status
`sudo systemctl status httpd`
### Check Server Logs
`sudo journalctl -u httpd -n 200 --no-pager`
### Test Server Errors
`sudo apachectl configtest`
### Check the port in same server ( to check which process is using port 3000 on your server.)
`sudo netstat -tulpn | grep :3000`
### Stop a service
`systemctl stop sendmail`
### giving root privilage (root user)
`sudo -i`
### Stop a service
`systemctl stop sendmail`
###Desable a service
`systemctl disable sendmail`
### Restart a service
`systemctl restart httpd`
### Check the port in same server ( to check which process is using port 3000 on your server.)
`sudo netstat -tulpn | grep :3003`
### Check the IP of the server
`ip a`
### Allow IP and port to access the server
`sudo iptables -I INPUT -p tcp -s 172.16.238.3 --dport 3000 -j ACCEPT`

### Save and restart iptables
### 
`sudo iptables-save | sudo tee /etc/sysconfig/iptables`
### 
`sudo iptables-save | sudo tee /etc/sysconfig/iptables`
### 
`sudo systemctl restart iptables`

### Allow the IP 172.16.238.3 to access port 3000 on my server.
```
sudo iptables -I INPUT -p tcp -s 172.16.238.3 --dport 3000 -j ACCEPT
```



