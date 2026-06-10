# Docker: Containerization Fundamentals

## 1. Core Architecture

Docker operates on a client-server architecture:
1. **Docker Client**: The CLI utility (`docker`) used by developers to enter commands.
2. **Docker Daemon (`dockerd`)**: A background service running on the host that listens for API requests and manages Docker objects (containers, images, networks, volumes).
3. **Docker Registry**: A service that stores and distributes Docker images (e.g., **Docker Hub** is the default public registry).
4. **Docker Image**: A read-only blueprint containing the filesystem and setup instructions for your application.
5. **Docker Container**: A running, writeable instance of an image.

---

## 2. Essential Command Reference

### Image Operations
- **Pull an image from Docker Hub**:
  ```bash
  docker pull nginx:alpine
  ```
- **List downloaded images**:
  ```bash
  docker images
  ```
- **Delete an image from local storage**:
  ```bash
  docker rmi nginx:alpine
  ```

### Container Lifecycle
- **Run a container in detached mode (background)**:
  ```bash
  docker run -d --name my-web -p 8080:80 nginx:alpine
  ```
  *`-d` runs it in background, `--name` sets a friendly name, and `-p 8080:80` forwards port 8080 on the host to port 80 in the container.*
- **Run a container interactively (good for debugging)**:
  ```bash
  docker run -it --name debug-box ubuntu:latest /bin/bash
  ```
- **Stop a running container**:
  ```bash
  docker stop my-web
  ```
- **Start a stopped container**:
  ```bash
  docker start my-web
  ```
- **Remove a container (must be stopped first)**:
  ```bash
  docker rm my-web
  ```
- **Force delete a running container**:
  ```bash
  docker rm -f my-web
  ```

### Inspection & Monitoring
- **List running containers**:
  ```bash
  docker ps
  ```
- **List all containers (running & stopped)**:
  ```bash
  docker ps -a
  ```
- **View container console logs**:
  ```bash
  docker logs -f my-web
  ```
- **Execute a command inside a running container**:
  ```bash
  docker exec -it my-web /bin/sh
  ```
- **Inspect container configuration and IP addresses**:
  ```bash
  docker inspect my-web
  ```

---

## 3. Data Persistence: Volumes and Bind Mounts

By default, data created inside a container is ephemeral—it is destroyed when the container is deleted. To persist data, Docker provides two main options:

```text
    [Host Filesystem]                    [Container]
 ┌───────────────────────┐            ┌───────────────┐
 │ /var/lib/docker/...   │ ─────────> │ /app/data     │  (Named Volume: Docker managed)
 ├───────────────────────┤            └───────────────┘
 │ /home/user/project    │ ─────────> │ /app/src      │  (Bind Mount: Developer managed)
 └───────────────────────┘            └───────────────┘
```

### 1. Named Volumes (Recommended)
Docker manages where these files are stored on the host (usually `/var/lib/docker/volumes/`). Recommended for database storage.
- **Create a volume**:
  ```bash
  docker volume create pg_data
  ```
- **Run a container with a volume**:
  ```bash
  docker run -d --name db -v pg_data:/var/lib/postgresql/data postgres:15
  ```

### 2. Bind Mounts
Maps a specific directory on your host machine to a directory inside the container. Best for live-reloading code during development.
- **Run container with a bind mount**:
  ```bash
  docker run -d -p 3000:3000 -v $(pwd):/app node:18-alpine
  ```

---

## 4. Docker Networking

Docker networks allow containers to communicate with each other and the outside world.

### Default Network Drivers
- **Bridge (Default)**: Creates a private virtual network. Containers on the same bridge can talk to each other, but are isolated from others.
- **Host**: Removes isolation between the container and the host. The container shares the host's networking ports directly.
- **None**: Disables all networking for the container.

### Communicating between Containers
To connect containers, create a custom user-defined bridge network:
1. **Create network**:
   ```bash
   docker network create app-net
   ```
2. **Launch containers inside the network**:
   ```bash
   docker run -d --name database --network app-net postgres:15
   ```
   ```bash
   docker run -d --name backend-api --network app-net -p 8080:8080 my-api-image
   ```
3. **DNS Resolution**: Containers inside `app-net` can now talk to each other using their container names as hostnames. For example, `backend-api` can connect to PostgreSQL using `host: database` instead of an IP address.

---

## 5. Building Images: The Dockerfile

A `Dockerfile` is a text document containing instructions to assemble a Docker image.

### Example Dockerfile (Python Flask App)
```dockerfile
# 1. Base Image
FROM python:3.8-slim

# 2. Set working directory in container
WORKDIR /app

# 3. Copy files from host to container
COPY requirements.txt /app/

# 4. Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy the rest of the application code
COPY . /app/

# 6. Expose the port the app runs on
EXPOSE 5000

# 7. Default command execution
CMD ["python", "app.py"]
```

### Steps to Build and Run:
1. **Build the image**:
   ```bash
   docker build -t my-flask-app:1.0 .
   ```
   *`-t` tags the image with a name and version. The `.` specifies the build context (current folder).*
2. **Run the built image**:
   ```bash
   docker run -d --name flask-web -p 5000:5000 my-flask-app:1.0
   ```