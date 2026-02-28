# DC-01 VM Creation in Proxmox

DC-01 is the domain controller VM running Windows Server 2022 Core. This document covers the complete VM creation process in Proxmox.

Your DC-01 VM is configured as follows:

### CPU: 2 Cores (Not 1)

- **Active Directory needs multi-core** for:
  - LDAP query processing
  - DNS resolution
  - Group Policy application
  - User authentication
- 2 cores provides responsive AD without excessive resource use
- pfSense (1 core) is I/O-bound; AD is CPU-bound

### Memory: 2 GB

- **Minimum for Windows Server 2022 Core:** 512 MB
- **Recommended for AD services:** 2 GB
- **Your setup includes:**
  - Active Directory Database (~200-500 MB depending on users)
  - DNS service
  - DHCP service
  - Group Policy
  - System cache and overhead
- 2 GB ensures responsive domain operations

### Storage: 60 GB

- **Windows Server Core install:** ~4-5 GB
- **AD Database:** Grows with users/computers
- **NTDS.dit (AD database):** Initial ~10 MB, grows with objects
- **System Reserve:** ~10 GB for logs, updates, snapshots
- **Buffer:** Extra space for growth
