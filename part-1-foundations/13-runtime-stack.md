
# Chapter 13 — Docker Runtime Stack: dockerd → containerd → runc 🧬⚙️

In the previous chapter, you saw Docker as a **client–server system**.  
Now we go one level deeper — into the **runtime stack**.

This chapter answers a critical question:

> When Docker runs a container, **who actually does what**?

Understanding this stack is the difference between:
- “Docker feels magical”
- “Docker feels predictable”

---

## Why the Runtime Stack Exists 🧠

Early Docker versions did *everything* inside one binary.

That didn’t scale.

As containers became industry infrastructure, Docker needed:
- Clear separation of responsibilities
- Standardised execution
- Kubernetes compatibility
- Long-term stability

The result was a **layered runtime stack**.

---

## The Runtime Stack (High-Level View) 🏗️

Here is the canonical flow:

```

docker CLI
↓
dockerd (Docker Engine)
↓
containerd (container lifecycle)
↓
runc (OCI runtime)
↓
Linux Kernel

````

Each layer does **one job** — and does it well.

---

## dockerd — The Orchestrator 🧭

### What dockerd is
`dockerd` is the **Docker Engine daemon**.

It is responsible for:
- Receiving API requests
- Managing images
- Managing containers (high-level)
- Creating networks and volumes
- Enforcing Docker policies

Think of `dockerd` as:
> **The conductor of the orchestra**

It does **not**:
- Create namespaces directly
- Apply cgroups
- Start Linux processes

---

### Why dockerd Should Stay High-Level 📌

By staying high-level:
- Docker can evolve independently
- Other runtimes can be swapped in
- Kubernetes can bypass Docker entirely

This is intentional design.

---

## containerd — The Lifecycle Manager 🧱

### Why containerd exists
Docker extracted container execution into a **separate daemon** called `containerd`.

`containerd` focuses on:
- Image pulling and unpacking
- Snapshot management (filesystem layers)
- Container creation
- Container start / stop / delete
- Container state tracking

Think of `containerd` as:
> **The operations manager**

---

### containerd Is Not Docker-Specific 🔓

Important fact:

> `containerd` is a **graduated CNCF project**

This means:
- Kubernetes uses containerd directly
- Docker uses containerd
- containerd follows OCI specs

📌 containerd does **not care** about Dockerfiles, UX, or CLI design.

---

## Snapshots & Filesystems (containerd’s Hidden Power) 📁

containerd manages:
- Image layers
- Writable container layers
- Copy-on-write behavior

It uses snapshotters like:
- overlayfs
- btrfs
- zfs

📌 This is why containers are fast and storage-efficient.

---

## runc — The Moment of Creation 🔩

When it’s time to *actually run* a container:

- containerd calls `runc`
- runc implements the **OCI Runtime Specification**

### What runc does
- Creates namespaces
- Applies cgroups
- Sets root filesystem
- Drops capabilities
- Executes the process

Then runc **exits**.

📌 runc is not a daemon.  
📌 It runs, creates the container, and leaves.

---

## The OCI Runtime Contract 📜

OCI defines:
- How a container should be created
- What inputs are required
- What lifecycle hooks exist

runc is just:
> The reference implementation of those rules

This is why:
- Multiple runtimes can exist
- Docker is not locked in
- Kubernetes can switch runtimes safely

---

## The Linux Kernel — Final Authority 🐧

After runc starts the container:
- The kernel enforces namespaces
- The kernel enforces cgroups
- The kernel schedules CPU & memory

At this point:
> Docker is out of the picture.

Containers live and die by kernel behavior.

---

## A Walkthrough: `docker run` (Runtime View) 🧪

Let’s replay the command:

```bash
docker run nginx
````

From a runtime perspective:

1️⃣ `docker` CLI sends request
2️⃣ `dockerd` validates & prepares
3️⃣ `containerd` pulls image & prepares filesystem
4️⃣ `runc` creates namespaces & cgroups
5️⃣ Kernel starts `nginx` process

📌 The container **is the process**.

---

## Why This Stack Matters for Kubernetes ☸️

Kubernetes:

* Does **not** need Docker
* Talks directly to containerd (or CRI-O)
* Uses OCI runtimes underneath

This is why:

> Docker knowledge transfers directly to Kubernetes.

You are learning the **shared foundation**.

---

## Common Misconceptions (Kill These Early) ⚠️

❌ “Docker runs containers”
❌ “containerd replaces Docker”
❌ “runc is Docker-specific”

Correct view:

> Docker **coordinates**
> containerd **manages**
> runc **executes**
> kernel **enforces**

---

## Mental Model to Lock In 🔐

Think of the stack like this:

* dockerd → **Planner**
* containerd → **Manager**
* runc → **Executor**
* kernel → **Law**

Each layer has power — but also limits.

---

## Diagram References to Visualise the Runtime Stack 🖼️

Search for:

* *Docker runtime stack diagram*
* *dockerd containerd runc flow*
* *OCI runtime architecture*

Helpful visual references:

* Docker runtime deep dive
  [https://www.docker.com/blog/what-is-containerd-runtime/](https://www.docker.com/blog/what-is-containerd-runtime/)

* OCI overview
  [https://opencontainers.org/about/overview/](https://opencontainers.org/about/overview/)

---

## External References 📚

### Official

* containerd project
  [https://containerd.io/](https://containerd.io/)

### Deep Conceptual Read

* “Docker, containerd, runc — explained” — Ian Lewis
  [https://www.ianlewis.org/en/container-runtimes-part-1-introduction-container-r](https://www.ianlewis.org/en/container-runtimes-part-1-introduction-container-r)

---

## Why This Chapter Is a Turning Point 🚦

After this chapter:

* Docker stops being mysterious
* Errors feel traceable
* Kubernetes internals become approachable

You now understand **who does the work**.

---

## What You Learned in This Chapter ✅

* Why Docker uses a layered runtime stack
* The role of dockerd
* The responsibilities of containerd
* What runc actually does
* How the Linux kernel fits into everything

---

📖 **Next Chapter:**
**Chapter 14 — What Happens When You Run `docker run` (Step-by-Step)**

Now we trace the entire lifecycle — instruction by instruction.


```
