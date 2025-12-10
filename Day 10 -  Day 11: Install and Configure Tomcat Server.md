## The Nautilus application development team recently finished the beta version of one of their Java-based applications, which they are planning to deploy on one of the app servers in Stratos DC. After an internal team meeting, they have decided to use the tomcat application server. Based on the requirements mentioned below complete the task:

### a. Install tomcat server on App Server 2.

### b. Configure it to run on port 8084.

###  c. There is a ROOT.war file on Jump host at location /tmp.


### Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e curl http://stapp02:8084
### Login As steve
`ssh steve@stapp02`

### check OS
`cat /etc/os-release`

### 1. Install java jdk using yum
`sudo yum install -y java-17-openjdk`

### 2. Install Tomcat using yum
`sudo yum install tomcat tomcat-webapps tomcat-admin-webapps tomcat-docs-webapp tomcat-jsvc -y`

### 3. Enable and start Tomcat service
`sudo systemctl enable tomcat`
`sudo systemctl start tomcat`
`sudo systemctl status tomcat`

### 4. Change Tomcat default port
`sudo vi /etc/tomcat/server.xml`
`i`
```
<Connector port="3004" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```
`:wq`

### 5. Restart Tomcat
`sudo systemctl restart tomcat`

### 6. On Jump Host: Restart Tomcat
`scp /tmp/ROOT.war your_user@stapp03:/tmp/`

### On App Server 3: Move to tomcat webapps folder
`sudo mv /tmp/ROOT.war /usr/share/tomcat/webapps/`

### Verify Deployment
`curl http://stapp03:3004`
