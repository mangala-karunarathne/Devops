# Kubernetes Guestbook App Deployment — Study Material
> **Task:** Nautilus Application — Guestbook App on Kubernetes  
> **Level:** Beginner to Intermediate  
> **Stack:** Redis (Master/Slave) + PHP Frontend on Kubernetes

---

## 📋 Original Task

### BACK-END TIER

1. Create a deployment named `redis-master` for Redis master.
   - Replicas count should be `1`
   - Container name should be `master-redis-xfusion` and it should use image `redis`
   - Request resources: `CPU` → `100m`, `Memory` → `100Mi`
   - Container port should be Redis default port i.e `6379`

2. Create a service named `redis-master` for Redis master.
   - Port and targetPort should be Redis default port i.e `6379`

3. Create another deployment named `redis-slave` for Redis slave.
   - Replicas count should be `2`
   - Container name should be `slave-redis-xfusion` and it should use `gcr.io/google_samples/gb-redisslave:v3` image
   - Request resources: `CPU` → `100m`, `Memory` → `100Mi`
   - Define an environment variable named `GET_HOSTS_FROM` and its value should be `dns`
   - Container port should be Redis default port i.e `6379`

4. Create another service named `redis-slave`.
   - It should use Redis default port i.e `6379`

### FRONT-END TIER

1. Create a deployment named `frontend`.
   - Replicas count should be `3`
   - Container name should be `php-redis-xfusion` and it should use `gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff` image
   - Request resources: `CPU` → `100m`, `Memory` → `100Mi`
   - Define an environment variable named `GET_HOSTS_FROM` and its value should be `dns`
   - Container port should be `80`

2. Create a service named `frontend`.
   - Type should be `NodePort`
   - Port should be `80`
   - nodePort should be `30009`

---

## 🏗️ Architecture Overview

### 3-Tier Architecture

```
Tier 1 — PRESENTATION  →  Frontend (PHP app)        port 80
Tier 2 — APPLICATION   →  (PHP handles logic too)
Tier 3 — DATA          →  Redis Master + Slave       port 6379
```

### Full Component Flow

```
OUTSIDE WORLD
     │
     │  HTTP :30009
     ▼
┌─────────────────────┐
│   frontend Service  │  ← NodePort Service
│   (NodePort)        │    exposes Node IP:30009
│   ClusterIP:80      │    routes to port 80
└─────────┬───────────┘
          │ selector: app=guestbook
          ▼
┌─────────────────────┐
│   frontend Pods     │  ← 3 replicas (PHP app)
│   (3 replicas)      │    env: GET_HOSTS_FROM=dns
│   containerPort:80  │    uses DNS to find Redis
└──────┬──────────────┘
       │
       │  DNS lookup: "redis-master" / "redis-slave"
       │  CoreDNS resolves Service name → ClusterIP
       │
       ├──────────────────────────────────┐
       ▼                                  ▼
┌─────────────────┐            ┌─────────────────────┐
│ redis-master    │            │  redis-slave Svc     │
│ Service         │            │  (ClusterIP)         │
│ (ClusterIP:6379)│            │  ClusterIP:6379      │
└────────┬────────┘            └──────────┬──────────┘
         │ selector: role=master          │ selector: role=slave
         ▼                                ▼
┌─────────────────┐            ┌─────────────────────┐
│ redis-master Pod│            │  redis-slave Pods    │
│ (1 replica)     │            │  (2 replicas)        │
│ WRITE operations│            │  READ operations     │
│ port: 6379      │            │  syncs from master   │
└─────────────────┘            └─────────────────────┘
```

---

## 🔑 Core Kubernetes Concepts

### Pod
- The **smallest deployable unit** in Kubernetes
- One or more containers running together
- Has a temporary IP — changes every restart
- Think of it as: **one running instance of your app**

### Deployment
- A **manager** that ensures N copies of a pod always run
- If a pod crashes, Deployment automatically restarts it
- Handles rolling updates and rollbacks
- Think of it as: **"always keep 3 frontend pods running"**

