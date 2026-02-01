
# Chapter 22 — Multi-Stage Builds: Smaller, Safer Images 🏗️🛡️

In the previous chapters, you learned:
- How images are built from **layers**
- Why images should be **immutable**
- Why containers are **ephemeral (temporary by design)**

Now we bring everything together with one of the **most important production-grade Docker concepts**:

> **Multi-stage builds** — a technique that cleanly separates *building* an application from *running* it.

This chapter exists to answer **one core question** clearly:

> **How does Docker remove the compiler and build tools from the final image?**

We’ll explain this **mechanically**, step by step — no magic, no assumptions.

---

## The Core Problem (Why This Exists) 🧠

Most applications need **heavy tools to build**:

- Compilers (Go compiler, Java compiler, C compiler)
- Package managers (npm, pip, Maven)
- Build dependencies
- Debug tools

But once the application is built, **production does NOT need**:
- The compiler
- The source code
- The build tools

Before multi-stage builds, this caused bloated images:
- Larger disk usage ❌
- Slower downloads ❌
- Bigger security attack surface ❌

---

## First: What Is a Compiler (In This Context)? 🔧

Let’s use **Go (Golang)** as the example.

```dockerfile
FROM golang:1.22
````

The image `golang:1.22` contains:

* The **Go compiler** (`go`)
* Standard Go libraries
* Build tools
* Linux OS packages

When Docker runs:

```bash
go build -o app
```

The **Go compiler**:

* Translates human-readable source code
* Into a **machine-executable binary** (`app`)

📌 **Production only needs the binary**, not the compiler.

---

## ❌ Single-Stage Build (What Goes Wrong)

```dockerfile
FROM golang:1.22
WORKDIR /app
COPY . .
RUN go build -o app
CMD ["./app"]
```

### What really happens

The final image contains:

* Go compiler ❌
* Build tools ❌
* Source code ❌
* Compiled binary ✅

Even though:

* The compiler is never used again
* Source code is unnecessary at runtime

📦 Result: **Large, unsafe image**

---

## What Is a Multi-Stage Build? 🧩

A **multi-stage build** is a Dockerfile that:

* Uses **multiple `FROM` instructions**
* Creates **temporary build images**
* Copies only the final required artifacts
* Discards everything else

In simple terms:

> **Build in one container.
> Run in another clean container.**

---

## The Core Mental Model (Very Important) 🔐

```
Stage 1 → Build environment (temporary, disposable)
Stage 2 → Runtime environment (clean, minimal, final)
```

Only the **last stage** becomes the final image.

---

## Multi-Stage Build Example (Correct Way) ✅

```dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM gcr.io/distroless/base-debian12
COPY --from=builder /app/app /app/app
CMD ["/app/app"]
```

Now let’s break this **line by line**, slowly.

---

## Stage 1 — Builder Stage 🧱

```dockerfile
FROM golang:1.22 AS builder
```

This:

* Starts a new image
* Includes the Go compiler and build tools
* Names this stage **`builder`**

📌 The name `builder` is just a **label**, like naming a workspace.

Inside this stage:

```dockerfile
RUN go build -o app
```

This produces:

```
/app/app   ← compiled binary
```

At this point, **inside the builder image**, we have:

* Go compiler
* Build tools
* Source code
* Compiled binary

---

## 🔑 Critical Rule (This Is the Key Insight)

> **Docker does NOT automatically carry anything from one stage to the next.**

Each `FROM` starts a **completely new image**.

Nothing is shared unless you **explicitly copy it**.

---

## Stage 2 — Runtime Stage 🧬

```dockerfile
FROM gcr.io/distroless/base-debian12
```

This starts a **fresh image** with:

* Minimal operating system
* ❌ No compiler
* ❌ No shell
* ❌ No package manager

This image has **zero knowledge** of Go.

---

## ✂️ The Most Important Line

```dockerfile
COPY --from=builder /app/app /app/app
```

Read it literally:

> “From the image named `builder`,
> copy the file `/app/app`
> into the current image.”

### What gets copied?

✅ Only the compiled binary

### What does NOT get copied?

❌ Go compiler
❌ Source code
❌ Build tools
❌ Temporary files

📌 This is **how the compiler is removed** —
it is **never copied into the final image**.

---

## Why Naming the Stage Matters 🏷️

```dockerfile
FROM golang:1.22 AS builder
```

Naming:

* Makes `COPY --from=builder` readable
* Improves maintainability
* Avoids numeric stage references (`--from=0`)

📌 Naming improves **clarity**, not image size.

---

## Why the Final Image Is Smaller 📦

### Single-stage image contains:

* OS
* Compiler
* Build tools
* Source code
* Binary

### Multi-stage final image contains:

* Minimal OS
* Binary only

📉 Result:

* Smaller image
* Faster downloads
* Faster startup

---

## Why This Is More Secure 🛡️

If an attacker breaks into the container:

### Single-stage image:

* Compiler exists ❌
* Shell exists ❌
* More tools to exploit ❌

### Multi-stage image:

* No compiler ✅
* No shell (distroless) ✅
* Binary-only runtime ✅

📌 **Fewer tools = smaller attack surface**

---

## Multi-Stage Builds Are Language-Agnostic 🌍

This pattern works for:

* Go (compiler → binary)
* Java (JDK → JRE)
* Node.js (build → serve static files)
* Python (build wheels → runtime)
* C / C++

Example (Node.js):

```dockerfile
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

