# Oracle VirtualBox Settings (Debian 12 Headless on Windows 11 Home)
This document outlines the essential VirtualBox settings I used to run Debian 12 headless servers on Windows 11 Home. Only the settings that require adjustment for compatibility and performance are listed.

## System
- Base Memory: `6055MB`
- Chipset: `P!iX3`
- TPM Version: `None`
- Motherboard I/O APiC: `On`
- Motherboard Hardware clock in UTC: `On`
- Motherboard UEFI: `Off`
- Motherboard Secure Boot: `Off`
- Amount CPU's:
    - Control plane server: `6`
    - Worker nodes: `4`
- Processing Cap: `100%`
- Feature PAE/NX: `On`
- Feature Nested VT-x/AMD-V: `Off`
- Paravirtualization Interface: `KVM`
- Hardware Virtualization Nested Paging: `On`

## Storage
- Controller Name: `SATA`
- Controller Type: `AHCI`
- Port Count: `2`
- Use Host I/O Cache: `On`

## Network (I use only 1 adapter)
- Enable Network Adapter: `On`
- Attached to: `Bridged Adapter`
- Adapter Type: `Intel PRO/1000 MT Desktop (82540OEM)
- Promiscuous Mode: `Allow All`

# WARNING!
I'm running Windows 11 Home edition. This project is just for learning. One thing I have learned is that Windows 11 Home and Hyper-V are not a fan off VirtualBox. My VM servers are experiences issues like CPU soft lockups.

```bash
BUG: soft lockup - CPU#1 stuck for 21s! [swapper/1:0]
```

After some research this is caused by Windows using Hyper-V for it's security, meaning it's best to leave it **on**. VirtualBox will detect this and use NEM (Native Execution Mode) which is less performant.

I've setup my laptop power settings to the max, so the CPU does not Idle, no sleeping mode in Windows to keep things running fast.

In Debian I made the following adjustment in the hope the servers will run more smoothly:
Open the Grub file:
```bash
sudo vi /etc/default/grub
```
Change the following:
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet"      
```
To:
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet nohz=off idle=poll"
```
- **nohz=off** : Force the Kernel to regulary send interrupts, so it does not go into a quite state.
- **idle=poll** : Force the CPU to stay awake, no sleeping for him.

