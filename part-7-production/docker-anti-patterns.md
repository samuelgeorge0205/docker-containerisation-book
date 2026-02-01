
# Docker Anti-Patterns — What Breaks in Production 🚨🐳

This chapter exists for one reason:

> **Most Docker outages are not caused by Docker.  
> They are caused by bad habits carried into containers.**

Anti-patterns are **not beginner mistakes**.  
They are patterns that *seem reasonable* — until scale, failure, or time exposes them.

If you understand this chapter, you’ll avoid **years of painful lessons**.

---

## What Is an Anti-Pattern? 🧠

An anti-pattern is:
- A solution that *works initially*
- Feels convenient
- Becomes dangerous in production
- Breaks reliability, scalability, or security

📌 Docker anti-patterns usually come from:
- VM-era thinking
- Manual operations mindset
- Fear of automation
- Misunderstanding container lifecycles

---

## 🚫 Anti-Pattern 1 — SSH Into Containers

### ❌ What People Do
```text
ssh into the container
fix the issue
exit
````

### Why It Feels Right

* Familiar (VM habits)
* Fast in emergencies
* Gives illusion of control

### Why It Breaks Production

* Changes are not reproducible
* Fix disappears on restart
* Creates configuration drift
* Bypasses CI/CD entirely

### ✅ Correct Approach

* Fix the Dockerfile or config
* Rebuild the image
* Redeploy the container

📌 **If you SSH into a container, the system is already broken.**

---

## 🚫 Anti-Pattern 2 — Treating Containers Like Virtual Machines

### ❌ What People Do

* Long-lived containers
* Manual patching
* Expecting “uptime”
* Caring about container identity

### Why It Breaks

* Containers are ephemeral
* Containers are replaceable
* Orchestrators assume disposability

### ✅ Correct Approach

* Treat containers as cattle
* Replace, don’t repair
* Assume restart at any time

📌 Containers are **processes**, not servers.

---

## 🚫 Anti-Pattern 3 — Stateful Containers

### ❌ What People Do

* Store database data inside container filesystem
* Rely on container-local state
* Assume container won’t restart

### Why It Breaks

* Data lost on recreation
* Scaling becomes impossible
* Backups become unreliable

### ✅ Correct Approach

* Externalize state
* Use volumes or external storage
* Design containers as stateless

📌 **State lives outside containers. Always.**

---

## 🚫 Anti-Pattern 4 — Using `latest` in Production

### ❌ What People Do

```text
myapp:latest
```

### Why It Breaks

* Tag is mutable
* Rollbacks are impossible
* Debugging becomes guesswork
* Deployments are unpredictable

### ✅ Correct Approach

* Use immutable version tags
* Promote images, don’t rebuild
* Track image digests

📌 `latest` is a convenience tag — not a deployment strategy.

---

## 🚫 Anti-Pattern 5 — Rebuilding Images Per Environment

### ❌ What People Do

* Build image in dev
* Rebuild in staging
* Rebuild again in prod

### Why It Breaks

* Environment drift
* Inconsistent behavior
* Impossible debugging

### ✅ Correct Approach

* Build once in CI
* Test the image
* Promote the same image everywhere

📌 **CI builds. Production runs.**

---

## 🚫 Anti-Pattern 6 — Baking Secrets Into Images

### ❌ What People Do

* Add passwords in Dockerfile
* Store secrets in environment baked at build time
* Commit secrets accidentally

### Why It Breaks

* Secrets leak via image history
* Rotation becomes impossible
* Security incidents escalate quickly

### ✅ Correct Approach

* Inject secrets at runtime
* Use environment variables or secret stores
* Keep images environment-agnostic

📌 Images are public artifacts — treat them as such.

---

## 🚫 Anti-Pattern 7 — No Resource Limits

### ❌ What People Do

* Run containers without CPU/memory limits
* Assume “Docker will handle it”

### Why It Breaks

* One container can starve others
* Node instability
* Random outages

### ✅ Correct Approach

* Always define resource limits
* Understand cgroups behavior
* Monitor resource usage

📌 Containers don’t self-regulate — the kernel does.

---

## 🚫 Anti-Pattern 8 — Logging to Files Inside Containers

### ❌ What People Do

* Write logs to `/var/log`
* Tail files manually
* Copy logs out during incidents

### Why It Breaks

* Logs disappear with container
* No central visibility
* Hard to debug at scale

### ✅ Correct Approach

* Log to stdout/stderr
* Use logging drivers
* Centralize logs

📌 Containers emit logs. Platforms collect them.

---

## 🚫 Anti-Pattern 9 — Manual Fixes in Production

### ❌ What People Do

* Hotfix running containers
* Restart things “until it works”
* Apply one-off changes

### Why It Breaks

* No audit trail
* No reproducibility
* Fragile systems

### ✅ Correct Approach

* Fix via CI/CD
* Redeploy cleanly
* Roll back safely

📌 Manual fixes are **technical debt with interest**.

---

## 🚫 Anti-Pattern 10 — Ignoring Failure as a Design Input

### ❌ What People Do

* Assume containers won’t crash
* Assume nodes won’t fail
* Design happy-path-only systems

### Why It Breaks

* Real systems fail
* Recovery becomes chaotic
* On-call stress skyrockets

### ✅ Correct Approach

* Design for failure
* Assume restarts
* Automate recovery

📌 **Failure is normal. Design accordingly.**

---

## Why Docker Exposes These Anti-Patterns 🧠

Docker doesn’t *cause* these issues.
It **reveals them earlier**.

Containers:

* Remove hiding places
* Enforce consistency
* Punish shortcuts

📌 Docker accelerates learning — sometimes painfully.

---

## The Production Rule to Remember 🔐

> **If a fix cannot be reproduced from source control and CI,
> it is not a fix — it is a liability.**

---

## What You Learned in This Chapter ✅

* The most common Docker anti-patterns
* Why they appear reasonable at first
* How they break production systems
* The correct, production-safe alternatives
* How Docker enforces better engineering discipline

---

## Further Reading (Optional) 📚

* Docker production best practices
  [https://docs.docker.com/develop/dev-best-practices/](https://docs.docker.com/develop/dev-best-practices/)

* Twelve-Factor App methodology
  [https://12factor.net/](https://12factor.net/)

* Docker security overview
  [https://docs.docker.com/engine/security/](https://docs.docker.com/engine/security/)

* Container logging best practices
  [https://docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/)

---

📖 **Next Chapter:**
**Performance & Resource Tuning — Making Containers Behave Under Load**

Next, we’ll look at how containers interact with CPU, memory, and the kernel —
and why performance issues often appear *only* in production ⚙️🔥.


