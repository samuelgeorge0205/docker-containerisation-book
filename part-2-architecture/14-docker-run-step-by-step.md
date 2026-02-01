
# Chapter 14 — What Happens When You Run `docker run` (Step-by-Step) 🧪🧭

You’ve learned the *pieces*:
- Docker’s client–server model
- The runtime stack (dockerd → containerd → runc)
- The kernel primitives underneath

Now we connect everything.

This chapter follows **one command** from your keyboard  
all the way down to a **running Linux process**.

> If you understand this chapter, Docker will never feel like magic again.

---

## The Command That Starts It All ▶️

```bash
docker run nginx
````

This looks simple.
Internally, it triggers **dozens of coordinated actions**.

We’ll walk through them **in exact order**.

---

## Big Picture Timeline 🧠

Here’s the full journey we’re about to trace:

```
You
↓
Docker CLI
↓
Docker Engine (dockerd)
↓
containerd
↓
runc
↓
Linux Kernel
↓
nginx process (container)
```

Each step has a clear responsibility.

---

## Step 1️⃣ — Docker CLI Parses Your Intent 🖥️

When you type:

```bash
docker run nginx
```

The Docker CLI:

* Parses flags and arguments
* Translates them into an API request
* Sends the request to the Docker Engine

📌 The CLI **does not**:

* Pull images
* Create containers
* Touch the kernel

It only **asks**.

---

## Step 2️⃣ — Docker Engine Receives the Request ⚙️

The Docker Engine (`dockerd`):

* Receives the REST API call
* Validates syntax and permissions
* Checks local state

At this point, dockerd asks:

> “Do I already have the `nginx` image?”

---

## Step 3️⃣ — Image Resolution & Pull 📦

If the image is **not present locally**:

1. Docker Engine contacts the registry (Docker Hub by default)
2. Pulls image metadata
3. Downloads image layers **only if missing**
4. Stores them locally

📌 Image layers are cached and shared across containers.

This is why:

* The first run is slow
* Subsequent runs are fast

---

## Step 4️⃣ — Container Configuration Is Created 🧾

Before anything runs, Docker prepares a **container configuration**:

* Image reference
* Command (`nginx`)
* Environment variables
* Network settings
* Volume mounts
* Resource limits

This configuration is:

* Pure metadata
* Not a running process yet

📌 At this stage, **no container exists**.

---

## Step 5️⃣ — containerd Takes Over 🧱

Docker Engine now delegates execution to **containerd**.

containerd:

* Receives the container spec
* Prepares filesystem snapshots
* Sets up writable layers
* Creates container metadata

Think of containerd as:

> “Everything needed *before* execution”

---

## Step 6️⃣ — Filesystem Is Assembled 📁

containerd assembles the container filesystem using:

* Read-only image layers
* One writable container layer
* Overlay filesystem (OverlayFS)

From inside the container:

* `/` looks like a full filesystem

From the host:

* Files are layered and shared efficiently

📌 This is where immutability meets flexibility.

---

## Step 7️⃣ — runc Is Invoked 🔩

Now comes the **birth moment**.

containerd calls `runc` with:

* OCI runtime specification
* Root filesystem path
* Namespace configuration
* cgroup limits

`runc`:

* Creates namespaces (PID, NET, MNT, etc.)
* Applies cgroups
* Drops privileges
* Executes the process

📌 This is the *only* step that directly touches the kernel to create isolation.

---

## Step 8️⃣ — The Kernel Starts the Process 🐧

At this point:

* The Linux kernel creates a new process
* Applies namespace isolation
* Enforces resource limits
* Schedules CPU and memory

The process started is:

```text
nginx
```

🎉 **The container is now running.**

Remember:

> The container *is* the process.

---

## Step 9️⃣ — runc Exits, containerd Watches 👀

After starting the process:

* runc exits immediately
* containerd continues monitoring
* Docker Engine tracks high-level state

If the process:

* Exits → container stops
* Crashes → exit code recorded

📌 Docker does **not** keep runc running.

---

## Step 🔟 — Docker Reports Back to You 🖥️

Finally:

* Docker Engine sends status to CLI
* CLI prints output or attaches logs
* Control returns to your terminal

At this point:

* The container lifecycle has begun
* Docker’s role becomes **observational**

---

## Detached vs Foreground (Quick Insight) 🎛️

* `docker run nginx`

  * Runs in foreground
  * Attaches STDOUT/STDERR

* `docker run -d nginx`

  * Runs in background
  * CLI detaches immediately

📌 Execution path is the same — only attachment differs.

---

## Where Errors Can Occur (Debugging Map) 🧭

Understanding the flow helps debugging:

| Failure Point         | Likely Layer       |
| --------------------- | ------------------ |
| Command syntax error  | Docker CLI         |
| Permission denied     | Docker Engine      |
| Image pull fails      | Registry / Network |
| Container won’t start | containerd / runc  |
| Process crashes       | Application        |
| OOMKilled             | Kernel (cgroups)   |

📌 Each error belongs to a **layer**.

---

## Mental Model to Lock In 🔐

> `docker run` is **a request**, not an action.

Docker asks.
Runtimes execute.
The kernel decides.

---

## Diagram References to Visualise the Flow 🖼️

Search for:

* *docker run internal flow diagram*
* *container lifecycle docker*
* *docker runtime execution flow*

Helpful visuals:

* Docker internals overview
  [https://www.docker.com/blog/what-is-containerd-runtime/](https://www.docker.com/blog/what-is-containerd-runtime/)

* OCI runtime flow
  [https://opencontainers.org/about/overview/](https://opencontainers.org/about/overview/)

---

## External References 📚

### Official

* Docker run reference
  [https://docs.docker.com/engine/reference/run/](https://docs.docker.com/engine/reference/run/)

### Deep Conceptual Read

* “What happens when you run docker run?” — Bret Fisher
  [https://www.bretfisher.com/what-happens-when-you-run-docker-run/](https://www.bretfisher.com/what-happens-when-you-run-docker-run/)

---

## Why This Chapter Changes Everything 🚦

After this chapter:

* You know **where to look** when things fail
* Docker errors feel logical
* Kubernetes internals stop being scary

This is the **execution spine** of containerisation.

---

## What You Learned in This Chapter ✅

* The exact lifecycle triggered by `docker run`
* Which component does what
* How images become running processes
* Where failures originate
* Why containers are just processes

---

📖 **Next Chapter:**
**Chapter 15 — OCI in Practice: Image Spec & Runtime Spec**

Now we formalise everything using standards.

