# Wazuh 4.14 Migration Troubleshooting

## Scenario

After upgrading Wazuh (4.x → 4.14.x), the following issues occurred:

* Login loop after password reset
* `401 Unauthorized` API errors
* Wazuh API offline
* `wazuh-manager` showing `active (exited)`
* `wazuh-indexer` stopped
* Dashboard showing:

  ```
  Wazuh dashboard server is not ready yet
  ```
* Dashboard working in incognito but not in normal browser

This document explains:

* What caused each issue
* How to diagnose properly
* The correct recovery procedure
* Lessons learned for future migrations

---

# Understanding Wazuh Architecture (Critical Before Troubleshooting)

![Image](https://documentation.wazuh.com/current/_images/deployment-architecture1.png)

![Image](https://documentation.wazuh.com/current/_images/integration-diagram-opensearch1.png)

![Image](https://www.researchgate.net/publication/385664962/figure/fig1/AS%3A11431281289310685%401731136673244/Logical-Process-Flow-Diagram-of-Wazuh-System-32-Phase-2-Implementation-of-Wazuh-System.png)

![Image](https://documentation.wazuh.com/current/_images/sca-sequence-diagram1.png)

Wazuh stack consists of 3 major components:

###  Wazuh Indexer

* Based on OpenSearch
* Runs on port `9200`
* Stores alerts, data, RBAC, internal users

### Wazuh Manager

* Core detection engine
* Runs Wazuh API on port `55000`
* Handles agents, rules, analysis

### Wazuh Dashboard

* Web UI
* Talks to:

  * Indexer (9200)
  * Manager API (55000)

### Dependency Chain

```
Indexer → Manager → Dashboard
```

If Indexer fails → Dashboard fails
If Manager API fails → Dashboard fails

---

# Issue 1: Password Reset Broke Login

## Cause

Running:

```bash
wazuh-passwords-tool.sh --change-all
```

Changes:

* admin
* kibanaserver
* logstash
* readall
* snapshotrestore

But it does NOT automatically update:

* Dashboard keystore
* API user sync

---

## Symptom

Login loop:

```
Wazuh logo → back to sign-in page
```

Dashboard logs:

```
401 Unauthorized
```

---

## Fix

### Update Dashboard Keystore

```bash
sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-keystore add opensearch.username
sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-keystore add opensearch.password
```

Then restart:

```bash
sudo systemctl restart wazuh-dashboard
```

---

# Issue 2: API Showing Offline

## Symptom

Dashboard → API Connections:

```
default → Offline
```

Check:

```bash
sudo ss -tulnp | grep 55000
```

No output.

---

## Cause

Wazuh API (part of manager) not running.

Possible reasons:

* Corrupted RBAC database
* Partial deletion of `/var/ossec/api`
* Configuration mismatch

---

# Critical Mistake (Learning Section)

We deleted:

```bash
sudo rm -rf /var/ossec/api
```

This removed:

* wazuh_apid.py
* API scripts
* Configuration
* Security store

Result:

```
/var/ossec/api/scripts/wazuh_apid.py: No such file
```

Manager failed to start.

---

# Correct Recovery Procedure

## Step 1: Reinstall Wazuh Manager

```bash
sudo apt-get install --reinstall wazuh-manager
```

This restores missing API files.

---

## Step 2: Start Services in Correct Order

```bash
sudo systemctl restart wazuh-indexer
sleep 15

sudo systemctl restart wazuh-manager
sleep 15

sudo systemctl restart wazuh-dashboard
```

---

# Issue 3: Dashboard "Server is not ready yet"

## Symptom

```
Wazuh dashboard server is not ready yet
```

Dashboard logs:

```
ECONNREFUSED 127.0.0.1:9200
```

---

## Cause

Indexer service was stopped.

Check:

```bash
sudo systemctl status wazuh-indexer
```

It showed:

```
inactive (dead)
```

---

## Fix

```bash
sudo systemctl start wazuh-indexer
```

Confirm:

```bash
sudo ss -tulnp | grep 9200
```

---

# Issue 4: Works in Incognito But Not Normal Browser

## Cause

Browser retained:

* old cookies
* session tokens
* localStorage
* security tenant cache

Even clearing "history" did NOT remove site storage.

---

## Fix

In Chrome:

1. Go to:

   ```
   chrome://settings/siteData
   ```
2. Search Wazuh IP
3. Delete all entries

OR

Open DevTools → Application → Clear Site Data

---

# Migration Lessons Learned

### Always Understand Component Dependency

Do NOT troubleshoot dashboard first.

Check in this order:

```
1. Indexer
2. Manager
3. API port 55000
4. Dashboard
```

---

### Never Delete Core Directories Without Knowing Contents

Safe to delete:

```
/usr/share/wazuh-dashboard/data/*
```

Dangerous to delete:

```
/var/ossec/api
```

---

### Use Logs First, Not Assumptions

Best debugging commands:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo journalctl -u wazuh-dashboard -n 50
sudo ss -tulnp | grep 9200
sudo ss -tulnp | grep 55000
```

---

# Health Check Checklist (Post-Recovery)

Run these:

```bash
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
```

Check ports:

```bash
sudo ss -tulnp | grep 9200
sudo ss -tulnp | grep 55000
```

Test indexer:

```bash
curl -k https://localhost:9200
```
