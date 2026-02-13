## Layer 6: Wazuh Security Monitoring - Installation & Configuration

### Installation Date & Details
- **Wazuh Version:** 4.7.5
- **Base OS:** Ubuntu 24.04.3 LTS (minimized)
- **VM ID:** 103 (Wazuh)
- **IP Address:** 172.16.1.50/24
- **Network:** vmbr1 (Internal Lab Network)

### System Configuration

#### Ubuntu 24.04 LTS Installation
**Installation Type:**
- Selected: Ubuntu Server (minimized) installation
- Reason: Minimal footprint, optimized for security appliances

**Network Configuration:**
```
Interface: enp6s18
IP Address: 172.16.1.50
Subnet Mask: 255.255.255.0 (/24)
Gateway: 172.16.1.1
DNS Server: 172.16.1.1
Domain: lab.internal
Configuration Type: Static IP (Manual)
```

**Storage Configuration:**
```
Disk Size: 50 GB
Filesystem: ext4 with LVM
Layout:
  /boot: 2 GB (ext4)
  /: 23.996 GB (LVM logical volume)
  Available Free: ~24 GB
Controller: VirtIO SCSI single
Options: Discard enabled, IO Thread enabled
```

**User Account Created:**
```
Username: labadmin
Full Name: labadmin
SSH: Enabled with password authentication
Sudo: Enabled
```

#### System Updates Applied
```bash
# Commands executed post-installation:
su -
apt update
apt upgrade -y
apt install qemu-guest-agent curl wget gnupg -y
systemctl enable qemu-guest-agent
systemctl start qemu-guest-agent
```

### Wazuh Installation Process

#### Installation Commands
```bash
# Download Wazuh installation script
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# Run all-in-one installation with ignore-check for Ubuntu 24.04
bash ./wazuh-install.sh -a -i
```

**Note:** Used `-i` flag to bypass system compatibility check
- Wazuh script only officially lists Ubuntu up to 22.04
- Ubuntu 24.04 is fully compatible with Wazuh 4.7.5
- The compatibility check is conservative for support purposes

#### Installation Components
The all-in-one installation (`-a` flag) installs:

1. **Wazuh Manager**
   - Central management and analysis server
   - Receives alerts from agents
   - Policy management and rule configuration
   - Status: ✅ Installed and Running

2. **Wazuh Indexer**
   - Elasticsearch-based data storage
   - Stores all alerts and event data
   - Handles indexing and retention
   - Status: ✅ Installed and Running

3. **Filebeat**
   - Log forwarding tool
   - Forwards manager logs to indexer
   - Lightweight agent for log collection
   - Status: ✅ Installed and Running

4. **Wazuh Dashboard**
   - Web-based user interface (Kibana-based)
   - Visualization of security alerts
   - Event search and analysis
   - Configuration management interface
   - Status: ✅ Installed and Running


### Access Credentials

**Dashboard Access:**
```
URL: https://172.16.1.50:443/
Username: admin
Password: Pla+6+sH4KscR6OHPA3l9SQ.X*IcD?Mj
Port: 443
```

**Access from Lab Machines:**
- From Bastion Host: `https://172.16.1.50`
- From Domain Controller: `https://172.16.1.50`
- From any VM on 172.16.1.0/24 network

⚠️ **Important Security Notes:**
- Dashboard uses self-signed SSL certificate
- Accept certificate warning when accessing
- Change default admin password after initial access
- Credentials stored in `/root/wazuh-install-files.tar.gz`

