# Active Directory OU Structure  


The Organizational Unit (OU) structure for the Active Directory domain `lab.internal`.

The design follows a clean and scalable structure with:

- Tier-based separation (Tier 0 / Tier 1 / Tier 2)
- Centralized group management
- Centralized service account management
- Simple and consistent naming conventions

The structure is built to support enterprise-style administration, RBAC, and future expansion.

![Overall Architecture](../../assets/diagrams/ou-arch.png)

## Root OU Structure

```

lab.internal
│
├── Tier0
├── Tier1
├── Tier2
├── Groups
├── ServiceAccounts
└── Disabled

```

## 00_Tier0

Tier 0 contains the most privileged objects in the domain.

```

Tier0
├── DomainControllers
└── AdminAccounts

```

### DomainControllers
Contains domain controller computer objects.

### AdminAccounts
Contains highly privileged administrative accounts (e.g., domain admins).

## 01_Tier1

Tier 1 contains server systems and server-level administrators.

```

Tier1
├── Servers
└── AdminAccounts

```

### Servers
Contains domain-joined server systems (e.g., SIEM, Git, CI/CD).

### AdminAccounts
Contains accounts responsible for managing servers.


## 02_Tier2

Tier 2 contains standard user systems and regular users.

```

Tier2
├── Workstations
└── Users

```

### Workstations
Contains domain-joined endpoint systems.

### Users
Contains standard human user accounts.

Example users:
- labadmin
- labanalyst
- labaudit
- akhilesh


## Groups

Contains all security groups used for RBAC.

```

Groups
├── Teams
└── Tools

```

### Teams
Represents job roles.

Examples:
- secops
- devops
- infraops
- auditor

### Tools
Represents application or system access groups.

Examples:
- wazuh-admin
- wazuh-analyst
- gitea-dev
- jenkins-admin
- vpn-users
- linux-admin

Team groups are nested into tool groups to automate permission assignment.


## ServiceAccounts

Contains all non-human service accounts.

```

ServiceAccounts
├── LDAP
├── Automation
└── Monitoring

```

Examples:
- svc-wazuh-ldap
- svc-gitea-ldap
- svc-jenkins-ldap
- svc-pfsense-ldap

Service accounts are separated from human users for clarity and auditing.


## Disabled

Contains disabled or deprecated accounts and systems.

Used for:
- Decommissioned users
- Retired service accounts
- Old computer objects


## Design Principles

- Keep structure simple and consistent
- Separate identity from permissions
- Avoid unnecessary OU nesting
- Use groups for permission management
- Keep service accounts centralized
- Maintain clean naming conventions

