# Docker Study Material — Custom Apache Image on Ubuntu 24.04

---

## 📋 Original Task

> **Nautilus Application Development Team — Docker Image Requirements**
>
> Create a Docker file at `/opt/docker/Dockerfile` (capital `D`) on **App Server 3** in **Stratos DC** and configure it to build an image with the following requirements:
>
> - **a.** Use `ubuntu:24.04` as the base image.
> - **b.** Install `apache2` and configure it to work on port **6200**. *(Do not update any other Apache configuration settings like document root etc.)*

---

## 🗂️ Table of Contents

1. [Key Concepts](#1-key-concepts)
2. [Environment Info — Stratos DC](#2-environment-info--stratos-dc)
3. [Step-by-Step Solution](#3-step-by-step-solution)
4. [The Dockerfile — Explained Line by Line](#4-the-dockerfile--explained-line-by-line)
5. [Apache Port Configuration Deep Dive](#5-apache-port-configuration-deep-dive)
6. [Build & Test Commands](#6-build--test-commands)
7. [Common Errors & Fixes](#7-common-errors--fixes)
8. [Dockerfile Best Practices](#8-dockerfile-best-practices)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## 1. Key Concepts

### What is Docker?
Docker is a platform for building, shipping, and running applications inside **containers** — lightweight, isolated environments that package code and all its dependencies.

### What is a Dockerfile?
A `Dockerfile` is a plain text file containing a set of **instructions** that Docker reads top-to-bottom to **build an image**.

```
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
```

### Image vs Container
| Term | Description |
|------|-------------|
| **Image** | A read-only blueprint (like a class in OOP) |
| **Container** | A running instance of an image (like an object) |

### What is Apache2?
Apache2 (`apache2`) is one of the most widely used open-source **HTTP web servers**. By default it listens on **port 80**, but it can be reconfigured to any port.

---

## 2. Environment Info — Stratos DC

| Server | Hostname | User | Password |
|--------|----------|------|----------|
| App Server 3 | `stapp03` | `banner` | `BigGreen123` |

> **Note:** Always switch to root after logging in: `sudo su -`

---

## 3. Step-by-Step Solution

### Step 1 — SSH into App Server 3

```bash
ssh banner@stapp03
# Enter password: BigGreen123
```

### Step 2 — Switch to Root

```bash
sudo su -
# Enter password when prompted
```

### Step 3 — Create the Target Directory

```bash
mkdir -p /opt/docker
```

> `-p` flag creates parent directories as needed and does **not** error if the directory already exists.

### Step 4 — Create the Dockerfile

```bash
vi /opt/docker/Dockerfile
```

> ⚠️ **Important:** The filename must be `Dockerfile` with a capital **D**. Docker looks for this exact filename by default.

### Step 5 — Write the Dockerfile Content

```dockerfile
# Use ubuntu:24.04 as the base image
FROM ubuntu:24.04

# Install apache2
RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean

# Configure Apache to listen on port 6200
RUN sed -i 's/Listen 80/Listen 6200/' /etc/apache2/ports.conf && \
    sed -i 's/<VirtualHost \*:80>/<VirtualHost *:6200>/' /etc/apache2/sites-enabled/000-default.conf

# Expose port 6200
EXPOSE 6200

# Start Apache in the foreground
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

Save and exit `vi`:
- Press `Esc`
- Type `:wq`
- Press `Enter`

### Step 6 — Verify the File

```bash
cat /opt/docker/Dockerfile
ls -la /opt/docker/
```

---

## 4. The Dockerfile — Explained Line by Line

### `FROM ubuntu:24.04`
- Every Dockerfile **must** start with `FROM`.
- Specifies the **base image** to build upon.
- `ubuntu:24.04` pulls Ubuntu Noble Numbat from Docker Hub.
- Format: `image_name:tag` — if no tag is given, Docker uses `latest`.

### `RUN apt-get update && apt-get install -y apache2 && apt-get clean`
- `RUN` executes a command **during the image build** process.
- `apt-get update` — refreshes the package list.
- `apt-get install -y apache2` — installs Apache2; `-y` auto-confirms prompts.
- `apt-get clean` — removes cached package files to **reduce image size**.
- Chaining with `&&` keeps everything in **one layer** (good practice).

### `RUN sed -i 's/Listen 80/Listen 6200/' /etc/apache2/ports.conf`
- `sed -i` — in-place stream editor (modifies the file directly).
- Changes Apache's listening port from `80` → `6200` in `ports.conf`.

### `RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost *:6200>/' /etc/apache2/sites-enabled/000-default.conf`
- Updates the VirtualHost declaration to match the new port.
- The `\*` escapes the `*` character inside the `sed` pattern.

### `EXPOSE 6200`
- Documents that the container **listens on port 6200** at runtime.
- This is **informational** — it does not actually publish the port.
- To publish, use `-p 6200:6200` in `docker run`.

### `CMD ["apache2ctl", "-D", "FOREGROUND"]`
- Defines the **default command** to run when a container starts.
- Uses **exec form** (JSON array) — preferred over shell form.
- `-D FOREGROUND` keeps Apache running in the foreground so the container doesn't exit immediately.

---

## 5. Apache Port Configuration Deep Dive

### Files Involved in Port Configuration

| File | Purpose |
|------|---------|
| `/etc/apache2/ports.conf` | Defines which ports Apache **listens** on |
| `/etc/apache2/sites-enabled/000-default.conf` | Default VirtualHost — must match the port in `ports.conf` |

### Default Content of `ports.conf`
```apache
Listen 80

<IfModule ssl_module>
    Listen 443
</IfModule>

<IfModule mod_gnutls.c>
    Listen 443
</IfModule>
```

### After `sed` Modification
```apache
Listen 6200      # ← Changed from 80 to 6200

<IfModule ssl_module>
    Listen 443
</IfModule>
...
```

### Why Both Files Need Updating
- `ports.conf` tells Apache **which port to bind to**.
- `000-default.conf` defines the VirtualHost scope — if it still says `*:80`, Apache will try to serve on port 80 instead of 6200, causing a mismatch.

---

## 6. Build & Test Commands

### Build the Image

```bash
docker build -t nautilus-apache /opt/docker/
```

| Flag/Arg | Meaning |
|----------|---------|
| `-t nautilus-apache` | Tags/names the image |
| `/opt/docker/` | Path to the directory containing the Dockerfile |

### Run a Test Container

```bash
docker run -d -p 6200:6200 --name test-apache nautilus-apache
```

| Flag | Meaning |
|------|---------|
| `-d` | Detached mode (runs in background) |
| `-p 6200:6200` | Maps host port 6200 → container port 6200 |
| `--name test-apache` | Names the container |

### Verify the Container is Running

```bash
docker ps
```

### Test the Web Server

```bash
curl http://localhost:6200
```

You should see the default Apache HTML page content.

### View Container Logs

```bash
docker logs test-apache
```

### Stop and Remove the Test Container

```bash
docker stop test-apache && docker rm test-apache
```

### List All Images

```bash
docker images
```

### Remove an Image

```bash
docker rmi nautilus-apache
```

---

## 7. Common Errors & Fixes

### ❌ Error: `port is already allocated`
**Cause:** Something else is already using port 6200 on the host.
```bash
# Find what's using the port
sudo lsof -i :6200
# Kill it or use a different host port
docker run -d -p 6201:6200 --name test-apache nautilus-apache
```

### ❌ Error: `apache2: Could not reliably determine the server's fully qualified domain name`
**Cause:** Apache warning, not a fatal error. Optionally suppress it by adding:
```dockerfile
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
```

### ❌ Error: `Cannot connect to the Docker daemon`
**Cause:** Docker service is not running.
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### ❌ Error: `Dockerfile: no such file or directory`
**Cause:** Incorrect filename casing. Must be `Dockerfile` (capital D), not `dockerfile`.

### ❌ Container exits immediately
**Cause:** Apache was not started in foreground mode.
**Fix:** Ensure `CMD` uses `-D FOREGROUND`.

---

## 8. Dockerfile Best Practices

| Practice | Why |
|----------|-----|
| Chain `RUN` commands with `&&` | Reduces the number of image layers |
| Run `apt-get clean` after installs | Reduces image size |
| Use `EXPOSE` to document ports | Improves readability and tooling support |
| Use exec form for `CMD`: `["cmd", "arg"]` | Avoids shell wrapper; handles signals properly |
| Use specific image tags (`ubuntu:24.04`) | Avoids unexpected behavior from `latest` tag |
| Put least-changing instructions first | Maximises Docker layer cache efficiency |

---

## 9. Quick Reference Cheat Sheet

```bash
# SSH to App Server 3
ssh banner@stapp03

# Become root
sudo su -

# Create directory
mkdir -p /opt/docker

# Edit Dockerfile
vi /opt/docker/Dockerfile

# Build image
docker build -t <image-name> /opt/docker/

# Run container
docker run -d -p 6200:6200 --name <container-name> <image-name>

# Test
curl http://localhost:6200

# Check running containers
docker ps

# Check all containers (including stopped)
docker ps -a

# View logs
docker logs <container-name>

# Stop container
docker stop <container-name>

# Remove container
docker rm <container-name>

# List images
docker images

# Remove image
docker rmi <image-name>
```

---

### Key Files Inside the Container

| Path | Purpose |
|------|---------|
| `/etc/apache2/ports.conf` | Apache port binding configuration |
| `/etc/apache2/sites-enabled/000-default.conf` | Default VirtualHost configuration |
| `/etc/apache2/apache2.conf` | Main Apache configuration file |
| `/var/www/html/` | Default document root |
| `/var/log/apache2/` | Apache log files |

---

*Study material generated for the Nautilus DevOps Lab — Docker + Apache task.*
