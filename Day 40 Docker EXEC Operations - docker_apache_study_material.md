# Docker & Apache — Study Material
> Based on a real DevOps task: Installing & configuring Apache inside a Docker container

---

## 1. Core Concepts

### What is Docker?
Docker is a **containerization platform** that lets you run applications in isolated environments called **containers**.

- Think of a container like a **lightweight virtual machine** — but faster and more portable
- Each container has its own filesystem, network, and processes
- Containers run on top of a **host machine** (like stapp02 in our task)

### Key Docker Terms

| Term | Meaning |
|------|---------|
| **Image** | A blueprint/template for a container (like a class in OOP) |
| **Container** | A running instance of an image (like an object) |
| **Docker Host** | The physical/virtual machine running Docker (stapp02) |
| **docker exec** | Run a command inside an already running container |

---

## 2. Docker Commands Used in This Task

### Check containers
```bash
docker ps          # running containers only
docker ps -a       # all containers (including stopped)
```

### Start a stopped container
```bash
docker start kkloud
```

### Run commands inside a container
```bash
docker exec <container_name> <command>

# Examples:
docker exec kkloud apt-get update
docker exec kkloud service apache2 start
```

### Get container's IP address
```bash
docker inspect kkloud | grep IPAddress
```

---

## 3. Apache Web Server Basics

### What is Apache?
Apache (`apache2`) is one of the most popular **open-source web servers**. It serves web pages/content over HTTP.

### How Apache is installed (inside a Debian/Ubuntu container)
```bash
apt-get update -y
apt-get install -y apache2
```

### Key Apache Config Files

| File | Purpose |
|------|---------|
| `/etc/apache2/ports.conf` | Defines which **port** Apache listens on |
| `/etc/apache2/sites-enabled/000-default.conf` | Default **virtual host** configuration |
| `/etc/apache2/apache2.conf` | Main Apache configuration file |

---

## 4. Configuring Apache to Listen on a Custom Port

### Default behavior
Apache listens on **port 80** by default.

### Changing to port 8088

**File 1: `/etc/apache2/ports.conf`**
```
# Before
Listen 80

# After
Listen 8088
```

**File 2: `/etc/apache2/sites-enabled/000-default.conf`**
```
# Before
<VirtualHost *:80>

# After
<VirtualHost *:8088>
```

### The `sed` command used to do this
```bash
# sed -i = in-place edit (modifies the file directly)
sed -i 's/Listen 80/Listen 8088/' /etc/apache2/ports.conf
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8088>/' /etc/apache2/sites-enabled/000-default.conf
```

---

## 5. IP Binding — Specific vs All Interfaces

### Bind to a specific IP (restrictive)
```
Listen 192.168.1.5:8088
```
- Only accepts requests to that one IP
- If container restarts with a new IP → **everything breaks**

### Bind to all interfaces (flexible) ✅
```
Listen 8088
```
- Accepts requests on **all network interfaces**
- Works with: `localhost`, `127.0.0.1`, container IP, etc.
- **Best practice for containers**

---

## 6. Managing Apache Service

```bash
# Start Apache
service apache2 start

# Stop Apache
service apache2 stop

# Restart Apache (after config changes)
service apache2 restart

# Check status
service apache2 status
```

---

## 7. Verifying the Setup

### Check Apache is responding on port 8088
```bash
# From inside the container
docker exec kkloud curl -s http://localhost:8088

# From the host (stapp02) using container IP
curl http://<container-ip>:8088

# Check which ports are listening
docker exec kkloud ss -tlnp | grep 8088
```

---

## 8. Full Task Walkthrough (End to End)

```bash
# 1. SSH into App Server 2
ssh steve@stapp02

# 2. Switch to root
sudo su -

# 3. Check container
docker ps -a | grep kkloud

# 4. Start if stopped
docker start kkloud

# 5. Install Apache
docker exec kkloud apt-get update -y
docker exec kkloud apt-get install -y apache2

# 6. Configure port 8088
docker exec kkloud sed -i 's/Listen 80/Listen 8088/' /etc/apache2/ports.conf
docker exec kkloud sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8088>/' /etc/apache2/sites-enabled/000-default.conf

# 7. Start Apache
docker exec kkloud service apache2 start

# 8. Verify
docker exec kkloud curl -s http://localhost:8088 | head -5
docker ps | grep kkloud
```

---

## 9. Real World Use Cases

| Scenario | Why custom port? |
|----------|-----------------|
| Multiple services on same host | Avoid port conflicts |
| Dev/Staging environments | Port 80 reserved for production |
| Microservices architecture | Each service gets its own port |
| Security | Obscure default ports from basic scanners |

---

## 10. Quick Reference — Common Ports

| Port | Service |
|------|---------|
| 80 | HTTP (default) |
| 443 | HTTPS |
| 22 | SSH |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 8080, 8088, 8443 | Common alternative HTTP ports |

---

*Happy Learning! 🚀 Keep completing those KodeKloud tasks!*
