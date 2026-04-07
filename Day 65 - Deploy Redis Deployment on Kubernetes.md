# 📚 Redis on Kubernetes — Complete Study Material
> **Task:** Deploy Redis on a Kubernetes cluster for the Nautilus application team  
> **Environment:** Single-node k3s cluster (jump-host)  
> **Date:** April 2026

---

## 📋 Table of Contents
1. [Original Task](#original-task)
2. [Prerequisites & Concepts](#prerequisites--concepts)
3. [Step-by-Step Implementation](#step-by-step-implementation)
4. [Full YAML Reference](#full-yaml-reference)
5. [Verification Commands](#verification-commands)
6. [Key Learning Points](#key-learning-points)
7. [CPU & Resources Deep Dive](#cpu--resources-deep-dive)
8. [Kubernetes Cost & Cloud Pricing](#kubernetes-cost--cloud-pricing)
9. [Quick Reference Cheatsheet](#quick-reference-cheatsheet)

---

## 📌 Original Task

> The Nautilus application development team observed some performance issues with one of the applications deployed in a Kubernetes cluster. After looking into a number of factors, the team suggested using an in-memory caching utility for the DB service. After discussions, they decided to use **Redis**.
>
> Initially they would like to deploy Redis on the Kubernetes cluster for **testing**, and later move it to **production**.

### Requirements:
1. Create a **ConfigMap** called `my-redis-config` having `maxmemory 2mb` in `redis-config`.
2. **Deployment** name: `redis-deployment`, image: `redis:alpine`, container name: `redis-container`, replicas: `1`.
3. Container should request `1` CPU.
4. Mount **2 volumes**:
   - An **EmptyDir** volume called `data` at path `/redis-master-data`
   - A **ConfigMap** volume called `redis-config` at path `/redis-master`
5. Container should expose port `6379`.
6. `redis-deployment` should be **up and running**.

---

## 🧠 Prerequisites & Concepts

### What is Kubernetes (K8s)?
Kubernetes is a **container orchestration platform** that manages, scales, and maintains containerized applications automatically.

### What is k3s?
- A **lightweight version of Kubernetes** designed for single-node setups, testing, and edge environments.
- In this task, the jump-host itself acts as **both control plane and worker node**.
- Status `Ready` on a single node is perfectly valid.

```
NAME        STATUS   ROLES           AGE   VERSION
jump-host   Ready    control-plane   25m   v1.34.1+k3s1
```

### What is Redis?
- Redis is an **in-memory data store** used as a cache, message broker, or database.
- It stores data in **RAM**, making reads/writes extremely fast.
- Used to reduce load on the main database (DB service in this case).

### What is a ConfigMap?
- A Kubernetes object to **store non-sensitive configuration data** as key-value pairs.
- **Decouples configuration from the container image** — no need to hardcode config inside the image.
- Can be mounted as a **file inside the container** or injected as environment variables.

> 💡 Think of ConfigMap as a **settings file** that Kubernetes passes to your container.

### What is a Deployment?
- Tells Kubernetes **what to run, how many copies, and how to run it**.
- Manages **desired state** — if a pod crashes, the Deployment automatically restarts it.
- Always creates a **ReplicaSet** and **Pods** underneath.

### Kubernetes Object Hierarchy
```
Deployment
    └── ReplicaSet        (auto-created — manages replicas)
            └── Pod       (auto-created — runs the container)
                    └── Container  (your actual Redis app)
```
> 💡 You only create the **Deployment** — Kubernetes automatically creates the ReplicaSet and Pod!

### What are Volumes?
Two types used in this task:

| Type | Name | Purpose |
|---|---|---|
| `emptyDir` | `data` | Temporary storage — lives as long as the pod. Good for scratch space. |
| `configMap` | `redis-config` | Mounts ConfigMap data as files inside the container. |

### Labels and Selectors
- **Labels** are key-value tags attached to Kubernetes objects.
- **Selectors** filter objects by their labels.

```bash
kubectl get pods -l app=redis   # -l flag means "filter by label"
```

> 💡 Labels are how Kubernetes objects **find and talk to each other**.

---

## 🛠️ Step-by-Step Implementation

### Step 1: Verify Cluster Access

```bash
kubectl get nodes
```

**Expected Output:**
```
NAME        STATUS   ROLES           AGE   VERSION
jump-host   Ready    control-plane   25m   v1.34.1+k3s1
```
✅ Single-node k3s cluster — this is perfectly fine for testing.

---

### Step 2: Create the ConfigMap

```bash
kubectl create configmap my-redis-config --from-literal=redis-config="maxmemory 2mb"
```

**Expected Output:**
```
configmap/my-redis-config created
```

**Verify the ConfigMap:**
```bash
kubectl describe configmap my-redis-config
```

**Expected Output:**
```
Name:         my-redis-config
Namespace:    default
Data
====
redis-config:
----
maxmemory 2mb
```

---

### Step 3: Create the Deployment YAML

Use the EOF method (most convenient — no editor needed):

```bash
cat <<EOF > redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis-container
          image: redis:alpine
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: "1"
          volumeMounts:
            - name: data
              mountPath: /redis-master-data
            - name: redis-config
              mountPath: /redis-master
      volumes:
        - name: data
          emptyDir: {}
        - name: redis-config
          configMap:
            name: my-redis-config
EOF
```

> **3 ways to create a file in Linux:**
> - `vi redis-deployment.yaml` — open with vi editor
> - `cat <<EOF > file.yaml ... EOF` — write directly from terminal (recommended)
> - `mkdir + vi` — create a folder first, then edit inside it

---

### Step 4: Apply the Deployment

```bash
kubectl apply -f redis-deployment.yaml
```

**Expected Output:**
```
deployment.apps/redis-deployment created
```

---

### Step 5: Verify Everything is Running

**Check deployment:**
```bash
kubectl get deployment redis-deployment
```
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           45s
```

**Check pods:**
```bash
kubectl get pods -l app=redis
```
```
NAME                                READY   STATUS    RESTARTS   AGE
redis-deployment-6d8f9b7c4d-xk2pq   1/1     Running   0          60s
```

---

### Step 6: Deep Verification — Config Inside Container

**Capture the pod name into a variable:**
```bash
POD_NAME=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
```
> ⚠️ This command stores the pod name silently — no output is normal!

**Print the variable to confirm:**
```bash
echo $POD_NAME
# Output: redis-deployment-6d8f9b7c4d-xk2pq
```

**Check config is correctly mounted inside the container:**
```bash
kubectl exec -it $POD_NAME -- cat /redis-master/redis-config
```

**Expected Output (confirms task complete ✅):**
```
maxmemory 2mb
```

---

## 📄 Full YAML Reference

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment        # Deployment name
  labels:
    app: redis
spec:
  replicas: 1                   # Only 1 pod
  selector:
    matchLabels:
      app: redis                # Must match pod labels below
  template:
    metadata:
      labels:
        app: redis              # Pod label
    spec:
      containers:
        - name: redis-container # Container name
          image: redis:alpine   # Docker image
          ports:
            - containerPort: 6379  # Redis default port
          resources:
            requests:
              cpu: "1"          # Request 1 full CPU core
          volumeMounts:
            - name: data
              mountPath: /redis-master-data   # EmptyDir mount path
            - name: redis-config
              mountPath: /redis-master        # ConfigMap mount path
      volumes:
        - name: data
          emptyDir: {}          # Temporary scratch volume
        - name: redis-config
          configMap:
            name: my-redis-config  # References the ConfigMap we created
```

---

## ✅ Verification Commands

| Command | Purpose |
|---|---|
| `kubectl get nodes` | Check cluster and node status |
| `kubectl create configmap <name> --from-literal=key=value` | Create a ConfigMap |
| `kubectl describe configmap <name>` | Inspect ConfigMap details |
| `kubectl apply -f file.yaml` | Create or update resources from YAML |
| `kubectl get deployment <name>` | Check deployment status |
| `kubectl get pods -l app=redis` | List pods filtered by label |
| `kubectl describe deployment <name>` | Full deployment details |
| `kubectl exec -it <pod> -- <command>` | Run command inside a running pod |
| `echo $POD_NAME` | Print a stored shell variable |

---

## 🎓 Key Learning Points

### 1. YAML is the Language of Kubernetes
Every resource in Kubernetes is defined in YAML. Key fields:
- `apiVersion` — which Kubernetes API to use
- `kind` — type of object (Deployment, ConfigMap, Service, etc.)
- `metadata` — name and labels
- `spec` — the actual desired configuration

### 2. Pod Naming Convention
Pod names are **auto-generated** from the Deployment name:
```
redis-deployment  -  6d8f9b7c4d  -  xk2pq
      ↑                   ↑            ↑
 Deployment name    ReplicaSet hash  Pod hash
```

### 3. Variable Assignment in Bash
```bash
POD_NAME=$(command)   # Stores output silently — no output is expected
echo $POD_NAME        # Prints the stored value
```

### 4. Pod States
| Status | CPU Usage | Meaning |
|---|---|---|
| `Pending` | ❌ No | Waiting — not enough resources |
| `Running` | ✅ Yes | Active and consuming resources |
| `Succeeded` | ❌ No | Finished (batch jobs) |
| `Failed` | ❌ No | Crashed |
| `Terminating` | 🔻 Minimal | Shutting down |

### 5. The Golden Rule of Kubernetes
> You describe **what you want** — Kubernetes figures out **how to make it happen**. That's the power of **declarative configuration**!

---

## 💻 CPU & Resources Deep Dive

### How CPU is Measured in Kubernetes
CPU is measured in **millicores (m)**:

| You Write | Meaning |
|---|---|
| `"1"` | 1 full CPU core = 1000 millicores |
| `"0.5"` | Half a CPU core = 500 millicores |
| `"250m"` | Quarter CPU core = 250 millicores |
| `"100m"` | 1/10th of a CPU core |

> 💡 Think of millicores like **slices of a pizza** — a 4-CPU node has 4000m to share among all pods.

### Requests vs Limits
```yaml
resources:
  requests:
    cpu: "250m"    # Minimum guaranteed CPU (Kubernetes reserves this)
  limits:
    cpu: "500m"    # Hard cap — container cannot exceed this
```

| Field | Meaning |
|---|---|
| `requests` | **Guaranteed** minimum — Kubernetes uses this for scheduling |
| `limits` | **Hard cap** — container is throttled/killed if exceeded |

### Rules for CPU Allocation
- You **cannot request more** than the node has available
- If you request too much → pod stays in **Pending** state ⚠️
- If no limits are set → a pod can consume all available CPU (risky!)

### Real-World CPU Use Cases
| Application | Recommended CPU Request | Why |
|---|---|---|
| Redis (cache) | `250m` – `1` | Fast in-memory operations |
| Simple web API | `100m` – `250m` | Light processing |
| ML model serving | `2` – `4` | Heavy computation |
| MySQL Database | `500m` – `2` | Query processing |
| Nginx web server | `100m` – `250m` | Mostly I/O bound |

---

## ☁️ Kubernetes Cost & Cloud Pricing

### Who Do You Pay?
```
You NEVER pay for Kubernetes itself (it's open source)
        ↓
You pay for the SERVERS (nodes) running underneath it
        ↓
Kubernetes is just software running ON TOP of those servers
```

### Cost Layers
```
☁️  Cloud Provider (AWS / GCP / Azure)
        ↓  You pay for:
    ├── Virtual Machines (nodes/servers)
    ├── Storage (disks, volumes)
    ├── Networking (data transfer)
    └── Load Balancers
        ↓
    Kubernetes runs ON TOP of those VMs (free)
        ↓
    Your pods run inside Kubernetes
```

### Approximate Cloud Pricing

**AWS:**
| Node Type | CPU | RAM | Cost/month |
|---|---|---|---|
| t3.small | 2 vCPU | 2GB | ~$15 |
| t3.medium | 2 vCPU | 4GB | ~$30 |
| t3.large | 2 vCPU | 8GB | ~$60 |
| m5.xlarge | 4 vCPU | 16GB | ~$140 |

**GCP:**
| Node Type | CPU | RAM | Cost/month |
|---|---|---|---|
| e2-small | 2 vCPU | 2GB | ~$13 |
| e2-medium | 2 vCPU | 4GB | ~$27 |
| e2-standard-4 | 4 vCPU | 16GB | ~$97 |

**Azure:**
| Node Type | CPU | RAM | Cost/month |
|---|---|---|---|
| B2s | 2 vCPU | 4GB | ~$30 |
| D2s v3 | 2 vCPU | 8GB | ~$70 |

### Managed Kubernetes Services
| Service | Provider | Control Plane Cost |
|---|---|---|
| EKS | AWS | ~$73/month |
| GKE | Google | Free (Standard) |
| AKS | Azure | Free |

> 💡 With managed K8s, you still pay for worker nodes (VMs) + optionally for the control plane.

### Cost Optimization Tips
1. **Right-size your pods** — don't request more CPU/RAM than needed
2. **Use spot/preemptible VMs** — up to 90% cheaper (can be terminated anytime)
3. **Auto-scaling** — scale down when traffic is low
4. **Namespace resource quotas** — prevent teams from over-requesting resources
5. **Monitor usage** — use tools like Prometheus, Grafana, or Datadog

---

## 📎 Quick Reference Cheatsheet

### Resources Created in This Task
| Resource | Name | Details |
|---|---|---|
| ConfigMap | `my-redis-config` | Key: `redis-config`, Value: `maxmemory 2mb` |
| Deployment | `redis-deployment` | Image: `redis:alpine`, Replicas: 1 |
| Container | `redis-container` | Port: 6379, CPU Request: 1 |
| Volume 1 | `data` | EmptyDir → `/redis-master-data` |
| Volume 2 | `redis-config` | ConfigMap → `/redis-master` |

### Big Picture Flow
```
ConfigMap (config data)
      +
Deployment YAML (desired state)
      ↓
Kubernetes creates:
      ├── ReplicaSet
      └── Pod
            └── Container (Redis running!)
                    └── Volumes mounted
                          ├── /redis-master-data  (emptyDir)
                          └── /redis-master        (configMap) ✅
```

### Final Verification Output = Task Complete ✅
```bash
kubectl exec -it $POD_NAME -- cat /redis-master/redis-config
# Output:
maxmemory 2mb
```

---

*Study material compiled from hands-on task execution — Redis on Kubernetes (k3s), April 2026*
