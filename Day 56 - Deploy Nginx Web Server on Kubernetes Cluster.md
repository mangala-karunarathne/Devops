# 📚 Kubernetes Nginx Deployment — Study Material

---

## 📋 Original Task

> The Nautilus team developers are developing a static website and want to deploy it on a Kubernetes cluster. They want it to be **highly available and scalable**. The DevOps team has decided to create a deployment with multiple replicas.
>
> **Requirements:**
> 1. Create a deployment using `nginx:latest` image. Name it `nginx-deployment`. The container should be named `nginx-container`. Replica count must be `3`.
> 2. Create a `NodePort` type service named `nginx-service`. The nodePort should be `30011`.
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🐳 Docker vs ☸️ Kubernetes — Key Differences

| | Docker | Kubernetes |
|---|---|---|
| Analogy | You cook **one meal** at home | A **restaurant** manages many meals, chefs, tables |
| Config File | `Dockerfile` | `.yaml` files |
| Scale | 1 container | Hundreds of containers |
| Self-healing | ❌ No | ✅ Yes (restarts if crashed) |

### Why YAML instead of Dockerfile?

- A `Dockerfile` **builds/creates** an image — like a recipe
- A `deployment.yaml` **runs & manages** containers — how many, which image
- A `service.yaml` **exposes** containers — how to reach them

> In this task, `nginx:latest` was already available on Docker Hub — so we skipped Dockerfile and went straight to Kubernetes YAML.

---

## 📄 YAML File 1 — `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 🔍 Line by Line Explanation

#### Block 1 — What kind of thing are we creating?
```yaml
apiVersion: apps/v1
kind: Deployment
```
| Line | Meaning |
|---|---|
| `apiVersion: apps/v1` | Which version of Kubernetes API to use. `apps/v1` is standard for Deployments |
| `kind: Deployment` | We are creating a **Deployment** (not a Pod, Service, etc.) |

> 💡 Tells Kubernetes "Hey, I want to create a Deployment"

---

#### Block 2 — What is its name?
```yaml
metadata:
  name: nginx-deployment
```
| Line | Meaning |
|---|---|
| `metadata` | Information **about** the object |
| `name: nginx-deployment` | The name of this deployment — seen in `kubectl get deployment` |

> 💡 Like a **name tag** for your deployment

---

#### Block 3 — How many copies do we want?
```yaml
spec:
  replicas: 3
```
| Line | Meaning |
|---|---|
| `spec` | The actual specification — what we want Kubernetes to do |
| `replicas: 3` | Run **3 copies** of this container at all times |

> 💡 If one crashes, Kubernetes automatically starts a new one to maintain 3

---

#### Block 4 — Which pods does this deployment manage?
```yaml
  selector:
    matchLabels:
      app: nginx
```
| Line | Meaning |
|---|---|
| `selector` | Tells the Deployment **which pods to manage** |
| `matchLabels` | Match pods that have this label |
| `app: nginx` | Only manage pods labeled `app: nginx` |

> 💡 Like a **manager's ID badge** — only manages employees (pods) wearing the `app=nginx` badge

---

#### Block 5 — What does each pod look like?
```yaml
  template:
    metadata:
      labels:
        app: nginx
```
| Line | Meaning |
|---|---|
| `template` | The **blueprint** for each pod that gets created |
| `labels` | Stick a label on each pod |
| `app: nginx` | The label — **must match** the selector above |

> 💡 The **employee badge** — must match what the manager (selector) is looking for

---

#### Block 6 — What runs inside each pod?
```yaml
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```
| Line | Meaning |
|---|---|
| `containers` | List of containers inside this pod |
| `- name: nginx-container` | Name of the container (`-` means list item) |
| `image: nginx:latest` | Pull this image from Docker Hub |
| `containerPort: 80` | Container listens on port **80** (default nginx port) |

---

## 📄 YAML File 2 — `nginx-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30011
```

### 🔍 Line by Line Explanation

#### Block 1 — What kind of thing are we creating?
```yaml
apiVersion: v1
kind: Service
```
| Line | Meaning |
|---|---|
| `apiVersion: v1` | Services use the core `v1` API |
| `kind: Service` | We are creating a **Service** |

---

#### Block 2 — What is its name?
```yaml
metadata:
  name: nginx-service
```
| Line | Meaning |
|---|---|
| `name: nginx-service` | Name of the service — seen in `kubectl get service` |

---

#### Block 3 — What type of service is it?
```yaml
spec:
  type: NodePort
```
| Line | Meaning |
|---|---|
| `type: NodePort` | Expose this service on a port on the Node so outside world can access |

**3 Service Types:**
| Type | Access |
|---|---|
| `ClusterIP` | Only inside the cluster |
| `NodePort` | Accessible from outside via a port ← **used in this task** |
| `LoadBalancer` | For cloud providers (AWS, GCP etc.) |

---

#### Block 4 — Which pods should receive traffic?
```yaml
  selector:
    app: nginx
