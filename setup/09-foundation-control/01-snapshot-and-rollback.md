# Snapshot and Rollback Strategy

## Purpose

This document defines the official snapshot and rollback strategy for the Enterprise Security Lab environment.

The objective is to:

* Protect lab stability during offensive security testing
* Enable rapid recovery from misconfiguration or system compromise
* Preserve investigation integrity
* Simulate enterprise-grade change control procedures
* Ensure reproducibility of attack and detection scenarios

This strategy must be followed before performing:

* Attack simulations
* Active Directory modifications
* Firewall rule changes
* Wazuh rule or decoder modifications
* LDAP configuration updates
* Security hardening experiments
* Patch testing

---

# Why Snapshots Are Mandatory

Security testing intentionally introduces instability.

Common risks include:

* Firewall lockouts
* LDAP authentication failures
* Domain Controller corruption
* Misconfigured GPO policies
* Broken Wazuh decoders
* Rule parsing failures
* Service crashes
* Accidental service exposure
* Hardening misconfiguration

Snapshots provide:

* Rapid rollback capability
* Clean forensic baseline comparison
* Controlled testing workflow
* Enterprise-style change safety net

In real enterprise environments, this role is performed by:

* Backup systems
* DR replication
* Snapshot orchestration platforms
* Change management workflows

In this lab, Proxmox snapshots fulfill that function.

---

# Snapshot Naming Convention

To maintain clarity and traceability, use structured naming.

### Baseline Snapshots

```
BASELINE-STABLE-INFRA-YYYY-MM-DD
```

Example:

```
BASELINE-STABLE-INFRA-2026-02-16
```
---

# Snapshot Scope Requirements

The following virtual machines must be snapshotted before major changes:

| Layer     | VM                                      | Snapshot Required |
| --------- | --------------------------------------- | ----------------- |
| Network   | pfSense                                 | Yes               |
| Identity  | Domain Controller (Windows Server 2022) | Yes               |
| Security  | Wazuh Server                            | Yes               |
| Jump Host | Debian Bastion                          | Recommended       |
| DevSecOps | Ubuntu DevOps VM                        | Recommended       |
| Client    | Windows 11                              | Optional          |
| Attack    | Kali / UTM Attack VM                    | Optional          |

Minimum required before attack simulations:

* pfSense
* Domain Controller
* Wazuh

---

# Creating a Snapshot in Proxmox

### Step 1: Stop Critical Services

For database consistency:

* Stop heavy applications if necessary
* Ensure no pending updates running
* Confirm stable state

---

### Step 2: Create Snapshot

1. Log into Proxmox Web UI

2. Select the VM

3. Navigate to **Snapshots**

4. Click **Take Snapshot**

5. Enter snapshot name (follow naming convention)

6. Add description:

   * What is configured
   * Why snapshot is taken
   * Planned activity

7. Choose:

   * ✔ Include RAM (optional)
   * ✔ Include filesystem state

8. Click **Create**

---

# Snapshot Description Best Practice

Always include:

* Infrastructure state
* Logging validation status
* LDAP status
* Firewall status
* Wazuh agent connectivity
* Known issues (if any)

Example:

> All systems operational.
> Wazuh archives enabled.
> pfSense logs flowing to UDP 5514.
> AD LDAP integrated with Wazuh and Gitea.
> No pending updates.
> Ready for MITRE T1059 simulation.

---

# Rollback Procedure

 Warning: Rollback will discard all changes made after snapshot.

---

### Step 1: Power Off VM

Never rollback while running.

---

### Step 2: Select Snapshot

* VM → Snapshots
* Select desired snapshot

---

### Step 3: Click Rollback

* Confirm rollback
* Wait for operation to complete

---

### Step 4: Start VM

* Monitor boot
* Verify:

  * Services running
  * Network reachable
  * Wazuh agents connected
  * Domain authentication working

---

# Post-Rollback Verification Checklist

After rollback:

### Network

* pfSense accessible
* Firewall rules intact
* Log forwarding working

### Identity

* AD services running
* LDAP authentication working
* Domain login functional

### Security

* Wazuh manager running
* Filebeat running
* Archives index active
* Logs flowing

### Validation Commands

On Wazuh:

```
systemctl status wazuh-manager
systemctl status filebeat
curl -k -u user:pass https://localhost:9200/_cat/indices?v
```

On pfSense:

* Confirm Remote Logging still enabled
* Confirm UDP 5514 traffic visible via tcpdump

---

# Baseline Snapshot Requirement

Before starting Enterprise Attack Simulations, create:

```
BASELINE-STABLE-INFRA
```

Requirements:

* All VMs operational
* pfSense logging to Wazuh verified
* Wazuh archives index active
* Sysmon installed on Windows
* Advanced auditing enabled
* No active alerts

This snapshot becomes the master restore point.

---

# Change Control Workflow (Enterprise Model)

Every major change must follow:

```
1️⃣ Snapshot
2️⃣ Perform Change / Attack
3️⃣ Validate System
4️⃣ Document Observations
5️⃣ If unstable → Rollback
6️⃣ If stable → Continue
```

---

# Snapshot Retention Strategy

Do not accumulate excessive snapshots.

Recommended:

* Keep one baseline
* Keep one per active attack scenario
* Remove obsolete snapshots after documentation complete

---

# When NOT to Use Snapshot

Snapshots are not long-term backup solutions.

Do not rely on them for:

* Permanent storage
* Long-term data retention
* Disaster recovery simulation

They are tactical rollback mechanisms.

---

# Final Rule

Never begin:

* MITRE simulation
* Hardening phase
* Firewall restructuring
* AD schema modification
* Wazuh decoder creation

Without first taking a snapshot.

---

# Summary

This lab operates under controlled change principles.

Snapshots ensure:

* Stability
* Reproducibility
* Safe experimentation
* Enterprise realism

All offensive and defensive testing must follow this strategy.
