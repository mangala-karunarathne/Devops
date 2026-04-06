# Kubernetes Troubleshooting Study Guide
### Task: Deploy a Python Flask App on Kubernetes

---

## 📋 Original Task

> One of the DevOps engineers was trying to deploy a Python Flask app on a Kubernetes cluster. Unfortunately, due to some mis-configuration, the application is not coming up. Please take a look into it and fix the issues. Application should be accessible on the specified nodePort.
>
> 1. The deployment name is `python-deployment-devops`, it's using `poroko/flask-demo-app` image. The deployment and service of this app is already deployed.
> 2. nodePort should be `32345` and targetPort should be the Python Flask app's default port.
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🔍 Diagnosis Phase

### Step 1: Check Overall Cluster State

Always start by getting a high-level view:

```bash
kubectl get deployments
kubectl get pods
kubectl get svc
```

**What to look for:**
- Deployment `READY` column — e.g., `0/1` means pods are not running
- Pod `STATUS` — tells you what kind of error you have
- Service `PORT(S)` — shows port mapping (e.g., `8080:32345/TCP`)

**Actual output we saw:**
```
NAME                                   READY   STATUS             RESTARTS   AGE
python-deployment-devops-7dd8f6ddf8-v6xtf   0/1   ImagePullBackOff   0          11m

NAME                    TYPE       CLUSTER-IP      PORT(S)          AGE
python-service-devops   NodePort   10.43.14.167    8080:32345/TCP   12m
```

---

### Step 2: Describe the Pod (Deep Inspection)

```bash
kubectl describe pod <pod-name>
# OR using label selector
kubectl describe pod -l app=python-deployment-devops
```

**Why this matters:** The `Events` section at the bottom shows exactly what Kubernetes tried to do and what failed.

**Actual Events output we saw:**
```
Events:
  Normal   Pulling    kubelet   Pulling image "poroko/flask-app-demo"
  Warning  Failed     kubelet   Failed to pull image "poroko/flask-app-demo":
                                failed to resolve reference: pull access denied,
                                repository does not exist
  Warning  Failed     kubelet   Error: ErrImagePull
  Warning  Failed     kubelet   Error: ImagePullBackOff
```

**Root cause identified:** Image name typo — `flask-app-demo` instead of `flask-demo-app`

---

### Step 3: Check the Service Configuration

```bash
kubectl get svc
kubectl describe svc python-service-devops | grep TargetPort
```

**Actual output we saw:**
```
NAME                    TYPE       PORT(S)
python-service-devops   NodePort   8080:32345/TCP
TargetPort: 8080/TCP
```

**Root cause identified:** `targetPort` was `8080` but Flask runs on `5000`

---

## 🐛 Issues Found & Fixes Applied

### Issue #1 — Wrong Docker Image Name

| | Value |
|---|---|
| ❌ Wrong | `poroko/flask-app-demo` |
| ✅ Correct | `poroko/flask-demo-app` |

**How to find the container name first:**
```bash
kubectl get deployment python-deployment-devops \
  -o jsonpath='{.spec.template.spec.containers[*].name}'
```
Output: `python-container-devops`

**Fix:**
```bash
kubectl set image deployment/python-deployment-devops \
  python-container-devops=poroko/flask-demo-app
```

Expected output:
```
deployment.apps/python-deployment-devops image updated
```

---

### Issue #2 — Wrong targetPort on Service

| | Value |
|---|---|
| ❌ Wrong | `targetPort: 8080` |
| ✅ Correct | `targetPort: 5000` (Flask default) |