```yaml
spec:
  replicas: 3          # always maintain 3 pods
  selector:
    matchLabels:
      app: guestbook   # manage pods with this label
```

### Service
- A **stable network endpoint** (fixed IP + DNS name) for pods
- Pods have changing IPs — Service gives a permanent address
- Routes traffic to matching pods via **label selectors**
- Think of it as: **a load balancer / fixed hostname for your pods**

```yaml
selector:
  app: guestbook       # route to pods with this label
ports:
  - port: 80           # Service listens here
    targetPort: 80     # forwards to pod's port
```

### Labels & Selectors
- Labels are **key-value tags** on pods
- Selectors let Services **find the right pods**
- This is how Services and Deployments are linked

```yaml
# Deployment gives pods this label
labels:
  app: guestbook
  tier: frontend

# Service uses selector to find those pods
selector:
  app: guestbook
  tier: frontend
```

---

## 🌐 IP Types in Kubernetes

### Building Analogy

```
┌─────────────────────────────────────────────────┐
│           Kubernetes Node (The Building)         │
│   Node IP: 10.244.97.190                        │
│   ← BUILDING's street address (externally       │
│     reachable within the network)               │
│                                                 │
│   ┌─────────────────────────────────────────┐   │
│   │      Cluster Network (The Floors)        │   │
│   │                                         │   │
│   │  frontend Service  ClusterIP:10.43.x.x  │   │
│   │  redis-master Svc  ClusterIP:10.43.x.x  │   │
│   │  ← FLOOR numbers: only valid INSIDE      │   │
│   │    the building (cluster)               │   │
│   │                                         │   │
│   │  ┌──────────────────────────────────┐   │   │
│   │  │         Pods (The Rooms)          │   │   │
│   │  │  frontend-pod-1  172.16.x.x      │   │   │
│   │  │  redis-master    172.16.x.x      │   │   │
│   │  │  ← ROOM numbers: temporary,       │   │   │
│   │  │    change on every restart       │   │   │
│   │  └──────────────────────────────────┘   │   │
│   └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

### IP Reference Table

| IP Type | Example | Reachable From | Changes? | Use For |
|---|---|---|---|---|
| **Node IP** | `10.244.97.190` | Anywhere in network | No | External access via NodePort |
| **ClusterIP** | `10.43.127.21` | Inside cluster only | No (stable) | Internal service-to-service |
| **Pod IP** | `172.16.x.x` | Inside cluster only | Yes (every restart) | Never use directly |

---

## 🔌 Service Types

### ClusterIP (Default)
- Only accessible **inside** the cluster
- Used for internal communication (e.g., frontend → redis)
- Kubernetes auto-assigns a virtual IP

```yaml
spec:
  type: ClusterIP      # default, can be omitted
  ports:
    - port: 6379
      targetPort: 6379
```

### NodePort
- Opens a **specific port on every Node**
- Accessible from outside the cluster via `NodeIP:NodePort`
- Port range: `30000–32767`

```yaml
spec:
  type: NodePort
  ports:
    - port: 80          # ClusterIP port (internal)
      targetPort: 80    # Pod's container port
      nodePort: 30009   # External port on Node
```

### Access Pattern
```
External User → NodeIP:30009 → Service:80 → Pod:80
```

---

## 🧬 DNS in Kubernetes (CoreDNS)

### Why GET_HOSTS_FROM=dns?
Kubernetes has a built-in DNS server (**CoreDNS**) that automatically registers every Service by name.

```
# Instead of hardcoding IPs (BAD):
redis.connect('10.43.187.238', 6379)  # breaks if IP changes

# Use DNS name (GOOD):
redis.connect('redis-master', 6379)   # always resolves correctly
```

### DNS Resolution Flow
```
PHP app calls: redis-master:6379
       │
       ▼
CoreDNS lookup: "redis-master"
       │
       ▼
Returns ClusterIP: 10.43.187.238
       │
       ▼
