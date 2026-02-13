# Proxmox Foundation - Infrastructure Base

This section covers the bare metal installation and initial configuration of Proxmox VE, which serves as the foundation for your entire lab infrastructure.

## Overview

Proxmox VE (PVE) is a powerful, open-source virtualization platform built on Debian Linux. It provides:
- KVM hypervisor for virtual machine management
- LXC containers for lightweight containerization
- High availability and clustering support
- Web-based management interface
- REST API for automation

## What You Will Learn

This guide covers:
1. **Proxmox VE 9.1 bare metal installation** from ISO
2. **Network configuration** for management access
3. **Storage setup** with LVM and ZFS
4. **Initial post-installation steps**
5. **Accessing the Proxmox web interface**

## Prerequisites

Before starting, ensure you have:
- A physical or virtual server with 4GB+ RAM
- At least 100GB storage (SSD recommended for better performance)
- Network connectivity (DHCP or static IP assignment)
- Proxmox VE 9.1 ISO file
- Bootable USB drive or ISO mounted

## Your Lab Setup

### **Network Configuration**
- **Management Network:** 192.168.1.0/24 (external/home network)
- **Proxmox Host IP:** 192.168.1.100
- **Hostname:** pve.lab.internal
- **Gateway:** 192.168.1.1
- **DNS:** 192.168.1.1

### **Server Hardware**
- **Filesystem:** ext4 with LVM
- **Disk:** /dev/sda (100GB)
- **Root Volume:** /dev/mapper/pve-root
- **Boot:** UEFI with GRUB2

## Next Steps

After completing Proxmox installation, you will:
1. Access the web interface at `https://192.168.1.100:8006/`
2. Configure network bridges (vmbr0, vmbr1)
3. Set up storage pools for VMs
4. Begin creating VMs for each lab layer
