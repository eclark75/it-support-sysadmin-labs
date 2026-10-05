# it-support-sysadmin-labs
Hands-on lab documentation for Active Directory, Windows Server, network administration, and IT support.
## Active Directory Lab: Automated SMB Share Mapping via Group Policy

### Objective
Design and implement a secure, automated departmental network share for a finance team within an Active Directory Domain Services (AD DS) environment using Group Policy Preferences.

### Environment Architecture
- **Domain Controller:** `DC01-Lab` (Windows Server 2022) | Domain: `adlab.local`
- **Client Workstation:** `CLIENT01-Win10` (Windows 10 Enterprise)
- **Directory Structure:** Dedicated Organizational Unit (`Finance`), Security Group (`Finance-Users`), User Account (`johnd`)

### Implementation Details

#### 1. Storage & Share Configuration
- Created central storage directory on `DC01-Lab` at `C:\FinanceShare`.
- Configured SMB Share permissions: `Finance-Users` granted **Change** and **Read**.
- Enforced NTFS Security DACLs: `Finance-Users` granted **Modify**, **Read & Execute**, **List folder contents**, **Read**, and **Write** (Least Privilege principle).

#### 2. Group Policy Automation
- Configured a new Group Policy Object (`Map-FinanceShare`) linked directly to the `Finance` OU.
- Navigated to `User Configuration > Preferences > Windows Settings > Drive Maps`.
- Configured Drive Map action to **Update**:
  - Target Path: `\\192.168.10.10\FinanceShare`
  - Reconnect: Enabled
  - Label: `Finance Department Share`
  - Drive Letter: `Z:`

#### 3. Verification & Troubleshooting
- Logged into domain-joined client `CLIENT01-Win10` as standard domain user `johnd`.
- Verified automatic policy propagation upon logon; confirmed the `Z:` drive mounted with the custom departmental label.
- Tested end-to-end file creation and modification permissions within the mapped volume.
- Validated endpoint hardening policies (standard domain user access restrictions on administrative command utilities).
