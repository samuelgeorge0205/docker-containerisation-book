
# 🐳 Docker Cheat Sheet — Concepts, Commands & Mental Models

A **practical, no-fluff reference** for Docker fundamentals, internals, and production usage.

---

## 🧠 Core Mental Models (READ THIS FIRST)

- **Image** → Immutable blueprint (read-only)
- **Container** → Running instance of an image (a Linux process)
- **Containers are ephemeral** → Replace, don’t repair
- **State lives outside containers** → Volumes / external storage
- **Docker ≠ Virtual Machines**
- **Docker uses Linux** → namespaces + cgroups + filesystems
- **Build once, run everywhere**

---

## 📦 Docker Objects (Big Picture)

| Object | What it is |
|------|-----------|
| Image | Immutable filesystem + metadata |
| Container | Running process from an image |
| Volume | Persistent data storage |
| Network | Virtual network for containers |
| Registry | Image storage & distribution |
| Service (Swarm) | Desired-state container definition |
| Task (Swarm) | Single container execution |

---

## 🔧 Docker Engine Basics

### Check Docker
```bash
docker version
docker info
````

### Docker Help

```bash
docker help
docker <command> --help
```

---

## 📸 Images

### List Images

```bash
docker images
docker image ls
```

### Pull Image

```bash
docker pull nginx
docker pull nginx:1.25
```

### Remove Image

```bash
docker rmi nginx
```

### Inspect Image

```bash
docker inspect nginx
docker history nginx
```

---

## 🚀 Containers

### Run a Container

```bash
docker run nginx
docker run -d nginx
docker run -it ubuntu bash
```

### Common Flags

```bash
-d        # detached
-it       # interactive terminal
--name    # container name
-p        # port mapping
-v        # volume mount
-e        # environment variable
--rm      # auto-remove container
```

### Example

```bash
docker run -d --name web -p 8080:80 nginx
```

---

### List Containers

```bash
docker ps
docker ps -a
```

### Stop / Start / Remove

```bash
docker stop <container>
docker start <container>
docker rm <container>
```

---

### Execute Command in Running Container

```bash
docker exec -it <container> bash
```

📌 For debugging only — **not for fixing production issues**

---

## 🧱 Dockerfile (Build Images)

### Common Instructions

| Instruction | Purpose                       |
| ----------- | ----------------------------- |
| FROM        | Base image                    |
| RUN         | Execute command at build time |
| COPY        | Copy files                    |
| ADD         | Copy + extra features         |
| CMD         | Default command               |
| ENTRYPOINT  | Main executable               |
| WORKDIR     | Working directory             |
| EXPOSE      | Document port                 |
| ENV         | Environment variable          |

---

### Build Image

```bash
docker build -t myapp:1.0 .
```

### .dockerignore

```text
node_modules
.git
.env
```

📌 Reduces image size & build time

---

## 🧪 Multi-Stage Builds

```dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM alpine
WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]
```

📌 Compiler exists only in **builder stage**, not final image.

---

## 🌐 Networking

### List Networks

```bash
docker network ls
```

### Create Network

```bash
docker network create mynet
```

### Attach Container to Network

```bash
docker run --network mynet nginx
```

📌 Containers communicate using **names**, not IPs.

---

### Port Publishing

```bash
-p host_port:container_port
docker run -p 8080:80 nginx
```

---

## 💾 Volumes & Storage

### List Volumes

```bash
docker volume ls
```

### Create Volume

```bash
docker volume create mydata
```

### Use Volume

```bash
docker run -v mydata:/data nginx
```

📌 Stored on host:

```
/var/lib/docker/volumes/mydata/_data
```

---

### Bind Mount

```bash
docker run -v /host/path:/container/path nginx
```

📌 Bind mounts are **host-dependent**

---

### tmpfs (In-Memory)

```bash
docker run --tmpfs /tmp nginx
```

📌 Data disappears on container stop

---

## 📜 Logs & Debugging

### View Logs

```bash
docker logs <container>
docker logs -f <container>
```

### Inspect Container

```bash
docker inspect <container>
```

### Resource Usage

```bash
docker stats
```

---

## ⚙️ Resource Limits (cgroups)

### CPU

```bash
--cpus="1.5"
```

### Memory

```bash
--memory="512m"
```

📌 Kernel enforces limits, not Docker.

---

## 🔐 Security Basics

* Run as **non-root**
* Avoid `--privileged`
* Drop capabilities
* Don’t bake secrets into images
* Use minimal base images

---

## 📦 Docker Compose

### Start Services

```bash
docker compose up
docker compose up -d
```

### Stop Services

```bash
docker compose down
```

### View Services

```bash
docker compose ps
```

### Logs

```bash
docker compose logs
```

📌 Compose = **multi-container on one host**

---

## ☸️ Docker Swarm (Orchestration)

### Init Swarm

```bash
docker swarm init
```

### Nodes

```bash
docker node ls
```

### Create Service

```bash
docker service create --replicas 3 nginx
```

### Scale Service

```bash
docker service scale web=5
```

### Update Service

```bash
docker service update --image nginx:1.25 web
```

📌 Swarm uses **desired state**

---

## 🔁 CI/CD Best Practices

* Build image **once**
* Test the image
* Push to registry
* Promote same image across environments
* Never rebuild in production

---

## 🚫 Common Anti-Patterns

❌ SSH into containers
❌ Stateful containers
❌ Using `latest` in prod
❌ Rebuilding images per environment
❌ No resource limits
❌ Manual fixes in production

---

## 🧠 One-Line Truths (Interview Gold)

* Containers are **processes**, not VMs
* Docker doesn’t run containers — the **kernel does**
* Images are immutable
* Containers are disposable
* Volumes outlive containers
* Orchestration replaces repair with replacement

---

## 📚 Useful References

* Docker Docs: [https://docs.docker.com/](https://docs.docker.com/)
* OCI: [https://opencontainers.org/](https://opencontainers.org/)
* Linux namespaces: [https://man7.org/linux/man-pages/man7/namespaces.7.html](https://man7.org/linux/man-pages/man7/namespaces.7.html)
* cgroups v2: [https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)

---

## 🔐 Final Mental Model

> **Docker is disciplined use of Linux primitives
> wrapped in tooling to create reliable systems.**

If this sentence makes sense, you understand Docker.
