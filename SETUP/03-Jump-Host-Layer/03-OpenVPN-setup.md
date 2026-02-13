# OpenVPN Remote Access Setup

## Overview

This guide covers the configuration of OpenVPN on pfSense to enable secure remote access to the internal lab network (172.16.1.0/24). This allows SSH and management access to all lab VMs from external networks.

## What We Accomplished

✅ OpenVPN Server configured on pfSense  
✅ Certificate Authority and certificates created  
✅ VPN user account with certificate generated  
✅ Client export package installed  
✅ `.ovpn` configuration file ready for download  
✅ VPN tested and working - able to ping internal network

***

## OpenVPN Server Configuration

### Step 1: Start OpenVPN Wizard

**Navigate to:** VPN → OpenVPN → Wizards

**Selected:** OpenVPN Remote Access Server Setup

***

### Step 2: Authentication Backend

**Type of Server:** Local User Access

- Uses pfSense's local user database
- Simple for lab environments
- No external authentication required

**Action:** Click Next

***

### Step 3: Certificate Authority (CA)

**Created new CA with:**

| Setting | Value |
|---------|-------|
| Descriptive name | lab-vpn-ca |
| Randomize Serial | ✓ Enabled |
| Key length | 2048 bit |
| Lifetime | 3650 days |
| Common Name | pfSense.lab.internal |
| Country/State/City | (left blank) |

**Action:** Click Add new CA

***

### Step 4: Server Certificate

**Created server certificate with:**

| Setting | Value |
|---------|-------|
| Descriptive name | lab-vpn-server |
| Key length | 2048 bit |
| Lifetime | 398 days |
| Common Name | pfSense.lab.internal |
| Geographic fields | (left blank) |

**Action:** Click Add new Certificate

***

### Step 5: General Server Settings

#### Endpoint Configuration
- **Protocol:** UDP on IPv4 only
- **Interface:** WAN
- **Local Port:** 1194 (standard OpenVPN port)

#### Cryptographic Settings
- **TLS Authentication:** ✓ Enabled
- **Generate TLS Key:** ✓ Automatically generate

***

### Step 6: Tunnel Network Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| **IPv4 Tunnel Network** | 10.0.0.0/24 | VPN client IP pool |
| **Redirect IPv4 Gateway** | ☐ Unchecked | Don't force all traffic through VPN |
| **IPv4 Local Network** | 172.16.1.0/24 | Internal lab network access |
| **Concurrent Connections** | (empty) | Unlimited connections |

#### Security Settings
- **Allow Compression:** Refuse any non-stub compression (Most secure)
- **Compression:** Disable Compression
- **Inter-Client Communication:** ☐ Disabled
- **Duplicate Connections:** ☐ Disabled

***

### Step 7: Client DNS & Time Settings

#### DNS Configuration
```
DNS Default Domain:  pfSense.lab.internal
DNS Server 1:        172.16.1.10 (DC-01)
DNS Server 2:        8.8.8.8 (Backup)
```

**Purpose:** VPN clients can resolve `lab.internal` domain names

#### NTP Configuration
```
NTP Server:  172.16.1.10 (DC-01)
```

#### NetBIOS/WINS
- Not configured (not needed for our lab)

**Action:** Click Next

***

### Step 8: Firewall Rules

**Two rules created automatically:**

1. **✓ WAN Rule:** Permit OpenVPN connections from anywhere on Internet
   - Allows UDP 1194 inbound on WAN interface

2. **✓ OpenVPN Rule:** Allow all traffic from VPN clients through tunnel
   - Allows VPN clients to access 172.16.1.0/24 network

**Action:** Click Next

***

### Step 9: Completion

**OpenVPN Server Created Successfully:**

| Setting | Value |
|---------|-------|
| Interface | WAN |
| Protocol/Port | UDP4 / 1194 (TUN) |
| Tunnel Network | 10.0.0.0/24 |
| Local Network | 172.16.1.0/24 |
| Mode | Remote Access (SSL/TLS + User Auth) |
| Encryption | AES-256-GCM, AES-128-GCM, CHACHA20-POLY1305, AES-256-CBC |
| Digest | SHA256 |
| D-H Params | 2048 bits |

**Verify:** VPN → OpenVPN → Servers shows the configured server

***

## VPN User Creation

### Step 1: Create User Account

**Navigate to:** System → User Manager → Users → Add

**User Configuration:**
```
Username:      labadmin
Password:      (set during creation)
Full name:     (optional)
Disabled:      ☐ Unchecked
Expiration:    (blank - no expiration)
Group:         (not assigned to admins)
```

***

### Step 2: Generate User Certificate

