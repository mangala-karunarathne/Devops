# Kubernetes Pod Resource Limits — Study Material

---

## 📋 Original Task

> The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization. Here are the details:
>
> Create a pod named **`httpd-pod`** with a container named **`httpd-container`**. Use the **`httpd`** image with the **`latest`** tag (specify as `httpd:latest`). Set the following resource limits:
>
> - **Requests:** Memory: `15Mi`, CPU: `100m`
> - **Limits:** Memory: `20Mi`, CPU: `100m`
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🧠 Concepts to Understand

### 1. What are Resource Requests and Limits?

| Term | Description |
|---|---|
| **Request** | The **minimum** guaranteed resources a container needs. Used by the scheduler to decide which node to place the pod on. |
| **Limit** | The **maximum** resources a container is allowed to use. If exceeded, the container may be throttled (CPU) or OOMKilled (Memory). |

### 2. Resource Units

| Resource | Unit | Meaning |
|---|---|---|
| CPU | `m` (millicores) | `100m` = 0.1 of a CPU core |
| Memory | `Mi` (Mebibytes) | `15Mi` = ~15.7 MB |

> 💡 `Mi` (Mebibytes) ≠ `MB` (Megabytes). Kubernetes uses binary units: `1Mi = 1024 * 1024 bytes`.

### 3. QoS (Quality of Service) Classes

Kubernetes assigns a QoS class based on how requests and limits are set:

| QoS Class | Condition |
|---|---|
| **Guaranteed** | Requests == Limits for ALL resources |
| **Burstable** | Requests < Limits for at least one resource |
| **BestEffort** | No requests or limits set at all |

> In this task, memory request (`15Mi`) < memory limit (`20Mi`), so the QoS class is **Burstable**.

---

## 📄 Pod YAML Manifest

```yaml
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
```

### YAML Field Breakdown

| Field | Value | Purpose |
|---|---|---|
| `apiVersion` | `v1` | Core Kubernetes API version for Pods |
| `kind` | `Pod` | Resource type |
| `metadata.name` | `httpd-pod` | Name of the pod |
| `containers[].name` | `httpd-container` | Name of the container inside the pod |
| `containers[].image` | `httpd:latest` | Docker image to use |
| `resources.requests.memory` | `15Mi` | Minimum memory guaranteed |
| `resources.requests.cpu` | `100m` | Minimum CPU guaranteed |
| `resources.limits.memory` | `20Mi` | Maximum memory allowed |
| `resources.limits.cpu` | `100m` | Maximum CPU allowed |

---

## 🛠️ Step-by-Step Execution

### Step 1 — Verify Cluster Access
```bash
kubectl cluster-info
```
**Expected Output:**
```
Kubernetes control plane is running at https://<master-ip>:6443
```

---

### Step 2 — Create the YAML Manifest
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

### Step 3 — Apply the Manifest
```bash
kubectl apply -f httpd-pod.yaml
```
**Expected Output:**
```
pod/httpd-pod created
```

---

### Step 4 — Verify Pod Status
```bash
kubectl get pod httpd-pod
```
**Expected Output:**
```
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          30s
```

---

### Step 5 — Describe the Pod (Full Verification)
```bash
kubectl describe pod httpd-pod
```
**Key section to verify:**
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

## ✅ Final Verification Summary

| Field | Configured Value | Status |
|---|---|---|
| Pod Name | `httpd-pod` | ✅ |
| Namespace | `default` | ✅ |
| Container Name | `httpd-container` | ✅ |
| Image | `httpd:latest` | ✅ |
| CPU Limit | `100m` | ✅ |
| Memory Limit | `20Mi` | ✅ |
| CPU Request | `100m` | ✅ |
| Memory Request | `15Mi` | ✅ |
| Pod Status | `Running` | ✅ |
| QoS Class | `Burstable` | ✅ |

---

## ⚠️ Common Mistakes to Avoid

1. **Wrong memory unit** — Using `MB` instead of `Mi` can cause unexpected behavior. Always use `Mi` or `Gi` in Kubernetes.
2. **Limits lower than requests** — Kubernetes will reject the pod. Limits must always be ≥ requests.
3. **Forgetting the `resources` block** — Without it, the container has no limits and can consume all node resources.
4. **Using wrong `apiVersion`** — Pods use `v1`, not `apps/v1` (that's for Deployments).
5. **Image tag omission** — Always explicitly specify `:latest` or a version tag for clarity and reproducibility.

---

## 🔁 Useful kubectl Commands for Resource Management

```bash
# View resource usage of pods (requires metrics-server)
kubectl top pod

# View resource usage of nodes
kubectl top node

# Delete the pod
kubectl delete pod httpd-pod

# Edit the pod (limited fields editable after creation)
kubectl edit pod httpd-pod

# Get pod in YAML format
kubectl get pod httpd-pod -o yaml

# Watch pod status in real time
kubectl get pod httpd-pod -w
```

---

## 📚 Further Reading

- [Kubernetes Docs — Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Kubernetes Docs — Pod QoS Classes](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/)
- [Kubernetes Docs — Configure Default Memory Requests and Limits](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

---

*Study material generated on: 22 March 2026*
