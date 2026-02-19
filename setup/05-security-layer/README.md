## Security Monitoring VM Creation (Wazuh)

### Overview
Created a dedicated Virtual Machine for Wazuh - an open-source Security Information and Event Management (SIEM) and Extended Detection and Response (XDR) platform.

### VM Purpose
Wazuh will provide:
- **Security Event Monitoring** - Real-time log analysis and threat detection
- **Intrusion Detection** - Network and host-based intrusion detection
- **Vulnerability Detection** - Automated security vulnerability scanning
- **File Integrity Monitoring** - Track changes to critical system files
- **Compliance Monitoring** - PCI-DSS, HIPAA, GDPR compliance checks
- **Log Analysis** - Centralized log collection and analysis

### VM Specifications

#### Basic Configuration
- **VM ID:** 103
- **Name:** Wazuh
- **Node:** pve
- **Status:** Created (Stopped)

#### Hardware Resources
**CPU Configuration:**
- **Sockets:** 1
- **Cores:** 2
- **Total vCPUs:** 2
- **CPU Type:** x86-64-v2-AES
- **Reason:** Minimum 2 cores required for Wazuh manager performance

**Memory Allocation:**
- **RAM:** 4096 MB (4 GB)
- **Reason:** Minimum requirement for Wazuh in small lab environment
- **Note:** 8 GB recommended for production, but 4 GB workable for lab with limited resources

**Storage Configuration:**
- **Disk Size:** 50 GB
- **Storage:** local-lvm (LVM-thin)
- **Bus/Device:** SCSI (scsi0)
- **Controller:** VirtIO SCSI single
- **Cache:** Default (No cache)
- **Discard:** Enabled ✓
- **IO Thread:** Enabled ✓
- **Reason:** 50 GB provides adequate space for logs, indices, and system growth

#### System Settings
**Machine Type:**
- **Type:** q35
- **Reason:** Modern chipset with better performance and features

**BIOS:**
- **Type:** Default (SeaBIOS)

**Graphics Card:**
- **Type:** Default

**SCSI Controller:**
- **Type:** VirtIO SCSI single

**Qemu Guest Agent:**
- **Enabled:** Yes ✓
- **Reason:** Better VM management and monitoring from Proxmox

#### Operating System
**ISO Image:**
- **Distribution:** Ubuntu 24.04.3 LTS
- **Image:** ubuntu-24.04.3-live-server-amd64.iso
- **Guest OS Type:** Linux
- **Version:** 6.x - 2.6 Kernel
- **Size:** 3.30 GB

**Why Ubuntu 24.04.3 LTS:**
- Official Wazuh support and documentation
- Long-term support (5 years)
- Stable and well-tested
- Easy package management with APT
- Resource-efficient for security appliances

#### Network Configuration
**Network Interface (net0):**
- **Bridge:** vmbr1 (Internal_Office_LAN)
- **Model:** VirtIO (paravirtualized)
- **MAC Address:** Auto-generated
- **Firewall:** Enabled ✓
- **VLAN:** None

**Network Placement Rationale:**
- Connected to vmbr1 (192.168.1.0/24 network)
- Can monitor all internal VMs on same network
- Will receive IP via DHCP from pfSense or static configuration
- Accessible from bastion host for management

### Resource Allocation Summary

**Total Lab Resources:**
- **Total RAM Available:** 16 GB
- **Current Allocation:**
  - pfSense (VM 100): 512 MB
  - Bastion Host (VM 101): 512 MB
  - Domain Controller (VM 102): 4 GB
  - Wazuh (VM 103): 4 GB
  - **Total Used:** 9 GB
  - **Remaining:** 7 GB

**CPU Allocation:**
- **Host Total:** 4 cores (assumed)
- **Current Allocation:**
  - pfSense: 1 core
  - Bastion: 1 core
  - Domain Controller: 1 core
  - Wazuh: 2 cores
  - **Total Allocated:** 5 vCPUs (with overcommit)

### Optimization Features Enabled

1. **VirtIO Drivers**
   - SCSI controller for better disk performance
   - Network adapter for reduced CPU overhead

2. **Discard Support**
   - Enables TRIM for thin-provisioned storage
   - Reclaims unused space automatically

3. **IO Thread**
   - Separates IO operations to dedicated thread
   - Improves disk performance under load

4. **Q35 Machine Type**
   - Modern PCIe-based chipset
   - Better device support and performance

5. **Qemu Guest Agent**
   - Better shutdown/reboot handling
   - Enables backup with filesystem quiesce
   - Provides IP address visibility in Proxmox
