# Enterprise IT: Zero-Trust Identity Deprovisioning & Access Revocation

## Project Overview
This project demonstrates an enterprise-grade identity lifecycle management workflow, integrating virtualization, Active Directory administration, IT Service Management (ITSM), and standard operating procedures (SOPs). 

It models a high-priority, zero-trust employee offboarding request executed across a Windows Server 2022 domain controller, Jira Service Management Cloud, and Confluence.

---

## Architecture & Technology Stack
* **Virtualization & Systems:** Oracle VM VirtualBox, Windows Server 2022 (`DC01-Lab`, domain: `adlab.local`)
* **Identity & Access Management (IAM):** Active Directory Domain Services (AD DS), PowerShell CLI
* **IT Service Management (ITSM):** Atlassian Jira Service Management (Ticket Lifecycle, SLA tracking, Audit logging)
* **Knowledge Management:** Atlassian Confluence (Standard Operating Procedures, Knowledge Base integration)

---

## Operational Workflow
Get-ADUser "<TargetUser>" -Properties Enabled, MemberOf, DistinguishedName | Select-Object Name, Enabled, DistinguishedName, MemberOf
