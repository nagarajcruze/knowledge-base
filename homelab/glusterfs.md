# GlusterFS Setup Guide

A guide for setting up GlusterFS replicated storage volumes across virtual machines in a homelab environment.

---

## 1. Installation

Run the following commands on **both VMs** to install the GlusterFS server:
```bash
sudo apt update
sudo apt install -y glusterfs-server
sudo systemctl enable --now glusterd
sudo systemctl status glusterd   # Verify it is active (running)
```

---

## 2. Directory Structure Setup

Run on **both VMs** to create the storage brick directories and set the ownership to the current non-root user:
```bash
sudo mkdir -p /gluster/jenkins
sudo chown -R $(whoami):$(whoami) /gluster/jenkins
```

---

## 3. Trusted Storage Pool Configuration

Run on **one VM** (e.g., `192.168.1.148`) to peer probe the other node (`192.168.1.111`):
```bash
sudo gluster peer probe 192.168.1.111
```

Verify peer status on either VM:
```bash
sudo gluster peer status
```

---

## 4. Replicated Volume Creation

Run on the primary VM (`192.168.1.148`) to create a 2-way replicated GlusterFS volume:
```bash
sudo gluster volume create jenkins-volume replica 2 \
  192.168.1.148:/gluster/jenkins \
  192.168.1.111:/gluster/jenkins
```

Start the volume:
```bash
sudo gluster volume start jenkins-volume
```

Verify volume status:
```bash
sudo gluster volume info
```

---

## 5. Mounting the GlusterFS Volume

### Option A: Temporary Mount (for Testing)
Run on **both nodes** to mount the volume to a local mount point:
```bash
sudo mkdir -p /mnt/jenkins
sudo mount -t glusterfs 192.168.1.148:/jenkins-volume /mnt/jenkins
```

Test replication by writing a file on one node and verifying it on the other:
```bash
# On 192.168.1.148
echo "hello from 148" > /mnt/jenkins/test1.txt

# On 192.168.1.111
cat /mnt/jenkins/test1.txt
```

### Option B: Permanent Mount (via `/etc/fstab`)
Add the following line to `/etc/fstab` on **both nodes** for persistent mounting on boot:
```text
192.168.1.148:/jenkins-volume /mnt/jenkins glusterfs defaults,_netdev 0 0
```

Test the fstab entry:
```bash
sudo mount -a
df -h | grep jenkins
```

---

## 6. Performance Tuning & Optimizations (Optional)

To improve read/write performance for workloads, run these commands to enable write-behind cache, quick-read cache, and adjust network ping timeouts:
```bash
sudo gluster volume set jenkins-volume performance.write-behind on
sudo gluster volume set jenkins-volume performance.quick-read on
sudo gluster volume set jenkins-volume network.ping-timeout 5
```
