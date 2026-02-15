# Wazuh Upgrade & Troubleshooting Documentation

## Environment

* **Wazuh Version:** `WAZUH_VERSION="v4.14.3"` (previously `4.7.5`)
* OS: Ubuntu Server
* Services:

  * Wazuh Indexer
  * Wazuh Manager
  * Wazuh Dashboard
* Installation method: APT packages
* Upgrade method: `apt --only-upgrade`

---

# Upgrade Procedure Performed

## 1. Services Stopped

Before upgrading, the following services were stopped:

```bash
sudo systemctl stop wazuh-indexer
sudo systemctl stop wazuh-manager
sudo systemctl stop wazuh-dashboard
sudo systemctl stop filebeat
```

---

## 2. Component Upgrade Order

The components were upgraded individually using APT.

### Step 1 – Upgrade Wazuh Indexer

```bash
sudo apt --only-upgrade install wazuh-indexer
```

After upgrade:

```bash
sudo systemctl start wazuh-indexer
```

---

### Step 2 – Upgrade Wazuh Manager

```bash
sudo apt --only-upgrade install wazuh-manager
```

After upgrade:

```bash
sudo systemctl start wazuh-manager
```

---

### Step 3 – Upgrade Wazuh Dashboard

```bash
sudo apt --only-upgrade install wazuh-dashboard
```

After upgrade:

```bash
sudo systemctl start wazuh-dashboard
```

---

# Issues Encountered After Upgrade

---

## Issue 1 – Dashboard Not Listening on Port 443

### Observed:

```bash
sudo ss -tulnp | grep 443
```

Returned nothing.

But:

```bash
sudo systemctl status wazuh-dashboard
```

Showed the service was repeatedly restarting.

### Logs showed:

```
Error: ENOENT: no such file or directory
open '/etc/wazuh-dashboard/certs/dashboard-key.pem'
```

### Root Cause:

* SSL was enabled in `/etc/wazuh-dashboard/opensearch_dashboards.yml`
* Certificate files were missing
* Dashboard failed to start because SSL key file did not exist

---

## Fix Applied

Temporarily disabled SSL:

```yaml
server.ssl.enabled: false
#server.ssl.key:
#server.ssl.certificate:
```

Restarted dashboard:

```bash
sudo systemctl restart wazuh-dashboard
```

---

## Issue 2 – Browser Error

Accessing:

```
https://172.16.1.50
```

Received:

```
ERR_SSL_PROTOCOL_ERROR
```

### Cause:

* SSL was disabled
* Browser was still using HTTPS

### Resolution:

* Access via HTTP:

```
http://172.16.1.50:443
```

* Recommended: change port back to 5601 for clarity when SSL is disabled.

---

## Issue 3 – Login Credentials No Longer Working

### Investigation:

```bash
cat /etc/wazuh-indexer/opensearch-security/internal_users.yml
```

File showed demo users:

* admin (Demo admin user)
* anomalyadmin
* kibanaserver
* kibanaro
* logstash
* readall
* snapshotrestore

### Root Cause:

* Security index reinitialized during indexer upgrade
* Previous auto-generated passwords were lost
* System reverted to default demo users

---

# Current State

* Wazuh Indexer upgraded
* Wazuh Manager upgraded
* Wazuh Dashboard upgraded
* SSL disabled on dashboard (temporary)
* Security database reset to demo configuration
* Admin password must be reset manually
