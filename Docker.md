# Docker 
---

## 1. What Problem Does Docker Solve?

Imagine you build an app on your laptop. It works perfectly. You send it to a friend or deploy it to a server — and it crashes.

Why? Because your laptop has:
- Python 3.11
- PostgreSQL 15
- Library X version 2.1

But the server has:
- Python 3.9
- PostgreSQL 13
- Library X version 1.8

Same code, different environment = different behaviour. This is the classic **"it works on my machine"** problem.

**Docker solves this by packing your app + Python + all libraries + the OS into one box. That box runs identically everywhere.**

---

## 2. The Real World Analogy

Think of **shipping containers** (the big metal ones on cargo ships).

Before standardized containers, every ship had a different shape, and loading/unloading was a nightmare. After containers — same box, fits on any ship, any truck, any port.

Docker does the same for software. Pack your app into a standard container. Run it on any machine.

---

## 3. Three Core Concepts

### Image
An image is like a **blueprint or recipe**. It is read-only — you cannot run it directly.

> Think of it like a class in programming. You don't run the class. You create an instance from it.

Example: `python:3.11-slim` is an image. It contains Python 3.11 pre-installed on a slim Linux OS.

### Container
A container is a **running instance of an image**.

> Think of it like an object created from a class.

You can create 10 containers from the same image. Each one is isolated — has its own files, environment variables, and processes. One crashing does not affect the others.

### Dockerfile
A Dockerfile is the **step-by-step instructions for building an image**.

> Think of it like a recipe for a cake. The recipe is the Dockerfile. The baked cake is the image. Eating the cake is the container running.

---

## 4. How Docker Works Under The Hood (Simple Version)

Docker uses three Linux features:

**Namespaces — Isolation**
Each container thinks it is the only process on the machine. It cannot see other containers' files, processes, or network. Complete isolation.

**Cgroups — Resource Limits**
You can limit how much RAM or CPU a container uses. Multiple containers share the server without stealing from each other.

**Union File System — Layers**
Docker builds images in layers, like stacking transparent sheets. Each line in your Dockerfile creates one layer. Layers are cached and reused, making builds fast.

---

## 5. The Dockerfile — Explained Line by Line

```dockerfile
FROM python:3.11-slim
```
Start with an existing image as your base. Like starting a recipe with store-bought dough instead of making it from scratch.

```dockerfile
WORKDIR /app
```
Set the working directory inside the container. All future commands run from here.

```dockerfile
COPY requirements.txt .
```
Copy `requirements.txt` from your machine into the container's `/app` folder.

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```
Run this command inside the container. Installs all your Python packages.

```dockerfile
COPY . .
```
Copy your entire project code into the container.

```dockerfile
EXPOSE 8000
```
Document that this container listens on port 8000. Does not actually open the port — just a label.

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```
The command that runs when the container starts. This is what actually starts your app.

---

## 6. Why Order Matters in a Dockerfile

Docker caches each layer. If a layer has not changed, it reuses the cached version and skips it.

**Wrong order (slow every time):**
```dockerfile
COPY . .                          # copies everything including code
RUN pip install -r requirements.txt  # reinstalls every time code changes
```

**Right order (fast after first build):**
```dockerfile
COPY requirements.txt .           # only requirements
RUN pip install -r requirements.txt  # cached if requirements didn't change
COPY . .                          # your code (changes often, but comes after)
```

Every time you change your code and rebuild, Docker skips the slow pip install step because `requirements.txt` did not change.

---

## 7. Key Docker Commands

```bash
# Build an image from a Dockerfile in current directory
docker build -t my-app .

# Run a container from an image
docker run -d -p 8000:8000 --name my-container my-app

# See running containers
docker ps

# See all containers including stopped ones
docker ps -a

# See all images on your machine
docker images

# Stop a container
docker stop my-container

# Remove a container
docker rm my-container

# Remove an image
docker rmi my-app

# See live logs from a container
docker logs -f my-container

# Open a terminal inside a running container
docker exec -it my-container bash
```

---

## 8. The Run Command — Explained

```bash
docker run -d -p 8000:8000 --name my-container my-app
```

| Flag | Meaning |
|------|---------|
| `-d` | Detached mode — runs in background, does not block your terminal |
| `-p 8000:8000` | Port mapping — your machine's port 8000 → container's port 8000 |
| `--name my-container` | Give the container a name instead of a random one |
| `my-app` | Which image to use |

Port mapping explained:
```
Your browser → localhost:8000 → Docker → container's port 8000 → your app
```

---

## 9. Docker vs Virtual Machines

