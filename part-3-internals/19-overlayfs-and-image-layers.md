
# Chapter 19 — OverlayFS & Image Layers: How Images Really Work 🗂️🐧

So far, we’ve understood that:
- Containers are **processes**
- Containers are **isolated** using namespaces
- Containers are **limited** using cgroups (Control Groups)

But every process needs files:
- Binaries
- Libraries
- Configuration
- Runtime data

This chapter explains **how container filesystems work**, why images are **small**, why containers are **fast**, and why data inside containers is **ephemeral** (temporary) by design.

At the center of all this is a Linux kernel feature called **OverlayFS (Overlay Filesystem)**.

---

## The Core Problem Containers Had to Solve 🧠

Imagine this scenario:

> You run 100 containers based on the same Linux image.

Key question:
- Do we copy the entire filesystem **100 times**?

If we did:
- ❌ Disk usage would explode
- ❌ Startup would be slow
- ❌ Updates would be inefficient

Containers needed a filesystem model that supports:
- Sharing
- Isolation
- Immutability
- Speed

This is exactly what **OverlayFS** provides.

---

## What Is OverlayFS (Overlay Filesystem)? 🧩

### Full Form
**OverlayFS** = Overlay Filesystem

**OverlayFS** is a Linux kernel filesystem that:
- Combines multiple directories (layers)
- Presents them as **one unified filesystem**
- Uses **copy-on-write (CoW)** behavior

In simple terms:
> OverlayFS stacks filesystems on top of each other  
> and makes them look like a single filesystem.

---

## The Core Mental Model 🔐

Think of OverlayFS like transparent sheets stacked together:

```

## Writable layer (your changes)

## Image layer (application)

## Image layer (runtime)

Image layer (base OS)

````

You see **one filesystem (`/`)**,  
but it is actually made of **multiple layers**.

---

## Docker Images Are Built from Layers 📦

A **Docker image** is not a single filesystem.  
It is a **stack of read-only layers**.

Each layer:
- Is created from a Dockerfile instruction
- Represents a filesystem change
- Is immutable (never changes)

Example Dockerfile:
```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install nginx
````

This produces:

1. Ubuntu base layer
2. Package metadata layer
3. Nginx installation layer

📌 **One Dockerfile instruction → one image layer**

---

## Why Image Layers Are Read-Only 🔒

Image layers are:

* Stored on disk
* Identified by content hashes
* Shared across images and containers
* Never modified

This guarantees:

* Reproducibility
* Caching
* Safety
* Space efficiency

📌 If two images use the same base layer, that layer exists **only once** on disk.

---

## The Writable Container Layer ✍️

When a container starts:

* Docker adds **one writable layer on top of the image layers**
* This layer belongs **only to that container**

All changes go here:

* Creating files
* Modifying files
* Deleting files

📌 Image layers stay untouched.

---

## Copy-on-Write (CoW) Explained 🧠

### Full Form

**CoW** = Copy-on-Write

How Copy-on-Write works:

1. Container reads a file → read from image layer
2. Container modifies the file → file is copied to writable layer
3. Changes happen **only** in the writable layer

This gives:

* Fast reads
* Isolated writes
* Minimal disk usage

📌 Containers share data until they need to change it.

---

## What “Ephemeral” Means in Containers 🌱

### Meaning of Ephemeral

**Ephemeral** means:

> Temporary, short-lived, and not meant to persist.

---

### Ephemeral in Container Context

When we say:

> **Containers are ephemeral in nature**

We mean:

* Containers are designed to be created and destroyed frequently
* Data inside the container filesystem is **temporary**
* When a container is deleted, its writable layer is deleted

Example:

```bash
docker run alpine
# write some data
docker rm alpine-container
# all data is gone
```

📌 This is **intentional design**, not a limitation.

---

## Why Containers Are Ephemeral by Design 🔑

Containers are ephemeral because:

* Automation prefers **recreation over repair**
* Scaling requires fast replacement
* Failures are expected
* Systems like Kubernetes rely on disposability

