# Kubernetes Init Containers — Study Material
> Topic: Init Containers, Shared Volumes, and Deployment Configuration  
> Based on: xFusionCorp Industries Lab Task

---

## Table of Contents
1. [Original Task](#original-task)
2. [What is an Init Container?](#what-is-an-init-container)
3. [How Init Containers Work](#how-init-containers-work)
4. [Key Kubernetes Concepts Used](#key-kubernetes-concepts-used)
5. [Step-by-Step Task Execution](#step-by-step-task-execution)
6. [Full YAML Manifest](#full-yaml-manifest)
7. [Verification Commands](#verification-commands)
8. [Real World Use Cases](#real-world-use-cases)
9. [DB Availability Check with curl/nc](#db-availability-check-with-curlnc)
10. [How Kubernetes DNS and Services Work](#how-kubernetes-dns-and-services-work)
11. [Passing Config via Shared Volume](#passing-config-via-shared-volume)
12. [Quick Reference — Ports and Tools](#quick-reference--ports-and-tools)

---

## Original Task

> There are some applications that need to be deployed on Kubernetes cluster and these apps have some pre-requisites where some configurations need to be changed before deploying the app container. Some of these changes cannot be made inside the images so the DevOps team has come up with a solution to use init containers to perform these tasks during deployment. Below is a sample scenario that the team is going to test first.

**Requirements:**

1. Create a `Deployment` named as `ic-deploy-xfusion`.
2. Configure `spec` as replicas should be `1`, labels `app` should be `ic-xfusion`, template's metadata labels `app` should be the same `ic-xfusion`.
3. The `initContainers` should be named as `ic-msg-xfusion`, use image `fedora` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'echo Init Done - Welcome to xFusionCorp Industries > /ic/ecommerce'`. The volume mount should be named as `ic-volume-xfusion` and mount path should be `/ic`.
4. Main container should be named as `ic-main-xfusion`, use image `fedora` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'while true; do cat /ic/ecommerce; sleep 5; done'`. The volume mount should be named as `ic-volume-xfusion` and mount path should be `/ic`.
5. Volume to be named as `ic-volume-xfusion` and it should be an `emptyDir` type.

> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## What is an Init Container?

An **Init Container** is a special container that runs **before** the main application container starts inside a Pod. It is used to perform setup or pre-configuration tasks that the main container depends on.

### Key Characteristics

| Property | Init Container | Main Container |
|----------|---------------|----------------|
| Runs first? | ✅ Always | ❌ Only after init completes |
| Must complete before main starts? | ✅ Yes | N/A |
| If init fails, does main start? | ❌ Never | N/A |
| Can share volumes with main? | ✅ Yes | ✅ Yes |
| Runs continuously? | ❌ Runs once and exits | ✅ Runs continuously |

---

## How Init Containers Work

```
Pod Starts
    │
    ▼
Init Container runs
    │  (performs pre-setup tasks)
    │  (writes config/files to shared volume)
    ▼
Init Container completes (Exit 0)
    │
    ▼
Main Container starts
    │  (reads from shared volume)
    │  (starts the actual application)
    ▼
Main Container runs continuously
```

### What Happens if Init Fails?

```
Init Container fails (non-zero exit)
    │
    ▼
Kubernetes restarts the Init Container
    │
    ▼
Main Container NEVER starts
    │
    ▼
Pod stays in "Init:Error" or "Init:CrashLoopBackOff" state
```

---

## Key Kubernetes Concepts Used

### 1. Deployment
A Kubernetes object that manages a set of identical Pods. It ensures the desired number of replicas are always running.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
```

### 2. Replicas
The number of Pod copies you want running at any time.
```yaml
spec:
  replicas: 1
```

### 3. Labels and Selectors
Labels are key-value pairs attached to objects. The Deployment uses `selector` to find and manage Pods with matching labels.
```yaml
selector:
  matchLabels:
    app: ic-xfusion
template:
  metadata:
    labels:
      app: ic-xfusion
```

### 4. emptyDir Volume
A temporary storage volume that is created when a Pod starts and deleted when the Pod stops. Both init and main containers can mount the same `emptyDir` volume to share data.

```yaml
volumes:
  - name: ic-volume-xfusion
    emptyDir: {}
```

### 5. volumeMounts
Tells a container where to mount the shared volume inside its filesystem.
```yaml
volumeMounts:
  - name: ic-volume-xfusion
    mountPath: /ic
```

---

## Step-by-Step Task Execution

### Step 1 — Verify Cluster Access
```bash
kubectl get nodes
```
**Expected Output:**
```
NAME           STATUS   ROLES           AGE   VERSION
master-node    Ready    control-plane   10d   v1.28.0
worker-node1   Ready    <none>          10d   v1.28.0
```

---

### Step 2 — Create the YAML Manifest
```bash
cat <<EOF > ic-deploy-xfusion.yaml
# (paste the full YAML here — see Full YAML Manifest section)
EOF
```

---

### Step 3 — Apply the Deployment
```bash
kubectl apply -f ic-deploy-xfusion.yaml
```
**Expected Output:**
```
deployment.apps/ic-deploy-xfusion created
```

---

### Step 4 — Watch Pod Initialization
```bash
kubectl get pods -l app=ic-xfusion -w
```
**Expected Output (progression):**
```
NAME                                 READY   STATUS            RESTARTS   AGE
ic-deploy-xfusion-6d9f7b8c4-xk2pq   0/1     Init:0/1          0          5s
ic-deploy-xfusion-6d9f7b8c4-xk2pq   0/1     PodInitializing   0          15s
ic-deploy-xfusion-6d9f7b8c4-xk2pq   1/1     Running           0          20s
```
> Press `Ctrl+C` once the pod reaches `Running` state.  
> `Ctrl+C` only stops the watch command — it does NOT stop the pod.

---

### Step 5 — Final Verification
```bash
# Check deployment is healthy
kubectl get deployment ic-deploy-xfusion

# Check main container logs
kubectl logs -l app=ic-xfusion -c ic-main-xfusion
```

**Expected Output:**
```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-xfusion   1/1     1            1           7m45s

Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
```

---

## Full YAML Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  template:
    metadata:
      labels:
        app: ic-xfusion
    spec:
      initContainers:
        - name: ic-msg-xfusion
          image: fedora:latest
          command:
            - '/bin/bash'
            - '-c'
            - 'echo Init Done - Welcome to xFusionCorp Industries > /ic/ecommerce'
          volumeMounts:
            - name: ic-volume-xfusion
              mountPath: /ic
      containers:
        - name: ic-main-xfusion
          image: fedora:latest
          command:
            - '/bin/bash'
            - '-c'
            - 'while true; do cat /ic/ecommerce; sleep 5; done'
          volumeMounts:
            - name: ic-volume-xfusion
              mountPath: /ic
      volumes:
        - name: ic-volume-xfusion
          emptyDir: {}
```

---

## Verification Commands

| Purpose | Command |
|---------|---------|
| Check all pods | `kubectl get pods` |
| Check specific pods by label | `kubectl get pods -l app=ic-xfusion` |
| Watch pod status live | `kubectl get pods -l app=ic-xfusion -w` |
| Check deployment | `kubectl get deployment ic-deploy-xfusion` |
| Describe pod (full detail) | `kubectl describe pod -l app=ic-xfusion` |
| Init container logs | `kubectl logs -l app=ic-xfusion -c ic-msg-xfusion` |
| Main container logs | `kubectl logs -l app=ic-xfusion -c ic-main-xfusion` |
| Delete deployment | `kubectl delete deployment ic-deploy-xfusion` |

---

## Real World Use Cases

### 1. Waiting for Database to be Ready
The most common use case. Init container checks if the DB is up before the app starts — preventing crash loops.

```
Without Init Container:
App starts → DB not ready → App crashes → CrashLoopBackOff ❌

With Init Container:
Init waits → DB becomes ready → Init exits → App starts safely ✅
```

### 2. Database Migrations
Run schema migrations before the app starts, ensuring the DB structure is up to date.

### 3. Fetching Secrets / Configurations
Pull secrets from a vault (e.g., HashiCorp Vault) and write them to a shared volume for the main app to read.

### 4. Downloading Assets / ML Models
Download large files (models, static assets) into a shared volume so the main container doesn't need to handle it.

### 5. Kernel / System-level Configuration
Change OS-level settings (like `sysctl` parameters needed by Elasticsearch) that can't be done inside a regular container image.

---

## DB Availability Check with curl/nc

### Using netcat (nc) — Check if TCP Port is Open
```yaml
initContainers:
  - name: wait-for-db
    image: busybox
    command:
      - '/bin/sh'
      - '-c'
      - |
        until nc -z db-service 5432; do
          echo "Waiting for database...";
          sleep 2;
        done;
        echo "Database is up!"
```

**Command Breakdown:**
```
nc        = netcat (network tool)
-z        = just check if port is open, don't send data
db-service = Kubernetes Service NAME of the database
5432      = port number (PostgreSQL=5432, MySQL=3306, Redis=6379)
```

---

### Using curl — Check HTTP Endpoint
```yaml
initContainers:
  - name: wait-for-api
    image: curlimages/curl
    command:
      - '/bin/sh'
      - '-c'
      - |
        until curl -sf http://api-service:8080/health; do
          echo "API not ready, retrying in 3s...";
          sleep 3;
        done;
        echo "API is ready!"
```

**Command Breakdown:**
```
curl      = tool to make HTTP requests
-s        = silent mode (no progress bar)
-f        = fail on HTTP error (returns non-zero exit code)
```

---

### The `until` Loop — How it Works

```bash
until nc -z db-service 5432; do
  echo "Waiting for database...";
  sleep 2;
done
```

Read as plain English:
```
UNTIL (DB port is open and responding)
  DO
    Print "Waiting..."
    Wait 2 seconds
    Try again
DONE → exit loop → Init container exits → Main app starts
```

Visual flow:
```
Attempt 1 → DB Down → wait 2s → retry
Attempt 2 → DB Down → wait 2s → retry
Attempt 3 → DB Down → wait 2s → retry
Attempt 4 → DB UP ✅ → exit loop → Main app starts
```

---

## How Kubernetes DNS and Services Work

Init containers **never call DB pods directly by IP**. They always use the **Service name**, which Kubernetes DNS resolves automatically.

```
Init Container
     │
     │  nc -z db-service 5432
     ▼
Kubernetes DNS
     │  resolves "db-service" → Pod IP (e.g., 10.96.0.5)
     ▼
Database Pod (running PostgreSQL/MySQL)
     │
     └── Is port open? YES → Init exits / NO → retry
```

### Why Use Service Name, Not Pod IP?

| | Pod IP | Service Name |
|---|--------|-------------|
| Changes on pod restart? | ✅ YES — always changes | ❌ NO — always stable |
| Safe to hardcode? | ❌ Never | ✅ Always |
| How to use | Don't | `db-service`, `mysql-service` etc. |

> **Rule:** Always refer to other services by their **Kubernetes Service name**, never by IP address.

---

## Passing Config via Shared Volume

This combines the DB availability check with the shared volume pattern — the init container writes connection details to the shared volume, and the main app reads them.

```yaml
initContainers:
  - name: wait-for-db
    image: busybox
    command:
      - '/bin/sh'
      - '-c'
      - |
        until nc -z db-service 5432; do
          echo "Waiting for DB...";
          sleep 2;
        done;
        echo "DB_HOST=db-service" > /config/db.env
        echo "DB_PORT=5432"      >> /config/db.env
        echo "Config written!"
    volumeMounts:
      - name: config-volume
        mountPath: /config

containers:
  - name: main-app
    image: my-app:latest
    command:
      - '/bin/sh'
      - '-c'
      - |
        source /config/db.env
        echo "Connecting to $DB_HOST:$DB_PORT"
        ./start-app.sh
    volumeMounts:
      - name: config-volume
        mountPath: /config

volumes:
  - name: config-volume
    emptyDir: {}
```

### Flow:
```
Init Container
  └── Waits until DB is ready
  └── Writes DB_HOST and DB_PORT to /config/db.env (shared volume)
  └── Exits successfully
        │
        ▼
Main Container
  └── Reads /config/db.env from shared volume
  └── Connects to DB using those values
  └── Starts the application
```

---

## Quick Reference — Ports and Tools

### Common Database Ports
| Database | Default Port |
|----------|-------------|
| PostgreSQL | 5432 |
| MySQL / MariaDB | 3306 |
| MongoDB | 27017 |
| Redis | 6379 |
| Elasticsearch | 9200 |
| Cassandra | 9042 |

### DB-Specific Readiness Check Tools
| Tool | Database | Command |
|------|----------|---------|
| `pg_isready` | PostgreSQL | `pg_isready -h db-service -p 5432` |
| `mysqladmin ping` | MySQL | `mysqladmin ping -h db-service` |
| `nc -z` | Any TCP | `nc -z db-service 5432` |
| `curl` | HTTP APIs | `curl -sf http://service:port/health` |

> **Tip:** `nc -z` and `curl` only verify the **port is open**. DB-specific tools like `pg_isready` verify the DB is actually ready to serve queries — preferred for production.

---

## Summary

```
┌─────────────────────────────────────────────────────┐
│                   THE BIG PICTURE                   │
│                                                     │
│  Problem:  App needs DB but DB might not be ready   │
│                                                     │
│  Solution: Init Container acts as a gatekeeper      │
│                                                     │
│  Init Container                                     │
│    ├── Checks DB is ready (nc/curl/pg_isready)      │
│    ├── Writes config to shared volume (emptyDir)    │
│    └── Exits → signals "all clear"                  │
│                                                     │
│  Main Container                                     │
│    ├── Starts ONLY after init exits successfully    │
│    ├── Reads config from shared volume              │
│    └── Connects to DB and runs the application      │
│                                                     │
│  Kubernetes Service                                 │
│    └── Provides stable DNS name for DB pod          │
│        so init container always finds it            │
└─────────────────────────────────────────────────────┘
```

> **Core Idea:** Init containers ensure prerequisites are ready first, and pass needed information through a shared volume to the main application — so the app only starts when everything it depends on is confirmed ready.