Traffic routed to redis-master Pod
```

### Full DNS name format in Kubernetes:
```
<service-name>.<namespace>.svc.cluster.local
redis-master.default.svc.cluster.local

# Within same namespace, short name works:
redis-master
```

---

## 📦 Resource Requests

```yaml
resources:
  requests:
    cpu: 100m       # 100 millicores = 0.1 CPU core
    memory: 100Mi   # 100 Mebibytes RAM
```

| Value | Meaning |
|---|---|
| `100m` CPU | 1/10th of a CPU core (m = millicores) |
| `1000m` CPU | 1 full CPU core |
| `100Mi` Memory | ~104 MB RAM |
| `1Gi` Memory | ~1.07 GB RAM |

> **requests** = minimum guaranteed resources  
> **limits** = maximum allowed resources (not set here)

---

## 📝 Complete YAML Manifests

### 1. Redis Master Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
      - name: master-redis-xfusion
        image: redis
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
```

### 2. Redis Master Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
spec:
  selector:
    app: redis
    role: master
  ports:
  - port: 6379
    targetPort: 6379
```

### 3. Redis Slave Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
  template:
    metadata:
      labels:
        app: redis
        role: slave
    spec:
      containers:
      - name: slave-redis-xfusion
        image: gcr.io/google_samples/gb-redisslave:v3
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
```

### 4. Redis Slave Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
spec:
  selector:
    app: redis
    role: slave
  ports:
  - port: 6379
    targetPort: 6379
```

### 5. Frontend Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis-xfusion
        image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
```

### 6. Frontend Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: guestbook
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30009
```

---

## 🛠️ Essential kubectl Commands

### Verification Commands
```bash
# Check all deployments
kubectl get deployments

# Check all pods with details
kubectl get pods -o wide

# Check all services
kubectl get services

# Check node IPs
kubectl get nodes -o wide
```

### Debugging Commands
```bash
# Describe a deployment (check events, config)
kubectl describe deployment <name>

# Describe a pod (check why it's failing)
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# View logs of a specific container in pod
kubectl logs <pod-name> -c <container-name>

# Execute into a running pod
kubectl exec -it <pod-name> -- /bin/bash
```

### Apply & Delete
```bash
# Apply a manifest file
kubectl apply -f filename.yaml

# Delete a resource
kubectl delete deployment <name>
kubectl delete service <name>

# Delete using manifest file
kubectl delete -f filename.yaml
```

---

## ❌ Mistakes Made & Lessons Learned

### Mistake 1: Typing URL directly in terminal
```bash
# WRONG - bash treats URL as a command
thor@jump-host ~$ http://10.43.127.21:30009
bash: http://10.43.127.21:30009: No such file or directory

# CORRECT - use curl
curl http://10.244.97.190:30009
```

### Mistake 2: Using ClusterIP for external access
```bash
# WRONG - ClusterIP is internal only, curl hangs
curl http://10.43.127.21:30009

# CORRECT - use Node IP with NodePort
curl http://10.244.97.190:30009
#           ↑ Node IP        ↑ NodePort
```

---

## 📊 Final Summary Table

| Resource | Name | Kind | Replicas | Image | Port | Type |
|---|---|---|---|---|---|---|
| Redis Master | `redis-master` | Deployment | 1 | `redis` | 6379 | — |
| Redis Master | `redis-master` | Service | — | — | 6379 | ClusterIP |
| Redis Slave | `redis-slave` | Deployment | 2 | `gb-redisslave:v3` | 6379 | — |
| Redis Slave | `redis-slave` | Service | — | — | 6379 | ClusterIP |
| Frontend | `frontend` | Deployment | 3 | `gb-frontend` | 80 | — |
| Frontend | `frontend` | Service | — | — | 80:30009 | NodePort |

---

## 🔗 Further Reading

- [Kubernetes Official Docs — Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Official Docs — Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes Official Docs — DNS for Services](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Kubernetes Official Docs — Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Original Guestbook Example](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/)

---

*Study material generated from hands-on task walkthrough — Nautilus Guestbook App Deployment*
