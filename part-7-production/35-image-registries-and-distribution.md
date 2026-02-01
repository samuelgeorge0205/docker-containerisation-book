
# Chapter 35 — Image Registries: Docker Hub, Private Registries & Distribution 📦🌍

You’ve learned how to **build images** and **run containers**.
Now we answer the question that makes containers truly powerful:

> **How do images move between laptops, servers, CI systems, and production?**

The answer is **image registries**.

This chapter explains:
- What an image registry really is
- How Docker Hub works
- Public vs private registries
- Image naming and tagging (deeply)
- Pull / push flows under the hood
- Security, trust, and best practices

This is about **distribution**, not just storage.

---

## The Big Idea 🧠

> **Images are immutable artifacts.  
> Registries are distribution systems for those artifacts.**

You build once → distribute everywhere.

---

## What Is an Image Registry? 📦

An **image registry** is:

> A service that stores Docker images and makes them available for pull and push.

A registry:
- Stores **image layers**
- Stores **image metadata**
- Handles authentication & authorization
- Serves images over HTTP(S)

📌 A registry is **not Docker itself** — Docker is a client.

---

## The Default Registry: Docker Hub 🐳

By default, Docker uses **Docker Hub**.

When you run:
```bash
docker pull nginx
````

Docker actually pulls:

```text
docker.io/library/nginx:latest
```

Because:

* `docker.io` → Docker Hub
* `library/` → official images namespace
* `nginx` → image name
* `latest` → tag

---

## Image Naming Structure (Critical) 🔑

Full image name format:

```text
[registry]/[namespace]/[repository]:[tag]
```

Examples:

```text
nginx:latest
docker.io/library/nginx:latest
docker.io/myuser/myapp:v1.2.0
ghcr.io/org/service:sha-abcdef
```

📌 If registry is omitted → Docker Hub is assumed.

---

## Tags Are NOT Versions (Very Important) ⚠️

A **tag** is:

* A mutable pointer
* A label
* Not guaranteed to be stable

Example:

```text
latest
```

Today ≠ tomorrow.

📌 **Never rely on `latest` in production.**

---

## Better Tagging Strategies 🎯

Good practices:

* Semantic versioning (`1.2.3`)
* Git commit SHA
* Build numbers

Examples:

```text
myapp:1.4.2
myapp:git-a1b2c3d
```

📌 Predictability beats convenience.

---

## How Pulling an Image Works (Under the Hood) 🧩

When you run:

```bash
docker pull myapp:1.0
```

Docker:

1. Contacts registry
2. Authenticates (if required)
3. Fetches image manifest
4. Downloads missing layers
5. Verifies checksums
6. Stores layers locally

📌 Layers are cached and shared between images.

---

## Why Registries Are Efficient 🚀

Because:

* Images are layered
* Layers are content-addressed
* Identical layers are reused

This means:

* Faster pulls
* Less bandwidth
* Less disk usage

📌 This is why small images matter.

---

## Public vs Private Registries 🔐

### Public Registry

* Anyone can pull
* Optional authentication
* Used for open-source or shared images

Examples:

* Docker Hub (public repos)
* GitHub Container Registry (public)

---

### Private Registry

* Authentication required
* Access-controlled
* Used for internal or proprietary images

Examples:

* Docker Hub private repos
* Self-hosted registry
* Cloud registries

📌 Most production systems use **private registries**.

---

## Self-Hosted Docker Registry 🏠

Docker provides an official registry image:

```bash
docker run -d -p 5000:5000 registry:2
```

Now you have:

```text
localhost:5000/myapp:1.0
```

Push:

```bash
docker push localhost:5000/myapp:1.0
```

📌 Simple, but lacks advanced features by default.

---

## Cloud & Managed Registries ☁️

Common managed registries:

* AWS Elastic Container Registry (ECR)
* Google Artifact Registry
* Azure Container Registry
* GitHub Container Registry (GHCR)

Benefits:

* Authentication integration
* High availability
* Scanning
* Access control
* Audit logs

📌 Registries are part of your **supply chain**.

---

## Authentication & Authorization 🔐

Docker uses:

```bash
docker login
```

Credentials are:

* Stored locally (credential helpers)
* Sent securely to registry
* Scoped by namespace/repo

Registries enforce:

* Who can pull
* Who can push
* Who can delete

📌 Access control is registry-level security.

---

## Image Integrity & Trust 🧠

Docker ensures:

* Layer integrity via content hashes
* Image immutability

But Docker **does not** guarantee:

* Who built the image
* Whether it’s safe

This is why:

* Trusted sources matter
* Image scanning matters
* Supply-chain security matters

---

## Image Distribution in CI/CD 🔄

Typical flow:

```text
Code → Build Image → Tag → Push to Registry → Deploy
```

Registries act as:

* The handoff point
* The contract between build and run

📌 If the registry is compromised, everything is compromised.

---

## Garbage Collection & Cleanup 🧹

Registries accumulate:

* Old tags
* Unused layers

Good hygiene:

* Retention policies
* Immutable tags
* Automated cleanup

📌 Storage grows silently if ignored.

---

## Common Beginner Mistakes ❌

* Using `latest` in production
* Rebuilding images with same tag
* Pushing secrets into images
* Using public registries blindly
* Not scanning images

---

## Mental Model to Lock In 🔐

> **Registries don’t run containers.
> They distribute immutable image artifacts.**

Build once.
Store safely.
Run everywhere.

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *Docker image registry architecture*
* *Docker pull push workflow*
* *Container image distribution pipeline*

---

## Official References 📚

* Docker registry overview
  [https://docs.docker.com/registry/](https://docs.docker.com/registry/)

* Docker Hub
  [https://hub.docker.com/](https://hub.docker.com/)

* Image tagging best practices
  [https://docs.docker.com/build/building/tagging/](https://docs.docker.com/build/building/tagging/)

---

## What You Learned in This Chapter ✅

* What image registries are and why they exist
* How Docker Hub works by default
* Image naming and tagging conventions
* Why tags are mutable and risky
* How pull and push work internally
* Public vs private registries
* Role of registries in CI/CD pipelines
* Security considerations in image distribution

---

📖 **Next Chapter:**
**Chapter 36 — Docker Swarm: Concepts & Architecture**

Now we move into **orchestration**: running containers across machines, not just on one host ☸️🚀.
