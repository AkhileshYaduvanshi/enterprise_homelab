# Active Directory Setup - Promote DC-01 to Domain Controller

## Overview

This document covers the complete Active Directory promotion process, converting DC-01 from a standalone Windows Server into the first domain controller for the `lab.internal` domain.

## Prerequisites

Before starting AD promotion, ensure:
- Windows Server 2022 Core installed on DC-01 ✓
- Network configured: IP 172.16.1.10/24, Gateway 172.16.1.1 ✓
- Server renamed to DC-01 ✓
- VirtIO NIC driver installed ✓

---

## Step 1: Install Active Directory Domain Services (AD DS)

From the Server Core command prompt, run:

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

**Expected Output:**
```
Success Restart Needed Exit Code    Feature Result
------  -------------- ---------    ---------------
True    No             Success      {Active Directory Domain Services, Group P...
```

This installs:
- Active Directory core binaries
- Domain Services tools for management
- DNS Server (required for AD)
- DHCP Server management tools

---

## Step 2: Promote DC-01 to Domain Controller

This creates a new Active Directory forest with the root domain `lab.internal`.

```powershell
Install-ADDSForest -DomainName "lab.internal" `
  -DomainNetbiosName "LAB" `
  -SafeModeAdministratorPassword (ConvertTo-SecureString -AsPlainText "P@ssw0rd!DSRM2025" -Force) `
  -NoRebootOnCompletion `
  -Confirm:$false
```

**Parameters Explained:**

| Parameter | Value | Purpose |
|-----------|-------|---------|
| DomainName | lab.internal | Full DNS name of your AD domain |
| DomainNetbiosName | LAB | Legacy short domain name (max 15 chars) |
| SafeModeAdministratorPassword | P@ssw0rd!DSRM2025 | DSRM password (Directory Services Restore Mode - for recovery) |
| NoRebootOnCompletion | (flag) | Don't reboot yet; we'll do it manually |
| Confirm:$false | (flag) | Proceed without prompts |

**Important:** Replace the password with a strong, unique one. Store it securely!

**Expected Output:**
```
Message                                      Context              RebootRequired  Status
-------                                      -------              ---------------  ------
You must restart this computer to complete... DCPromo.General.4   True             Success
```

---

## Step 3: Restart the Server

```powershell
Restart-Computer
```

The server will restart and fully promote DC-01 as the first domain controller. This takes 2-5 minutes.

**During reboot:**
- AD databases are created
- DNS service is initialized
- Active Directory is brought online
- Replication infrastructure is set up
- Group Policy is initialized

---

## Step 4: Verify Active Directory Promotion

After reboot, log in again as `Administrator` and verify AD is working:

```powershell
Get-ADDomain -Identity "lab.internal"
```

**Expected Output:**
```
AllowedDNSSuffixes             : {}
ChildDomains                   : {}
ComputersContainer             : CN=Computers,DC=lab,DC=internal
DeletedObjectsContainer        : CN=Deleted Objects,DC=lab,DC=internal
DistinguishedName              : DC=lab,DC=internal
DNSRoot                        : lab.internal
DomainControllersContainer     : OU=Domain Controllers,DC=lab,DC=internal
DomainMode                     : Windows2016
DomainSID                      : S-1-5-21-[numbers]
Forest                         : lab.internal
Name                           : lab
NetBIOSName                    : LAB
ParentDomain                   :
```

Verify DC-01 is recognized as a domain controller:

```powershell
Get-ADDomainController
```

**Expected Output:**
```
ComputerObjectDN : CN=DC-01,OU=Domain Controllers,DC=lab,DC=internal
DefaultPartition : DC=lab,DC=internal
Domain           : lab.internal
Forest           : lab.internal
HostName         : DC-01.lab.internal
IsGlobalCatalog  : True
IsReadOnly       : False
Name             : DC-01
OperatingSystem  : Windows Server 2022 Standard
Site             : Default-First-Site-Name
```

---

## Step 5: Create Domain Admin User Account

Create a dedicated admin account for daily use (don't use built-in Administrator).

```powershell
New-ADUser -Name "labadmin" `
  -SamAccountName "labadmin" `
  -UserPrincipalName "labadmin@lab.internal" `
  -AccountPassword (ConvertTo-SecureString -AsPlainText "P@ssw0rd!Admin2025" -Force) `
  -Enabled $true `
  -PasswordNeverExpires $true
```

Add this user to Domain Admins group:

```powershell
Add-ADGroupMember -Identity "Domain Admins" -Members "labadmin"
```

Verify the user was created:

```powershell
Get-ADUser -Identity "labadmin"
```

---

## Step 6: Verify Sconfig Shows Domain

Return to sconfig menu:

```powershell
sconfig
```

Check that:
- **Option 1) Domain/workgroup**: Shows `Domain: lab.internal` ✓
- **Option 2) Computer name**: Shows `DC-01` ✓

---

## AD Configuration Summary

Your Active Directory is now ready! Here's what has been created:

```
Active Directory Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Forest Name:                lab.internal
Domain Name:                lab.internal
NetBIOS Name:               LAB
Forest Level:               Windows Server 2016
Domain Level:               Windows Server 2016
Root DC:                    DC-01.lab.internal
DC Hostname:                DC-01
DC IP Address:              172.16.1.10
DC Operating System:        Windows Server 2022 Core

DEFAULT ADMIN CREDENTIALS:
  Built-in Admin:           LAB\Administrator
  Domain Admin User:        LAB\labadmin
  
DEFAULT GROUPS:
  Domain Admins             - Full AD permissions
  Domain Users              - Regular user group
  Domain Computers          - All computer objects
  
DEFAULT ORGANIZATIONAL UNITS:
  Domain Controllers        - DC objects
  Users                     - User objects (default)
  Computers                 - Computer objects (default)
  
DNS STATUS:
  Integrated DNS:           Active (DC-01.lab.internal)
  Primary DNS Zone:         lab.internal
  Forwarders:               8.8.8.8 (Google DNS fallback)
  
DHCP STATUS:
  Server Ready for Config:  Yes (install DHCP role if needed)
  
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Next Steps

Now that Active Directory is established:

1. **Domain-join other VMs** (clients, servers) to `lab.internal`
2. **Configure Group Policy** for security and management
3. **Set up DHCP** (if not already configured on pfSense)
4. **Create organizational units (OUs)** for organization
5. **Add users and groups** as needed for your lab
6. **Regular backups** of AD database (System State backup)

---

## Troubleshooting

### Issue: Cannot connect to AD from other VMs

**Solution:**
- Verify DNS is pointing to DC-01 (172.16.1.10)
- Ensure network connectivity between VMs and DC-01
- Check firewall rules allow AD ports (88, 135, 389, 445, etc.)
- Verify DC-01 is still running and AD services are active

### Issue: AD promotion failed

**Solution:**
- Check available disk space (need at least 500 MB free)
- Verify network configuration is correct
- Run: `dcdiag` to check DC health
- Review Application Event Log for errors

### Issue: Cannot log in with domain account

**Solution:**
- Verify user exists: `Get-ADUser -Identity "labadmin"`
- Check password hasn't expired (we set PasswordNeverExpires)
- Ensure VM is domain-joined (not just network-connected)
- Run `gpupdate /force` on client to refresh policy
