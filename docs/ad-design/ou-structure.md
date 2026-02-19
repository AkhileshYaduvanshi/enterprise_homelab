# OU Structure – Enterprise Cybersecurity Lab

## Overview
This document defines the Organizational Unit (OU) structure for the Active Directory domain `lab.internal`.

The design implements a **Hybrid RBAC model** combining:
- Persona-based user identities
- Team-based role groups
- Tool-specific permission groups
- Service accounts for integrations

This layered approach improves scalability, simplifies onboarding, and supports realistic enterprise security workflows.

---

## Design Principles

- Separate identity from permissions
- Use team groups for role assignment
- Use tool groups for application access
- Implement group nesting for RBAC automation
- Separate service accounts from human users
- Enable detection engineering and attack simulation scenarios

---

## Root OU Layout

```

lab.internal
│
├── Users
├── Service-Accounts
├── Groups
└── Computers

```

---

## Users OU
This OU contains human identities representing enterprise personas.

```

Users
├── Admin
├── Analysts
├── Audit
└── Employees

```

### Example User Accounts

| Username | Persona | Description |
|---------|---------|-------------|
| labadmin | Infrastructure Admin | Full administrative access |
| labanalyst | SOC Analyst | Monitoring and detection workflows |
| labaudit | Auditor | Read-only investigation and compliance |
| akhilesh | Employee | Standard domain user |

---

## ⚙️ Service-Accounts OU
Dedicated OU for application and integration identities.

```

Service-Accounts
├── LDAP
├── Automation
└── Monitoring

```

### Example Accounts

| Account | Purpose |
|--------|---------|
| svc-wazuh-ldap | Wazuh LDAP binding |
| svc-gitea-ldap | Gitea LDAP integration |
| svc-jenkins-ldap | Jenkins authentication |
| svc-pfsense-ldap | pfSense LDAP integration |

---

## Groups OU (Hybrid RBAC)

```

Groups
├── Teams
└── Tools

```

---

## Team-Based Groups (Identity Layer)
These groups represent job roles and simplify user onboarding.

```

Groups
└── Teams
├── secops
├── devops
├── infraops
└── auditor

```

### Purpose
- Assign users based on job function
- Avoid direct permission assignment
- Provide scalable identity management

---

## Tool-Based Groups (Permission Layer)
These groups define application-level access used by LDAP integrations.

```

Groups
└── Tools
├── wazuh-admin
├── wazuh-analyst
├── wazuh-audit
├── gitea-admin
├── gitea-dev
├── jenkins-admin
├── jenkins-dev
├── vpn-users
└── linux-admin

```

---

## Group Nesting Model (Hybrid RBAC)

Team groups are nested inside tool groups to automate access assignment.

### Example Mapping

| Team Group | Nested Into | Result |
|------------|-------------|--------|
| secops | wazuh-analyst | SOC monitoring access |
| auditor | wazuh-audit | Read-only SIEM access |
| infraops | wazuh-admin | SIEM administration |
| devops | gitea-dev | Repository access |
| devops | jenkins-dev | CI/CD pipeline access |
| infraops | jenkins-admin | Pipeline administration |
| secops | vpn-users | Secure remote access |

---

## User → Team Mapping Example

| User | Team Membership |
|------|----------------|
| labadmin | infraops |
| labanalyst | secops |
| labaudit | auditor |
| akhilesh | employee / vpn-users |

---

## Computers OU
Organizes domain-joined machines for easier GPO targeting.

```

Computers
├── Servers
├── Workstations
├── Security
└── DevSecOps

```

---

## Security Benefits

- Prevents direct permission sprawl
- Enables least privilege access
- Simplifies onboarding and offboarding
- Supports realistic privilege escalation scenarios
- Enables detection engineering for identity attacks
- Improves audit visibility

---

## Future Enhancements

- Tiered administration model
- Privileged Access Workstations (PAW)
- Just-in-Time admin accounts
- Shadow admin detection
- Azure AD hybrid identity mapping
- Identity honeypot accounts

---
