**CyberArk PAM Automation: REST API Password Retrieval**

&nbsp;

**Project Overview**
This project demonstrates a secure, programmatic workflow for retrieving privileged credentials from a CyberArk Vault without human intervention. The goal was to simulate a "Robot User" (e.g., a vulnerability scanner or orchestration tool) that needs to authenticate, locate a specific target account, and retrieve its password securely via the CyberArk REST API.

&nbsp;

**Objective:** Eliminate hard-coded credentials in scripts by implementing a dynamic API call to the Vault.


**Phase 1: Identity & Access Configuration**
The first step was establishing a digital identity for the automation script. I created a dedicated CyberArk internal user, svc_automation, to act as the service account.


&nbsp;


<img width="555" height="437" alt="Screenshot 2026-01-05 165001" src="https://github.com/user-attachments/assets/8b694520-93bf-4fda-ad23-bc5d1e6655e4" />

Figure 1: Verified creation of the 'svc_automation' EPVUser.

The Target
The mission was to retrieve the credentials for a domain service account (svc_app_01) stored in the Tier1_Operations Safe.


&nbsp;


<img width="895" height="534" alt="Screenshot 2026-01-05 165629" src="https://github.com/user-attachments/assets/782092a2-03b0-463f-84cb-db801bfb64ee" />

Figure 2: The target account securely stored in the Vault.


&nbsp;


**Phase 2: The "Blind Robot" Challenge (Permission Engineering)**
During initial testing, the script could authenticate but failed to find any accounts, returning a "Safe is Empty" error. This is a common issue known as the "Blind Robot" scenario—the user has permission to access the safe but not to see its contents.

Troubleshooting Steps
List Permissions: I realized the List accounts permission is distinct from Retrieve accounts. Without List, the API search function returns zero results.

Confirmation Workflows: I also identified that "Dual Control" (requiring human approval) would break the automation.

The Fix: I configured the Safe permissions to grant the robot full visibility and bypass manual approvals.

<img width="785" height="431" alt="Screenshot 2026-01-05 180537" src="https://github.com/user-attachments/assets/d41d5a31-16c0-4df7-8fc4-66ba0b007c58" />

Figure 3: Enabled "Access Safe without confirmation" to bypass manual approval workflows.


&nbsp;


<img width="806" height="455" alt="Screenshot 2026-01-05 180607" src="https://github.com/user-attachments/assets/0d134bde-fb04-4e52-afc6-d1a663f49f0a" />

Figure 4: Verified "List", "Use", and "Retrieve" permissions were active.


&nbsp;


**Phase 3: The Automation Script (PowerShell)**
I developed a custom PowerShell script to interact with the CyberArk REST API. The script handles:

Authentication: Exchanges a password for a secure Session Token.

Error Handling: Manages SSL/TLS trust issues common in lab environments.

Token Parsing: Dynamically handles both raw string and JSON responses from the PVWA.

Retrieval: Enumerates the Safe and extracts the password.

Authentication Flow
The script uses Get-Credential to securely capture the robot's password, ensuring it is never hard-coded in the script file itself.


<img width="681" height="493" alt="Screenshot 2026-01-05 173450" src="https://github.com/user-attachments/assets/433eefab-5334-48e2-b3b7-b70dd5f5f941" />

Figure 5: Secure credential object handling.


&nbsp;


**Phase 4: Execution & Results**
The final execution validated the entire chain. The script successfully logged in, listed the contents of Tier1_Operations, and retrieved the password for the first available privileged account (adm_recon) as a proof-of-concept.

<img width="830" height="647" alt="Screenshot 2026-01-06 141748" src="https://github.com/user-attachments/assets/87c94c27-af25-4bb6-a8f4-5c6dcc7e3e1c" />

Figure 6: Successful execution. The script authenticated, obtained a token, identified the account ID (30_3), and retrieved the live password.


&nbsp;


Technical Key Takeaways
Least Privilege: The robot user was restricted only to the specific Safe (Tier1_Operations) it needed.

API Logic: Learned to handle raw API responses and debug HTTP status codes.

Infrastructure Dependency: Troubleshooting required diagnosing Vault service availability and IIS states during the process.
