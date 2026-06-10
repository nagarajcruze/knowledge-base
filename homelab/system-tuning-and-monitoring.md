# System Tuning & Monitoring Guide

This guide details homelab server system optimization techniques, including CPU frequency governor configurations and real-time performance monitoring.

---

## 1. CPU Power Saving & Turbo Boost Optimization

Optimize power efficiency and heat output for small-form-factor homelab systems (e.g., Dell OptiPlex) by controlling CPU frequencies.

### Prerequisite: Install cpupower
On Debian, install CPU governor control utilities:
```bash
sudo apt install -y linux-cpupower
```

### Create systemd Service for Automation
Configure a systemd service that enforces powersave policies and disables CPU turboboost at boot.

```bash
sudo nano /etc/systemd/system/cpuboost.service
```

Paste the following configurations:
```ini
[Unit]
Description=Disable CPU Boost & Enforce Powersave Profile
After=syslog.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c "/usr/bin/cpupower frequency-set -g powersave && /usr/bin/cpupower frequency-set -u 2.4GHz && echo 0 > /sys/devices/system/cpu/cpufreq/boost"
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

### Enable and Run the Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now cpuboost.service
```

### Manual Tuning Commands
To temporarily adjust CPU max limits on the fly:
```bash
# Limit CPU frequency
sudo cpupower frequency-set -u 2.5GHz

# View current scheduler profiles and clock speeds
cpupower frequency-info
```

---

## 2. Essential Homelab Troubleshooting Commands

Quick commands to assess system status and debug failures.

### Check Active USB Mounts
```bash
mount | grep -i usb
```

### Storage Space & Free Inodes
```bash
df -h
df -i
```

### Docker Services Health
```bash
docker ps -a
docker logs --tail=100 -f <container_name>
```

### GPU Utilization & Temperatures
```bash
nvidia-smi
```

### CPU Power Profiling
```bash
cpupower frequency-info
```
