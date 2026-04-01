# Grafana Deployment on Kubernetes — Study Material

---

## 📋 Original Task

> **Nautilus DevOps Team — Grafana Setup on Kubernetes**
>
> The Nautilus DevOps team is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on a Kubernetes cluster. Below are the details:
>
> 1. Create a deployment named `grafana-deployment-datacenter` using any Grafana image for the Grafana app. Set other parameters as per your choice.
> 2. Create a `NodePort` type service with `nodePort: 32000` to expose the app.
>
> _You do not need to make any configuration changes inside the Grafana app once deployed; just make sure you can access the Grafana login page._
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🧠 Key Concepts

### What is Grafana?
Grafana is an open-source **monitoring and analytics platform**. It lets you visualize metrics, logs, and data from various sources through dashboards.

### What is Kubernetes?
Kubernetes (K8s) is a system that **automates deploying, scaling, and managing containerized applications**. Think of it as an operating system for your cluster of servers.

### What is a Pod?
A Pod is the **smallest unit in Kubernetes**. It wraps one or more containers. Grafana runs inside a Pod.

### What is a Deployment?
A Deployment is a Kubernetes object that **manages Pods**. It ensures:
- The desired number of replicas are always running
- If a Pod crashes, it is automatically restarted

### What is a Service?
A Service is a Kubernetes object that **exposes a Pod to network traffic**. Without a Service, a Pod is only reachable from inside the cluster.

### What is NodePort?
NodePort is a type of Service that **opens a specific port on every Node** in the cluster, allowing external access. Traffic hitting `<NodeIP>:<nodePort>` is forwarded to the target Pod.

---

## 🗺️ Architecture — Traffic Flow

```
Browser / curl
      |
      ▼
Node IP : 32000          ← NodePort (the "door")
      |
      ▼
grafana-service          ← Kubernetes Service
      |
      ▼
grafana Pod : 3000       ← Grafana container inside the cluster
```

---

## 📁 YAML Files Used

### 1. Deployment — `grafana-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-datacenter
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: "admin"
```

**Key fields explained:**

| Field | Value | Meaning |
|-------|-------|---------|
| `kind` | Deployment | Type of Kubernetes object |
| `name` | grafana-deployment-datacenter | Name of the deployment |
| `replicas` | 1 | Run 1 Pod at a time |
| `image` | grafana/grafana:latest | Docker image to use |
| `containerPort` | 3000 | Port Grafana listens on inside container |
| `GF_SECURITY_ADMIN_PASSWORD` | admin | Sets the Grafana admin password |

---

### 2. Service — `grafana-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  labels:
    app: grafana
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 32000
```

**Key fields explained:**

| Field | Value | Meaning |
|-------|-------|---------|
| `type` | NodePort | Expose the service on a static port on each Node |
| `selector` | app: grafana | Route traffic to pods with this label |
| `port` | 3000 | Port exposed by the Service internally |
| `targetPort` | 3000 | Port on the Pod to forward traffic to |
| `nodePort` | 32000 | External port on the Node to access the app |

---

## 🖥️ Commands Used — Step by Step

### Verify Cluster Access
```bash
kubectl get nodes
```

### Apply the Deployment
```bash
kubectl apply -f grafana-deployment.yaml
```

### Apply the Service
```bash
kubectl apply -f grafana-service.yaml
```

### Verify Deployment is Ready
```bash
kubectl get deployments
```

### Verify Pod is Running
```bash
kubectl get pods -l app=grafana
```

### Verify Service is Created
```bash
kubectl get svc grafana-service
```

### Get Node IP
```bash
kubectl get nodes -o wide
```

### Test with curl
```bash
# Basic check
curl http://<NODE-IP>:32000

# Check HTTP headers only
curl -I http://<NODE-IP>:32000

# Follow redirect to login page
curl -L http://<NODE-IP>:32000/login
```

**In this task, the Node IP was `10.244.247.211`, so:**
```bash
curl http://10.244.247.211:32000
```

### Access in Browser
```
http://<NODE-IP>:32000
```
Default credentials:
- **Username:** `admin`
- **Password:** `admin`

---

## 🔍 Expected Outputs

### `kubectl get nodes`
```
NAME           STATUS   ROLES           AGE   VERSION
node01         Ready    worker          10d   v1.29.0
controlplane   Ready    control-plane   10d   v1.29.0
```

### `kubectl get deployments`
```
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
grafana-deployment-datacenter   1/1     1            1           30s
```

### `kubectl get pods -l app=grafana`
```
NAME                                             READY   STATUS    RESTARTS   AGE
grafana-deployment-datacenter-6d7f9c8b4-xk9p2   1/1     Running   0          45s
```

### `kubectl get svc grafana-service`
```
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana-service   NodePort   10.96.145.212   <none>        3000:32000/TCP   20s
```

### `curl -I http://10.244.247.211:32000`
```
HTTP/1.1 302 Found
Location: /login
```

---

## 🛠️ Troubleshooting Cheatsheet

| Problem | Command |
|---------|---------|
| Pod not starting | `kubectl describe pod -l app=grafana` |
| Check pod logs | `kubectl logs -l app=grafana` |
| Service not routing | `kubectl describe svc grafana-service` |
| Port not accessible | `kubectl get endpoints grafana-service` |
| Delete and recreate all | `kubectl delete -f grafana-deployment.yaml && kubectl delete -f grafana-service.yaml` |

---

## 📌 Important Notes

- Grafana listens on **port 3000** by default inside the container.
- NodePort range in Kubernetes is **30000–32767**.
- The `selector` in the Service must **match the labels** on the Pod — this is how Kubernetes links them together.
- A Deployment with `replicas: 1` means only **one Pod** runs at a time. You can scale it up with `kubectl scale deployment grafana-deployment-datacenter --replicas=3`.
- `grafana/grafana:latest` pulls the latest Grafana image from **Docker Hub**.

---

## 📚 Further Reading

- [Grafana Official Docs](https://grafana.com/docs/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [NodePort Service Type](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)

---

*Study material prepared based on a hands-on Kubernetes task — Nautilus DevOps Grafana Deployment.*
