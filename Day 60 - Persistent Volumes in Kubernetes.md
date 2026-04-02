# Kubernetes Persistent Storage & Pod Deployment — Study Guide

---

## Original Task (Nautilus DevOps Scenario)

> The Nautilus DevOps team is working on a Kubernetes template to deploy a web application on the cluster. There are some requirements to create/use persistent volumes to store the application code, and the template needs to be designed accordingly.

### Requirements

1. Create a `PersistentVolume` named `pv-devops`. Configure the spec as:
   - Storage class: `manual`
   - Capacity: `4Gi`
   - Access mode: `ReadWriteOnce`
   - Volume type: `hostPath`
   - Path: `/mnt/finance`

2. Create a `PersistentVolumeClaim` named `pvc-devops`. Configure the spec as:
   - Storage class: `manual`
   - Request: `3Gi` of storage
   - Access mode: `ReadWriteOnce`

3. Create a `Pod` named `pod-devops`:
   - Mount the persistent volume with claim name `pvc-devops` at the document root of the web server
   - Container name: `container-devops`
   - Image: `nginx:latest`

4. Create a NodePort type service named `web-devops`:
   - NodePort: `30008`
   - Exposes the web server running within the pod

---

## Architecture / Flow

```
External User
     │
     │  port 30008
     ▼
[Service: web-devops]  (NodePort)
     │
     │  port 80
     ▼
[Pod: pod-devops]
[Container: container-devops (nginx:latest)]
     │
     │  mounted at /usr/share/nginx/html
     ▼
[PVC: pvc-devops]  (requests 3Gi)
     │
     │  binds to
     ▼
[PV: pv-devops]  (4Gi, hostPath: /mnt/finance)
```

---

## Solution — YAML Files with Explanations

### 1. PersistentVolume (`pv-devops`)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-devops            # Requirement 1: "named as pv-devops"
spec:
  storageClassName: manual   # Requirement 1: "storage class should be manual"
  capacity:
    storage: 4Gi             # Requirement 1: "set capacity to 4Gi"
  accessModes:
    - ReadWriteOnce          # Requirement 1: "set access mode to ReadWriteOnce"
  hostPath:
    path: /mnt/finance       # Requirement 1: "volume type hostPath, path /mnt/finance"
```

**What it does:** Creates actual storage on the host node at `/mnt/finance`. Think of it as a physical hard drive attached to the node. It sits in `Available` state and waits to be claimed.

**Apply command:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-devops
spec:
  storageClassName: manual
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/finance
EOF
```

**Verify:**
```bash
kubectl get pv pv-devops
```

**Expected output:**
```
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   AGE
pv-devops   4Gi        RWO            Retain           Available           manual         5s
```

---

### 2. PersistentVolumeClaim (`pvc-devops`)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-devops           # Requirement 2: "named as pvc-devops"
spec:
  storageClassName: manual   # Requirement 2: "storage class should be manual"
  accessModes:
    - ReadWriteOnce          # Requirement 2: "set access mode to ReadWriteOnce"
  resources:
    requests:
      storage: 3Gi           # Requirement 2: "request 3Gi of storage"
```

**What it does:** Sends a request for storage. Kubernetes scans all Available PVs, finds a match, and auto-binds them. Think of it as a booking ticket for the storage room.

**Apply command:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-devops
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
EOF
```

**Verify:**
```bash
kubectl get pvc pvc-devops
```

**Expected output:**
```
NAME         STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-devops   Bound    pv-devops   4Gi        RWO            manual         5s
```

> ✅ STATUS must be **Bound** — confirms PVC successfully matched and bound to the PV.

---

### 3. Pod (`pod-devops`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-devops                    # Requirement 3: "pod named as pod-devops"
  labels:
    app: web-devops                   # Label used by Service to find this pod
spec:
  containers:
    - name: container-devops          # Requirement 3: "container named container-devops"
      image: nginx:latest             # Requirement 3: "image nginx with latest tag"
      volumeMounts:
        - mountPath: /usr/share/nginx/html  # Requirement 3: "document root of web server"
          name: web-storage
  volumes:
    - name: web-storage
      persistentVolumeClaim:
        claimName: pvc-devops         # Requirement 3: "with claim name pvc-devops"
