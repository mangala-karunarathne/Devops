# Docker Study Material — Creating an Image from a Container

---

## 🧪 The Task (Nautilus Lab)

> **Create an image `games:devops` on Application Server 1 from a running container called `ubuntu_latest`.**

---

## 🛠️ Commands Used

```bash
# Step 1 — SSH into Application Server 1
ssh tony@stapp01
# Password: Ir0nM@n

# Step 2 — Verify the running container
docker ps

# Step 3 — Commit the container as a new image
docker commit ubuntu_latest games:devops

# Step 4 — Verify the image was created
docker images | grep games
```

---

## 📖 Concept Breakdown

### 1. `docker ps`
Lists all **currently running containers**.

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    NAMES
a1b2c3d4e5f6   ubuntu    "bash"    1 hr ago  Up 1 hr   ubuntu_latest
```

---

### 2. `docker commit`
Creates a **new image** from the current state of a running (or stopped) container.
Think of it as taking a **snapshot** — all changes the developer made inside the container are preserved.

**Syntax:**
```bash
docker commit <container_name_or_id> <image_name>:<tag>
```

**Our example:**
```bash
docker commit ubuntu_latest games:devops
#             ^container      ^image  ^tag
```

---

### 3. `docker images`
Lists all images available locally on the machine.

```
REPOSITORY   TAG      IMAGE ID       CREATED          SIZE
games        devops   ghi789abc123   5 seconds ago    77 MB
ubuntu       latest   abc123def456   2 weeks ago      77 MB
```

---

### 4. The Pipe `|` and `grep`

```bash
docker images | grep games
```

| Part | Role |
|---|---|
| `docker images` | Produces the full list of images |
| `\|` | **Pipe** — sends output of the left command as input to the right command |
| `grep games` | Filters lines, showing only those containing "games" |

**Flow:**
```
docker images  →→→(pipe)→→→  grep games
  (full list)                 (filtered result)
```

**Without pipe** — you see ALL images:
```
REPOSITORY   TAG       IMAGE ID
ubuntu       latest    abc123
nginx        1.25      def456
games        devops    ghi789
```

**With pipe** — only the matching line:
```
games        devops    ghi789
```

> 💡 `grep` is for **searching/filtering text**, but it needs data to search through — the pipe `|` is what supplies it.

---

## 🗺️ Full Picture — How It All Connects

```
Developer makes changes
  inside ubuntu_latest container
          ↓
   docker commit
          ↓
  New image: games:devops
  (snapshot of container's state)
          ↓
   docker images | grep games
          ↓
  Confirms image exists locally
```

---

## 📚 Key Terms

| Term | Definition |
|---|---|
| **Container** | A running instance of an image (like a running program) |
| **Image** | A read-only template used to create containers (like a blueprint) |
| **Tag** | A label for an image version (e.g., `devops`, `latest`, `v1.0`) |
| **`docker commit`** | Saves a container's current state as a new image |
| **Pipe `\|`** | Passes output of one command as input to another |
| **`grep`** | Searches/filters text for a matching pattern |

---

## ✅ Quick Reference

```bash
docker ps                                  # List running containers
docker images                              # List all local images
docker commit <container> <image>:<tag>    # Create image from container
docker images | grep <name>                # Filter images by name
```
