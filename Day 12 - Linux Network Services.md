## Day 12 - Linux Network Services.md
### Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port 8082 (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue. Use tools like telnet, netstat, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings. Once fixed, you can test the same using command curl http://stapp01:8082 command from jump host. Note: Please do not try to alter the existing index.html code, as it will lead to task failure.`curl -v http://stapp01:3000/`

`telnet stapp01 3000`

`ssh tony@stapp01`
`sudo systemctl status httpd`
`sudo journalctl -u httpd -n 200 --no-pager`
`sudo apachectl configtest`
`sudo netstat -tulpn | grep :3000`
`systemctl stop sendmail`
`sudo -i`
`systemctl stop sendmail`
`systemctl disable sendmail`

`systemctl restart httpd`

`sudo netstat -tulpn | grep :3003`

`ip a`

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



