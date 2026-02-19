# Windows Client Domain Join over OpenVPN (Lab Architecture)

## Overview

This document details the complete process of joining a remote Windows client to the `lab.internal` Active Directory domain **over OpenVPN**, including all troubleshooting steps and final delegation of secure domain join permissions.

This setup simulates a real-world hybrid environment where:

* Domain Controller is on internal network (172.16.1.10)
* Client connects remotely via OpenVPN (10.0.0.0/24 tunnel)
* Domain join must work securely over VPN
* No Domain Admin account is used for joining

---

# Lab Architecture Context

| Component            | IP          | Network      |
| -------------------- | ----------- | ------------ |
| DC-01                | 172.16.1.10 | Internal LAN |
| pfSense              | 172.16.1.1  | Gateway      |
| OpenVPN Tunnel       | 10.0.0.0/24 | Remote VPN   |
| Windows Client (VPN) | 10.0.0.2    | VPN IP       |

---

# Phase 1 — Initial Problem

After connecting the client to OpenVPN:

```
nslookup lab.internal → SUCCESS
Test-NetConnection 172.16.1.10 -Port 389 → SUCCESS
```

But:

```
nltest /dsgetdc:lab.internal
→ ERROR_NO_SUCH_DOMAIN
```

Meaning:

* LDAP reachable
* Kerberos reachable
* SMB reachable
* But DC discovery failed

This indicates a **routing or DNS priority issue**.

---

# Phase 2 — Validate Required Services

From client:

```powershell
Test-NetConnection 172.16.1.10 -Port 389
Test-NetConnection 172.16.1.10 -Port 88
Test-NetConnection 172.16.1.10 -Port 445
Test-NetConnection 172.16.1.10 -Port 135
```

All returned:

```
TcpTestSucceeded : True
```

So firewall was NOT the issue.

---

# Phase 3 — Validate SRV Records

```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.lab.internal
```

Returned:

```
dc-01.lab.internal
172.16.1.10
```

DNS SRV was correct.

---

# Phase 4 — Identify Root Cause

Running:

```powershell
ipconfig /all
```

Revealed:

Client had:

```
Ethernet (192.168.1.x)
Local Area Connection (OpenVPN 10.0.0.2)
```

Windows was preferring local Ethernet over VPN.

Domain discovery traffic was not using VPN.

---

# Phase 5 — Fix Interface Priority (Critical Step)

We manually adjusted interface metrics.

### Set VPN Interface Higher Priority

```powershell
Set-NetIPInterface -InterfaceAlias "Local Area Connection" -InterfaceMetric 5
Set-NetIPInterface -InterfaceAlias "Ethernet" -InterfaceMetric 50
```

Verify:

```powershell
Get-NetIPInterface
```

VPN interface must show lower metric (higher priority).

---

# Phase 6 — Successful DC Discovery

After metric adjustment:

```powershell
nltest /dsgetdc:lab.internal
```

Returned:

```
DC: \\DC-01.lab.internal
Address: \\172.16.1.10
Flags: PDC GC DS LDAP KDC DNS_DC DNS_DOMAIN
The command completed successfully
```

Domain controller discovery over VPN now working.

---

# Phase 7 — Reverse DNS Validation

Originally:

```
nslookup 172.16.1.10 → Non-existent domain
```

Reverse lookup zone missing.

After fixing DNS reverse zone:

```
nslookup 172.16.1.10
→ dc-01.lab.internal
```

Reverse DNS now healthy.

---

# Phase 8 — Secure Domain Join Account Design

Instead of using `labadmin`, we implemented least privilege.

---

## Create Dedicated Service Account

User:

```
svc_domain_join
```

Password:

```
Password@1
```

Created under:

```
OU=lab_internal,DC=lab,DC=internal
```

---

## Create Delegation Group

Group:

```
Domain_Join_Operators
```

Scope:

```
Global Security Group
```

---

## Create Computers OU

```
OU=Computers,OU=lab_internal,DC=lab,DC=internal
```

This OU will store all joined machines.

---

# Phase 9 — Delegate Join Rights (Enterprise Method)

### Get NetBIOS Name

```powershell
(Get-ADDomain).NetBIOSName
```

Result:

```
LAB
```

---

### Grant Create/Delete Computer Permissions

```powershell
$OUDN = "OU=Computers,OU=lab_internal,DC=lab,DC=internal"

dsacls "$OUDN" /G "LAB\Domain_Join_Operators:CC;computer"
dsacls "$OUDN" /G "LAB\Domain_Join_Operators:DC;computer"
```

This grants:

* Create Computer objects
* Delete Computer objects
* Only inside Computers OU

No Domain Admin rights granted.

---

# Phase 10 — Domain Join

On Client:

1. Open:

   ```
   System Properties → Computer Name → Change
   ```

2. Select:

   ```
   Domain: lab.internal
   ```

3. Use credentials:

   ```
   svc_domain_join
   Password@1!
   ```

4. Restart machine.

---

# Phase 11 — Post-Reboot Behavior

Important concept:

VPN disconnects on reboot.

However:

Windows caches domain credentials.

You can log in using:

```
akhilesh@lab.internal
```

Even if VPN is not connected.

After login:

* Connect OpenVPN manually
* Domain connectivity restored

This simulates real remote workforce architecture.

---

# Verification Commands

On DC:

```powershell
Get-ADComputer -SearchBase "OU=Computers,OU=lab_internal,DC=lab,DC=internal"
```

Client computer object should appear.

---

# Security Architecture Achieved

You now have:

* Secure remote domain join over VPN
* Proper DNS validation
* Interface metric routing control
* Reverse lookup zone configured
* Least-privilege service account
* OU-scoped delegation
* No use of Domain Admin for joins
* Enterprise-ready design

---

# Lessons Learned

1. DNS SRV records are critical for DC discovery.
2. Interface metric controls routing priority.
3. Reverse DNS impacts domain discovery reliability.
4. dsacls requires `DOMAIN\GroupName` format.
5. Cached credentials allow login without VPN.
6. Service account delegation is best practice.

---

# Final State

Client joined to:

```
lab.internal
```

Stored under:

```
OU=Computers,OU=lab_internal
```

Joined using:

```
svc_domain_join
```