This enables:

* Self-healing systems
* Rolling updates
* Easy rollbacks

📌 **Containers are cattle, not pets.**

---

## What “Overhead” Means ⚙️

### Meaning of Overhead

**Overhead** means:

> Extra resource cost required to support an application,
> beyond the application itself.

Examples:

* Extra memory
* Extra CPU usage
* Extra storage

---

## Overhead: Virtual Machines vs Containers ⚖️

### Virtual Machines (High Overhead)

Each Virtual Machine includes:

* Full operating system
* Separate kernel
* Background system services

This causes:

* High memory usage
* Slower startup
* Lower density

📌 That extra cost is **overhead**.

---

### Containers (Low Overhead)

Containers:

* Share the host kernel
* Do not boot an operating system
* Run only the application process

This results in:

* Low memory usage
* Fast startup
* High density

📌 **Low overhead** is why containers scale so well.

---

## OverlayFS and Low Overhead 🧠

OverlayFS contributes to low overhead because:

* Files are shared, not copied
* Only changes are stored
* No full filesystem duplication

This keeps:

* Images small
* Containers fast
* Storage efficient

---

## What Happens When a Container Is Deleted 🗑️

When you delete a container:

```bash
docker rm my-container
```

Docker removes:

* The writable container layer

Docker keeps:

* Image layers

📌 This is why deleting a container does **not** delete the image.

---

## Why You Should NOT Store Data in Containers ⚠️

Because:

* Writable layers are ephemeral
* Data disappears when containers are removed
* Performance degrades with heavy writes

Instead, use:

* Docker Volumes
* Bind Mounts
* External storage systems

📌 Containers are for **applications**, not **state**.

---

## Storage Drivers and OverlayFS 🔄

Docker supports multiple storage drivers:

* `overlay2` (recommended, default)
* `btrfs`
* `zfs`
* `devicemapper` (legacy)

### Why `overlay2` Is Preferred

* In-kernel
* Simple design
* High performance
* Widely supported

📌 Most modern Linux systems use `overlay2`.

---

## Images vs Containers (Filesystem View) ⚖️

| Aspect    | Image     | Container            |
| --------- | --------- | -------------------- |
| Layers    | Read-only | Read-only + writable |
| Mutable   | ❌ No      | ✅ Yes                |
| Shared    | ✅ Yes     | ❌ No                 |
| Ephemeral | ❌ No      | ✅ Yes                |

Understanding this table prevents **data-loss mistakes**.

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *OverlayFS Docker layers diagram*
* *Docker overlay2 filesystem*
* *Copy-on-write container filesystem*

---

## Official & Stable References 📚

### Linux Kernel Documentation

* OverlayFS
  [https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html](https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html)

### Docker Documentation

* Docker storage drivers overview
  [https://docs.docker.com/storage/storagedriver/](https://docs.docker.com/storage/storagedriver/)

* overlay2 storage driver
  [https://docs.docker.com/storage/storagedriver/overlayfs-driver/](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)

### OCI (Open Container Initiative)

* OCI Image Specification
  [https://github.com/opencontainers/image-spec](https://github.com/opencontainers/image-spec)

---

## The Mental Model to Lock In 🔐

> **Images are immutable blueprints.
> Containers add a temporary writable layer on top.**

Delete the container — the blueprint remains.

---

## Why This Chapter Matters 🚦

Understanding OverlayFS explains:

* Why containers are fast
* Why images are small
* Why data disappears
* Why volumes exist
* Why Dockerfile order matters

This is filesystem truth — not Docker magic.

---

## What You Learned in This Chapter ✅

* What OverlayFS (Overlay Filesystem) is
* How image layers work
* What copy-on-write means
* What “ephemeral” means in containers
* What “overhead” means in systems
* Why containers should not store data
* Why `overlay2` is the default storage driver

---

📖 **Next Chapter:**
**Chapter 20 — Docker Images Explained: Anatomy, Layers, and Caching**

Now we connect filesystem theory directly to image builds and Dockerfiles.
