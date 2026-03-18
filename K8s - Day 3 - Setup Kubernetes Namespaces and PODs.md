# Kubernetes Study Material — Namespace & Pod Deployment

> 📅 Date: March 18, 2026  
> 👤 Platform: Kubernetes (jump-host)  
> 🎯 Topic: Namespaces, Pods, Nodes, Clusters

---

## 📋 Original Task

> **Task:** The Nautilus DevOps team is planning to deploy some microservices on Kubernetes platform. The team has already set up a Kubernetes cluster and now they want to set up some namespaces, deployments etc. Based on the current requirements, the team has shared some details as below:
>
> - Create a namespace named **`dev`**
> - Deploy a POD within it
> - Name the pod **`dev-nginx-pod`**
> - Use the **`nginx`** image with the **`latest`** tag — `nginx:latest`
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🧠 Key Concepts

### 1. Cluster
The **entire Kubernetes environment** — everything combined.

- A collection of machines (servers) that work together
- Contains two types of machines:
  - **Master Node** — the brain/manager that controls everything
  - **Worker Nodes** — where your applications actually run
- Everything in Kubernetes lives inside a cluster

> 🏢 **Analogy:** Think of it as the **entire company building** with all its floors, rooms, and staff.

---

### 2. Node
A **single machine (physical or virtual server)** inside the cluster.

- The actual hardware/VM where your applications run
- A cluster can have multiple nodes
- Each node has its own CPU, RAM, and storage
- In this task: `Node: jump-host / 10.244.195.252`

> 🏢 **Analogy:** Think of it as a **single floor** in the company building.

---

### 3. Namespace
A **logical partition/grouping** inside a cluster.

- Used to **organize and isolate** resources
- Multiple teams can use the same cluster but in different namespaces
- Resources in one namespace are separated from another
- Default namespaces in Kubernetes:

| Namespace | Purpose |
|-----------|---------|
| `default` | General use if no namespace is specified |
| `kube-system` | Kubernetes internal components |
| `kube-public` | Publicly accessible resources |
| `kube-node-lease` | Node heartbeat tracking |

> 🏢 **Analogy:** Think of it as **different departments** in the same building (HR, IT, Finance) — they share the building but are separated.

**Common team-based usage:**

| Namespace | Purpose |
|-----------|---------|
| `dev` | Development team (created in this task) |
| `staging` | Testing before production |
| `production` | Live/production applications |

---

### 4. Pod
The **smallest deployable unit** in Kubernetes — wraps your container(s).

- Holds one or more **containers** (your actual application)
- Every pod gets its own **IP address** (in this task: `10.22.0.9`)
- Pods are **temporary** — if a pod dies, Kubernetes can create a new one
- In this task: `dev-nginx-pod` ran the `nginx` web server

> 🏢 **Analogy:** Think of it as a **single employee's desk/workspace** inside a department (namespace).

---

## 🗺️ Full Architecture Picture

```
CLUSTER  (the whole K8s environment)
│
├── Node: jump-host (10.244.195.252)   ← the physical/virtual machine
│
├── Namespace: dev                      ← created in this task
│   └── Pod: dev-nginx-pod (10.22.0.9) ← created in this task
│       └── Container: nginx:latest    ← actual app running here
│
├── Namespace: kube-system
│   └── Pods: (Kubernetes internal pods)
│
├── Namespace: kube-public
└── Namespace: default
```

---

## 🔑 Quick Reference Card

| Concept | What it is | Analogy |
|---------|-----------|---------|
| **Cluster** | Entire K8s environment | Whole company building |
| **Node** | A single server/machine | One floor of the building |
| **Namespace** | Logical grouping of resources | A department (HR, IT, Finance) |
| **Pod** | Smallest unit, runs your app | An employee's desk |

> **Remember the hierarchy:** `Cluster → Node → Namespace → Pod`

---

## 🛠️ Step-by-Step Task Execution

### Step 1: Verify kubectl and Cluster Access

```bash
kubectl cluster-info
```

**Output:**
```
Kubernetes control plane is running at https://<master-ip>:6443
CoreDNS is running at https://<master-ip>:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

---

### Step 2: Create the `dev` Namespace

```bash
kubectl create namespace dev
```

**Output:**
```
namespace/dev created
```

---

### Step 3: Verify the Namespace

```bash
kubectl get namespaces
```

**Output:**
```
NAME              STATUS   AGE
default           Active   40m
dev               Active   33s
kube-node-lease   Active   40m
kube-public       Active   40m
kube-system       Active   40m
```

---

### Step 4: Create the Pod

#### Option A — Imperative (Quick Command) ✅

```bash
kubectl run dev-nginx-pod --image=nginx:latest --namespace=dev
```

**Output:**
```
pod/dev-nginx-pod created
```

#### Option B — Declarative (YAML Manifest)

Create a file `dev-nginx-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx-pod
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

Apply it:

```bash
kubectl apply -f dev-nginx-pod.yaml
```

**Output:**
```
pod/dev-nginx-pod created
```

---

### Step 5: Verify the Pod is Running

```bash
kubectl get pods -n dev
```

**Output:**
```
NAME            READY   STATUS    RESTARTS   AGE
dev-nginx-pod   1/1     Running   0          32s
```

> ⚠️ If STATUS shows `ContainerCreating`, wait a few seconds — it transitions to `Running` once the image is pulled.

---

### Step 6: Describe the Pod (Verification)

```bash
kubectl describe pod dev-nginx-pod -n dev
```

**Key Output from this task:**
```
Name:         dev-nginx-pod
Namespace:    dev
Status:       Running
Node:         jump-host/10.244.195.252
Start Time:   Wed, 18 Mar 2026 16:36:37 +0000
IP:           10.22.0.9

Containers:
  dev-nginx-pod:
    Image:     nginx:latest
    Image ID:  docker.io/library/nginx@sha256:dec7a90bd0973b076832dc56933fe876bc014929e14b4ec49923951405370112
```

---

## 📌 All Commands — Summary

```bash
# 1. Verify cluster
kubectl cluster-info

# 2. Create namespace
kubectl create namespace dev

# 3. Verify namespace
kubectl get namespaces

# 4. Create pod
kubectl run dev-nginx-pod --image=nginx:latest --namespace=dev

# 5. Verify pod
kubectl get pods -n dev

# 6. Describe pod
kubectl describe pod dev-nginx-pod -n dev
```

---

## ✅ Task Completion Verification

| Requirement | Status |
|-------------|--------|
| Namespace `dev` created | ✅ |
| Pod named `dev-nginx-pod` | ✅ |
| Image `nginx:latest` used | ✅ |
| Pod deployed inside `dev` namespace | ✅ |
| Pod STATUS is `Running` | ✅ |
| Pod READY is `1/1` | ✅ |

---

## 📚 Useful kubectl Commands to Remember

| Command | Description |
|---------|-------------|
| `kubectl get namespaces` | List all namespaces |
| `kubectl get pods -n <namespace>` | List pods in a namespace |
| `kubectl describe pod <pod-name> -n <namespace>` | Detailed info about a pod |
| `kubectl delete pod <pod-name> -n <namespace>` | Delete a pod |
| `kubectl delete namespace <name>` | Delete a namespace |
| `kubectl get all -n <namespace>` | List all resources in a namespace |
| `kubectl logs <pod-name> -n <namespace>` | View pod logs |

---

*Study material generated from hands-on Kubernetes task completed on jump-host.*
