# Proxmox VE 9.1 - Bare Metal Installation & Configuration

## Installation Overview

This guide documents the complete installation of Proxmox VE 9.1 on bare metal hardware using the graphical installer.

**Proxmox Version:** 9.1 (ISO 1)
**Installation Mode:** Graphical Installer
**Target OS:** Debian-based Linux with KVM hypervisor

---

## Step 1: Boot and Accept EULA

### What Happens
- System boots from Proxmox VE 9.1 ISO
- Installer loads the graphical environment
- Hardware virtualization warning appears (expected in nested/lab setups)

### Actions
1. System automatically initializes the installer
2. Accept the warning about hardware acceleration
3. Review the End User License Agreement (EULA)
4. Click **I Agree** to continue

**Note:** The EULA is lengthy and covers:
- GNU Affero General Public License v3 (AGPLv3)
- Limited warranty disclaimer
- Intellectual property rights
- Third-party software licensing

---

## Step 2: Select Installation Target (Storage)

### Screen: "Installation Target"

The installer shows the disk(s) available for installation.

**Configuration Used:**

Target Harddisk: /dev/sda (100.00 GiB, QEMU HARDDISK)

**Important Notes:**
⚠️ **WARNING:** All existing partitions and data on the selected disk will be destroyed!

### What the Installer Does
- Automatically partitions the disk
- Creates LVM logical volumes for:
  - Root filesystem
  - Swap space
  - Data volumes
- Installs all required packages
- Makes the system bootable

### Actions
1. Verify the correct disk is selected (usually `/dev/sda`)
2. Review available storage in the dropdown (if multiple disks exist)
3. Click **Next** to proceed

---

## Step 3: Configure Location and Time Zone

### Screen: "Location and Time Zone Selection"

The installer sets up localization and keyboard layout for your system.

**Configuration Used:**

Country: India
Timezone: Asia/Kolkata
Keyboard Layout: U.S. English

### Why This Matters
- **Timezone:** Ensures system time is correct (Asia/Kolkata is IST UTC+5:30)
- **Keyboard Layout:** Sets keyboard input method (en-us = standard US QWERTY)
- **Country:** Used for NTP server selection and locale settings

### Actions
1. Select **Country:** India (or your location)
2. Select **Timezone:** Asia/Kolkata (or your timezone)
3. Select **Keyboard Layout:** U.S. English (or your preferred layout)
4. Click **Next** to continue

---

## Step 4: Set Root Password and Email Address

### Screen: "Administration Password and Email Address"

This step configures critical credentials for Proxmox administration.

**Fields to Fill:**

#### **Password**
- **Length:** Minimum 8 characters
- **Required:** Mix of UPPERCASE, lowercase, numbers, symbols
- **Example:** `Proxmox@Lab2025!`
- **Security:** This is the `root` password - use a strong, unique password
- **Storage:** Keep this password safe (write it down or store in a password manager)

#### **Confirm Password**
- Re-enter the same password to verify it matches

#### **Email Address**
- **Format:** Valid email address
- **Purpose:** Proxmox sends critical alerts, backup notifications, and high-availability events to this email
- **Example:** `admin@yourdomain.com` or `your.email@gmail.com`

**Configuration Used:**

Password: [Your secure password]
Confirm: [Same password]
Email: admin@lab.internal

### Actions
1. Enter a strong root password
2. Re-enter the same password in **Confirm** field
3. Enter a valid email address
4. Click **Next** to continue

---

## Step 5: Configure Management Network

### Screen: "Management Network Configuration"

This is one of the most important steps - it configures how you access Proxmox.

**Configuration Used:**

Management Interface: nic0 (virtio_net) - bc:24:11:60:a1:d5
Hostname (FQDN): pve.lab.internal
IP Address (CIDR): 192.168.1.100 / 24
Gateway: 192.168.1.1
DNS Server: 192.168.1.1
Pin network interface names: ✓ Checked


### Understanding These Fields

#### **Management Interface**
- The network adapter Proxmox will use for the web UI and management traffic
- Example: `nic0 - bc:24:11:60:a1:d5 (virtio_net)`
- Usually the first (or only) NIC in the system

#### **Hostname (FQDN)**
- **FQDN:** Fully Qualified Domain Name
- **Your Choice:** `pve.lab.internal`
- **Breakdown:**
  - `pve` = short hostname
  - `lab.internal` = domain name
- **Used For:** System identification, SSL certificates, DNS resolution

#### **IP Address (CIDR)**
- **Your IP:** `192.168.1.100`
- **CIDR Notation:** `/24` = subnet mask 255.255.255.0
- **Network Range:** 192.168.1.0 - 192.168.1.255
- **Usable IPs:** 192.168.1.1 to 192.168.1.254

#### **Gateway**
- **Your Gateway:** `192.168.1.1`
- **Purpose:** IP address of your router/firewall
- **Used For:** Routing traffic outside the local subnet
- **In Your Lab:** This is your home router or pfSense firewall

