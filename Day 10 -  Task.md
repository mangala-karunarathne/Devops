The Nautilus application development team recently finished the beta version of one of their Java-based applications, which they are planning to deploy on one of the app servers in Stratos DC. After an internal team meeting, they have decided to use the tomcat application server. Based on the requirements mentioned below complete the task:



a. Install tomcat server on App Server 2.

b. Configure it to run on port 8084.

c. There is a ROOT.war file on Jump host at location /tmp.


Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e curl http://stapp02:8084

ssh steve@stapp02
sudo yum install -y java-17-openjdk
cat /etc/os-release
wget --spider https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.50/bin/apache-tomcat-10.1.50.tar.gz
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.50/bin/apache-tomcat-10.1.50.tar.gz
sudo wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.50/bin/apache-tomcat-10.1.50.tar.gz
sudo mv apache-tomcat-10.1.50.tar.gz /opt/
ls -ld /opt
cd /opt
sudo tar -xzf apache-tomcat-10.1.50.tar.gz
ls -lh ~
ls -lh /opt
cd ~
sudo tar -xzf apache-tomcat-10.1.50.tar.gz.1
ls
sudo mv apache-tomcat-10.1.50 tomcat
sudo vi /opt/tomcat/conf/server.xml
sudo vi /tomcat/conf/server.xml
sudo vi ~/tomcat/conf/server.xml
edit port into 8084
:wq
sudo cd ~/tomcat/bin
sudo ~/tomcat/bin/startup.sh
sudo netstat -tulpn | grep 8087 
ssh thor@jump_host
ls -l /tmp/ROOT.war
scp /tmp/ROOT.war steve@stapp02:/home/steve/
ssh steve@stapp02
sudo mv ~/ROOT.war ~/tomcat/webapps/
sudo /home/steve/tomcat/bin/startup.sh




