# Jenkins Deployment & DevSecOps Security Monitoring

## Objective

Deploy Jenkins as the CI/CD engine for the DevSecOps layer and integrate full security monitoring through Wazuh.

This phase introduces:

- CI/CD automation
- Container build capability
- Security observability
- File integrity monitoring
- DevSecOps attack surface visibility

---

#  Architecture Overview

Current DevSecOps Stack:

DevSecOps-01 (172.16.1.100)

- Docker Engine
- Gitea (Git Server)
- Jenkins (CI/CD)
- Wazuh Agent

External Integrations:

- Active Directory (172.16.1.10)
- Wazuh Server (172.16.1.50)

Logical Flow:

User → Gitea → Jenkins → Docker → (Future: Registry) → Runtime

All activity monitored by Wazuh.

---

# Jenkins Deployment

## Why Jenkins?

Jenkins acts as the automation engine that:

- Pulls code from Git
- Runs builds
- Executes security scans
- Builds containers
- Deploys artifacts

In enterprise, Jenkins is a high-value target.

---

## Step 1 – Create Persistent Directory

```bash
sudo mkdir -p /opt/devsecops/jenkins
sudo chown -R devops:devops /opt/devsecops/jenkins
````

Why?

Containerized services must not lose configuration after reboot.

All Jenkins data is stored in:

```
/opt/devsecops/jenkins
```

---

## Step 2 – Deploy Jenkins Container

```bash
docker run -d \
  --name jenkins \
  --network devsecops-net \
  -p 8080:8080 \
  -p 50000:50000 \
  -v /opt/devsecops/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart always \
  jenkins/jenkins:lts
```

---

## Explanation of Key Options

| Option                    | Purpose                               |
| ------------------------- | ------------------------------------- |
| -p 8080:8080              | Web UI                                |
| -p 50000:50000            | Agent communication                   |
| -v /opt/devsecops/jenkins | Persistent configuration              |
| -v /var/run/docker.sock   | Allows Jenkins to build Docker images |
| --network devsecops-net   | Isolated DevSecOps network            |
| --restart always          | Auto-start after reboot               |

---

## Important Security Note

Mounting:

```
/var/run/docker.sock
```

gives Jenkins the ability to control Docker on the host.

If Jenkins is compromised:

An attacker can control the entire DevSecOps VM.

This will later be used for attack simulation and detection engineering.

---

# Initial Jenkins Setup

Access:

```
http://172.16.1.100:8080
```

Retrieve initial admin password:

```bash
cat /opt/devsecops/jenkins/secrets/initialAdminPassword
```

Install suggested plugins.

Create local admin user (LDAP integration will be done later).

---

# Wazuh Monitoring – Docker Visibility

After deployment, Wazuh detected:

* Docker image pull
* Container start
* Network join events
* sudo command execution

Example detected events:

* Docker image pulled
* Container joined devsecops-net
* Successful sudo to root

This confirms:

System-level monitoring is functioning.

---

# Identified Monitoring Gap

Observation:

Wazuh saw Docker daemon activity but did NOT see:

* Jenkins login attempts
* Job creation
* Credential changes
* Plugin installation

Reason:

Jenkins logs are inside the container filesystem.

This is a common enterprise monitoring blind spot.

---

# Enable File Integrity Monitoring (FIM)

We configured Wazuh Agent on DevSecOps-01 to monitor Jenkins and Gitea directories.

Edit:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Inside `<syscheck>` section, add:

```xml
<directories check_all="yes" realtime="yes">/opt/devsecops/jenkins</directories>
<directories check_all="yes" realtime="yes">/opt/devsecops/gitea</directories>
```

Restart agent:

```bash
sudo systemctl restart wazuh-agent
```

---

## What This Detects

* Job creation
* Job modification
* Credential changes
* Plugin installation
* Configuration tampering
* Repository modification

This enables DevSecOps-level detection engineering.

---

# Enable Gitea Log Monitoring

To monitor authentication and Git activity, add:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/opt/devsecops/gitea/data/log/gitea.log</location>
</localfile>
```

Restart agent:

```bash
sudo systemctl restart wazuh-agent
```

Now Wazuh can detect:

* Failed login attempts
* Authentication events
* Git operations
* Admin activity