People often confuse these. They are different.

| | Virtual Machine | Docker Container |
|---|---|---|
| Size | 5–20 GB each | 100–500 MB each |
| Startup time | 30–60 seconds | Under 1 second |
| Has its own OS | Yes, full OS | No, shares host kernel |
| Isolation | Complete | Near-complete |
| Use case | Run different OS | Deploy apps consistently |

Docker is lightweight because containers share the host machine's Linux kernel. VMs carry a full OS inside them.

---

## 10. .dockerignore — Keep Your Image Small

Just like `.gitignore` tells Git which files to ignore, `.dockerignore` tells Docker which files to exclude from the build.

Without `.dockerignore`:
```
Sending build context to Docker daemon  527.7MB  ← your entire project
```

With `.dockerignore`:
```
Sending build context to Docker daemon  683kB  ← only what's needed
```

Example `.dockerignore`:
```
venv/
__pycache__/
*.pyc
.env
.git
node_modules/
```

---

## 11. Docker Compose — Running Multiple Containers Together

Most real apps need more than one service. Example: your app + a database.

Without Compose, you would run each container manually and wire them together. Painful.

With Compose, you write one `docker-compose.yml` file:

```yaml
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    env_file:
      - ./backend/.env
    restart: always
```

Then one command starts everything:
```bash
docker-compose up -d
```

**Key Docker Compose commands:**
```bash
docker-compose up -d       # start all services in background
docker-compose down        # stop and remove all containers
docker-compose logs -f     # watch live logs
docker-compose ps          # see status of all services
docker-compose up --build  # rebuild images and restart
```

---

## 12. Container vs Your Machine — What Lives Where

```
Your machine (host)
│
├── Docker daemon (runs in background, manages everything)
│
├── Image: python:3.11-slim (downloaded from Docker Hub)
├── Image: chikitsa-backend (built from your Dockerfile)
│
└── Container: chikitsa (running instance of chikitsa-backend)
      ├── Has its own filesystem (/app/...)
      ├── Has its own network (isolated)
      ├── Has its own processes (uvicorn running as PID 1)
      └── Port 8000 mapped to your machine's port 8000
```

---

## 13. Docker Hub — The Registry

Docker Hub is like GitHub but for Docker images.

- You `push` your image to Docker Hub
- Server `pulls` your image from Docker Hub
- Server `runs` the container

```bash
# Tag your image with your Docker Hub username
docker tag my-app yourusername/my-app:v1.0

# Push to Docker Hub
docker push yourusername/my-app:v1.0

# Pull on any machine
docker pull yourusername/my-app:v1.0

# Run it
docker run yourusername/my-app:v1.0
```

This is how CI/CD pipelines work — build the image, push it, deploy it.

---

## 14. Why Docker Matters for Deployment

| Scenario | Without Docker | With Docker |
|---|---|---|
| New developer joins team | 2 days setting up environment | `docker-compose up` — done in 5 minutes |
| Deploy to production | SSH, install dependencies, pray | `docker pull`, `docker run` — done in 2 minutes |
| App crashes on server | "Works on my machine" — debug for hours | Same image locally and on server — reproduce instantly |
| Scale to handle more traffic | Manual server setup for each instance | Spin up 10 containers with one command |
| Rollback bad deployment | Re-install old version manually | `docker run my-app:v1.9` — instant rollback |

---

## 15. The Full Mental Model

```
You write code
    ↓
You write a Dockerfile (recipe)
    ↓
docker build (follow the recipe)
    ↓
Image created (frozen blueprint, stored locally)
    ↓
docker push (upload to Docker Hub)
    ↓
docker pull on any server (download the blueprint)
    ↓
docker run (create a live container from the blueprint)
    ↓
Your app is running — identically on every machine
```

---

## Quick Reference Card

| Term | Plain English |
|---|---|
| Image | Blueprint / frozen snapshot of your app + environment |
| Container | Running instance of an image |
| Dockerfile | Step-by-step instructions to build an image |
| Docker Hub | Online store for Docker images |
| Registry | Any place that stores Docker images |
| Build context | Files Docker copies during `docker build` |
| Layer | One step in a Dockerfile = one cached layer |
| Namespace | Linux feature that isolates containers from each other |
| Cgroup | Linux feature that limits container resource usage |
| Volume | Persistent storage that survives container restarts |
| Port mapping | Connecting your machine's port to container's port |
| Compose | Tool to manage multiple containers as one app |
| `.dockerignore` | List of files to exclude from Docker build |

---

*These notes were written while learning deployment hands-on. Every concept here was practiced with a real FastAPI project.*
