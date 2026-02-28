## Security Monitoring VM Creation (Wazuh)

Wazuh - an open-source Security Information and Event Management (SIEM) and Extended Detection and Response (XDR) platform.

Wazuh will provide:
- **Security Event Monitoring** - Real-time log analysis and threat detection
- **Intrusion Detection** - Network and host-based intrusion detection
- **Vulnerability Detection** - Automated security vulnerability scanning
- **File Integrity Monitoring** - Track changes to critical system files
- **Compliance Monitoring** - PCI-DSS, HIPAA, GDPR compliance checks
- **Log Analysis** - Centralized log collection and analysis

### VM Specifications

- **Cores:** 2
  - **Reason:** Minimum 2 cores required for Wazuh manager performance
- **RAM:** 4096 MB (4 GB)
  - **Reason:** Minimum requirement for Wazuh in small lab environment
- **Disk Size:** 50 GB
  - **Reason:** 50 GB provides adequate space for logs, indices, and system growth


#### Operating System
- **Distribution:** Ubuntu 24.04.3 LTS
- **Image:** ubuntu-24.04.3-live-server-amd64.iso

**Why Ubuntu 24.04.3 LTS:**
- Official Wazuh support and documentation
- Long-term support (5 years)
- Stable and well-tested
- Easy package management with APT
- Resource-efficient for security appliances

**Network Interface (net0):**Bridge:** vmbr1 (Internal_Office_LAN)

- Connected to vmbr1 (192.168.1.0/24 network)
- Can monitor all internal VMs on same network
- Will receive IP via DHCP from pfSense or static configuration
- Accessible from bastion host for management
