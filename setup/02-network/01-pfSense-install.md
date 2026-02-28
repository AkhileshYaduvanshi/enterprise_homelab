# pfSense VM Creation in Proxmox

pfSense is a free, open-source firewall and router based on FreeBSD. It provides enterprise-grade features for your lab:
- Firewall with stateful filtering
- Network address translation (NAT)
- VPN (OpenVPN, WireGuard)
- DHCP server
- DNS forwarding
- Advanced routing


### pfSense VM Creation in Proxmox

CPU: 1 Core (Not 2)
- **Why 1 core:** pfSense is I/O-bound, not CPU-bound
- **Packet processing:** Single core handles routing/filtering efficiently
- **Resource savings:** Leaves more cores for DC-01, Wazuh, Jenkins
- **Lab environment:** Sufficient for firewall duties in internal network

**Note:** If you add heavy VPN usage or advanced rules, can increase to 2 cores later.

Memory: 1 GB
- **Base pfSense:** 256-512 MB minimum
- **VPN Support:** +256-512 MB for encryption/decryption
- **Logs & State Tables:** +256 MB for connections tracking
- **Total:** 1 GB is optimal for lab firewall with VPN
- **Headroom:** Not excessive, balanced for your 16GB system

Storage: 32 GB
- **pfSense install:** ~2-3 GB
- **Logs:** Room for 6-12 months of logs (depending on traffic)
- **RRD graphs:** Historical traffic graphs stored locally
- **Backups:** Configuration backups (optional)
- **Safe margin:** 32 GB ensures no disk space issues

Network Interfaces: Two VirtIO NICs
- **net0 (WAN):** vmbr0 → Connects to management network (192.168.1.x)
  - Gets IP from home router or static assignment
  - Communicates with Proxmox host and external networks
  
- **net1 (LAN):** vmbr1 → Connects to lab network (172.16.1.x)
  - Will be configured as 172.16.1.1 (gateway)
  - All lab VMs (DC-01, etc) use this as default gateway

**VirtIO Model:** Best performance for network I/O, low overhead

### Troubleshooting

Network interfaces not visible
- Ensure bridges (vmbr0, vmbr1) exist in Proxmox Network configuration
- Verify network bridges are properly configured on the host


## Installation Process

### Prerequisites

- pfSense VM created in Proxmox (VM ID 100)
- Both network interfaces configured (WAN on vmbr0, LAN on vmbr1)
- pfSense 2.8.1 ISO attached to VM
- Proxmox host accessible at 192.168.1.100:8006

### Step-by-Step Installation

1. Boot and License
2. Confirm Installation Target
3. Select Installation ISO
4. Configure Interfaces During Installation
5. Select pfSense Edition
6. Installation Options
7. ZFS Virtual Device Configuration
8. Disk Selection
9. Installation Complete

### First Boot - Disable Firewall

Enter `8` to go to shell, then run:

```bash
pfctl -d
```

This disables the firewall temporarily, allowing web UI access:
```
pf disabled
```

## Web UI Configuration

### Access Web UI

Open your browser and navigate to:
```
https://192.168.1.101
```

**Initial Login (Default credentials):**
- Username: `admin`
- Password: `pfsense`

### Setup Wizard Steps

Step 1: General Information

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

Step 2: Time Server Information

Step 3: Configure WAN Interface

Step 4: Configure LAN Interface

Step 5: Change Admin Password

Step 6: Reload Configuration

Step 7: Finish (Dashboard)

### Firewall Status

Go to **System** → **Status** → **Firewall**

- Firewall should be **Enabled** (it was disabled for setup)

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

## Common Issues & Solutions

Issue: Cannot access Web UI at https://192.168.1.101

**Solution:**
- Ensure firewall is disabled (`pfctl -d` from console)
- Verify WAN has IP address (check console interface status)
- Try http://192.168.1.101 (not https) temporarily
- Check browser SSL certificate warning acceptance

Issue: LAN VMs cannot reach WAN/internet

**Solution:**
- Verify pfSense firewall is **enabled** after setup wizard
- Check LAN firewall rules allow outbound traffic
- Verify default gateway on lab VMs is 172.16.1.1
- Check DNS is set to 172.16.1.1 or DC-01 (172.16.1.10)

Issue: DHCP not working for lab VMs

**Solution:**
- Go to Services → DHCP Server → LAN
- Verify DHCP Server is **Enabled**
- Check Range: 172.16.1.10 - 172.16.1.199
- Restart DHCP: Services → DHCP Server → Restart DHCP Server

Issue: Cannot resolve domain names (DNS failing)

**Solution:**
- Verify Primary DNS is set to 172.16.1.10 (DC-01) in System → General
- Secondary DNS to 8.8.8.8 for internet queries
- Once DC-01 is running and has DNS, it will be your primary
- Temporarily use 8.8.8.8 as primary if DC-01 not running
