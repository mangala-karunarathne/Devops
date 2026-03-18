# 🐳 Docker Compose Deployment — Study Material
### Task: Containerized Stack Deployment on Stratos Datacenter (App Server 3)

---

## 📋 Table of Contents
1. [Original Task Requirements](#original-task-requirements)
2. [Key Concepts](#key-concepts)
3. [Tools & Technologies Used](#tools--technologies-used)
4. [Step-by-Step Solution](#step-by-step-solution)
5. [The Docker Compose File — Explained](#the-docker-compose-file--explained)
6. [Architecture Overview](#architecture-overview)
7. [Common Errors & Fixes](#common-errors--fixes)
8. [Useful Commands Cheat Sheet](#useful-commands-cheat-sheet)
9. [Key Takeaways](#key-takeaways)

---

## 📌 Original Task Requirements

> **Nautilus Application Team — Containerized Stack Deployment**

1. On **App Server 3** in **Stratos Datacenter**, create a Docker Compose file at:
   `/opt/finance/docker-compose.yml` (must be named exactly)

2. The compose file must deploy **two services**: `web` and `db`

### Web Service Requirements:
| Parameter | Value |
|-----------|-------|
| Container Name | `php_host` |
| Image | `php` with any `apache` tag (e.g., `php:8.2-apache`) |
| Port Mapping | Host `8084` → Container `80` |
| Volume Mapping | Host `/var/www/html` → Container `/var/www/html` |

### DB Service Requirements:
| Parameter | Value |
|-----------|-------|
| Container Name | `mysql_host` |
| Image | `mariadb:latest` |
| Port Mapping | Host `3306` → Container `3306` |
| Volume Mapping | Host `/var/lib/mysql` → Container `/var/lib/mysql` |
| Environment Variables | `MYSQL_DATABASE=database_host`, custom non-root user with complex password |

3. After running `docker compose up`, the app must be accessible via:
   ```
   curl <server-ip or hostname>:8084/
   ```

> ⚠️ **Note:** On clicking FINISH, all running/stopped containers are destroyed and the stack is redeployed using your compose file.

---

## 🧠 Key Concepts

### What is Docker?
Docker is a platform that packages applications and their dependencies into **containers** — lightweight, portable, isolated environments that run consistently across any system.

### What is Docker Compose?
Docker Compose is a tool to **define and run multi-container Docker applications** using a single YAML configuration file (`docker-compose.yml`). Instead of running multiple `docker run` commands, you describe all services in one file and start them with a single command.

### What is a Container?
A container is a running instance of a Docker **image**. It is isolated, has its own filesystem, network, and processes — but shares the host OS kernel.

### What is a Docker Image?
A Docker image is a **read-only template** used to create containers. Images are pulled from Docker Hub (e.g., `php:8.2-apache`, `mariadb:latest`).

### What is a Volume?
A volume maps a **host directory** to a **container directory**, allowing:
- Data to **persist** even when the container is removed
- Files on the host to be **immediately reflected** inside the container

### What is Port Mapping?
Port mapping exposes a container's internal port to the outside world via a host port.
Format: `HOST_PORT:CONTAINER_PORT`
Example: `"8084:80"` means accessing `localhost:8084` routes to port `80` inside the container.

### What is a Network in Docker Compose?
Docker Compose automatically creates a **shared network** for all services defined in the same compose file. Services can communicate with each other using their **service name** as the hostname (e.g., `db` resolves to the MariaDB container).

---

## 🛠️ Tools & Technologies Used

| Tool | Purpose |
|------|---------|
| **Docker** | Container runtime |
| **Docker Compose v2** | Multi-container orchestration |
| **PHP 8.2 with Apache** | Web server serving PHP pages |
| **MariaDB** | Relational database (MySQL-compatible) |
| **vim** | Text editor to create the compose file |
| **curl** | Command-line HTTP testing tool |

---

## 🚶 Step-by-Step Solution

### Step 1 — SSH into App Server 3
```bash
ssh banner@stapp03
sudo su -
```

### Step 2 — Create the directory
```bash
mkdir -p /opt/finance
```
> `-p` flag creates parent directories if they don't exist and doesn't error if it already exists.

### Step 3 — Open vim and create the compose file
```bash
vi /opt/finance/docker-compose.yml
```
- Press `i` to enter **INSERT** mode
- Paste the compose file content (see below)
- Press `Esc`, then type `:wq` and hit **Enter** to save and exit

### Step 4 — Verify the file
```bash
cat /opt/finance/docker-compose.yml
```

### Step 5 — Navigate to directory and start the stack
```bash
cd /opt/finance
docker compose up -d
```
> `-d` flag runs containers in **detached mode** (background)

### Step 6 — Verify running containers
```bash
docker ps
```

### Step 7 — Test the web service
```bash
curl http://localhost:8084/
```
Expected: HTML response from the PHP/Apache container ✅

---

## 📄 The Docker Compose File — Explained

```yaml
version: '3'
```
> Specifies the Docker Compose file format version. Now obsolete in Compose v2 but still accepted (causes only a warning, not an error).

---

```yaml
services:
```
> The root section that defines all containers to be deployed.

---

### 🌐 Web Service

```yaml
  web:
    container_name: php_host
```
> Defines a service called `web`. The container is given a fixed name `php_host`.

```yaml
    image: php:8.2-apache
```
> Uses the official PHP image with Apache web server built in. The `8.2-apache` tag means PHP version 8.2.

```yaml
    ports:
      - "8084:80"
```
> Maps **host port 8084** to **container port 80** (Apache's default port).
> Accessing `http://server-ip:8084` is routed to Apache inside the container.

```yaml
    volumes:
      - /var/www/html:/var/www/html
```
> Mounts the host's `/var/www/html` directory into the container's web root.
> Any files placed here are immediately served by Apache — no container rebuild needed.

---

### 🗄️ DB Service

```yaml
  db:
    container_name: mysql_host
```
> Defines the database service. Container is named `mysql_host`.

```yaml
    image: mariadb:latest
```
> Uses the latest official MariaDB image. MariaDB is a drop-in MySQL replacement, fully compatible with PHP applications.

```yaml
    ports:
      - "3306:3306"
```
> Maps the standard MySQL/MariaDB port from container to host.
> Allows external tools (e.g., MySQL Workbench) to connect directly.

```yaml
    volumes:
      - /var/lib/mysql:/var/lib/mysql
```
> Persists MariaDB's data directory on the host.
> Database data survives container restarts and removals.

```yaml
    environment:
      MYSQL_DATABASE: database_host
      MYSQL_USER: nautilus_user
      MYSQL_PASSWORD: N@utilu5$ecure#2024
      MYSQL_ROOT_PASSWORD: R00t$ecure#2024
```
> Sets environment variables that MariaDB reads on **first startup** to auto-configure:

| Variable | Purpose |
|----------|---------|
| `MYSQL_DATABASE` | Auto-creates a database named `database_host` |
| `MYSQL_USER` | Creates a non-root user `nautilus_user` |
| `MYSQL_PASSWORD` | Password for `nautilus_user` |
| `MYSQL_ROOT_PASSWORD` | Password for the `root` superuser |

> ⚠️ The task requires a **non-root user** — never use `root` as `MYSQL_USER`.

---

## 🏗️ Architecture Overview

```
                    ┌─────────────────────────────────────┐
                    │           App Server 3               │
                    │                                      │
  curl :8084 ──────►│  Host Port 8084                      │
                    │       │                              │
                    │       ▼                              │
                    │  ┌─────────────────┐                 │
                    │  │   php_host      │                 │
                    │  │  PHP 8.2-Apache │──────────────┐  │
                    │  │  Port 80        │              │  │
                    │  └─────────────────┘              │  │
                    │       │                           │  │
                    │  /var/www/html (volume)            │  │
                    │                                   ▼  │
                    │                    ┌──────────────────┐│
                    │                    │   mysql_host     ││
                    │                    │   MariaDB:latest ││
                    │                    │   Port 3306      ││
                    │                    └──────────────────┘│
                    │                           │            │
                    │                    /var/lib/mysql       │
                    │                    (volume)             │
                    │                                        │
                    │    Both on: finance_default network    │
                    └─────────────────────────────────────────┘
```

The `php_host` container can reach `mysql_host` internally using the hostname `db` (service name), without going through the host network.

---

## ❌ Common Errors & Fixes

### Error 1: `docker-compose: command not found`
**Cause:** Docker Compose v1 (standalone binary) is not installed.

**Fix:** Use Docker Compose v2 syntax (no hyphen):
```bash
docker compose up -d   # ✅ v2 syntax
docker-compose up -d   # ❌ v1 syntax — may not be available
```

Or install v1 manually:
```bash
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

---

### Error 2: `WARN: version is obsolete`
**Cause:** Docker Compose v2 no longer requires the `version:` field.

**Fix:** This is just a **warning, not an error**. The stack still deploys correctly. You can safely remove the `version: '3'` line from your file to eliminate the warning.

---

### Error 3: Port already in use
**Cause:** Another process or container is using port 8084 or 3306.

**Fix:**
```bash
# Check what's using the port
netstat -tulpn | grep -E '8084|3306'

# Stop all containers
docker stop $(docker ps -q)
docker rm $(docker ps -aq)

# Then re-run
docker compose up -d
```

---

### Error 4: YAML indentation error
**Cause:** Tabs used instead of spaces, or inconsistent spacing.

**Fix:** YAML requires **spaces only** (no tabs). Each indentation level = **2 spaces**. Validate with:
```bash
docker compose config   # validates and prints resolved compose file
```

---

## 📝 Useful Commands Cheat Sheet

| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start all services in background |
| `docker compose down` | Stop and remove all containers + network |
| `docker compose ps` | List services and their status |
| `docker compose logs` | View logs from all services |
| `docker compose logs web` | View logs from web service only |
| `docker ps` | List all running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker stop <name>` | Stop a container |
| `docker rm <name>` | Remove a stopped container |
| `docker images` | List downloaded images |
| `docker exec -it php_host bash` | Open shell inside `php_host` container |
| `docker exec -it mysql_host bash` | Open shell inside `mysql_host` container |
| `docker compose config` | Validate and view the resolved compose file |
| `curl http://localhost:8084/` | Test the web service |

---

## ✅ Key Takeaways

1. **Docker Compose** simplifies multi-container deployments using a single YAML file.
2. **Services** in the same compose file share a network and can communicate using service names as hostnames.
3. **Port mapping** (`HOST:CONTAINER`) exposes container services to the outside world.
4. **Volume mapping** (`HOST_PATH:CONTAINER_PATH`) persists data and enables live file sharing.
5. **Environment variables** in MariaDB configure the database automatically on first startup.
6. Always use a **non-root database user** — never use `root` as `MYSQL_USER`.
7. **Docker Compose v2** uses `docker compose` (with a space), not `docker-compose` (with a hyphen).
8. YAML files are **indentation-sensitive** — always use spaces, never tabs.
9. The `version:` field is **optional** in Compose v2 and can be removed.
10. **Volumes ensure data persistence** — without volumes, all data is lost when the container is removed.

---

*Study material generated based on hands-on task completion on Stratos Datacenter — App Server 3.*
