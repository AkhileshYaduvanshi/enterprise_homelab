# DevOps VM Setup (DevSecOps)

## Objective

Deploy a dedicated Ubuntu Server VM to host the DevSecOps infrastructure.

---

## VM Configuration

Hypervisor: Proxmox  
VM Name: DevSecOps 

| Resource | Value |
|----------|-------|
| CPU      | 2 Cores |
| RAM      | 4GB |
| Disk     | 40GB |
| OS       | Ubuntu Server (LTS) |
| Network  | Internal Lab Network |

---

## Static Network Configuration

| Setting | Value |
|----------|-------|
| IP Address | 172.16.1.100 |
| Subnet | 172.16.1.0/24 |
| Gateway | 172.16.1.1 |
| DNS Server | 172.16.1.10 (Active Directory) |
| Search Domain | lab.internal |

> DNS is pointed to Active Directory to support future domain integration and LDAP authentication.

---

## Network Validation

After installation, connectivity was verified:

### Memory Check
```bash
free -h
````

### Interface Check

```bash
ip a
```

### Gateway Reachability

```bash
ping 172.16.1.1
```

### Active Directory Reachability

```bash
ping 172.16.1.10
```

### Wazuh Server Reachability

```bash
ping 172.16.1.50
```

All connectivity tests were successful.

---

## System Update

System packages updated:

```bash
sudo apt update && sudo apt upgrade -y
```

System rebooted after upgrade.

---

## Docker Installation (Official Repository)

Docker was installed using the official Docker repository:

### Install prerequisites

```bash
sudo apt install ca-certificates curl gnupg -y
```

### Add Docker GPG key

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### Add Docker repository

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin -y
```

---

## Docker Validation

Verify Docker installation:

```bash
docker --version
sudo systemctl status docker
```

Allow non-root usage:

```bash
sudo usermod -aG docker devops
```

Re-login required.

Test container execution:

```bash
docker run hello-world
```

Docker successfully installed and validated.
