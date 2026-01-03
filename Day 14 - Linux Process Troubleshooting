## Day 14 - Linux Process Troubleshooting
#### The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC. Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not have hosted any code yet on these servers, so you don’t need to worry if Apache isn’t serving any pages. Just make sure the service is up and running. Also, make sure Apache is running on port 8083 on all app servers.

## Make sure to check all the ap servers and verify which is not running the apache service. Then do the following in which is not running apache service...
## 1. Check Apache service status
```
systemctl status httpd
```
## 2. Check 8083 port stattus like what is runnig on that port
```
netstat -tlnp | grep :8083
```
## 3. Kill if anything running there ( by mention9ing the PID )
```
kill <PID>
```
## 4. Restart apache service 
```
systemctl restart httpd
```
## 5. Restart apache service 
```
systemctl restart httpd
```
## 6. Check Apache service status
```
systemctl status httpd
```
## 7. Check wheather web page is loading or not
```
curl http://appserver01:8083
```


