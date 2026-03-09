# Nautilus Project – Docker Image Pull & Re-tag

## Task Description
Nautilus project developers are planning to start testing on a new project. As per their meeting with the DevOps team, they want to test containerized environment application features.

**Objective:**
- Pull `busybox:musl` image on **App Server 2** in Stratos DC
- Re-tag (create new tag) this image as `busybox:media`

---

## Environment Details

| Detail        | Value       |
|---------------|-------------|
| Server        | App Server 2 |
| Hostname      | `stapp02`   |
| User          | `steve`     |
| Password      | `Am3ric@`   |

---

## Solution

### Step 1: SSH into App Server 2
```bash
ssh steve@stapp02
# Password: Am3ric@
```

### Step 2: Pull the `busybox:musl` image
```bash
sudo docker pull busybox:musl
```

### Step 3: Re-tag the image as `busybox:media`
```bash
sudo docker tag busybox:musl busybox:media
```

### Step 4: Verify both tags exist
```bash
sudo docker images | grep busybox
```

---

## Expected Output

```
REPOSITORY   TAG     IMAGE ID       CREATED        SIZE
busybox      media   0188a8de47ca   <date>         1.51MB
busybox      musl    0188a8de47ca   <date>         1.51MB
```

> ✅ Both tags share the **same Image ID** — confirming the re-tag was successful.

---

## Key Concepts

- **`docker pull`** — Downloads the specified image from Docker Hub.
- **`docker tag`** — Creates a new tag/alias for an existing image. It does **not** duplicate the image; both tags share the same underlying image layers.
- **`sudo`** — Required since the user `steve` needs elevated privileges to run Docker commands.

---

## Summary

| Command | Purpose |
|--------|---------|
| `docker pull busybox:musl` | Pull the image from Docker Hub |
| `docker tag busybox:musl busybox:media` | Create a new tag pointing to the same image |
| `docker images \| grep busybox` | Verify both tags exist with the same Image ID |