#### **DNS Server**
- **Your DNS:** `192.168.1.1`
- **Purpose:** Domain name resolution
- **In Your Lab:** Typically your router acts as DNS, or can point to a future DNS server
- **Note:** Later, once you set up DC-01 (domain controller), this can point to the AD DNS

#### **Pin Network Interface Names**
- **Checked ✓** = Use consistent interface naming (nic0, nic1, etc.)
- **Uncovered ☐** = Use traditional naming (eth0, eth1, etc.)
- **Recommended:** Keep checked for consistency

### Actions
1. Keep **Management Interface** at default (nic0)
2. Set **Hostname:** `pve.lab.internal` (or your choice)
3. Set **IP Address:** `192.168.1.100` (adjust if you use different range)
4. Set **Gateway:** `192.168.1.1` (your router/firewall IP)
5. Set **DNS Server:** `192.168.1.1` (or your DNS server)
6. Ensure **Pin network interface names** is checked
7. Click **Next** to continue

---

## Step 6: Review Summary and Install

### Screen: "Summary"

A final review of all installation settings before partitioning begins.

**Summary Table - Your Configuration:**

| Option | Value |
|--------|-------|
| Filesystem | ext4 |
| Disk(s) | /dev/sda |
| Country | India |
| Timezone | Asia/Kolkata |
| Keymap | en-us |
| Email | admin@lab.internal |
| Management Interface | nic0 |
| Hostname | pve |
| IP CIDR | 192.168.1.100/24 |
| Gateway | 192.168.1.1 |
| DNS | 192.168.1.1 |
| Auto-reboot | ✓ Checked |

### What Happens Next
**⚠️ WARNING:** Clicking **Install** will:
1. Partition and format `/dev/sda` - **ALL DATA WILL BE LOST**
2. Create LVM logical volumes
3. Install Debian + Proxmox packages
4. Configure bootloader (GRUB2)
5. Automatically reboot when complete

### Actions
1. Review all settings carefully
2. Ensure **"Automatically reboot after successful installation"** is checked
3. Click **Install** button to begin installation
4. **Wait** for installation to complete (10-20 minutes depending on hardware/IO)

---

## Step 7: Installation Process

### What Happens During Installation

The system will:
1. Boot from ISO and load the installer
2. Partition the disk using LVM
3. Format filesystems (ext4)
4. Mount volumes
5. Extract and install packages from ISO
6. Configure bootloader (GRUB2)
7. Create initial system configuration
8. Reboot automatically when complete

### Estimated Time
- **Total Installation:** 10-30 minutes (depends on disk speed, CPU, RAM)
- **Reboot:** 2-5 minutes until Proxmox fully starts
- **First Boot:** Proxmox services initialize (5-10 minutes)

---

## Step 8: First Boot - Proxmox Started Successfully

### What You'll See

After reboot, the system will boot into Proxmox VE. You'll see:

Welcome to the Proxmox Virtual Environment. Please use your web browser to
configure this server - connect to:

https://192.168.1.100:8006/

pve login:

### Next Steps - Access Proxmox Web UI

1. **Open a web browser** on your management computer
2. **Navigate to:** `https://192.168.1.100:8006/`
3. **Accept SSL certificate warning** (self-signed certificate is normal)
4. **Login with:**
   - **Realm:** Proxmox (pam)
   - **Username:** root
   - **Password:** [Your root password from Step 4]

---

## Important Information to Remember

### Root Password
- **Username:** `root`
- **Password:** [Password you set in Step 4]
- **Storage:** Keep this secure!
- **Recovery:** No way to reset without console access


---

## Troubleshooting

### Issue: "No support for hardware-accelerated KVM virtualization detected"
**Solution:** This is normal in nested/lab setups. Continue with installation. Nested virtualization will work, just with reduced performance.

### Issue: Installation hangs during boot
**Solution:** 
- Increase VM resources (vCPU, RAM)
- Check disk space availability
- Try installing with Terminal UI instead of graphical

### Issue: Cannot access web UI at https://192.168.1.100:8006/
**Solution:**
- Verify IP address is correct (login prompt shows IP on boot)
- Ensure network connectivity (ping 192.168.1.100 from management computer)
- Check firewall rules allow HTTPS (port 8006)
- Wait 5 minutes for Proxmox services to fully initialize

### Issue: Forgot root password
**Solution:**
- Boot system to GRUB menu
- Edit kernel parameters to boot in single-user mode
- Reset root password using `passwd` command
- (Requires physical/console access)

---

## What's Next

After Proxmox installation is complete:

1. **Access Web UI** at `https://192.168.1.100:8006/`
2. **Configure Network Bridges:**
   - `vmbr0` → Management network (192.168.1.x)
   - `vmbr1` → Lab network (172.16.1.x)
3. **Set Up Storage Pools** for VM disks
4. **Upload ISOs** for next layer VMs
5. **Begin VM creation** for network, identity, and security layers
