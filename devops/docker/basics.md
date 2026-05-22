# Docker Basics

Docker is a platform for developing, shipping, and running applications in **containers** — lightweight, portable, and self-sufficient environments.

## Why Docker?

- **Consistency** — Works the same in dev, staging, and production
- **Isolation** — Each container runs independently
- **Speed** — Containers start in seconds, not minutes
- **Efficiency** — Share the host OS kernel, less overhead than VMs

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Image** | A read-only template with instructions for creating a container |
| **Container** | A runnable instance of an image |
| **Dockerfile** | A text file with instructions to build an image |
| **Registry** | A storage and distribution system for images (e.g., Docker Hub) |

## Essential Commands

```bash
# Pull an image from Docker Hub
docker pull nginx:latest

# Run a container
docker run -d --name my-nginx -p 8080:80 nginx:latest

# List running containers
docker ps

# View container logs
docker logs my-nginx

# Stop and remove a container
docker stop my-nginx
docker rm my-nginx

# Build an image from a Dockerfile
docker build -t my-app:1.0 .
```

## Dockerfile Example

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

> **Pro Tip:** Always use specific image tags (e.g., `node:18-alpine`) instead of `latest` in production to ensure reproducible builds.
