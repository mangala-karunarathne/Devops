## Day 15 - Setup SSL for Nginx

#### The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 1 in Stratos Datacenter. They have some pre-requisites to get ready that server for application deployment. Prepare the server as per requirements shared below: Install and configure nginx on App Server 1. On App Server 1 there is a self-signed SSL certificate and key present at location /tmp/nautilus.crt and /tmp/nautilus.key. Move them to some appropriate location and deploy the same in Nginx. Create an index.html file with content Welcome! under Nginx document root. 4. For final testing try to access the nginx server link (either hostname or IP) from jump host using curl command. For example

## Login into Appserver 01
```
ssh tony@stapp01
```

## Make it as root user
```
sudo su -
```

## Check Nginx service's status 
```
systemctl status nginx
```

## Check OS ( As installations may differ from Os to OS ) 
```
cat /etc/os-release
```

## Install Nginix
```
sudo yum install nginx -y
```

## Check Nginx status
```
systemctl status nginx
```

## Start Nginx
```
systemctl start nginx
```

## Check Nginx status
```
systemctl status nginx
```

## Go to `/tmp` directory and check available docs 
```
cd /tmp 
```
```
ls -l
```

## `cd` and `ls` the below directories and move sertificate and key to there ( common locations wher TLS certifications and keys handles )
```
cd  /etc/pki/tls/certs
```

```
ls -l
```

```
cd /etc/pki/tls/private/
```

```
ls -l 
```
## Move certification 

```
mv /tmp/nautilus.crt /etc/pki/tls/certs  
```

## Move Key
```
mv /tmp/nautilus.key /etc/pki/tls/private/
```

## Check the read write access

```
ls -l /etc/pki/tls/certs/  | grep "nautilus.crt"
```

```
ls -l /etc/pki/tls/private/ | grep "nautilus.key"
```
## Just check the nginx config path and its available docs ( check weather config files available )
#### When we use path as `/` before the path means it refers the path from root level 

```
cd /etc/nginx
```

```
ls -l
```

## Get into nginx config file to edit

```
vi nginx.conf
```
## Just press `i` to start edit 

## Edit `server_name` with appserver01 ip `server_name 172.18.138.10`

## Edit same files TLS settings as follows by uncommenting and updating the paths required

## Updated Settings for a TLS enabled server should look like follows. (Make sure to unjcomment them as required)

```
server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  172.16.238.10;
    root         /usr/share/nginx/html;

    ssl_certificate     "/etc/pki/tls/certs/nautilus.crt";
    ssl_certificate_key "/etc/pki/tls/private/nautilus.key";
    ssl_session_cache   shared:SSL:1m;
    ssl_session_timeout 10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}

```

## After done editing just press `Esc` key and enter below to exit from vim ( it saves if eddited any )

`:x`

## Restart nginx

```
systemctl restart nginx
```

## Get into html file location and `ls` to see available docs
```
cd /usr/share/nginx/html
```
```
ls -l
```

## Just to read the content)

```
cat index.html
```

## Remove the file(html) content 

```
rm -rf index.html
```

## Get into `index.html` file to update as given

```
vi index.html
```

## just press `i` to edit the file 

## Copy the content to add ther form question and escape by pressing `Esc` key

## Save the changes done and Exit 

```
:x
```
## Test for any issue

```
nginx -t 
```
## Go to jump host ( jsut open a terminal )

## Check wheather the API request to appserver is working or not
```
curl -Ik https://stapp01
```
#### should get status as 200

```
curl -k https://stapp01
```
#### should get status as 200

