## Step-by-Step VM Creation Process

### Step 1: Open VM Creation Wizard
1. Navigate to Proxmox Web UI (`https://192.168.1.100:8006`)
2. Select node `pve` from the left sidebar
3. Click **"Create VM"** button in the top-right corner

### Step 2: General Settings Tab
Configure the basic VM parameters:
- **Node:** pve
- **VM ID:** 101 (Between pfSense-100 and DC-01-102)
- **Name:** bastionHost
- **Resource Pool:** Leave empty (default)
- **Add to HA:** Unchecked
- Click **"Next"**

### Step 3: OS Settings Tab
Select the Debian installation media:
- **Use CD/DVD disc image file (iso):** Selected ✓
- **Storage:** local
- **ISO Image:** debian-12.0.0-amd64-netinst.iso
  - **Why netinst?** Network installer downloads only required packages, keeping VM lightweight (773.85 MB vs 5+ GB for full ISO)
- **Guest OS Type:** Linux
- **Version:** 6.x - 2.6 Kernel
- Click **"Next"**

### Step 4: System Settings Tab
Configure system hardware features:
- **Graphic Card:** Default
- **Machine:** **q35** (Modern chipset - recommended)
  - Better hardware compatibility
  - Supports modern features like PCIe
  - Future-proof for OS updates
- **SCSI Controller:** VirtIO SCSI single
- **Qemu Agent:** **Enabled** ✓ (Important!)
  - Provides better VM shutdown/reboot handling
  - Enables proper monitoring
  - Required for IP address visibility in Proxmox UI
- **BIOS:** Default (SeaBIOS)
- **Add TPM:** Unchecked
- Click **"Next"**

### Step 5: Disks Settings Tab
Configure virtual disk:
- **Bus/Device:** SCSI 0
- **Storage:** local-lvm
- **Disk size (GiB):** **20** (Increased from default 10 GB)
  - Base OS: ~3-5 GB
  - Tools and packages: ~2-3 GB
  - Logs and configs: ~2-3 GB
  - Buffer for growth: ~10 GB
- **Cache:** Default (No cache)
- **Discard:** **Enabled** ✓
  - Enables TRIM support
  - Improves storage efficiency
  - Reclaims unused space
- **IO thread:** **Enabled** ✓
  - Better disk I/O performance
  - Reduces CPU overhead for disk operations
- **Format:** Raw disk image (raw)
- Click **"Next"**

### Step 6: CPU Settings Tab
Configure processor allocation:
- **Sockets:** 1
- **Cores:** 1
- **Type:** x86-64-v2-AES
  - AES support beneficial for SSH encryption
  - Good balance of compatibility and features
- **Total cores:** 1

**Why 1 vCPU?**
- Jump host workload is primarily I/O and network-bound
- SSH sessions don't require heavy CPU processing
- Resource-efficient for 16GB RAM lab environment
- Can be increased later if needed

Click **"Next"**

### Step 7: Memory Settings Tab
Configure RAM allocation:
- **Memory (MiB):** 512

**Why 512 MB?**
- Debian 12 is extremely lightweight
- Sufficient for SSH daemon and multiple sessions
- Conservative approach: can be increased to 1GB if performance issues arise
- Leaves maximum resources for other lab VMs

Click **"Next"**

### Step 8: Network Settings Tab
Configure network interface:
- **No network device:** Unchecked (Add network interface)
- **Bridge:** vmbr1 (Internal LAN)
  - Jump host should NOT be directly exposed to WAN
  - Accessible only from internal network
- **VLAN Tag:** no VLAN
- **Model:** VirtIO (paravirtualized)
  - Best network performance for Linux guests
  - Lower CPU overhead
- **MAC address:** auto (Proxmox generates unique MAC)
- **Firewall:** Enabled ✓ (Additional security layer)
- Click **"Next"**

### Step 9: Confirm Settings Tab
Review the complete configuration summary:

```
cores:      1
cpu:        x86-64-v2-AES
ide2:       local:iso/debian-12.0.0-amd64-netinst.iso,media=cdrom
machine:    q35
memory:     512
name:       bastionHost
net0:       virtio,bridge=vmbr1
nodename:   pve
numa:       0
ostype:     l26
scsi0:      local-lvm:25,discard=on,iothread=on
scsihw:     virtio-scsi-single
sockets:    1
vmid:       101
```

- **Start after created:** Leave unchecked (We'll start manually after reviewing)
- Click **"Finish"** to create the VM


The Jump Host Layer documentation is now complete! This provides a comprehensive record of the VM creation process. When you're ready, we can start the VM and proceed with Debian installation.
