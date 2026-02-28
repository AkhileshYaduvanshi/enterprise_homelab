## Debian 12 Installation

**Step 1: Boot the VM**

**Step 2: Select Installation Method**
- Boot Menu Options:
  - Graphical install
  - **Install** ✅ (Selected)
  - Advanced options
  - Help

**Recommendation:** Select **"Install"** (text-based installer)
- Lighter on resources (512 MB RAM)
- More stable for server installations
- Professional approach

**Action:** Arrow down to "Install" → Press **Enter**


**Prompt:** Please enter the hostname for this system

**Hostname:** `bastion`

**Domain Name:** `lab.internal`

- Maintains consistency across lab infrastructure
- `.internal` TLD reserved for internal networks
- **Full FQDN:** bastion.lab.internal

**Full Hostname Structure:**
```
pfSense:  pfSense.lab.internal
DC-01:    dc-01.lab.internal (will be configured)
Bastion:  bastion.lab.internal
```

**Action:** Type `lab.internal` → Press **Enter**

### Software Selection (Critical Step)

**Prompt:** Choose software to install

#### Initial Selection (Incorrect for Bastion)
```
[*] Debian desktop environment  ← SELECTED (need to remove)
[ ] ... GNOME
[ ] ... Xfce
[ ] ... GNOME Flashback
[ ] ... KDE Plasma
[ ] ... Cinnamon
[ ] ... MATE
[ ] ... LXDE
[ ] ... LXQt
[ ] web server
[ ] SSH server                   ← NOT SELECTED (need to add!)
[*] standard system utilities    ← SELECTED (keep this)
```

#### Required Changes for Jump Host

**1. UNSELECT Debian desktop environment**
- Navigate to "Debian desktop environment"
- Press **Space** to unselect (remove asterisk)

**Why Remove Desktop?**
- No GUI needed for SSH-only server
- Saves ~2-3 GB RAM
- Reduces attack surface
- Faster performance
- Professional server practice

**2. SELECT SSH server**
- Navigate to "SSH server"  
- Press **Space** to select (add asterisk)

**Why SSH Server is Essential?**
- **Core purpose** of jump/bastion host
- Enables remote access
- Required for administration
- Main service for the server

**3. KEEP standard system utilities**
- Already selected (has asterisk)
- Provides basic command-line tools
- Essential system management utilities

#### Final Configuration
```
[ ] Debian desktop environment  (UNSELECTED) ✅
[ ] SSH server                  (SELECTED)   ✅
[*] standard system utilities   (SELECTED)   ✅
```

**Action:** Tab to <Continue> → Press **Enter**

***

### Post-Installation: First Boot

Check Current Network Status

**Command:**
```bash
ip addr show
```

**Output:**
```
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether bc:24:11:ea:00:94 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.12/24 brd 172.16.1.255 scope global dynamic enp6s18
```

**Current Configuration:**
- Interface: **enp6s18**
- IP: 172.16.1.12/24 (DHCP-assigned)
- Status: UP and operational

***

#### Configure Static IP Address

Step 1: Edit Network Interfaces File

**Command:**
```bash
nano /etc/network/interfaces
```

Step 2: Modify Configuration

**Find the DHCP configuration:**
```
allow-hotplug enp6s18
iface enp6s18 inet dhcp
```

**Replace with static configuration:**
```
allow-hotplug enp6s18
iface enp6s18 inet static
    address 172.16.1.20
    netmask 255.255.255.0
    gateway 172.16.1.1
    dns-nameservers 8.8.8.8
```

**Configuration Explained:**
- `address 172.16.1.20` - Static IP for bastion host
- `netmask 255.255.255.0` - /24 subnet  
- `gateway 172.16.1.1` - pfSense LAN interface
- `dns-nameservers 8.8.8.8` - Google DNS (reliable)

Step 3: Save and Exit

- Press **Ctrl+X** to exit
- Press **Y** to confirm save
- Press **Enter** to confirm filename

#### Step 4: Restart Networking

**Command:**
```bash
systemctl restart networking
```

**Alternative (if above fails):**
```bash
reboot
```

***

### Verify Static IP Configuration

#### Check IP Address

**Command:**
```bash
ip addr show
```

**Expected Output:**
```
2: enp6s18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether bc:24:11:6c:5e:86 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.20/24 brd 172.16.1.255 scope global enp6s18
```

#### Test Network Connectivity

**Test 1: Ping Gateway (pfSense)**
```bash
ping 172.16.1.1
```

**Test 2: Ping Internet (Google DNS)**
```bash
ping 8.8.8.8
```
