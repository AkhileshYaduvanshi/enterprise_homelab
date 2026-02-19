# Windows Server 2022 Core Installation on DC-01

## Overview

This document covers the complete installation of Windows Server 2022 Core on DC-01 VM, including VirtIO driver installation for optimal performance.

## Installation Process

### Step 1: Start VM and Boot from ISO

1. In Proxmox, select **VM 102 (DC-01)** and click **Start**
2. Open **Console** tab
3. VM will boot from Windows Server 2022 ISO
4. Press a key when prompted: "Press any key to boot from CD or DVD…"

### Step 2: Language and Installation Setup

1. Select your **Language**: English (or your preference)
2. Click **Next**
3. Click **Install now**

### Step 3: Accept License

1. Read the Microsoft Software License Terms
2. Check **"I accept the license terms"**
3. Click **Next**

### Step 4: Select Installation Type

1. Choose **"Custom: Install Windows only (advanced)"** (fresh VM, so clean install)

### Step 5: Select Disk and Load VirtIO Storage Driver

**This is critical for proper disk performance.**

1. On the **"Where do you want to install Windows?"** screen, you will see the disk list
2. **If disk is NOT showing:**
   - Click **"Load driver"**
   - Point to VirtIO ISO CD drive (usually `E:` or `F:`)
   - Navigate to: `vioscsi\2k22\amd64\`
   - Select and load the VirtIO SCSI driver
   - The `32 GB` disk should now appear
3. **Select the disk** and click **Next**

### Step 6: Installation and First Boot

1. Windows Server 2022 Core will now install
2. This takes 5-10 minutes; wait for completion
3. VM will reboot automatically
4. After reboot, you'll see "Your password has been changed"

### Step 7: Set Administrator Password

1. You will be prompted for Administrator password
2. Enter a **strong password**:
   - Minimum 8 characters (recommend 12+)
   - UPPERCASE + lowercase + numbers + symbols
   - Example: `DC@dmin2025!Lab`
3. Press **Enter**

### Step 8: Server Core Console Menu

After password is set and VM reboots fully, you will see:

```
Microsoft Windows [Version 10.0.20348]

Administrator
pve login: Administrator
Copyright (c) Microsoft Corporation. All rights reserved.

C:\Users\Administrator>_
```

This is **Server Core** - CLI-only, no GUI.

---

## Post-Installation Configuration

### Install VirtIO NIC Driver

The network adapter needs its driver for proper connectivity.

1. At the command prompt, run:
   ```powershell
   pnputil /add-driver E:\NetKVM\2k22\amd64\*.inf /install
   ```
   (Replace `E:` with your VirtIO ISO drive letter if different)

2. You should see:
   ```
   Total driver packages: 1
   Added driver packages: 1
   ```

3. Verify NIC is detected:
   ```powershell
   Get-NetAdapter
   ```
   Should show `Ethernet` adapter with status `Up`.

### Return to Server Core Menu

```powershell
sconfig
```

This launches the configuration menu where you'll set hostname, IP, etc.

---

## Server Core Configuration

From the sconfig menu (option numbers shown):

### 1. Configure Network (Option 8)

1. Select **Option 8: Network settings**
2. Select the **Ethernet** adapter
3. Configure **Static IP**:
   - IP Address: `172.16.1.10`
   - Subnet Mask: `255.255.255.0` (/24)
   - Default Gateway: `172.16.1.1` (pfSense)
   - DNS Server: `172.16.1.10` (DC-01 itself, after AD promotion)

### 2. Rename Server (Option 2)

1. Select **Option 2: Computer name**
2. Enter: `DC-01`
3. Confirm restart: **Yes**
4. Server reboots with new name

### 3. Verify Configuration

After restart, sconfig should show:
```
Computer name:    DC-01
IP Address:       172.16.1.10
Gateway:          172.16.1.1
```
