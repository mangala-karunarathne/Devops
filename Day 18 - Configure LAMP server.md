# Day 18 – Configure LAMP Server
-----
## xFusionCorp Industries is planning to host a WordPress website on their infra in Stratos Datacenter. They have already done infrastructure configuration—for example, on the storage server they already have a shared directory /var/www/html that is mounted on each app host under /var/www/html directory. Please perform the following steps to accomplish the task:
a. Install httpd, php and its dependencies on all app hosts.
b. Apache should serve on port 8085 within the apps.
c. Install/Configure MariaDB server on DB Server.
d. Create a database named kodekloud_db8 and create a database user named kodekloud_gem identified as password TmPcZitRtx. Further make sure this newly created user is able to perform all operations on the database you created.
Finally, you should be able to access the website on LBR link by clicking on the App button on the top bar. You should see a message like:
"WordPress site running on kodekloud_app1"


## Click on the given app url to check wheather it's up and running now. As they told now it should not work. 
---

## Log into appserver01
```
ssh tony@stapp01
```
## make the user as sudo user
```
sudo su -
```

## install php(new and recomended than yum install)
```
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
```

## Install apache service 
```
yum install httpd -y
```

## Enable apache, php then restart those services and check status ( all steps in one single command)
```
systemctl enable httpd php; systemctl restart httpd php; systemctl status httpd php
```

## Just go to the config file to edit port as reqested
```
vi /etc/httpd/conf/httpd.conf
```
## press i to edit

## Just check the Listen keyword and edit the port as reqested

## press Esc to escape

## press ;x to save the updaed changes

## Restart the httpd service
```
systemctl restart httpd
```
---

## Open another termainal and log into appserver02
```
ssh steve@stapp02
```

## Do all the stuff as same as did in appserver01
```
sudo su -
```
---
## Open another termainal and log into appserver03
```
ssh steve@stapp02
```

## Do all the stuff as same as did in appserver02
```
sudo su -
```
---

## Open another termainal and log into database server
```
ssh peter@stdb01
```

## make the user as sudo user
```
sudo su -
```

## Check OS in database server
```
cat /etc/os-release
```

## if the databaseOS is centOS then install mariadb as follows
```
yum install mariadb-server mariadb -y
```

## Enable maria db
```
systemctl enable mariadb
```

## check mariadb status
```
systemctl status mariadb
```

## just restart mariadb
```
systemctl restart mariadb
```

## check mariadb status(just to verify and should have active and running status)
```
systemctl status mariadb
```

## Check the avaiable databasescheck all db users with the allowed host to connect
```
SELECT user, host FROM mysql.user;
```

## create database with given name in the test
```
CREATE DATABASE kodekloud_db6;
```

## Check the avaiable databases
```
SHOW DATABASES;
```

## Create a databases user with the given name
```
CREATE USER 'kodekloud_aim'@'%' IDENTIFIED BY 'Rc5C9EyvbU';
```

## grant all privilages to the created db user
```
GRANT ALL PRIVILEGES ON kodekloud_db6.* TO 'kodekloud_aim'@'%';
```

## Apply changes
```
FLUSH PRIVILEGES;
```

## check all db users with the allowed host to connect
```
SELECT user, host FROM mysql.user;
```

## check all created user privilages to verify given permisions to the created  db user
```
SHOW GRANTS FOR 'kodekloud_aim'@'%';;
```

## Click on the given app url to check wheather it's up and running now
