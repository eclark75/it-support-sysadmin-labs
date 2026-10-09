# Enterprise IAM: Microsoft Entra ID RBAC & Scoped Delegation

## Project Overview
Configured and verified enterprise Role-Based Access Control (RBAC) and scoped identity administration within a hybrid-ready Microsoft Entra ID tenant. Designed to adhere strictly to the Principle of Least Privilege (PoLP) by restricting Tier 1 / Tier 2 support capabilities to designated scopes.

## Architecture & Implementation
* **Identity Roles Assigned:**
  * `Helpdesk Administrator`: Standard identity credential and self-service administration.
  * `User Administrator`: Complete lifecycle management (provisioning, profile updates, soft-deletion).
* **Scope Scaffolding:**
  * **Administrative Unit:** `Chicago Office`
  * **Delegation Model:** Scoped local container permissions paired with tenant-wide support oversight.
* **Security Boundaries:**
  * Enforced Entra ID administrative boundaries preventing delegated helpdesk roles from modifying privileged service identities or accessing high-risk tenant telemetry.

## Operational Verification Workflow
1. **Delegation Provisioning:** Assigned required RBAC roles to designated support personnel (`mvance@RemnantsofFire.onmicrosoft.com`).
2. **Access Token Validation:** Conducted independent session isolation tests in Microsoft Edge and Chrome to verify tenant-side token refresh and eliminate unauthorized API blocks.
3. **Identity Lifecycle Testing:**
   * Provisioned standard identity (`Test User`).
   * Executed non-privileged password reset and recorded temporary credential generation.
   * Executed object deletion and validated soft-delete retention in the 30-day directory recycle bin.
4. **Administrative Unit Scoping:** Validated inheritance rules and local scoping within the `Chicago Office` AU (`Directory (Inherited)` vs. `This resource`).

## Skills Demonstrated
* Microsoft Entra ID (Azure AD) Administration
* Cloud Identity Governance & RBAC
* Administrative Units (AU) Scoping
* Identity Lifecycle Management & Helpdesk Operations
## Microsoft 365 Tenant Administration & Messaging Infrastructure

### Operational Architecture
* **Centralized Tier 1 Shared Mailbox:** Deployed `Helpdesk Support` (`helpdesk@RemnantsofFire.onmicrosoft.com`) to serve as the unified intake queue for tenant IT requests.
* **Access Delegation & Permissions:**
  * Configured `Read and Manage` (Full Access) delegation for IT operations staff.
  * Granted `Send As` authority to enable outbound ticketing correspondence under the shared support alias.
* **Functional Mail Routing Verification:**
  * Successfully validated outbound message dispatch from the shared identity to verify Exchange Online mail flow and permission inheritance.
  * Verified audit logging and message tracking across delegated sessions.
- **Self-Service Password Reset (SSPR) & Authentication Methods Deployment:** Enabled tenant-wide SSPR policy and deployed modern authentication methods (Email OTP and Microsoft Authenticator) in Microsoft Entra ID. Verified end-to-end self-service recovery workflows in isolated sessions to automate user credential resets and deflect Tier 1 help desk ticket volume.
