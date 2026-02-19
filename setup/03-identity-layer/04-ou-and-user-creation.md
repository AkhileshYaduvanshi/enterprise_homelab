# 04-Identity-Layer-RBAC-Setup.md

---

# 04 – Identity Layer & RBAC Setup

## Objective

In this step, we implement the **Identity and Access Management (IAM) layer** for the lab.

The goal is to:

* Create structured Organizational Units (OU)
* Implement Role-Based Access Control (RBAC)
* Create administrative and analyst accounts
* Prepare the environment for LDAP integration with:

  * pfSense
  * Wazuh

This completes the foundational identity architecture of the lab.

---

# Environment Information

| Item              | Value               |
| ----------------- | ------------------- |
| Domain Name       | lab.internal        |
| Base DN           | DC=lab,DC=internal  |
| Domain Controller | DC-01.lab.internal  |
| Functional Level  | Windows Server 2016 |

---

# Step 1 – Create OU Structure

We create a clean and scalable OU structure.

### Planned Structure

```
DC=lab.internal
└── OU=lab_internal
    ├── OU=Users
    ├── OU=Groups
    └── OU=ServiceAccounts
```

### Why?

* Separation of users and groups
* Clean policy management
* Enterprise-style structure
* Scalable for future expansion

---

# Step 2 – Define RBAC Groups

We create application-specific security groups.

## Administrative Groups

* AD_Admin
* pfSense_Admin
* Wazuh_Admin

## Analyst Groups

* pfSense_Analyst
* Wazuh_Analyst
* AD_Read_only

---

# Step 3 – Create User Accounts

## labadmin

Full administrative account across:

* Active Directory
* pfSense
* Wazuh

## labanalyst

Standard SOC analyst account with:

* Read-only AD access
* Analyst access to pfSense
* Analyst access to Wazuh

---

# Password Configuration (Lab Only)

For simplicity:

```
Password: P@$$w0rd!
Password Never Expires: Enabled
```

⚠ This is for lab purposes only.

---

# Step 4 – Automated Setup Script

Save the following script as:

```
Setup-LabIdentity.ps1
```

Run on the Domain Controller.

---

## PowerShell Script

```powershell
Import-Module ActiveDirectory

# =============================
# VARIABLES
# =============================
$BaseDN   = "DC=lab,DC=internal"
$TopOU    = "OU=lab_internal,$BaseDN"
$UsersOU  = "OU=Users,$TopOU"
$GroupsOU = "OU=Groups,$TopOU"
$SvcOU    = "OU=ServiceAccounts,$TopOU"

$Password = ConvertTo-SecureString "Password@!" -AsPlainText -Force

Clear-Host
Write-Host "=====================================" -ForegroundColor Cyan
Write-Host "  LAB.IDENTITY - AUTOMATED SETUP     " -ForegroundColor Cyan
Write-Host "=====================================" -ForegroundColor Cyan
Write-Host ""

# =============================
# FUNCTION: ENSURE OU
# =============================
function Ensure-OU {
    param ($Name, $Path)

    $Existing = Get-ADOrganizationalUnit -Filter "Name -eq '$Name'" -SearchBase $Path -ErrorAction SilentlyContinue

    if (-not $Existing) {
        New-ADOrganizationalUnit -Name $Name -Path $Path
        Write-Host "Created OU: $Name" -ForegroundColor Green
    }
    else {
        Write-Host "OU already exists: $Name" -ForegroundColor Yellow
    }
}

# =============================
# FUNCTION: ENSURE GROUP
# =============================
function Ensure-Group {
    param ($GroupName)

    $Existing = Get-ADGroup -Filter "Name -eq '$GroupName'" -ErrorAction SilentlyContinue

    if (-not $Existing) {
        New-ADGroup `
        -Name $GroupName `
        -SamAccountName $GroupName `
        -GroupCategory Security `
        -GroupScope Global `
        -Path $GroupsOU

        Write-Host "Created Group: $GroupName" -ForegroundColor Green
    }
    else {
        Write-Host "Group already exists: $GroupName" -ForegroundColor Yellow
    }
}

