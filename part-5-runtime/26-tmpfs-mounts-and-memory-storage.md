
# Chapter 26 — tmpfs Mounts & In-Memory Storage ⚡🧠

So far, you’ve learned how Docker handles **persistent storage** using volumes and bind mounts.
Now we explore the **opposite end of the spectrum**:

> **What if you want data that should NEVER touch disk?**

Enter **tmpfs mounts** — storage that lives **only in memory (RAM)**.

This chapter explains:
- What tmpfs really is (Linux-level)
- Why in-memory storage exists
- How Docker exposes tmpfs
- When tmpfs is the *right* choice
- When it is **dangerous**
- How it fits the container lifecycle

---

## First: What Is tmpfs? 🧠

**tmpfs** stands for:

> **Temporary File System**

At the Linux kernel level, tmpfs is:
- A filesystem backed by **RAM**
- Optionally backed by **swap**
- Fully **ephemeral**
- Automatically cleaned up when unmounted

📌 **Nothing written to tmpfs is stored on disk.**

---

## Why tmpfs Exists (The Real Reason) 🔍

tmpfs exists because sometimes:
- Disk is too slow
- Disk persistence is unsafe
- Data must disappear immediately
- Security matters more than durability

Examples:
- Secrets
- Temporary caches
- Runtime scratch space
- Sensitive intermediate data

---

## tmpfs vs Disk-Based Storage 🆚

| Property | Volume / Bind Mount | tmpfs |
|-------|--------------------|------|
| Backed by disk | ✅ Yes | ❌ No |
| Persists restart | ✅ Yes | ❌ No |
| Fast access | ⚠️ | ✅ Very fast |
| Secure (no disk trace) | ❌ | ✅ |
| Memory usage | Low | Uses RAM |

📌 tmpfs trades **durability for speed and safety**.

---

## tmpfs in Containers (Big Picture) 🧩

In Docker:
- tmpfs is mounted **inside the container**
- It exists only for that container’s lifetime
- When the container stops → data is gone

This fits perfectly with:
> **Ephemeral container design**

---

## How to Use tmpfs in Docker 🧪

### Option 1 — `--tmpfs` (Simple)

```bash
docker run --tmpfs /cache nginx
````

This:

* Creates a tmpfs mount at `/cache`
* Uses system defaults
* Deletes data on container stop

---

### Option 2 — `--mount` (Explicit, Recommended)

```bash
docker run \
  --mount type=tmpfs,target=/cache \
  nginx
```

Clearer, more explicit, production-friendly.

---

## Limiting tmpfs Size (Important) ⚠️

Because tmpfs uses RAM, **unbounded usage can crash your host**.

Limit it:

```bash
docker run \
  --mount type=tmpfs,target=/cache,tmpfs-size=64m \
  nginx
```

📌 Always set size limits in production.

---

## Where tmpfs Data Actually Lives 🧠

tmpfs data lives:

* In **kernel memory**
* Possibly backed by **swap**
* Not under `/var/lib/docker`
* Not in volumes
* Not on disk

📌 Docker does not manage this data — the kernel does.

---

## How `dockerd` Handles tmpfs (Under the Hood) 🔧

When you specify:

```bash
--mount type=tmpfs
```

`dockerd`:

1. Parses the mount request
2. Marks it as **tmpfs**
3. Passes it to the container runtime
4. The Linux kernel creates the tmpfs mount
5. Docker never touches the data itself

📌 Docker is just a **translator**, not the storage owner.

---

## tmpfs vs Container Writable Layer 🧠

Important distinction:

| Writable Layer                  | tmpfs           |
| ------------------------------- | --------------- |
| Backed by disk                  | Backed by RAM   |
| Slower                          | Faster          |
| Persist until container removal | Deleted on stop |
| Can grow large                  | Must be limited |

📌 tmpfs avoids OverlayFS overhead entirely.

---

## When tmpfs Is the RIGHT Choice ✅

Use tmpfs for:

* Application caches
* Session data
* Temporary build artifacts
* Sensitive secrets
* Cryptographic material
* Token storage

Example:

```bash
docker run \
  --mount type=tmpfs,target=/run/secrets,tmpfs-size=10m \
  myapp
```

---

## When tmpfs Is a BAD Idea ❌

Do **not** use tmpfs for:

* Databases
* Logs you need to keep
* User uploads
* Anything important after restart

📌 tmpfs = **guaranteed data loss on stop**.

---

## tmpfs and Security 🔐

tmpfs improves security because:

* No disk persistence
* No leftover files
* Data disappears instantly
* Harder to exfiltrate via disk

This is why:

* Secrets should live in memory
* Not in images
* Not in volumes

---

## tmpfs and Orchestration (Preview) ☸️

In Kubernetes:

* tmpfs maps to `emptyDir` with `medium: Memory`
* Used heavily for:

  * Secrets
  * Temporary state
  * Init containers

📌 Understanding tmpfs now makes Kubernetes storage easier later.

---

## Mental Model to Lock In 🔐

> **Volumes persist.
> Bind mounts share disk.
> tmpfs forgets everything.**

Each exists for a **different purpose**.

---

## Common Beginner Mistakes ❌

* Using tmpfs for important data
* Forgetting size limits
* Confusing tmpfs with volumes
* Assuming tmpfs improves durability

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *Linux tmpfs filesystem diagram*
* *Docker tmpfs mount architecture*
* *Container memory filesystem*

---

## Official References 📚

* Docker tmpfs mounts
  [https://docs.docker.com/storage/tmpfs/](https://docs.docker.com/storage/tmpfs/)

* Linux tmpfs documentation
  [https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html](https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html)

---

## What You Learned in This Chapter ✅

* What tmpfs is at the Linux kernel level
* Why in-memory storage exists
* How Docker implements tmpfs
* How tmpfs differs from volumes and bind mounts
* When tmpfs is the right tool
* When tmpfs causes data loss
* How tmpfs fits ephemeral containers

---

📖 **Next Chapter:**
**Chapter 27 — Volume Drivers & External Storage**

Next, we move from local disks to **networked and cloud-backed storage** 🌐💽.

