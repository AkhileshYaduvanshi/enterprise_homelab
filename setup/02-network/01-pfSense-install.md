# pfSense VM Creation in Proxmox - Complete Guide

## Overview

pfSense is a free, open-source firewall and router based on FreeBSD. It provides enterprise-grade features for your lab:
- Firewall with stateful filtering
- Network address translation (NAT)
- VPN (OpenVPN, WireGuard)
- DHCP server
- DNS forwarding
- Advanced routing

## VM Specifications

Your pfSense VM is configured for optimal performance in a resource-constrained lab environment:

M Configuration Summary

VM ID 100
Name pfSense
Node pve

SYSTEM
Machine Type q35 (modern chipset)
BIOS/Firmware SeaBIOS
Qemu Agent Enabled
TPM Not enabled

CPU & MEMORY
CPU Sockets 1
CPU Cores 1 (total 1 vCPU)
CPU Type x86-64-v2-AES
Memory 1024 MiB (1 GB)

STORAGE
Boot Disk scsi0
Size 32 GB
Storage Backend local-lvm
Filesystem Format Raw disk image (raw)
Controller VirtIO SCSI single
Optimizations Discard enabled, I/O thread enabled

OPERATING SYSTEM
OS pfSense 2.8.1-RELEASE
ISO netgate-installer-v1.1.1-RELEASE-amd64.iso
Boot Device CD/DVD (ide2)

NETWORK INTERFACES
Interface 1 (WAN) net0 → vmbr0 (management network)
Model: VirtIO (paravirtualized)
MAC: auto-generated
Firewall: Enabled

Interface 2 (LAN) net1 → vmbr1 (lab network)
Model: VirtIO (paravirtualized)
MAC: auto-generated
Firewall: Enabled


## Why These Settings?

### CPU: 1 Core (Not 2)
- **Why 1 core:** pfSense is I/O-bound, not CPU-bound
- **Packet processing:** Single core handles routing/filtering efficiently
- **Resource savings:** Leaves more cores for DC-01, Wazuh, Jenkins
- **Lab environment:** Sufficient for firewall duties in internal network

**Note:** If you add heavy VPN usage or advanced rules, can increase to 2 cores later.

### Memory: 1 GB
- **Base pfSense:** 256-512 MB minimum
- **VPN Support:** +256-512 MB for encryption/decryption
- **Logs & State Tables:** +256 MB for connections tracking
- **Total:** 1 GB is optimal for lab firewall with VPN
- **Headroom:** Not excessive, balanced for your 16GB system

### Storage: 32 GB
- **pfSense install:** ~2-3 GB
- **Logs:** Room for 6-12 months of logs (depending on traffic)
- **RRD graphs:** Historical traffic graphs stored locally
- **Backups:** Configuration backups (optional)
- **Safe margin:** 32 GB ensures no disk space issues

### Machine Type: q35
- **Modern:** Latest Intel Q35 chipset (recommended)
- **Performance:** Better than legacy i440fx
- **Consistency:** Matches DC-01 and future VMs
- **Compatibility:** Works perfectly with FreeBSD/pfSense

### Network Interfaces: Two VirtIO NICs
- **net0 (WAN):** vmbr0 → Connects to management network (192.168.1.x)
  - Gets IP from home router or static assignment
  - Communicates with Proxmox host and external networks
  
- **net1 (LAN):** vmbr1 → Connects to lab network (172.16.1.x)
  - Will be configured as 172.16.1.1 (gateway)
  - All lab VMs (DC-01, etc) use this as default gateway

**VirtIO Model:** Best performance for network I/O, low overhead

---

## Step-by-Step: pfSense VM Creation in Proxmox

### Step 1: Access Proxmox Web UI

1. Open browser and navigate to `https://192.168.1.100:8006/`
2. Login with `root` and your Proxmox password
3. You're now in the Proxmox VE dashboard

### Step 2: Create New VM

1. Click **"Create VM"** button (top right)
2. The VM creation wizard opens

### Step 3: General Tab

**Values to Enter:**
- **Node:** `pve` (already selected)
- **VM ID:** `100`
- **Name:** `pfSense`
- **Add to HA:** leave unchecked
- **Resource Pool:** leave empty

**Click: Next**

### Step 4: OS Tab

**Values to Select:**
- **Use CD/DVD disc image file (iso):** selected
- **Storage:** `local` (where ISOs are stored)
- **ISO image:** `netgate-installer-v1.1.1-RELEASE-amd64.iso`
- **Guest OS Type:** `Linux`
- **Version:** `6.x - 2.6 Kernel`

**Why Linux type?** FreeBSD (pfSense base) is compatible with Linux kernel option in Proxmox

**Click: Next**

### Step 5: System Tab

