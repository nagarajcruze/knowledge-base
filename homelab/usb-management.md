# USB Device Management: Passthrough & Safe Removal

This guide describes how to pass external USB storage drives through Proxmox hosts to LXC containers and Docker, and how to safely unmount and spin down physical HDDs to prevent filesystem corruption.

---

## 1. Mounting External USB Drives in LXC & Docker

Pass-through host directories (such as external USB mounts) into an LXC container, then expose them to your Docker stack.

### Step 1: Add Mount Point to LXC Config
On the Proxmox host, edit your LXC container configuration (e.g., container ID `101`):
```bash
nano /etc/pve/lxc/101.conf
```
Add a mountpoint entry linking the host directory to the container directory:
```ini
mp1: /mnt/usb,mp=/mnt/usb
```

### Step 2: Restart the LXC Container
Apply configuration updates by rebooting the container:
```bash
pct reboot 101
```

### Step 3: Map Mountpoint inside Docker Compose
In the LXC container, edit your service's `docker-compose.yml` to bind-mount the mapped directory:
```yaml
services:
  immich-server:
    image: ghcr.io/immich-app/immich-server:release
    volumes:
      - /mnt/usb:/mnt/usb:ro  # Read-only configuration prevents accidental data deletion
```

### Step 4: Deploy & Verify
```bash
# Restart Docker stack
docker compose down && docker compose up -d

# Verify mount accessibility within the running container
docker exec -it immich_server ls /mnt/usb
```

---

## 2. USB Safe Removal & Troubleshooting

Steps to safely unmount and spin down external HDDs to prevent filesystem corruption and hardware damage.

### Prerequisite: Install hdparm
```bash
sudo apt install -y hdparm
```

### Step 1: Check for Active Disk Users
Identify any running processes or open shells holding locks on the USB directory:
```bash
lsof +D /mnt/usb
# OR
fuser -vm /mnt/usb
```
*Make sure to kill or stop any listed tasks (e.g., stop the Docker container using the mount) before continuing.*

### Step 2: Safe Disk Unmount & Spin Down
Run the sequential commands below to unmount the drive, flush kernel caching structures, put the drive to sleep, and safely detach it from the SCSI subsystem.

```bash
# Method A: Deep kernel disconnect (Standard)
sudo umount /dev/sdb2 && sync && sudo hdparm -y /dev/sdb && echo 1 | sudo tee /sys/block/sdb/device/delete

# Method B: Udisksctl utility (User space helper)
sudo umount /dev/sdb* && udisksctl power-off -b /dev/sdb
```

**Commands Breakdown:**
* `umount`: Safely detaches the partition from the directory tree.
* `sync`: Flushes files currently residing in the RAM write-cache onto the physical platters.
* `hdparm -y`: Spins down the drive motor immediately (enters standby mode).
* `delete`: Safely removes the disk registration from the operating system kernel.

Verify the disk is no longer registered:
```bash
lsblk | grep sdb
```
