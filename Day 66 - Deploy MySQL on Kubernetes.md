# 📘 MySQL Deployment on Kubernetes — Study Material

> **Topic:** Deploying a stateful MySQL application on Kubernetes using Secrets, PersistentVolumes, Deployments, and Services
> **Level:** Intermediate DevOps / Kubernetes
> **Author:** Study notes compiled from hands-on lab

---

## 📋 Table of Contents

1. [Original Task](#original-task)
2. [Concepts Overview](#concepts-overview)
3. [Step-by-Step Implementation](#step-by-step-implementation)
4. [Verification Commands](#verification-commands)
5. [Real World Use Case](#real-world-use-case)
6. [Architecture Diagram](#architecture-diagram)
7. [Troubleshooting Guide](#troubleshooting-guide)
8. [Key Takeaways](#key-takeaways)
9. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## 📌 Original Task

> A new MySQL server needs to be deployed on a Kubernetes cluster. The Nautilus DevOps team finalized the following requirements:

1. Create a **PersistentVolume** named `mysql-pv` with capacity `250Mi`.
2. Create a **PersistentVolumeClaim** named `mysql-pv-claim` requesting `250Mi` of storage.
3. Create a **Deployment** named `mysql-deployment` using a MySQL image. Mount the PersistentVolume at `/var/lib/mysql`.
4. Create a **NodePort** type Service named `mysql` with `nodePort: 30007`.
5. Create the following **Secrets**:
   - `mysql-root-pass` → key: `password`, value: `YUIidhb667`
   - `mysql-user-pass` → key: `username`, value: `kodekloud_gem` | key: `password`, value: `LQfKeWWxWD`
   - `mysql-db-url` → key: `database`, value: `kodekloud_db8`
6. Define the following **environment variables** in the container using `secretKeyRef`:
   - `MYSQL_ROOT_PASSWORD` → from `mysql-root-pass` → key `password`
   - `MYSQL_DATABASE` → from `mysql-db-url` → key `database`
   - `MYSQL_USER` → from `mysql-user-pass` → key `username`
   - `MYSQL_PASSWORD` → from `mysql-user-pass` → key `password`

---

## 🧠 Concepts Overview

### 1. Kubernetes Secrets

A **Secret** is a Kubernetes object used to store sensitive data such as passwords, tokens, and keys.

- Data is stored **base64-encoded** (not encrypted by default, but can be encrypted at rest)
- Referenced in pods via `secretKeyRef` (env vars) or `volumeMount` (files)
- Decouples sensitive config from application code and container images

**Types of Secrets:**
| Type | Use Case |
|------|----------|
| `Opaque` | Generic key-value secrets (used in this task) |
| `kubernetes.io/tls` | TLS certificates |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials |

---

### 2. PersistentVolume (PV)

A **PersistentVolume** is a piece of storage in the cluster that has been provisioned by an administrator or dynamically by a StorageClass.

- Exists independently of any pod
- Has its own lifecycle — survives pod deletion
- Defined with capacity, access modes, and reclaim policy

**Access Modes:**
| Mode | Description |
|------|-------------|
| `ReadWriteOnce (RWO)` | Mounted read-write by a single node |
| `ReadOnlyMany (ROX)` | Mounted read-only by many nodes |
| `ReadWriteMany (RWX)` | Mounted read-write by many nodes |

**Reclaim Policies:**
| Policy | Behaviour after PVC deletion |
|--------|-------------------------------|
| `Retain` | Data kept, manual cleanup needed |
| `Delete` | Volume and data deleted automatically |
| `Recycle` | Data scrubbed, volume made available again |

---

### 3. PersistentVolumeClaim (PVC)

A **PersistentVolumeClaim** is a request for storage by a user/pod.

- Binds to a matching PV based on `storageClassName`, `accessMode`, and `capacity`
- Once bound, the pod uses the PVC — it does not interact with the PV directly
- Status moves from `Pending` → `Bound` once a matching PV is found

---

### 4. Deployment

A **Deployment** manages a set of identical pods and ensures the desired number of replicas are always running.

- Provides **self-healing**: restarts crashed pods automatically
- Supports **rolling updates** and **rollbacks**
- Uses a **Pod template** to define the container spec

---

### 5. NodePort Service

A **Service** is an abstraction that exposes pods to network traffic.

**Service Types:**
| Type | Description |
|------|-------------|
| `ClusterIP` | Internal cluster access only (default) |
| `NodePort` | Exposes on a static port on each node's IP |
| `LoadBalancer` | External load balancer (cloud environments) |
| `ExternalName` | Maps to an external DNS name |

NodePort range: **30000–32767**

---

## 🛠️ Step-by-Step Implementation

### Step 1: Create Secrets

```bash
# Root password secret
kubectl create secret generic mysql-root-pass \
  --from-literal=password=YUIidhb667

# User credentials secret
kubectl create secret generic mysql-user-pass \
  --from-literal=username=kodekloud_gem \
  --from-literal=password=LQfKeWWxWD

# Database name secret
kubectl create secret generic mysql-db-url \
  --from-literal=database=kodekloud_db8
```

**Expected Output:**
```
secret/mysql-root-pass created
secret/mysql-user-pass created
secret/mysql-db-url created
```

---

### Step 2: Create PersistentVolume

```yaml
# mysql-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/mysql-pv
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
```

```bash
kubectl apply -f mysql-pv.yaml
# OR inline:
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/mysql-pv
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
EOF
```

**Expected Output:**
```
persistentvolume/mysql-pv created
```

---

### Step 3: Create PersistentVolumeClaim

```yaml
# mysql-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
  storageClassName: manual
```

```bash
kubectl apply -f mysql-pvc.yaml
```

**Expected Output:**
```
persistentvolumeclaim/mysql-pv-claim created
```

> ⚠️ **Important:** The `storageClassName` in PVC **must match** the PV's `storageClassName` for binding to occur.

---

### Step 4: Create Deployment

```yaml
# mysql-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

```bash
kubectl apply -f mysql-deployment.yaml
```

**Expected Output:**
```
deployment.apps/mysql-deployment created
```

---

### Step 5: Create NodePort Service

```yaml
# mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30007
```

```bash
kubectl apply -f mysql-service.yaml
```

**Expected Output:**
```
service/mysql created
```

---

## ✅ Verification Commands

```bash
# Check all resources at once
kubectl get pv,pvc,deployment,pods,svc,secrets

# Check PV status (should be Bound)
kubectl get pv mysql-pv

# Check PVC status (should be Bound)
kubectl get pvc mysql-pv-claim

# Check pod status (should be Running)
kubectl get pods -l app=mysql

# Check service (should show 3306:30007/TCP)
kubectl get svc mysql

# Verify environment variables inside the pod
kubectl exec -it <pod-name> -- env | grep MYSQL

# Check MySQL logs
kubectl logs <pod-name>

# Describe a resource for detailed info
kubectl describe deployment mysql-deployment
kubectl describe pod <pod-name>
kubectl describe pvc mysql-pv-claim
```

**Final expected state:**
```
NAME                        CAPACITY   STATUS   CLAIM
persistentvolume/mysql-pv   250Mi      Bound    default/mysql-pv-claim

NAME                                   STATUS   VOLUME
persistentvolumeclaim/mysql-pv-claim   Bound    mysql-pv

NAME                               READY   UP-TO-DATE   AVAILABLE
deployment.apps/mysql-deployment   1/1     1            1

NAME                                    READY   STATUS
pod/mysql-deployment-xxxx-xxxx          1/1     Running

NAME            TYPE       PORT(S)
service/mysql   NodePort   3306:30007/TCP

NAME                    TYPE     DATA
secret/mysql-db-url     Opaque   1
secret/mysql-root-pass  Opaque   1
secret/mysql-user-pass  Opaque   2
```

---

## 🌍 Real World Use Case

### The Scenario

A company building a SaaS web application needs a MySQL database that is:
- **Secure** — no hardcoded passwords in code or config files
- **Persistent** — data survives pod restarts and crashes
- **Accessible** — available to internal microservices and external DBAs

### Real World DevOps Workflow

```
Developer writes app code using env vars
            ↓
DevOps writes Kubernetes manifests (like this task)
            ↓
Code pushed to GitHub
            ↓
CI/CD pipeline (ArgoCD / GitHub Actions) applies manifests
            ↓
Kubernetes spins up MySQL pod with:
  - Credentials injected from Secrets
  - Data mounted from PersistentVolume
  - Exposed via NodePort for external access
            ↓
App pods connect to MySQL via ClusterIP (internally)
DBA connects via NodePort 30007 (externally)
Monitoring tools (Prometheus/Grafana) track health
            ↓
Pod crashes → Deployment auto-restarts it (data safe on PV)
Password rotated → Update Secret only (no redeployment)
Disk fills up → Resize PVC (no downtime)
```

### Production Enhancements over this Lab

| Lab Setup | Production Setup |
|-----------|-----------------|
| `hostPath` volume | AWS EBS / GCP Persistent Disk / Azure Disk |
| 250Mi storage | 500GB+ SSD with automated snapshots |
| 1 replica | 3 replicas with MySQL replication |
| NodePort | LoadBalancer + Ingress Controller |
| Manual secrets | HashiCorp Vault / AWS Secrets Manager auto-rotation |
| No resource limits | CPU/Memory requests and limits defined |
| No health checks | `livenessProbe` and `readinessProbe` configured |

### Industries Using This Pattern

| Industry | Use Case |
|----------|----------|
| E-commerce | Product catalog, orders, user accounts |
| Banking / Fintech | Transaction records, account data |
| Healthcare | Patient records (with encryption) |
| SaaS Platforms | Per-tenant databases |
| Gaming | Player profiles, leaderboards |

---

## 🗺️ Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                  Kubernetes Cluster                  │
│                                                      │
│  ┌─────────────┐     ┌──────────────────────────┐   │
│  │   Secrets   │────▶│   mysql-deployment Pod   │   │
│  │─────────────│     │──────────────────────────│   │
│  │mysql-root-  │     │  Container: mysql:8.0    │   │
│  │  pass       │     │  Port: 3306              │   │
│  │mysql-user-  │     │  Env: MYSQL_ROOT_PASSWORD│   │
│  │  pass       │     │       MYSQL_DATABASE     │   │
│  │mysql-db-url │     │       MYSQL_USER         │   │
│  └─────────────┘     │       MYSQL_PASSWORD     │   │
│                      │  VolumeMount:            │   │
│  ┌─────────────┐     │  /var/lib/mysql          │   │
│  │     PV      │     └──────────┬───────────────┘   │
│  │  mysql-pv   │◀──────────────┘                    │
│  │  250Mi      │     ┌──────────────────────────┐   │
│  └──────┬──────┘     │     NodePort Service      │   │
│         │            │         mysql             │   │
│  ┌──────▼──────┐     │   3306 ──▶ 30007         │   │
│  │     PVC     │     └──────────────────────────┘   │
│  │mysql-pv-    │                  ▲                  │
│  │  claim      │                  │                  │
│  └─────────────┘                  │                  │
└──────────────────────────────────-│────────────────-─┘
                                    │
                            External Access
                         (DBA / App / Tools)
                           NodeIP:30007
```

---

## 🔧 Troubleshooting Guide

| Problem | Symptom | Command | Fix |
|---------|---------|---------|-----|
| PVC stuck in Pending | `STATUS: Pending` | `kubectl describe pvc mysql-pv-claim` | Check storageClassName matches PV |
| Pod in CrashLoopBackOff | `RESTARTS > 0` | `kubectl logs <pod>` | Check secret names/keys are correct |
| Pod in Init state | `STATUS: Init` | `kubectl describe pod <pod>` | Check image pull, resource limits |
| Service not accessible | Can't connect on 30007 | `kubectl get svc mysql` | Verify nodePort and selector labels match |
| Env vars not set | MySQL auth fails | `kubectl exec -it <pod> -- env` | Verify secret key names match secretKeyRef |
| PV not binding | PVC stays Pending | `kubectl get pv` | Check capacity, accessMode, storageClass |

---

## 💡 Key Takeaways

1. **Never hardcode credentials** — Always use Kubernetes Secrets with `secretKeyRef`
2. **Stateful apps need PVs** — Without persistent storage, all data is lost on pod restart
3. **PVC is the abstraction layer** — Pods claim storage via PVC, not directly from PV
4. **Deployment = self-healing** — Kubernetes restarts failed pods automatically
5. **Labels are the glue** — Service finds pods via `selector` matching pod `labels`
6. **StorageClass must match** — PV and PVC storageClassName must be identical for binding
7. **Secrets are not encrypted by default** — In production, enable encryption at rest or use Vault

---

## 📝 Quick Reference Cheat Sheet

```bash
# ── Secrets ──────────────────────────────────────────
kubectl create secret generic <name> --from-literal=<key>=<value>
kubectl get secrets
kubectl describe secret <name>
kubectl delete secret <name>

# ── PV / PVC ─────────────────────────────────────────
kubectl get pv
kubectl get pvc
kubectl describe pv <name>
kubectl describe pvc <name>

# ── Deployments ──────────────────────────────────────
kubectl apply -f <file>.yaml
kubectl get deployments
kubectl describe deployment <name>
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>      # rollback

# ── Pods ─────────────────────────────────────────────
kubectl get pods
kubectl get pods -l app=mysql              # filter by label
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/bash   # shell into pod
kubectl describe pod <pod-name>

# ── Services ─────────────────────────────────────────
kubectl get svc
kubectl describe svc <name>

# ── General ──────────────────────────────────────────
kubectl get all                            # all resources
kubectl delete -f <file>.yaml             # delete from file
kubectl apply -f <directory>/             # apply all YAMLs in dir
```

---

## 📚 Further Reading

- [Kubernetes Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Persistent Volumes Documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Deployments Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [MySQL Docker Image Reference](https://hub.docker.com/_/mysql)

---

*Study material compiled from hands-on KodeKloud Kubernetes lab — MySQL on Kubernetes deployment task.*
