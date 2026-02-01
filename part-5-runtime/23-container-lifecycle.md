
# Chapter 23 — Container Lifecycle: Create, Start, Stop, Remove 🔄🐳

Until now, we focused on **building images** correctly.  
Now we shift to the next critical skill:

> **Understanding how containers live, run, stop, and disappear.**

Many Docker issues in real projects are **not image problems**,  
they are **lifecycle misunderstandings**.

This chapter explains the **container lifecycle** in a clean, step-by-step way —  
from the moment a container is created to the moment it is removed.

---

## The Core Truth (Set the Context) 🧠

> **A container is a running process created from an image.**

So its lifecycle is tightly tied to:
- Process lifecycle
- Signals
- Exit codes
- Docker commands

No process → no container.

---

## High-Level Container Lifecycle 🧬

At a high level, a container goes through these stages:

```

Image
↓
Create
↓
Start (Running)
↓
Stop (Exited)
↓
Remove

````

Each stage has:
- A specific Docker command
- A specific purpose
- A specific state

---

## Container States (Important Vocabulary) 📌

Docker tracks container **state**, not intent.

Common states:
- `created`
- `running`
- `paused`
- `exited`
- `dead`

You can see this using:
```bash
docker ps -a
````

---

## 1️⃣ Create — Container Is Defined (But Not Running) 🧱

### Command

```bash
docker create nginx
```

### What Docker Does

* Creates a container from the image
* Sets up:

  * Filesystem (OverlayFS layers)
  * Namespaces
  * cgroups (Control Groups)
* Assigns a container ID
* **Does NOT start the process**

📌 No application is running yet.

---

### Why `create` Exists

* Lets you prepare containers ahead of time
* Useful for inspection and debugging
* Separates setup from execution

📌 Most users don’t use `create` directly — but Docker does internally.

---

## 2️⃣ Start — Process Begins Running ▶️

### Command

```bash
docker start my-container
```

### What Docker Does

* Starts the container’s main process
* That process becomes **PID 1 (Process ID 1)** inside the container
* Container state becomes `running`

📌 If the process exits, the container stops.

---

### `docker run` = Create + Start ⚡

Most people use:

```bash
docker run nginx
```

This is shorthand for:

```bash
docker create nginx
docker start <container-id>
```

📌 `docker run` is a **convenience command**.

---

## 3️⃣ Running — Container Is Alive 🟢

When running:

* The main process is executing
* STDOUT (Standard Output) and STDERR (Standard Error) are captured
* Docker monitors the process

Check running containers:

```bash
docker ps
```

📌 Docker does **not** manage application logic — only the process.

---

## 4️⃣ Stop — Graceful Shutdown ⏹️

### Command

```bash
docker stop my-container
```

### What Docker Actually Does

1. Sends `SIGTERM` (Signal Terminate) to PID 1
2. Waits (default: 10 seconds)
3. If still running → sends `SIGKILL`

📌 This relies on **proper signal handling** inside the container.

---

### Why Containers Sometimes Don’t Stop ❌

Common reasons:

* PID 1 ignores signals
* Shell form `CMD` used
* No init process
* Zombie processes

📌 This directly connects to **Chapter 18 (PID 1 & Signals)**.

---

## 5️⃣ Exit — Process Ends 🛑

When:

* The main process exits (success or failure)

Docker:

* Marks container state as `exited`
* Stores the exit code
* Keeps filesystem and metadata

Check exited containers:

```bash
docker ps -a
```

📌 Exited containers still exist.

---

### Exit Codes Matter 🎯

Exit codes indicate:

* `0` → success
* Non-zero → failure

You can inspect:

```bash
docker inspect my-container --format='{{.State.ExitCode}}'
```

📌 Orchestration systems rely heavily on exit codes.

---

## 6️⃣ Restart — Controlled Re-execution 🔁

### Restart policies

```bash
--restart=no
--restart=always
--restart=on-failure
```

Example:

```bash
docker run --restart=on-failure myapp
```

Docker:

* Watches the process
* Restarts container based on policy
* Does not care *why* it exited

📌 Restart ≠ fix. It’s retry logic.

---

## 7️⃣ Remove — Container Is Deleted 🗑️

### Command

```bash
docker rm my-container
```

### What Docker Removes

* Container metadata
* Writable container layer
* State information

### What Docker Keeps

* Image
* Volumes (unless explicitly removed)

📌 Removing a container does **not** remove the image.

---

## Forced Removal ⚠️

If container is running:

```bash
docker rm -f my-container
```

This:

* Sends `SIGKILL`
* Immediately deletes the container

📌 Use carefully — no graceful shutdown.

---

## Lifecycle + Ephemeral Nature 🌱

Containers are **ephemeral** (temporary by design):

* Meant to be created
* Meant to be destroyed
* Meant to be replaced

Correct mindset:

> **Recreate containers, don’t repair them.**

---

## Images vs Containers (Lifecycle View) ⚖️

| Aspect       | Image | Container |
| ------------ | ----- | --------- |
| Created once | ✅     | ❌         |
| Modified     | ❌     | Temporary |
| Reusable     | ✅     | ❌         |
| Ephemeral    | ❌     | ✅         |

This table explains **why data must live outside containers**.

---

## Container Lifecycle in Orchestration (Preview) ☸️

In Kubernetes (Container Orchestration Platform):

* Containers crash → restarted
* Containers fail → replaced
* Containers disappear → recreated

📌 Lifecycle understanding is mandatory before orchestration.

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *Docker container lifecycle diagram*
* *Docker run create start stop flow*
* *Container state transitions Docker*

---

## Official & Stable References 📚

### Docker Documentation

* Docker container lifecycle
  [https://docs.docker.com/engine/reference/commandline/container/](https://docs.docker.com/engine/reference/commandline/container/)

* docker run reference
  [https://docs.docker.com/engine/reference/run/](https://docs.docker.com/engine/reference/run/)

* docker stop behavior
  [https://docs.docker.com/engine/reference/commandline/stop/](https://docs.docker.com/engine/reference/commandline/stop/)

---

## The Mental Model to Lock In 🔐

> **Docker manages containers by managing processes.
> Container lifecycle = process lifecycle + metadata.**

If the process dies, the container dies.

---

## What You Learned in This Chapter ✅

* What a container lifecycle is
* Difference between create, start, run, stop, exit, and remove
* How Docker handles signals during stop
* Why exited containers still exist
* How restart policies work
* Why containers are ephemeral by design

---

📖 **Next Chapter:**
**Chapter 24 — Docker Networking Basics: Bridge, Ports, and Isolation**

Now we connect containers to the outside world 🌐.
