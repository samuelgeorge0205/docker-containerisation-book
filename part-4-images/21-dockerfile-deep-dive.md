
# Chapter 21 — Dockerfile Deep Dive: Instructions, Best Practices, and Anti-Patterns 🧱📝

You now understand **images**, **layers**, **OverlayFS (Overlay Filesystem)**, and **build caching**.
This chapter turns that knowledge into **skill**.

> **The Dockerfile is not just a setup script.  
> It is a design document for performance, security, and reproducibility.**

We’ll cover:
- Every important Dockerfile instruction
- How each instruction affects **layers and cache**
- Proven **best practices**
- Common **anti-patterns** (what *not* to do)

---

## What Is a Dockerfile? (Precise) 🧠

A **Dockerfile** is a **declarative build recipe** that tells Docker **how to create an image**.

Key properties:
- Read top → bottom
- Each instruction may create a **layer**
- Cached step-by-step
- Deterministic when written well

📌 Dockerfiles **build images**.  
📌 They do **not** run containers.

---

## The Golden Rule (Memorise This) 🔐

> **Every Dockerfile instruction is a potential layer and cache boundary.**

Good Dockerfiles:
- Maximise cache reuse
- Minimise image size
- Minimise attack surface

---

## Core Dockerfile Instructions (Deep but Clear) 🧩

We’ll go in **build order**, not alphabetically.

---

## `FROM` — Base Image 🏗️

### What it does
- Sets the base image
- Starts a new build stage

Example:
```dockerfile
FROM python:3.11-slim
````

### Best practices

* Prefer **minimal** images (`-slim`, `alpine` when appropriate)
* Pin versions (avoid `latest`)

❌ Anti-pattern:

```dockerfile
FROM ubuntu:latest
```

Why bad:

* Non-reproducible
* Unexpected changes

---

## `WORKDIR` — Working Directory 📂

### What it does

* Sets the default directory for subsequent instructions

Example:

```dockerfile
WORKDIR /app
```

### Why it matters

* Cleaner paths
* Avoids `cd` usage
* Improves readability

❌ Anti-pattern:

```dockerfile
RUN cd /app && python app.py
```

---

## `COPY` vs `ADD` — File Transfer 📁

### `COPY` (preferred)

```dockerfile
COPY requirements.txt .
```

Use when:

* Copying files/directories
* Predictability matters

### `ADD` (special cases only)

```dockerfile
ADD archive.tar.gz /app
```

Adds:

* Auto-extract for local tar files
* URL download (discouraged)

📌 **Rule**: Use `COPY` unless you *specifically* need `ADD`.

---

## `RUN` — Build-Time Execution ⚙️

### What it does

* Executes commands at **build time**
* Creates a new layer

Example:

```dockerfile
RUN apt-get update && apt-get install -y curl
```

### Best practices

* Combine related commands
* Clean package caches in the same layer

✅ Good:

```dockerfile
RUN apt-get update \
 && apt-get install -y curl \
 && rm -rf /var/lib/apt/lists/*
```

❌ Bad:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
```

Why bad:

* Extra layers
* Larger image
* Cache inefficiency

---

## `ENV` — Environment Variables 🌍

### What it does

* Sets environment variables **inside the image**

Example:

```dockerfile
ENV NODE_ENV=production
```

### Important note

* `ENV` values persist in the image
* Visible via `docker inspect`

📌 Do **not** store secrets here.

---

## `EXPOSE` — Documentation, Not Security 🚪

### What it does

* Documents intended listening ports

Example:

```dockerfile
EXPOSE 8080
```

Important:

* Does **not** publish ports
* Does **not** open firewalls

📌 Port publishing happens at `docker run`.

---

## `USER` — Drop Privileges 🔐

### What it does

* Sets the user for runtime execution

Example:

```dockerfile
USER appuser
```

### Why it matters

* Reduces blast radius
* Improves security posture

❌ Anti-pattern:

```dockerfile
USER root
```

Unless absolutely required.

---

## `CMD` vs `ENTRYPOINT` — Execution Semantics 🧬

This confuses many beginners. Let’s clarify.

---

### `CMD` — Default Command

```dockerfile
CMD ["node", "app.js"]
```

* Can be overridden at runtime
* Provides defaults

---

### `ENTRYPOINT` — Fixed Entry

```dockerfile
ENTRYPOINT ["node"]
CMD ["app.js"]
```

* ENTRYPOINT = executable
* CMD = default arguments

📌 **Best practice**: Use exec form (JSON array) to ensure proper signal handling.

❌ Anti-pattern:

```dockerfile
CMD node app.js
```

(Uses shell form → breaks signals)

---

## Layer Ordering for Cache Efficiency ⚡

Bad order:

```dockerfile
COPY . .
RUN npm install
```

Good order:

```dockerfile
COPY package.json package-lock.json ./
RUN npm install
COPY . .
```

Why:

* Dependency layer is cached
* Code changes don’t invalidate expensive installs

📌 This single change can save **minutes per build**.

---

## Multi-Stage Builds (High Impact) 🏗️🏁

### What they solve

* Large build dependencies
* Bloated final images

Example:

```dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM gcr.io/distroless/base-debian12
COPY --from=builder /app/app /app/app
CMD ["/app/app"]
```

Result:

* Small runtime image
* No compiler in production
* Improved security

📌 Multi-stage builds are **mandatory** for modern Docker usage.

---

## Common Dockerfile Anti-Patterns ❌

### ❌ Installing unnecessary tools

* Editors
* Debuggers
* Package managers in runtime image

### ❌ Storing secrets

* API keys in `ENV`
* Tokens in files

### ❌ Large context

* `COPY . .` without `.dockerignore`

### ❌ Multiple processes

* Using Dockerfile as an init system

---

## `.dockerignore` — The Silent Optimiser 🧹

Just like `.gitignore`, it:

* Reduces build context
* Improves speed
* Prevents leaks

Example:

```text
node_modules
.git
.env
```

📌 Missing `.dockerignore` is a **hidden performance killer**.

---

## The Mental Model to Lock In 🔐

> **Dockerfiles are cache-optimized build graphs, not shell scripts.**

Write them like **engineering artifacts**, not quick hacks.

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *Dockerfile instruction layers diagram*
* *Docker build cache invalidation*
* *Multi-stage Docker build diagram*

---

## Official & Stable References 📚

### Docker Documentation

* Dockerfile reference
  [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)

* Best practices for writing Dockerfiles
  [https://docs.docker.com/develop/develop-images/dockerfile_best-practices/](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

* Docker build cache
  [https://docs.docker.com/build/cache/](https://docs.docker.com/build/cache/)

---

## Why This Chapter Matters 🚦

Bad Dockerfiles:

* Slow CI/CD pipelines
* Large images
* Security risks
* Unpredictable builds

Good Dockerfiles:

* Fast builds
* Small images
* Safer runtime
* Easier scaling

This chapter is **pure leverage**.

---

## What You Learned in This Chapter ✅

* Purpose of every major Dockerfile instruction
* How instructions affect layers and cache
* Correct usage of `CMD` vs `ENTRYPOINT`
* How to structure Dockerfiles for speed
* Why multi-stage builds matter
* Common mistakes to avoid
* Why `.dockerignore` is critical

---

📖 **Next Chapter:**
**Chapter 22 — Multi-Stage Builds: Smaller, Safer Images**

We’ll go deeper into build pipelines and production-grade images.