```
| Line | Meaning |
|---|---|
| `selector` | Route traffic **only to pods** with this label |
| `app: nginx` | Target pods labeled `app=nginx` |

> 💡 This is how the Service **finds your 3 nginx pods** — by matching the label

---

#### Block 5 — How does traffic flow?
```yaml
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30011
```
| Line | Meaning |
|---|---|
| `port: 80` | Port the **Service itself** listens on (inside cluster) |
| `targetPort: 80` | Port on the **Pod/container** to forward traffic to |
| `nodePort: 30011` | Port on the **Node** that outside world uses to access |

**Traffic Journey:**
```
Your curl request
      ⬇
Node port 30011        ← nodePort (you type this in browser/curl)
      ⬇
Service port 80        ← port (internal cluster)
      ⬇
Container port 80      ← targetPort (nginx inside pod)
      ⬇
nginx serves the page ✅
```

---

## 🔑 The Critical Link Between Both Files

> The **label `app: nginx`** is the key connection between both YAML files.
> - The **Deployment** puts this label on every pod it creates
> - The **Service** uses this label to find and route traffic to those pods
>
> If these labels don't match → Service shows `Endpoints: <none>` → No traffic reaches pods ❌

---

## ⚙️ Understanding `kind` — Why It Matters A Lot

`kind` is NOT just a name — it tells Kubernetes **what to do** with the YAML.

Think of it like a Government Form:
> - Form A = Birth Certificate → Registry dept
> - Form B = Passport → Immigration dept
> - Form C = Tax → Revenue dept

```yaml
kind: Deployment  → Deployment Controller
kind: Service     → Service Controller
kind: Pod         → Pod Controller
```

| `kind` | What Kubernetes Does |
|---|---|
| `Pod` | Runs **one container**, no restart if it dies |
| `Deployment` | Runs pods, **maintains replicas**, restarts if crashed |
| `Service` | Sets up **networking**, routes traffic to pods |
| `ConfigMap` | Stores **config data** for containers |
| `Secret` | Stores **sensitive data** like passwords |

**Why `kind: Deployment` was critical in this task:**
- Created **3 replica pods** automatically
- Watches them **24/7**
- **Restarts** any pod that crashes
- Allows **scaling** up/down anytime

> If `kind: Pod` was used instead → only **1 pod**, no replicas, no self-healing → task would **fail!**

---

## 🖥️ Commands Used in This Task

### 1. Verify Cluster Access
```bash
kubectl get nodes
kubectl get nodes -o wide   # shows IP addresses
```

### 2. Apply YAML Files
```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

### 3. Verify Deployment
```bash
kubectl get deployment nginx-deployment
kubectl describe deployment nginx-deployment
```

### 4. Verify Pods (3 replicas)
```bash
kubectl get pods -l app=nginx
```

### 5. Verify Service
```bash
kubectl get service nginx-service
kubectl describe service nginx-service   # check Endpoints field
```

### 6. Verify Both Together
```bash
kubectl get pods,svc -l app=nginx
```

### 7. Test Accessibility
```bash
kubectl get nodes -o wide          # Step 1: Get Node IP
curl http://<NODE-IP>:30011        # Step 2: Use that IP to test
```

> In this task the Node IP was `10.244.247.237`, so the curl was:
> ```bash
> curl http://10.244.247.237:30011
> ```

---

## ✅ Verification Checklist

| Check | Command | What to Look For |
|---|---|---|
| Deployment ready | `kubectl get deployment nginx-deployment` | `3/3` under READY |
| 3 pods running | `kubectl get pods -l app=nginx` | All pods `Running` |
| Service created | `kubectl get service nginx-service` | `80:30011/TCP` |
| Service connected to pods | `kubectl describe service nginx-service` | 3 IPs under `Endpoints` |
| Nginx accessible | `curl http://<NODE-IP>:30011` | `Welcome to nginx!` HTML |

---

## 🧠 Full Picture — How Everything Fits Together

```
Step 1: Docker Hub has nginx:latest image (pre-built)
                      ⬇
Step 2: deployment.yaml tells K8s
        "Run 3 replicas of nginx:latest"
                      ⬇
Step 3: Kubernetes creates 3 Pods
        each labeled app=nginx
                      ⬇
Step 4: service.yaml tells K8s
        "Expose pods labeled app=nginx on port 30011"
                      ⬇
Step 5: User runs curl http://<NODE-IP>:30011
                      ⬇
Step 6: Traffic flows: 30011 → Service:80 → Pod:80
                      ⬇
Step 7: nginx serves Welcome page ✅
```

---

## 📝 Quick Reference Summary

```
nginx-deployment.yaml          nginx-service.yaml
─────────────────────          ──────────────────
apiVersion  → K8s API          apiVersion → K8s API
kind        → Deployment       kind       → Service
metadata    → Name tag         metadata   → Name tag
replicas    → 3 copies         type       → NodePort
selector    → Manage nginx     selector   → Find nginx pods
  pods       by label app=nginx  by label app=nginx
template    → Pod blueprint    ports      → 30011→80→80
containers  → nginx:latest
              on port 80
```

---

*Study material based on KodeKloud Kubernetes task — Nginx Deployment with NodePort Service*
