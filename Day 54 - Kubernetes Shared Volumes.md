# Kubernetes Shared Volume Study Material
### Topic: Sharing Volumes Among Containers in a Pod

---

## 📋 Original Task

> **Nautilus DevOps Team — Kubernetes Volume Sharing Task**
>
> We are working on an application that will be deployed on multiple containers within a pod on a Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario.
>
> **Requirements:**
> 1. Create a pod named `volume-share-xfusion`.
> 2. For the first container, use image `fedora:latest`, container should be named `volume-container-xfusion-1`, run a `sleep` command so it remains running. Volume `volume-share` should be mounted at path `/tmp/media`.
> 3. For the second container, use image `fedora:latest`, container should be named `volume-container-xfusion-2`, run a `sleep` command so it remains running. Volume `volume-share` should be mounted at path `/tmp/demo`.
> 4. Volume name should be `volume-share` of type `emptyDir`.
> 5. After creating the pod, exec into the first container and create a file `media.txt` with the content `Welcome to xFusionCorp Industries` under `/tmp/media`.
> 6. The file `media.txt` should be present under `/tmp/demo` on the second container as well, since they share the volume.

---

## 🧠 Concepts You Need to Know

### What is a Pod?
A **Pod** is the smallest deployable unit in Kubernetes. It can contain one or more containers that:
- Share the same network namespace (same IP address)
- Can share storage volumes
- Run together on the same node

Think of a Pod as a **small box** that holds your containers.

---

### What is an emptyDir Volume?
`emptyDir` is a type of Kubernetes volume that:
- Is created **empty** when the Pod starts
- Is shared between **all containers** in the Pod
- Is **deleted permanently** when the Pod is removed
- Lives as long as the Pod lives

Think of it like a **shared whiteboard** — all containers in the Pod can read and write to it, but it gets erased when the Pod is gone.

---

### What is a VolumeMount?
A `volumeMount` is how a container **connects to a volume**. Each container can mount the same volume at a **different path**:

| Container | Mount Path |
|-----------|-----------|
| container-1 | `/tmp/media` |
| container-2 | `/tmp/demo` |

Both paths point to the **same underlying storage**. A file written in one path is instantly visible in the other.

---

