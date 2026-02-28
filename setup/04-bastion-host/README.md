# Jump Host (Bastion Server) Setup

The Jump Host (also known as Bastion Host) serves as a secure entry point for accessing internal lab infrastructure. This lightweight Debian-based server provides SSH access to manage all VMs within the enterprise homelab environment.

### Basic Configuration
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
