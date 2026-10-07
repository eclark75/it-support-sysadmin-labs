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
