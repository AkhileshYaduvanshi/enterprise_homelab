# pfSense Installation and Web UI Configuration

## Overview

This document covers the complete installation and initial configuration of pfSense 2.8.1 Community Edition from ISO boot through final Web UI setup.

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
