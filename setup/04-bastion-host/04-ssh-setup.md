# SSH Access Configuration

SSH setup for secure remote access to lab systems. SSH was configured on the Bastion Host (Debian) and DC-01 (Windows Server 2022 Core) to enable direct terminal access from external networks via VPN.

 **Bastion Host:** SSH server verified and SSH key authentication configured  
 **DC-01:** OpenSSH Server installed and configured on Windows Server Core  
 **Passwordless Access:** SSH keys set up for bastion host  
 **Firewall Rules:** SSH access allowed on both systems  
 **Tested & Working:** Verified SSH connectivity to both systems  

***

####Part 1: Bastion Host SSH Configuration

Step 1: Verify SSH Service

**Verify SSH is running:**

```bash
# Check SSH service status
sudo systemctl status ssh

# Should show: Active: active (running)
```

**Default SSH configuration:**
- Port: 22
- Password authentication: Enabled
- Root login: Permitted
- Key authentication: Enabled

***

Step 2: Generate SSH Key Pair (Client Side)

**On your local machine (Mac/Linux/Windows):**

```bash
# Generate ED25519 key (recommended - most secure)
ssh-keygen -t ed25519 -C "labadmin@homelab"

# Or use RSA if ED25519 not supported
ssh-keygen -t rsa -b 4096 -C "labadmin@homelab"
```

**Prompts:**
```
Enter file in which to save the key (/Users/yourname/.ssh/id_ed25519): [Press Enter]
Enter passphrase (empty for no passphrase): [Press Enter or set passphrase]
Enter same passphrase again: [Press Enter]
```

**Note:** The `-C "labadmin@homelab"` is just a comment/label for identification - it's optional.

**Keys generated:**
- **Private key:** `~/.ssh/id_ed25519` (keep secret!)
- **Public key:** `~/.ssh/id_ed25519.pub` (safe to share)

***

Step 3: Copy Public Key to Bastion

**Method 1: Using ssh-copy-id (Recommended)**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub labadmin@172.16.1.20
```

**Enter password** when prompted (last time!)

**Method 2: Manual Copy**

```bash
# Display public key
cat ~/.ssh/id_ed25519.pub

# Copy the output, then SSH to bastion and add it manually:
ssh labadmin@172.16.1.20

# On bastion:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste the public key, save and exit

chmod 600 ~/.ssh/authorized_keys
```

***

Step 4: Test SSH Key Authentication

**Connect without password:**

```bash
ssh labadmin@172.16.1.20
```

**Expected result:** Login successful without password prompt! 

#### Part 2: DC-01 SSH Configuration (Windows Server 2022 Core)

- DC-01 (172.16.1.10) running Windows Server 2022 Core
- Access to DC-01 console via Proxmox
- Administrator credentials

Step 1: Access DC-01 Console

Check OpenSSH Availability

**Command:**

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```


Install OpenSSH Server

**Command:**

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
Start SSH Service

**Start the service:**

```powershell
Start-Service sshd
```

**Set to start automatically:**

```powershell
Set-Service -Name sshd -StartupType 'Automatic'
```

**Verify service status:**

```powershell
Get-Service sshd
```

**Output:**

```
Status   Name      DisplayName
------   ----      -----------
Running  sshd      OpenSSH SSH Server
```

**Result:** SSH service running and set to automatic startup

***

Configure Windows Firewall

**Create firewall rule:**

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH**' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

**Output:**

```
Name                  : sshd
DisplayName           : OpenSSH**
Description           :
DisplayGroup          :
Group                 :
Enabled               : True
Profile               : Any
Platform              : {}
Direction             : Inbound
Action                : Allow
EdgeTraversalPolicy   : Block
LooseSourceMapping    : False
LocalOnlyMapping      : False
Owner                 :
PrimaryStatus         : OK
Status                : The rule was parsed successfully from the store. (65536)
EnforcementStatus     : NotApplicable
PolicyStoreSource     : PersistentStore
PolicyStoreSourceType : Local
RemoteDynamicKeywordAddresses : {}
```

**Result:** Firewall rule created - SSH port 22 accessible

***

Test SSH Connection to DC-01

**From your local machine (with VPN connected):**

```bash
ssh administrator@172.16.1.10
```

Issue: "Permission denied (publickey)"

**Bastion - check authorized_keys:**
```bash
ls -la ~/.ssh/authorized_keys
cat ~/.ssh/authorized_keys
```

**Permissions should be:**
- `~/.ssh/` directory: 700
- `~/.ssh/authorized_keys`: 600

Issue: "Host key verification failed"

**Remove old host key:**
```bash
ssh-keygen -R 172.16.1.20
```

Then reconnect and accept new fingerprint.

***

## Optional: Copy SSH Key to DC-01

**For passwordless access to DC-01 (optional):**

```powershell
# On DC-01, create .ssh directory
mkdir C:\Users\Administrator\.ssh

# Copy your public key content to:
# C:\Users\Administrator\.ssh\authorized_keys

# Set proper permissions
icacls C:\Users\Administrator\.ssh\authorized_keys /inheritance:r
icacls C:\Users\Administrator\.ssh\authorized_keys /grant "Administrator:F"
```
