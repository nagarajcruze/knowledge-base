# Docker Interview Preparation Questions (IPQ)

### Q: What is Docker?
Docker is a containerization platform that allows developers to package an application with all of its dependencies into a single container. This container can be easily deployed and run on any host machine that supports Docker. This guarantees that applications run consistently across different development, testing, and production environments by isolating processes and using lightweight, portable container technology.

### Q: What command can you run to export a Docker image as an archive?
Use the `docker save` command to save an image to a `.tar` archive:
```bash
docker save -o <output_file_name>.tar <image_name>
```

### Q: What command can be run to import a pre-exported Docker image into another Docker host?
Use the `docker load` command to load an image from a `.tar` archive:
```bash
docker load -i <input_file_name>.tar
```

### Q: Can a paused container be removed from Docker?
No, a container must be stopped or forced to be removed. To remove a stopped container, run:
```bash
docker rm <container_id>
```
*(To force-remove a running or paused container, you can add the `-f` flag: `docker rm -f <container_id>`)*.

### Q: What is the difference between CMD and ENTRYPOINT in a Dockerfile?
- **CMD**: Defines default commands and arguments that will be executed when a container starts. It is easily overridden by passing command-line arguments to the `docker run` command.
- **ENTRYPOINT**: Configures the container to run as an executable. Arguments passed during startup are appended to the ENTRYPOINT command. It is more strict than CMD and is used when you want to enforce specific executable behavior.

### Q: What does the `docker info` command do?
The `docker info` command displays system-wide information regarding the Docker installation. This includes details such as the number of containers (running, paused, stopped), the number of images, the storage driver in use, system resources, and other configuration parameters. It is useful for auditing the Docker host environment.

### Q: Where are Docker volumes stored on the host machine?
By default, Docker volumes are stored in the host machine's directory:
```text
/var/lib/docker/volumes
```
This directory ensures the persistence of data even after the associated container is removed.

### Q: What is the difference between a Docker image and a layer?
- **Docker Image**: A read-only snapshot representing a root filesystem and all application dependencies.
- **Layer**: A modification state of the image filesystem. An image is composed of multiple stacked read-only layers. Each instruction in a Dockerfile creates a new layer. This architecture allows Docker to cache and share layers efficiently across different images, saving storage and build time.

### Q: Can a container restart automatically?
Yes. A container can be configured to restart automatically by specifying a restart policy using the `--restart` flag during `docker run` or in a compose file. For example:
```bash
docker run --restart always <image_name>
```
Available policies include `always`, `on-failure`, and `no`.

### Q: What is the difference between Docker restart policies "no", "on-failure", and "always"?
- **`no`**: The default policy. Do not automatically restart the container.
- **`on-failure`**: Restarts the container only if it exits with a non-zero exit code (indicating an error). You can optionally limit the number of restart attempts.
- **`always`**: Always restarts the container if it stops, regardless of the exit code. If it is manually stopped, it is restarted only when the Docker daemon restarts or the container is manually started.

### Q: What are the essential Docker commands and their purposes?
- `docker run`: Launches a new container from a specified Docker image.
- `docker ps`: Lists currently running containers (`docker ps -a` lists all containers, including stopped ones).
- `docker exec`: Executes a command inside an already running container (e.g., `docker exec -it <container_id> bash`).
- `docker stop`: Gracefully stops a running container.

### Q: Why is `docker system prune` used, and what does it do?
`docker system prune` is used to clean up unused Docker resources and free up disk space. It removes:
- All stopped containers.
- All networks not used by at least one container.
- All dangling images (images not associated with any container and not tagged).
- All build caches.

```bash
docker system prune
```

### Q: How do you limit the CPU and memory usage of a Docker container?
You can set resource constraints using the `--cpus` option to limit CPU allocation and `-m` (or `--memory`) to limit RAM allocation. For example:
```bash
docker run --cpus=3 -m 1024M <image_name>
```
This limits the container to 3 CPUs and 1024 megabytes of memory.

