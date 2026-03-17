# 🐳 Docker & Dockerfile Study Material
### Task: Fix & Build a Dockerfile on App Server 3 (Stratos DC)

---

## 📋 Original Task

> One of the Nautilus DevOps team members is working to create a `Dockerfile` on **App Server 3** in **Stratos DC**. While working on it she ran into issues in which the docker build is failing and displaying errors. Look into the issue and fix it to build an image as per details mentioned below:
>
> - **a.** The `Dockerfile` is placed on App Server 3 under `/opt/docker` directory.
> - **b.** Fix the issues with this file and make sure it is able to build the image.
> - **c.** Do not change base image, any other valid configuration within Dockerfile, or any of the data been used — for example, `index.html`.

---

## 🗂️ Directory Structure on App Server 3

```
/opt/docker/
├── Dockerfile
├── certs/
│   ├── server.crt
│   └── server.key
└── html/
    └── index.html
```

---

## 📄 Original (Broken) Dockerfile

```dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

RUN cp certs/server.crt /usr/local/apache2/conf/server.crt

RUN cp certs/server.key /usr/local/apache2/conf/server.key

RUN cp html/index.html /usr/local/apache2/htdocs/
```

### ❌ Problems in the Original Dockerfile:
1. `RUN cp` was used instead of `COPY` — local files cannot be copied into the image using shell `cp` during build
2. Missing `CMD` instruction — Apache would never start when a container is launched from this image

---

## ✅ Fixed Dockerfile

```dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/index.html

CMD ["httpd-foreground"]
```

### ✅ Fixes Applied:
| What Changed | From | To | Why |
|---|---|---|---|
| File copy method | `RUN cp` | `COPY` | `COPY` is the correct Dockerfile instruction to bring local files into the image |
| Missing startup command | _(nothing)_ | `CMD ["httpd-foreground"]` | Without this, the container has no process to run and immediately exits |

---

## 📖 Dockerfile Instructions Explained (Line by Line)

