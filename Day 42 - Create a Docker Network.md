# Docker Networks – Study Material
### Nautilus DevOps Task: Creating a Docker Network

---

## 📋 Original Task Description

> **Ticket:** Create Docker networks to be used later by application containers.
>
> **Requirements:**
> - **a.** Create a docker network named `news` on **App Server 3** in **Stratos DC**
> - **b.** Configure it to use `bridge` drivers
> - **c.** Set it to use subnet `172.168.0.0/24` and iprange `172.168.0.0/24`

---

## 🖥️ Environment Details

| Detail         | Value         |
|----------------|---------------|
| Server         | App Server 3  |
| Hostname       | `stapp03`     |
| User           | `banner`      |
| Password       | `BigGr33n`    |
| Location       | Stratos DC    |

---

## ✅ Solution: Step-by-Step Commands

### Step 1 – SSH into App Server 3
```bash
ssh banner@stapp03
```

### Step 2 – Switch to root
```bash
sudo su -
# Enter password: BigGr33n
```

### Step 3 – Verify Docker is running
```bash
docker --version
systemctl status docker
```

### Step 4 – Create the Docker network
```bash
docker network create \
  --driver bridge \
  --subnet 172.168.0.0/24 \
  --ip-range 172.168.0.0/24 \
  news
```

### Step 5 – Verify the network
```bash
docker network ls | grep news
docker network inspect news
```

### ✅ Expected Output from `docker network inspect news`
```json
{
    "Name": "news",
    "Driver": "bridge",
    "IPAM": {
        "Config": [
            {
                "Subnet": "172.168.0.0/24",
                "IPRange": "172.168.0.0/24"
            }
        ]
    }
}
```

---

## 📚 Concepts Explained

### 1. What is a Docker Network?

A Docker network is a virtual networking layer that allows containers to:
- Communicate with each other
- Communicate with the outside world
- Stay isolated from other networks

Just like physical machines need a network switch to communicate, Docker containers need a virtual network.

---

### 2. Docker Network Drivers

| Driver    | Use Case                                               |
|-----------|--------------------------------------------------------|
| `bridge`  | Default. Isolated network on a single host             |
| `host`    | Container shares the host's network stack              |
| `overlay` | Multi-host communication (used in Docker Swarm)        |
| `macvlan` | Container gets its own MAC address on the physical network |
| `none`    | No networking — fully isolated container               |

> **In this task:** `bridge` was used — the most common driver for single-host container communication.

---

### 3. Bridge Network – How It Works

```
Host Machine (stapp03)
┌────────────────────────────────────┐
│        Bridge Network "news"       │
│                                    │
│  ┌─────────────┐  ┌─────────────┐  │
│  │ Container A │  │ Container B │  │
│  │172.168.0.2  │  │172.168.0.3  │  │
│  └─────────────┘  └─────────────┘  │
│                                    │
│          Virtual Bridge            │
│         (docker0 or custom)        │
└────────────────────────────────────┘
         │
    Physical NIC
         │
    External Network
```

- Containers on the **same bridge** network can talk to each other
- Containers on **different** bridge networks are isolated
- The bridge connects to the host's physical NIC for external access

---

### 4. Subnet vs IP Range

| Parameter    | Value             | Meaning                                     |
|--------------|-------------------|---------------------------------------------|
| `--subnet`   | `172.168.0.0/24`  | Total pool of IPs (172.168.0.0 – 172.168.0.255) |
| `--ip-range` | `172.168.0.0/24`  | IPs Docker actually assigns to containers   |

**CIDR `/24` means:**
- 256 total IP addresses
- Usable range: `172.168.0.1` – `172.168.0.254`
- Network address: `172.168.0.0`
- Broadcast address: `172.168.0.255`

> **Note:** When `subnet` and `ip-range` are the same, Docker can use the full range for containers.

---

### 5. Why Pre-Create Networks?

```
DevOps Ticket Created
        ↓
Network "news" created on stapp03   ← (This Task)
        ↓
Dev team later deploys containers
        ↓
Containers attach to "news" network
   docker run --network news my_app
        ↓
Containers get IPs from 172.168.0.0/24
        ↓
Containers communicate with each other
```

**Benefits of pre-creating networks:**

| Benefit       | Description                                              |
|---------------|----------------------------------------------------------|
| **Isolation** | Apps on different networks cannot interfere              |
| **Planning**  | IP ranges reserved in advance to avoid conflicts         |
| **Reusability** | Multiple containers can join the same network          |
| **Security**  | Only containers on `news` network can talk within it     |
| **Control**   | Predictable IP addressing for services                   |

---

## 🔧 Useful Docker Network Commands

```bash
# List all networks
docker network ls

# Create a network
docker network create --driver bridge --subnet X.X.X.X/X --ip-range X.X.X.X/X <name>

# Inspect a network (detailed info)
docker network inspect <name>

# Connect a running container to a network
docker network connect <network> <container>

# Disconnect a container from a network
docker network disconnect <network> <container>

# Remove a network
docker network rm <name>

# Remove all unused networks
docker network prune
```

---

## 📝 Key Takeaways

1. **Docker networks** provide isolation and communication between containers
2. **Bridge driver** is the default and most commonly used driver for single-host setups
3. **Subnet** defines the overall IP pool; **IP range** controls what Docker assigns
4. **CIDR /24** gives 256 addresses in the range
5. Pre-creating networks is a DevOps best practice for **infrastructure-as-code** workflows
6. Use `docker network inspect` to always **verify** your configuration

---

## 🏷️ Quick Reference Card

```
docker network create \
  --driver bridge \          # Network type
  --subnet 172.168.0.0/24 \  # Total IP pool
  --ip-range 172.168.0.0/24 \# Assignable IPs
  news                        # Network name
```

---

*Study material generated for Nautilus DevOps Team – Stratos DC*
