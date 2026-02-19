# IP Address Scheme – Enterprise Cybersecurity Lab

## Overview
This document defines the IP addressing structure used across the Enterprise Cybersecurity Lab.  
The network is segmented to simulate a real corporate environment with clear separation between:

- External/Home Network
- VPN Access Network
- Internal Lab Network
- Future expansion zones

The segmentation improves security, monitoring clarity, and attack simulation realism.

---

## Network Segments

| Segment | CIDR | Purpose |
|--------|------|---------|
| Home Network | 192.168.1.0/24 | External connectivity & WAN segment |
| VPN Network | 10.0.0.0/24 | Secure remote access into lab |
| Internal Lab Network | 172.16.1.0/24 | Core enterprise services and workloads |

---

## Home Network – 192.168.1.0/24
This segment represents the external/home environment and WAN connectivity.

| Host | IP Address | Notes |
|------|-----------|------|
| ISP Router / Gateway | 192.168.1.1 | Default gateway |
| Proxmox Host | 192.168.1.100 | Virtualization platform |
| pfSense WAN | 192.168.1.101 | Firewall WAN interface |
| Client Machine | 192.168.1.10 | Domain-joined endpoint |
| Attacker Machine | 192.168.1.50 | Red team simulation |

**Note** : All the above mentioed host are alloted static IPs.
---

## VPN Network – 10.0.0.0/24
This segment provides secure remote connectivity into the lab environment.

| Host | IP Address | Notes |
|------|-----------|------|
| OpenVPN Gateway | 10.0.0.1 | pfSense VPN interface |
| VPN Clients | 10.0.0.x | Remote administrative access |

### 🔎 Purpose
- Secure remote access
- Administrative entry point
- Controlled routing into internal lab

---

## Internal Lab Network – 172.16.1.0/24
This is the core enterprise segment hosting infrastructure and security services.

| Host | IP Address | Role |
|------|-----------|------|
| pfSense LAN | 172.16.1.1 | Internal gateway |
| Domain Controller | 172.16.1.10 | Active Directory |
| Jump Host | 172.16.1.20 | Bastion / Admin Gateway |
| Wazuh SIEM | 172.16.1.50 | Centralized logging |
| CI/CD Server | 172.16.1.100 | Jenkins + Gitea |
| Cluster Node | TBD | Application workloads |
| Database | TBD | Application backend |

---

## Traffic Flow Summary
- WAN traffic enters via pfSense WAN
- VPN users authenticate and access internal network
- Administrative access enforced through Jump Host
- Internal hosts forward logs to Wazuh SIEM
- CI/CD systems interact with cluster workloads

---

## Design Considerations
- Segmented networks for security isolation
- Predictable static addressing for infrastructure
- VPN-based administrative access
- Centralized logging for detection engineering
- Scalable addressing for future workloads

---

## Future Expansion Plan
Potential future segments:

| Segment | CIDR | Purpose |
|--------|------|---------|
| Management Network | 172.16.2.0/24 | Out-of-band management |
| DMZ | 172.16.10.0/24 | Internet-facing workloads |
| Cloud Hybrid | 10.10.0.0/16 | Cloud extension |

---