### 1. `FROM httpd:2.4.43`
- **What it does:** Pulls the official Apache HTTP Server image (version 2.4.43) from Docker Hub as the base
- **MERN equivalent:** Like `"express": "4.18.2"` in `package.json` — using a pre-built package as your foundation
- **Key point:** You never memorize image names; always check [hub.docker.com](https://hub.docker.com)

---

### 2. `RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf`
- **What it does:** Changes Apache's listening port from `80` to `8080` inside the config file
- **`sed` explained:** A Linux find-and-replace tool. `s/old/new/g` means "substitute old with new, globally"
- **MERN equivalent:**
  ```javascript
  app.listen(8080) // changing the port in Express
  ```

---

### 3. `RUN sed -i '/LoadModule\ ssl_module/s/^#//g' conf/httpd.conf`
### 4. `RUN sed -i '/LoadModule\ socache_shmcb_module/s/^#//g' conf/httpd.conf`
### 5. `RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf`
- **What they do:** Apache ships with SSL disabled by default (lines are commented out with `#`). These commands remove the `#` to enable:
  - SSL module
  - Shared memory cache module (required for SSL sessions)
  - SSL configuration file
- **`s/^#//g` explained:** Find lines starting with `#` and remove the `#`
- **MERN equivalent:**
  ```javascript
  // const helmet = require('helmet')   ← commented out (disabled)
  const helmet = require('helmet')      ← uncommented (enabled)
  app.use(helmet())
  ```

---

### 6. `COPY certs/server.crt /usr/local/apache2/conf/server.crt`
### 7. `COPY certs/server.key /usr/local/apache2/conf/server.key`
- **What they do:** Copy SSL certificate and private key from your local machine INTO the Docker image
- **MERN equivalent:**
  ```javascript
  const https = require('https')
  const fs = require('fs')
  https.createServer({
    cert: fs.readFileSync('./certs/server.crt'),
    key:  fs.readFileSync('./certs/server.key')
  }, app)
  ```
- **Why not `RUN cp`?** `cp` is a Linux shell command that works on files already inside the container. `COPY` is the Dockerfile instruction specifically designed to bring files FROM your local machine INTO the image.

---

### 8. `COPY html/index.html /usr/local/apache2/htdocs/index.html`
- **What it does:** Copies your website's HTML file into Apache's web root (the folder Apache serves files from)
- **MERN equivalent:**
  ```javascript
  // Serving React build output in Express
  app.use(express.static(path.join(__dirname, 'client/build')))
  ```
  `htdocs` in Apache = `client/build` in MERN — both are the folder that serves your frontend

---

### 9. `CMD ["httpd-foreground"]`
- **What it does:** Starts the Apache web server when a container is launched from this image
- **Critical concept:** Docker containers stay alive only as long as a foreground process is running. `httpd-foreground` keeps Apache running in the foreground.
- **MERN equivalent:**
  ```javascript
  app.listen(8080, () => {
    console.log('Server running on port 8080') // 🚀 this starts everything
  })
  ```
- **Without CMD:** The image builds fine, but every container launched from it immediately exits because there is nothing to run.

---

## 🆚 `RUN` vs `CMD` — Key Difference

| | `RUN` | `CMD` |
|---|---|---|
| **When it runs** | During `docker build` | During `docker run` |
| **Purpose** | Sets up the image (install packages, configure files) | Starts the application |
| **How many times** | Once, at build time | Every time a container starts |
| **MERN analogy** | `npm install` + config setup | `node server.js` |

> 💡 **Simple rule:** `RUN` builds the house. `CMD` turns on the lights when you move in.

---

## 🆚 `RUN cp` vs `COPY` — Key Difference

| | `RUN cp` | `COPY` |
|---|---|---|
| **What it copies** | Files already inside the container | Files from your local machine into the container |
| **Use case** | Moving files within the image | Bringing files from build context into image |
| **In this task** | ❌ Wrong — certs and html are on your machine | ✅ Correct |

---

## 🔧 Commands Used in This Task

### Connect to App Server 3
```bash
ssh banner@stapp03
sudo su -
```

### Inspect the Dockerfile
```bash
cd /opt/docker
cat Dockerfile          # view contents
cat -A Dockerfile       # view with hidden characters ($=end of line)
ls -la /opt/docker/     # list all files
```

### Fix the Dockerfile (heredoc method)
```bash
cat > /opt/docker/Dockerfile << 'EOF'
# ... paste fixed content ...
EOF
```
> **What is EOF?** It stands for "End Of File". It's a shell technique to write multi-line content to a file. Everything between the two `EOF` markers gets written to the file.

### Build the Docker Image
```bash
cd /opt/docker
docker build -t nautilus-image .
```
> The `.` at the end means "look for Dockerfile in the current directory"

### Verify the image was created
```bash
docker images
```

---

## 🗺️ Config File Paths — Not Universal!

The path `/usr/local/apache2/conf/httpd.conf` is specific to the **official Apache Docker image**. The same Apache software has different paths in different environments:

| Environment | Apache Config Path |
|---|---|
| Docker `httpd` image | `/usr/local/apache2/conf/httpd.conf` |
| Ubuntu/Debian server | `/etc/apache2/apache2.conf` |
| CentOS/RHEL server | `/etc/httpd/conf/httpd.conf` |
| Mac (Homebrew) | `/usr/local/etc/httpd/httpd.conf` |

> **How to find the right path:** Check official docs on hub.docker.com, or explore inside the container with `find / -name "httpd.conf"`

---

## 🌐 Every Server Needs a CMD — Real World Examples

```dockerfile
# Node.js / Express
CMD ["node", "server.js"]

# npm start
CMD ["npm", "start"]

# Python Flask
CMD ["python", "app.py"]

# Nginx
CMD ["nginx", "-g", "daemon off;"]

# Apache (this task)
CMD ["httpd-foreground"]

# MongoDB
CMD ["mongod"]
```

> 💡 **Rule of thumb:** Whatever you type in your terminal to start your server → that becomes your CMD in Dockerfile

---

## 🏗️ What This Dockerfile Builds (Big Picture)

This Dockerfile creates a **containerized Apache web server** that:
- Runs on **port 8080** (not the default 80)
- Has **HTTPS/SSL enabled** using provided certificates
- Serves a **static HTML file** (`index.html`)

In MERN terms, it's equivalent to an Express server that:
```javascript
const https = require('https')
const express = require('express')
const app = express()

app.use(express.static(path.join(__dirname, 'html')))

https.createServer({
  cert: fs.readFileSync('./certs/server.crt'),
  key:  fs.readFileSync('./certs/server.key')
}, app).listen(8080)
```

---

## 📊 MERN vs Dockerfile — Full Comparison

| Dockerfile Instruction | MERN / Node Equivalent |
|---|---|
| `FROM httpd:2.4.43` | `"express": "4.18.2"` in package.json |
| `RUN sed` (port change) | `app.listen(8080)` |
| `RUN sed` (uncomment SSL) | `app.use(helmet())` |
| `COPY certs/` | `fs.readFileSync('./certs/server.crt')` |
| `COPY html/index.html` | `express.static('client/build')` |
| `CMD ["httpd-foreground"]` | `app.listen()` at bottom of server.js |

---

## 💡 What You Need to Remember vs Google

### ✅ Remember (only ~10 core instructions):
```
FROM      → base image
RUN       → execute command during build
COPY      → bring local files into image
CMD       → start the app at runtime
EXPOSE    → declare the port
ENV       → set environment variables
WORKDIR   → set working directory
ARG       → build-time variables
ENTRYPOINT → like CMD but harder to override
VOLUME    → mount points for persistent data
```

### 🔍 Always Google:
- `sed`/`awk` syntax
- Specific config file paths
- Apache/Nginx specific settings
- Image names and versions on Docker Hub

> 💡 **The real skill is not memorizing commands — it's knowing what to Google and understanding what you find!**

---

*Study material generated from a live debugging session on Stratos DC — App Server 3*
