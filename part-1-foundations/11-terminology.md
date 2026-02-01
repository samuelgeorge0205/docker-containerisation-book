
# Chapter 11 — Core Docker Terminology & Mental Models 🧠🧩

At this point, you understand **why Docker exists** and **what role it plays**.

Now comes a critical transition.

> Docker stops feeling hard not when you learn commands,  
> but when **the words stop being confusing**.

This chapter exists to do one thing well:
**turn Docker vocabulary into clear mental models**.

---

## Why Terminology Matters More Than Commands 🗣️

Most Docker confusion sounds like this:

- “Is an image the same as a container?”
- “Is Docker the runtime?”
- “What exactly is containerd?”
- “Where does Kubernetes fit?”

These are not *skill* problems.
They are **mental model problems**.

Let’s fix them systematically.

---

## The Golden Mental Model (Read This First) 🔐

Before we define anything, lock this model in:

```

Dockerfile → Image → Container → Process

```

Everything in Docker revolves around this flow.

If this is clear, Docker is clear.

---

## Dockerfile 📝 — *The Recipe*

### What it is
A **Dockerfile** is a text file that describes **how to build an image**.

Think of it as:
> A cooking recipe, not the meal.

### Key properties
- Declarative
- Version-controlled
- Repeatable
- Build-time only

📌 **Dockerfiles do not run applications.  
They only create images.**

---

## Image 📦 — *The Blueprint*

### What it is
A **Docker image** is a **read-only template** used to create containers.

Think of it as:
> A blueprint or class definition.

### Key properties
- Immutable
- Layered
- Versioned
- Portable

📌 Images do **not change at runtime**.

---

## Container 📦▶️ — *The Running Instance*

### What it is
A **container** is a **running instance of an image**.

Think of it as:
> An object created from a class.

### Key properties
- Has a lifecycle
- Has state
- Can start, stop, restart, die
- Ephemeral by design

📌 **If the container dies, the image remains unchanged.**

---

## Process 🧬 — *The Truth Underneath*

### What it is
Inside every container is just:
> A normal Linux process.

The container:
- Isolated via namespaces
- Limited via cgroups
- Managed by a runtime

📌 Containers are not special at the kernel level.
They are **constrained processes**.

---

## Docker Engine ⚙️ — *The Manager*

### What it is
The **Docker Engine** (daemon) is the service that:
- Listens for Docker API requests
- Manages images, containers, networks, volumes
- Coordinates with runtimes

Think of it as:
> The control plane of Docker.

📌 The engine does **not** execute containers itself.

---

## Docker CLI 🖥️ — *The Remote Control*

### What it is
The **Docker CLI** is just a client.

It:
- Sends commands
- Talks to the Docker Engine via API
- Can be remote

📌 CLI ≠ Engine  
📌 CLI ≠ Runtime

---

## Runtime 🧬 — *The Execution Layer*

### What it is
The **runtime** is what actually creates containers.

In Docker:
- `containerd` manages lifecycle
- `runc` creates containers
- Kernel enforces isolation

Think of runtime as:
> The bridge between Docker and Linux.

📌 Docker uses a runtime — Docker is not the runtime.

---

## containerd 🧱 — *The Lifecycle Manager*

### What it is
`containerd` is responsible for:
- Pulling images
- Creating containers
- Managing container state
- Talking to low-level runtimes

It is:
- OCI-compliant
- Used by Docker and Kubernetes

📌 containerd does not care about Dockerfiles or UX.

---

## runc 🔩 — *The Final Executor*

### What it is
`runc`:
- Implements OCI runtime spec
- Applies namespaces and cgroups
- Starts the container process

Think of it as:
> “The moment a container is born.”

📌 runc directly interacts with the Linux kernel.

---

## Registry 🌍 — *The Image Warehouse*

### What it is
A **registry** stores Docker images.

Examples:
- Docker Hub
- Amazon ECR
- Google Artifact Registry

Think of it as:
> GitHub, but for images.

📌 Registries store images, not containers.

---

## Volume 💾 — *Persistent Data*

### What it is
A **volume** is managed storage **outside the container lifecycle**.

Used for:
- Databases
- Logs
- State

📌 Containers are disposable.  
📌 Data must survive them.

---

## Network 🌐 — *The Communication Layer*

### What it is
A Docker **network** allows containers to:
- Discover each other
- Communicate safely
- Remain isolated from the host

Docker networking is:
- Software-defined
- DNS-based
- Namespaced

---

## Compose 🧩 — *Systems, Not Containers*

### What it is
Docker Compose defines **multi-container applications**.

Think of it as:
> A blueprint for systems, not individual containers.

Compose handles:
- Multiple services
- Networks
- Volumes
- Dependencies

---

## The Full Mental Model (Put It Together) 🧠

```

Dockerfile
↓ (build)
Image
↓ (run)
Container
↓
Process (namespaces + cgroups)

```

And around it:

```

Docker CLI → Docker Engine → containerd → runc → Kernel

```

If you can redraw this from memory, you understand Docker.

---

## Common Terminology Traps (Avoid These) ⚠️

❌ “Docker runs containers directly”  
❌ “Docker = container runtime”  
❌ “Images are containers”  
❌ “Containers store data permanently”  

Each of these causes real-world confusion.

---

## Diagram References to Reinforce Models 🖼️

Search for:
- *Docker architecture diagram*
- *Dockerfile vs image vs container*
- *containerd vs runc stack*

Helpful visuals:
- Docker overview diagrams  
  https://docs.docker.com/get-started/overview/

- Runtime architecture deep dive  
  https://www.docker.com/blog/what-is-containerd-runtime/

---

## External References 📚

### Official
- Docker terminology overview  
  https://docs.docker.com/get-started/docker-concepts/the-basics/

### Deep Conceptual Read
- “Docker is not a container runtime” — Jérôme Petazzoni  
  https://jpetazzo.github.io/2015/06/14/docker-not-a-container-runtime/

---

## The One-Line Test (Self Check) ✅

If someone asks you:

> “What is a container?”

And you answer:

> “A container is a Linux process running in isolation with controlled resources, created from an immutable image.”

You’ve passed.

---

## What You Learned in This Chapter ✅

- Precise meanings of core Docker terms
- How Docker components fit together
- Correct mental models for images vs containers
- Why runtime ≠ Docker
- How to avoid common conceptual traps

---

📖 **Next Chapter:**  
**Chapter 12 — Docker Architecture (Client–Server Model)**

Now we move from *vocabulary* to *system design*.
