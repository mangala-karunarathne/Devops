# Kubernetes Deployment Rollback — Study Material

---

## 📋 Original Task

> **Scenario:** Earlier today, the Nautilus DevOps team deployed a new release for an application. However, a customer has reported a bug related to this recent release. Consequently, the team aims to revert to the previous version.
>
> There exists a deployment named `nginx-deployment`; initiate a rollback to the previous revision.
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 🧠 Key Concepts

### What is a Kubernetes Deployment?
A **Deployment** is a Kubernetes object that manages a set of identical Pods. It ensures the desired number of replicas are running and supports rolling updates and rollbacks.

### What is a Rollback?
A rollback reverts a deployment to a previously known good state (revision). Kubernetes keeps a history of revisions for each deployment, making it easy to undo changes.

### What is a Revision?
Every time a deployment is updated (e.g., new image, config change), Kubernetes saves the old state as a **revision**. You can roll back to any saved revision.

---

## 🔍 How to Find the Deployment Name

### If the task tells you:
Use the name directly — in this case it was `nginx-deployment`.

### If the task does NOT tell you:
```bash
kubectl get deployments
```
**Output:**
```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment    3/3     3            3           12m
```
> ✅ Always start here in real-world scenarios to discover available deployments.

---

## 🛠️ Step-by-Step Commands

### Step 1: Check Deployment Status
```bash
kubectl get deployment nginx-deployment
```
**Output:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           10m
```

---

### Step 2: View Rollout History
```bash
kubectl rollout history deployment/nginx-deployment
```
**Actual Output from this task:**
```
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine-perl --record=true
```
> - **Revision 2** = buggy release (`nginx:alpine-perl`) — deployed today
> - **Revision 1** = previous stable version — target for rollback

---

### Step 3: (Best Practice) Inspect the Target Revision Before Rolling Back
```bash
kubectl rollout history deployment/nginx-deployment --revision=1
```
**Output:**
```
Pod Template:
  Containers:
   nginx-container:
    Image:  nginx:1.16    ← confirms what you're rolling back to
```
> ⚠️ In this task, this step was skipped. The image `nginx:1.16` was only confirmed **after** the rollback via `kubectl describe`. Always inspect first in production!

---

### Step 4: Perform the Rollback
```bash
kubectl rollout undo deployment/nginx-deployment
```
**Output:**
```
deployment.apps/nginx-deployment rolled back
```
> To roll back to a **specific revision** (not just the previous one):
> ```bash
> kubectl rollout undo deployment/nginx-deployment --to-revision=1
> ```

---

### Step 5: Monitor the Rollback
```bash
kubectl rollout status deployment/nginx-deployment
```
**Output:**
```
deployment "nginx-deployment" successfully rolled out
```

---

### Step 6: Verify the Rollback
```bash
kubectl get deployment nginx-deployment
kubectl describe deployment nginx-deployment | grep -i image
```
**Actual Output from this task:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           12m

    Image:  nginx:1.16
```
> ✅ Confirms the image reverted to `nginx:1.16`

---

### Step 7: Confirm Revision History Updated
```bash
kubectl rollout history deployment/nginx-deployment
```
**Actual Output from this task:**
```
REVISION  CHANGE-CAUSE
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine-perl --record=true
3         <none>
```
> Kubernetes does **not** delete old revisions. Instead, it appends the rolled-back config as a **new revision (3)**. Revision 1 becomes Revision 3 after the undo.

---

## 📊 What Happened — Visual Timeline

```
Revision 1  →  nginx:1.16          (original stable version)
     ↓
Revision 2  →  nginx:alpine-perl   (buggy release — today's deployment)
     ↓
Revision 3  →  nginx:1.16          (rollback — Kubernetes re-applied Revision 1 as new revision)
```

---

## 📚 Command Quick Reference

| Purpose | Command |
|---|---|
| List all deployments | `kubectl get deployments` |
| Check deployment status | `kubectl get deployment <name>` |
| View rollout history | `kubectl rollout history deployment/<name>` |
| Inspect a specific revision | `kubectl rollout history deployment/<name> --revision=<N>` |
| Rollback to previous revision | `kubectl rollout undo deployment/<name>` |
| Rollback to specific revision | `kubectl rollout undo deployment/<name> --to-revision=<N>` |
| Monitor rollback progress | `kubectl rollout status deployment/<name>` |
| Verify image after rollback | `kubectl describe deployment <name> \| grep -i image` |

---

## 🔗 Kubernetes vs Git Analogy

| Git | Kubernetes |
|---|---|
| `git log` | `kubectl rollout history deployment/<name>` |
| `git show <commit>` | `kubectl rollout history deployment/<name> --revision=N` |
| `git revert` | `kubectl rollout undo deployment/<name>` |

---

## ⚠️ Important Notes

1. **`--revision=N` is read-only** — it only inspects, never changes anything.
2. **Rollback creates a new revision** — it does not delete the buggy revision from history.
3. **`--record=true` flag** — records the command used in `CHANGE-CAUSE` column of rollout history. This is how Revision 2 showed the full `kubectl set image` command.
4. **Default rollback** — `kubectl rollout undo` always goes to the **immediately previous** revision unless `--to-revision` is specified.
5. **All 3 replicas must be healthy** after rollback — confirm with `READY 3/3`.

---

*Study material generated from a live Kubernetes rollback task on jump-host.*