**Fix attempt using patch (didn't work due to named port):**
```bash
kubectl patch svc python-service-devops \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/ports/0/targetPort", "value": 5000}]'
```

**Fix using direct edit (worked):**
```bash
kubectl edit svc python-service-devops
```

In the vi editor:
1. Find `targetPort: 8080`
2. Press `i` (insert mode)
3. Change `8080` → `5000`
4. Press `Esc`
5. Type `:wq` and press `Enter`

---

## ✅ Verification Phase

### Verify Pod is Running
```bash
kubectl get pods -w
```

Expected:
```
NAME                                        READY   STATUS    RESTARTS   AGE
python-deployment-devops-6fdb46f474-hxq7v   1/1     Running   0          53s
```

### Verify Service targetPort
```bash
kubectl get svc
kubectl describe svc python-service-devops | grep TargetPort
```

Expected:
```
NAME                    TYPE       PORT(S)
python-service-devops   NodePort   5000:32345/TCP

TargetPort: 5000/TCP
```

### Test the Application
```bash
# Get node IP
kubectl get nodes -o wide

# Hit the app
curl http://<node-ip>:32345
```

**Actual final output:**
```
Hello World Pyvo 1!
```

🎉 **Application successfully deployed!**

---

## 📚 Core Kubernetes Concepts

### Port Mapping Explained

This is the most confusing part for beginners — there are 3 different ports:

```
Internet / User Browser
        │
        ▼
  nodePort (32345)      ← Port exposed on the Kubernetes Node (external access)
        │
        ▼
    port (5000)         ← Port the Service listens on inside the cluster
        │
        ▼
  targetPort (5000)     ← Port the app listens on INSIDE the container
        │
        ▼
  Flask App (5000)      ← Your actual application
```

**Simple rule:** `targetPort` must always match the port your application is coded to listen on.

| App | Default Port |
|-----|-------------|
| Flask (Python) | 5000 |
| Node.js | 3000 |
| Nginx | 80 |
| Spring Boot (Java) | 8080 |
| Django (Python) | 8000 |

---

### Pod Status Reference

| Status | Meaning | Common Cause |
|--------|---------|--------------|
| `Pending` | Pod waiting to be scheduled | Not enough resources on nodes |
| `ImagePullBackOff` | Can't download Docker image | Wrong image name, private registry |
| `ErrImagePull` | Initial image pull failure | Same as above |
| `CrashLoopBackOff` | App starts but keeps crashing | Bug in app, wrong config, missing env vars |
| `Running` | Everything is working ✅ | — |
| `Completed` | One-time job finished | — |
| `OOMKilled` | Out of memory | Need higher memory limits |

---

### Kubernetes Object Types Used

| Object | Purpose |
|--------|---------|
| `Deployment` | Manages pods, handles rolling updates, ensures desired pod count |
| `Pod` | Smallest unit — runs your container |
| `Service (NodePort)` | Exposes your app to external traffic via a port on the node |

---

## 🛠️ Essential kubectl Commands Cheatsheet

### Inspection Commands
```bash
# Get overview of resources
kubectl get pods
kubectl get deployments
kubectl get svc
kubectl get nodes -o wide          # Shows node IPs

# Watch pods in real-time
kubectl get pods -w

# Detailed info with events (most useful for debugging)
kubectl describe pod <pod-name>
kubectl describe deployment <name>
kubectl describe svc <name>

# View logs from inside the container
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # Logs from crashed pod

# Get raw YAML of any resource
kubectl get deployment <name> -o yaml
kubectl get svc <name> -o yaml

# Extract specific fields with jsonpath
kubectl get deployment python-deployment-devops \
  -o jsonpath='{.spec.template.spec.containers[*].name}'
```

### Fix / Edit Commands
```bash
# Update container image
kubectl set image deployment/<deploy-name> <container-name>=<new-image>

# Patch a resource field
kubectl patch svc <svc-name> --type='json' \
  -p='[{"op": "replace", "path": "/spec/ports/0/targetPort", "value": 5000}]'

# Live edit a resource (opens vi editor)
kubectl edit deployment <name>
kubectl edit svc <name>

# Delete and recreate a pod (deployment will recreate it)
kubectl delete pod <pod-name>

# Scale a deployment
kubectl scale deployment <name> --replicas=3
```

### vi Editor Quick Reference (used in kubectl edit)
```
i         → Enter insert/edit mode
Esc       → Exit insert mode
:wq       → Save and quit
:q!       → Quit without saving
/word     → Search for "word"
n         → Next search result
```

---

## 🔁 Troubleshooting Flow Diagram

```
App not working?
       │
       ▼
kubectl get pods
       │
       ├── ImagePullBackOff ──► Check image name in deployment
       │                        kubectl describe pod → Events section
       │                        Fix: kubectl set image ...
       │
       ├── CrashLoopBackOff ──► Check app logs
       │                        kubectl logs <pod-name> --previous
       │
       ├── Pending ──────────► Check node resources
       │                        kubectl describe pod → Events section
       │
       └── Running but can't access ──► Check Service
                                         kubectl get svc
                                         kubectl describe svc
                                         Verify: nodePort, targetPort, port
```

---

## 💡 Golden Rules for Beginners

1. **Always `describe` first** — the Events section tells you exactly what went wrong
2. **Image names are case-sensitive** — must exactly match DockerHub or your registry
3. **`targetPort` = your app's port** — Flask=5000, not 8080
4. **`kubectl edit` is your safety net** — when patches don't work, edit directly
5. **`kubectl get pods -w`** — watch mode shows real-time status changes
6. **Check logs for CrashLoopBackOff** — `kubectl logs <pod>` reveals app-level errors
7. **nodePort range is 30000–32767** — Kubernetes restricts external ports to this range

---

## 📝 Summary of This Task

| # | Problem | Fix Command |
|---|---------|------------|
| 1 | Image `poroko/flask-app-demo` didn't exist | `kubectl set image deployment/python-deployment-devops python-container-devops=poroko/flask-demo-app` |
| 2 | Service `targetPort` was `8080`, Flask needs `5000` | `kubectl edit svc python-service-devops` → changed targetPort to 5000 |

**Final result:** App accessible at `http://10.244.73.187:32345` → `Hello World Pyvo 1!` 🎉
