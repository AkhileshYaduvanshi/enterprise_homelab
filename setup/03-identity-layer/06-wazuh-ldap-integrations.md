#  Wazuh 4.14+ LDAP Integration (Active Directory)

### Complete Production Walkthrough + Troubleshooting Guide

---

# 📌 Environment Overview

| Component          | Value                                                               |
| ------------------ | ------------------------------------------------------------------- |
| Wazuh Version      | 4.14.x                                                              |
| OpenSearch Version | 2.19.x                                                              |
| Domain             | lab.internal                                                        |
| Domain Controller  | DC-01.lab.internal                                                  |
| DC IP              | 172.16.1.10                                                         |
| LDAP Port Used     | 389                                                                 |
| Service Account    | wazuh-ldap                                                          |
| LDAP Bind DN       | CN=wazuh-ldap,OU=ServiceAccounts,OU=lab_internal,DC=lab,DC=internal |

---

# 🏗 Architecture Overview

![Image](https://documentation.wazuh.com/current/_images/deployment-architecture1.png)

![Image](https://documentation.wazuh.com/current/_images/wazuh-indexer1.png)

![Image](https://www.tecracer.com/blog/img/2023/05/image-20230403211849575.png)

![Image](https://media2.dev.to/dynamic/image/width%3D800%2Cheight%3D%2Cfit%3Dscale-down%2Cgravity%3Dauto%2Cformat%3Dauto/https%3A%2F%2Fraw.githubusercontent.com%2Fvumdao%2Faws-vpc-opensearch%2Fmaster%2Fdocs%2Fimages%2Farchitect.png)

Authentication Flow:

1. User logs into Wazuh Dashboard
2. Dashboard sends credentials to OpenSearch
3. OpenSearch Security:

   * Authenticates via LDAP
   * Authorizes via LDAP role mapping
4. Roles mapped to OpenSearch roles
5. Index permissions applied
6. Dashboard loads

---

# Problems We Faced (Important Lessons)

### API Not Running

Cause:

* Corrupted API directory
* Migration artifact from previous version

Fix:

```bash
sudo rm -rf /var/ossec/api/configuration/security
sudo systemctl restart wazuh-manager
```

---

### Wazuh Dashboard "Server not ready yet"

Cause:

* wazuh-indexer not running
* Port 9200 not listening

Check:

```bash
sudo ss -tulnp | grep 9200
```

Fix:

```bash
sudo systemctl restart wazuh-indexer
```

---

### securityadmin.sh Not Working

Cause:

* Used old syntax (`-c`)
* JAVA_HOME not exported correctly
* Using `sudo` without `-E`

Correct Command (Wazuh 4.14+ syntax):

```bash
export OPENSEARCH_JAVA_HOME=/usr/share/wazuh-indexer/jdk

sudo -E ./securityadmin.sh \
-cd /etc/wazuh-indexer/opensearch-security/ \
-icl -nhnv \
-h localhost -p 9200 \
-cert /etc/wazuh-indexer/certs/admin.pem \
-key /etc/wazuh-indexer/certs/admin-key.pem \
-cacert /etc/wazuh-indexer/certs/root-ca.pem
```

---

# Step 1 — Create LDAP Service Account (AD Side)

On Domain Controller:

```powershell
New-ADUser -Name "wazuh-ldap" `
-SamAccountName "wazuh-ldap" `
-AccountPassword (ConvertTo-SecureString "StrongPassword123!" -AsPlainText -Force) `
-Enabled $true `
-Path "OU=ServiceAccounts,OU=lab_internal,DC=lab,DC=internal"
```

This account:

* Is used only for binding/searching
* Should not have admin privileges

---

# Step 2 — Verify LDAP Connectivity

From Wazuh server:

```bash
ldapsearch -H ldap://172.16.1.10:389 \
-D "CN=wazuh-ldap,OU=ServiceAccounts,OU=lab_internal,DC=lab,DC=internal" \
-w 'StrongPassword123!' \
-b "DC=lab,DC=internal" \
"(sAMAccountName=labanalyst)"
```

If this fails → OpenSearch LDAP will fail.

---

# Step 3 — Configure LDAP in OpenSearch

File:

```
/etc/wazuh-indexer/opensearch-security/config.yml
```

### Authentication Section

```yaml
ldap:
  description: "Authenticate via LDAP"
  http_enabled: true
  transport_enabled: false
  order: 1
  http_authenticator:
    type: basic
    challenge: false
  authentication_backend:
    type: ldap
    config:
      enable_ssl: false
      hosts:
        - 172.16.1.10:389
      bind_dn: "CN=wazuh-ldap,OU=ServiceAccounts,OU=lab_internal,DC=lab,DC=internal"
      password: "StrongPassword123!"
      userbase: "DC=lab,DC=internal"
      usersearch: '(sAMAccountName={0})'
```

---

### Authorization Section

```yaml
roles_from_myldap:
  http_enabled: true
  authorization_backend:
    type: ldap
    config:
      enable_ssl: false
      hosts:
        - 172.16.1.10:389
      bind_dn: "CN=wazuh-ldap,OU=ServiceAccounts,OU=lab_internal,DC=lab,DC=internal"
      password: "StrongPassword123!"
      rolebase: "OU=Groups,OU=lab_internal,DC=lab,DC=internal"
      rolesearch: '(member={0})'
      rolename: cn
      resolve_nested_roles: true
```

---

# Step 4 — Create OpenSearch Roles

File:

```
/etc/wazuh-indexer/opensearch-security/roles.yml
```

### Admin Role

```yaml
wazuh_admin_role:
  cluster_permissions:
    - cluster_all
  index_permissions:
    - index_patterns:
        - "*"
      allowed_actions:
        - indices_all
  tenant_permissions:
    - tenant_patterns:
        - "*"
      allowed_actions:
        - kibana_all_write
```

---

### Analyst Role

```yaml
wazuh_analyst_role:
  cluster_permissions:
    - cluster_composite_ops_ro
  index_permissions:
    - index_patterns:
        - "wazuh-*"
        - ".kibana*"
        - ".plugins*"
      allowed_actions:
        - read
        - search
  tenant_permissions:
    - tenant_patterns:
        - "*"
      allowed_actions:
        - kibana_all_read
```

---

# Step 5 — Map AD Groups to Roles

File:

```
roles_mapping.yml
```

```yaml
wazuh_admin_role:
  backend_roles:
    - "Wazuh_Admin"

wazuh_analyst_role:
  backend_roles:
    - "Wazuh_Analyst"
```

Important:
`rolename: cn` means OpenSearch extracts only:

```
Wazuh_Admin
Wazuh_Analyst
```

NOT full DN.

---

# Step 6 — Apply Configuration

```bash
sudo -E ./securityadmin.sh \
-cd /etc/wazuh-indexer/opensearch-security/ \
-icl -nhnv \
-h localhost -p 9200 \
-cert /etc/wazuh-indexer/certs/admin.pem \
-key /etc/wazuh-indexer/certs/admin-key.pem \
-cacert /etc/wazuh-indexer/certs/root-ca.pem
```

Restart:

```bash
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard
```

---

# Debugging Methodology

## Check Roles Mapping

```bash
curl -k --cert admin.pem --key admin-key.pem \
--cacert root-ca.pem \
https://localhost:9200/_plugins/_security/api/rolesmapping
```

---

## Check LDAP Auth Config

```bash
curl -k --cert admin.pem --key admin-key.pem \
--cacert root-ca.pem \
https://localhost:9200/_plugins/_security/api/securityconfig
```

---

## Common Errors

### ❌ Invalid username/password

Cause:

* LDAP bind failed
* SSL mismatch
* Wrong port

---

### ❌ No permissions for indices:data/read/search

Cause:

* Role exists but index permissions missing

Fix:
Add wazuh-* and .kibana* index patterns

---

### ❌ securityadmin.sh not applying

Cause:

* Wrong flag (-c instead of -cd)
* Forgot sudo -E
* JAVA_HOME not exported
