
# Chapter 15 — OCI in Practice: Image Spec & Runtime Spec 📜⚙️

In earlier chapters, we introduced **OCI**.  
Now we slow down and explain it **properly**, with **full forms**, real files, and real behavior.

This chapter answers a very important beginner question:

> **Why can a Docker image run in Kubernetes, Podman, or containerd without changes?**

The answer is **OCI (Open Container Initiative)**.

---

## What Is OCI (Open Container Initiative)? 🧠

**OCI (Open Container Initiative)** is a standards body under the  
**Linux Foundation**.

OCI does **not**:
- Build container tools
- Run containers
- Manage clusters

OCI only defines **specifications (rules)**.

📌 Think of OCI as:
> **The rulebook that all container tools agree to follow**

---

## Why OCI Was Needed (Quick Recap) ⚠️

Before OCI:
- Docker defined its own image format
- Docker defined how containers run
- Other tools had to follow Docker

This caused fear of:
- Vendor lock-in
- Ecosystem fragmentation
- Docker becoming “too central”

OCI solved this by making containers **tool-independent**.

---

## OCI Has Two Core Specifications 🏛️

OCI is built on **two practical specs** that you interact with every day:

1️⃣ **OCI Image Specification**  
2️⃣ **OCI Runtime Specification**

We’ll break both down clearly.

---

## Part 1️⃣ — OCI Image Specification 📦

### Full Form
**OCI Image Specification**  
(Open Container Initiative Image Specification)

---

### What Problem It Solves

It defines:
- How a container image is structured
- How image layers are stored
- How metadata is described

So that:
> An image built by one tool can run on another tool.

---

## Anatomy of an OCI Image 🧬

An OCI-compliant image consists of:

```

Image Manifest (JSON)
├── Image Configuration (JSON)
├── Layer Metadata
└── Filesystem Layers (tar blobs)

```

Each part has a strict definition.

---

### Image Layers (Filesystem Reality) 📁

**Layers** are:
- Read-only
- Content-addressed (identified by SHA-256 hashes)
- Shared across images

This enables:
- Layer reuse
- Storage efficiency
- Faster pulls

📌 If two images share a layer, it is stored **only once**.

---

### Image Configuration (What Runs) 🧾

The **image configuration file** defines:
- Default command (`CMD`)
- Entrypoint (`ENTRYPOINT`)
- Environment variables (`ENV`)
- Working directory
- User

Important:
> This configuration is **runtime-agnostic**.

---

## Why OCI Image Spec Matters in Practice 🔍

Because of OCI Image Spec:
- Docker images run in Kubernetes
- Podman images run in Docker
- containerd pulls images from Docker Hub

📌 “Docker image” is actually an **OCI image**.

---

## Part 2️⃣ — OCI Runtime Specification 🧬

### Full Form
**OCI Runtime Specification**  
(Open Container Initiative Runtime Specification)

---

### What Problem It Solves

It defines:
- How a container is created
- How isolation is applied
- How the process is started

It standardises **container execution**.

---

## The Runtime Contract: `config.json` 📄

At runtime, everything becomes a file named:

```

config.json

```

This file defines:
- Namespaces (PID – Process ID, NET – Network, MNT – Mount, etc.)
- cgroups (Control Groups) limits
- Root filesystem path
- Command to execute
- Linux capabilities

📌 This file is the **contract** between tools and runtimes.

---

## runc — OCI Runtime in Action 🔩

### What Is runc?

**runc** is:
- The reference implementation of OCI Runtime Spec
- A low-level container runtime
- A command-line tool (not a daemon)

---

### What runc Does

When invoked, `runc`:
1. Reads `config.json`
2. Creates Linux namespaces
3. Applies cgroups (Control Groups)
4. Drops unnecessary capabilities
5. Starts the process
6. Exits

📌 After this, the **Linux kernel** takes over.

---

## Important Clarification ⚠️

- **OCI Runtime ≠ Orchestrator**
- **OCI Runtime ≠ Docker**
- **OCI Runtime ≠ Kubernetes**

OCI runtime:
> Only starts containers correctly.

---

## OCI + containerd + Kubernetes ☸️

Let’s connect all full forms:

```

Kubernetes
↓
CRI (Container Runtime Interface)
↓
containerd (Container Daemon)
↓
OCI Runtime (runc)
↓
Linux Kernel

```

Because of OCI:
- Kubernetes can change runtimes
- Docker can be removed
- Images keep working

---

## Why Kubernetes Could Remove Docker 🧠

Kubernetes did **not** remove containers.
It removed **Docker as a dependency**.

Why this worked:
- Images followed OCI Image Spec
- Runtimes followed OCI Runtime Spec

📌 OCI protected the ecosystem.

---

## Mental Model to Lock In 🔐

> **OCI defines WHAT a container looks like and HOW it must run.  
> Tools decide WHEN and WHERE to run it.**

- Image Spec → content
- Runtime Spec → execution
- Tools → experience

---

## Diagram References to Visualise OCI 🖼️

Search for:
- *OCI image format diagram*
- *OCI runtime config.json*
- *containerd runc OCI flow*

Helpful references:
- OCI Overview  
  https://opencontainers.org/about/overview/

- Docker runtime architecture  
  https://www.docker.com/blog/what-is-containerd-runtime/

---

## External References 📚

### Official
- OCI Image Specification  
  https://github.com/opencontainers/image-spec

- OCI Runtime Specification  
  https://github.com/opencontainers/runtime-spec

### Deep Conceptual Read
- Red Hat — Understanding OCI  
  https://www.redhat.com/en/blog/containers-oci-image-format

---

## Why This Chapter Is Critical 🚦

After this chapter:
- Docker vs Kubernetes confusion disappears
- Runtime discussions make sense
- Container portability becomes obvious
- You understand why standards matter

OCI is the **silent backbone** of containers.

---

## What You Learned in This Chapter ✅

- What OCI (Open Container Initiative) really is
- Difference between Image Spec and Runtime Spec
- How images remain portable
- How runtimes execute containers
- Why OCI saved the container ecosystem

---

📖 **Next Chapter:**  
**Chapter 16 — Linux Namespaces (Deep Dive)**  
*(PID – Process ID, NET – Network, MNT – Mount, UTS – UNIX Time-sharing System, IPC – Inter-Process Communication)*

Now we dive into the **kernel isolation engine**.
