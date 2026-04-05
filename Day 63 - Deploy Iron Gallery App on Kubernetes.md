# Kubernetes Study Guide — Iron Gallery Deployment Task

> **Task completed on:** Kubernetes cluster using `kubectl` from a jump-host  
> **Cluster runtime:** containerd 1.6.8 | k3s v1.34.1 | Alpine Linux v3.16

---

## Table of Contents

1. [Original Task](#1-original-task)
2. [Kubernetes Concepts — What is a "kind"?](#2-kubernetes-concepts--what-is-a-kind)
3. [Architecture Overview](#3-architecture-overview)
4. [Step-by-Step Commands with Explanations](#4-step-by-step-commands-with-explanations)
5. [The EOF Heredoc Explained](#5-the-eof-heredoc-explained)
6. [Labels and Selectors — The Glue of Kubernetes](#6-labels-and-selectors--the-glue-of-kubernetes)
7. [Service Types — ClusterIP vs NodePort](#7-service-types--clusterip-vs-nodeport)
8. [Common Beginner Mistakes](#8-common-beginner-mistakes)
9. [Verification Commands](#9-verification-commands)
10. [Key Learning Points Summary](#10-key-learning-points-summary)

---

## 1. Original Task

> The Nautilus DevOps team has customized the Iron Gallery app and wants to deploy it on the Kubernetes cluster.

### Requirements

**1. Namespace**
- Create namespace: `iron-namespace-datacenter`

**2. Iron Gallery Deployment** (`iron-gallery-deployment-datacenter`)
- Namespace: `iron-namespace-datacenter`
- Label: `run: iron-gallery`
- Replicas: `1`
- Selector matchLabels: `run: iron-gallery`
- Template labels: `run: iron-gallery`
- Container name: `iron-gallery-container-datacenter`
- Image: `kodekloud/irongallery:2.0` (exact)
- Resource limits: memory `100Mi`, CPU `50m`
- VolumeMount 1: name `config`, mountPath `/usr/share/nginx/html/data`
- VolumeMount 2: name `images`, mountPath `/usr/share/nginx/html/uploads`
- Volume 1: name `config`, type `emptyDir`
- Volume 2: name `images`, type `emptyDir`

**3. Iron DB Deployment** (`iron-db-deployment-datacenter`)
- Namespace: `iron-namespace-datacenter`
- Label: `db: mariadb`
- Replicas: `1`
- Selector matchLabels: `db: mariadb`
- Template labels: `db: mariadb`
- Container name: `iron-db-container-datacenter`
- Image: `kodekloud/irondb:2.0` (exact)
- Environment variables:
  - `MYSQL_DATABASE` = `database_apache`
  - `MYSQL_ROOT_PASSWORD` = (complex password)
  - `MYSQL_PASSWORD` = (complex password)
  - `MYSQL_USER` = (any user except root)
- VolumeMount: name `db`, mountPath `/var/lib/mysql`
- Volume: name `db`, type `emptyDir`

**4. Iron DB Service** (`iron-db-service-datacenter`)
- Namespace: `iron-namespace-datacenter`
- Selector: `db: mariadb`
- Protocol: `TCP`
- Port & targetPort: `3306`
- Type: `ClusterIP`

**5. Iron Gallery Service** (`iron-gallery-service-datacenter`)
- Namespace: `iron-namespace-datacenter`
- Selector: `run: iron-gallery`
- Protocol: `TCP`
- Port & targetPort: `80`
- NodePort: `32678`
- Type: `NodePort`

> **Note:** No database-frontend connection needed. If the installation page loads, the task is complete.

---

## 2. Kubernetes Concepts — What is a "kind"?

In Kubernetes, `kind` is a field in every YAML manifest that tells Kubernetes **what type of resource (API object)** you want to create or manage.

The correct terminology for these is **Kubernetes Resource Types** or **Kubernetes API Objects**.

### Common Resource Types (kinds)

| Kind | Purpose | Scope |
|------|---------|-------|
| `Namespace` | Logical isolation layer — like a virtual cluster inside your cluster | Cluster-wide |
| `Pod` | Smallest deployable unit — one or more containers running together | Namespaced |
| `Deployment` | Manages pods; ensures the right number are always running | Namespaced |
| `ReplicaSet` | Created automatically by a Deployment to maintain replica count | Namespaced |
| `Service` | Stable network endpoint to expose pods — survives pod restarts | Namespaced |
| `ConfigMap` | Store non-sensitive configuration data separately from containers | Namespaced |
| `Secret` | Store sensitive data (passwords, tokens) — base64 encoded | Namespaced |
| `PersistentVolume` | Durable storage that survives pod restarts | Cluster-wide |
| `PersistentVolumeClaim` | A request for storage by a pod | Namespaced |
| `Ingress` | HTTP/HTTPS routing rules to Services | Namespaced |
| `StatefulSet` | Like Deployment, but for stateful apps (databases) | Namespaced |
| `DaemonSet` | Run one pod per node (e.g. log collectors) | Namespaced |
| `Job` | Run a pod to completion (one-off tasks) | Namespaced |
| `CronJob` | Run a Job on a schedule | Namespaced |

### How kinds appear in YAML

```yaml
apiVersion: apps/v1      # Which API group and version handles this kind
kind: Deployment         # The resource type
metadata:
  name: my-app
  namespace: my-namespace
spec:
  ...
```

> **Think of it this way:** `kind` is like a class in programming. Each kind has its own set of fields (spec) and behaviour. Kubernetes reads the `kind` and knows exactly how to handle that resource.

---

## 3. Architecture Overview

```
                        INTERNET / USER
                              |
                         NodeIP:32678
                              |
              ┌───────────────────────────────────────────┐
              │       iron-namespace-datacenter            │
              │                                           │
              │  ┌─────────────────────────────────────┐  │
              │  │  iron-gallery-service-datacenter     │  │
              │  │  Type: NodePort | Port 80 → 32678    │  │
              │  └──────────────┬──────────────────────┘  │
              │                 │ selector: run=iron-gallery│
              │  ┌──────────────▼──────────────────────┐  │
              │  │  iron-gallery-deployment-datacenter  │  │
              │  │  ┌────────────────────────────────┐  │  │
              │  │  │  Pod (iron-gallery-container)  │  │  │
              │  │  │  image: kodekloud/irongallery  │  │  │
              │  │  │  limits: 100Mi RAM / 50m CPU   │  │  │
              │  │  │  vol: config → /html/data      │  │  │
              │  │  │  vol: images → /html/uploads   │  │  │
              │  │  └────────────────────────────────┘  │  │
              │  └──────────────────────────────────────┘  │
              │                                           │
              │  ┌──────────────────────────────────────┐  │
              │  │  iron-db-service-datacenter          │  │
              │  │  Type: ClusterIP | Port 3306         │  │
              │  └──────────────┬──────────────────────┘  │
              │                 │ selector: db=mariadb     │
              │  ┌──────────────▼──────────────────────┐  │
              │  │  iron-db-deployment-datacenter       │  │
              │  │  ┌────────────────────────────────┐  │  │
              │  │  │  Pod (iron-db-container)        │  │  │
              │  │  │  image: kodekloud/irondb:2.0   │  │  │
              │  │  │  env: MYSQL_DATABASE etc.      │  │  │
              │  │  │  vol: db → /var/lib/mysql      │  │  │
              │  │  └────────────────────────────────┘  │  │
              │  └──────────────────────────────────────┘  │
              └───────────────────────────────────────────┘
```

---

## 4. Step-by-Step Commands with Explanations

### Step 1 — Create the Namespace

```bash
kubectl create namespace iron-namespace-datacenter
```

**Expected output:**
```
namespace/iron-namespace-datacenter created
```

**Verify:**
```bash
kubectl get namespace iron-namespace-datacenter
```
```
NAME                        STATUS   AGE
iron-namespace-datacenter   Active   5s
```

> A **Namespace** is like a folder inside Kubernetes. All the resources for this app live inside it, isolated from resources in other namespaces.

---

### Step 2 — Create Iron Gallery Deployment

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-datacenter
  namespace: iron-namespace-datacenter
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
        - name: iron-gallery-container-datacenter
          image: kodekloud/irongallery:2.0
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data
            - name: images
              mountPath: /usr/share/nginx/html/uploads
      volumes:
        - name: config
          emptyDir: {}
        - name: images
          emptyDir: {}
EOF
```

**Expected output:**
```
deployment.apps/iron-gallery-deployment-datacenter created
```

**Key fields explained:**

| Field | Value | Why |
|-------|-------|-----|
| `replicas: 1` | 1 pod | Only one instance needed |
| `image` | `kodekloud/irongallery:2.0` | Exact image name — must match |
| `limits.memory` | `100Mi` | Max 100 mebibytes RAM the container can use |
| `limits.cpu` | `50m` | 50 millicores = 5% of one CPU core |
| `emptyDir: {}` | Empty volume | Temporary storage that lives with the pod |
| `mountPath` | `/usr/share/nginx/html/data` | Where the volume appears inside the container |

---

### Step 3 — Create Iron DB Deployment

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-datacenter
  namespace: iron-namespace-datacenter
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
        - name: iron-db-container-datacenter
          image: kodekloud/irondb:2.0
          env:
            - name: MYSQL_DATABASE
              value: "database_apache"
            - name: MYSQL_ROOT_PASSWORD
              value: "R00t@S3cur3P@ss!"
            - name: MYSQL_PASSWORD
              value: "Us3r@S3cur3P@ss!"
            - name: MYSQL_USER
              value: "irongallery_user"
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
      volumes:
        - name: db
          emptyDir: {}
EOF
```

**Expected output:**
```
deployment.apps/iron-db-deployment-datacenter created
```

**Environment variables explained:**

| Variable | Purpose |
|----------|---------|
| `MYSQL_DATABASE` | Name of the database to auto-create on startup |
| `MYSQL_ROOT_PASSWORD` | Password for the root (admin) MySQL user |
| `MYSQL_PASSWORD` | Password for the regular application user |
| `MYSQL_USER` | Username for the regular application user (never use `root`) |

> **Security note:** In production, passwords should be stored in a Kubernetes `Secret`, not hardcoded in the Deployment YAML.

---

### Step 4 — Create Iron DB Service (ClusterIP)

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-datacenter
  namespace: iron-namespace-datacenter
spec:
  type: ClusterIP
  selector:
    db: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
EOF
```

**Expected output:**
```
service/iron-db-service-datacenter created
```

> `ClusterIP` means this service is **only reachable from inside the cluster**. The database should never be exposed to the internet.

---

### Step 5 — Create Iron Gallery Service (NodePort)

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-datacenter
  namespace: iron-namespace-datacenter
spec:
  type: NodePort
  selector:
    run: iron-gallery
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32678
EOF
```

**Expected output:**
```
service/iron-gallery-service-datacenter created
```

> `NodePort` opens port `32678` on the **node's actual network interface**, making the app reachable from outside the cluster at `NodeIP:32678`.

---

### Step 6 — Verify Everything

```bash
kubectl get all -n iron-namespace-datacenter
```

**Expected output:**
```
NAME                                                       READY   STATUS    RESTARTS   AGE
pod/iron-db-deployment-datacenter-5c94b79ddc-sqnxl        1/1     Running   0          3m20s
pod/iron-gallery-deployment-datacenter-7f74bfd494-mqdvr   1/1     Running   0          5m36s

NAME                                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/iron-db-service-datacenter         ClusterIP   10.43.7.183    <none>        3306/TCP       2m14s
service/iron-gallery-service-datacenter    NodePort    10.43.53.63    <none>        80:32678/TCP   23s

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/iron-db-deployment-datacenter         1/1     1            1           3m20s
deployment.apps/iron-gallery-deployment-datacenter    1/1     1            1           5m36s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/iron-db-deployment-datacenter-5c94b79ddc       1         1         1       3m20s
replicaset.apps/iron-gallery-deployment-datacenter-7f74bfd494  1         1         1       5m36s
```

### Step 7 — Access the App

```bash
# Get the Node IP
kubectl get nodes -o wide

# Access the app
curl http://<NODE-INTERNAL-IP>:32678
```

In this task the Node IP was `10.244.195.9`, so:

```bash
curl http://10.244.195.9:32678
```

A successful response returns the HTML of the Iron Gallery installation page.

---

## 5. The EOF Heredoc Explained

A **heredoc** (here document) is a shell feature that lets you write multi-line text inline in a command, without creating a file.

### Syntax breakdown

```bash
cat <<EOF | kubectl apply -f -
... YAML content ...
EOF
```

| Part | Meaning |
|------|---------|
| `cat` | Outputs the text that follows |
| `<<EOF` | "Start collecting text until you see `EOF` alone on a line" |
| `\| kubectl apply -f -` | Pipe that output to kubectl; `-f -` means "read from stdin" |
| `EOF` (alone on last line) | Signals the end of the heredoc block |

### Why use it?

- No need to create a `.yaml` file on disk
- Everything in one command — fast and clean
- Great for quick deployments on remote servers
- The word `EOF` is just a convention — you could use any word (`END`, `YAML`, etc.) as long as both sides match

### Alternative: using a file

The same result can be achieved by writing a file and applying it:

```bash
# Write the file
cat > gallery-deployment.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
...
EOF

# Apply it
kubectl apply -f gallery-deployment.yaml
```

For production environments, saving YAML files is preferred because you can version-control them with Git.

---

## 6. Labels and Selectors — The Glue of Kubernetes

Labels are the most important concept for connecting Kubernetes resources together.

### How it works

```
Deployment (matchLabels)  ──matches──▶  Pod (labels)  ◀──selects──  Service (selector)
run: iron-gallery                        run: iron-gallery             run: iron-gallery
```

- The **Deployment's `matchLabels`** tells it which pods belong to it
- The **Pod template's `labels`** tag each pod with an identity
- The **Service's `selector`** finds pods to route traffic to, by matching labels

### In the YAML

```yaml
# Deployment
spec:
  selector:
    matchLabels:
      run: iron-gallery       # ← Deployment looks for pods with this label

  template:
    metadata:
      labels:
        run: iron-gallery     # ← Every pod gets this label stamped on it

---
# Service
spec:
  selector:
    run: iron-gallery         # ← Service routes to all pods with this label
```

> **If the labels don't match, the Service sends traffic nowhere and the app appears unreachable.**

---

## 7. Service Types — ClusterIP vs NodePort

### ClusterIP (used for Iron DB)

```
[Pod A] ──▶ [ClusterIP:3306] ──▶ [DB Pod]
                                  ✅ reachable from inside cluster
                                  ❌ not reachable from outside
```

- **Default** service type in Kubernetes
- Gets a virtual IP only visible inside the cluster
- Used for internal services like databases, caches, internal APIs
- More secure — never exposed to the internet

### NodePort (used for Iron Gallery)

```
[Browser] ──▶ [NodeIP:32678] ──▶ [NodePort Service] ──▶ [Gallery Pod]
               ✅ reachable from outside the cluster
```

- Opens a port (30000–32767) on every node's real network IP
- Traffic arriving at `NodeIP:NodePort` is forwarded to the service
- Used when you need external access without a load balancer
- The ClusterIP of the service (`10.43.53.63`) is **NOT** the right IP — always use the **Node's IP**

### IP confusion — a common beginner trap

```
kubectl get nodes -o wide      →  Node IP:    10.244.195.9   ← USE THIS for NodePort
kubectl get svc                →  ClusterIP:  10.43.53.63    ← only works inside cluster
```

| IP | Reachable from outside? |
|----|------------------------|
| Node IP (`10.244.195.9`) | ✅ Yes |
| ClusterIP (`10.43.53.63`) | ❌ No |

---

## 8. Common Beginner Mistakes

### 1. Using ClusterIP instead of Node IP for NodePort access

```bash
# WRONG ❌
curl http://10.43.53.63:32678   # ClusterIP — only works inside cluster

# CORRECT ✅
curl http://10.244.195.9:32678  # Node IP — works from outside
```

### 2. Label mismatch between Deployment and Service

```yaml
# Deployment pod template label
labels:
  run: iron-gallery

# Service selector — must match exactly
selector:
  run: iron-gallry   # ← typo! Service routes to no pods
```

### 3. Wrong namespace

```bash
# Pods exist but you can't see them because you forgot -n
kubectl get pods                          # looks in 'default' namespace
kubectl get pods -n iron-namespace-datacenter  # correct
```

### 4. Hardcoding passwords in YAML (bad practice)

```yaml
# NOT recommended for production
env:
  - name: MYSQL_ROOT_PASSWORD
    value: "mypassword"    # visible to anyone who can read the Deployment

# Better practice — use a Secret
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: root-password
```

### 5. emptyDir volumes lose data on pod restart

`emptyDir` is tied to the pod's lifetime. If the pod restarts or is rescheduled, all data in `emptyDir` is gone. For real persistence, use a `PersistentVolumeClaim`.

---

## 9. Verification Commands

```bash
# See all resources in the namespace
kubectl get all -n iron-namespace-datacenter

# Detailed info about a deployment
kubectl describe deployment iron-gallery-deployment-datacenter -n iron-namespace-datacenter

# Check pod logs (useful for debugging)
kubectl logs <pod-name> -n iron-namespace-datacenter

# Check node IPs
kubectl get nodes -o wide

# Describe a service
kubectl describe service iron-gallery-service-datacenter -n iron-namespace-datacenter

# Check events (great for debugging why pods won't start)
kubectl get events -n iron-namespace-datacenter --sort-by='.lastTimestamp'

# Execute a command inside a running pod
kubectl exec -it <pod-name> -n iron-namespace-datacenter -- /bin/sh
```

---

## 10. Key Learning Points Summary

| # | Concept | What to Remember |
|---|---------|-----------------|
| 1 | **Kubernetes kinds** | Called Resource Types or API Objects. Each `kind` is a blueprint for a resource k8s manages. |
| 2 | **Namespace** | Logical boundary. All resources in this task lived in `iron-namespace-datacenter`. |
| 3 | **Deployment** | Manages pods and keeps them running. Creates a ReplicaSet automatically. |
| 4 | **ReplicaSet** | Created by Deployment. Ensures N pods are always alive. You usually don't create these manually. |
| 5 | **Pod** | One running instance. Contains the actual container(s). |
| 6 | **Labels** | Key-value tags on resources. Used by selectors to connect Deployments → Pods → Services. |
| 7 | **emptyDir volume** | Temporary storage. Lives and dies with the pod. Not for persistent data. |
| 8 | **Resource limits** | `50m` CPU = 5% of one core. `100Mi` memory = ~105 MB. Prevents one container from consuming all node resources. |
| 9 | **ClusterIP service** | Internal only. Used for DB — other pods can reach it, the internet cannot. |
| 10 | **NodePort service** | Opens a real port on the node. Used for the web app — reachable from outside. |
| 11 | **EOF heredoc** | Shell trick to write YAML inline and pipe it to kubectl, no file needed. |
| 12 | **Node IP vs ClusterIP** | For NodePort access, always use the node's real IP from `kubectl get nodes -o wide`. |
| 13 | **Environment variables** | Pass config into containers. In production use `Secret` objects, not plain text values. |
| 14 | **kubectl get all** | Your best friend for checking the state of everything in a namespace at once. |

---

> **Next steps to explore:**
> - Replace `emptyDir` with `PersistentVolumeClaims` for real data persistence
> - Move DB passwords to a Kubernetes `Secret`
> - Add an `Ingress` resource for proper domain-based HTTP routing
> - Explore `kubectl describe` and `kubectl logs` for debugging
> - Learn `kubectl rollout` for zero-downtime deployments
