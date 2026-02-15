# Compelete Cybersecurity Home Lab

![Status](https://img.shields.io/badge/Status-Active-green)
![Hardware](https://img.shields.io/badge/RAM-16GB-blue)
![Focus](https://img.shields.io/badge/Focus-SOC%20Engineering-red)
![Architecture](https://img.shields.io/badge/Design-Segmented-orange)

A full enterprise-style Security Operations lab built on a 16GB RAM system.

This project simulates a real-world organization with identity management, firewall segmentation, SIEM monitoring, containerized services, Kubernetes workloads, CI/CD automation, red team simulations, and incident response workflows.

**Note - This is not a tool installation project — it is a structured security engineering journey.**

## Why This Lab Is Different

This project demonstrates:

- Infrastructure design under hardware constraints
- Enterprise-style segmentation
- Detection engineering maturity
- DevSecOps integration
- Threat simulation & validation
- Real incident response lifecycle

Every phase builds upon the previous one:

Infrastructure → Visibility → Hardening → Attack Simulation → Response

---

# Project Objective

To design, deploy, validate, harden, and simulate attacks against a production-style infrastructure using limited hardware.

This lab focuses on:

- Infrastructure Engineering
- Network Segmentation
- Active Directory Deployment
- SIEM Integration (Wazuh)
- Dockerized Services
- Kubernetes & CI/CD
- Detection Engineering
- MITRE ATT&CK Mapping
- Incident Response

---

# Hardware Constraints & Philosophy

This lab is built on:

- 16GB RAM
- Limited CPU cores
- Single primary server

Because of hardware limitations:

- Not all services run simultaneously
- Docker is used to reduce memory usage
- Windows Server Core is preferred over GUI
- Lightweight Linux VMs are prioritized
- Services are deployed in phases

---

# High-Level Architecture

Core Hypervisor: Proxmox VE  
Firewall & VPN: pfSense  
Identity Layer: Active Directory (Windows Server Core)  
Monitoring Layer: Wazuh SIEM  
Container Services: Docker (Pi-hole, Nginx Proxy, Internal Apps)  
Orchestration: Lightweight Kubernetes (k3s)  
Access Control: Jump Host  
Threat Simulation: Red Team Canary  
Mapping: MITRE ATT&CK Navigator  
Decoy Assets: Honeypot  

External Devices:

- User Laptop (VPN Client + Domain User Simulation)
- Attacker Laptop (Isolated Red Team Environment)

---

# Project Lifecycle

This lab is built in structured phases.

<img width="1320" height="1237" alt="CIpher drawio-2" src="https://github.com/user-attachments/assets/43518f17-6c41-4834-be52-435570da009f" />

---

# Continuous Improvement

This lab will evolve to include:

- Additional endpoints
- EDR integration
- Advanced threat hunting
- Purple team simulations
- Automation playbooks
- Performance tuning under 16GB

---

# Author

Akhilesh  
Cybersecurity Analyst  
Focused on Detection Engineering, SOC Operations & Infrastructure Security