---

## Why Multi-Stage Builds Fit Ephemeral Containers 🌱

Containers are **ephemeral**:

* Created
* Destroyed
* Recreated frequently

Multi-stage builds ensure:

* Clean rebuilds
* No runtime mutation
* Predictable images

📌 Rebuild, don’t repair.

---

## Common Anti-Patterns ❌

### ❌ Not using multi-stage builds

* Bloated images
* Security risks

### ❌ Copying everything

```dockerfile
COPY --from=builder /app /app
```

Instead:

* Copy only the final artifact

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *Docker multi-stage build diagram*
* *Builder vs runtime container*
* *Compiler removal multi-stage Docker*

---

## Official & Stable References 📚

### Docker Documentation

* Multi-stage builds
  [https://docs.docker.com/build/building/multi-stage/](https://docs.docker.com/build/building/multi-stage/)

* Dockerfile best practices
  [https://docs.docker.com/develop/develop-images/dockerfile_best-practices/](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

## The Mental Model to Lock In 🔐

> **Build environments are temporary.
> Runtime images should contain only what is required to run.**

If a tool is not needed at runtime, it should **not exist** in the final image.


---

# 🧠 One-Sentence Summary (Lock This In)

> **Multi-stage builds use one container to build artifacts, throw it away, and copy only the final output into a clean runtime container.**

---

# 🧪 Tiny Thought Experiment

Imagine this in real life:

* You cook food in a messy kitchen (builder)
* You plate food in a clean dining room (runtime)
* You don’t bring:

  * Stove
  * Gas cylinder
  * Trash
  * Utensils
* Only the food 🍽️

That’s multi-stage builds.

---

# ✅ Final Sanity Check

You should now be able to answer:

* Where is the compiler? → **Only in the builder stage**
* Is it in the final image? → **No**
* How was it removed? → **By never copying it**
* Why is the image smaller? → **No build tools included**
* Why is this safer? → **Fewer tools to exploit**

---

## FAQ — Multi-Stage Builds (Very Important Clarifications) ❓

This section exists because **almost everyone gets confused here at first**.
If you fully understand these questions, you truly understand multi-stage builds.

---

### ❓ Will I See Two Images When I Build a Multi-Stage Dockerfile?

**Short answer:**  
👉 **No, you will see only ONE image by default.**

When you run:
```bash
docker build -t myapp .
````

And then:

```bash
docker images
```

You will see **only the final image**, not the builder image.

---

### 🧠 Why Don’t I See the Builder Image?

Because of this rule:

> **Only the last stage of a Dockerfile is tagged and exposed as an image.**

Internally, Docker *does* create intermediate images for each stage, but:

* They are **temporary**
* They are **not tagged**
* They are treated as **build cache**
* Docker may delete them automatically

So they do **not** appear in normal image listings.

---

### 🔬 Can I See the Builder Image at All?

Yes — **only if you explicitly ask for it**.

Example:

```bash
docker build --target builder -t myapp-builder .
```

Now:

```bash
docker images
```

You will see:

```
myapp-builder   latest   (large size, has compiler)
myapp           latest   (small size, runtime only)
```

📌 This proves the builder stage exists, but it is **not shipped by default**.

---

### ❓ How Does Docker Discard Build Stages?

Docker does **not merge stages**.

Each `FROM` instruction:

* Starts a **completely new image**
* With a **fresh filesystem**
* With **no automatic inheritance**

Earlier stages become:

* Temporary build artifacts
* Cache material only
* Not part of the final image

> **Docker discards build stages simply by never tagging them as final images.**

---

### ❓ Exactly How Are Compilers Removed?

Important clarification:

> **Docker does not “remove” the compiler.
> It simply never copies it into the final image.**

Let’s be precise.

In this stage:

```dockerfile
FROM golang:1.22 AS builder
RUN go build -o app
```

The **Go compiler** exists and is used **only here**.

Then Docker sees:

```dockerfile
FROM gcr.io/distroless/base-debian12
```

This:

* Starts a brand-new image
* With **no compiler**
* With **no knowledge of the previous stage**

The only connection is:

```dockerfile
COPY --from=builder /app/app /app/app
```

Docker copies:

* ✅ The compiled binary

Docker does **not** copy:

* ❌ The Go compiler
* ❌ Build tools
* ❌ Source code

📌 There is **no deletion step**.
The compiler simply **never enters the final image**.

---

### 🧠 The One Mental Model That Explains Everything

> **Each build stage is like a separate container.
> Only files you explicitly copy survive.
> Everything else dies with that stage.**

---

### 🎯 Interview-Perfect Summary

* Multi-stage builds create **intermediate images**
* Only the **final stage is visible and shipped**
* Compilers exist **only in builder stages**
* Compilers are not removed — they are **never included**
* Smaller images happen because **nothing extra is copied**

If this FAQ makes sense, you fully understand multi-stage builds.

---
Perfect 👍
Below is a **copy-paste ready “Deep Insight” section** designed to sit **just before the FAQ** in **Chapter 22**.

This section explains the *mental model* once more, but from a **systems / internals perspective**, so readers don’t mistake multi-stage builds as “cleanup” or “removal”.

---

## Deep Insight — Multi-Stage Builds Are NOT Cleanup 🔍

This is the most important conceptual correction in this chapter.

Many learners think:

> “Docker builds everything first, then removes the compiler.”

❌ **That is NOT what happens.**

Let’s fix this misunderstanding permanently.

---

### ❌ What Docker Does NOT Do

Docker does **not**:
- Uninstall compilers
- Delete build tools
- Run cleanup commands
- Modify earlier images

There is **no removal step**.

---

### ✅ What Docker ACTUALLY Does

Docker follows a much simpler and more powerful rule:

> **Each `FROM` instruction starts a completely new image.**

This means:
- Each stage has its **own filesystem**
- Files do **not** automatically carry over
- Nothing survives unless you explicitly copy it

---

### Build Stages Are Disposable by Design 🗑️

Think of each build stage as:

- A temporary workspace
- A throwaway container
- A build-only environment

When Docker reaches a new `FROM`:
- The previous filesystem view is abandoned
- Docker starts fresh
- The old stage is kept only as build cache (optional)

📌 **Stages are discarded by default because they are never tagged as final images.**

---

### `COPY --from` Is the ONLY Bridge 🌉

This instruction:

```dockerfile
COPY --from=builder /app/app /app/app
````

Is the **only reason anything survives**.

Docker does exactly this:

1. Mounts the builder stage filesystem (read-only)
2. Copies the specified file
3. Places it into the new image
4. Forgets everything else

No smart logic.
No dependency tracking.
No magic.

---

### Why This Design Is So Powerful 💡

Because Docker:

* Never needs cleanup logic
* Never risks deleting the wrong files
* Never mixes build and runtime concerns

The final image is **guaranteed clean**, because:

* It never had build tools in the first place

---

### Correct Mental Model (Burn This In) 🔐

> **Multi-stage builds don’t clean images.
> They prevent unwanted files from ever entering them.**

Once you understand this, multi-stage builds feel obvious — not clever.

---

### Why This Matters in Real Systems 🚦

This design:

* Makes images reproducible
* Makes security audits simpler
* Makes builds deterministic
* Eliminates “forgot to clean” bugs

It’s not an optimisation trick.
It’s a **foundational design choice**.

---

### One-Line Truth (Interview-Grade) 🎯

> **Docker discards build stages by starting a new image at each `FROM` and only copying explicitly requested files into the final image.**

If you remember this sentence, you will never be confused again.

---

## What You Learned in This Chapter ✅

* What a compiler is in container builds
* Why single-stage builds are wasteful
* What multi-stage builds really do
* How Docker discards build stages
* How naming stages works
* Exactly how compilers are removed
* Why multi-stage builds reduce size and improve security

---

📖 **Next Chapter:**
**Chapter 23 — Container Lifecycle: Create, Start, Stop, Remove**

Now we shift from *building images* to *running containers correctly*.
