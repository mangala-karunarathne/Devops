# Nautilus PHP Application Deployment Notes

This document captures the step-by-step procedure to install and configure **Nginx** with **PHP-FPM 8.2** on **appserver02 (stapp02)** as per Nautilus application requirements.

---

## Scenario Requirements

* Install **nginx** on **appserver02**
* Configure nginx to listen on **port 8095**
* Document root must be **/var/www/html**
* Install **PHP-FPM 8.2**
* PHP-FPM must use unix socket:

  ```
  /var/run/php-fpm/default.sock
  ```
* Configure nginx and PHP-FPM to work together
* Validate from jump host:

  ```
  curl http://stapp02:8095/index.php
  ```

---

## Initial Validation (Jump Host)

> This should **fail initially** because nginx is not yet configured.

```bash
curl http://stapp02:8095/index.php
```

---

## Login to App Server 02

```bash
ssh steve@stapp02
sudo su -
```

Check OS:

```bash
cat /etc/os-release
```

---

## Install and Configure Nginx

### Install nginx (CentOS)

```bash
yum install nginx -y
```

### Update nginx configuration

```bash
vi /etc/nginx/nginx.conf
```

**Existing configuration:**

```nginx
server {
    listen       80;
    listen       [::]:80;
    server_name  _;
    root         /usr/share/nginx/html;
}
```

**Updated configuration:**

```nginx
server {
    listen       8095;
    listen       [::]:8095;
    server_name  _;
    root         /var/www/html;
    index        index.php index.html;
}
```

Save and exit.

### Restart and verify nginx

```bash
systemctl restart nginx
systemctl status nginx
```

---

## PHP Installation (PHP-FPM 8.2)

### Remove default PHP

```bash
yum remove php php-fpm -y
```

### Install PHP 8.2 (CentOS 9 Stream)

```bash
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y
dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
```

Check available PHP modules:

```bash
dnf module list php
```

Install PHP 8.2:

```bash
dnf module install php:remi-8.2 -y
dnf update -y
```

Verify version:

```bash
php -v
```

---

## Configure PHP-FPM

### Update pool configuration

```bash
vi /etc/php-fpm.d/www.conf
```

Ensure the following:

```ini
listen = /var/run/php-fpm/default.sock
user = nginx
group = nginx
```

Save and exit.

---

## Configure Nginx to Use PHP-FPM

Edit nginx config again:

```bash
vi /etc/nginx/nginx.conf
```

**Add PHP location block:**

```nginx
server {
    listen       8095;
    listen       [::]:8095;
    server_name  _;
    root         /var/www/html;
    index        index.php index.html;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Save and exit.

---

## Restart Services

```bash
systemctl restart php-fpm
systemctl restart nginx
```

Verify services:

```bash
systemctl status php-fpm
systemctl status nginx
```

Verify PHP version:

```bash
php -v
```

---

## Validate from Jump Host

```bash
curl http://stapp02:8095/index.php
```

(Other servers should fail if not configured)

```bash
curl http://stapp03:8094/index.php
```

---

## Unix Socket Validation

```bash
ls -l /var/run/php-fpm/default.sock
```

If ownership needs adjustment:

```bash
chown nginx:nginx /var/run/php-fpm/default.sock
```

Re-check:

```bash
ls -l /var/run/php-fpm/default.sock
```

---

## Final Validation

Test nginx configuration:

```bash
nginx -t
```

Final test:

```bash
curl http://stapp02:8095/index.php
```

---

## Notes

* Socket ownership is normally controlled by `www.conf`; manual `chown` is not persistent across restarts
* `+` sign in permissions indicates ACL or SELinux context
* Always ensure nginx and php-fpm users match socket ownership

---

✅ **Setup complete and verified**
