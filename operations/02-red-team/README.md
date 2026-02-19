# Red Teaming Operations – Enterprise Lab

## Purpose

This folder documents structured adversary simulation activities performed inside the Enterprise Lab.

The objective is to:

* Simulate realistic attack chains
* Map activities to MITRE ATT&CK
* Validate logging visibility
* Test Wazuh detections
* Identify detection gaps
* Improve defensive posture
* Feed hardening decisions

This is **controlled adversary simulation**, not random tool execution.

---

## Scope of Engagement

All attacks are performed ONLY within:

* Lab virtual machines
* Isolated internal network segments
* Dedicated attack machine
* Authorized domain systems
* Firewall-controlled environment

No external systems are targeted.

---

## Safety & Snapshot Policy (MANDATORY)

Before every attack simulation:

1. Take snapshot of:

   * Domain Controller
   * Target system
   * Wazuh server (if rules will be modified)
   * pfSense (if firewall testing involved)

2. Confirm:

   * Logging is working
   * Wazuh archives index is receiving data
   * Time synchronization is correct

3. Document snapshot name inside technique folder.

After testing:

* Restore snapshot if system integrity impacted.

---

## Attack Methodology Framework

All attacks are structured according to:

* MITRE ATT&CK Enterprise Matrix
* Kill Chain progression
* Detection engineering feedback loop

Each technique folder corresponds to one ATT&CK phase:

```
01-Initial-Access/
02-Execution/
03-Persistence/
04-Privilege-Escalation/
05-Defense-Evasion/
06-Credential-Access/
07-Discovery/
08-Lateral-Movement/
09-Collection/
10-Exfiltration/
11-Impact/
```

---

## Standard Technique Folder Structure

Each simulated attack MUST follow this documentation template:

```
Technique_Name/
├── Technique.md
├── Evidence/
├── Logs/
├── Screenshots/
└── Findings.md
```

---

## Required Documentation Template

### Technique.md

Must include:

### • MITRE Mapping

* Technique ID (e.g., T1003)
* Tactic (Credential Access, Lateral Movement, etc.)
* Sub-technique if applicable

### • Objective

What are we trying to simulate?

### • Target System

VM name, OS, network segment.

### • Attack Tools Used

Example:

* PowerShell
* Built-in commands
* Red Team utilities
* Custom scripts

### • Commands Executed

Exact commands executed (no summarization).

### • Expected Logs

What logs should appear?

* Windows Security logs?
* Sysmon events?
* pfSense filterlog entries?
* Wazuh rule triggers?
* Firewall events?
* Authentication events?

### • Observed Behavior

What actually appeared?

### • Detection Status

* Detected automatically?
* Only visible in archives?
* No detection?
* Partial visibility?

---

## Logging Validation Checklist

After each attack:

Check:

* Is event in wazuh-archives?
* Is decoder parsing correctly?
* Is rule triggered?
* Is MITRE field populated?
* Is source IP correct?
* Is agent name correct?
* Is timestamp aligned?

If something fails:

* Document gap
* Move to Blue Team folder

---

## Red Team Execution Rules

1. No destructive payloads.
2. No ransomware simulation.
3. No wiping logs.
4. No disabling Wazuh permanently.
5. No external beaconing to real C2 infrastructure.
6. Simulate C2 locally or via controlled test methods.

This lab is for detection improvement, not destruction.

---

## Attack Campaign Model

We will simulate full attack chains, not isolated commands.

Example Campaign:

1. Initial Access – Credential brute force
2. Execution – PowerShell payload
3. Privilege Escalation – Token manipulation
4. Credential Access – Dump LSASS
5. Lateral Movement – Remote execution
6. Collection – File access
7. Exfiltration – Simulated transfer
8. Impact – Controlled service modification

Each phase must be:

* Logged
* Detected
* Investigated

---

## Integration With Blue Teaming

After completing a technique:

Move to:

```
OPERATIONS/02-Blue-Teaming/
```

There you will:

* Write detection queries
* Improve Wazuh rules
* Add correlation logic
* Identify missing telemetry
* Document detection maturity

Red Team creates activity.
Blue Team validates visibility.

---

## Integration With Hardening

If weakness discovered:

Document mitigation inside:

```
OPERATIONS/03-Hardening/
```

Include:

* Configuration changes
* GPO modifications
* Firewall rule updates
* Wazuh rule tuning
* Logging enhancements

Then retest attack.

---

## Detection Maturity Levels

Each technique will be graded:

Level 0 – No logs generated
Level 1 – Logs exist in archives only
Level 2 – Parsed but no alert
Level 3 – Alert triggered
Level 4 – Correlated detection
Level 5 – Prevented or blocked

Goal: Move every technique toward Level 4–5.

---

## Naming Convention

Each technique folder:

```
TXXXX-Technique-Name
```

Example:

```
T1003-Credential-Dumping/
T1059-PowerShell-Execution/
T1021-Remote-Services/
```

This makes ATT&CK coverage measurable.

---

## Logging Sources Covered

The lab currently supports:

* Windows Security Logs
* Sysmon Events
* pfSense filterlog
* Web interface logs (nginx/php-fpm)
* Wazuh archive logs
* Syscollector process inventory
* Firewall traffic logs

Each attack must attempt to trigger at least one of these.

---

## Metrics We Will Track

For each technique:

* Detection time
* Log completeness
* False positives
* Missing telemetry
* Required tuning
* Hardening applied

Over time this becomes:

A measurable ATT&CK coverage dashboard.

---

## Long-Term Objective

Transform this lab into:

* A structured ATT&CK coverage validation platform
* A detection engineering training environment
* A SOC analyst skill-building framework
* A documentation-rich portfolio project

---

## Red Team Operating Philosophy

This is not about:

Running tools blindly.

This is about:

Understanding how systems generate telemetry.

Understanding how logs correlate.

Understanding what attackers leave behind.

And learning how defenders catch them.
