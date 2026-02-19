# Overall Architecture – Enterprise Cybersecurity Lab

## Overview
This document provides a high-level overview of the Enterprise Cybersecurity Lab architecture.  
The lab is designed to simulate a corporate hybrid environment supporting:

- Identity management using Active Directory
- Security monitoring with Wazuh SIEM
- Network segmentation and firewalling via pfSense
- Virtualization using Proxmox
- Secure remote access using OpenVPN
- DevSecOps pipeline using Jenkins & Gitea
- Attack simulation and detection engineering workflows

The architecture follows a layered approach to replicate enterprise infrastructure and security operations.

---

## Design Goals
The lab architecture is built with the following goals:

- Simulate a real corporate network environment
- Enable red team vs blue team exercises
- Provide centralized logging and monitoring
- Enforce controlled administrative access via jump host
- Support CI/CD security testing workflows
- Enable safe attack simulation and detection validation

---

## Network Overview

### External Network
- Home network: **192.168.1.0/24**
- Proxmox Host: **192.168.1.101**
- pfSense WAN: **192.168.1.101**
- Attacker Machine: **192.168.1.50**
- Client Machine: **192.168.1.10**

### VPN Network
- VPN Segment: **10.0.0.0/24**
- Used for secure remote connectivity into lab environment
- Provides controlled gateway access to internal resources

### Internal Lab Network
- Internal Segment: **172.16.1.0/24**
- Segmented behind pfSense firewall
- Hosts core enterprise services

---

## 🏗️ Architecture Diagram

![Overall Architecture](../../assets/diagrams/overall-architecture.png)

---

## Core Components

### 🖥️ Virtualization Layer
**Proxmox Server**
- Hosts all lab VMs
- Provides network bridges and segmentation
- Enables snapshot and rollback strategy

---

### Network Security Layer
**pfSense Firewall**
- WAN: 192.168.1.101
- LAN: 172.16.1.0/24
- VPN Gateway: 10.0.0.0/24
- Controls routing, segmentation, and VPN access

---

### Access Control Layer
**Jump Host / Bastion (172.16.1.20)**
- Central administrative entry point
- SSH gateway to internal services
- Restricts direct administrative access

---

### Identity Layer
**Active Directory Domain Controller (172.16.1.10)**
- Domain services
- LDAP authentication
- Centralized identity and RBAC

---

### Security Monitoring Layer
**Wazuh SIEM (172.16.1.50)**
- Centralized log collection
- Sysmon & Windows auditing ingestion
- Detection engineering & threat hunting

---

### DevSecOps Layer
**CI/CD Node (Jenkins + Gitea) – 172.16.1.100**
- Code repository and pipeline automation
- LDAP integration for RBAC
- Build, deploy, and security testing workflows

---

### Attack Simulation Layer
**Kali / Attack Machine**
- Simulates adversary activity
- Used for red team exercises and detection validation

---

## Logging & Monitoring Flow
- Windows endpoints forward logs to Wazuh
- Linux systems send Syslog to Wazuh
- Security telemetry centralized for detection engineering
- CI/CD environment generates logs for pipeline monitoring

---

## Access Flow
1. Remote user connects via OpenVPN
2. Access routed through pfSense firewall
3. Administrative access enforced via Jump Host
4. Services accessed using domain authentication
5. Logs collected and monitored by Wazuh SIEM

---

## Future Enhancements
- Multi-DC architecture
- EDR integration
- Cloud hybrid extension
- Purple team automation workflows
- SOAR playbooks

---

## Related Documents
- Network IP Scheme
- OU Structure Design
- Wazuh Architecture
- Detection Engineering Workflows
- Red Team Attack Scenarios
