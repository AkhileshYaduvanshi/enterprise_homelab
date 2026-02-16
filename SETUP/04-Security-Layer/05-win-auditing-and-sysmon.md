# Windows Advanced Auditing & Sysmon Deployment

## Overview

This document describes the configuration of:

* Advanced Windows Audit Policy
* Command-line process logging
* Sysmon deployment (Server Core compatible)
* Wazuh agent integration for Sysmon channel
* Validation of telemetry ingestion

This configuration elevates the lab from basic logging to **DFIR-grade visibility**.

---

# Objectives

After completing this configuration, the environment will:

* Log detailed authentication events
* Capture process creation with full command line
* Record parent-child process relationships
* Capture network connections per process
* Forward Sysmon telemetry to Wazuh archives
* Enable threat hunting in OpenSearch

---

# Advanced Audit Policy Configuration (Domain Controller)

> All commands executed via PowerShell (Administrator)

---

## Enable Critical Audit Subcategories

```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Logoff" /success:enable /failure:enable
auditpol /set /subcategory:"Special Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Directory Service Access" /success:enable /failure:enable
auditpol /set /subcategory:"Directory Service Changes" /success:enable /failure:enable
auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable
```

---

## Verify Configuration

```powershell
auditpol /get /category:*
```

Confirm:

* Process Creation → Success & Failure
* Logon → Success & Failure
* Sensitive Privilege Use → Enabled

---

# Enable Command-Line Logging (Critical for IR)

Without this, Event ID 4688 will NOT contain command line.

```powershell
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f
```

Reboot:

```powershell
shutdown /r /t 0
```

---

# Sysmon Installation (Server Core Compatible)

## Create Directory Structure

```powershell
New-Item -ItemType Directory -Path C:\Tools -Force
New-Item -ItemType Directory -Path C:\Tools\Sysmon -Force
```

---

## Download Sysmon

```powershell
Invoke-WebRequest -Uri https://download.sysinternals.com/files/Sysmon.zip -OutFile C:\Tools\Sysmon\Sysmon.zip
```

---

## Extract Sysmon

```powershell
Expand-Archive -Path C:\Tools\Sysmon\Sysmon.zip -DestinationPath C:\Tools\Sysmon -Force
```

Verify:

```powershell
Get-ChildItem C:\Tools\Sysmon
```

Expected files:

* Sysmon64.exe
* Sysmon.exe
* EULA.txt

---

# Download Production Sysmon Configuration

Using SwiftOnSecurity baseline:

```powershell
Invoke-WebRequest -Uri https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -OutFile C:\Tools\Sysmon\sysmonconfig.xml
```

---

# Install Sysmon with Configuration

```powershell
cd C:\Tools\Sysmon
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```

Expected output:

```
Sysmon installed.
```

---

## Verify Sysmon Service

```powershell
Get-Service Sysmon64
```

Status must show:

Running

---

# Validate Local Sysmon Logging

Generate test process:

```powershell
notepad.exe
```

Check events:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

Look for:

* Event ID 1 (Process Creation)

---

# Enable Sysmon Channel in Wazuh Agent

By default, Wazuh does NOT collect Sysmon channel.

Edit agent configuration:

```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```

Add inside `<ossec_config>`:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Save file.

---

## Restart Wazuh Agent

```powershell
Restart-Service Wazuh
```

Verify:

```powershell
Get-Service Wazuh
```

---

# Validate Ingestion in Wazuh

Generate test event again:

```powershell
notepad.exe
```

Go to:

Dashboard → Discover
Select:

```
wazuh-archives-4.x-*
```

Filter:

```
win.system.channel : "Microsoft-Windows-Sysmon/Operational"
```

Expected result:
Sysmon Event ID 1 entries visible.

---

# What This Enables

After configuration, the lab now supports:

* Process tree reconstruction
* Command-line analysis
* Lateral movement detection
* Privilege abuse tracking
* Network connection per process
* Behavioral hunting

This forms the foundation for:

* Credential dumping detection
* Ransomware simulation
* MITRE ATT&CK mapping
* Full timeline reconstruction

---

# Telemetry Chain Validation

```
Sysmon
   ↓
Windows Event Log
   ↓
Wazuh Agent
   ↓
Wazuh Manager
   ↓
archives.json
   ↓
Filebeat
   ↓
OpenSearch
   ↓
Wazuh Dashboard
```
