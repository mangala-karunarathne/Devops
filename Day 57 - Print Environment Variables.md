# Kubernetes - Environment Variables in Pods
### Study Material | KodeKloud Lab Task

---

## 📋 Original Task

> **Nautilus DevOps Team — Prerequisites Setup**
>
> There is a sample deployment that needs to be tested. Below is the scenario configured on the Kubernetes cluster:
>
> 1. Create a `pod` named `print-envars-greeting`.
> 2. Configure spec as, the container name should be `print-env-container` and use `bash` image.
> 3. Create three environment variables:
>    - `GREETING` with value `Welcome to`
>    - `COMPANY` with value `Stratos`
>    - `GROUP` with value `Industries`
> 4. Use command `["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']` and set `restartPolicy` to `Never`.
> 5. Verify output using `kubectl logs -f print-envars-greeting`.

---

## ✅ Final Solution (YAML Manifest)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
    - name: print-env-container
      image: bash
      env:
        - name: GREETING
          value: "Welcome to"
        - name: COMPANY
          value: "Stratos"
        - name: GROUP
          value: "Industries"
      command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
```

> ⚠️ **Important:** When creating this file using `cat << EOF`, always use `<< 'EOF'` (with quotes) to prevent the shell from interpreting `$` variables before writing the file.

---

## 🛠️ Step-by-Step Commands

```bash
# Step 1 — Verify cluster access
kubectl get nodes

# Step 2 — Create the YAML file (note the quoted 'EOF')
cat > print-envars-greeting.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
    - name: print-env-container
      image: bash
      env:
        - name: GREETING
          value: "Welcome to"
        - name: COMPANY
          value: "Stratos"
        - name: GROUP
          value: "Industries"
      command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
EOF

# Step 3 — Verify the file
cat print-envars-greeting.yaml

# Step 4 — Apply the manifest
kubectl apply -f print-envars-greeting.yaml

# Step 5 — Check pod status
kubectl get pod print-envars-greeting

# Step 6 — Check logs
kubectl logs -f print-envars-greeting
```

---

## 📤 Expected Outputs

| Step | Command | Expected Output |
|------|---------|----------------|
| 1 | `kubectl get nodes` | Node with `Ready` status |
| 4 | `kubectl apply -f ...` | `pod/print-envars-greeting created` |
| 5 | `kubectl get pod ...` | `STATUS: Completed` |
| 6 | `kubectl logs -f ...` | `Welcome to Stratos Industries` |

---

## 🧠 Key Concepts Explained

### What is a Manifest?
A manifest is a **YAML blueprint file** that tells Kubernetes what you want it to create.
Just like an architect's blueprint tells a construction crew how to build a house —
you write the desired state, and Kubernetes makes it happen.

```
You write YAML  →  kubectl apply  →  Kubernetes builds it
  (blueprint)       (hand it over)     (construction)
```

---

### What Are Environment Variables in Kubernetes?

Environment variables are **settings/config injected into a container at startup**.
The container doesn't hardcode values — it reads whatever is passed in.

**Flow:**
```
YAML env section
      ↓
Kubernetes injects into container at startup
      ↓
$(VAR_NAME) is available inside the container
```

---

### restartPolicy Options

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `Never` | Run once, done | One-time jobs, scripts, tests |
| `OnFailure` | Restart only if it crashes | Batch jobs |
| `Always` | Always restart | Web servers, APIs |

---

### What is `control-plane` Role?

When you run `kubectl get nodes` and see `control-plane` as the ROLES:
- That node is the **brain/master** of the Kubernetes cluster
- It handles scheduling, API requests, and stores cluster state
- In lab environments (like KodeKloud), a single node acts as **both control-plane AND worker**
- `k3s` in the VERSION means you're running **K3s** — a lightweight Kubernetes for labs/testing

---

## 🔗 Connection to PERN Stack (What You Already Know)

You already use environment variables in your PERN stack projects!

**PERN Stack `.env` file:**
```env
DB_HOST=localhost
DB_PORT=5432
DB_PASSWORD=mysecretpassword
PORT=5000
```

**Read in Node.js:**
```javascript
const port = process.env.PORT
const dbPassword = process.env.DB_PASSWORD
```

**Kubernetes YAML:**
```yaml
env:
  - name: DB_HOST
    value: "localhost"
  - name: DB_PASSWORD
    value: "mysecretpassword"
```

### Side-by-Side Comparison

| | PERN Stack | Kubernetes |
|---|---|---|
| Where vars are defined | `.env` file | YAML manifest |
| How app reads it | `process.env.VAR_NAME` | `$(VAR_NAME)` |
| Keeps secrets safe | `.gitignore` the `.env` | Kubernetes Secrets |
| Change config | Edit `.env` | Edit YAML / ConfigMap |
| Purpose | Don't hardcode values | Same |

> **Bottom line:** Kubernetes env vars are just your `.env` file — but for containers running in a cluster instead of your local machine.

---

## 🌍 Real World Use Cases

### 1. Database Credentials
```yaml
env:
  - name: DB_HOST
    value: "mysql-service"
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:        # Pulled from Kubernetes Secret (like .gitignore)
        name: db-secret
        key: password
```

### 2. Different Config Per Environment
```yaml
# Development
env:
  - name: APP_ENV
    value: "development"
  - name: LOG_LEVEL
    value: "debug"

# Production
env:
  - name: APP_ENV
    value: "production"
  - name: LOG_LEVEL
    value: "error"
```
> Same Docker image, different behavior per environment.

### 3. Personalized Notification Apps
```yaml
env:
  - name: GREETING
    value: "Dear"
  - name: COMPANY
    value: "Google"
```
App sends: *"Dear Google Team, your invoice is ready."*
Change env vars → no code change needed.

### 4. Microservices Communication
```yaml
env:
  - name: PAYMENT_SERVICE_URL
    value: "http://payment-service:8080"
  - name: INVENTORY_SERVICE_URL
    value: "http://inventory-service:9090"
```

---

## 🐛 Troubleshooting

### Empty logs (blank output)
**Cause:** Shell expanded `$(VAR)` before writing the file — variables became empty strings.

**Fix:** Use `<< 'EOF'` instead of `<< EOF` when creating the file.

```bash
# Wrong — shell interprets $() before writing
cat > file.yaml << EOF

# Correct — shell writes literally
cat > file.yaml << 'EOF'
```

### Other common issues

| Issue | Fix |
|-------|-----|
| `STATUS: Error` | Run `kubectl describe pod print-envars-greeting` |
| `STATUS: Pending` | Check nodes with `kubectl get nodes` |
| Wrong log output | Delete pod, fix YAML, reapply |

**Delete and recreate:**
```bash
kubectl delete pod print-envars-greeting
kubectl apply -f print-envars-greeting.yaml
```

---

## 📌 Quick Reference

```bash
kubectl get nodes                          # Check cluster
kubectl apply -f <file>.yaml              # Create/update resource
kubectl get pod <pod-name>                # Check pod status
kubectl logs -f <pod-name>                # View pod logs
kubectl describe pod <pod-name>           # Detailed pod info
kubectl delete pod <pod-name>             # Delete a pod
```

---

*Task completed on KodeKloud | Kubernetes Environment Variables Lab*
