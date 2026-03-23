# 📘 Kubernetes Rolling Update Guide (nginx Deployment)

---

## 🧾 Original Task

An application currently running on the Kubernetes cluster employs the nginx web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image `nginx:1.17` with the latest updates.

### Objective:
- Execute a rolling update for this application
- Deployment name: `nginx-deployment`
- Update image to: `nginx:1.17`
- Ensure all pods are operational after update

---

## 🧠 Key Concepts

### 🔹 What is a Rolling Update?
A rolling update is a deployment strategy where:
- Pods are updated gradually
- Old pods are replaced with new ones one by one
- Ensures zero downtime

---

## ⚙️ Step-by-Step Execution

### ✅ Step 1: Check Existing Pods
```bash
kubectl get pods
```

### Expected Output
```
nginx-deployment-xxxxx   1/1   Running
```

---

### ✅ Step 2: Check Current Image Version
```bash
kubectl describe pod <pod-name> | grep Image
```

### Expected Output
```
Image: nginx:1.16
```

---

### ✅ Step 3: Identify Container Name
```bash
kubectl get deployment nginx-deployment -o yaml | grep name:
```

### Example Output
```
name: nginx-container
```

---

### ✅ Step 4: Perform Rolling Update
```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.17
```

### Expected Output
```
deployment.apps/nginx-deployment image updated
```

---

### ✅ Step 5: Monitor Rollout
```bash
kubectl rollout status deployment/nginx-deployment
```

### Expected Output
```
Waiting for deployment "nginx-deployment" rollout to finish...
deployment "nginx-deployment" successfully rolled out
```

---

### ✅ Step 6: Verify New Pods
```bash
kubectl get pods
```

### Expected Output
```
nginx-deployment-newhash-xxxxx   1/1   Running
```

---

### ✅ Step 7: Confirm Updated Image
```bash
kubectl describe pod <new-pod-name> | grep Image
```

### Expected Output
```
Image: nginx:1.17
```

---

## 🚨 Troubleshooting

### ❌ Error: Container Not Found
```
error: unable to find container named "nginx"
```

### ✅ Fix:
- Check container name using:
```bash
kubectl get deployment nginx-deployment -o yaml
```
- Use correct name in update command

---

## 🔄 Rollback (If Needed)

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

## 📊 Useful Commands Summary

| Purpose | Command |
|--------|--------|
| List pods | kubectl get pods |
| Check deployment | kubectl get deployment |
| Describe pod | kubectl describe pod <pod-name> |
| Update image | kubectl set image deployment/... |
| Check rollout | kubectl rollout status deployment/... |
| Rollback | kubectl rollout undo deployment/... |

---

## 🧪 Interview Tips

### Common Question:
**How do you perform a rolling update in Kubernetes?**

### Answer:
1. Update image using `kubectl set image`
2. Monitor using `kubectl rollout status`
3. Verify pods and image

---

## ✅ Final Checklist

- [x] Deployment exists
- [x] Container name identified
- [x] Image updated
- [x] Rollout successful
- [x] All pods running
- [x] Image verified

---

## 🚀 Notes

- Rolling updates ensure zero downtime
- Kubernetes automatically manages pod replacement
- Always verify after deployment

---

**End of Guide**