# =============================
# FUNCTION: ENSURE USER
# =============================
function Ensure-User {
    param ($Name, $Sam, $Given, $Surname)

    $Existing = Get-ADUser -Filter "SamAccountName -eq '$Sam'" -ErrorAction SilentlyContinue

    if (-not $Existing) {
        New-ADUser `
        -Name $Name `
        -GivenName $Given `
        -Surname $Surname `
        -SamAccountName $Sam `
        -UserPrincipalName "$Sam@lab.internal" `
        -Path $UsersOU `
        -AccountPassword $Password `
        -Enabled $true `
        -PasswordNeverExpires $true

        Write-Host "Created User: $Sam" -ForegroundColor Green
    }
    else {
        Write-Host "User already exists: $Sam" -ForegroundColor Yellow
    }
}

# =============================
# FUNCTION: ENSURE MEMBERSHIP
# =============================
function Ensure-Membership {
    param ($Group, $User)

    $Members = Get-ADGroupMember $Group -ErrorAction SilentlyContinue

    if ($Members.SamAccountName -notcontains $User) {
        Add-ADGroupMember -Identity $Group -Members $User
        Write-Host "Added $User to $Group" -ForegroundColor Green
    }
    else {
        Write-Host "$User already member of $Group" -ForegroundColor Yellow
    }
}

# =============================
# CREATE OU STRUCTURE
# =============================
Ensure-OU "lab_internal" $BaseDN
Ensure-OU "Users" $TopOU
Ensure-OU "Groups" $TopOU
Ensure-OU "ServiceAccounts" $TopOU

# =============================
# CREATE GROUPS
# =============================
$Groups = @(
"pfSense_Admin",
"pfSense_Analyst",
"Wazuh_Admin",
"Wazuh_Analyst",
"AD_Admin",
"AD_Read_only"
)

foreach ($Group in $Groups) {
    Ensure-Group $Group
}

# =============================
# CREATE USERS
# =============================
Ensure-User "Lab Administrator" "labadmin" "Lab" "Administrator"
Ensure-User "Lab Analyst" "labanalyst" "Lab" "Analyst"

# =============================
# ADMIN GROUP MEMBERSHIP
# labadmin = SUPER ADMIN
# =============================
$AdminGroups = @(
"AD_Admin",
"pfSense_Admin",
"Wazuh_Admin"
)

foreach ($Group in $AdminGroups) {
    Ensure-Membership $Group "labadmin"
}

# =============================
# ANALYST GROUP MEMBERSHIP
# =============================
Ensure-Membership "pfSense_Analyst" "labanalyst"
Ensure-Membership "Wazuh_Analyst" "labanalyst"
Ensure-Membership "AD_Read_only" "labanalyst"

Write-Host ""
Write-Host "=====================================" -ForegroundColor Cyan
Write-Host " Lab Identity Setup Completed Safely " -ForegroundColor Cyan
Write-Host "=====================================" -ForegroundColor Cyan

```

---

# Verification

Run:

```powershell
Import-Module ActiveDirectory

Clear-Host
Write-Host "=========================================" -ForegroundColor Cyan
Write-Host "   LAB.INTERNAL - IDENTITY AUDIT REPORT  " -ForegroundColor Cyan
Write-Host "=========================================" -ForegroundColor Cyan
Write-Host ""

# -------------------------
# DOMAIN INFORMATION
# -------------------------
Write-Host ">> DOMAIN INFORMATION" -ForegroundColor Yellow
$Domain = Get-ADDomain
$Domain | Select DNSRoot, NetBIOSName, DomainMode | Format-Table
Write-Host ""

# -------------------------
# DOMAIN CONTROLLER INFO
# -------------------------
Write-Host ">> DOMAIN CONTROLLERS" -ForegroundColor Yellow
Get-ADDomainController | Select HostName, IPv4Address, Site | Format-Table
Write-Host ""

# -------------------------
# OU STRUCTURE
# -------------------------
Write-Host ">> CUSTOM OU STRUCTURE (lab_internal)" -ForegroundColor Yellow
$BaseOU = "OU=lab_internal,DC=lab,DC=internal"
Get-ADOrganizationalUnit -SearchBase $BaseOU -Filter * |
Select Name, DistinguishedName | Format-Table
Write-Host ""

# -------------------------
# APPLICATION GROUPS
# -------------------------
Write-Host ">> APPLICATION GROUPS" -ForegroundColor Yellow
$AppGroups = Get-ADGroup -SearchBase "OU=Groups,OU=lab_internal,DC=lab,DC=internal" -Filter *

$AppGroups | Select Name, GroupScope | Format-Table
Write-Host ""

# -------------------------
# USERS IN LAB OU
# -------------------------
Write-Host ">> USERS IN LAB OU" -ForegroundColor Yellow
$Users = Get-ADUser -SearchBase "OU=Users,OU=lab_internal,DC=lab,DC=internal" -Filter *

$Users | Select Name, SamAccountName, Enabled | Format-Table
Write-Host ""

# -------------------------
# GROUP MEMBERSHIP MAPPING
# -------------------------
Write-Host ">> GROUP MEMBERSHIP DETAILS" -ForegroundColor Yellow

foreach ($Group in $AppGroups) {
    Write-Host ""
    Write-Host "Group: $($Group.Name)" -ForegroundColor Green
    $Members = Get-ADGroupMember $Group.Name -ErrorAction SilentlyContinue
    
    if ($Members) {
        $Members | Select Name, SamAccountName, ObjectClass | Format-Table
    }
    else {
        Write-Host "No Members Assigned" -ForegroundColor DarkGray
    }
}

Write-Host ""
Write-Host "=========================================" -ForegroundColor Cyan
Write-Host "   AUDIT COMPLETED SUCCESSFULLY         " -ForegroundColor Cyan
Write-Host "=========================================" -ForegroundColor Cyan

```
