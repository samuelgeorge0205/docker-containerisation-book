
# Chapter 32 — Logging & Monitoring Containers 📊🔍

You’ve built containers.  
You’ve wired networking and storage.  
Now comes the **operational question** every real system faces:

> **How do I know what my containers are doing—right now and over time?**

This chapter explains **logging and monitoring for containers** from first principles:
- Where logs actually go
- How Docker handles logs
- Why monitoring is different from logging
- What to do in development vs production
- Mental models that scale to orchestration

No magic tools yet—just **clear fundamentals**.

---

## First: Logging vs Monitoring (Do Not Mix These) 🧠

### Logging
> **Logs answer: “What happened?”**

- Errors
- Requests
- Stack traces
- Application messages

📌 Logs are **event records**.

---

### Monitoring
> **Monitoring answers: “How is it behaving?”**

- CPU usage
- Memory consumption
- Disk I/O
- Network traffic
- Health status

📌 Monitoring is **continuous measurement**.

You need **both**, for different reasons.

---

## The Container Logging Problem 🚧

In traditional systems:
- Apps wrote logs to files
- Files lived on disk
- Operators tailed files

In containers:
- Filesystems are ephemeral
- Containers restart frequently
- Writing logs to files causes:
  - Data loss
  - Disk bloat
  - Complex volume management

So containers follow a **new rule**.

---

## The Golden Rule of Container Logging 🔐

> **Containers should log to stdout and stderr—not to files.**

Why?
- Docker captures stdout/stderr automatically
- Logs survive container restarts (until rotation)
- Logs can be forwarded easily
- Works consistently across environments

📌 This rule underpins everything else.

---

## How Docker Logging Works (Under the Hood) 🧩

When a container runs:
1. Application writes to stdout/stderr
2. Docker captures the output
3. Docker sends it to a **logging driver**
4. Logs are stored or forwarded

```text
App → stdout/stderr → Docker → Logging Driver
````

📌 Docker does **not parse logs**—it only transports them.

---

## Default Logging Driver: `json-file` 📄

By default, Docker uses:

```text
json-file
```

This means:

* Logs are stored as JSON
* Stored on the host filesystem
* Rotated based on configuration

You can see logs with:

```bash
docker logs <container>
```

---

## Where Are Logs Stored? 📍

On native Linux hosts:

```text
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

On Docker Desktop:

* Inside the Linux VM
* Not directly accessible from host OS

📌 **Do not manually edit these files.**

---

## Log Growth & Rotation (Very Important) ⚠️

Without limits:

* Logs grow forever
* Disk fills up
* Host crashes

Always configure rotation:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

📌 Logging misconfiguration is a **real production outage cause**.

---

## Logging in Docker Compose 🧩

Compose supports logging configuration:

```yaml
services:
  api:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

📌 This applies per service.

---

## Centralized Logging (Why Local Logs Are Not Enough) 🌐

On a single machine:

* `docker logs` is fine

In real systems:

* Containers restart
* Hosts change
* Logs must be searchable
* Logs must persist independently

This leads to:

> **Centralized logging**

---

## Common Logging Drivers (Conceptual) 🗂️

Docker supports drivers like:

* `json-file` (local)
* `syslog`
* `journald`
* `fluentd`
* `awslogs`

📌 Docker doesn’t care **where logs go**—drivers do.

---

## Logging Mental Model 🔐

> **Containers emit logs.
> Docker ships logs.
> External systems store and analyze logs.**

Never reverse this flow.

---

## Now Monitoring: A Different Problem 📈

Monitoring is about:

* Resource usage
* Performance
* Health
* Capacity planning

Logs won’t tell you:

* CPU spikes
* Memory leaks
* Throttling
* Network saturation

You need **metrics**.

---

## Docker-Level Monitoring Basics 🔍

### `docker stats`

```bash
docker stats
```

Shows:

* CPU usage
* Memory usage
* Network I/O
* Block I/O

📌 This is **real-time**, not historical.

---

## What Docker Can and Cannot Monitor 🧠

Docker can show:

* Per-container resource usage
* cgroups (Control Groups) limits & usage

Docker cannot:

* Store historical metrics
* Alert
* Correlate across hosts

📌 Docker gives **visibility**, not observability.

---

## Resource Limits Make Monitoring Meaningful ⚖️

Monitoring only matters if limits exist.

Example:

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          memory: 512m
```

Now:

* Exceeding memory → container killed
* Metrics have context
* Behavior is predictable

📌 Unlimited resources = meaningless metrics.

---

## Health Checks (Monitoring at the App Level) ❤️

Docker supports health checks:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000 || exit 1
```

Docker tracks:

* healthy
* unhealthy
* starting

Check status:

```bash
docker ps
```

📌 Health ≠ running.

---

## Health Checks in Compose 🧩

```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
      interval: 30s
      retries: 3
```

📌 This bridges **application state** and **container state**.

---

## Logs vs Metrics vs Health (Side-by-Side) ⚖️

| Aspect    | Logs            | Metrics          | Health              |
| --------- | --------------- | ---------------- | ------------------- |
| Purpose   | Debug events    | Observe behavior | Determine readiness |
| Frequency | Event-driven    | Continuous       | Periodic            |
| Stored    | Yes             | Usually          | No                  |
| Used for  | Troubleshooting | Alerting         | Automation          |

You need **all three**.

---

## Common Beginner Mistakes ❌

* Logging to files inside containers
* Not rotating logs
* Treating `docker stats` as monitoring
* Ignoring health checks
* Mixing logs and metrics

---

## Mental Model to Lock In 🔐

> **Logs explain failures.
> Metrics predict failures.
> Health checks automate reactions.**

If you remember this, you’ll design sane systems.

---

## Diagram References (Search-Friendly) 🖼️

Search for:

* *Docker logging architecture*
* *Container monitoring metrics flow*
* *Docker health check lifecycle*

---

## Official References 📚

* Docker logging overview
  [https://docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/)

* Docker stats
  [https://docs.docker.com/engine/reference/commandline/stats/](https://docs.docker.com/engine/reference/commandline/stats/)

* Docker health checks
  [https://docs.docker.com/engine/reference/builder/#healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck)

---

## What You Learned in This Chapter ✅

* Difference between logging and monitoring
* Why containers log to stdout/stderr
* How Docker captures and stores logs
* Why log rotation is critical
* Docker-level monitoring capabilities
* Role of health checks
* How logging and monitoring fit container operations

---

📖 **Next Chapter:**
**Chapter 33 — Container Security Fundamentals**

Next, we focus on **running containers safely**: users, permissions, capabilities, and attack surfaces 🔐🛡️.
