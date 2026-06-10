# Linux Disk Management & Storage Administration

A comprehensive SRE guide for disk discovery, partitioning, formatting, mount operations, hardware diagnostics, and archiving.

---

## 1. Device Discovery & Identification

Before modifying block devices, verify their identity to prevent accidental data loss.

- **List Block Devices**:
  ```bash
  lsblk
  ```
  *Lists block devices, sizes, mount points, and partition layouts.*
- **List SCSI Devices**:
  ```bash
  lsblk -S
  ```
  *Queries specific physical disk parameters (transports, controller types).*
- **Retrieve Filesystem UUIDs**:
  ```bash
  blkid
  ```
  *Displays partition UUIDs, labels, and file system formats (e.g. ext4, NTFS).*
- **Inspect Partition Formats**:
  ```bash
  sudo disktype /dev/sda
  ```
  *Analyzes sector structures to verify filesystem formats and identify partition tables on unformatted disks.*

---

## 2. Partitioning & Formatting

Storage preparation is a two-step process: **Partitioning** (mapping physical drive slices) and **Formatting** (writing the filesystem database structure).

```text
    [Raw Disk: /dev/sda]
┌──────────────────────────────────────────────┐
│  Partition 1 (/dev/sda1)  │  Partition 2    │  <- Partitioning Map (MBR/GPT)
├───────────────────────────┼──────────────────┤
│  mkfs.ext4 (Filesystem)   │  mkfs.ntfs       │  <- Formatting (Filesystem structures)
└───────────────────────────└──────────────────┘
```

### A. Partitioning Schemes
- **MBR (Master Boot Record)**: Legacy standard. Supports up to 4 primary partitions and a maximum disk size of 2TB. Managed via `fdisk`.
- **GPT (GUID Partition Table)**: Modern standard. Supports virtually unlimited partitions and disks larger than 2TB. Highly recommended for modern drives. Managed via `parted`.

#### 1. Managing MBR Partitions (`fdisk`)
Launch interactive fdisk:
```bash
sudo fdisk /dev/sda
```
- **Interactive Commands**:
  - `o`: Create a new empty MBR partition table.
  - `n`: Add a new partition (specify primary/extended, sector bounds, size e.g. `+10G`).
  - `t`: Change partition filesystem type code (e.g. `83` Linux, `7` NTFS, `b` FAT32).
  - `p`: Print current partition layout table.
  - `w`: Write changes to disk and exit (changes are not applied until written).
  - `q`: Quit without saving.

#### 2. Managing GPT Partitions (`parted`)
Parted applies changes live.
- **Interactive Session**:
  ```bash
  sudo parted /dev/sda
  ```
- **Commands**:
  - `mklabel gpt`: Set partitioning scheme to GPT.
  - `mkpart primary ext4 0% 50%`: Create a primary partition covering the first half of the disk.
  - `print`: Display partition layouts.
  - `quit`: Exit session.

---

### B. Formatting Partitions (`mkfs`)
Apply the filesystem database to partition block devices.
- **EXT4 (Linux standard)**:
  ```bash
  sudo mkfs.ext4 -L "my_ext4_vol" /dev/sda1
  ```
- **NTFS (Windows compatible, quick format)**:
  ```bash
  sudo mkfs.ntfs -Q -L "my_ntfs_vol" /dev/sda2
  ```
- **FAT32 (Universal compatibility)**:
  ```bash
  sudo mkfs.vfat -n "my_fat_vol" /dev/sda3
  ```

### C. Wiping Filesystem Signatures
To clear partition tables and filesystem headers without zeroing the entire drive:
```bash
sudo wipefs -a /dev/sda
```

---

## 3. Mounting, Unmounting, and Access Control

Mounting links a physical storage partition to a folder path in the Linux directory tree.

### Mounting filesystems
Create a directory under `/mnt/` or `/media/` to act as the mount point:
```bash
sudo mkdir -p /mnt/data
```
- **Standard Mount**:
  ```bash
  sudo mount /dev/sda1 /mnt/data
  ```
- **Mount with Specific Filesystem**:
  ```bash
  sudo mount -t ntfs /dev/sda2 /mnt/data
  ```
