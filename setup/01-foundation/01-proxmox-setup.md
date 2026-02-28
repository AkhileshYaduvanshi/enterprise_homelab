# Proxmox - Infrastructure Base

This section covers the bare metal installation and initial configuration of Proxmox VE, which serves as the foundation for your entire lab infrastructure.

### Overview

Proxmox VE (PVE) is a powerful, open-source virtualization platform built on Debian Linux. It provides:
- KVM hypervisor for virtual machine management
- LXC containers for lightweight containerization
- High availability and clustering support
- Web-based management interface

This guide documents the complete installation of Proxmox VE 9.1 on bare metal hardware using the graphical installer.

Step 1: Boot and Accept EULA

Step 2: Select Installation Target (Storage)

Step 3: Configure Location and Time Zone

Step 4: Set Root Password and Email Address

Step 5: Configure Management Network

This is one of the most important steps - it configures how you access Proxmox.

**Configuration Used:**

Management Interface: nic0 (virtio_net) - mac address
Hostname (FQDN): pve.lab.internal
IP Address (CIDR): 192.168.1.100 / 24
Gateway: 192.168.1.1
DNS Server: 192.168.1.1
Pin network interface names: ✓ Checked


Step 6: Review Summary and Install

Step 7: Installation Process

Step 8: First Boot - Proxmox Started Successfully

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
