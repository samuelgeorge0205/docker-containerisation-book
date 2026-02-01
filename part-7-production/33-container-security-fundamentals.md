
# Chapter 33 — Container Security Fundamentals 🔐🛡️

Up to now, you’ve learned how to **build, run, connect, and observe containers**.
Now comes the question that separates *working systems* from *safe systems*:

> **Just because a container runs, does that mean it’s secure?**

Short answer: **No.**

This chapter builds a **security-first mental model** for containers:
- What containers *do and do not* protect you from
- Where the real attack surfaces are
- How Docker and the Linux kernel enforce isolation
- What practices actually improve security (and which don’t)

This is **fundamental security**, not fear-driven checklists.

---

## First: The Most Important Truth 🚨

> **Containers are NOT virtual machines.**

They:
- Share the host kernel
- Run as Linux processes
- Depend heavily on kernel security features

📌 Container security is **kernel security + configuration discipline**.

---

## Threat Model: What Are We Protecting Against? 🧠

Before tools, understand threats.

Typical risks:
- Compromised application inside a container
- Escaping the container to the host
- Unauthorized access to secrets
- Privilege escalation
- Lateral movement between containers

📌 Security starts with **knowing what can go wrong**.

---

## Container Isolation: What Actually Protects You 🧩

Containers rely on **Linux kernel primitives**:

| Mechanism | What It Protects |
|--------|------------------|
| Namespaces | Visibility isolation |
| cgroups (Control Groups) | Resource limits |
| Capabilities | Privilege restriction |
| seccomp (Secure Computing Mode) | Syscall filtering |
| AppArmor / SELinux | Mandatory Access Control |

Docker does not invent these — it **configures them**.

---

## Running as Root: The #1 Mistake ❌

### The Problem

By default:
- Containers run as `root`
- Root inside container ≠ harmless
- Root maps to real kernel privileges

If an attacker:
- Exploits the app
- Gains root inside container
- They gain **significant host power**

---

## Solution: Run as Non-Root User ✅

### In Dockerfile
```dockerfile
RUN adduser -D appuser
USER appuser
````

This ensures:

* Process runs with minimal privileges
* Even if compromised, damage is limited

📌 This is **one of the biggest security improvements** you can make.

---

## Linux Capabilities: Fine-Grained Privileges 🧠

Traditionally, `root` had *all* powers.
Linux breaks these into **capabilities**.

Examples:

* `CAP_NET_BIND_SERVICE` → bind to low ports
* `CAP_SYS_ADMIN` → dangerous (avoid!)

Docker:

* Drops many capabilities by default
* Allows you to add or remove explicitly

---

### Dropping Capabilities (Best Practice)

```bash
docker run --cap-drop=ALL myapp
```

Or selectively:

```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp
```

📌 Principle of **least privilege**.

---

## seccomp: Blocking Dangerous Syscalls 🚫

**seccomp (Secure Computing Mode)**:

* Filters Linux system calls
* Blocks dangerous kernel interactions

Docker applies a **default seccomp profile** that:

* Blocks risky syscalls
* Prevents kernel abuse
* Reduces attack surface

📌 Most apps work fine with defaults.

---

## AppArmor & SELinux: Mandatory Access Control 🔒

These systems:

* Enforce strict policies
* Control filesystem and process access
* Stop actions even if process is compromised

Docker integrates with:

* **AppArmor** (commonly Ubuntu)
* **SELinux** (commonly Red Hat–based systems)

📌 This is **defense in depth**, not optional.

---

## Read-Only Filesystems 📁🔐

If your app doesn’t need to write files:

```bash
docker run --read-only myapp
```

This:

* Prevents file tampering
* Blocks malware persistence
* Forces clean design

Temporary writes can use:

* tmpfs mounts

📌 Powerful and underused.

---

## Secrets: What NOT to Do ❌

Never:

* Bake secrets into images
* Commit secrets to Git
* Store secrets in environment files blindly

Why?

* Images are immutable and shared
* Environment variables leak easily
* Logs can expose them

---

## Better Approaches (At Docker Level) ✅

* Use environment variables **at runtime**
* Use tmpfs for secrets
* Inject secrets externally
* Rotate frequently

📌 Orchestration improves this further (later chapters).

---

## Image Security: Trust What You Run 🧠

Risks:

* Vulnerable base images
* Outdated dependencies
* Unknown provenance

Best practices:

* Use official images
* Use minimal base images
* Pin versions
* Scan images

Example:

```bash
docker scan myimage
```

📌 Smaller images = smaller attack surface.

---

## Network Security Basics 🌐

Containers:

* Can talk freely on the same network
* Need intentional isolation

Best practices:

* Separate frontend and backend networks
* Expose only required ports
* Avoid publishing internal services

📌 Network boundaries are security boundaries.

---

## What Containers Do NOT Protect You From ❌

Containers do **not**:

* Fix vulnerable code
* Replace patching
* Prevent logic bugs
* Secure bad credentials
* Stop supply-chain attacks

📌 Containers reduce blast radius — not responsibility.

---

## Security Is Layered (The Right Mental Model) 🧅

```
Application security
↓
Container configuration
↓
Kernel isolation
↓
Host security
↓
Infrastructure security
```

Break one layer → others still protect you.

---

## Common Beginner Security Myths ❌

* “Containers are isolated like VMs”
* “Root inside container is safe”
* “Docker images are secure by default”
* “Security is ops’ problem”

📌 Security is a **design decision**, not a toggle.

---

## Mental Model to Lock In 🔐

> **Containers limit damage — they don’t prevent compromise.
> Your job is to reduce privileges, reduce surface area, and assume breach.**

This mindset scales from Docker → Kubernetes → Cloud.

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *Docker security architecture*
* *Linux namespaces and capabilities diagram*
* *Container defense in depth*

---

## Official References 📚

* Docker security overview
  [https://docs.docker.com/engine/security/](https://docs.docker.com/engine/security/)

* Linux capabilities
  [https://man7.org/linux/man-pages/man7/capabilities.7.html](https://man7.org/linux/man-pages/man7/capabilities.7.html)

* seccomp
  [https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/)

---

## What You Learned in This Chapter ✅

* Why containers are not VMs
* Where container security actually comes from
* Why running as root is dangerous
* How Linux capabilities restrict privileges
* Role of seccomp, AppArmor, and SELinux
* Secure handling of secrets
* Image and network security basics
* Defense-in-depth mental model

---

📖 **Next Chapter:**
**Chapter 34 — Resource Limits, Performance & Tuning**

Next, we focus on **performance, limits, and efficiency**: making containers fast *and* predictable ⚙️📈.
