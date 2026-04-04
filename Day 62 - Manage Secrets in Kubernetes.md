# Kubernetes Secrets — Study Material
### Task: Storing Licence Keys Securely in Kubernetes

---

## 📋 Original Task

> The Nautilus DevOps team is working to deploy some tools in a Kubernetes cluster. Some of the tools are licence based so that licence information needs to be stored securely within the Kubernetes cluster. Therefore, the team wants to utilize Kubernetes secrets to store those secrets.
>
> **Requirements:**
> 1. A secret key file `blog.txt` already exists under the `/opt/` directory. Create a **generic secret** named `blog`, containing the password/licence-number present in `blog.txt`.
> 2. Create a **pod** named `secret-nautilus`.
> 3. Configure the pod's spec:
>    - Container name: `secret-container-nautilus`
>    - Image: `ubuntu:latest` (always mention the tag)
>    - Use `sleep` command so the container remains in running state
>    - Consume the created secret and **mount it under `/opt/demo`** within the container
> 4. Verify by exec-ing into the container and checking the secret key under `/opt/demo`

---

## 🧠 Concepts Explained

### What is a Kubernetes Secret?

A Kubernetes Secret is an object that stores sensitive data such as:
- Passwords
- API keys
- Licence keys
- TLS certificates
- SSH keys

Secrets are stored in **etcd** (Kubernetes internal database) and are **base64 encoded** by default. They are kept separate from application code and container images.

### Why Not Just Copy the File Into the Container?

| Approach | Problem |
|---|---|
| Hardcode in Docker image | Anyone who pulls the image can read it |
| Copy file to each node | Difficult to manage, insecure |
| Kubernetes Secret | Centrally managed, access controlled, injected at runtime |

### Two Ways to Use Secrets in Pods

| Method | How | Use Case |
|---|---|---|
| **Volume Mount** | Secret appears as files inside container | Config files, certificates, licence keys |
| **Environment Variable** | Secret value injected as env var | API keys, passwords for apps |

> In this task, we used the **Volume Mount** approach.

---

## 🛠️ Step-by-Step Solution

### Step 1 — Read the Secret File

```bash
cat /opt/blog.txt
```

This shows the licence key / password that will be stored as a secret.

---

### Step 2 — Create the Generic Secret

```bash
kubectl create secret generic blog --from-file=password=/opt/blog.txt
```

**Breaking down the command:**

| Part | Meaning |
|---|---|
| `kubectl create secret` | Create a secret object |
| `generic` | Type of secret (Opaque) |
| `blog` | Name of the secret |
| `--from-file=password=` | Key name inside the secret |
| `/opt/blog.txt` | File whose contents become the secret value |

**Verify the secret was created:**

```bash
kubectl describe secret blog
```

Expected output:
```
Name:         blog
Namespace:    default
Type:         Opaque

Data
====
password:  7 bytes
```

> Note: Kubernetes never shows the actual secret value in describe — only the byte size. This is intentional for security.

---

### Step 3 — Create the Pod YAML Manifest

```bash
cat <<EOF > secret-nautilus.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-nautilus
spec:
  containers:
  - name: secret-container-nautilus
    image: ubuntu:latest
    command: ["sleep", "infinity"]
    volumeMounts:
    - name: secret-volume
      mountPath: /opt/demo
  volumes:
  - name: secret-volume
    secret:
      secretName: blog
EOF
```

**YAML explained section by section:**

```yaml
apiVersion: v1          # Kubernetes API version for Pod
kind: Pod               # Object type
metadata:
  name: secret-nautilus # Name of the pod
spec:
  containers:
  - name: secret-container-nautilus   # Container name
    image: ubuntu:latest              # Always include the tag!
    command: ["sleep", "infinity"]    # Keeps container running
    volumeMounts:
    - name: secret-volume             # Must match volume name below
      mountPath: /opt/demo            # Where secret appears inside container
  volumes:
  - name: secret-volume               # Volume name (referenced above)
    secret:
      secretName: blog                # Name of the Kubernetes secret
```

---

### Step 4 — Deploy the Pod

```bash
kubectl apply -f secret-nautilus.yaml
```

Expected output:
```
pod/secret-nautilus created
```

---

### Step 5 — Verify Pod is Running

```bash
kubectl get pod secret-nautilus
```

Expected output:
```
NAME              READY   STATUS    RESTARTS   AGE
secret-nautilus   1/1     Running   0          30s
```

> If STATUS shows `ContainerCreating`, wait a few seconds and run again.

---

### Step 6 — Verify Secret Inside the Container

```bash
kubectl exec -it secret-nautilus -c secret-container-nautilus -- ls /opt/demo
```

Expected output:
```
password
```

```bash
kubectl exec -it secret-nautilus -c secret-container-nautilus -- cat /opt/demo/password
```

Expected output:
```
5ecur3
```

This should match the content of `/opt/blog.txt` — confirming the secret is correctly mounted. ✅

---

## 🔄 How the Data Flows

```
/opt/blog.txt          Kubernetes Secret         Pod Container
(on jump-host)    →    (stored in etcd)    →    /opt/demo/password
                        name: blog               (read by application)
                        key: password
```

---

## 📚 Key kubectl Commands for Secrets

```bash
# Create secret from file
kubectl create secret generic <name> --from-file=<key>=<filepath>

# Create secret from literal value
kubectl create secret generic <name> --from-literal=<key>=<value>

# List all secrets
kubectl get secrets

# Describe a secret (shows keys, not values)
kubectl describe secret <name>

# View encoded secret value
kubectl get secret <name> -o yaml

# Decode a secret value
kubectl get secret <name> -o jsonpath='{.data.<key>}' | base64 --decode

# Delete a secret
kubectl delete secret <name>
```

---

## 🌍 Real-World Use Cases

| Use Case | Secret Key | Mounted Path |
|---|---|---|
| Database password | `db-password` | `/etc/db/password` |
| Licence key (like this task) | `password` | `/opt/demo/password` |
| TLS certificate | `tls.crt`, `tls.key` | `/etc/ssl/` |
| AWS credentials | `access-key`, `secret-key` | `/root/.aws/` |
| JWT signing key | `jwt-secret` | `/etc/auth/secret` |

---

## ⚠️ Common Mistakes to Avoid

1. **Forgetting the image tag** — always use `ubuntu:latest` not just `ubuntu`
2. **Volume name mismatch** — `volumeMounts.name` must exactly match `volumes.name`
3. **Wrong secret name** — `secretName` must match the secret you created
4. **Not waiting for Running state** — always confirm pod is Running before verifying
5. **Sharing actual secrets in chat or logs** — never expose real credentials

---

## 🔐 Security Best Practices (Real World)

- Enable **etcd encryption at rest** in production clusters
- Use **RBAC** to restrict who can read secrets
- Consider tools like **HashiCorp Vault** or **AWS Secrets Manager** for advanced secret management
- Avoid printing secrets in logs or terminal output
- Rotate secrets regularly

---

## 📖 Further Reading

- [Kubernetes Official Docs — Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes Official Docs — Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

*Study material generated after completing the Nautilus DevOps Kubernetes Secrets lab task.*
