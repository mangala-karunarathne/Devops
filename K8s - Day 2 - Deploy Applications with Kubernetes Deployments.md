# Kubernetes Study Material — Nautilus DevOps Task
> **Topic:** Kubernetes Deployments with MERN App Context  
> **Level:** Beginner-friendly (Ex-MERN Developer)  
> **Date:** March 17, 2026

---

## 📋 Table of Contents

1. [Original Task](#original-task)
2. [Step-by-Step Task Execution](#step-by-step-task-execution)
3. [Behind the Scenes — What Really Happened](#behind-the-scenes)
4. [The Evolution: Code → Docker → Kubernetes](#the-evolution)
5. [MERN App + Kubernetes — Full Picture](#mern-app--kubernetes)
6. [Clearing the Confusion — All Deployment Types](#clearing-the-confusion)
7. [Key Kubernetes Components Glossary](#key-kubernetes-components-glossary)
8. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Original Task

> **Task:** Create a deployment named `httpd` to deploy the application `httpd` using the image `httpd:latest` (ensure to specify the tag).
>
> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

---

## Step-by-Step Task Execution

### Step 1: Verify kubectl is configured and cluster is accessible

```bash
kubectl cluster-info
```

**Expected Output:**
```
Kubernetes control plane is running at https://<cluster-ip>:6443
CoreDNS is running at https://<cluster-ip>:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

---

### Step 2: Check existing deployments (baseline check)

```bash
kubectl get deployments
```

**Expected Output (if no deployments exist yet):**
```
No resources found in default namespace.
```

---

### Step 3: Create the `httpd` deployment ⭐

```bash
kubectl create deployment httpd --image=httpd:latest
```

**Expected Output:**
```
deployment.apps/httpd created
```

---

### Step 4: Verify the deployment was created

```bash
kubectl get deployments
```

**Expected Output:**
```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
httpd   1/1     1            1           30s
```

---

### Step 5: Verify the pod is running

```bash
kubectl get pods
```

**Expected Output:**
```
NAME                      READY   STATUS    RESTARTS   AGE
httpd-6f9d8c7b5d-xk9p2    1/1     Running   0          45s
```

> ⚠️ If STATUS shows `ContainerCreating`, wait a few seconds and re-run — the image is still being pulled from Docker Hub.

---

### Step 6: Inspect the deployment details (confirm image tag)

```bash
kubectl describe deployment httpd
```

**Expected Output (key section):**
```
Name:                   httpd
Namespace:              default
...
Pod Template:
  Containers:
   httpd:
    Image:      httpd:latest
...
Replicas:  1 desired | 1 updated | 1 total | 1 available | 0 unavailable
```

**✅ Actual output from the completed task:**
```
Name:               httpd
Namespace:          default
CreationTimestamp:  Tue, 17 Mar 2026 17:20:38 +0000
Labels:             app=httpd
Replicas:           1 desired | 1 updated | 1 total | 1 available | 0 unavailable
Image:              httpd:latest
Conditions:
  Available    True   MinimumReplicasAvailable
  Progressing  True   NewReplicaSetAvailable
NewReplicaSet:  httpd-6c755866c7 (1/1 replicas created)
Events:
  Normal  ScalingReplicaSet  76s  deployment-controller
         Scaled up replica set httpd-6c755866c7 from 0 to 1
```

### Task Completion Summary

| Check | Detail | Status |
|-------|--------|--------|
| Deployment Name | `httpd` | ✅ |
| Namespace | `default` | ✅ |
| Image | `httpd:latest` | ✅ |
| Replicas | 1 desired, 1 available | ✅ |
| Available Condition | `True` | ✅ |
| Progressing Condition | `True` | ✅ |
| ReplicaSet | `httpd-6c755866c7` (1/1) | ✅ |

---

## Behind the Scenes

### What happens at each step

#### Step 1: `kubectl cluster-info`

```
You (kubectl) ──asks──► API Server: "Hey, are you alive?"
API Server ──replies──► "Yes! I'm running at https://<ip>:6443"
```

`kubectl` reads your `~/.kube/config` file to find the cluster address and credentials, then sends an HTTP request to the Kubernetes API Server to confirm it's reachable.

---

#### Step 2: `kubectl get deployments` (baseline)

```
kubectl ──asks──► API Server: "List all deployments"
API Server ──checks──► etcd (the database): "Any deployments stored?"
etcd ──replies──► "Nothing found"
```

Kubernetes stores ALL its state in a database called **etcd**. The API Server queries it and returns the result.

---

#### Step 3: `kubectl create deployment httpd --image=httpd:latest` ⭐

This triggers a **chain reaction** across 4 components:

```
┌─────────┐      ┌────────────┐      ┌───────┐
│ kubectl │─────►│ API Server │─────►│ etcd  │
└─────────┘      └────────────┘      └───────┘
  "Create           "Store this        "Saved!"
  httpd deploy"     deployment"

                        │
                        ▼
               ┌──────────────────┐
               │ Controller       │
               │ Manager          │
               │ "New deployment! │
               │  Creating a      │
               │  ReplicaSet"     │
               └──────────────────┘
                        │
                        ▼
               ┌──────────────────┐
               │ Scheduler        │
               │ "New Pod needs   │
               │  a home. Finding │
               │  a suitable Node"│
               └──────────────────┘
                        │
                        ▼
               ┌──────────────────┐
               │ kubelet          │
               │ (on the Node)    │
               │ "Pulling         │
               │  httpd:latest    │
               │  from Docker Hub │
               │  and starting it"│
               └──────────────────┘
```

| Component | What it did |
|-----------|-------------|
| **API Server** | Received your command, saved desired state: "I want 1 httpd pod" |
| **etcd** | Stored the desired state as a record in the database |
| **Controller Manager** | Noticed the new deployment → created ReplicaSet `httpd-6c755866c7` |
| **Scheduler** | Found a suitable Node and assigned the Pod to it |
| **kubelet** | Pulled `httpd:latest` from Docker Hub and started the container |

---

#### The 3-Layer Hierarchy

When you run `kubectl create deployment`, K8s creates this structure automatically:

```
Deployment  (httpd)
    └──► ReplicaSet  (httpd-6c755866c7)
              └──► Pod  (httpd-6c755866c7-xxxxx)
                      └──► Container (httpd:latest)
```

| Layer | Purpose |
|-------|---------|
| **Deployment** | Manages rollouts, rollbacks, updates |
| **ReplicaSet** | Ensures X number of Pods always running |
| **Pod** | Smallest deployable unit — wraps the container |
| **Container** | Your actual app running inside |

---

## The Evolution

### Chapter 1: The Old Way (Just a Server)

```
Your MERN App Code
        │
        ▼
   Ubuntu Server
   (manually install Node, MongoDB, Nginx...)
        │
        ▼
   App is Live 🎉
```

**The Problems:**
- "Works on my machine, breaks on server" 😩
- "I forgot to install Node 18, server has Node 14"
- Scaling = buying another physical server 💸
- Manual updates = downtime

---

### Chapter 2: Docker Enters 🐳

Docker solved the **"works on my machine"** problem.

**Your MERN app has 3 parts:**

```
Your MERN App
├── frontend/     (React)
├── backend/      (Express + Node)
└── database/     (MongoDB)
```

You write a `Dockerfile` for each:

```dockerfile
# Backend Dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

Each part runs in a **container** — a box that has EVERYTHING it needs inside it.

```
┌──────────────────────────────────┐
│        Docker Container          │
│  ┌──────────────────────────┐   │
│  │   Your Express App       │   │
│  │   Node 18                │   │
│  │   All npm packages       │   │
│  └──────────────────────────┘   │
└──────────────────────────────────┘
```

**With `docker-compose` your full MERN stack:**

```yaml
# docker-compose.yml
services:
  frontend:    # React container
  backend:     # Express container
  mongodb:     # MongoDB container
```

```bash
docker-compose up   # Entire MERN stack starts!
```

**The Problems with just Docker:**
- Crashed container at 3AM? Nobody restarts it 😴
- 10,000 users spike? Can't auto-scale
- Server dies? Your app dies too 💀
- Managing 10 servers manually = nightmare

---

### Chapter 3: Kubernetes Enters ☸️

> **Simple Definition:** Kubernetes is a system that manages your Docker containers automatically — starts them, restarts them, scales them, and keeps them healthy across multiple servers.

```
Without K8s:                    With K8s:
You manually restart        VS  K8s auto-restarts crashed ✅
crashed containers              containers

You manually scale          VS  K8s auto-scales based    ✅
containers                      on traffic

You manage servers          VS  K8s spreads containers   ✅
manually                        across servers
```

---

## MERN App + Kubernetes

### Full Picture

```
👨‍💻 YOU WRITE CODE
        │
        ▼
┌───────────────────────────────────────┐
│           YOUR MERN APP               │
│  ┌──────────┐ ┌──────────┐ ┌───────┐ │
│  │  React   │ │ Express  │ │ Mongo │ │
│  │ Frontend │ │ Backend  │ │  DB   │ │
│  └──────────┘ └──────────┘ └───────┘ │
└───────────────────────────────────────┘
        │
        │  Step 1: Dockerize each part
        ▼
┌───────────────────────────────────────┐
│         DOCKER IMAGES                 │
│  myapp/frontend:latest                │
│  myapp/backend:latest                 │
│  mongo:latest                         │
└───────────────────────────────────────┘
        │
        │  Step 2: Push to Docker Registry
        │  (Docker Hub / AWS ECR / GCR)
        ▼
┌───────────────────────────────────────────────────────┐
│                  KUBERNETES CLUSTER                    │
│                                                        │
│  ┌─────────────────────────────────────────────────┐  │
│  │               K8s Deployments                   │  │
│  │                                                 │  │
│  │  ┌───────────┐ ┌───────────┐  ┌───────────┐   │  │
│  │  │  React    │ │  Express  │  │  MongoDB  │   │  │
│  │  │   Pod     │ │   Pod     │  │   Pod     │   │  │
│  │  └───────────┘ └───────────┘  └───────────┘   │  │
│  │                                                 │  │
│  │  🔄 Auto-restart if crashed                    │  │
│  │  📈 Auto-scale if traffic spikes               │  │
│  │  ⚖️  Load balance between pods                 │  │
│  └─────────────────────────────────────────────────┘  │
│                                                        │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐        │
│  │ Server 1 │    │ Server 2 │    │ Server 3 │        │
│  │  (Node)  │    │  (Node)  │    │  (Node)  │        │
│  └──────────┘    └──────────┘    └──────────┘        │
└───────────────────────────────────────────────────────┘
```

### Real-World Scenario

Imagine your MERN app goes viral:

```
Normal day:         100 users   → 1 Express Pod is enough
Traffic spike:    10,000 users  → K8s auto-scales to 10 Express Pods
Spike ends:         100 users   → K8s scales back down to 1 Pod
Express crash 3AM:              → K8s restarts it in seconds 😴
```

---

## Clearing the Confusion

### All "Deployment" Types Explained

| Term | What it is | MERN Analogy |
|------|-----------|--------------|
| **Docker Image** | Blueprint/snapshot of your app | Like `package.json` — defines what's needed |
| **Docker Container** | A running instance of that image | Like running `node server.js` — it's alive |
| **Docker Compose Deployment** | Multi-container app on **one machine** | Running full MERN stack locally |
| **K8s Deployment** | Managing containers on **many machines** | MERN in production across servers |
| **K8s Pod** | Wrapper around one or more containers | One running Express container instance |
| **K8s ReplicaSet** | Ensures X Pods always run | "Always keep 3 Express servers running" |

### One-Line Summary

```
Docker         = Packages your MERN app into containers       📦
Docker Compose = Runs all MERN containers on 1 machine        🖥️
Kubernetes     = Runs & manages MERN containers across
                 many machines, automatically                  🌐
```

> **K8s doesn't replace Docker — it uses Docker containers and manages them at scale.**

---

## Key Kubernetes Components Glossary

| Component | Role | Restaurant Analogy |
|-----------|------|--------------------|
| **kubectl** | CLI tool to talk to K8s | Customer placing order |
| **API Server** | Central gateway for all K8s operations | Waiter taking your order |
| **etcd** | Database storing cluster state | Order book / notepad |
| **Controller Manager** | Ensures desired state = actual state | Kitchen manager |
| **Scheduler** | Assigns Pods to Nodes | Deciding which chef handles it |
| **kubelet** | Agent on each Node, runs containers | The chef cooking the food |
| **Pod** | Smallest deployable unit in K8s | The final dish served |
| **Node** | A worker machine (VM or physical) | The kitchen station |
| **Cluster** | Collection of Nodes managed by K8s | The entire restaurant |
| **Deployment** | Manages Pod rollout and lifecycle | Catering order manager |
| **ReplicaSet** | Maintains a stable set of Pod replicas | Ensures 3 dishes always ready |
| **Namespace** | Virtual cluster / logical separation | Different restaurant sections |

---

## Quick Reference Cheat Sheet

### Essential kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Deployments
kubectl get deployments
kubectl create deployment <name> --image=<image>:<tag>
kubectl describe deployment <name>
kubectl delete deployment <name>

# Pods
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>

# Output as YAML (useful for learning)
kubectl get deployment <name> -o yaml
kubectl get pod <pod-name> -o yaml
```

### This Task's Key Command

```bash
kubectl create deployment httpd --image=httpd:latest
```

| Part | Meaning |
|------|---------|
| `kubectl` | The CLI tool |
| `create deployment` | Action — create a new deployment |
| `httpd` | Name of the deployment |
| `--image=httpd:latest` | Docker image to use (with explicit tag) |

### Why Specify the Tag `:latest`?

Always specify the tag in production:

```bash
# ✅ Good — explicit tag
kubectl create deployment httpd --image=httpd:latest
kubectl create deployment myapp --image=myapp:v1.2.3

# ⚠️ Risky — no tag defaults to :latest, but not explicit
kubectl create deployment httpd --image=httpd
```

In real production environments, use specific version tags like `httpd:2.4.58` instead of `:latest` to avoid unexpected updates.

---

## Summary Flow Diagram

```
You type:
kubectl create deployment httpd --image=httpd:latest
        │
        ▼
    API Server  ──saves to──►  etcd
        │
        ▼
  Controller Manager
  creates ReplicaSet
        │
        ▼
    Scheduler
  assigns Pod to Node
        │
        ▼
    kubelet (on Node)
  pulls httpd:latest from Docker Hub
  starts the container
        │
        ▼
  Pod is Running ✅
  httpd web server is live!
```

---

*Study material compiled from Nautilus DevOps K8s task — March 17, 2026*
