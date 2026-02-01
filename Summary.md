# Summary

---

## Part 0 — The Origin Story (Why Containers Exist)

- [Introduction: How to Read This Book](part-0-history/01-introduction.md)
- [The Bare Metal Era: Life Before Containers](part-0-history/02-bare-metal-era.md)
- [Virtual Machines: The First Revolution](part-0-history/03-virtual-machines.md)
- [Google’s Container Story: Running Millions of Processes](part-0-history/04-google-containers.md)
- [Linux Kernel Evolves: Namespaces, cgroups, and the Missing Pieces](part-0-history/05-linux-kernel-evolution.md)
- [The Birth of Docker (2013): Making Containers Usable](part-0-history/06-birth-of-docker.md)
- [OCI & Standardisation: Containers Grow Up](part-0-history/07-oci.md)

---

## Part 1 — Foundations (What Containers Really Are)

- [What Is Containerisation Really?](part-1-foundations/08-containerisation.md)
- [Containers vs Virtual Machines (Deep Comparison)](part-1-foundations/09-containers-vs-vms.md)
- [What Is Docker? (Big Picture)](part-1-foundations/10-what-is-docker.md)
- [Core Docker Terminology & Mental Models](part-1-foundations/11-terminology.md)

---

## Part 2 — Docker Architecture (How It Works)

- [Docker Architecture: Client–Server Model](part-2-architecture/12-docker-architecture.md)
- [Docker Runtime Stack: dockerd → containerd → runc](part-2-architecture/13-docker-runtime-stack.md)
- [What Happens When You Run `docker run`](part-2-architecture/14-docker-run-flow.md)
- [OCI in Practice: Image Spec & Runtime Spec](part-2-architecture/15-oci-in-practice.md)

---

## Part 3 — Containers Under the Hood (Linux Internals)

- [Linux Namespaces (Deep Dive)](part-3-internals/16-linux-namespaces.md)
- [cgroups (Control Groups): Resource Limits & Enforcement](part-3-internals/17-cgroups.md)
- [Containers Are Processes: PID 1, Signals, and Lifecycle](part-3-internals/18-containers-are-processes.md)
- [OverlayFS & Image Layers: How Images Really Work](part-3-internals/19-overlayfs-and-layers.md)

---

## Part 4 — Images & Builds

- [Docker Images Explained: Anatomy, Layers, and Caching](part-4-images/20-docker-images.md)
- [Dockerfile Deep Dive: Instructions, Best Practices, and Anti-Patterns](part-4-images/21-dockerfile-deep-dive.md)
- [Multi-Stage Builds: Smaller, Safer Images](part-4-images/22-multi-stage-builds.md)
- [Image Tagging & Versioning: Immutability in Practice](part-4-images/23-image-tagging.md)

---

## Part 5 — Running Containers (Runtime Behaviour)

- [Container Lifecycle: Create, Start, Stop, Remove](part-5-runtime/24-container-lifecycle.md)
- [Docker Networking Basics: Bridge, Ports, and Isolation](part-5-runtime/25-docker-networking.md)
- [Docker Volumes & Bind Mounts: Persistent Data Done Right](part-5-runtime/26-docker-volumes.md)
- [tmpfs Mounts & In-Memory Storage](part-5-runtime/27-tmpfs.md)
- [Volume Drivers & External Storage](part-5-runtime/28-volume-drivers.md)

---

## Part 6 — Multi-Container Systems

- [Why Docker Compose Exists](part-6-compose/29-why-docker-compose.md)
- [`docker-compose.yml` Explained Line by Line](part-6-compose/30-compose-file-explained.md)
- [Building a Real Multi-Container Application](part-6-compose/31-multi-container-app.md)
- [Networking & Volumes in Docker Compose (Deep Dive)](part-6-compose/32-compose-networking-volumes.md)

---

## Part 7 — Docker in Production (Real-World Operations)

- [Logging & Monitoring Containers](part-7-production/33-logging-and-monitoring.md)
- [Container Security Fundamentals](part-7-production/34-container-security.md)
- [Resource Limits, Performance & Tuning](part-7-production/35-performance-and-tuning.md)
- [Docker in CI/CD Pipelines](part-7-production/docker-in-ci-cd-pipelines.md)
- [Docker Anti-Patterns: What Breaks in Production](part-7-production/docker-anti-patterns.md)

---

## Part 8 — Orchestration (Docker Swarm)

- [Docker Swarm: Concepts, Architecture, Commands & Examples](part-8-orchestration/docker-swarm-complete-guide.md)
- [Docker vs Kubernetes: Philosophy Shift](part-8-orchestration/38-docker-vs-kubernetes.md)

---

## Part 9 — Interview & Mastery

- [Docker Interview Questions (Beginner)](part-09-interview/40-docker-interview-questions-beginner.md)
- [Docker Interview Questions (Advanced)](part-09-interview/41-docker-interview-questions-advanced.md)
- [Debugging Scenarios: Real-World Docker Incidents](part-09-interview/42-debugging-scenarios-real-world-docker-incidents.md)
- [Explain Docker Like a Story (STAR Method)](part-09-interview/43-explain-docker-like-a-story-star-method.md)

---

## Part 10 — Epilogue

- [Docker Is Not Magic](part-10-epilogue/44-docker-is-not-magic.md)
- [The Engineer’s Mindset](part-10-epilogue/45-the-engineers-mindset.md)