### Q: What is the purpose of the `docker checkpoint` command?
The `docker checkpoint` command creates a checkpoint/snapshot of a running container's state, including its filesystem state and memory contents. This is useful for debugging, live migration, or testing backup states.
For example, to checkpoint a container named `my_container`:
```bash
docker checkpoint create my_container checkpoint_name
```

### Q: How do the Docker daemon and the Docker client communicate with each other?
They communicate using a REST API over a UNIX socket (by default at `/var/run/docker.sock`) or over a network TCP socket. The client sends API commands to the daemon, which processes them and manages containers, images, volumes, and networks.

### Q: How do you create a multi-stage build in Docker?
A multi-stage build uses multiple `FROM` instructions in a single `Dockerfile`. Each stage can use a different base image and can copy build artifacts from previous stages. This helps keep the final production image size small by excluding build-time dependencies.
Example:
```dockerfile
# Build stage
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Final stage
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

### Q: How do you copy files or folders between a container and the local filesystem?
Use the `docker cp` command:
- Copy from container to host:
  ```bash
  docker cp CONTAINER:SRC_PATH DEST_PATH
  ```
- Copy from host to container:
  ```bash
  docker cp SRC_PATH CONTAINER:DEST_PATH
  ```

### Q: How do you create a custom Docker network?
Use the `docker network create` command. For example, to create a bridge network:
```bash
docker network create mynetwork
```
This creates a custom user-defined network using the default bridge driver, which allows connected containers to resolve each other by name (DNS).

### Q: What is the purpose of the bridge network driver in Docker?
The bridge driver is the default network driver for containers. It establishes an internal private network on the host machine. Containers connected to the same bridge network can communicate with one another, while remaining isolated from external networks unless ports are mapped explicitly.

### Q: What is the difference between the COPY and ADD instructions in a Dockerfile?
- **`COPY`**: Simply copies local files or directories from the host machine into the container filesystem. It is straightforward and recommended for most copy tasks.
- **`ADD`**: Can do everything `COPY` does, plus it can download files from remote URLs and automatically extract compressed archives (e.g., `.tar.gz`, `.tar`) into the destination path. It is recommended to use `COPY` unless you explicitly need the tar extraction or URL download capabilities of `ADD`.

### Q: What is the purpose of Docker namespaces?
Docker namespaces provide isolation for containers. When a container runs, Docker creates a set of namespaces for it, isolating its view of system resources so that it cannot see or modify the host system or other containers' namespaces. Key namespaces include:
- **PID**: Process isolation (processes inside the container cannot see host processes).
- **NET**: Network resource isolation (separate interfaces, ports, and route tables).
- **MNT**: Filesystem mount point isolation.
- **IPC**: Interprocess communication isolation.
- **UTS**: Hostname and domain name isolation.
- **USER**: User and group ID isolation.

---

## Dockerfile Reference Sheet

| Instruction | Purpose |
| :--- | :--- |
| **`FROM`** | Creates a new build stage from a base image. |
| **`RUN`** | Executes build commands to construct the image layers. |
| **`CMD`** | Specifies default commands and arguments for container startup. |
| **`LABEL`** | Adds metadata to an image. |
| **`EXPOSE`** | Documents which ports the container listens on at runtime. |
| **`ENV`** | Sets persistent environment variables. |
| **`ADD`** | Copies files, pulls from URLs, or unpacks compressed tar files. |
| **`COPY`** | Copies files and directories into the container. |
| **`ENTRYPOINT`** | Configures the container to run as a fixed executable. |
| **`VOLUME`** | Creates volume mounts for persistent data. |
| **`USER`** | Sets the user and group ID for executing subsequent instructions. |
| **`WORKDIR`** | Changes the working directory for commands. |
| **`ARG`** | Defines build-time variables. |
| **`ONBUILD`** | Registers trigger instructions to run when the image is used as a base. |
| **`STOPSIGNAL`** | Specifies the system call signal for exiting the container. |
| **`HEALTHCHECK`** | Defines a command to check the container's health on startup. |
| **`SHELL`** | Sets the default shell executable for running commands. |
