# Project 8: Enterprise Linux System Administration, Security & Automation

## 1. Project Overview
This project documents the provisioning, access control hardening, and automation workflows implemented on an enterprise Ubuntu Linux environment (via WSL2). The implementation enforces the Principle of Least Privilege (PoLP), strict directory permissions following the Filesystem Hierarchy Standard (`/opt`), automated operational telemetry, and host-level firewall enforcement.

---

## 2. Environment Specifications
- **Operating System:** Ubuntu 26.04 LTS (Linux Kernel 6.18)
- **Deployment Platform:** Native Windows Subsystem for Linux (WSL2)
- **Core Tooling:** `bash`, `ufw`, `iptables`, `coreutils`, `net-tools`

---

## 3. Implementation Steps & Command Execution

### Phase 1: Identity & Access Management (PoLP)
To prevent operational risks associated with root logins, a departmental group was established alongside a dedicated operational support account mapped with explicit elevated administrative rights:
```bash
# Provision departmental group and operational user account
sudo addgroup it_ops
sudo adduser --gecos "" l_support
sudo usermod -aG sudo,it_ops l_support

# Verify security group memberships and UID/GID mapping
id l_support
# Create directory structure
sudo mkdir -p /opt/it_dept/scripts

# Set ownership and enforce 770 permissions
sudo chown -R root:it_ops /opt/it_dept
sudo chmod -R 770 /opt/it_dept

# Verify access string
ls -ld /opt/it_dept
#!/bin/bash
echo "=== System Hostname & Uptime ==="
hostnamectl status | grep -E "Static hostname|Operating System|Kernel"
uptime -p

echo -e "\n=== Memory Usage ==="
free -h

echo -e "\n=== Disk Utilization ==="
df -h --output=source,size,used,avail,pcent,target -x tmpfs -x devtmpfs

echo -e "\n=== Network Interface & IP ==="
ip -br addr show
# Set baseline traffic rules
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Whitelist standard SSH port
sudo ufw allow 22/tcp

# Enable and inspect firewall status
sudo ufw --force enable
sudo ufw status verbose
