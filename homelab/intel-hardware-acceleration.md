# Intel IOMMU & Hardware Video Acceleration Guide

This guide describes how to configure Intel IOMMU for PCI device passthrough and setup VA-API (Intel QuickSync Video) hardware-accelerated transcoding on Intel-based systems.

---

## 1. Intel IOMMU Configuration for PCI Passthrough

Enables Input-Output Memory Management Unit (IOMMU) on Intel-based systems to allow passing raw PCI devices (like physical GPUs or NICs) directly into Virtual Machines (VMs).

### Edit Bootloader Configuration
Edit the GRUB configuration file:
```bash
sudo nano /etc/default/grub
```

Find the `GRUB_CMDLINE_LINUX_DEFAULT` setting and append the Intel IOMMU options:
```bash
# Before:
GRUB_CMDLINE_LINUX_DEFAULT="quiet"

# After:
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```
* `intel_iommu=on`: Enables Intel IOMMU kernel drivers.
* `iommu=pt` (optional but recommended): Prevents performance degradation by setting the IOMMU mode to "pass-through", letting devices access host memory directly.

### Apply Changes & Reboot
```bash
sudo update-grub
sudo reboot
```

### Verify IOMMU Status
Verify the kernel command line successfully enabled IOMMU:
```bash
dmesg | grep -i iommu
```
Ensure you see confirmations like `Intel-IOMMU: enabled` or `IOMMU enabled`.

### Identify PCI Devices for Passthrough
Find your GPU and corresponding audio device addresses (needed for Proxmox passthrough configs):
```bash
lspci -nn | grep -i -E "vga|audio"
```

---

## 2. Intel VA-API Hardware Acceleration

Configure Intel QuickSync Video (QSV) / VA-API hardware acceleration for video transcoding tasks (Plex, Jellyfin, Handbrake).

### Install Drivers
Ensure the non-free repositories are enabled in `/etc/apt/sources.list`, then install the VA-API drivers:
```bash
sudo apt update
sudo apt install -y intel-media-driver mesa-va-drivers vainfo
```
* `intel-media-driver`: Standard VA-API driver for Intel Broadwell and newer architectures (Gen8+). Use `i965-va-driver` if running on vintage hardware.

### Verify Hardware Driver Loading
Check if the drivers load and identify available video codecs:
```bash
vainfo
```
If configured correctly, `vainfo` will return a list of supported entry points (e.g., `VAProfileH264Encode`, `VAProfileHEVCMain`).