**In same user creation form:**

**Certificate Settings:**
```
✓ Click to create a user certificate

Descriptive name:      labadmin-vpn-cert
Certificate authority: lab-vpn-ca
Key type:             RSA
Key length:           2048
Digest Algorithm:     sha256
Lifetime:             3650 days
```

**Action:** Click Save

**Result:** User `labadmin` created with certificate `labadmin-vpn-cert`

***

## Client Export Package Installation

### Why Install This Package?

Provides easy download of pre-configured `.ovpn` files with embedded certificates.

### Installation Steps

1. **Navigate to:** System → Package Manager → Available Packages

2. **Search for:** `openvpn-client-export`

3. **Found Package:**
   - Name: openvpn-client-export
   - Version: 1.9.6
   - Description: Exports pre-configured OpenVPN Client configurations

4. **Dependencies (auto-installed):**
   - openvpn-client-export-2.6.17
   - openvpn-2.6.16
   - zip-3.0_3
   - 7-zip-24.09

5. **Action:** Click Install button

6. **Wait:** 1-2 minutes for installation

***

## Client Configuration Export

### Step 1: Access Client Export Utility

**Navigate to:** VPN → OpenVPN → Client Export

(This tab appears after package installation)

***

### Step 2: Download Configuration

**OpenVPN Clients Section:**
```
User:              labadmin
Certificate Name:  labadmin-vpn-cert
```

**Available Download Options:**

#### Inline Configurations
- **Most Clients** - Standard .ovpn file (recommended)
- **Android** - Android-optimized
- **OpenVPN Connect (iOS/Android)** - For Connect app

#### Bundled Configurations
- **Archive** - ZIP with separate files
- **Config File Only** - Configuration only

#### Windows Installers
- **64-bit** - Windows installer (2.6.17-lx001)
- **32-bit** - 32-bit Windows installer

#### Mac/Linux
- **Viscosity Bundle** - For Viscosity client
- **Viscosity Inline Config** - Viscosity format

***

### Step 3: Downloaded File

**Recommended:** Click "Most Clients" button

**File Downloaded:** `pfSense-UDP4-1194-labadmin-install-labadmin-vpn-cert.ovpn`

**File Contains (all embedded):**
- Client configuration
- CA certificate
- Client certificate
- Client private key
- TLS authentication key
- Server connection details
- DNS/routing settings

***

## Connection & Testing

### Connect to VPN

**Method depends on your OS:**
- Windows: OpenVPN GUI or Windows installer
- Linux: `sudo openvpn --config <file>.ovpn`
- Mac: Tunnelblick or OpenVPN Connect

### Verify Connection

**Check VPN IP Assignment:**
```bash
# Linux/Mac
ip addr show tun0
# Should show: inet 10.0.0.x/24

# Windows
ipconfig
# Should show: OpenVPN TAP adapter with 10.0.0.x
```

**Test Internal Network Access:**
```bash
# Ping pfSense LAN
ping 172.16.1.1

# Ping DC-01
ping 172.16.1.10

# Ping Bastion Host
ping 172.16.1.20
```

**Result:** ✅ All pings successful - VPN working!

***

## Network Architecture

```
Your Home PC
    ↓
VPN Client (10.0.0.x)
    ↓ UDP 1194 (encrypted tunnel)
pfSense WAN (192.168.1.101)
    ↓
pfSense OpenVPN Server
    ↓ VPN Tunnel
pfSense LAN (172.16.1.1)
    ↓
Internal Lab Network (172.16.1.0/24)
    ├── DC-01: 172.16.1.10
    ├── Bastion: 172.16.1.20
    └── Future VMs
```

***

## Security Configuration Summary

**Encryption:**
- Data Ciphers: AES-256-GCM, AES-128-GCM, CHACHA20-POLY1305
- Auth Digest: SHA256
- TLS Authentication: Enabled
- Key Size: 2048-bit RSA

**Authentication:**
- Certificate-based (client cert required)
- User/password from pfSense local database

**Network Protection:**
- WAN firewall rule limits access to UDP 1194 only
- VPN clients isolated (no inter-client communication)
- Compression disabled (security best practice)

***

## What's Now Possible

✅ **SSH Access from Terminal:**
```bash
ssh labadmin@172.16.1.20  # Bastion Host
ssh administrator@172.16.1.10  # DC-01 (if SSH configured)
```

✅ **Copy/Paste Works:** No more noVNC limitations

✅ **Use Your Tools:** Your preferred terminal, SSH client, tools

✅ **Multiple Connections:** Connect to multiple VMs simultaneously

✅ **Remote Access:** Access your lab from anywhere with internet
