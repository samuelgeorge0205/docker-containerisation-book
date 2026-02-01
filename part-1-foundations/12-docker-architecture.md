
# Chapter 12 — Docker Architecture: The Client–Server Model 🏗️🐳

Up to now, Docker has been explained as **ideas** and **mental models**.  
In this chapter, we zoom in on Docker as a **system**.

> When you type `docker run`, *who* actually does the work?

This chapter answers that by breaking Docker into **clear architectural pieces** and showing how they collaborate.

---

## Why Architecture Matters 🧠

If you don’t understand Docker’s architecture:
- Errors feel random
- Debugging feels like guessing
- Kubernetes feels overwhelming later

If you **do** understand it:
- Logs make sense
- Failures are predictable
- You know *where* to look when things break

---

## Docker Is a Client–Server System 🖥️↔️⚙️

At its core, Docker follows a **client–server architecture**.

That means:
- One component **accepts commands**
- Another component **does the work**

Even on your laptop, Docker is **not a single program**.

---

## The High-Level Architecture 🔍

Here’s the bird’s-eye view:

```

Docker CLI (Client)
↓
REST API (HTTP)
↓
Docker Engine (Server / Daemon)
↓
Container Runtime (containerd → runc)
↓
Linux Kernel

```

📌 Every Docker command follows this path.

---

## Docker CLI — The Client 🖥️

### What it is
The Docker CLI (`docker`) is:
- A command-line client
- Stateless
- Replaceable

It:
- Parses your command
- Sends an API request
- Displays the response

📌 The CLI **does not run containers**.

---

### Why This Matters
Because the CLI is just a client:
- You can control Docker remotely
- Docker commands can be scripted
- CI/CD systems can use Docker safely

This design enables **automation**.

---

## Docker Engine — The Server ⚙️

### What it is
The Docker Engine (often called `dockerd`) is:
- A long-running daemon
- The brain of Docker
- The system that owns resources

It:
- Listens for API requests
- Manages images
- Creates containers
- Sets up networking and storage

📌 If Docker is “down”, the daemon is usually the problem — not the CLI.

---

## Communication: REST API 🧩

The Docker CLI talks to the Docker Engine using a **REST API**.

This API:
- Uses HTTP
- Can run over:
  - Unix socket (local)
  - TCP (remote)

Example conceptually:
```

POST /containers/create
POST /containers/start

````

📌 This API-first design is why Docker integrates so well with other tools.

---

## Local vs Remote Docker Engines 🌍

Because of the client–server model:

- CLI and Engine **don’t need to be on the same machine**
- You can run:
  - CLI on your laptop
  - Engine on a remote server

This is common in:
- CI/CD pipelines
- Remote build systems
- Production environments

📌 Docker feels local, but it doesn’t have to be.

---

## Inside the Docker Engine 🧠

The Docker Engine itself does **not** talk directly to the kernel for container creation.

Instead, it delegates.

Internally, the Engine coordinates:
- Image management
- Networking
- Volumes
- Container lifecycle

And hands execution off to the runtime layer.

---

## containerd — The Lifecycle Manager 🧱

`containerd` is a **separate component** used by Docker.

Its responsibilities:
- Pull images
- Manage snapshots (filesystem layers)
- Create and manage containers
- Track container state

📌 containerd is:
- OCI-compliant
- Used by Docker **and** Kubernetes

This is a crucial architectural bridge.

---

## runc — The Kernel Interface 🔩

When it’s time to actually *start* a container:

- containerd calls `runc`
- runc:
  - Creates namespaces
  - Applies cgroups
  - Executes the process

At this point:
> The Linux kernel takes over.

📌 Docker does not bypass the kernel.  
📌 Docker **respects kernel rules**.

---

## The Kernel — The Ultimate Authority 🐧

No matter what Docker does:
- The kernel enforces isolation
- The kernel enforces limits
- The kernel schedules CPU and memory

Docker cannot:
- Break kernel rules
- Ignore cgroups
- Override namespaces

This is why:
> **Docker bugs don’t equal kernel bugs.**

---

## Putting It All Together 🧠

Let’s walk through a command:

```bash
docker run nginx
````

What happens conceptually:

1️⃣ CLI sends API request
2️⃣ Docker Engine validates request
3️⃣ Image is pulled (if needed)
4️⃣ containerd prepares container
5️⃣ runc creates namespaces & cgroups
6️⃣ Kernel runs the process

Each layer has **one job**.

---

## Why Docker Chose This Design 🏛️

This architecture provides:

* Separation of concerns
* Replaceable components
* Standardisation (OCI)
* Kubernetes compatibility
* Long-term stability

Docker could evolve without breaking users.

---

## Common Misunderstandings (Fix These) ⚠️

❌ “Docker CLI runs containers”
❌ “Docker Engine is the runtime”
❌ “Docker directly manages kernel isolation”

Correct understanding:

> Docker **coordinates**, runtimes **execute**, kernel **enforces**.

---

## Diagram References to Visualise Docker Architecture 🖼️

Search for diagrams showing:

* *Docker client server architecture*
* *Docker engine vs containerd vs runc*
* *Docker runtime stack*

Helpful visual references:

* Docker architecture overview
  [https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/)

* Runtime deep dive
  [https://www.docker.com/blog/what-is-containerd-runtime/](https://www.docker.com/blog/what-is-containerd-runtime/)

---

## External References 📚

### Official

* Docker Architecture
  [https://docs.docker.com/engine/](https://docs.docker.com/engine/)

### Deep Conceptual Read

* “Docker Internals” — Jérôme Petazzoni
  [https://jpetazzo.github.io/2014/06/10/docker-internals/](https://jpetazzo.github.io/2014/06/10/docker-internals/)

---

## The Mental Model to Lock In 🔐

> **Docker is a control system, not an execution engine.**

It tells *others* what to do — cleanly and predictably.

---

## What You Learned in This Chapter ✅

* Docker uses a client–server architecture
* The CLI is just a client
* The Engine coordinates everything
* containerd manages lifecycle
* runc creates containers
* The Linux kernel enforces reality

---

📖 **Next Chapter:**
**Chapter 13 — Docker Runtime Stack: dockerd → containerd → runc**

Now we zoom in even deeper — layer by layer.

