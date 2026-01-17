login into appserver01
ssh tony@stapp01

make the user as sudo user
sudo su -

Check OS in appserver02 server
cat /etc/os-release

if the appserver02 is centOS then install apache service
yum install httpd -y

Given backups on path /home/thor so that open another terminal as thor user (default)

check the available files (should see both given two folders - games, ecommerce)
ls

redirect into games folder
cd games

check all files & permission
ls -l

Display the contents of the file index.html (as already noticed index file is available there)
cat index.html

get back to root directory
cd ..

redirect into ecommerce folder
cd ecommerce

check all files & permission
ls -l

Display the contents of the file index.html (as already noticed index file is available there)
cat index.html

Securely copy the index.html file via ssh from the /ecommerce/ path to /tmp path ( ~ used to get the absolute path)
scp ~/ecommerce/index.html tony@stapp01:/tmp

Now get back to the terminal which is used to log into appserver02

Go into /tmp directory
cd /tmp

check the available files (should see index.html as we coppied short while ago)
ls

create two folders as ecommerce and games
mkdir ecommerce games

move that index.html file copied from ~/ecommerce to this newly created /temp/ecommerce folder
mv index.html ecommerce/

Just open the appache config file to edit port as reqested
vi /etc/httpd/conf/httpd.conf

press i to edit

Just check the Listen keyword and edit the port as reqested

press Esc to escape

press ;x to save the updaed changes

Restart the httpd service
systemctl restart httpd

check apache service status
systemctl status httpd

check wheather the URL works or not (should not work at this stage)
curl http://localhost:8083/ecommerce/

Navigate to the web site file directory (/var/www/html is the path for apache service)
cd /var/www/html

check the available files (if nothing availablke means apache wont serve anything for web URL)
ls

create two folders as ecommerce and games( as we are to host two web sites)
mkdir ecommerce games

Navigate to /tmp directory
cd /tmp

Move all content in ecommerce folder(only have index.html as we really knew)
sudo mv /tmp/ecommerce/* /var/www/html/ecommerce/

Navigate to /var/www/html/ecommerce/
cd /var/www/html/ecommerce/

check the available files (copied index.html should be there)
ls


Now get back to the terminal which is used to log into thor user

Securely copy the index.html file via ssh from the /apps/ path to /tmp path ( ~ used to get the absolute path)
scp ~/games/index.html tony@stapp01:/tmp

Now get back to the terminal which is used to log into appserver02

check the available files (copied index.html should be there)
ls

Then move that index.html to the apps directory
sudo mv index.html games

Move all content in ecommerce folder(only have index.html as we really knew)
mv /tmp/games/* /var/www/html/games/

Navigate to /var/www/html/apps/
cd /var/www/html/games/

check the available files (copied index.html should be there)
ls

Navigate into /var/www/html/
cd /var/www/html/

check all files & permission( both permission should set as root if everything correct so far.. tehn only it serves properly when host)
ls -l

change ownership to the apache to serve them for hosting
chown -R apache:apache /var/www/html/

Restart apache service
systemctl restart 

chack the status of apache service
systemctl status httpd

check wheather the URL works or not (should work now)
curl http://localhost:8087/ecommerce/

check wheather the URL works or not (should work now)
curl http://localhost:8087/apps/
