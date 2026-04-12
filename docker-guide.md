# 🐳 Docker — Complete Guide

---

## Table of Contents
1. [What is Docker?](#1-what-is-docker)
2. [Why Docker?](#2-why-docker)
3. [Core Concepts](#3-core-concepts)
4. [Docker Architecture](#4-docker-architecture)
5. [Installing Docker](#5-installing-docker)
6. [Docker Images](#6-docker-images)
7. [Docker Containers](#7-docker-containers)
8. [Dockerfile](#8-dockerfile)
9. [Docker Volumes](#9-docker-volumes)
10. [Docker Networking](#10-docker-networking)
11. [Docker Compose](#11-docker-compose)
12. [Docker Registry](#12-docker-registry)
13. [Docker vs VM](#13-docker-vs-vm)
14. [Common Commands Cheat Sheet](#14-common-commands-cheat-sheet)
15. [Best Practices](#15-best-practices)

---

## 1. What is Docker?

**Docker** is an open-source platform that enables developers to **build, ship, and run** applications inside lightweight, portable containers.

A **container** packages your application code together with all its dependencies (libraries, runtime, config files) so it runs **consistently across any environment** — development, staging, or production.

> Docker was released in 2013 by Docker Inc. and is now a standard tool in modern DevOps and cloud-native development.

---

## 2. Why Docker?

| Problem without Docker | Solution with Docker |
|---|---|
| "It works on my machine" | Same container runs anywhere |
| Dependency conflicts between apps | Each container has isolated dependencies |
| Slow VM startup (minutes) | Containers start in seconds |
| Inconsistent dev/prod environments | Single image used for all environments |
| Complex manual deployment | Automate with Dockerfile + CI/CD |

---

## 3. Core Concepts

### 🔷 Image
- A **read-only template** used to create containers.
- Contains OS layer, runtime, app code, and dependencies.
- Built from a `Dockerfile`.
- Immutable — once built, it doesn't change.

### 🔷 Container
- A **running instance** of an image.
- Isolated from the host and other containers.
- Has its own filesystem, network, and process space.
- Stateless by default (data lost on removal unless volumes are used).

### 🔷 Dockerfile
- A **text file with instructions** to build a Docker image.
- Each instruction creates a new layer in the image.

### 🔷 Registry
- A **storage and distribution system** for Docker images.
- **Docker Hub** is the default public registry.
- Can be private (e.g., AWS ECR, GitHub Container Registry, Harbor).

### 🔷 Volume
- A **persistent storage mechanism** for containers.
- Data in volumes survives container restarts and removals.

### 🔷 Network
- Docker creates virtual networks that allow containers to **communicate** with each other or with the host.

---

## 4. Docker Architecture

```
┌──────────────────────────────────────────┐
│               Docker Client              │  ← CLI: `docker build`, `docker run`
└────────────────────┬─────────────────────┘
                     │ REST API
┌────────────────────▼─────────────────────┐
│              Docker Daemon (dockerd)      │  ← Manages images, containers, networks, volumes
└──────┬────────────────────────┬──────────┘
       │                        │
┌──────▼──────┐          ┌──────▼──────┐
│  containerd │          │   Registry  │  ← Docker Hub or private registry
│  (runtime)  │          │  (images)   │
└──────┬──────┘          └─────────────┘
       │
┌──────▼──────┐
│   runc      │  ← Low-level container runtime (OCI-compliant)
└─────────────┘
```

**Key components:**
- **Docker Client** — The CLI tool you use to interact with Docker.
- **Docker Daemon (`dockerd`)** — The background service that manages Docker objects.
- **containerd** — The container runtime that manages the container lifecycle.
- **runc** — The low-level tool that creates and runs containers according to OCI specs.

---

## 5. Installing Docker

### On Ubuntu/Debian
```bash
# Update package index
sudo apt update

# Install required packages
sudo apt install -y ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add Docker repo
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify installation
docker --version
docker run hello-world
```

### Post-install (run Docker without sudo)
```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## 6. Docker Images

### Pulling an image from Docker Hub
```bash
docker pull nginx              # latest tag
docker pull nginx:1.25         # specific version
docker pull ubuntu:22.04
```

### Listing images
```bash
docker images
docker image ls
```

### Inspecting an image
```bash
docker inspect nginx
docker image history nginx     # view layers
```

### Removing an image
```bash
docker rmi nginx
docker image rm nginx:1.25
docker image prune             # remove dangling (unused) images
docker image prune -a          # remove all unused images
```

### Tagging an image
```bash
docker tag nginx:latest myrepo/nginx:v1
```

### Image Layers
Every instruction in a Dockerfile creates a **layer**. Layers are cached and reused, making builds faster.

```
Layer 5: COPY ./app /app       ← your code
Layer 4: RUN npm install
Layer 3: WORKDIR /app
Layer 2: FROM node:18           ← base image layers
Layer 1: (OS base)
```

---

## 7. Docker Containers

### Running a container
```bash
# Basic run (exits after command finishes)
docker run ubuntu echo "Hello from container"

# Interactive terminal
docker run -it ubuntu bash

# Detached mode (background)
docker run -d nginx

# Named container
docker run -d --name my-nginx nginx

# Port mapping (host:container)
docker run -d -p 8080:80 nginx

# Environment variables
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Auto-remove on exit
docker run --rm ubuntu echo "I will be removed"
```

### Managing containers
```bash
docker ps                      # list running containers
docker ps -a                   # list all containers (including stopped)
docker stop <container>        # graceful stop (SIGTERM)
docker kill <container>        # force stop (SIGKILL)
docker start <container>       # start a stopped container
docker restart <container>     # restart
docker rm <container>          # remove stopped container
docker rm -f <container>       # force remove running container
docker container prune         # remove all stopped containers
```

### Interacting with containers
```bash
docker exec -it <container> bash          # open shell inside running container
docker exec <container> ls /app           # run single command
docker logs <container>                   # view logs
docker logs -f <container>               # follow logs (live tail)
docker logs --tail 100 <container>       # last 100 lines
docker top <container>                   # view running processes
docker stats                             # live resource usage
docker inspect <container>               # full configuration detail
docker cp <container>:/path/file ./      # copy file from container
docker cp ./file <container>:/path/      # copy file to container
```

### Container Lifecycle
```
Created → Running → Paused → Running → Stopped → Removed
           ↑                              ↓
        docker start              docker stop / kill
```

---

## 8. Dockerfile

A `Dockerfile` is a script of instructions to build a Docker image. Each instruction is a separate **layer**.

### Common Instructions

| Instruction | Description |
|---|---|
| `FROM` | Base image to start from |
| `RUN` | Execute command during **build** |
| `CMD` | Default command when container **starts** (overridable) |
| `ENTRYPOINT` | Main command that always runs (not easily overridden) |
| `COPY` | Copy files from host to image |
| `ADD` | Like COPY but also supports URLs and auto-extracts tarballs |
| `WORKDIR` | Set working directory inside container |
| `ENV` | Set environment variables |
| `ARG` | Build-time variables (not in final image) |
| `EXPOSE` | Document which port the container uses |
| `VOLUME` | Define mount points for volumes |
| `USER` | Set the user to run subsequent commands |
| `LABEL` | Add metadata to image |
| `HEALTHCHECK` | Define a health check command |

### Example 1 — Node.js App
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

### Example 2 — Python App
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Example 3 — Nginx with custom config
```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY ./dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Multi-Stage Build
Multi-stage builds allow you to **reduce final image size** by separating build tools from the runtime.

```dockerfile
# Build stage (Go app)
FROM golang:1.21 AS builder
WORKDIR /src
COPY . .
RUN go build -o /app/server .

# Final stage — minimal image
FROM alpine:3.18
COPY --from=builder /app/server /usr/local/bin/server
EXPOSE 8080
ENTRYPOINT ["server"]
```

### Building an image
```bash
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f path/to/Dockerfile .
docker build --no-cache -t myapp:1.0 .
docker build --build-arg NODE_ENV=production -t myapp .
```

### CMD vs ENTRYPOINT

| | CMD | ENTRYPOINT |
|---|---|---|
| **Purpose** | Default arguments | Fixed executable |
| **Override with `docker run`** | Yes (easily) | Only with `--entrypoint` |
| **Best for** | Providing defaults | Defining main process |

```dockerfile
# Using both together
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]     # default arg, can be overridden
```

---

## 9. Docker Volumes

Volumes provide **persistent data storage** that lives outside the container filesystem.

### Types of storage

| Type | Description | Use Case |
|---|---|---|
| **Volume** | Managed by Docker in `/var/lib/docker/volumes/` | Production data, databases |
| **Bind Mount** | Maps a host directory directly | Development (live reload) |
| **tmpfs** | In-memory, not persisted | Sensitive/temp data |

### Volume commands
```bash
docker volume create my-data
docker volume ls
docker volume inspect my-data
docker volume rm my-data
docker volume prune              # remove unused volumes
```

### Using volumes
```bash
# Named volume
docker run -d -v my-data:/var/lib/mysql mysql

# Bind mount (absolute path required)
docker run -d -v /host/path:/container/path nginx
docker run -d -v $(pwd):/app node:18        # current directory

# Read-only mount
docker run -d -v my-data:/data:ro nginx

# tmpfs
docker run -d --tmpfs /tmp nginx
```

### Volume in Dockerfile
```dockerfile
VOLUME ["/data"]   # declares a mount point, but doesn't create a named volume
```

---

## 10. Docker Networking

Docker provides several network drivers to control how containers communicate.

### Network Drivers

| Driver | Description | Use Case |
|---|---|---|
| `bridge` | Default. Creates a private network on the host | Single host, multi-container apps |
| `host` | Container shares host network stack | High-performance, Linux only |
| `none` | No networking | Completely isolated container |
| `overlay` | Multi-host networking via Swarm/overlay | Docker Swarm clusters |
| `macvlan` | Assigns MAC address; appears as physical device | Legacy apps, specific network needs |

### Network commands
```bash
docker network ls
docker network create my-net
docker network create --driver bridge --subnet 172.20.0.0/16 my-net
docker network inspect my-net
docker network rm my-net
docker network prune
```

### Connecting containers
```bash
# Connect a container to a network at creation
docker run -d --name web --network my-net nginx

# Connect a running container to a network
docker network connect my-net my-container

# Disconnect
docker network disconnect my-net my-container
```

### Container DNS
Containers on the same custom network can **communicate by name**:
```bash
docker network create app-net
docker run -d --name db --network app-net postgres
docker run -d --name backend --network app-net myapp
# backend container can reach db via hostname "db"
```

### Port Publishing
```bash
docker run -p 8080:80 nginx        # host:container
docker run -p 127.0.0.1:8080:80   # bind to loopback only
docker run -P nginx                # publish all EXPOSE'd ports to random host ports
```

---

## 11. Docker Compose

**Docker Compose** is a tool for defining and running **multi-container applications** using a YAML file.

### `docker-compose.yml` example

```yaml
version: "3.9"

services:
  # Web application
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - NODE_ENV=production
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-net
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped

  # Database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-net

  # Cache
  cache:
    image: redis:7-alpine
    command: redis-server --requirepass secret
    ports:
      - "6379:6379"
    networks:
      - app-net

volumes:
  pgdata:

networks:
  app-net:
    driver: bridge
```

### Docker Compose commands
```bash
docker compose up                   # start all services
docker compose up -d                # start in detached mode
docker compose up --build           # rebuild images before starting
docker compose down                 # stop and remove containers
docker compose down -v              # also remove volumes
docker compose ps                   # list services
docker compose logs                 # view all logs
docker compose logs -f web          # follow logs for a service
docker compose exec web bash        # exec into a service
docker compose stop                 # stop services (don't remove)
docker compose start                # start stopped services
docker compose restart web          # restart a service
docker compose pull                 # pull latest images
docker compose build                # build images only
docker compose config               # validate and view config
```

### Environment files
```bash
# .env file (auto-loaded by docker compose)
DATABASE_URL=postgres://localhost:5432/mydb
SECRET_KEY=mysecretkey

# docker-compose.yml
services:
  web:
    env_file:
      - .env
```

---

## 12. Docker Registry

### Docker Hub
```bash
# Login
docker login
docker login -u username -p password

# Push image
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0

# Pull image
docker pull username/myapp:1.0

# Logout
docker logout
```

### Private Registry
```bash
# Run a local registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag and push to local registry
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0

# Pull from local registry
docker pull localhost:5000/myapp:1.0
```

### Other Registries
| Registry | URL |
|---|---|
| Docker Hub | `hub.docker.com` |
| GitHub Container Registry | `ghcr.io` |
| AWS ECR | `<account>.dkr.ecr.<region>.amazonaws.com` |
| Google Artifact Registry | `<region>-docker.pkg.dev` |
| Azure Container Registry | `<name>.azurecr.io` |

---

## 13. Docker vs VM

```
┌─────────────────────────────────┐   ┌─────────────────────────────────┐
│         VIRTUAL MACHINE         │   │            DOCKER                │
├────────────┬────────────────────┤   ├────────────┬────────────────────┤
│   App A    │   App B            │   │  Container │  Container         │
│ (bins/libs)│ (bins/libs)        │   │    App A   │    App B           │
├────────────┴────────────────────┤   │ (bins/libs)│ (bins/libs)        │
│       Guest OS (full)           │   ├────────────┴────────────────────┤
├─────────────────────────────────┤   │         Docker Engine           │
│           Hypervisor            │   ├─────────────────────────────────┤
├─────────────────────────────────┤   │         Host OS Kernel          │
│           Host OS               │   ├─────────────────────────────────┤
├─────────────────────────────────┤   │           Hardware              │
│           Hardware              │   └─────────────────────────────────┘
└─────────────────────────────────┘
```

| Feature | Docker Container | Virtual Machine |
|---|---|---|
| **Startup time** | Seconds | Minutes |
| **Size** | MBs | GBs |
| **OS overhead** | Shares host kernel | Full OS per VM |
| **Isolation** | Process-level | Hardware-level |
| **Security** | Good (namespace/cgroups) | Stronger |
| **Portability** | Excellent | Good |
| **Performance** | Near-native | Slight overhead |
| **Use case** | Microservices, CI/CD | Full OS isolation, security |

---

## 14. Common Commands Cheat Sheet

```bash
# ── Images ──────────────────────────────────────
docker pull <image>                  # download image
docker build -t <name>:<tag> .       # build from Dockerfile
docker images                        # list images
docker rmi <image>                   # remove image
docker image prune -a               # remove unused images

# ── Containers ──────────────────────────────────
docker run -d -p 8080:80 --name web nginx     # run detached
docker run -it ubuntu bash                     # interactive shell
docker run --rm alpine echo hello              # auto-remove
docker ps / docker ps -a                       # list containers
docker stop / start / restart <container>
docker rm <container>                          # remove container
docker rm -f <container>                       # force remove
docker exec -it <container> bash               # shell into container
docker logs -f <container>                     # follow logs
docker inspect <container>                     # details

# ── Volumes ─────────────────────────────────────
docker volume create <name>
docker volume ls
docker volume rm <name>
docker volume prune

# ── Networks ────────────────────────────────────
docker network create <name>
docker network ls
docker network inspect <name>
docker network rm <name>

# ── Registry ────────────────────────────────────
docker login / docker logout
docker pull / docker push <image>
docker tag <src> <dst>

# ── Compose ─────────────────────────────────────
docker compose up -d
docker compose down
docker compose logs -f
docker compose exec <service> bash
docker compose ps

# ── System ──────────────────────────────────────
docker info                          # system info
docker version                       # client and daemon versions
docker system df                     # disk usage
docker system prune                  # remove all unused resources
docker system prune -a --volumes     # aggressive cleanup
docker stats                         # live resource usage
```

---

## 15. Best Practices

### Dockerfile Best Practices

1. **Use official, minimal base images**
   ```dockerfile
   FROM node:18-alpine   # ✅ alpine is smaller
   # avoid: FROM ubuntu  # ❌ too large
   ```

2. **Order instructions from least to most frequently changing**
   ```dockerfile
   COPY package.json .    # rarely changes → cached
   RUN npm install
   COPY . .               # often changes → invalidates cache
   ```

3. **Combine RUN commands to reduce layers**
   ```dockerfile
   # ✅ Good
   RUN apt-get update && apt-get install -y curl git && rm -rf /var/lib/apt/lists/*
   
   # ❌ Bad (3 layers instead of 1)
   RUN apt-get update
   RUN apt-get install -y curl
   RUN apt-get install -y git
   ```

4. **Use multi-stage builds** to reduce final image size

5. **Don't run as root**
   ```dockerfile
   RUN addgroup -S appgroup && adduser -S appuser -G appgroup
   USER appuser
   ```

6. **Use `.dockerignore`** to exclude unnecessary files
   ```
   # .dockerignore
   node_modules/
   .git/
   .env
   *.log
   dist/
   ```

7. **Pin image versions** for reproducibility
   ```dockerfile
   FROM node:18.17.1-alpine3.18   # ✅ pinned
   # avoid: FROM node:latest       # ❌ unpredictable
   ```

8. **Use HEALTHCHECK** for production images
   ```dockerfile
   HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
     CMD curl -f http://localhost:3000/health || exit 1
   ```

### Security Best Practices

- Never store secrets (passwords, tokens) in Dockerfile or image — use env vars or secret management tools
- Regularly scan images for vulnerabilities: `docker scout cves myimage`
- Use read-only filesystems where possible: `docker run --read-only`
- Limit container capabilities: `docker run --cap-drop ALL`
- Set memory/CPU limits: `docker run -m 512m --cpus 1 myapp`

### Production Best Practices

- Always use **restart policies**: `--restart unless-stopped`
- Use **named volumes** for persistent data, not bind mounts
- Implement **log rotation**: `docker run --log-opt max-size=10m --log-opt max-file=3`
- Use **Docker Compose** or orchestration tools (Kubernetes) for multi-service apps
- Monitor containers with `docker stats` or tools like Prometheus + cAdvisor

---

## Quick Reference: Key Concepts Summary

```
Image      → Blueprint (read-only template)
Container  → Running instance of an image
Dockerfile → Recipe to build an image
Volume     → Persistent data storage
Network    → Communication between containers
Registry   → Remote storage for images (Docker Hub)
Compose    → Tool to run multi-container apps (docker-compose.yml)
```

---

*Last updated: April 2026 | Docker version: 24.x+*
