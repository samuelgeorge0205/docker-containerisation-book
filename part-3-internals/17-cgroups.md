
# Chapter 17 — cgroups (Control Groups): Resource Limits & Enforcement 📊🐧

In the previous chapter, we learned how **Linux namespaces** isolate *what a process can see*.

But isolation alone is dangerous.

> What if an isolated process uses **all the CPU**?  
> What if it consumes **all memory** and crashes the system?

This chapter introduces the second half of the container equation:

> **cgroups (Control Groups)** — the Linux kernel feature that enforces limits.

If namespaces create **boundaries**,  
cgroups create **discipline**.

---

## What Are cgroups (Control Groups)? 🧠

**cgroups** stands for **Control Groups**.

A **cgroup (Control Group)** is a Linux kernel mechanism that:
- Limits how much resources a process can use
- Accounts for resource usage
- Enforces fairness between processes

In simple terms:
> cgroups answer the question:  
> **“How much is this process allowed to consume?”**

---

## Why cgroups Were Needed 📈

Before cgroups:
- Processes competed freely for resources
- One runaway process could:
  - Consume all CPU
  - Exhaust all memory
  - Crash the entire system

This was unacceptable for:
- Multi-tenant systems
- Shared servers
- Containers

Google and other large-scale users needed **hard limits**, not polite requests.

---

## Namespaces vs cgroups (Clear Separation) ⚖️

Let’s clearly separate responsibilities:

| Feature | Purpose |
|------|--------|
| Namespaces | What a process can **see** |
| cgroups | What a process can **use** |

📌 Containers require **both**.

---

## The Core Mental Model 🧠🔐

> **Namespaces isolate perception.  
> cgroups enforce reality.**

A process may *think* it’s alone —  
but cgroups decide how much power it really has.

---

## What Resources Can cgroups Control? 📊

cgroups can control and track:

- **CPU** (Central Processing Unit)
- **Memory** (RAM)
- **Disk I/O** (Input / Output)
- **Network bandwidth** (indirectly)
- **Process counts** (PIDs)

Docker uses these controls constantly — often without you noticing.

---

## CPU Control with cgroups 🖥️

### What CPU cgroups do
CPU cgroups:
- Limit CPU time
- Control scheduling priority
- Prevent CPU starvation

Example Docker flag:
```bash
docker run --cpus="1.5" nginx
````

This means:

* The container can use at most 1.5 CPU cores
* Even if more are available

📌 The kernel enforces this — not Docker.

---

## Memory Control with cgroups 🧠

### Why Memory Limits Matter

Memory is finite.
When it’s exhausted, the system becomes unstable.

cgroups allow:

* Hard memory limits
* Memory accounting
* Automatic enforcement

Example:

```bash
docker run --memory="512m" nginx
```

This means:

* The container cannot exceed 512 MB of RAM

---

## OOMKill (Out Of Memory Kill) ☠️

### Full Form

**OOMKill** = Out Of Memory Kill

When a process exceeds its memory limit:

* The kernel kills it immediately
* The container exits
* An OOM event is recorded

📌 This is not a Docker bug.
📌 This is the kernel protecting the system.

---

## Disk I/O Control (Input / Output) 💾

### What It Controls

cgroups can:

* Throttle disk reads
* Throttle disk writes
* Prevent I/O-heavy containers from starving others

This matters for:

* Databases
* Logging-heavy applications
* Shared storage systems

---

## Process Limits (PIDs Controller) 🧬

### Full Form

**PID** = Process ID

cgroups can limit:

* Number of processes inside a container

Example use case:

* Prevent fork bombs
* Prevent accidental process explosions

📌 This is critical for container security and stability.

---

## cgroups v1 vs cgroups v2 (Important) 🔄

Linux currently supports two versions.

---

### cgroups v1 (Legacy)

* Multiple hierarchies
* Complex configuration
* Hard to reason about
* Widely deployed historically

---

### cgroups v2 (Modern)

* Single unified hierarchy
* Simpler model
* Stronger guarantees
* Better suited for containers

📌 Modern Linux distributions default to **cgroups v2**.

Docker and Kubernetes fully support both.

---

## How Docker Uses cgroups 🧩

When you run:

```bash
docker run --memory=512m --cpus=1 nginx
```

Docker:

1. Converts flags into cgroup configuration
2. Passes them to the runtime
3. The kernel enforces the limits

Docker never enforces limits itself.

> **Docker requests.
> The kernel decides.**

---

## cgroups + Namespaces = Containers 🧠

This is the **complete container formula**:

```
Process
├─ Namespaces → isolation (what it sees)
└─ cgroups → limits (what it can use)
```

Remove either one — containers break.

---

## Why cgroups Are Critical for Multi-Tenancy 🏢

In shared environments:

* Many containers
* Many users
* Limited resources

Without cgroups:

* One bad container ruins everything

With cgroups:

* Fairness is enforced
* Systems remain stable
* Scaling becomes safe

---

## Diagram References to Visualise cgroups 🖼️

Search for:

* *Linux cgroups diagram*
* *Container resource limits cgroups*
* *cgroups v1 vs v2 architecture*

Helpful references:

* Linux cgroups v2 documentation
  [https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)

* Red Hat — Control Groups explained
  [https://www.redhat.com/en/blog/world-according-cgroups-part-one](https://www.redhat.com/en/blog/world-according-cgroups-part-one)

---

## External References 📚

### Official (Kernel Docs)

* cgroups v2 — Linux Kernel Documentation
  [https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)

### Deep Conceptual Read

* “The world according to cgroups” — Red Hat
  [https://www.redhat.com/en/blog/world-according-cgroups-part-one](https://www.redhat.com/en/blog/world-according-cgroups-part-one)

---

## Important Reality Check ⚠️

cgroups:

* Do not guarantee performance
* Do not prevent all denial-of-service attacks
* Are not security boundaries by themselves

They are **resource controls**, not magic shields.

---

## The Mental Model to Lock In 🔐

> **Namespaces define the container’s world.
> cgroups define the container’s limits.**

Together, they make containers safe.

---

## What You Learned in This Chapter ✅

* What cgroups (Control Groups) are
* Why resource limits are necessary
* How CPU, memory, and I/O are controlled
* What OOMKill (Out Of Memory Kill) means
* The difference between cgroups v1 and v2
* How Docker relies on the kernel for enforcement

---

📖 **Next Chapter:**
**Chapter 18 — Containers Are Processes: PID 1, Signals, and Lifecycle**

Now we confront the truth every container engineer must understand.
