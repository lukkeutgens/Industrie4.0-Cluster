# Oracle VirtualBox Settings (Debian 12 Headless on Windows 11 Home)
This document outlines the essential VirtualBox settings I used to run Debian 12 headless servers on Windows 11 Home. Only the settings that require adjustment for compatibility and performance are listed.

---

## VirtualBox Settings
### System
- Base Memory: `6055MB`
- Chipset: `PIIX3`
- TPM Version: `None`
- I/O APiC: `Enabled`
- Hardware clock in UTC: `Enabled`
- UEFI: `Disabled`
- Motherboard Secure Boot: `Disabled`
- Amount CPU's:
    - Control plane server: `6`
    - Worker nodes: `4`
- Processing Cap: `100%`
- PAE/NX: `Enabled`
- Nested VT-x/AMD-V: `Disabled`
- Paravirtualization Interface: `KVM`
- Hardware Virtualization Nested Paging: `Enabled`

### Storage
- Controller Name: `SATA`
- Controller Type: `AHCI`
- Port Count: `2`
- Use Host I/O Cache: `Enabled`
- disk1.vdi 
    - Hard Disk: SATA Port 0
    - Solid-state Drive: Enabled
    - Hot-pluggable: Disabled
    - Disk size: 60GB
 - disk2.vdi 
    - Hard Disk: SATA Port 1
    - Solid-state Drive: Enabled
    - Hot-pluggable: Disabled
    - Disk size: 160GB
  
### Network (Single Adapter)
- Enable Network Adapter: `Enabled`
- Attached to: `Bridged Adapter`
- Adapter Type: `Intel PRO/1000 MT Desktop (82540OEM)
- Promiscuous Mode: `Allow All`

---

## ⚠️ Important Notes on Windows 11 Home + VirtualBox
This setup is for learning purposes only. Windows 11 Home uses Hyper-V components for security, which interferes with VirtualBox's native virtualization. As a result, VirtualBox falls back to **NEM (Native Execution Mode)**, which is less performant and can cause issues like CPU soft lockups.

Typical error observed:
```bash
BUG: soft lockup - CPU#1 stuck for 21s! [swapper/1:0]
```

Since Hyper-V is tied to Windows security features, it's best to leave it enabled. To mitigate performance issues, I’ve configured my laptop’s power profile to maximum performance, disabling sleep and idle states.

### Debian Kernel Tweaks (GRUB)
To improve VM responsiveness under NEM mode, I applied the following kernel parameters:
1. Edit GRUB
```bash
sudo vi /etc/default/grub
```
2. Modify the line
From:
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet"      
```
To:
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet mitigations=off nohz=off idle=halt"
```
Explanation:
- **nohz=off** : Forces the kernel to send regular timer interrupts, preventing deep idle states.
- **idle=halt** : Allows the CPU to enter a light halt state while still responding to interrupts, reducing lockup risk in virtualized environments.
- **mitigations=off** : Disables CPU vulnerability mitigations (e.g. Spectre, Meltdown) to reduce overhead and improve performance in virtualized environments.
- **quiet** : Suppresses most kernel boot messages to reduce console output during startup (standard active in GRUB).

3. Apply changes
```bash
sudo update-grub
sudo reboot
```



