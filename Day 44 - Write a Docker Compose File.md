# 📦 Docker Compose – Hosting Static Website with `httpd` on Stratos DC

---

## 📋 Original Task

> **Task Description:**
> The Nautilus application development team shared static website content that needs to be hosted on the `httpd` web server using a containerised platform. The team has shared details with the DevOps team, and we need to set up an environment according to those guidelines. Below are the details:
>
> a. On **App Server 1** in **Stratos DC** create a container named `httpd` using a docker compose file `/opt/docker/docker-compose.yml` (please use the exact name for file).
>
> b. Use `httpd` (preferably `latest` tag) image for container and make sure container is named as `httpd`; you can use any name for service.
>
> c. Map **port 80** of the container with **port 3000** of docker host.
>
> d. Map container's `/usr/local/apache2/htdocs` volume with `/opt/devops` volume of docker host which is already there. *(please do not modify any data within these locations)*

---

## 🧠 Concepts Covered

| Concept | Description |
|---|---|
| Docker Compose | Tool for defining and running multi-container Docker applications using a YAML file |
| `httpd` Image | Official Apache HTTP Server Docker image |
| Port Mapping | Binding a host port to a container port (`host:container`) |
| Volume Mounting | Sharing a host directory with a container directory |
| Detached Mode | Running containers in the background using `-d` flag |

---

## 🗂️ Key Terminology

### Docker Compose File (`docker-compose.yml`)
A YAML configuration file that defines services, networks, and volumes for Docker applications. It allows you to configure all container settings in one place and manage them with simple commands.

### `container_name`
Explicitly sets the name of the running container. Without this, Docker auto-generates a name. Required here to ensure the container is named exactly `httpd`.

### Port Mapping (`ports`)
Format: `"HOST_PORT:CONTAINER_PORT"`
- `"3000:80"` means traffic arriving at host port `3000` is forwarded to port `80` inside the container.

### Volume Mapping (`volumes`)
Format: `HOST_PATH:CONTAINER_PATH`
- `/opt/devops:/usr/local/apache2/htdocs` mounts the host directory into the Apache web root inside the container, so Apache serves files from the host.

---

## 📝 Docker Compose File Used

**File path:** `/opt/docker/docker-compose.yml`

```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3000:80"
    volumes:
      - /opt/devops:/usr/local/apache2/htdocs
```

### Breakdown of Each Field

| Field | Value | Purpose |
|---|---|---|
| `version` | `'3'` | Compose file format version |
| `services` | — | Defines the list of containers/services |
| `web` | (service name) | Logical name for the service (can be anything) |
| `image` | `httpd:latest` | Pulls the latest official Apache HTTP Server image |
| `container_name` | `httpd` | Names the running container `httpd` |
| `ports` | `"3000:80"` | Host port 3000 → Container port 80 |
| `volumes` | `/opt/devops:/usr/local/apache2/htdocs` | Mounts host web content into Apache's web root |

---

## 🚀 Step-by-Step Execution

### Step 1 — SSH into App Server 1
```bash
ssh tony@stapp01
```
> Default user for App Server 1 in Stratos DC is `tony`.

---

### Step 2 — Verify Prerequisites
```bash
# Check Docker installation
docker --version

# Check Docker Compose installation
docker compose version
```

Verify the host volume directory exists (do NOT modify contents):
```bash
ls -la /opt/devops
```

---

### Step 3 — Create the Directory for Compose File
```bash
sudo mkdir -p /opt/docker
```

---

### Step 4 — Create the Docker Compose File
```bash
sudo vi /opt/docker/docker-compose.yml
```

Paste the YAML content shown in the section above and save.

---

### Step 5 — Validate the Compose File
```bash
sudo docker compose -f /opt/docker/docker-compose.yml config
```
This checks for syntax errors before running anything.

---

### Step 6 — Start the Container
```bash
sudo docker compose -f /opt/docker/docker-compose.yml up -d
```
- `-f` specifies the compose file path
- `up` creates and starts the containers
- `-d` runs in detached (background) mode

---

### Step 7 — Verify the Container is Running
```bash
# List running containers
sudo docker ps

# Or via compose
sudo docker compose -f /opt/docker/docker-compose.yml ps
```

**Expected output (docker ps):**
```
CONTAINER ID   IMAGE          COMMAND              PORTS                  NAMES
xxxxxxxxxxxx   httpd:latest   "httpd-foreground"   0.0.0.0:3000->80/tcp   httpd
```

---

### Step 8 — Test the Web Server
```bash
# Test locally on App Server 1
curl http://localhost:3000
```
You should receive an HTML response — the content served from `/opt/devops`.

---

## 🔧 Useful Troubleshooting Commands

| Scenario | Command |
|---|---|
| View container logs | `sudo docker logs httpd` |
| Restart the container | `sudo docker compose -f /opt/docker/docker-compose.yml restart` |
| Stop and remove containers | `sudo docker compose -f /opt/docker/docker-compose.yml down` |
| Check port binding on host | `sudo ss -tlnp \| grep 3000` |
| Inspect container details | `sudo docker inspect httpd` |
| Check volume mounts | `sudo docker inspect httpd \| grep -A 10 Mounts` |

---

## ⚠️ Common Mistakes to Avoid

1. **Wrong file path** — The compose file must be at `/opt/docker/docker-compose.yml` exactly.
2. **Wrong container name** — `container_name` must be `httpd`, not the service name.
3. **Reversed port mapping** — Always `HOST:CONTAINER`, so `3000:80` not `80:3000`.
4. **Modifying `/opt/devops`** — The task explicitly says do NOT modify data in the host volume.
5. **Not using `sudo`** — Docker commands typically require elevated privileges.
6. **Forgetting `-d`** — Without detached mode, the container runs in the foreground and blocks your terminal.

---

## 📌 Key Facts to Remember

- **Apache web root inside `httpd` container:** `/usr/local/apache2/htdocs`
- **Default Apache HTTP port:** `80`
- **Docker Compose file format:** YAML (indentation matters!)
- **Service name vs container name:** Service name (`web`) is used within Compose; `container_name` (`httpd`) is the actual running container name visible in `docker ps`.
- **`version: '3'`** is a widely compatible Compose file version for most Docker installations.

---

## 🔗 References

- [Official `httpd` Docker Image – Docker Hub](https://hub.docker.com/_/httpd)
- [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Docker CLI Reference – `docker compose up`](https://docs.docker.com/engine/reference/commandline/compose_up/)
