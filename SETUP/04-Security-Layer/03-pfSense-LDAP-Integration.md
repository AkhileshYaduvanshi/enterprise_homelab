# pfSense LDAP Integration with Active Directory

## Objective

Integrate **pfSense Community Edition** with **Active Directory (lab.internal)** using LDAP authentication.

After this step:

* pfSense authenticates users against AD
* RBAC is enforced via AD groups
* No local pfSense users required
* Centralized identity management is implemented

---

Authentication Flow:

```
User → pfSense Login Page
      → LDAP Bind (svc_ldap_bind)
      → Search User (sAMAccountName)
      → Bind as User
      → Check AD Group Membership
      → Map to pfSense Local Group
      → Assign WebGUI Privileges
```

---

# Environment Details

| Component         | Value              |
| ----------------- | ------------------ |
| Domain            | lab.internal       |
| Base DN           | DC=lab,DC=internal |
| Domain Controller | 172.16.1.10        |
| LDAP Port         | 389                |
| pfSense Version   | 2.8.1-RELEASE      |

---

# Step 1 – Create LDAP Bind Service Account (On Domain Controller)

We create a low-privilege account for LDAP bind.

```powershell
New-ADUser `
-Name "svc_ldap_bind" `
-GivenName "svc" `
-Surname "ldap_bind" `
-SamAccountName "svc_ldap_bind" `
-UserPrincipalName "svc_ldap_bind@lab.internal" `
-Path "OU=ServiceAccounts,OU=lab_internal,DC=lab,DC=internal" `
-AccountPassword (ConvertTo-SecureString "Password!" -AsPlainText -Force) `
-Enabled $true `
-PasswordNeverExpires $true
```

---

# Step 2 – Verify LDAP Connectivity from pfSense

From pfSense shell:

```sh
ldapwhoami -x -H ldap://172.16.1.10 \
-D "svc_ldap_bind@lab.internal" \
-w Password!
```

Expected output:

```
u:LAB\svc_ldap_bind
```

---

# ⚙ Step 3 – Configure LDAP Server in pfSense

Go to:

```
System → User Manager → Authentication Servers → Add
```

## Configuration

### General Settings

| Field            | Value          |
| ---------------- | -------------- |
| Type             | LDAP           |
| Descriptive Name | Lab-AD-Ldap    |
| Hostname         | 172.16.1.10    |
| Port             | 389            |
| Transport        | Standard TCP   |
| Protocol Version | 3              |
| Search Scope     | Entire Subtree |

---

### LDAP Settings

| Field                     | Value                                                           |
| ------------------------- | --------------------------------------------------------------- |
| Base DN                   | DC=lab,DC=internal                                              |
| Authentication Containers | OU=Users,OU=lab_internal,DC=lab,DC=internal                     |
| Bind DN                   | [svc_ldap_bind@lab.internal] |
| User Naming Attribute     | sAMAccountName                                                  |
| Group Naming Attribute    | cn                                                              |
| Group Member Attribute    | memberOf                                                        |
| RFC2307                   | Unchecked                                                       |
| Extended Query            | Unchecked                                                       |
| Anonymous Bind            | Unchecked                                                       |

---

# ⚠ Critical Lesson Learned

Simple LDAP bind does **NOT reliably support**:

```
LAB\username
```

Always use:

```
user@domain
```

or full Distinguished Name.

---

# Step 4 – Set LDAP as Default Authentication Server

Go to:

```
System → User Manager → Settings
```

Set:

```
Authentication Server = Lab-AD-Ldap
```

Save.

---

# Step 5 – Create pfSense Local Groups for RBAC

Go to:

```
System → User Manager → Groups
```

---

## Create pfSense_Admin

* Scope: Local
* Remote Groups: `pfSense_Admin`
* Privileges: `WebCfg - All pages`

---

## Create pfSense_Analyst

* Scope: Local
* Remote Groups: `pfSense_Analyst`
* Privileges:

  * WebCfg - Dashboard
  * Diagnostics pages
  * Status pages

---

# Step 6 – Test User Authentication from pfSense Shell

Test user bind:

```sh
ldapwhoami -x -H ldap://172.16.1.10 \
-D "labanalyst@lab.internal" \
-w Password!
```

Expected:

```
u:LAB\labanalyst
```

---

# Final Login Format (CRITICAL)

Login must use UPN format:

```
labanalyst@lab.internal
```

NOT:

```
labanalyst
```

Because pfSense performs user bind using supplied username.

---

# Troubleshooting Section

## Problem: Username or Password incorrect

### Possible Causes

* Wrong Bind DN format
* Wrong User Naming Attribute (must be `sAMAccountName`)
* Wrong Authentication Container
* Password incorrect
* Using `DOMAIN\username` format
* Logging in without UPN format

---

## Problem: No page assigned to this user

Cause:

* LDAP authentication successful
* But no WebCfg privilege assigned

Fix:

Add at least one WebCfg privilege to mapped group.

---

## Useful Debug Commands

### Test Bind

```sh
ldapwhoami -x -H ldap://172.16.1.10 -D "svc_ldap_bind@lab.internal" -w Password!
```

### Test Search

```sh
ldapsearch -x -H ldap://172.16.1.10 \
-D "svc_ldap_bind@lab.internal" \
-w Password! \
-b "DC=lab,DC=internal" \
"(sAMAccountName=labanalyst)"
```

### Watch Authentication Log

```sh
tail -f /var/log/auth.log
```

---

# Result

After successful configuration:

* pfSense uses centralized AD authentication
* User access controlled by AD group membership
* No local pfSense users required
* Clean RBAC implementation achieved
