# DC-01 VM Creation in Proxmox

## Overview

DC-01 is the domain controller VM running Windows Server 2022 Core. This document covers the complete VM creation process in Proxmox.

## VM Specifications

Your DC-01 VM is configured as follows:

```
VM Configuration Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Property              Value
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
VM ID                 102
Name                  DC-01
Node                  pve
Status                Created (ready for install)

SYSTEM
  Machine Type        q35 (modern chipset)
  BIOS/Firmware       SeaBIOS
  Qemu Agent          Enabled

CPU & MEMORY
  CPU Sockets         1
  CPU Cores           2 (total 2 vCPU)
  CPU Type            x86-64-v2-AES
  Memory              2048 MiB (2 GB)

STORAGE
  Boot Disk           scsi0
  Size                32 GB
  Storage Backend     local-lvm
  Filesystem Format   Raw disk image (raw)
  Controller          VirtIO SCSI single
  Optimizations       Discard enabled, I/O thread enabled

OPERATING SYSTEM
  OS                  Windows Server 2022 Core
  ISO                 SERVER_EVAL_x64FRE_en-us.iso
  Boot Device         CD/DVD (ide2)

NETWORK INTERFACES
  Interface 1 (LAN)   net0 → vmbr1 (internal lab network)
                      Model: VirtIO (paravirtualized)
                      MAC: auto-generated
                      Firewall: Enabled

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Why These Settings?

### CPU: 2 Cores (Not 1)

- **Active Directory needs multi-core** for:
  - LDAP query processing
  - DNS resolution
  - Group Policy application
  - User authentication
- 2 cores provides responsive AD without excessive resource use
- pfSense (1 core) is I/O-bound; AD is CPU-bound

### Memory: 2 GB

- **Minimum for Windows Server 2022 Core:** 512 MB
- **Recommended for AD services:** 2 GB
- **Your setup includes:**
  - Active Directory Database (~200-500 MB depending on users)
  - DNS service
  - DHCP service
  - Group Policy
  - System cache and overhead
- 2 GB ensures responsive domain operations

### Storage: 60 GB

- **Windows Server Core install:** ~4-5 GB
- **AD Database:** Grows with users/computers
- **NTDS.dit (AD database):** Initial ~10 MB, grows with objects
- **System Reserve:** ~10 GB for logs, updates, snapshots
- **Buffer:** Extra space for growth

### Machine Type: q35

- Modern Intel Q35 chipset
- Better compatibility than legacy i440fx
- Consistent with pfSense setup

### Network: Single NIC on vmbr1

- Connected to internal lab network (172.16.1.0/24)
- vmbr1 bridge connects all internal lab VMs
- DC-01 acts as gateway router (172.16.1.1 is pfSense)
- No need for WAN/management NIC on DC-01

### SCSI Controller: VirtIO

- Best I/O performance for lab
- Low overhead
- Modern and reliable

---

## Step-by-Step VM Creation in Proxmox

### Step 1: Access Proxmox

1. Open browser: `https://192.168.1.100:8006/`
2. Login with `root` and Proxmox password
3. Click **"Create VM"** (top right)

### Step 2: General Tab

```
Node:              pve
VM ID:             102
Name:              DC-01
Add to HA:         unchecked
Resource Pool:     empty
```

**Click: Next**

### Step 3: OS Tab

```
Use CD/DVD disc image file (iso):  selected
Storage:                           local
ISO image:                         SERVER_EVAL_x64FRE_en-us.iso
Guest OS Type:                     Microsoft Windows
Guest OS Version:                  11/2022
```

**Click: Next**

### Step 4: System Tab

```
Graphic card:      Default
Machine:           q35
Firmware (BIOS):   Default (SeaBIOS)
SCSI Controller:   VirtIO SCSI single
Qemu Agent:        ☑️ checked
Add TPM:           unchecked
```

**Click: Next**

### Step 5: Disks Tab

```
Bus/Device:        SCSI
SCSI Controller:   VirtIO SCSI single
Storage:           local-lvm
Disk size (GiB):   32
Format:            Raw disk image (raw)
Cache:             Default (No cache)
Discard:           ☑️ checked
IO thread:         ☑️ checked
```

**Click: Next**

### Step 6: CPU Tab

```
Sockets:           1
Cores:             2
CPU Type:          x86-64-v2-AES
```

**Click: Next**

### Step 7: Memory Tab

```
Memory (MiB):      2048 (2 GB)
Minimum memory:    2048 (or leave empty)
```

**Click: Next**

### Step 8: Network Tab

```
Bridge:            vmbr1 (internal lab network)
VLAN Tag:          empty
Model:             VirtIO (paravirtualized)
MAC address:       auto
Firewall:          ☑️ checked
```

**Click: Next**

### Step 9: Confirm and Create

1. Review all settings
2. Click **"Finish"** to create VM 102 (DC-01)
Memory:            2 GB RAM
Processor:         2 vCPU
