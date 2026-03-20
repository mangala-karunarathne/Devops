# Kubernetes Pod — Study Material
### Task: Create a Nginx Pod on Kubernetes
---

## 📋 Original Task

> **Task Description:**
> The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:
>
> 1. Create a pod named `pod-nginx` using the `nginx` image with the `latest` tag. Ensure to specify the tag as `nginx:latest`.
> 2. Set the `app` label to `nginx_app`, and name the container as `nginx-container`.
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🐳 Docker → Kubernetes Mental Model

If you know Docker, use this mapping to understand Kubernetes concepts:

```
Dockerfile  →  builds  →  Docker Image  →  runs as  →  Docker Container
YAML file   →  defines →  Pod Spec      →  runs as  →  Pod (wraps containers)
```

| Concept         | Docker              | Kubernetes              |
|-----------------|---------------------|--------------------------|
| Definition file | `Dockerfile`        | `pod.yaml`               |
| Runnable unit   | `Container`         | `Pod`                    |
| Run command     | `docker run nginx`  | `kubectl apply -f pod.yaml` |
| List running    | `docker ps`         | `kubectl get pods`       |
| Inspect         | `docker inspect`    | `kubectl describe pod`   |
| Image used      | `nginx:latest`      | `nginx:latest` (same!)   |

---

## 📦 What is a Pod?

A **Pod** is the **smallest deployable unit in Kubernetes**.

- It is a **wrapper around one or more containers**
- All containers inside a Pod share the **same network IP and storage**
- Kubernetes manages Pods, not containers directly

```
┌─────────────────────────────┐
│           POD               │
│  ┌───────────────────────┐  │
│  │  Docker Container(s)  │  │
│  │  (nginx-container)    │  │
│  └───────────────────────┘  │
│                             │
│  - Shared Network/IP        │
│  - Shared Storage           │
└─────────────────────────────┘
```

> **Simple summary:** A Pod is to Kubernetes what a running container is to Docker — just with extra features like shared networking, labels, and cluster management.

---

## 🗂️ The YAML File (Blueprint)

Just like a `Dockerfile` is a blueprint for building an image, a **YAML file is a blueprint for creating a Pod**.

```yaml
apiVersion: v1          # Kubernetes API version
kind: Pod               # Type of resource we are creating
metadata:
  name: pod-nginx       # Name of the pod
  labels:
    app: nginx_app      # Label to identify/group this pod
spec:
  containers:
  - name: nginx-container   # Name of the container inside the pod
    image: nginx:latest     # Docker image to pull and run
```

### Dockerfile vs Pod YAML — Side by Side

| Dockerfile               | pod-nginx.yaml                  |
|--------------------------|---------------------------------|
| `FROM nginx:latest`      | `image: nginx:latest`           |
| *(no container naming)*  | `name: nginx-container`         |
| *(no labeling)*          | `labels: app=nginx_app`         |
| `docker build` + `docker run` | `kubectl apply -f pod.yaml` |

---

## 🛠️ Step-by-Step Commands

### Step 1 — Verify cluster is accessible
```bash
kubectl cluster-info
```
**Expected Output:**
```
Kubernetes control plane is running at https://<cluster-ip>:6443
```

---

### Step 2 — Check existing pods
```bash
kubectl get pods
```
**Expected Output (before creating):**
```
No resources found in default namespace.
```

---

### Step 3 — Create the YAML file using heredoc

```bash
cat <<EOF > pod-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx_app
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
EOF
```

> 💡 **What is `cat <<EOF > pod-nginx.yaml`?**
> This is a Bash **heredoc** — a shortcut to create a file with content in one command.
>
> | Part | Meaning |
> |------|---------|
> | `cat` | Reads and outputs text |
> | `<<EOF` | Start reading until the word `EOF` is found |
> | `> pod-nginx.yaml` | Save everything into this file |
> | `EOF` (at end) | Stop reading, file is saved |
>
> It's the same as opening `nano pod-nginx.yaml` and typing the content manually.

---

### Step 4 — Apply the YAML to create the pod
```bash
kubectl apply -f pod-nginx.yaml
```
**Expected Output:**
```
pod/pod-nginx created
```

---

### Step 5 — Verify pod is running
```bash
kubectl get pods
```
**Expected Output:**
```
NAME        READY   STATUS    RESTARTS   AGE
pod-nginx   1/1     Running   0          30s
```

> ⏳ If you see `ContainerCreating`, wait a few seconds — the image is still being pulled from Docker Hub.

---

### Step 6 — Verify labels
```bash
kubectl get pod pod-nginx --show-labels
```
**Expected Output:**
```
NAME        READY   STATUS    RESTARTS   AGE   LABELS
pod-nginx   1/1     Running   0          1m    app=nginx_app
```

---

### Step 7 — Full inspection
```bash
kubectl describe pod pod-nginx
```
**Key sections to verify:**
```
Name:         pod-nginx
Namespace:    default
Labels:       app=nginx_app
Containers:
  nginx-container:
    Image:    nginx:latest
    State:    Running
    Ready:    True
```

---

## ✅ Verification Checklist

| Requirement       | Expected Value     | Status |
|-------------------|--------------------|--------|
| Pod name          | `pod-nginx`        | ✅     |
| Namespace         | `default`          | ✅     |
| Label `app`       | `nginx_app`        | ✅     |
| Container name    | `nginx-container`  | ✅     |
| Image             | `nginx:latest`     | ✅     |
| Pod Status        | `Running`          | ✅     |
| Ready             | `True`             | ✅     |
| Restart Count     | `0`                | ✅     |

---

## 📌 Key Concepts Summary

| Term | Definition |
|------|------------|
| **Pod** | Smallest unit in Kubernetes; wraps one or more containers |
| **kubectl** | CLI tool to interact with Kubernetes cluster |
| **YAML** | Configuration file format used to define Kubernetes resources |
| **label** | Key-value metadata attached to a pod for identification/grouping |
| **image** | Docker image used to create the container inside the pod |
| **heredoc (`<<EOF`)** | Bash trick to write multi-line content into a file from terminal |
| **`kubectl apply`** | Command to create or update resources from a YAML file |
| **`kubectl describe`** | Command to get detailed info about a Kubernetes resource |

---

## 🔗 Useful Commands Reference

```bash
kubectl get pods                          # List all pods
kubectl get pods --show-labels            # List pods with labels
kubectl describe pod <pod-name>           # Detailed info about a pod
kubectl apply -f <file.yaml>              # Create/update resource from YAML
kubectl delete pod <pod-name>             # Delete a pod
kubectl logs <pod-name>                   # View logs of a pod
kubectl get pod <pod-name> -o yaml        # View full YAML of a running pod
```

---

*Study material generated for Nautilus DevOps Kubernetes Task — March 2026*
