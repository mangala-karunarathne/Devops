# Nginx + PHP-FPM on Kubernetes — Study Guide

---

## 📋 Original Task

> We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:
>
> - The pod name is `nginx-phpfpm` and configmap name is `nginx-config`
> - Identify and fix the problem
> - Once resolved, copy `/home/thor/index.php` file from the `jump host` to the `nginx-container` within the nginx document root
> - After this, you should be able to access the website using the `Website` button on the top bar
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🏗️ Architecture Overview

In this setup, there are **2 containers inside 1 pod** working together:

```
[ Pod: nginx-phpfpm ]
  ┌─────────────────────┐     ┌─────────────────────┐
  │   nginx-container   │────▶│ php-fpm-container   │
  │   (Web Server)      │     │ (PHP Processor)      │
  └─────────────────────┘     └─────────────────────┘
         ▲                            ▲
         │                            │
         └──────── shared-files ──────┘
                  (emptyDir volume)
```

- **Nginx** receives HTTP requests from the browser
- **PHP-FPM** processes `.php` files and returns HTML
- Both share a **volume** where PHP files live
- They communicate via `localhost:9000` (same pod = shared network)

---

## 📄 Config File 1: nginx.conf (stored in ConfigMap)

```nginx
events {
}
http {
  server {
    listen 8099 default_server;
    listen [::]:8099 default_server;

    root /var/www/html;
    index index.html index.htm index.php;
    server_name _;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
      include fastcgi_params;
      fastcgi_param REQUEST_METHOD $request_method;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_pass 127.0.0.1:9000;
    }
  }
}
```

### Line-by-Line Explanation

| Line | Explanation |
|---|---|
| `events {}` | Required by Nginx. Left empty = use defaults for connection handling |
| `http {}` | All HTTP traffic configuration lives inside here |
| `server {}` | Defines one virtual host / website |
| `listen 8099 default_server` | Listen on port 8099 for IPv4 traffic |
| `listen [::]:8099 default_server` | Listen on port 8099 for IPv6 traffic |
| `root /var/www/html` | 🔑 Document root — all website files must be here |
| `index index.html index.htm index.php` | Default files to look for when visiting `/` |
| `server_name _` | Wildcard — respond to any domain or IP |
| `try_files $uri $uri/ =404` | Try exact file → try as directory → 404 if not found |
| `location ~ \.php$` | `~` = regex match. Applies to any URL ending in `.php` |
| `include fastcgi_params` | Load standard FastCGI variables needed by PHP-FPM |
| `fastcgi_param REQUEST_METHOD` | Passes HTTP method (GET/POST) to PHP-FPM |
| `fastcgi_param SCRIPT_FILENAME` | 🔑 Full path of PHP file: `root` + `script name` |
| `fastcgi_pass 127.0.0.1:9000` | 🔑 Forward PHP requests to PHP-FPM on localhost port 9000 |

---

## 📄 Config File 2: Pod Spec (from `kubectl describe`)

### Volumes Section

```yaml
volumes:
  - name: shared-files
    emptyDir: {}

  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

| Volume | Type | Purpose |
|---|---|---|
| `shared-files` | emptyDir | Temporary shared folder. Lives as long as the pod. Both containers plug into this. |
| `nginx-config-volume` | ConfigMap | Injects nginx.conf from the ConfigMap into the container filesystem |

### Container: php-fpm-container

```yaml
- name: php-fpm-container
  image: php:7.2-fpm-alpine
  volumeMounts:
    - mountPath: /var/www/html       # ✅ After fix
      name: shared-files
```

### Container: nginx-container

```yaml
- name: nginx-container
  image: nginx:latest
  volumeMounts:
    - mountPath: /etc/nginx/nginx.conf
      name: nginx-config-volume
      subPath: nginx.conf
    - mountPath: /var/www/html
      name: shared-files
```

---

## 🔗 How Everything Links Together

```
ConfigMap (nginx-config)
    └── nginx.conf
          ├── root /var/www/html ──────────────────────┐
          └── fastcgi_pass 127.0.0.1:9000 ──────────┐  │
                                                      │  │
Pod Spec                                              │  │
    ├── shared-files (emptyDir volume)                │  │
    │     ├── nginx-container    → /var/www/html ─────┼──┘
    │     └── php-fpm-container  → /var/www/html ─────┘
    │
    └── nginx-config-volume (ConfigMap volume)
          └── nginx-container → /etc/nginx/nginx.conf
```

### Request Flow

```
Browser Request (port 8099)
      │
      ▼
