# Jump Host (Bastion Server) Setup

## Overview
The Jump Host (also known as Bastion Host) serves as a secure entry point for accessing internal lab infrastructure. This lightweight Debian-based server provides SSH access to manage all VMs within the enterprise homelab environment.

## VM Specifications

### Basic Configuration
- **VM ID:** 101
- **VM Name:** bastionHost
- **Operating System:** Debian 12 (Bookworm)
- **ISO Image:** debian-12.0.0-amd64-netinst.iso (773.85 MB)
- **Role:** SSH Jump/Bastion Server for secure internal access

### Hardware Resources
| Component | Specification | Reason |
|-----------|--------------|---------|
| **CPU** | 1 vCPU (x86-64-v2-AES) | Sufficient for SSH sessions and terminal operations |
| **RAM** | 512 MB | Minimal footprint; Debian is very lightweight (can be increased if needed) |
| **Disk** | 20 GB (SCSI) | Adequate space for OS, tools, logs, and SSH keys |
| **Network** | 1 NIC (VirtIO on vmbr1) | Internal LAN connectivity only |

### System Configuration

#### Machine Settings
- **Machine Type:** q35 (Modern chipset for better hardware compatibility)
- **BIOS:** SeaBIOS (Default, suitable for Linux)
- **SCSI Controller:** VirtIO SCSI Single
- **Qemu Guest Agent:** Enabled (For better VM management and monitoring)

#### Storage Configuration
- **Bus/Device:** SCSI 0
- **Storage Pool:** local-lvm
- **Disk Size:** 20 GiB
- **Format:** Raw disk image
- **Discard:** Enabled ✓ (TRIM support for better storage management)
- **IO Thread:** Enabled ✓ (Improved disk I/O performance)

#### Network Configuration
- **Bridge:** vmbr1 (Internal LAN)
- **Model:** VirtIO (Paravirtualized for best performance)
- **MAC Address:** Auto-generated
- **Firewall:** Enabled ✓
- **VLAN Tag:** None
- **Planned IP:** 172.16.1.20/24
- **Gateway:** 172.16.1.1 (pfSense LAN)
- **DNS:** 172.16.1.10 (DC-01) or 8.8.8.8
