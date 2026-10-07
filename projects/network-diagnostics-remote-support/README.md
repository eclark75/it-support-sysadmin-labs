# Enterprise IT Support: Network Diagnostics & Remote Support Runbook

## Overview
This operational guide and technical runbook establishes a systematic, bottom-up troubleshooting methodology for resolving Tier-1 and Tier-2 client connectivity, DNS resolution, and remote management issues. 

Designed for enterprise support environments, this runbook details standard CLI diagnostic protocols, incident escalation workflows, and remote assistance procedures aligned with ITIL and Google IT Support Specialist standards.

---

## Technical Stack & Tooling
* **Operating Systems:** Windows 11 / Windows 10, Windows Server 2022
* **Command Line Interfaces:** Windows PowerShell, Command Prompt (`cmd`)
* **Core Protocols:** TCP/IP, ICMP, DNS (UDP/TCP 53), DHCP (UDP 67/68), RDP (TCP 3389), HTTPS (TCP 443)
* **Remote Assistance Utilities:** Microsoft Quick Assist, Windows Remote Desktop Protocol (RDP), PowerShell Remoting (WinRM)

---

## Bottom-Up Troubleshooting Methodology (OSI Model Aligned)
# 1. Test local TCP/IP protocol stack integrity (Loopback test)
ping 127.0.0.1 -n 4

# 2. Test communication with the local Default Gateway (Router)
ping <Default-Gateway-IP> -n 4
# 1. Test direct Internet IP reachability (bypassing DNS)
ping 8.8.8.8 -n 4

# 2. Test Fully Qualified Domain Name (FQDN) reachability
ping google.com -n 4

# 3. Explicit DNS lookup and record validation
nslookup google.com
# Trace route to destination host
tracert -d google.com
# Test HTTPS connectivity (Port 443)
Test-NetConnection -ComputerName google.com -Port 443

# Test Remote Desktop Protocol reachability (Port 3389)
Test-NetConnection -ComputerName <Target-Host-IP> -Port 3389
---

## 2. Remote Support & Assistance Procedures

### Protocol 1: Microsoft Quick Assist (Cloud-Brokered Assistance)
* **Use Case:** Remote users off-VPN or requiring interactive desktop support over public networks.
* **Standard Operating Workflow:**
  1. IT Specialist launches Quick Assist (`Ctrl + Win + Q`) and selects **Help someone**.
  2. IT Specialist provides the time-sensitive 6-digit security code to the user via phone or corporate chat.
  3. User enters the security code and authorizes screen sharing.
  4. IT Specialist requests **Take control** to perform administrative tasks, driver reinstalls, or profile repairs.

### Protocol 2: Remote Desktop Protocol (RDP / mstsc)
* **Use Case:** Domain-joined workstations or servers on the internal network or connected via corporate VPN.
* **Execution:**
  1. Open Run dialog (`Win + R`) and launch `mstsc`.
  2. Input target FQDN or IP address.
  3. Authenticate with delegated administrator credentials (`adlab\admin-support`).
  4. Perform administrative maintenance without disrupting physical display hardware.

---

## 3. Simulated Incident Scenario & Resolution Write-up

### Ticket Information
* **Ticket ID:** `INC-1082`
* **Reporter:** Marketing Workstation (`DESKTOP-MKT-04`)
* **Priority:** Medium / User Impacted
* **Summary:** User cannot access cloud CRM or internet resources; reporting "No Internet Access".

### Investigation & Root Cause Analysis
1. **Initial Assessment:** Ran `ipconfig /all`. Found IPv4 address assigned was `169.254.82.11` (APIPA), indicating the client failed to receive an IP lease offer from the corporate DHCP server.
2. **Layer 1 & 2 Verification:** Confirmed physical Ethernet link lights were active. Cycled network interface adapter.
3. **Remediation:** Executed `ipconfig /release` followed by `ipconfig /renew`. The workstation successfully obtained lease `192.168.10.84` from `DC01-Lab`.
4. **Resolution Verification:** 
   * Flushed local resolver cache (`ipconfig /flushdns`).
   * Executed `Test-NetConnection -ComputerName google.com -Port 443` $\rightarrow$ Returned `TcpTestSucceeded: True`.
   * User confirmed full CRM access restored.
5. **Closure:** Ticket updated with command execution logs and closed within SLA limits.
