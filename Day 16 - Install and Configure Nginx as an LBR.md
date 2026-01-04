# Nginx Load Balancer Configuration Guide

## Login to Load Balancer Server (LBR)

```bash
ssh loki@stlb01
```

## Switch to **root user**

```bash
sudo su -
```

## Check **Nginx** status

```bash
systemctl status nginx
```

## Check **Operating System** details

```bash
cat /etc/os-release
```

## Install **Nginx**

```bash
yum install nginx -y
```

## Verify **Nginx** status after installation

```bash
systemctl status nginx
```

## Restart **Nginx** service

```bash
systemctl restart nginx
```

## Check **Nginx** status again

```bash
systemctl status nginx
```

---

## Login to **App Server 01**

```bash
ssh tonny@stapp01
```

## Check which port **Apache (httpd)** is listening on

```bash
sudo netstat -luntp | grep httpd
```

### If `netstat` is not available (**recommended method**)

```bash
sudo ss -luntp | grep httpd
```

## Check **Apache service** status

```bash
systemctl status httpd
```

---

## Login to **App Server 02**

```bash
ssh steve@stapp02
```

## Check **Apache listening port**

```bash
sudo ss -luntp | grep httpd
```

---

## Return to **Load Balancer Server (LBR)**

## Edit **Nginx configuration file**

```bash
vi /etc/nginx/nginx.conf
```

➡️ Press **`i`** to enter **insert mode**
📘 **Reference:** https://nginx.org/en/docs/http/load_balancing.html

---

## Configure **Nginx as Load Balancer**

```nginx
upstream appservers {
    server stapp01:8087;
    server stapp02:8087;
    server stapp03:8087;
}

server {
    listen 80;

    location / {
        proxy_pass http://appservers;
    }
}
```

---

## Save and exit the file

➡️ Press **`Esc`**, then type:

```bash
:x
```

---

## Test **Nginx configuration**

```bash
nginx -t
```

## Restart **Nginx** after configuration change

```bash
systemctl restart nginx
```

## Check final **Nginx** status

```bash
systemctl status nginx
```

---

## ✅ Verification

If **Nginx status is `active (running)`**, the **Load Balancer configuration is successful**.  
Open the **given website URL in the lab** to verify load balancing.
