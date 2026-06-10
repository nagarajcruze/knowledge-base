# Containerization Basics

## 1. Introduction to Containers

Historically, running applications meant installing them directly onto physical servers or inside Virtual Machines (VMs). However, this often led to the "works on my machine" problem—where code worked in development but failed in production due to mismatching dependency versions, OS updates, or configuration issues.

**Containerization** solves this by packaging an application together with all of its dependencies—libraries, config files, and runtimes—into a single, standardized unit called a **container**.

---

## 2. Containers vs. Virtual Machines

While both provide isolated environments for applications, they achieve it differently:

| Feature | Containers | Virtual Machines |
|---|---|---|
| **OS Architecture** | Shares the host operating system kernel. | Contains a complete guest operating system. |
| **Size** | Lightweight (typically Megabytes). | Heavyweight (typically Gigabytes). |
| **Startup Time** | Starts almost instantly (seconds). | Takes minutes to boot up the guest OS. |
| **Resource Efficiency**| Very high (minimal overhead). | Low (requires dedicated CPU/RAM allocations). |
| **Isolation** | Process-level isolation (Namespaces). | Hardware-level isolation (Hypervisor). |

---

## 3. Under the Hood: Namespaces & Cgroups

Docker and Podman are not hypervisors. Instead of virtualization, they rely on native Linux kernel features to run containers as isolated processes:
- **Namespaces**: Define what a container can **see**. Namespaces isolate system resources (e.g., processes, network interfaces, mount points, hostnames) so a container operates as if it has its own dedicated OS.
- **Control Groups (cgroups)**: Define what a container can **use**. Cgroups limit, audit, and throttle physical hardware usage (e.g., maximum memory, CPU shares, and disk write speeds) for a container.

---

## 4. Key Terminology

- **Image**: A read-only package containing the application code, libraries, settings, and filesystem. It is the blueprint used to build containers.
- **Container**: A running, writeable instance of an image. You can run, start, stop, move, or delete a container using a container engine.
- **Registry**: A service that hosts and distributes container images. Examples include:
  - **Docker Hub** (public repository, default for Docker).
  - **Quay.io** (commonly used with Red Hat/Podman).
  - **GHCR** (GitHub Container Registry).

---

## 5. Available Container Products

To run and manage containers, you need a container engine. The two most prominent engines are:

- **Docker**: The pioneer and industry standard for containerization. It uses a background daemon (`dockerd`) running as root to manage containers.
- **Podman**: A modern, security-focused alternative. It is daemonless (runs container processes directly under your shell) and rootless (requires no admin privileges to run containers), and natively supports Kubernetes-style Pods.
