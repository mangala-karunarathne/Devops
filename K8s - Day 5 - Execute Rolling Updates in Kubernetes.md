# Kubernetes Rolling Update — Study Material

---

## 📋 Original Task

> An application currently running on the Kubernetes cluster employs the nginx web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image `nginx:1.18` with the latest updates.
>
> Execute a rolling update for this application, integrating the `nginx:1.18` image. The deployment is named `nginx-deployment`.
>
> Ensure all pods are operational post-update.
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## 📖 Concepts to Understand

### What is a Rolling Update?
A **Rolling Update** allows you to update a deployment's pods gradually — replacing old pods with new ones **without downtime**. Kubernetes ensures that a minimum number of pods are always running during the update.

### Key Fields in RollingUpdateStrategy
| Field | Description |
|---|---|
| `maxUnavailable` | Max number/percentage of pods that can be unavailable during the update |
| `maxSurge` | Max number/percentage of extra pods that can be created above desired count |

In this task, the strategy was: **25% max unavailable, 25% max surge**

---

## 🔧 Step-by-Step Commands Used

### Step 1: Check the Current Deployment
```bash
kubectl get deployment nginx-deployment
```
- Verifies the deployment exists and shows READY/UP-TO-DATE/AVAILABLE pod counts.

**Sample Output:**
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           10d
```

---

### Step 2: Check the Current Image
```bash
kubectl describe deployment nginx-deployment | grep -i image
```
- Confirms the image currently running before the update.

**Sample Output:**
```
    Image: nginx:1.17
```

---

### Step 3: Identify the Container Name
```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[*].name}'
```
- **Critical step** — you need the exact container name to use in the `set image` command.

**Sample Output:**
```
nginx-container
```

> ⚠️ In this task, the container name was `nginx-container`, **not** `nginx`. Always verify this before running the update command.

---

### Step 4: Apply the Rolling Update
```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18
```

**Command Breakdown:**
```
kubectl set image  deployment/nginx-deployment   nginx-container  =  nginx:1.18
                         ↑                            ↑                  ↑
                   Deployment Name            Container Name       New Image:Tag
```

**Sample Output:**
```
deployment.apps/nginx-deployment image updated
```

---

### Step 5: Monitor the Rollout
```bash
kubectl rollout status deployment/nginx-deployment
```
- Watches the rolling update in real time until completion.

**Sample Output:**
```
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
```

---

### Step 6: Verify All Pods Are Running
```bash
kubectl get pods -l app=nginx-app
```
- Confirms all pods are up and running with the new image.

**Sample Output:**
```
NAME                                      READY   STATUS    RESTARTS   AGE
nginx-deployment-7c9b6d7f8d-jkl78         1/1     Running   0          2m
nginx-deployment-7c9b6d7f8d-mno90         1/1     Running   0          2m
nginx-deployment-7c9b6d7f8d-pqr12         1/1     Running   0          2m
```

> ✅ The pod name hash changes after the update — this confirms new pods were created with the new image.

---

### Step 7: Confirm the New Image
```bash
kubectl describe deployment nginx-deployment | grep -i image
```

**Sample Output:**
```
    Image: nginx:1.18
```

---

### Step 8: View Rollout History (Optional)
```bash
kubectl rollout history deployment/nginx-deployment
```

**Sample Output:**
```
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

---

## 🔁 Rollback Command (If Something Goes Wrong)

```bash
kubectl rollout undo deployment/nginx-deployment
```
- Reverts the deployment to the previous revision instantly.

To rollback to a specific revision:
```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

---

## 📝 What Was Observed in This Task

From the `kubectl describe` output after the update:

```
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge

Pod Template:
  Labels: app=nginx-app
  Containers:
   nginx-container:
    Image: nginx:1.18
    Port: <none>
```

| Field | Value |
|---|---|
| Strategy | RollingUpdate |
| Container Name | `nginx-container` |
| Updated Image | `nginx:1.18` |
| Max Unavailable | 25% |
| Max Surge | 25% |

---

## 🗂️ Quick Reference — All Commands

| Purpose | Command |
|---|---|
| Check deployment | `kubectl get deployment nginx-deployment` |
| Check current image | `kubectl describe deployment nginx-deployment \| grep -i image` |
| Get container name | `kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[*].name}'` |
| Apply rolling update | `kubectl set image deployment/nginx-deployment <container-name>=nginx:1.18` |
| Monitor rollout | `kubectl rollout status deployment/nginx-deployment` |
| Verify pods | `kubectl get pods -l app=<label>` |
| Confirm new image | `kubectl describe deployment nginx-deployment \| grep -i image` |
| View history | `kubectl rollout history deployment/nginx-deployment` |
| Rollback | `kubectl rollout undo deployment/nginx-deployment` |

---

## ⚠️ Common Mistakes to Avoid

1. **Wrong container name** — Always verify the container name with `jsonpath` before running `set image`. Using the wrong name silently fails or errors.
2. **Wrong label selector** — Use the correct label (e.g., `app=nginx-app`) when filtering pods with `-l`.
3. **Not monitoring the rollout** — Always run `kubectl rollout status` to ensure the update completes successfully.
4. **Skipping pod verification** — After the rollout, confirm all pods are `1/1 Running` and none are in `CrashLoopBackOff` or `Error` state.

---

## 📚 Additional Reading

- [Kubernetes Official Docs — Rolling Update](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Deployments — Kubernetes Docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
