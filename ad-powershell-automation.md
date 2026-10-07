# Enterprise Active Directory Deployment & Automated User Onboarding

## 1. Project Overview & Architecture
* **Objective:** Configure an enterprise Windows Server 2022 Active Directory domain controller and implement scalable PowerShell automation for bulk user onboarding across departmental Organizational Units (OUs).
* **Environment:** Hypervisor: Oracle VM VirtualBox | OS: Windows Server 2022 Datacenter (DC01-Lab) | Workstation: Windows 10 Enterprise (CLIENT01-Win10) | Network: Internal Isolated Subnet (intnet) | Identity Management: Active Directory Domain Services (AD DS)

## 2. Infrastructure Setup & Domain Provisioning
* Installed and promoted DC01-Lab as primary Domain Controller.
* Configured Active Directory Domain Services (AD DS), DNS Server, and DHCP scopes for client lease assignment.
* Joined client workstation CLIENT01-Win10 to domain; validated name resolution across intnet.

## 3. Directory Structure & Group Policy Objects (GPO)
* Structured departmental OUs: IT, Operations, and Sales.
* Configured Password & Account Lockout Policies to enforce complexity thresholds.
* Implemented Workstation Hardening GPOs: restricted Control Panel access, blocked unauthorized USB mass storage, applied baseline desktop policies.

## 4. Automation: Bulk User Provisioning via PowerShell
* Ingested user records via CSV (C:\ADLab\users.csv).
* Dynamically created missing departmental Organizational Units.
* Auto-generated standard sAMAccountNames and UPNs.
* Provisioned accounts with temporary credentials (P@ssw0rd2026!Lab) forcing initial logon password reset.
* # Enterprise Linux Host Access & OpenSSH Hardening Guide

## 1. Project Overview & Architecture
* **Objective:** Establish passwordless, cryptographic public-key authentication from a Windows client (PowerShell/OpenSSH) to a local Ubuntu/WSL environment while disabling insecure authentication vectors.
* **Environment:** Client: Windows 11 OpenSSH (gotvi) | Target Server: Ubuntu 24.04/WSL (Torahlife1975) | Service User: l_support | Key Algorithm: ED25519 (ssh-ed25519)

## 2. Key Generation & Distribution
* Generated ED25519 key pair on host: `ssh-keygen -t ed25519 -C "gotvi@Torahlife1975"`
* Extracted public key directly into server path: `sudo cp /mnt/c/Users/gotvi/.ssh/id_ed25519.pub /home/l_support/.ssh/authorized_keys`

## 3. POSIX Permissions & Privilege Management
* Enforced strict ownership: `sudo chown -R l_support:l_support /home/l_support`
* Set boundary permissions: `chmod 755 /home/l_support`, `chmod 700 /home/l_support/.ssh`, `chmod 600 /home/l_support/.ssh/authorized_keys`
* Granted administrative rights: `sudo usermod -aG sudo l_support`

## 4. OpenSSH Daemon Hardening
Configured `/etc/ssh/sshd_config`:
* `PermitRootLogin no`
* `PasswordAuthentication no`
* `PubkeyAuthentication yes`
* `AuthorizedKeysFile .ssh/authorized_keys`

## 5. Verification & Root Cause Analysis
* Diagnosed initial publickey failure using foreground debugging (`sudo /usr/sbin/sshd -d -p 2222`).
* Corrected cryptographic mismatch in authorized_keys.
* Successfully verified zero-password cryptographic login: `ssh -i "$HOME\.ssh\id_ed25519" l_support@127.0.0.1`

## 5. Verification & Audit Query
Validated accounts using:
```powershell
Get-ADUser -Filter 'Department -like "*"' -Properties Department, Title | Select-Object Name, SamAccountName, Department, Title, Enabled | Format-Table -AutoSize
