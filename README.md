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
---

## Project 2: Bulk Active Directory User Provisioning via PowerShell Automation

### Overview
Automated the enterprise onboarding pipeline by parsing an HR-generated CSV roster (`employees.csv`) to bulk-create Active Directory user accounts using PowerShell scripting, enforcing password baselines and target OU placement.

### Key Implementation Details
- **Data Source:** Structured employee roster (`C:\employees.csv`) containing departmental attributes, titles, and names.
- **Automation Logic:**
  - Standardized username generation formula: `FirstInitial + LastName` (e.g., `sconnor`).
  - UPN formatting aligned with domain suffix (`@adlab.local`).
  - Secure credential assignment using `ConvertTo-SecureString` with mandatory password change flag (`-ChangePasswordAtLogon $true`).
  - Pre-execution conflict detection (`Get-ADUser`) to prevent account collisions.
  - Automated targeted placement into the `Finance` Organizational Unit (`OU=Finance,DC=adlab,DC=local`).

### Automation Script (`BulkUserProvisioning.ps1`)
```powershell
Import-Module ActiveDirectory

$csvPath = "C:\employees.csv"

if (-not (Test-Path $csvPath)) {
    Write-Error "CSV file not found at $csvPath. Please verify location."
    Exit
}

$users = Import-Csv -Path $csvPath
$tempPassword = ConvertTo-SecureString "Welcome2026!" -AsPlainText -Force

foreach ($user in $users) {
    $firstname = $user.Firstname.Trim()
    $lastname  = $user.Lastname.Trim()
    $dept      = $user.Department.Trim()
    $title     = $user.Title.Trim()
    
    $username  = ($firstname.Substring(0,1) + $lastname).ToLower()
    $upn       = "$username@adlab.local"
    $name      = "$firstname $lastname"

    $targetOU = "OU=Finance,DC=adlab,DC=local"

    if (Get-ADUser -Filter "SamAccountName -eq '$username'") {
        Write-Warning "User account $username already exists. Skipping."
    } else {
        New-ADUser `
            -Name $name `
            -GivenName $firstname `
            -Surname $lastname `
            -SamAccountName $username `
            -UserPrincipalName $upn `
            -Title $title `
            -Department $dept `
            -Path $targetOU `
            -AccountPassword $tempPassword `
            -Enabled $true `
            -ChangePasswordAtLogon $true

        Write-Host "Successfully provisioned user account: $username ($name)" -ForegroundColor Green
    }
}
---

## Project 3: Automated Account Deprovisioning & Offboarding (Identity Lifecycle Management)

### Overview
Engineered a secure, automated offboarding workflow in PowerShell to process HR separation records (`termed_employees.csv`), revoke unauthorized access, mitigate lingering credential risks, and maintain compliance-ready audit trails within Active Directory.

### Architecture & Security Workflow
- **Data Ingestion:** Ingested structured HR termination rosters containing target usernames, ITSM ticket IDs, and separation reasons.
- **Account State Isolation:** Executed immediate administrative deactivation (`Disable-ADAccount`).
- **Credential Invalidation:** Reset passwords to cryptographically random 24-character strings (`Set-ADAccountPassword`) to eliminate session replay and credential reuse.
- **Permission Revocation:** Enumerated and revoked all security and distribution group memberships (`MemberOf`) to nullify access to shared file repositories, SaaS integrations, and internal resources.
- **Compliance Audit Stamping:** Injected timestamped operational metadata into the `Description` attribute documenting the ITSM reference ticket and departure reason.
- **Quarantine Relocation:** Migrated deprovisioned user objects out of operational OUs into a dedicated quarantine unit (`OU=Disabled_Accounts,DC=adlab,DC=local`).

### Automation Script (`DeprovisionUsers.ps1`)
```powershell
Import-Module ActiveDirectory

$csvPath = "C:\termed_employees.csv"
$quarantineOU = "OU=Disabled_Accounts,DC=adlab,DC=local"
$timestamp = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")

if (-not (Test-Path $csvPath)) {
    Write-Error "CSV roster not found at $csvPath."
    Exit
}

$termedUsers = Import-Csv -Path $csvPath

foreach ($record in $termedUsers) {
    $samAccount = $record.Username.Trim()
    $ticket     = $record.TicketNumber.Trim()
    $reason     = $record.Reason.Trim()

    $adUser = Get-ADUser -Filter "SamAccountName -eq '$samAccount'" -Properties MemberOf, Description

    if ($adUser) {
        Write-Host "Processing deprovisioning for: $samAccount..." -ForegroundColor Cyan

        # 1. Disable the Account
        Disable-ADAccount -Identity $adUser.DistinguishedName

        # 2. Randomize password to kill credential reuse
        $randomSecret = (-join ((65..90) + (97..122) + (48..57) | Get-Random -Count 24 | ForEach-Object {[char]$_}))
        $secureSecret = ConvertTo-SecureString $randomSecret -AsPlainText -Force
        Set-ADAccountPassword -Identity $adUser.DistinguishedName -NewPassword $secureSecret -Reset

        # 3. Strip all security and distribution groups
        $groups = $adUser.MemberOf
        foreach ($group in $groups) {
            Remove-ADGroupMember -Identity $group -Members $adUser.DistinguishedName -Confirm:$false
            Write-Host "  [-] Revoked group: $group" -ForegroundColor Yellow
        }

        # 4. Stamp Audit Trail into Description
        $auditNote = "Offboarded on $timestamp | Ref: $ticket | Reason: $reason"
        Set-ADUser -Identity $adUser.DistinguishedName -Description $auditNote

        # 5. Move object to Disabled_Accounts OU
        Move-ADObject -Identity $adUser.DistinguishedName -TargetPath $quarantineOU

        Write-Host "[SUCCESS] $samAccount fully isolated in Disabled_Accounts." -ForegroundColor Green
    } else {
        Write-Warning "User $samAccount not found in Active Directory. Skipping."
    }
}Get-ADUser -Filter * -SearchBase "OU=Disabled_Accounts,DC=adlab,DC=local" -Properties Enabled, Description | 
    Select-Object Name, SamAccountName, Enabled, Description | Format-Table -AutoSize
Name          SamAccountName Enabled Description
----          -------------- ------- -----------
Sarah Connor  sconnor        False   Offboarded on 2026-10-06 08:05:12 | Ref: TKT-10492 | Reason: Resignation
Michael Scott mscott         False   Offboarded on 2026-10-06 08:05:13 | Ref: TKT-10515 | Reason: Contract Ended