nginx-container
      │
      ├── Static files (.html/.css) → reads from /var/www/html directly
      │
      └── PHP files (.php) → forwards to 127.0.0.1:9000
                                        │
                                        ▼
                               php-fpm-container
                                        │
                                        └── reads .php from /var/www/html
                                                    │
                                                    ▼
                                           Processes PHP → HTML
                                                    │
                                                    ▼
                                        Back to Nginx → Browser
```

---

## 🐛 Root Cause of the Issue

The `php-fpm-container` had the shared volume mounted at the **wrong path**:

| Component | Path | Status |
|---|---|---|
| nginx.conf `root` directive | `/var/www/html` | ✅ Correct |
| nginx-container volume mount | `/var/www/html` | ✅ Correct |
| php-fpm-container volume mount | `/usr/share/nginx/html` | ❌ WRONG |

PHP-FPM was looking for PHP files in `/usr/share/nginx/html` while Nginx was serving from `/var/www/html` — a completely different folder. PHP-FPM could never find `index.php`.

---

## ✅ The Fix

### Step 1: Check pod status
```bash
kubectl get pod nginx-phpfpm
kubectl describe pod nginx-phpfpm
```

### Step 2: Check the ConfigMap
```bash
kubectl get configmap nginx-config -o yaml
```

### Step 3: Delete the broken pod
```bash
kubectl delete pod nginx-phpfpm
```

### Step 4: Recreate pod with corrected mount path (using EOF heredoc)
```bash
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  namespace: default
  labels:
    app: php-app
spec:
  volumes:
  - name: shared-files
    emptyDir: {}
  - name: nginx-config-volume
    configMap:
      name: nginx-config
  containers:
  - name: php-fpm-container
    image: php:7.2-fpm-alpine
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html        # Fixed: was /usr/share/nginx/html
  - name: nginx-container
    image: nginx:latest
    volumeMounts:
    - name: nginx-config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
    - name: shared-files
      mountPath: /var/www/html
EOF
```

### Step 5: Verify both containers are running
```bash
kubectl get pod nginx-phpfpm
```
Expected:
```
NAME            READY   STATUS    RESTARTS   AGE
nginx-phpfpm    2/2     Running   0          20s
```

### Step 6: Copy index.php into the nginx-container
```bash
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
```

### Step 7: Verify the file is in place
```bash
kubectl exec nginx-phpfpm -c nginx-container -- ls /var/www/html/
```

### Step 8: Test the website
```bash
kubectl exec nginx-phpfpm -c nginx-container -- curl http://localhost:8099/index.php
```

---

## 🔑 The Golden Rule

> **Every path and port referenced in one config must exactly match what is defined in another.**

| Config says | Must match |
|---|---|
| `root /var/www/html` in nginx.conf | `mountPath` in both containers |
| `fastcgi_pass 127.0.0.1:9000` | PHP-FPM's listening port (9000) |
| `mountPath` in pod spec | Actual path nginx/php-fpm expects |

---

## 🛠️ Useful kubectl Commands Reference

| Command | Purpose |
|---|---|
| `kubectl get pod <name>` | Check pod status and readiness |
| `kubectl describe pod <name>` | Detailed pod info — volumes, mounts, events |
| `kubectl get configmap <name> -o yaml` | View ConfigMap contents |
| `kubectl logs <pod> -c <container>` | View container logs |
| `kubectl exec <pod> -c <container> -- <cmd>` | Run command inside a container |
| `kubectl cp <src> <pod>:<dest> -c <container>` | Copy file into a container |
| `kubectl delete pod <name>` | Delete a pod |
| `kubectl apply -f <file>` | Create/update resources from YAML |

---

## 📚 Key Concepts Glossary

| Term | Definition |
|---|---|
| **Pod** | Smallest deployable unit in Kubernetes. Can contain one or more containers that share network and storage. |
| **Container** | A running instance of a Docker image. Isolated process with its own filesystem. |
| **ConfigMap** | Kubernetes object to store non-sensitive config data (like nginx.conf) outside of containers. |
| **Volume** | A directory accessible to containers in a pod. Survives container restarts. |
| **emptyDir** | A temporary volume created when a pod starts. Shared between containers in the same pod. Deleted when pod is deleted. |
| **volumeMount** | Where a volume is attached inside a container's filesystem. |
| **PHP-FPM** | FastCGI Process Manager. Handles PHP execution. Listens on port 9000 by default. |
| **FastCGI** | Protocol for communication between Nginx and PHP-FPM. |
| **document root** | The folder from which a web server serves files. Defined by `root` in nginx.conf. |
| **EOF heredoc** | Shell syntax to pass multiline text to a command without creating a file first. |

---

*Study guide generated from a live Kubernetes debugging session.*
