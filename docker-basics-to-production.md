# Docker – From Basics to Production

## Table of Contents
1. [What is Docker?](#what-is-docker)
2. [Why Use Docker?](#why-use-docker)
3. [Docker Architecture](#docker-architecture)
4. [Key Concepts](#key-concepts)
5. [Docker Installation and Setup](#docker-installation-and-setup)
6. [Dockerfile and Image Creation](#dockerfile-and-image-creation)
7. [Container Management](#container-management)
8. [Networking in Docker](#networking-in-docker)
9. [Docker Compose](#docker-compose)
10. [Debugging and Troubleshooting Containers](#debugging-and-troubleshooting-containers)
11. [Best Practices for Docker in Production](#best-practices-for-docker-in-production)
12. [Security Considerations](#security-considerations)
13. [Real-World Use Cases](#real-world-use-cases)
14. [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)
15. [Interview Questions](#interview-questions)
16. [Revision Notes](#revision-notes)

---

## 1. What is Docker?
Docker is an open-source platform that enables developers and operations teams to build, run, and share containerized applications. It standardizes the deployment environment, allowing consistent application behavior across all stages of development and production.

**Definition for Interviews:**
> Docker standardizes the development and deployment of applications by packaging them into lightweight, portable containers that include all dependencies.

**Key Characteristics:**
- **Containerization:** Isolates applications and their dependencies.
- **Lightweight:** Containers share the host OS kernel, reducing overhead.
- **Portable:** Run applications consistently across different environments.
- **Fast:** Faster than virtual machines, as there is no separate OS layer.

---

## 2. Why Use Docker?

Docker revolutionizes DevOps by enabling consistent application deployment and simplifying infrastructure management.

### Advantages:
- **Efficiency:**
  - Runs multiple containers on the same host.
  - Minimal resource consumption compared to virtual machines.
- **Consistency:**
  - Prevents “works on my machine” issues.
  - Identical environments for development, testing, and production.
- **Scalability:**
  - Easily scale containers horizontally.
  - Integrates with orchestration tools like Kubernetes.
- **Speed:**
  - Quick creation and teardown of containers.
  - Simplifies CI/CD pipelines.
- **Platform Agnostic:**
  - Works with Linux, Windows, and macOS.
- **Community Ecosystem:**
  - Hundreds of thousands of ready-to-use images in Docker Hub.

---

## 3. Docker Architecture

Docker uses a client-server model with key components interacting together:

```plaintext
+----------------------+   +-------------------------------+
|      Docker CLI      |   |          Docker Hub           |
+----------------------+   +-------------------------------+
        ↕                                ↕
+----------------------+   +-------------------------------+
|      Docker Daemon   | ← → |     Registry (Images)       |
+----------------------+   +-------------------------------+
      ↕
+-----------+   +-------------+
| Containers|   |  Images      |
+-----------+   +-------------+
```

### Key Components:
1. **Docker CLI:** User-facing command-line interface.
2. **Docker Daemon:** Background service managing containers.
3. **Images:** Pre-packaged application environments.
4. **Containers:** Running instances of Docker images.
5. **Docker Registry:** Stores and shares Docker images (e.g., Docker Hub).
6. **Docker Network and Volumes:** Handle container communication and persistent data.

---

## 4. Key Concepts

### Docker Image
- **Definition:** Immutable, lightweight template used to build containers.
- **Format:** Layers stacked on top of each other.
- **Example:** Official Nginx image (`nginx:latest`).

### Docker Container
- **Definition:** A running instance of an image.
- **Lifecycle: Start, Stop, Restart, Remove.**

### Dockerfile
- **Definition:** Script containing instructions to build an image.
- **Key Commands:** 
  - `FROM`: Base image.
  - `COPY`: Copy files into image.
  - `RUN`: Commands executing during build.
  - `CMD` & `ENTRYPOINT`: Define container's default process.

Example Dockerfile:
```Dockerfile
# Start with Node.js base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy application code
COPY package.json .
RUN npm install
COPY . .

# Expose port and start app
EXPOSE 3000
CMD ["node", "app.js"]
```

---

## 5. Docker Installation and Setup

### Installing Docker on Linux (Ubuntu):

1. **Update packages:**
   ```bash
   sudo apt update && sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```

2. **Add Docker’s official GPG key:**
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

3. **Add the Docker repository:**
   ```bash
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   ```

4. **Install Docker Engine:**
   ```bash
   sudo apt update && sudo apt install docker-ce -y
   ```

5. **Configure Docker Commands for non-root User (Optional):**
   ```bash
   sudo usermod -aG docker $USER  # Add current user to the Docker group
   ```

### Verify:
```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo docker --version  # See Docker version
```

### Additional Tools:
1. **Docker Compose:** Install using:
   ```bash
   sudo apt install docker-compose
   ```
2. **Test:** Create and deploy a one-container example.
   ```bash
docker run -d --name nginx-basic -p 80:80 nginx
```
   
---

Additional sections follow similar patterns (truncated for brevity), ensuring consistency with the table of contents.