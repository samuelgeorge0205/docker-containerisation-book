
# Chapter 42 — Debugging Scenarios (Real-World Docker Incidents) 🚨🧑‍🔧🧠

This chapter simulates **real production incidents**.
Not lab exercises. Not toy problems.

Interviewers and on-call rotations care about one thing:

> **Can you stay calm, reason clearly, and fix the system without making it worse?**

Each scenario below follows the same structure:
- **Symptoms**
- **What’s really happening**
- **How to debug (method, not magic)**
- **Correct fix**
- **Anti-patterns to avoid**
- **What the interviewer is testing**

---

## 🧭 How to Read This Chapter

Don’t jump to the fix.
Pause at the **Symptoms**, then ask yourself:

- What layer could this be?
- What changed recently?
- What signal would confirm or deny my hypothesis?

📌 Debugging is about **elimination**, not guessing.

---

## 🔥 Incident 1 — Container Keeps Restarting

### Symptoms
- Container status: `Restarting (1)`
- Service is flapping
- No traffic served

### What’s Really Happening
- The container’s **main process exits**
- Docker restarts it due to restart policy
- Often caused by:
  - App crash
  - Bad entrypoint
  - Missing config
  - Crash on startup

### How to Debug
```bash
docker logs <container>
docker inspect <container> --format '{{.State.ExitCode}}'
````

If logs are empty:

```bash
docker run -it --entrypoint sh <image>
```

### Correct Fix

* Fix the application error
* Validate `CMD` / `ENTRYPOINT`
* Ensure required env vars exist

### Anti-Patterns ❌

* Increasing restart delay
* Restarting endlessly without logs
* SSH-ing into running containers

### Interviewer Is Testing

🧠 **Process failure vs infrastructure failure**

---

## 🔥 Incident 2 — “It Works on My Machine” Image

### Symptoms

* Image works locally
* Fails in CI or production
* Missing binaries or libraries

### What’s Really Happening

* Hidden dependency on:

  * Local filesystem
  * Cached layers
  * Host libraries
* `.dockerignore` missing or wrong
* Multi-stage build misunderstood

### How to Debug

```bash
docker history <image>
docker run -it <image> sh
```

Compare:

* Local build vs CI build
* Layer contents

### Correct Fix

* Explicitly install dependencies
* Use clean, reproducible builds
* Use multi-stage builds correctly

### Anti-Patterns ❌

* Copying binaries from host
* Assuming local cache exists in CI

### Interviewer Is Testing

🧠 **Build reproducibility & image hygiene**

---

## 🔥 Incident 3 — Application Can’t Reach Another Container

### Symptoms

* Connection refused / timeout
* Services run but can’t talk
* IP-based configs fail

### What’s Really Happening

* Containers are on different networks
* Using hard-coded IPs
* Missing service DNS resolution

### How to Debug

```bash
docker network ls
docker network inspect <network>
docker exec -it <container> ping <service-name>
```

### Correct Fix

* Use user-defined networks
* Use container/service names
* Avoid IP addresses

### Anti-Patterns ❌

* Linking containers manually
* Hard-coding IPs

### Interviewer Is Testing

🧠 **Networking fundamentals & DNS understanding**

---

## 🔥 Incident 4 — Data Lost After Container Restart

### Symptoms

* Database data disappears
* Uploads missing
* Logs gone

### What’s Really Happening

* Data stored inside container filesystem
* Container was recreated
* Writable layer discarded

### How to Debug

```bash
docker inspect <container> | grep Mounts
docker volume ls
```

### Correct Fix

* Use volumes or bind mounts
* Separate state from containers
* Back up persistent data

### Anti-Patterns ❌

* “This container won’t restart”
* Writing data to `/app/data` without a volume

### Interviewer Is Testing

🧠 **Stateful vs stateless design**

---

## 🔥 Incident 5 — High Memory Usage, Random Crashes

### Symptoms

* Container killed unexpectedly
* Exit code `137`
* Node still has free memory

### What’s Really Happening

* cgroups memory limit exceeded
* Kernel OOM-kills container
* Host memory ≠ container memory

### How to Debug

```bash
docker inspect <container> | grep -i memory
docker stats
```

Check:

* Application memory usage
* Memory limits vs actual usage

### Correct Fix

* Increase memory limit
* Fix memory leak
* Tune JVM / runtime flags

### Anti-Patterns ❌

* Removing memory limits entirely
* Blaming Docker

### Interviewer Is Testing

🧠 **cgroups & kernel enforcement**

---

## 🔥 Incident 6 — CPU Spikes but No CPU Limit

### Symptoms

* Node CPU at 100%
* Other containers slow
* No CPU limits configured

### What’s Really Happening

* Container consumes all CPU
* Scheduler allows unlimited usage
* No fairness enforced

### How to Debug

```bash
docker stats
```

### Correct Fix

* Set CPU limits or shares
* Identify runaway process
* Optimize workload

### Anti-Patterns ❌

* Assuming containers self-limit
* Ignoring noisy neighbor problem

### Interviewer Is Testing

🧠 **Resource isolation awareness**

---

## 🔥 Incident 7 — Image Pull Works Locally, Fails in Prod

### Symptoms

* `image not found`
* Authentication errors
* CI succeeds, prod fails

### What’s Really Happening

* Registry auth missing
* Wrong tag
* `latest` drift
* Different registry endpoints

### How to Debug

```bash
docker pull <image>
docker login
docker service ps
```

### Correct Fix

* Use immutable tags
* Configure registry credentials
* Verify image existence

### Anti-Patterns ❌

* Retagging `latest`
* Manual image copying

### Interviewer Is Testing

🧠 **Registry & supply-chain understanding**

---

## 🔥 Incident 8 — Logs Missing in Production

### Symptoms

* App runs
* No logs in monitoring
* Logs exist locally

### What’s Really Happening

* Logging driver mismatch
* Logs written to files inside container
* No log shipping

### How to Debug

```bash
docker inspect <container> | grep LogConfig
docker logs <container>
```

### Correct Fix

* Log to stdout/stderr
* Use centralized logging
* Configure log rotation

### Anti-Patterns ❌

* Writing logs to `/var/log` inside container
* Manual log scraping

### Interviewer Is Testing

🧠 **Observability mindset**

---

## 🔥 Incident 9 — Deployment Broke After Image Update

### Symptoms

* Service behaves differently
* No config changes
* Same tag used

### What’s Really Happening

* Image tag overwritten
* Non-immutable builds
* Hidden breaking change

### How to Debug

```bash
docker inspect <container> | grep Image
docker history <image>
```

### Correct Fix

* Use versioned tags
* Promote images, don’t rebuild
* Roll back safely

### Anti-Patterns ❌

* Reusing tags
* Hot-fixing images

### Interviewer Is Testing

🧠 **Immutability & deployment discipline**

---

## 🧠 Debugging Mental Model (Lock This In)

> **Always ask:**
>
> * Is this build-time or run-time?
> * Is this container, host, or kernel?
> * Is this state or stateless?
> * Is this configuration or code?

If you answer those, the fix becomes obvious.

---

## What You Learned in This Chapter ✅

* How Docker fails in real production
* How to debug systematically
* How to avoid destructive “fixes”
* What interviewers really look for
* How Docker concepts surface under stress

---

## Further Reading (Optional) 📚

* Docker inspect reference
  [https://docs.docker.com/engine/reference/commandline/inspect/](https://docs.docker.com/engine/reference/commandline/inspect/)

* Docker logs & logging drivers
  [https://docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/)

* Docker stats & resource limits
  [https://docs.docker.com/config/containers/resource_constraints/](https://docs.docker.com/config/containers/resource_constraints/)

* OverlayFS behavior
  [https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html](https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html)

* cgroups memory management
  [https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)

---

📖 **Next Chapter:**
**Chapter 43 — Explain Docker Like a Story (STAR Method)**

Next, we convert all this knowledge into **clear storytelling** — the skill that wins interviews and earns trust 🗣️✨.

