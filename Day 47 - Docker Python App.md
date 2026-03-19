# 🐳 Dockerizing a Python App — Study Material

---

## 📋 Original Task

> A Python app needed to be Dockerized, and then deployed on **App Server 3**. A `requirements.txt` file (having the app dependencies) was already copied under `/python_app/src/` directory on App Server 3.
>
> **Requirements:**
> 1. Create a `Dockerfile` under `/python_app` directory:
>    - Use any `python` image as the base image.
>    - Install the dependencies using `requirements.txt` file.
>    - Expose the port `6100`.
>    - Run the `server.py` script using `CMD`.
> 2. Build an image named `nautilus/python-app` using this Dockerfile.
> 3. Once image is built, create a container named `pythonapp_nautilus`:
>    - Map port `6100` of the container to the host port `8093`.
> 4. Test the app using `curl` on App Server 3:
>    ```bash
>    curl http://localhost:8093/
>    ```

---

## 🧠 Concepts Covered

| Concept | Description |
|---|---|
| **Dockerfile** | A text file with instructions to build a Docker image |
| **Docker Image** | A read-only template used to create containers |
| **Docker Container** | A running instance of a Docker image |
| **Port Mapping** | Binding a container port to a host port (`host:container`) |
| **Base Image** | The starting image your Dockerfile builds upon |
| **EXPOSE** | Documents which port the container listens on |
| **CMD** | The default command that runs when the container starts |

---

## 📁 Directory Structure

```
/python_app/
├── Dockerfile          ← Created in this task
└── src/
    ├── requirements.txt  ← Pre-existing
    └── server.py         ← Pre-existing Python app
```

---

## 📝 Dockerfile — Full Explanation

```dockerfile
# 1. Base image: official Python slim image (lightweight)
FROM python:3.9-slim

# 2. Set the working directory inside the container
WORKDIR /app

# 3. Copy requirements.txt from host src/ into the container
COPY src/requirements.txt .

# 4. Install all Python dependencies listed in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy all remaining source files into the container
COPY src/ .

# 6. Document that the app listens on port 6100
EXPOSE 6100

# 7. Default command to run when the container starts
CMD ["python", "server.py"]
```

### 🔍 Dockerfile Instruction Breakdown

| Instruction | Purpose | Example |
|---|---|---|
| `FROM` | Sets the base image | `FROM python:3.9-slim` |
| `WORKDIR` | Sets working directory inside container | `WORKDIR /app` |
| `COPY` | Copies files from host to container | `COPY src/ .` |
| `RUN` | Executes a command during image build | `RUN pip install -r requirements.txt` |
| `EXPOSE` | Documents the port the app listens on | `EXPOSE 6100` |
| `CMD` | Default command when container starts | `CMD ["python", "server.py"]` |

> ⚠️ **Note:** `EXPOSE` does **not** actually publish the port. It's informational. The actual port publishing happens with `-p` flag in `docker run`.

---

## 🔨 Step-by-Step Commands

### Step 1 — Verify `requirements.txt` exists
```bash
ls -la /python_app/src/
```

### Step 2 — Create the Dockerfile
```bash
cat > /python_app/Dockerfile <<'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/ .
EXPOSE 6100
CMD ["python", "server.py"]
EOF
```

### Step 3 — Build the Docker image
```bash
cd /python_app
docker build -t nautilus/python-app .
```

### Step 4 — Run the container
```bash
docker run -d \
  --name pythonapp_nautilus \
  -p 8093:6100 \
  nautilus/python-app
```

### Step 5 — Verify the container is running
```bash
docker ps | grep pythonapp_nautilus
```

### Step 6 — Test the app
```bash
curl http://localhost:8093/
```

---

## ✅ Actual Output (from App Server 3)

```
# Image built successfully
nautilus/python-app   latest   226626083655   About a minute ago   133MB

# Container running
a453f224bcd3   nautilus/python-app   "python server.py"   28 seconds ago   Up 27 seconds   0.0.0.0:8093->6100/tcp   pythonapp_nautilus

# curl test response
Welcome to xFusionCorp Industries!
```

---

## 🔑 Key Docker Commands Reference

```bash
# Build an image from a Dockerfile
docker build -t <image-name> <path>

# Run a container in detached mode
docker run -d --name <container-name> -p <host-port>:<container-port> <image-name>

# List running containers
docker ps

# List all images
docker images

# View container logs
docker logs <container-name>

# Stop a container
docker stop <container-name>

# Remove a container
docker rm <container-name>

# Remove an image
docker rmi <image-name>

# Inspect container details
docker inspect <container-name>
```

---

## 🔁 Port Mapping — How It Works

```
Host Machine (App Server 3)          Docker Container
┌─────────────────────────┐          ┌──────────────────┐
│                         │          │                  │
│   curl localhost:8093 ──┼──────────┼──► server.py     │
│                         │  8093:6100│   (port 6100)   │
│                         │          │                  │
└─────────────────────────┘          └──────────────────┘
```

- **Host port `8093`** → receives external traffic
- **Container port `6100`** → where the app actually runs inside the container
- Format: `-p <host_port>:<container_port>`

---

## 🐍 Python Image Variants (Base Image Choices)

| Image | Size | Use Case |
|---|---|---|
| `python:3.x` | ~900MB | Full-featured, development |
| `python:3.x-slim` | ~130MB | Production, minimal OS packages |
| `python:3.x-alpine` | ~50MB | Ultra-lightweight, minimal |
| `python:3.x-bullseye` | ~900MB | Debian Bullseye base |

> ✅ **Best practice:** Use `slim` or `alpine` variants for production to reduce image size and attack surface.

---

## 🛠️ Troubleshooting Guide

| Problem | Possible Cause | Fix |
|---|---|---|
| `docker build` fails | Missing files or syntax error in Dockerfile | Check `COPY` paths and Dockerfile syntax |
| Container exits immediately | App crashes on start | Run `docker logs <container-name>` |
| Port already in use | Another process using port 8093 | Use `netstat -tulpn \| grep 8093` to find and kill it |
| `curl` connection refused | Container not running | Check `docker ps` and container logs |
| Dependencies not found | Wrong path to `requirements.txt` | Verify `COPY` instruction paths |

---

## 📌 Best Practices

1. **Use specific image tags** — avoid `latest` in production (e.g., `python:3.9-slim` not `python:latest`)
2. **Use `--no-cache-dir`** with pip to reduce image size
3. **Copy `requirements.txt` before source code** — leverages Docker layer caching
4. **Use `CMD` in exec form** — `["python", "server.py"]` not `python server.py` (avoids shell wrapping)
5. **Use `.dockerignore`** — exclude unnecessary files from the build context
6. **Run as non-root user** — add `USER` instruction for security in production

---

## 📚 Further Reading

- [Docker Official Docs](https://docs.docker.com/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Python Docker Hub](https://hub.docker.com/_/python)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