```

**What it does:** Runs the nginx container and mounts the persistent storage at nginx's document root `/usr/share/nginx/html`. The label `app: web-devops` is critical — the Service uses it to route traffic.

**Apply command:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-devops
  labels:
    app: web-devops
spec:
  containers:
    - name: container-devops
      image: nginx:latest
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: web-storage
  volumes:
    - name: web-storage
      persistentVolumeClaim:
        claimName: pvc-devops
EOF
```

**Verify:**
```bash
kubectl get pod pod-devops
```

**Expected output:**
```
NAME         READY   STATUS    RESTARTS   AGE
pod-devops   1/1     Running   0          15s
```

---

### 4. Service (`web-devops`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-devops           # Requirement 4: "service named web-devops"
spec:
  type: NodePort             # Requirement 4: "node port type service"
  selector:
    app: web-devops          # Matches the label on pod-devops
  ports:
    - port: 80               # Internal cluster port
      targetPort: 80         # Container port (nginx listens on 80)
      nodePort: 30008        # Requirement 4: "using node port 30008"
```

**What it does:** Opens port `30008` on every cluster node. Any traffic on that port is forwarded to port `80` on the nginx container. The `selector` field is how the Service finds the right pod using labels.

**Apply command:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: web-devops
spec:
  type: NodePort
  selector:
    app: web-devops
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
EOF
```

**Verify:**
```bash
kubectl get svc web-devops
```

**Expected output:**
```
NAME         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web-devops   NodePort   10.96.xxx.xxx   <none>        80:30008/TCP   5s
```

---

## Final Verification (All Resources at Once)

```bash
kubectl get pv pv-devops && kubectl get pvc pvc-devops && kubectl get pod pod-devops && kubectl get svc web-devops
```

**Test connectivity:**
```bash
curl http://<node-ip>:30008
```

---

## How PV & PVC Binding Works

When you apply the PVC, Kubernetes runs an automatic matchmaking process checking 3 criteria:

### Matching Criteria

| Criteria | PV Value | PVC Value | Match? |
|---|---|---|---|
| `storageClassName` | `manual` | `manual` | ✅ |
| `accessModes` | `ReadWriteOnce` | `ReadWriteOnce` | ✅ |
| Capacity | `4Gi` available | `3Gi` requested | ✅ (4 ≥ 3) |

### Binding Flow

```
kubectl apply PV (pv-devops)
        │
        ▼
Kubernetes stores it → STATUS: Available
        │
kubectl apply PVC (pvc-devops)
        │
        ▼
Kubernetes controller scans all Available PVs
        │
        ▼
Finds pv-devops → checks 3 criteria → all pass ✅
        │
        ▼
Kubernetes LOCKS pv-devops to pvc-devops
        │
        ▼
PV  STATUS: Available → Bound  (CLAIM: default/pvc-devops)
PVC STATUS: Pending   → Bound  (VOLUME: pv-devops)
```

### Important Binding Rules

- **Exclusive:** Once bound, no other PVC can claim the same PV — even leftover capacity is locked
- **Order doesn't matter:** PV waits as `Available`, PVC waits as `Pending` — binding happens the moment both exist and match
- **Automatic:** Binding is done by Kubernetes' PersistentVolume Controller, never manually
- **`default/pvc-devops`:** The `default` prefix is the namespace (default since none was specified)

### Real World Analogy — Hotel Room Booking

| Kubernetes | Hotel Analogy |
|---|---|
| PV (4Gi) | A hotel room available for rent |
| PVC (needs 3Gi) | A guest booking request |
| `storageClassName` | Hotel category (budget/luxury) |
| `accessModes` | Room type (single/double) |
| Capacity check | Room big enough for the guest |
| Bound | Room reserved & locked for that guest |
| Extra 1Gi unused | Extra space the guest doesn't use |

---

## Kubernetes `kind` Values — Complete Reference

The `kind` field tells Kubernetes what type of object you're creating. The `spec` section changes completely based on kind.

### Basic YAML Structure

```yaml
apiVersion: v1        # Which API group
kind: Pod             # What type of object
metadata:             # Name, labels, namespace
spec:                 # Configuration — changes per kind!
```

### Workloads — Run Your Application

| Kind | Behavior |
|---|---|
| `Pod` | Smallest unit. Runs containers. No self-healing — if it dies, it's gone |
| `Deployment` | Manages pods. Auto-restarts if pod dies. Supports rolling updates & rollback |
| `StatefulSet` | Like Deployment but for stateful apps (databases). Pods get stable names & storage |
| `DaemonSet` | Runs ONE pod on EVERY node. Used for monitoring agents, log collectors |
| `Job` | Runs a pod ONCE to completion. Used for batch tasks |
| `CronJob` | Runs a Job on a schedule (like linux cron). Used for backups, cleanups |
| `ReplicaSet` | Ensures N copies of a pod run. Usually managed by Deployment, rarely used directly |

