# Docker Setup & Utility Scripts

This guide walks through the installation of Docker CE and Docker Compose on Debian systems, alongside helper shell scripts for batch operations, Immich API interactions, and fast rsync file copy routines.

---

## 1. Docker & Docker Compose Installation on Debian

A clean installation walkthrough for installing official Docker Community Edition on a new Debian host.

### Step 1: Remove Old Packages
Remove preinstalled, conflicting container tools:
```bash
sudo apt remove -y docker.io docker-compose docker-doc podman-docker containerd runc
```

### Step 2: Install System Prerequisites
```bash
sudo apt update
sudo apt install -y curl wget ca-certificates gnupg
```

### Step 3: Add Official Docker GPG Key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### Step 4: Add Docker Repository to APT Sources
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Step 5: Install Docker Components
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 6: Verify Operation
Ensure Docker and the Compose plugin are running and accessible:
```bash
docker version
docker compose version
```

---

## 2. High-Performance File Transfers via Rsync

Transfer large media libraries locally or over a network using optimal `rsync` flags.

```bash
rsync -ah --info=progress2 --no-inc-recursive /mnt/usb/ /mnt/media/movies/
```

**Flags Breakdown:**
* `-a` (archive): Preserves symlinks, ownership, metadata, and timestamps.
* `-h` (human-readable): Outputs data rates and sizes in human-friendly units (MB/GB).
* `--info=progress2`: Shows progress based on total size rather than file-by-file progress.
* `--no-inc-recursive`: Forces rsync to scan the entire directory structure *before* beginning transfers. This enables highly accurate overall progress calculations.

---

## 3. Immich CLI Authentication & Uploads

Using the Immich command-line interface to import photo and video assets recursively.

### Log in to Immich CLI
```bash
immich login http://<IMMICH_HOST_IP>:2283/api <YOUR_API_KEY>
```

### Upload Media Recursively
Upload an entire directory structure while skipping duplicates:
```bash
immich upload ./ --recursive
```

---

## 4. Batch Copying Files from a List

Efficiently copy files from an input text file containing lists of absolute file paths.

```bash
# Create target directory
mkdir -p /mnt/media/temp/

# Loop read file list
while IFS= read -r file; do
  if [ -f "$file" ]; then
    cp "$file" /mnt/media/temp/
  else
    echo "Warning: File $file not found, skipping."
  fi
done < movies.txt
```
