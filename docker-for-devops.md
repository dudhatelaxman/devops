# Comprehensive Docker Guide for DevOps Engineers

**Last Updated:** 2025-12-29  
**Author:** DevOps Team  
**Version:** 1.0

---

## Table of Contents

1. [Introduction to Docker](#1-introduction-to-docker)
2. [Docker Architecture](#2-docker-architecture)
3. [Installation and Setup](#3-installation-and-setup)
4. [Docker Images](#4-docker-images)
5. [Docker Containers](#5-docker-containers)
6. [Dockerfile and Image Building](#6-dockerfile-and-image-building)
7. [Docker Networking](#7-docker-networking)
8. [Docker Volumes and Storage](#8-docker-volumes-and-storage)
9. [Docker Compose](#9-docker-compose)
10. [Docker Registry and Repository Management](#10-docker-registry-and-repository-management)
11. [Container Orchestration Basics](#11-container-orchestration-basics)
12. [Security and Best Practices](#12-security-and-best-practices)
13. [Monitoring and Logging](#13-monitoring-and-logging)
14. [Troubleshooting and Debugging](#14-troubleshooting-and-debugging)
15. [Advanced Docker Features](#15-advanced-docker-features)
16. [Docker in Production](#16-docker-in-production)

---

## 1. Introduction to Docker

### What is Docker?

Docker is a containerization platform that packages applications and their dependencies into standardized units called containers. It enables developers and DevOps engineers to build, deploy, and run applications consistently across different environments.

### Key Benefits

- **Consistency**: "Works on my machine" problem solved
- **Scalability**: Easy to scale applications horizontally
- **Isolation**: Containers are isolated from each other
- **Efficiency**: Lightweight compared to virtual machines
- **Portability**: Run anywhere Docker is installed

### Docker vs Virtual Machines

| Aspect | Docker | VM |
|--------|--------|-----|
| Overhead | Minimal (MB) | Significant (GB) |
| Startup Time | Seconds | Minutes |
| Isolation | Process-level | OS-level |
| Performance | Near-native | Slight overhead |
| Density | High | Lower |

### Use Cases

- Microservices deployment
- CI/CD pipelines
- Development environment standardization
- Legacy application modernization
- Multi-tenant applications

---

## 2. Docker Architecture

### Docker Components

#### Docker Engine
The core runtime that manages containers. Consists of:
- **Docker Daemon (dockerd)**: Background service managing Docker objects
- **Docker CLI**: Command-line interface for user interaction
- **Docker API**: RESTful API for programmatic access

#### Client-Server Architecture

```
Docker Client ──→ Docker Daemon ──→ Containers
   (CLI)            (Server)
```

### Docker Objects

#### Images
- Read-only templates for creating containers
- Built from Dockerfile instructions
- Stored in registries (Docker Hub, private registries)

#### Containers
- Running instances of images
- Isolated from host and other containers
- Can be started, stopped, removed

#### Networks
- Enable communication between containers
- Types: bridge, host, overlay, macvlan

#### Volumes
- Persistent data storage mechanism
- Decoupled from container lifecycle
- Can be shared between containers

### Docker Namespace and Cgroups

- **Namespaces**: Provide process isolation (PID, Network, IPC, Mount, User, UTS)
- **Cgroups**: Control resource limits (CPU, Memory, I/O)

---

## 3. Installation and Setup

### Installation on Linux (Ubuntu/Debian)

```bash
# Update package manager
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.x.x/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Enable Docker service
sudo systemctl enable docker
sudo systemctl start docker

# Add user to docker group (optional, for non-root access)
sudo usermod -aG docker $USER
newgrp docker
```

### Installation on macOS

```bash
# Using Homebrew
brew install docker docker-compose

# Or download Docker Desktop from official website
# https://www.docker.com/products/docker-desktop
```

### Installation on Windows

- Download Docker Desktop for Windows
- Enable WSL 2 (Windows Subsystem for Linux 2)
- Install from official Docker website

### Verify Installation

```bash
docker --version
docker run hello-world
docker-compose --version
```

### Docker Daemon Configuration

Edit `/etc/docker/daemon.json`:

```json
{
  "debug": false,
  "storage-driver": "overlay2",
  "live-restore": true,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

---

## 4. Docker Images

### Understanding Images

Docker images are built-up in layers, with each layer representing an instruction from a Dockerfile.

### Common Docker Commands for Images

```bash
# Search images
docker search <image-name>

# Pull image from registry
docker pull <image-name>:<tag>

# List local images
docker images
docker image ls

# Inspect image details
docker inspect <image-id>
docker image inspect <image-name>

# Remove image
docker rmi <image-id>
docker image remove <image-name>

# Tag image
docker tag <source-image> <new-image>:<tag>

# Save image to tar
docker save <image-name> > image.tar

# Load image from tar
docker load < image.tar

# Get image history
docker history <image-name>

# Push image to registry
docker push <registry>/<image-name>:<tag>
```

### Image Naming Convention

```
[REGISTRY_HOST[:REGISTRY_PORT]/]NAME[:TAG]

Example:
docker.io/library/ubuntu:22.04
registry.example.com:5000/myapp:v1.0.0
```

### Image Layers and Caching

Docker images are composed of multiple read-only layers. Each instruction in Dockerfile creates a new layer.

```
┌─────────────────────────────────────┐
│  Layer 5: Application code          │ (Latest)
├─────────────────────────────────────┤
│  Layer 4: Dependencies installation │
├─────────────────────────────────────┤
│  Layer 3: Environment setup         │
├─────────────────────────────────────┤
│  Layer 2: OS packages               │
├─────────────────────────────────────┤
│  Layer 1: Base image (Ubuntu)       │
└─────────────────────────────────────┘
```

---

## 5. Docker Containers

### Container Lifecycle

```
Created → Running → Paused ↕
  ↓         ↓
Stopped → Restarted
  ↓
Removed
```

### Essential Container Commands

```bash
# Create and run container
docker run [OPTIONS] IMAGE [COMMAND]
docker create <image-name>
docker start <container-id>

# Common run options
docker run -d \
  --name mycontainer \
  -p 8080:80 \
  -v /host/path:/container/path \
  -e VARIABLE=value \
  --memory="512m" \
  --cpus="1.0" \
  image-name

# View running containers
docker ps
docker ps -a

# Container information
docker inspect <container-id>
docker stats <container-id>
docker logs <container-id>
docker logs -f <container-id>  # Follow logs

# Execute commands in running container
docker exec -it <container-id> /bin/bash

# Stop and remove containers
docker stop <container-id>
docker kill <container-id>
docker rm <container-id>
docker container prune  # Remove all stopped containers

# Pause and unpause
docker pause <container-id>
docker unpause <container-id>

# Copy files
docker cp <container-id>:/path/in/container ./local/path
docker cp ./local/path <container-id>:/path/in/container

# Commit container to image
docker commit <container-id> <new-image-name>
```

### Resource Constraints

```bash
# Memory limit
docker run --memory="512m" --memory-swap="1g" image-name

# CPU limit
docker run --cpus="1.5" image-name

# I/O constraints
docker run --blkio-weight=300 image-name
```

---

## 6. Dockerfile and Image Building

### Dockerfile Basics

A Dockerfile is a text file containing instructions to build a Docker image.

### Dockerfile Instructions

```dockerfile
# Base image
FROM ubuntu:22.04

# Metadata
LABEL maintainer="devops@example.com"
LABEL version="1.0"
LABEL description="Example application"

# Set working directory
WORKDIR /app

# Copy files from host to container
COPY . /app

# Add files (supports URLs)
ADD https://example.com/file.tar.gz /app/

# Install dependencies
RUN apt-get update && \
    apt-get install -y \
    python3 \
    pip && \
    rm -rf /var/lib/apt/lists/*

# Set environment variables
ENV NODE_ENV=production
ENV APP_PORT=3000

# Expose ports
EXPOSE 3000 8080

# Create volumes
VOLUME ["/data", "/logs"]

# Create non-root user
RUN useradd -m appuser
USER appuser

# Set default command
CMD ["python3", "app.py"]

# Entrypoint (overrides CMD)
ENTRYPOINT ["./start.sh"]
```

### Dockerfile Best Practices

```dockerfile
# ✅ GOOD PRACTICES

# 1. Use specific base image tags
FROM python:3.11-slim

# 2. Minimize layers - combine RUN commands
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 3. Use .dockerignore
# .dockerignore should include: .git, node_modules, .env, etc.

# 4. Run as non-root user
RUN useradd -m appuser
USER appuser

# 5. Order instructions by change frequency
FROM base-image
RUN install-dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# 6. Use COPY instead of ADD when possible
COPY . /app

# 7. Clean up in same layer
RUN apt-get update && apt-get install -y package && apt-get clean

# ❌ BAD PRACTICES

# Don't do this
FROM ubuntu:latest          # Use specific version
RUN apt-get install *       # Install everything
RUN ... (multiple RUN)      # Creates many layers
COPY . /app
RUN some-command
COPY more-files /app
RUN another-command
```

### Building Images

```bash
# Basic build
docker build -t myapp:1.0 .

# Build with multiple tags
docker build -t myapp:1.0 -t myapp:latest .

# Build with build arguments
docker build \
  --build-arg NODE_ENV=production \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t myapp:1.0 .

# Build from different Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build without cache
docker build --no-cache -t myapp:1.0 .

# View build process
docker build --progress=plain -t myapp:1.0 .
```

### Multi-stage Builds

```dockerfile
# Stage 1: Build
FROM golang:1.19 as builder
WORKDIR /build
COPY . .
RUN go build -o app main.go

# Stage 2: Runtime
FROM alpine:3.17
RUN apk add --no-cache ca-certificates
COPY --from=builder /build/app /usr/local/bin/
ENTRYPOINT ["app"]
```

---

## 7. Docker Networking

### Network Types

#### Bridge Network (Default)
- Default network for containers
- Containers can communicate with each other
- Isolated from host network

```bash
# Create custom bridge network
docker network create --driver bridge mynetwork

# Run container on specific network
docker run --network mynetwork myimage
```

#### Host Network
- Container shares host's network stack
- High performance, reduced isolation

```bash
docker run --network host myimage
```

#### Overlay Network
- Used for Docker Swarm
- Enables communication across multiple Docker hosts

```bash
docker network create --driver overlay my-overlay-net
```

#### None Network
- Container has only loopback interface
- Used for isolation

```bash
docker run --network none myimage
```

### Network Commands

```bash
# List networks
docker network ls

# Inspect network
docker network inspect <network-name>

# Create network
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --ip-range 172.20.240.0/20 \
  mynetwork

# Connect container to network
docker network connect mynetwork <container-id>

# Disconnect container from network
docker network disconnect mynetwork <container-id>

# Remove network
docker network rm mynetwork
```

### DNS and Service Discovery

Docker provides automatic DNS resolution between containers on the same network:

```bash
# Container can reach other containers by service name
docker run --name db -d postgres
docker run --link db myapp  # Can reach 'db' hostname

# Better approach: use user-defined bridge network
docker network create mynet
docker run --name db --network mynet -d postgres
docker run --network mynet myapp  # Can reach 'db' hostname
```

### Port Mapping

```bash
# Map specific port
docker run -p 8080:80 myimage

# Map all exposed ports
docker run -P myimage

# Map to specific IP
docker run -p 127.0.0.1:8080:80 myimage

# Map port range
docker run -p 8000-9000:80 myimage

# View port mappings
docker port <container-id>
```

---

## 8. Docker Volumes and Storage

### Volume Types

#### Named Volumes
- Managed by Docker
- Persist data even after container removal

```bash
# Create named volume
docker volume create myvolume

# Use in container
docker run -v myvolume:/data myimage

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume
```

#### Bind Mounts
- Mount directories from host filesystem
- Direct access to host paths

```bash
# Bind mount
docker run -v /host/path:/container/path myimage

# Read-only bind mount
docker run -v /host/path:/container/path:ro myimage
```

#### tmpfs Mounts
- Store data in host's memory
- Ideal for temporary data

```bash
docker run --tmpfs /app/temp myimage
```

### Volume Management

```bash
# List volumes
docker volume ls

# Volume details
docker volume inspect myvolume

# Remove unused volumes
docker volume prune

# Remove specific volume
docker volume rm myvolume

# Backup volume
docker run -v myvolume:/data -v $(pwd):/backup \
  busybox tar czf /backup/volume.tar.gz -C /data .

# Restore volume
docker run -v myvolume:/data -v $(pwd):/backup \
  busybox tar xzf /backup/volume.tar.gz -C /data
```

### Storage Drivers

```bash
# View storage driver
docker info | grep "Storage Driver"

# Common drivers: overlay2, btrfs, zfs, devicemapper
```

### Volume Permissions

```dockerfile
FROM ubuntu:22.04

# Create directory with proper permissions
RUN mkdir -p /app/data && \
    chown -R 1000:1000 /app/data

USER 1000
VOLUME ["/app/data"]
```

---

## 9. Docker Compose

### docker-compose.yml Structure

```yaml
version: '3.9'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: web_container
    image: myapp:1.0
    ports:
      - "8080:80"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    volumes:
      - ./src:/app/src
      - app_logs:/app/logs
    depends_on:
      - db
    networks:
      - appnet
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:15
    container_name: db_container
    environment:
      - POSTGRES_USER=appuser
      - POSTGRES_PASSWORD=secure_password
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - appnet
    restart: unless-stopped

  cache:
    image: redis:7-alpine
    container_name: cache_container
    ports:
      - "6379:6379"
    networks:
      - appnet
    restart: unless-stopped

volumes:
  app_logs:
  postgres_data:

networks:
  appnet:
    driver: bridge
```

### Docker Compose Commands

```bash
# Start services
docker-compose up
docker-compose up -d  # Detached mode

# Stop services
docker-compose down

# Remove volumes with containers
docker-compose down -v

# View logs
docker-compose logs
docker-compose logs -f web  # Follow logs for specific service

# Execute command in service
docker-compose exec web bash

# Scale services
docker-compose up -d --scale web=3

# List services
docker-compose ps

# Build images
docker-compose build
docker-compose build --no-cache

# Push images
docker-compose push

# Pull images
docker-compose pull

# Remove stopped containers
docker-compose rm

# Restart services
docker-compose restart
docker-compose restart web  # Specific service

# Pause/unpause
docker-compose pause
docker-compose unpause

# View compose configuration
docker-compose config

# Validate compose file
docker-compose config --quiet
```

### Environment Variables

```yaml
# Using .env file
services:
  web:
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - API_KEY=${API_KEY}
```

Create `.env` file:
```
DATABASE_URL=postgres://user:pass@db:5432/mydb
API_KEY=secret_key_123
```

---

## 10. Docker Registry and Repository Management

### Docker Hub

```bash
# Login to Docker Hub
docker login

# Logout
docker logout

# Push image
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0

# Pull image
docker pull username/myapp:1.0
```

### Private Docker Registry

```bash
# Run private registry
docker run -d \
  --name registry \
  -p 5000:5000 \
  -v registry_data:/var/lib/registry \
  registry:2

# Tag and push to private registry
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0

# Pull from private registry
docker pull localhost:5000/myapp:1.0
```

### Using Other Registries

#### AWS ECR (Elastic Container Registry)

```bash
# Login to AWS ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Create repository
aws ecr create-repository --repository-name myapp

# Tag and push
docker tag myapp:1.0 \
  123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
docker push \
  123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
```

#### Google Container Registry (GCR)

```bash
# Configure authentication
gcloud auth configure-docker

# Tag and push
docker tag myapp:1.0 gcr.io/project-id/myapp:1.0
docker push gcr.io/project-id/myapp:1.0
```

#### Azure Container Registry (ACR)

```bash
# Login to ACR
az acr login --name myregistry

# Tag and push
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0
docker push myregistry.azurecr.io/myapp:1.0
```

### Image Signing and Verification

```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Push signed image
docker push myregistry/myapp:1.0

# Verify image signature
docker pull myregistry/myapp:1.0
```

---

## 11. Container Orchestration Basics

### What is Container Orchestration?

Container orchestration automates deployment, scaling, and management of containerized applications across clusters.

### Docker Swarm

#### Initialize Swarm

```bash
# Initialize manager node
docker swarm init

# Get join token for workers
docker swarm join-token worker

# Join as worker
docker swarm join --token SWMTKN-... <manager-ip>:2377

# View nodes
docker node ls
```

#### Deploy Services

```bash
# Create service
docker service create --name web -p 8080:80 myapp:1.0

# Service with replicas
docker service create \
  --name web \
  --replicas 3 \
  -p 8080:80 \
  myapp:1.0

# Scale service
docker service scale web=5

# View services
docker service ls
docker service ps web

# Update service
docker service update --image myapp:2.0 web

# Remove service
docker service rm web
```

### Kubernetes Basics

While Kubernetes is more complex than Docker Swarm, basic concepts:

```bash
# Deploy application
kubectl apply -f deployment.yaml

# View deployments
kubectl get deployments
kubectl get pods

# Scale application
kubectl scale deployment web --replicas=5

# Update deployment
kubectl set image deployment/web web=myapp:2.0
```

### Orchestration Comparison

| Feature | Docker Swarm | Kubernetes |
|---------|--------------|-----------|
| Complexity | Simple | Complex |
| Learning Curve | Easy | Steep |
| Scalability | Good | Excellent |
| Features | Basic | Advanced |
| Production Use | Small clusters | Enterprise |

---

## 12. Security and Best Practices

### Image Security

```dockerfile
# 1. Use minimal base images
FROM python:3.11-slim
# OR
FROM alpine:3.17

# 2. Run as non-root user
RUN useradd -m -u 1000 appuser
USER appuser

# 3. Use read-only filesystem
RUN mkdir -p /tmp /app && \
    chown appuser:appuser /tmp /app

# 4. Remove unnecessary packages
RUN apt-get update && \
    apt-get install -y only-needed-package && \
    apt-get remove -y apt-utils curl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 5. Set security headers
RUN echo "User-Agent: Forbidden" >> /etc/hosts
```

### Runtime Security

```bash
# Run with read-only filesystem
docker run --read-only myimage

# Disable privilege escalation
docker run --security-opt=no-new-privileges myimage

# Drop unnecessary capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myimage

# Limit resources
docker run \
  --memory="256m" \
  --cpus="0.5" \
  --pids-limit=100 \
  myimage

# Run as specific user
docker run --user 1000:1000 myimage
```

### Scanning for Vulnerabilities

```bash
# Docker Scout (built-in scanning)
docker scout cves myapp:1.0

# Trivy scanning
trivy image myapp:1.0

# Snyk scanning
snyk container test myapp:1.0
```

### Secrets Management

```dockerfile
# ❌ WRONG - Secrets hardcoded
FROM ubuntu
ENV DB_PASSWORD=secret123
```

```bash
# ✅ RIGHT - Use Docker secrets (Swarm)
echo "my_secret_password" | docker secret create db_password -

# ✅ RIGHT - Use environment files
docker run --env-file .env.prod myimage

# ✅ RIGHT - Use secrets manager
docker secret create my_secret /path/to/secret
```

### Network Security

```yaml
# docker-compose.yml with secure network
version: '3.9'

services:
  web:
    image: myapp:1.0
    networks:
      - web
    expose:
      - "8080"  # Don't publish to host

  db:
    image: postgres:15
    networks:
      - db
    # No web network access

networks:
  web:
    driver: bridge
  db:
    driver: bridge
    internal: true  # No external access
```

---

## 13. Monitoring and Logging

### Docker Events

```bash
# Monitor Docker events
docker events

# Filter events
docker events --filter type=container
docker events --filter event=start

# Real-time container stats
docker stats
docker stats --no-stream  # One-time output
```

### Container Logs

```bash
# View logs
docker logs <container-id>

# Follow logs (tail -f)
docker logs -f <container-id>

# Show last N lines
docker logs --tail 100 <container-id>

# Show logs with timestamp
docker logs -t <container-id>

# Show logs since specific time
docker logs --since 2024-01-15T10:00:00 <container-id>

# Save logs to file
docker logs <container-id> > container.log 2>&1
```

### Logging Drivers

Configure in `/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "service=app"
  }
}
```

Other drivers:
- syslog
- splunk
- awslogs
- gcplogs
- awsfirelens

### Monitoring Tools

#### Using Prometheus

```yaml
version: '3.9'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

volumes:
  prometheus_data:
```

#### Using ELK Stack

```yaml
version: '3.9'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.0.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"

  logstash:
    image: docker.elastic.co/logstash/logstash:8.0.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf

  kibana:
    image: docker.elastic.co/kibana/kibana:8.0.0
    ports:
      - "5601:5601"
```

### Health Checks

```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Or in docker-compose.yml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

---

## 14. Troubleshooting and Debugging

### Common Issues and Solutions

#### Issue: Container exits immediately

```bash
# Check logs
docker logs <container-id>

# Run with interactive terminal
docker run -it image-name bash

# Check exit code
docker inspect <container-id> | grep ExitCode
```

#### Issue: Cannot connect to container

```bash
# Check if container is running
docker ps -a

# Inspect network
docker inspect <container-id> | grep -A 10 NetworkSettings

# Check port mappings
docker port <container-id>

# Test network connectivity
docker exec <container-id> ping <target>
```

#### Issue: Out of disk space

```bash
# Check disk usage
docker system df

# Remove unused resources
docker system prune
docker system prune -a  # Remove all unused images

# Remove specific items
docker container prune
docker image prune
docker volume prune
docker network prune
```

### Debugging Tools

```bash
# Execute command in running container
docker exec -it <container-id> /bin/bash

# Get container details
docker inspect <container-id> | jq '.[0].Config'

# View processes in container
docker top <container-id>

# Check resource usage
docker stats <container-id>

# Export container filesystem
docker export <container-id> > container.tar

# Copy files for analysis
docker cp <container-id>:/app/logs ./logs
```

### Docker Compose Debugging

```bash
# View compose configuration
docker-compose config

# Validate compose file
docker-compose config --quiet

# View detailed logs
docker-compose logs --follow --timestamps web

# Inspect service
docker-compose exec web env

# Check service dependencies
docker-compose depends
```

---

## 15. Advanced Docker Features

### Docker Multi-stage Builds

```dockerfile
# Build stage
FROM node:18-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### BuildKit

Enable faster, more efficient builds:

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Build with BuildKit
docker build -t myapp:1.0 .

# Or enable globally in daemon.json
# {
#   "features": {
#     "buildkit": true
#   }
# }
```

### BuildKit Features

```dockerfile
# Cache mounts
FROM ubuntu:22.04

RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && \
    apt-get install -y curl

# Secrets
RUN --mount=type=secret,id=npm_token \
    npm config set //registry.npmjs.org/:_authToken=$(cat /run/secrets/npm_token)

# SSH
RUN --mount=type=ssh \
    git clone git@github.com:user/repo.git
```

### Docker Network Plugins

```bash
# Create custom network
docker network create \
  --driver bridge \
  --opt "com.docker.network.bridge.name"=br-custom \
  custom_network

# Use network driver options
docker network create \
  --driver bridge \
  --opt "com.docker.network.driver.mtu"=1200 \
  low_mtu_network
```

### User Namespaces

Enable in daemon.json for additional isolation:

```json
{
  "userns-remap": "appuser"
}
```

### Content Addressable Storage

Docker images are content-addressable using SHA256 digests:

```bash
# Reference by digest
docker pull myapp@sha256:abc123...

# Get image digest
docker inspect --format='{{index .RepoDigests 0}}' myapp:1.0
```

---

## 16. Docker in Production

### Production Deployment Strategies

#### Blue-Green Deployment

```yaml
version: '3.9'

services:
  # Blue environment
  app-blue:
    image: myapp:1.0
    ports:
      - "8080:8080"
    labels:
      - "environment=blue"

  # Green environment (new version)
  app-green:
    image: myapp:2.0
    ports:
      - "8081:8080"
    labels:
      - "environment=green"

  # Load balancer
  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
    depends_on:
      - app-blue
```

#### Canary Deployment

```yaml
version: '3.9'

services:
  # Stable version - 95%
  app-stable:
    image: myapp:1.0
    deploy:
      replicas: 95

  # New version - 5%
  app-canary:
    image: myapp:2.0
    deploy:
      replicas: 5

  # Load balancer distributes traffic
  nginx:
    image: nginx:latest
    volumes:
      - ./nginx-canary.conf:/etc/nginx/nginx.conf
```

#### Rolling Deployment

```bash
docker service update \
  --image myapp:2.0 \
  --update-parallelism 1 \
  --update-delay 10s \
  --update-failure-action pause \
  myapp_web
```

### Production Configuration

#### daemon.json

```json
{
  "debug": false,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "10"
  },
  "storage-driver": "overlay2",
  "live-restore": true,
  "userland-proxy": false,
  "icc": false,
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 65536,
      "Soft": 65536
    }
  },
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

### Resource Management

```yaml
version: '3.9'

services:
  web:
    image: myapp:1.0
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

### Backup and Disaster Recovery

```bash
# Backup Docker data
tar -czf docker-data-backup.tar.gz \
  /var/lib/docker \
  /etc/docker/daemon.json

# Backup named volumes
docker run --rm -v myvolume:/data -v $(pwd):/backup \
  alpine tar czf /backup/myvolume.tar.gz -C /data .

# Create snapshot
docker save myapp:1.0 | gzip > myapp-1.0.tar.gz

# Restore from snapshot
zcat myapp-1.0.tar.gz | docker load
```

### Production Checklist

- [ ] Use specific image tags (not latest)
- [ ] Implement health checks
- [ ] Configure resource limits
- [ ] Enable restart policies
- [ ] Setup logging and monitoring
- [ ] Implement secrets management
- [ ] Run security scans
- [ ] Document deployment process
- [ ] Setup CI/CD pipeline
- [ ] Plan for backup/recovery
- [ ] Configure reverse proxy
- [ ] Setup log aggregation
- [ ] Monitor container performance
- [ ] Plan capacity and scaling
- [ ] Document troubleshooting steps

### Container Registry Security

```bash
# Sign and verify images
export DOCKER_CONTENT_TRUST=1
docker push myregistry/myapp:1.0

# Scan for vulnerabilities
docker scan myapp:1.0

# Use image digest
docker run \
  myregistry/myapp@sha256:abc123...

# Implement image retention policies
# Configure registry garbage collection
# Setup image signing
# Implement access controls
```

### High Availability Setup

```yaml
version: '3.9'

services:
  web:
    image: myapp:1.0
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
      update_config:
        parallelism: 1
        delay: 10s

  db:
    image: postgres:15
    environment:
      POSTGRES_REPLICATION_MODE: master
    volumes:
      - db-data:/var/lib/postgresql/data
    deploy:
      placement:
        constraints:
          - node.role == manager

  db-replica:
    image: postgres:15
    environment:
      POSTGRES_REPLICATION_MODE: replica
    depends_on:
      - db

volumes:
  db-data:
```

---

## Additional Resources

### Official Documentation
- [Docker Official Documentation](https://docs.docker.com/)
- [Docker API Reference](https://docs.docker.com/engine/api/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)

### Learning Resources
- Docker Official GitHub Repository
- Docker YouTube Channel
- Community Forums and Stack Overflow

### Tools and Utilities
- Docker Desktop
- Docker CLI
- Docker Compose
- Docker Scout (Security Scanning)
- BuildKit
- Dive (Image Analysis)

---

## Conclusion

This comprehensive guide covers Docker from fundamentals to production deployment. Key takeaways:

1. **Understand Basics**: Images, containers, networks, and volumes
2. **Master Dockerfile**: Write efficient, secure Dockerfiles
3. **Use Docker Compose**: For multi-container applications
4. **Implement Security**: Run as non-root, scan for vulnerabilities
5. **Monitor and Log**: Track performance and troubleshoot issues
6. **Plan for Scale**: Use orchestration and implement HA
7. **Document Everything**: Maintain clear documentation and procedures

Docker is a powerful tool that, when used correctly, significantly improves development workflows and production deployments.

---

**Last Updated**: 2025-12-29  
**Version**: 1.0  
**Maintainer**: DevOps Team
