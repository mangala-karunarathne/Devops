# Kubernetes Debugging Study Material
### Task: Fix Redis Deployment on Kubernetes Cluster

---

## 📋 Original Task

> Last week, the Nautilus DevOps team deployed a Redis app on a Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but he made some mistakes and the app went down. We need to fix this as soon as possible.
>
> - **Deployment name:** `redis-deployment`
> - The pods are **not in running state**
> - The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster

---

## 🔍 Debugging Workflow

### Step 1 — Check Deployment Status
```bash
kubectl get deployment redis-deployment
```
- Look at the `READY` column. `0/1` means no pods are healthy.

---

### Step 2 — Check Pod Status
```bash
kubectl get pods | grep redis
```
Common bad statuses to watch for:

| Status | Meaning |
|---|---|
| `CrashLoopBackOff` | Container keeps crashing and restarting |
| `ImagePullBackOff` | Can't pull the container image |
| `Pending` | Not scheduled onto a node yet |
| `OOMKilled` | Ran out of memory |
| `ContainerCreating` | Stuck creating (volume/image issue) |

---

### Step 3 — Describe the Pod ⭐ Most Important Step
```bash
kubectl describe pod <pod-name>
```
- Always scroll to the **Events** section at the bottom
- This reveals the **root cause** 90% of the time

---

### Step 4 — Check Logs
```bash
kubectl logs <pod-name>
# If pod has restarted:
kubectl logs <pod-name> --previous
```

---

### Step 5 — Inspect Deployment Config
```bash
kubectl describe deployment redis-deployment
kubectl get deployment redis-deployment -o yaml
```

---

## 🐛 Bugs Found in This Task

### Bug 1 — ConfigMap Name Typo

**Symptom (from `kubectl describe pod`):**
```
Warning  FailedMount  97s (x10 over 5m47s)  kubelet
MountVolume.SetUp failed for volume "config" : configmap "redis-conig" not found
```

**Root Cause:**  
The deployment's volume reference had a typo: `redis-conig` instead of `redis-config`.

**Verification:**
```bash
kubectl get configmap | grep redis
# Output: redis-config   2   11m  ← correct one exists
```

**Fix:**
```bash
kubectl get deployment redis-deployment -o yaml | \
  sed 's/redis-conig/redis-config/g' | \
  kubectl apply -f -
```

> ⚠️ **Lesson:** `sed` replaces ALL occurrences in the YAML. If the typo pattern appears elsewhere (e.g., image name), it may cause unintended changes. Prefer targeted fixes when possible.

---

### Bug 2 — Image Tag Typo

**Symptom (from `kubectl get pods`):**
```
redis-deployment-85cd7f84f5-vzffg   0/1   ImagePullBackOff   0   4m25s
```

**Root Cause:**  
The image was set to `redis:alpin` instead of `redis:alpine` (missing the `e`).

**Verification:**
```bash
kubectl describe deployment redis-deployment | grep Image
# Output: Image: redis:alpin  ← typo confirmed
```

**Find the correct container name:**
```bash
kubectl get deployment redis-deployment \
  -o jsonpath='{.spec.template.spec.containers[*].name}'
# Output: redis-container
```

> 💡 **Note:** `jsonpath` output has no newline, so it appears joined with your shell prompt. The container name is everything before `thor@jump-host`.

**Fix:**
```bash
kubectl set image deployment/redis-deployment redis-container=redis:alpine
```

---

## ✅ Verification After Fix
```bash
kubectl rollout status deployment/redis-deployment
kubectl get pods | grep redis
```

**Healthy output:**
```
redis-deployment-5476b4ddd6-6n9hn   1/1   Running   0   39s
```
`1/1 Running` = pod is fully healthy ✅

---

## 📚 Key Commands Reference

| Command | Purpose |
|---|---|
| `kubectl get pods \| grep redis` | Check pod status |
| `kubectl describe pod <name>` | Detailed info + Events (root cause) |
| `kubectl describe deployment <name>` | Deployment config overview |
| `kubectl get deployment <name> -o yaml` | Full deployment YAML |
| `kubectl get configmap \| grep redis` | List ConfigMaps |
| `kubectl logs <pod>` | Container logs |
| `kubectl logs <pod> --previous` | Logs from crashed container |
| `kubectl set image deployment/<name> <container>=<image>` | Update container image |
| `kubectl edit deployment <name>` | Edit deployment in vi |
| `kubectl rollout status deployment/<name>` | Watch rollout progress |
| `kubectl get deployment <name> -o jsonpath='...'` | Extract specific fields |

---

## 🚨 Common Kubernetes Bugs (General Reference)

### Image Issues
- Wrong image tag (e.g., `redis:alpin` → `redis:alpine`)
- Private registry not authenticated → `ImagePullBackOff`
- Image doesn't exist on the registry

### ConfigMap / Secret Issues
- Name typos in volume references ← *happened here*
- ConfigMap deleted but deployment still references it
- Wrong key names inside the ConfigMap

### Resource Issues
- Memory/CPU limits too low → pod gets `OOMKilled`
- Node doesn't have enough resources → pod stays `Pending`

### Networking Issues
- Wrong `containerPort` defined
- Service selector labels don't match pod labels

### Volume Issues
- PersistentVolumeClaim not bound
- Wrong mount path inside the container

### Probe Issues
- Liveness/Readiness probe misconfigured → pod restarts in a loop even when the app is fine

---

## 💡 Golden Rules for Kubernetes Debugging

1. **Always start with `kubectl describe pod`** — Events section is your best friend
2. **Check `kubectl get configmap`** before assuming a ConfigMap is missing
3. **Use targeted fixes** (`kubectl set image`, `kubectl patch`) over broad `sed` replacements
4. **`Ctrl+C` on `kubectl rollout status`** is safe — it only cancels the watch, not the rollout
5. **`jsonpath` output has no newline** — don't confuse it with the shell prompt
6. **Two bugs can coexist** — fixing one may reveal another hiding underneath

---

*Study material generated from a live Kubernetes debugging session on the Nautilus DevOps cluster.*