## 📄 The YAML File — Explained

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-xfusion
spec:
  containers:
    - name: volume-container-xfusion-1
      image: fedora:latest
      command: ["sleep", "10000"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/media

    - name: volume-container-xfusion-2
      image: fedora:latest
      command: ["sleep", "10000"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/demo

  volumes:
    - name: volume-share
      emptyDir: {}
```

### Line-by-Line Breakdown

| Field | Meaning |
|-------|---------|
| `apiVersion: v1` | Use the stable core Kubernetes API |
| `kind: Pod` | We are creating a Pod resource |
| `metadata.name` | Unique name to identify this Pod in the cluster |
| `spec` | Specification — describes what goes inside the Pod |
| `containers` | List of containers to run inside the Pod |
| `name` | Name of the container |
| `image: fedora:latest` | Docker image to use (fedora OS, latest version) |
| `command: ["sleep", "10000"]` | Keep the container alive by sleeping for 10000 seconds |
| `volumeMounts` | Where to attach the shared volume inside this container |
| `mountPath` | The folder path inside the container that maps to the volume |
| `volumes` | Defines the actual volumes available to the Pod |
| `emptyDir: {}` | Volume type — starts empty, shared between containers, deleted with Pod |

---

## 🛠️ Step-by-Step Commands

### Step 1 — Create the YAML file using heredoc (EOF)

```bash
cat <<EOF > volume-share-xfusion.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-xfusion
spec:
  containers:
    - name: volume-container-xfusion-1
      image: fedora:latest
      command: ["sleep", "10000"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/media
    - name: volume-container-xfusion-2
      image: fedora:latest
      command: ["sleep", "10000"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/demo
  volumes:
    - name: volume-share
      emptyDir: {}
EOF
```

> **What is EOF / heredoc?**
> - `cat <<EOF` tells the terminal: "everything I type until I see `EOF` again is the content"
> - `> filename.yaml` saves that content into a file
> - The final `EOF` signals the end of content
> - It's like opening a text editor and saving a file — all from the terminal

**Verify the file:**
```bash
cat volume-share-xfusion.yaml
```

---

### Step 2 — Apply the YAML to create the Pod

```bash
kubectl apply -f volume-share-xfusion.yaml
```

**Expected output:**
```
pod/volume-share-xfusion created
```

> `kubectl apply -f` reads your YAML file and tells Kubernetes to create the resource exactly as described.

---

### Step 3 — Verify the Pod is running

```bash
kubectl get pod volume-share-xfusion
```

**Expected output:**
```
NAME                   READY   STATUS    RESTARTS   AGE
volume-share-xfusion   2/2     Running   0          30s
```

> - `2/2` under READY = both containers are running
> - `Running` under STATUS = Pod is healthy
> - If you see `ContainerCreating`, wait a few seconds and re-run

---

### Step 4 — Create a file inside container 1

```bash
kubectl exec -it volume-share-xfusion -c volume-container-xfusion-1 -- bash -c "echo 'Welcome to xFusionCorp Industries' > /tmp/media/media.txt"
```

**Command breakdown:**

| Part | What it does |
|------|-------------|
| `kubectl exec` | Go inside a running container and execute a command |
| `-it` | Interactive terminal mode |
| `volume-share-xfusion` | Name of the Pod to access |
| `-c volume-container-xfusion-1` | Which container to go into (Pod has 2) |
| `--` | Separator — everything after this runs inside the container |
| `bash -c "..."` | Run a shell command inside the container |
| `echo '...' > /tmp/media/media.txt` | Write text into a file |

**Expected output:** (none — success is silent)

---

### Step 5 — Verify the file in container 1

```bash
kubectl exec -it volume-share-xfusion -c volume-container-xfusion-1 -- cat /tmp/media/media.txt
```

**Expected output:**
```
Welcome to xFusionCorp Industries
```

---

### Step 6 — Verify the file in container 2

```bash
kubectl exec -it volume-share-xfusion -c volume-container-xfusion-2 -- cat /tmp/demo/media.txt
```

**Expected output:**
```
Welcome to xFusionCorp Industries
```

> This confirms the shared `emptyDir` volume is working. The file written by container 1 is instantly visible to container 2.

---

## 🗺️ Visual Architecture

```
POD: volume-share-xfusion
│
├── Container 1 (volume-container-xfusion-1)
│       └── Sees volume at: /tmp/media
│                               │
│                    ┌──────────┴──────────┐
│                    │   emptyDir Volume   │
│                    │   (volume-share)    │
│                    │   media.txt ✅      │
│                    └──────────┬──────────┘
│                               │
├── Container 2 (volume-container-xfusion-2)
│       └── Sees volume at: /tmp/demo
```

Both containers point to the **same underlying storage** — just under different folder names.

---

## 🌍 Real World Use Cases

### 1. Log Processing Pipeline (Most Common — Sidecar Pattern)

```
Container 1 (App)     →   Container 2 (Log Collector)   →   Container 3 (Log Shipper)
Writes app logs           Reads & filters logs               Ships to Elasticsearch
to /tmp/logs              from /tmp/logs                     / Datadog / Splunk
```

### 2. Web Server + Content Generator + Cache Warmer

```
Container 1               Container 2                    Container 3
Generates HTML   →        Nginx serves files    →        Pre-loads files
to /tmp/input             from /tmp/process              into cache (/tmp/output)
```

### 3. Media Processing Pipeline

```
Container 1          Container 2               Container 3
Raw video upload  →  Transcodes to MP4/HLS  →  Generates thumbnails
/tmp/input           /tmp/process               /tmp/output
```

### 4. AI / ML Inference Pipeline

```
Container 1           Container 2            Container 3
Data Preprocessor  →  ML Model Inference  →  Result Formatter
Cleans raw data       Runs predictions        Formats to JSON/CSV
```

---

## ✅ Can You Share a Volume With 3+ Containers?

**Yes!** You can have as many containers as needed sharing the same volume. Just add more containers and mount the same volume at different paths:

```yaml
spec:
  containers:
    - name: container-1
      volumeMounts:
        - name: shared-volume
          mountPath: /tmp/input

    - name: container-2
      volumeMounts:
        - name: shared-volume
          mountPath: /tmp/process

    - name: container-3
      volumeMounts:
        - name: shared-volume
          mountPath: /tmp/output

  volumes:
    - name: shared-volume
      emptyDir: {}
```

---

## 📚 Key kubectl Commands Reference

| Command | Purpose |
|---------|---------|
| `kubectl apply -f file.yaml` | Create or update a resource from a YAML file |
| `kubectl get pod <name>` | Check the status of a Pod |
| `kubectl get pods` | List all Pods |
| `kubectl describe pod <name>` | Detailed info about a Pod (useful for debugging) |
| `kubectl exec -it <pod> -c <container> -- bash` | Open a shell inside a container |
| `kubectl exec -it <pod> -c <container> -- <cmd>` | Run a command inside a container |
| `kubectl delete pod <name>` | Delete a Pod |
| `kubectl logs <pod> -c <container>` | View logs of a container |

---

## ⚠️ Important Notes

- `emptyDir` volume data is **lost when the Pod is deleted** — use `PersistentVolume` for data that must survive Pod restarts
- The `sleep` command is used to keep the container running — in real deployments, your application process keeps the container alive
- The `-c` flag in `kubectl exec` is required when a Pod has more than one container
- Always wait for `READY: 2/2` and `STATUS: Running` before exec-ing into containers

---

*Study material generated for Nautilus DevOps Kubernetes Volume Sharing Task*