### Networking — Expose Your Application

| Kind | Behavior |
|---|---|
| `Service` | Exposes pods internally or externally. Types: ClusterIP, NodePort, LoadBalancer |
| `Ingress` | HTTP/HTTPS routing rules. Routes traffic based on URL path |
| `NetworkPolicy` | Firewall rules between pods. Controls who can talk to who |

### Storage — Persist Your Data

| Kind | Behavior |
|---|---|
| `PersistentVolume` | Actual storage unit on the host node |
| `PersistentVolumeClaim` | Request for storage — binds to a matching PV |
| `StorageClass` | Defines HOW storage is provisioned (auto or manual) |
| `ConfigMap` | Stores non-sensitive config data (env vars, config files) |
| `Secret` | Stores sensitive data (passwords, tokens) — base64 encoded |

### Configuration & Access Control

| Kind | Behavior |
|---|---|
| `ServiceAccount` | Identity for a pod to interact with the Kubernetes API |
| `Role` | Defines permissions within a namespace |
| `ClusterRole` | Defines permissions across the whole cluster |
| `RoleBinding` | Assigns a Role to a user/serviceaccount |
| `ClusterRoleBinding` | Assigns a ClusterRole cluster-wide |

### Cluster Structure

| Kind | Behavior |
|---|---|
| `Namespace` | Virtual cluster inside a cluster. Isolates resources |
| `Node` | Represents a worker machine in the cluster |
| `ResourceQuota` | Limits total resources a namespace can use |
| `LimitRange` | Sets default/max resource limits for pods in a namespace |
| `HorizontalPodAutoscaler` | Auto-scales pod count based on CPU/memory usage |

### `apiVersion` by `kind`

```yaml
apiVersion: v1                              # Pod, Service, PV, PVC, ConfigMap, Secret, Namespace
apiVersion: apps/v1                         # Deployment, StatefulSet, DaemonSet, ReplicaSet
apiVersion: batch/v1                        # Job, CronJob
apiVersion: networking.k8s.io/v1            # Ingress, NetworkPolicy
apiVersion: autoscaling/v2                  # HorizontalPodAutoscaler
apiVersion: rbac.authorization.k8s.io/v1   # Role, ClusterRole, RoleBinding
```

> ⚠️ Wrong `apiVersion` for a `kind` = apply will fail. They must always match!

---

## Quick Decision Guide

```
Need to run a container?
├── One-time task?          → Job / CronJob
├── Needs stable storage?   → StatefulSet
├── Run on every node?      → DaemonSet
└── Normal app?             → Deployment (NOT raw Pod — no self-healing!)

Need to expose it?
├── Inside cluster only?    → Service (ClusterIP)
├── External via port?      → Service (NodePort)  ← used in this task
├── External via domain?    → Ingress
└── Cloud load balancer?    → Service (LoadBalancer)

Need to store data?
├── Actual storage?         → PersistentVolume + PersistentVolumeClaim
├── Config values?          → ConfigMap
└── Passwords/tokens?       → Secret
```

---

## Summary Table — Task Requirements vs YAML Fields

| Requirement | Resource Created | Key Values |
|---|---|---|
| PV storage | `pv-devops` | `manual`, `4Gi`, `ReadWriteOnce`, `/mnt/finance` |
| PVC request | `pvc-devops` | `manual`, `3Gi`, `ReadWriteOnce` |
| Pod + container | `pod-devops` / `container-devops` | `nginx:latest`, `/usr/share/nginx/html`, `pvc-devops` |
| Expose app | `web-devops` service | `NodePort`, `30008` |

---

## Key Takeaways

- Every value in the YAML came **directly from the requirements** — mapping plain English to YAML fields is the core skill
- `kind` is the **DNA** of a Kubernetes object — it defines everything about how Kubernetes handles that resource
- PV = the actual storage | PVC = the request/ticket for storage — they bind automatically when all 3 criteria match
- Labels on pods + selectors on services = how traffic routing works in Kubernetes
- `hostPath` stores data on the node's filesystem — suitable for single-node clusters and testing
- nginx's document root is always `/usr/share/nginx/html` — this is where web files are served from
