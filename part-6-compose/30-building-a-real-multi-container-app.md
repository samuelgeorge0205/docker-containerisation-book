
# Chapter 30 — Building a Real Multi-Container Application 🛠️🚀

So far, we’ve learned **why Docker Compose exists** and **how to read a `docker-compose.yml` file**.
Now it’s time to **use it for real**.

In this chapter, we will:
- Design a realistic multi-container system
- Wire services together correctly
- Use networks, volumes, and environment variables
- Understand *why* each choice is made
- Run, inspect, debug, and tear down the system

This is the chapter where **everything finally connects**.

---

## The Application We Will Build 🧩

We’ll build a **classic, real-world architecture**:

```

Client (Browser)
↓
Web App (Backend API)
↓
Database (PostgreSQL)

````

### Components

1. **Backend API**
   - Simple application (Node.js example)
   - Stateless
   - Talks to database via service name

2. **Database**
   - PostgreSQL (stateful)
   - Uses a Docker volume

3. **Docker Compose**
   - Single source of truth
   - One command to run everything

📌 This architecture mirrors what you’ll see in:
- Interviews
- Real projects
- Kubernetes later on

---

## Project Directory Structure 📂

Create a clean project layout:

```text
multi-container-app/
├── docker-compose.yml
└── app/
    ├── Dockerfile
    ├── package.json
    └── index.js
````

📌 Clear structure makes systems easier to reason about.

---

## Backend Application (Stateless Service) ⚙️

### `app/package.json`

```json
{
  "name": "sample-api",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "express": "^4.19.2",
    "pg": "^8.11.3"
  }
}
```

---

### `app/index.js`

```js
const express = require("express");
const { Client } = require("pg");

const app = express();

const client = new Client({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
});

client.connect();

app.get("/", async (req, res) => {
  const result = await client.query("SELECT NOW()");
  res.json({ time: result.rows[0] });
});

app.listen(3000, () => {
  console.log("API running on port 3000");
});
```

📌 Notice:

* No hardcoded IPs
* Configuration via environment variables
* Stateless logic

---

## Backend Dockerfile 🐳

### `app/Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY index.js .
CMD ["node", "index.js"]
```

📌 Simple, clean, reproducible.

---

## Docker Compose File (The System Blueprint) 📄

### `docker-compose.yml`

```yaml
version: "3.9"

services:
  api:
    build: ./app
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_USER: postgres
      DB_PASSWORD: example
      DB_NAME: appdb
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: example
      POSTGRES_DB: appdb
    volumes:
      - dbdata:/var/lib/postgresql/data

volumes:
  dbdata:
```

---

## Breaking Down the Architecture 🧠

### 1️⃣ Networking (Implicit but Critical)

* Docker Compose creates a **user-defined bridge network**
* Services can reach each other by **service name**
* `DB_HOST=db` works because of embedded DNS

📌 No IP addresses. No manual networks.

---

### 2️⃣ Storage (Stateful vs Stateless)

* `api` → stateless (no volume)
* `db` → stateful (volume-backed)

📌 If the database container is deleted → data remains.

---

### 3️⃣ Startup Ordering (`depends_on`) ⏱️

```yaml
depends_on:
  - db
```

This ensures:

* Database container starts first

⚠️ Important:

* It does **not** guarantee DB readiness
* Only startup order

📌 Production systems add retry logic.

---

## Running the Application ▶️

From the project root:

```bash
docker compose up --build
```

Docker Compose will:

1. Build the API image
2. Create a network
3. Create a volume
4. Start the database
5. Start the API

---

## Testing the System 🧪

Open browser or run:

```bash
curl http://localhost:3000
```

Expected output:

```json
{
  "time": "2026-02-01T12:34:56.789Z"
}
```

📌 This confirms:

* API is running
* Database connection works
* Containers are communicating

---

## Inspecting the Running System 🔍

### List containers

```bash
docker compose ps
```

### View logs

```bash
docker compose logs api
docker compose logs db
```

### Inspect volume

```bash
docker volume ls
```

📌 Everything is observable and debuggable.

---

## Stopping vs Destroying the System 🧹

### Stop containers

```bash
docker compose stop
```

* Containers stop
* Volumes remain
* Data remains

### Destroy everything

```bash
docker compose down
```

* Containers removed
* Network removed
* Volumes remain

To remove volumes too:

```bash
docker compose down -v
```

⚠️ This deletes database data.

---

## Why This Architecture Is “Correct” ✅

This design follows best practices:

* Stateless services
* Stateful data in volumes
* Declarative system definition
* No hardcoded networking
* Easy teardown & rebuild

📌 This is **production thinking**, even in local development.

---

## Common Beginner Mistakes ❌

* Using IP addresses instead of service names
* Storing DB data inside container filesystem
* Publishing database ports unnecessarily
* Forgetting volumes
* Assuming `depends_on` waits for readiness

---

## Mental Model to Lock In 🔐

> **Docker Compose describes a system, not containers.
> Services communicate by name, data lives in volumes, and everything is replaceable.**

If you understand this chapter, you understand **real containerized systems**.

---

## What You Learned in This Chapter ✅

* How to design a real multi-container application
* How services communicate using Docker networking
* How volumes protect stateful data
* How Compose orchestrates containers locally
* How to run, inspect, and tear down systems safely
* How Docker concepts combine into real architectures

---

📖 **Next Chapter:**
**Chapter 31 — Networking & Volumes in Docker Compose (Deep Dive)**

Next, we zoom in and master **how Compose wires networking and storage internally** 🧠🌐💾.