- **Secure Forensics/SRE Mount (Read-Only, No-Execution)**:
  ```bash
  sudo mount -t ext4 -o ro,noexec /dev/sda1 /mnt/data
  ```
  *The `noexec` option prevents binary execution from the mounted partition, protecting systems from malware during forensic analysis.*

### Unmounting filesystems
Disconnect filesystems before physical removal:
- **Unmount via Partition device**:
  ```bash
  sudo umount /dev/sda1
  ```
- **Unmount via Mount Point folder**:
  ```bash
  sudo umount /mnt/data
  ```

#### Troubleshooting: "target is busy" Errors
If `umount` fails with a target busy error:
1. Ensure your shell working directory is not inside the mount point. Run `cd ~` to leave.
2. Find open file descriptors using the drive:
   ```bash
   sudo lsof +f -- /mnt/data
   ```

---

## 4. Disk Space Analysis

Monitor and analyze storage utilization.

- **Filesystem Disk Usage (`df`)**:
  ```bash
  df -hT
  ```
  *Shows total, used, free space, and filesystem type (`-T`) in human-readable gigabytes/megabytes (`-h`).*
- **Directory Disk Usage (`du`)**:
  ```bash
  sudo du -shc /var/log/*
  ```
  *Estimates disk space used by files/subfolders in `/var/log`, returning a summary (`-s`), human-readable sizes (`-h`), and grand total (`-c`).*

---

## 5. Low-Level Hardware Diagnostics & Security

### A. Drive Information (`hdparm`)
Retrieve hardware parameters directly from the drive firmware:
```bash
sudo hdparm -I /dev/sda
```
*Displays model, serial number, firmware version, and supported capabilities.*

### B. Host Protected Area (HPA)
HPA is a hidden area of the drive reserved for boot code or recovery datasets, invisible to standard OS tools.
- **Check HPA status**:
  ```bash
  sudo hdparm -N /dev/sda
  ```
- **Temporarily disable or set HPA sectors**:
  ```bash
  sudo hdparm -N [sector_count] --yes-i-know-what-i-am-doing /dev/sda
  ```
  > [!WARNING]
  > Modifying HPA settings can result in data loss. Reboot or replug the device afterwards to update disk size registration.

### C. ATA Security & Secure Erase
Wipe or lock SATA drives using internal drive controllers.
1. **Set User Password**:
   ```bash
   sudo hdparm --user-master u --security-set-pass 'my_password' /dev/sda
   ```
2. **Execute ATA Secure Erase**:
   ```bash
   sudo hdparm --user-master u --security-erase 'my_password' /dev/sda
   ```
   *Instructs the drive firmware to zero out all sectors internally. This runs significantly faster than standard software overwrites.*

### D. Hexadecimal Inspections (`xxd`)
To check the raw bytes of a block device directly (e.g., checking partition tables or verifying if a disk is wiped):
```bash
sudo xxd -l 512 /dev/sda
```
*Outputs the first 512 bytes (MBR boot sector) of the drive in hex format.*

---

## 6. High-Performance Copying & Archiving

### A. Delta File Synchronization (`rsync`)
`rsync` copies files efficiently by transferring only block differences (deltas) between the source and destination.

- **Synchronize local directories**:
  ```bash
  rsync -av --progress /source/dir/ /destination/dir/
  ```
  *`-a` (archive) preserves permissions, owners, and timestamps. `-v` shows details.*
- **Resume broken transfers**:
  Running `rsync` again will skip unchanged files and only sync updated segments.

### B. Tape Archiving (`tar`)
Bundles directory trees into a single archive file.

- **Create Tarball**:
  ```bash
  tar -cvf backup.tar /etc/nginx
  ```
  *`-c` creates, `-v` is verbose, `-f` specifies target file.*
- **Append files to an existing archive**:
  ```bash
  tar -rvf backup.tar /var/log/nginx
  ```
- **Extract Tarball**:
  ```bash
  tar -xvf backup.tar
  ```
  > [!TIP]
  > Create and `cd` into a clean destination folder before running `tar -xvf` to prevent files from scattering or overwriting existing structures in your current directory.