**Values to Configure:**
- **Graphic card:** `Default`
- **Machine:** Change to `q35` (important for modern performance)
- **Firmware (BIOS):** `Default (SeaBIOS)`
- **SCSI Controller:** `VirtIO SCSI single`
- **Qemu Agent:** Check ✓ (enables better integration)
- **Add TPM:** Leave unchecked (not needed for firewall)

**Click: Next**

### Step 6: Disks Tab

**Values to Configure:**
- **Bus/Device:** `SCSI`
- **SCSI Controller:** `VirtIO SCSI single`
- **Storage:** `local-lvm`
- **Disk size (GiB):** `32`
- **Format:** `Raw disk image (raw)`
- **Cache:** `Default (No cache)`
- **Discard:** Check ✓ (helps with thin provisioning)
- **IO thread:** Check ✓ (improves I/O performance)

**Click: Next**

### Step 7: CPU Tab

**Values to Configure:**
- **Sockets:** `1`
- **Cores:** `1` (keep minimal, firewall doesn't need much CPU)
- **CPU Type:** `x86-64-v2-AES`

**Click: Next**

### Step 8: Memory Tab

**Values to Configure:**
- **Memory (MiB):** `1024` (1 GB - perfect for pfSense with VPN)
- **Minimum memory:** leave empty (no ballooning)

**Click: Next**

### Step 9: Network Tab

**First Interface (WAN):**
- **Bridge:** `vmbr0` (management network)
- **Model:** `VirtIO (paravirtualized)`
- **VLAN Tag:** leave empty
- **Firewall:** Check ✓
- **MAC address:** `auto`

**Do NOT click Next yet!** We need to add second interface.

**Add Second Interface (LAN):**
- Look for an "Add" button or similar option
- If not available in this tab, we'll add after VM creation ✓

**Click: Next**

### Step 10: Confirm Tab

Review all settings in the summary table. Should show:

cores 1
cpu x86-64-v2-AES
ide2 local:iso/netgate-installer...
machine q35
memory 1024
name pfSense
net0 virtio,bridge=vmbr0,firewall=1
nodename pve
ostype l26
scsi0 local-lvm:32,discard=on,iothread=on
scsihw virtio-scsi-single
sockets 1
vmid 100

**Click: Finish** to create the VM

### Step 11: Add Second Network Interface (LAN)

After VM is created:

1. Click on **VM 100 (pfSense)** in the left panel
2. Go to **Hardware** tab
3. Look for "Add" button to add a device
4. Select "Network Device"
5. Configure as:
   - **Bridge:** `vmbr1` (lab network)
   - **Model:** `VirtIO (paravirtualized)`
   - **Firewall:** Check ✓
   - **MAC address:** `auto`
6. Click **Add**

Now pfSense has both interfaces:
- **net0:** WAN (vmbr0 - management)
- **net1:** LAN (vmbr1 - lab network)

---

## Verification

After VM creation, verify in Proxmox:

1. **VM 100 (pfSense)** appears in left panel ✓
2. **Summary tab** shows:
   - Memory: 1.00 GiB
   - Processors: 1 (1 sockets, 1 core)
   - Status: Stopped (ready for install)
3. **Hardware tab** shows:
   - Network Device (net0) on vmbr0
   - Network Device (net1) on vmbr1
   - scsi0 disk (32GB)
   - ide2 CD drive (pfSense ISO)

---

## Resource Allocation Summary

Your lab uses these resources for pfSense:

CPU: 1 out of available cores (light footprint)
RAM: 1 GB out of 16 GB (6.25% of available)
Storage: 32 GB allocated (actual use ~3-5 GB)

Leaves available for:

DC-01: 2 cores, 2 GB RAM

Wazuh: 2 cores, 2 GB RAM

Jenkins: 2 cores, 2 GB RAM

Others: remaining resources

---

## Troubleshooting

### Issue: VM won't start
- Check if Proxmox host has sufficient resources
- Verify ISO file exists and is accessible

### Issue: Network interfaces not visible
- Ensure bridges (vmbr0, vmbr1) exist in Proxmox Network configuration
- Verify network bridges are properly configured on the host

### Issue: Too slow during installation
- Allocate more vCPU cores temporarily (can reduce later)
- Check host storage I/O isn't bottlenecked
# pfSense Installation and Web UI Configuration

## Overview

This document covers the complete installation and initial configuration of pfSense Community Edition from ISO boot through final Web UI setup.

## Installation Process

### Prerequisites

- pfSense VM created in Proxmox (VM ID 100)
- Both network interfaces configured (WAN on vmbr0, LAN on vmbr1)
- pfSense 2.8.1 ISO attached to VM
- Proxmox host accessible at 192.168.1.100:8006

### Step-by-Step Installation

#### 1. Boot and License

When the VM boots, you will see:
```
Netgate Installer - v1.1.1-RELEASE
```

- Accept the EULA (End User License Agreement)
- Select "Install pfSense (Graphical)" option

#### 2. Confirm Installation Target

- The installer shows disk: `da0` (32GB QEMU HARDDISK)
- Click OK to proceed
- **WARNING:** This will erase all data on the disk (fresh VM, so safe)

#### 3. Select Installation ISO

- Choose pfSense 2.8.1 ISO (or latest stable version available)

#### 4. Configure Interfaces During Installation

**WAN Interface Selection:**
- Select: `inet0 (vtnet0)` - This is your first NIC (vmbr0)
- This will be your WAN interface connecting to home network

**WAN Network Mode:**
- Interface Mode: `DHCP (client)` - Gets IP automatically from home router
- VLAN Tagging: Disabled
- Use local resolver: false

**LAN Interface Selection:**
- Select: `vtnet1` - This is your second NIC (vmbr1)
- This will be your internal lab network interface

**LAN Network Mode:**
- Interface Mode: `STATIC`
- IP Address: `172.16.1.1/24`
- DHCP Enabled: `true`
- DHCP Range Start: `172.16.1.10`
- DHCP Range End: `172.16.1.199`

**Interface Assignment Confirmation:**
- LAN: vtnet1 ✓
- WAN: vtnet0 ✓
- Click Continue

#### 5. Select pfSense Edition

- Choose: **pfSense Community Edition (CE)** - Free, open-source
- Netgate also offers Plus edition (paid) - not needed for lab

#### 6. Installation Options

- File System: `ZFS (recommended default)`
- Partition Scheme: `GPT (Compatible with MBR)`

#### 7. ZFS Virtual Device Configuration

- Type: `Stripe - No Redundancy` (single disk, this is correct)

#### 8. Disk Selection

- Select: `da0 (32GB QEMU HARDDISK)`
- Confirm: "Yes, destroy the contents"

#### 9. Installation Complete

After installation completes:
```
Installation of pfSense complete! Would you like to reboot into 
the installed system now?
```

- Click **[Reboot]** to boot into the newly installed pfSense

### First Boot - Disable Firewall

When pfSense boots for the first time, you'll see the console menu:

```
*** Welcome to pfSense 2.8.1-RELEASE (amd64) on pfSense ***

WAN (wan) -> vtnet0 -> v4/DHCPv4: 192.168.1.101/24
LAN (lan) -> vtnet1 -> v4: 172.16.1.1/24

0) Logout / Disconnect SSH
1) Assign Interfaces
...
8) Shell

Enter an option: 8
```

Enter `8` to go to shell, then run:

```bash
pfctl -d
```

This disables the firewall temporarily, allowing web UI access:
```
pf disabled
```

---

## Web UI Configuration

### Access Web UI

Open your browser and navigate to:
```
https://192.168.1.101
```

**Initial Login (Default credentials):**
- Username: `admin`
- Password: `pfsense`

**Note:** SSL certificate warning is normal (self-signed); click "Accept" to proceed.

### Setup Wizard Steps

The Web UI will launch a setup wizard (9 steps total).

#### Step 1: General Information

Configure domain, DNS, and system identity.

**Values to Enter:**
```
Hostname:              pfSense
Domain:                lab.internal
Primary DNS Server:    172.16.1.10 (will be your DC-01)
Secondary DNS Server:  8.8.8.8 (Google DNS fallback)
Override DNS:          ☑️ Checked
```

**Why these values:**
- Hostname: Identifies the firewall
- Domain: Matches your AD domain (lab.internal)
- Primary DNS: Points to DC-01 (future Active Directory DNS)
- Secondary DNS: Public fallback for internet queries
- Override DNS: Allows DHCP to override DNS if needed

**Click: Next**

#### Step 2: Time Server Information

Configure NTP and timezone.

**Values to Enter:**
```
Time server hostname:  2.pfsense.pool.ntp.org (default - good)
Timezone:              Asia/Kolkata (for India/IST)
```

**Click: Next**

#### Step 3: Configure WAN Interface

Configure your external/management network interface.

**Values to Enter:**
```
Configuration Type:    DHCP
Block RFC1918:         ☐ UNCHECK (Important! Your WAN is 192.168.1.x)
Block bogons:          ☑️ Keep checked
```

**Why uncheck "Block RFC1918":**
- Your WAN is on private network (192.168.1.x from home router)
- If enabled, it would block traffic from your home network
- Unchecking allows proper WAN/LAN communication

**Click: Next**

#### Step 4: Configure LAN Interface

Configure your internal lab network interface.

**Values to Enter:**
```
LAN IP Address:        172.16.1.1
Subnet Mask:           24 (/24 = 255.255.255.0)
```

**These are already correct from installation!**

**Click: Next**

#### Step 5: Change Admin Password

Change from default password for security.

**Values to Enter:**
```
New admin Password:    [Strong password, e.g., Pfsen$e@Lab2025!]
Confirm admin Password: [Same password]
```

**Requirements:**
- Minimum 8 characters (recommend 12+)
- Mix of UPPERCASE, lowercase, numbers, symbols
- Different from default "pfsense"

**Click: Next**

#### Step 6: Reload Configuration

Apply all wizard changes to pfSense.

**Action:**
- Click **[Reload]** button
- This applies all settings and reloads pfSense services

#### Step 7: Finish (Dashboard)

After reload, you will be taken to the **Status / Dashboard** page.

### Network Interfaces

Go to **Interfaces** → **Assignments**

- **WAN (vtnet0):** Should show DHCP IP from home router (~192.168.1.101)
- **LAN (vtnet1):** Should show static 172.16.1.1/24

### DNS Configuration

Go to **System** → **General**

- **Primary DNS:** Should show 172.16.1.10
- **Secondary DNS:** Should show 8.8.8.8

### DHCP Server

Go to **Services** → **DHCP Server** → **LAN**

- **Enable DHCP Server on LAN:** Should be enabled
- **Range:** 172.16.1.10 - 172.16.1.199

### Firewall Status

Go to **System** → **Status** → **Firewall**

- Firewall should be **Enabled** (it was disabled for setup)

---

## Post-Installation Configuration

### Important Next Steps

1. **Enable Firewall (if disabled):**
   ```
   System → Status → Firewall
   Click "Enable" if showing "pf disabled"
   ```

2. **Set up firewall rules** (recommended later):
   - Firewall → Rules (LAN, WAN, etc.)
   - Restrict traffic as needed for your lab

3. **Configure additional services** (optional):
   - Services → OpenVPN (for VPN access)
   - Services → DNS Resolver (for DNS forwarding)
   - Services → DHCP Server (already configured)

4. **Backup configuration:**
   - Diagnostics → Backup & Restore
   - Download configuration regularly for safety

---

## Common Issues & Solutions

### Issue: Cannot access Web UI at https://192.168.1.101

**Solution:**
- Ensure firewall is disabled (`pfctl -d` from console)
- Verify WAN has IP address (check console interface status)
- Try http://192.168.1.101 (not https) temporarily
- Check browser SSL certificate warning acceptance

### Issue: LAN VMs cannot reach WAN/internet

**Solution:**
- Verify pfSense firewall is **enabled** after setup wizard
- Check LAN firewall rules allow outbound traffic
- Verify default gateway on lab VMs is 172.16.1.1
- Check DNS is set to 172.16.1.1 or DC-01 (172.16.1.10)

### Issue: DHCP not working for lab VMs

**Solution:**
- Go to Services → DHCP Server → LAN
- Verify DHCP Server is **Enabled**
- Check Range: 172.16.1.10 - 172.16.1.199
- Restart DHCP: Services → DHCP Server → Restart DHCP Server

### Issue: Cannot resolve domain names (DNS failing)

**Solution:**
- Verify Primary DNS is set to 172.16.1.10 (DC-01) in System → General
- Secondary DNS to 8.8.8.8 for internet queries
- Once DC-01 is running and has DNS, it will be your primary
- Temporarily use 8.8.8.8 as primary if DC-01 not running

---

## Configuration Summary

Your pfSense firewall is now configured with:

```
pfSense Configuration Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Hostname:           pfSense.lab.internal
Domain:             lab.internal
Version:            2.8.1-RELEASE (amd64)

WAN Interface (vtnet0):
  IP:               192.168.1.101/24 (DHCP)
  Gateway:          192.168.1.1
  Purpose:          Management/Home network access

LAN Interface (vtnet1):
  IP:               172.16.1.1/24 (Static)
  Purpose:          Lab internal network gateway
  DHCP Pool:        172.16.1.10 - 172.16.1.199

DNS Configuration:
  Primary:          172.16.1.10 (DC-01 - when running)
  Secondary:        8.8.8.8 (Google DNS)

Firewall Status:    Enabled
Admin User:         admin (password changed)
Web UI URL:         https://192.168.1.101

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Network Architecture

```
┌─────────────────────────────────────────────────┐
│                 Home Network                     │
│              192.168.1.0/24                      │
│         (Your Home Router, Internet)             │
└──────────────────────┬──────────────────────────┘
                       │
              ┌────────▼────────┐
              │  pfSense VM 100 │
              │ (Firewall/GW)   │
              └────────┬────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
   WAN (vtnet0)               LAN (vtnet1)
   vmbr0 DHCP                vmbr1 172.16.1.1/24
   192.168.1.101             Gateway for lab
        │                             │
        │                   ┌─────────▼─────────┐
        │                   │  Lab Network      │
        │                   │ 172.16.1.0/24     │
        │                   │ (DC-01, etc)      │
        │                   └───────────────────┘
```
