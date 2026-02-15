# Gitea Installation (Secure Git Server)

## Objective

Deploy a lightweight, self-hosted Git server inside the Enterprise Home Lab.

Gitea will serve as:

- Internal source code repository
- Foundation for CI/CD pipelines
- Entry point for DevSecOps practices
- Supply chain security testing platform

This service runs inside Docker and is fully monitored by Wazuh.

---

## Why Gitea?

In enterprise environments, Git servers are critical because they:

- Store source code
- Trigger CI/CD pipelines
- Contain secrets if misconfigured
- Act as supply chain entry points

We selected **Gitea** because:

- Lightweight (low RAM usage)
- Supports LDAP integration
- Easy to containerize
- Suitable for home lab scale

---

## Architecture Position

```

Active Directory (172.16.1.10)
↑
↓
DevSecOps-01 VM (172.16.1.100)
├── Docker
├── Gitea
└── Wazuh Agent

```

Access URL:

```

[http://172.16.1.100:3000](http://172.16.1.100:3000)

````

---

## Step 1 – Create Persistent Storage

We do NOT run containers without persistent storage.

Create directories:

```bash
sudo mkdir -p /opt/devsecops/gitea/data
sudo chown -R devops:devops /opt/devsecops
````

### Why?

If the container is deleted, Docker volumes inside the container are lost.

By mapping:

```
/opt/devsecops/gitea/data → /data (inside container)
```

All repositories and configuration persist on disk.

Enterprise rule:
Always separate application from data.

---

## Step 2 – Create Dedicated Docker Network

```bash
docker network create devsecops-net
```

### Why?

This isolates DevSecOps services from default Docker network.

Later:

* Jenkins
* Registry
* Scanners

Will use this same isolated network.

Enterprise principle:
Service isolation improves security and observability.

---

## Step 3 – Deploy Gitea Container

```bash
docker run -d \
  --name gitea \
  --network devsecops-net \
  -p 3000:3000 \
  -p 2222:22 \
  -v /opt/devsecops/gitea/data:/data \
  -e USER_UID=1000 \
  -e USER_GID=1000 \
  --restart always \
  gitea/gitea:latest
```

---

## What Each Parameter Means

| Option                             | Explanation                 |
| ---------------------------------- | --------------------------- |
| -d                                 | Run container in background |
| --name gitea                       | Container name              |
| --network devsecops-net            | Attach to isolated network  |
| -p 3000:3000                       | Web UI port                 |
| -p 2222:22                         | SSH for Git operations      |
| -v /opt/devsecops/gitea/data:/data | Persistent storage mapping  |
| -e USER_UID=1000                   | File ownership alignment    |
| -e USER_GID=1000                   | File ownership alignment    |
| --restart always                   | Auto start after reboot     |

---

## Step 4 – Verify Container

Check running containers:

```bash
docker ps
```

Expected output includes:

```
gitea/gitea
0.0.0.0:3000->3000
0.0.0.0:2222->22
```

Check logs:

```bash
docker logs gitea
```

No critical errors should appear.

---

## Step 5 – Initial Web Setup

Open browser from Jump Host:

```
http://172.16.1.100:3000
```

Configuration:

* Database: SQLite (lightweight for lab)
* Application URL: [http://172.16.1.100:3000](http://172.16.1.100:3000)
* Create admin account

Click Install.

---

## Step 6 – Validate Git Functionality

From Jump Host:

Clone test repository:

```bash
git clone http://172.16.1.100:3000/<username>/<repo>.git
```

If clone works → Git server is operational.

---

## Persistence Test

Reboot VM:

```bash
sudo reboot
```

After reboot:

```bash
docker ps
```

Container should auto-start because of:

```
--restart always
```

This confirms enterprise-level service persistence.
