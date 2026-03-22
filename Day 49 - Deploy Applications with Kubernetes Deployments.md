# Kubernetes Nginx Deployment — Study Material

---

## 📋 Original Task

> **Task:** Create a deployment named `nginx` to deploy the application `nginx` using the image `nginx:latest` (ensure to specify the tag).
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 📖 Concepts Overview

### What is a Kubernetes Deployment?
A **Deployment** is a Kubernetes resource that manages a set of identical Pods. It ensures:
- The desired number of Pod replicas are always running.
- Rolling updates and rollbacks are handled gracefully.
- Self-healing: if a Pod crashes, it gets automatically replaced.

### What is kubectl?
`kubectl` is the command-line tool used to interact with a Kubernetes cluster. It communicates with the **Kubernetes API Server** to manage resources like Deployments, Pods, Services, etc.

### What is an Image Tag?
An image tag identifies a specific version of a container image.
- `nginx` → resolves to `nginx:latest` implicitly, but **not recommended** in production as it's ambiguous.
- `nginx:latest` → explicitly pulls the latest stable image. Always specifying a tag is best practice.

---

## 🛠️ Step-by-Step Commands

### Step 1: Verify Cluster Connectivity
```bash
kubectl cluster-info
```
**Expected Output:**
```
Kubernetes control plane is running at https://<cluster-ip>:6443
CoreDNS is running at https://<cluster-ip>:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

---

### Step 2: Check Existing Deployments (Baseline)
```bash
kubectl get deployments
```
**Expected Output (before creation):**
```
No resources found in default namespace.
```

---

### Step 3: Create the Nginx Deployment ⭐
```bash
kubectl create deployment nginx --image=nginx:latest
```
**Expected Output:**
```
deployment.apps/nginx created
```

---

### Step 4: Verify the Deployment
```bash
kubectl get deployments
```
**Expected Output:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           30s
```

---

### Step 5: Check Pods Spawned by the Deployment
```bash
kubectl get pods
```
**Expected Output:**
```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-<hash>-<hash>      1/1     Running   0          45s
```

---

### Step 6: Inspect Deployment Details
```bash
kubectl describe deployment nginx
```
**Key sections in output:**
```
Name:                   nginx
Namespace:              default
Replicas:               1 desired | 1 updated | 1 total | 1 available
Pod Template:
  Containers:
   nginx:
    Image:      nginx:latest
Conditions:
  Available     True    MinimumReplicasAvailable
```

---

### Step 7: Confirm Image Tag (Final Verification) ✅
```bash
kubectl get deployment nginx -o=jsonpath='{.spec.template.spec.containers[0].image}'
```
**Expected Output:**
```
nginx:latest
```

---

## 📊 Commands Summary Table

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `kubectl cluster-info` | Confirm cluster connectivity |
| 2 | `kubectl get deployments` | Baseline check |
| 3 | `kubectl create deployment nginx --image=nginx:latest` | **Create the deployment** |
| 4 | `kubectl get deployments` | Confirm deployment exists |
| 5 | `kubectl get pods` | Check pods are running |
| 6 | `kubectl describe deployment nginx` | Full detail verification |
| 7 | `kubectl get deployment nginx -o=jsonpath='{.spec.template.spec.containers[0].image}'` | Confirm image tag |

---

## 🔍 Key kubectl Flags Explained

| Flag / Option | Meaning |
|---------------|---------|
| `create deployment` | Imperative command to create a Deployment resource |
| `--image=nginx:latest` | Specifies the container image with an explicit tag |
| `get deployments` | Lists all Deployment resources in the current namespace |
| `describe deployment <name>` | Shows detailed info about a specific Deployment |
| `-o=jsonpath='...'` | Outputs a specific field using JSONPath query |
| `get pods` | Lists all Pods in the current namespace |

---

## 📝 JSONPath Explained

The final verification command uses **JSONPath** — a query language for JSON:

```bash
kubectl get deployment nginx -o=jsonpath='{.spec.template.spec.containers[0].image}'
```

Breaking it down:
```
.spec                          → Deployment specification
  .template                    → Pod template defined in the deployment
    .spec                      → Pod's specification
      .containers[0]           → First container in the Pod (index 0)
        .image                 → The container image value
```

This is a powerful way to extract specific fields from Kubernetes resources without parsing full YAML/JSON output manually.

---

## ⚙️ Deployment Manifest (YAML Equivalent)

The imperative command `kubectl create deployment nginx --image=nginx:latest` is equivalent to applying this YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

To apply via YAML:
```bash
kubectl apply -f nginx-deployment.yaml
```

---

## 💡 Key Takeaways

1. **Always specify image tags** — Using `nginx:latest` is better than `nginx` alone for clarity.
2. **Imperative vs Declarative** — `kubectl create` is imperative (quick); `kubectl apply -f` is declarative (preferred in production).
3. **JSONPath queries** are useful for scripting and verification in CI/CD pipelines.
4. **Deployments manage Pods** — You don't create Pods directly; the Deployment controller does it for you.
5. **Self-healing** — If you delete a Pod created by a Deployment, Kubernetes will recreate it automatically.

---

## 🔗 Further Reading

- [Kubernetes Deployments — Official Docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [JSONPath in kubectl](https://kubernetes.io/docs/reference/kubectl/jsonpath/)
- [Container Images Best Practices](https://kubernetes.io/docs/concepts/containers/images/)

---

*Study material generated for the Nautilus DevOps Team — Kubernetes Nginx Deployment Task*
