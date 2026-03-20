# 📚 Kubernetes Study Material — Resource Limits & Pod Creation

> **Topic:** Kubernetes Resource Management, Pods, Nodes, and Containers  
> **Level:** Beginner  
> **Based on:** Hands-on task completed on March 20, 2026

---

## 📋 Original Task

> The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization. Here are the details:
>
> Create a pod named `httpd-pod` with a container named `httpd-container`. Use the `httpd` image with the `latest` tag (specified as `httpd:latest`). Set the following resource limits:
>
> - **Requests:** Memory: `15Mi`, CPU: `100m`
> - **Limits:** Memory: `20Mi`, CPU: `100m`
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🗂️ Table of Contents

1. [Key Concepts](#key-concepts)
2. [The City Analogy](#the-city-analogy)
3. [Requests vs Limits Explained](#requests-vs-limits-explained)
4. [CPU and Memory Units](#cpu-and-memory-units)
5. [QoS Classes](#qos-classes)
6. [Step-by-Step Solution](#step-by-step-solution)
7. [The YAML Manifest](#the-yaml-manifest)
8. [Useful kubectl Commands](#useful-kubectl-commands)
9. [Key Takeaways](#key-takeaways)

---

## 1. Key Concepts

| Term | Definition |
|---|---|
| **Cluster** | A group of machines (nodes) managed by Kubernetes |
| **Node** | A physical or virtual machine that runs pods |
| **Pod** | The smallest deployable unit in Kubernetes; wraps one or more containers |
| **Container** | A lightweight, isolated runtime environment running your app |
| **kubelet** | An agent on each node that manages pods |
| **kubectl** | The command-line tool to interact with the Kubernetes cluster |
| **Scheduler** | Kubernetes component that decides which node a pod runs on |
| **Control Plane** | The brain of Kubernetes — manages the entire cluster |

---

## 2. The City Analogy

The easiest way to understand Kubernetes architecture:

```
🌆 KUBERNETES CLUSTER  =  A City
🏢 NODE                =  A Building in the city
🏠 POD                 =  An Apartment in the building
👤 CONTAINER           =  A Person living in the apartment
```

### Full Picture

```
🌆 CITY (Kubernetes Cluster)
│
│   🏢 BUILDING 1 (Node: jump-host / 10.244.81.29)
│   │
│   │   🏠 Apartment 101 (Pod: httpd-pod / 10.22.0.9)
│   │   │   👤 Person (Container: httpd-container)
│   │   │       Uses: 100m CPU, 15-20Mi Memory
│   │   │
│   │   🏠 Apartment 102 (Pod: some-other-pod)
│   │       👤 Person (Container: another-container)
│   │
│   🏢 BUILDING 2 (Node: worker-node-2)
│       🏠 Apartment 201 (Pod: another-pod)
│           👤 Person (Container: another-container)
│
🏛️ City Hall (Control Plane)
    └── Decides which building gets which apartment
    └── Monitors all buildings and apartments
    └── Restarts evicted people automatically
```

### Analogy Extended to Concepts

| Kubernetes Concept | City Analogy |
|---|---|
| Resource Request | Minimum apartment size you need |
| Resource Limit | Maximum space you're allowed to use |
| OOMKilled | Evicted for flooding the apartment |
| CPU Throttle | Electricity capped, lights dim |
| QoS Guaranteed | Long-term lease, last to be evicted |
| QoS Burstable | Month-to-month lease, evicted 2nd |
| QoS BestEffort | No lease at all, evicted first |
| Pod in Pending state | No apartment available in any building |
| Scheduler | Building manager assigning apartments |

---

## 3. Requests vs Limits Explained

### 📦 Requests — *"What the container is guaranteed to get"*

- Kubernetes **reserves** this amount when scheduling the pod on a node
- A pod is only placed on a node that has **at least this much free resource**
- Used by the **Scheduler** to decide which node to use

```
In this task:
  Memory Request: 15Mi  → Node must have at least 15Mi free
  CPU Request:   100m   → Node must have at least 100m free
```

### 🚧 Limits — *"The maximum the container is allowed to use"*

- This is the **hard ceiling** — container cannot exceed this
- **CPU over limit** → Gets **throttled** (slowed down, pod stays alive)
- **Memory over limit** → Gets **OOMKilled** (pod is killed and restarted)

```
In this task:
  Memory Limit: 20Mi  → Container can burst up to 20Mi, then gets killed
  CPU Limit:   100m   → Container gets throttled if it tries to use more
```

### Visual Comparison

```
Memory Usage
────────────────────────────────────────────
0Mi       15Mi          20Mi
|---------|-------------|
  Request    Can Burst     ← HARD LIMIT (OOMKilled if exceeded)
  (Guaranteed)
```

---

## 4. CPU and Memory Units

### ⚙️ CPU Units

```
1000m (millicores) = 1 full CPU core

100m = 0.1 of a CPU core (10% of one core)
500m = 0.5 of a CPU core (half a core)
```

> Think of a CPU core as having 1000 slices. You're requesting 100 of those slices.

### 💾 Memory Units

```
Mi = Mebibytes (binary-based)
Gi = Gibibytes

1Mi = 1,048,576 bytes ≈ 1.05 MB
1Gi = 1,073,741,824 bytes ≈ 1.07 GB

15Mi ≈ 15.7 MB
20Mi ≈ 20.9 MB
```

> ⚠️ `Mi` (Mebibytes) ≠ `MB` (Megabytes). Kubernetes uses binary units (`Mi`, `Gi`).

---

## 5. QoS Classes

Kubernetes automatically assigns a **Quality of Service (QoS)** class to every pod.

### Three QoS Classes

| QoS Class | Condition | Priority |
|---|---|---|
| **Guaranteed** | Requests = Limits for ALL resources | 🟢 Safest — killed last |
| **Burstable** | Requests < Limits for at least one resource | 🟡 Middle — killed second |
| **BestEffort** | No requests or limits set at all | 🔴 Riskiest — killed first |

### Your Task — QoS = `Burstable`

```
Why Burstable?

  CPU:    Request (100m) = Limit (100m)  ✅ Equal
  Memory: Request (15Mi) ≠ Limit (20Mi)  ❌ Not Equal

Since memory request ≠ memory limit → QoS = Burstable

To get Guaranteed QoS, you would need:
  Memory Request = Memory Limit = 20Mi (or any same value)
  CPU Request    = CPU Limit    = 100m
```

### Why QoS Matters — Eviction Order

When a node runs **out of memory**, Kubernetes must kill some pods:

```
1st to die → BestEffort  (no guarantees at all)
2nd to die → Burstable   (some guarantees)
Last to die → Guaranteed  (fully protected)
```

---

## 6. Step-by-Step Solution

### Step 1: Verify kubectl connection
```bash
kubectl get nodes
```
Expected output:
```
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   10d   v1.28.0
node01         Ready    <none>          10d   v1.28.0
```

---

### Step 2: Create the YAML manifest
```bash
cat <<EOF > httpd-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      resources:
        requests:
          memory: "15Mi"
          cpu: "100m"
        limits:
          memory: "20Mi"
          cpu: "100m"
EOF
```

---

### Step 3: Apply the manifest
```bash
kubectl apply -f httpd-pod.yaml
```
Expected output:
```
pod/httpd-pod created
```

---

### Step 4: Verify pod is running
```bash
kubectl get pod httpd-pod
```
Expected output:
```
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          30s
```

> ⏳ May briefly show `ContainerCreating` while the image pulls. Wait a few seconds.

---

### Step 5: Confirm resource limits
```bash
kubectl describe pod httpd-pod
```
Look for this section in the output:
```
Containers:
  httpd-container:
    Image:   httpd:latest
    Limits:
      cpu:     100m
      memory:  20Mi
    Requests:
      cpu:     100m
      memory:  15Mi
```

---

## 7. The YAML Manifest

```yaml
apiVersion: v1          # Kubernetes API version
kind: Pod               # Type of resource
metadata:
  name: httpd-pod       # Name of the pod
spec:
  containers:
    - name: httpd-container   # Name of the container
      image: httpd:latest     # Docker image to use
      resources:
        requests:             # Minimum guaranteed resources
          memory: "15Mi"
          cpu: "100m"
        limits:               # Maximum allowed resources
          memory: "20Mi"
          cpu: "100m"
```

### YAML Fields Explained

| Field | Purpose |
|---|---|
| `apiVersion` | Which Kubernetes API to use (`v1` for pods) |
| `kind` | The type of Kubernetes resource |
| `metadata.name` | The name to identify this pod |
| `spec.containers` | List of containers inside the pod |
| `image` | Docker image name and tag |
| `resources.requests` | Resources Kubernetes reserves for scheduling |
| `resources.limits` | Hard cap on resource usage at runtime |

---

## 8. Useful kubectl Commands

```bash
# View all pods
kubectl get pods

# View pods with more detail (IP, node)
kubectl get pods -o wide

# Describe a specific pod (full details)
kubectl describe pod httpd-pod

# View pod logs
kubectl logs httpd-pod

# Delete a pod
kubectl delete pod httpd-pod

# View all nodes
kubectl get nodes

# Describe a node
kubectl describe node <node-name>

# View resource usage of nodes (requires metrics-server)
kubectl top nodes

# View resource usage of pods
kubectl top pods
```

---

## 9. Key Takeaways

```
✅ Always set both Requests AND Limits — never leave them empty in production
✅ Requests  → affects pod SCHEDULING (where it lands on which node)
✅ Limits    → affects pod BEHAVIOR at RUNTIME (throttle or kill)
✅ CPU over limit   → Throttled (slowed) — pod stays alive
✅ Memory over limit → OOMKilled — pod is RESTARTED
✅ QoS class is auto-assigned — it affects eviction priority under memory pressure
✅ Burstable QoS = Request < Limit for at least one resource
✅ Guaranteed QoS = Request = Limit for ALL resources (safest)
✅ The Scheduler uses Requests (not Limits) to place pods on nodes
✅ A pod stays Pending if no node has enough resources to meet Requests
```

---

## ✅ Task Completion Summary

| Field | Value |
|---|---|
| Pod Name | `httpd-pod` |
| Container Name | `httpd-container` |
| Image | `httpd:latest` |
| Memory Request | `15Mi` |
| CPU Request | `100m` |
| Memory Limit | `20Mi` |
| CPU Limit | `100m` |
| Namespace | `default` |
| Node | `jump-host` |
| Pod IP | `10.22.0.9` |
| QoS Class | `Burstable` |
| Final Status | ✅ Running |

---

*Study material generated from hands-on Kubernetes task — March 20, 2026*
