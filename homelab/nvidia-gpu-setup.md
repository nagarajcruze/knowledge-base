# NVIDIA GPU Configuration Guide

This guide describes how to configure NVIDIA GPUs for containerized workloads on a Debian host using Docker and how to pass GPUs through to Proxmox LXC containers.

---

## 1. NVIDIA GPU Setup for Docker

Configure the host machine to allow Docker containers to access and utilize the NVIDIA GPU for hardware-accelerated workloads (such as Plex, Jellyfin, or Immich machine learning features).

### Prerequisite
Ensure you have the official NVIDIA proprietary drivers installed on the host before installing the Container Toolkit.

### Add NVIDIA Container Toolkit Repository
```bash
# Download and install the repository GPG key for secure package verification
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# Add the official apt sources list configured to trust the downloaded key ring
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

### Install Toolkit
```bash
sudo apt update
sudo apt install -y nvidia-container-toolkit
```

### Configure Docker Runtime
Update Docker's configuration (`/etc/docker/daemon.json`) to register the NVIDIA Container Runtime:
```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### Verify Installation
Check that `nvidia` is listed as an available runtime in Docker:
```bash
docker info | grep -i runtime
```
*Expected Output:*
```text
 Runtimes: io.containerd.runc.v2 nvidia runc
```

### Test GPU Passthrough
Run a temporary CUDA-equipped container to ensure Docker can communicate with the GPU:
```bash
docker run --rm --gpus all nvidia/cuda:12.2.0-runtime-ubuntu22.04 nvidia-smi
```

### Troubleshooting: Missing Device Files
If `/dev/nvidia*` nodes are not automatically created on host startup, force initialization:
```bash
sudo nvidia-modprobe -u -c=0
```

---

## 2. NVIDIA GPU Passthrough to Proxmox LXC

To expose an NVIDIA GPU to an LXC container (e.g., container ID `100`), you must mount the host GPU device nodes and share the NVIDIA driver libraries.

> [!IMPORTANT]
> The driver version installed inside the LXC container **must match** the driver version installed on the Proxmox host exactly. Do not install the full NVIDIA kernel driver inside the LXC; install only the library files and utilities (using the `--no-kernel-module` installer flag).

### Configure the LXC Container
Edit the container configuration file on the Proxmox host:
```bash
nano /etc/pve/lxc/100.conf
```

Append the following configuration lines to pass through the driver libraries and GPU hardware nodes:
```ini
# Expose NVIDIA character devices
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 509:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file

# Mount host NVIDIA library paths (Read-Only)
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1 usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1 none bind,ro,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libcuda.so.1 usr/lib/x86_64-linux-gnu/libcuda.so.1 none bind,ro,create=file
```

### Reboot the Container
```bash
pct reboot 100
```

### Verify GPU Inside LXC
Log into the LXC container and verify that the host's libraries are linked properly:
```bash
ldconfig -p | grep nvidia
```
Verify GPU status using the Docker passthrough test inside the container:
```bash
docker run --rm --gpus all nvidia/cuda:12.2.0-runtime-ubuntu22.04 nvidia-smi
```
