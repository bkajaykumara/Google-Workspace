# Google Workspace Administrator Interview Study Guide

Welcome to your study guide. This repository is designed to help you bridge the gap between mid-level Google Workspace Administration (3.7 years) and the senior systems engineer role (4–8 years of experience) described in the job qualifications. 

Use this guide to study architectural concepts, review terminal commands, analyze scripting examples, and walk through real-world Incident Response Playbooks (Level 3 Support).

---

## Guide Index

### 📂 [Module 1: Google Workspace Engineering Deep Dive](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/1_google_workspace_deep_dive.md)
*Core tenant architecture, email authentication systems, data security, and data recovery.*
- Hierarchical OU Design and Policy Inheritance Rules.
- Mail Flow Infrastructure: SPF, DKIM, DMARC, MTA-STS, and BIMI.
- Data Loss Prevention (DLP) and Optical Character Recognition (OCR) configurations.
- Google Vault: Retention Rules, Litigation Holds, Audits, and eDiscovery.
- Data Migration Frameworks: Coexistence, Dual/Split Delivery, and Migration Tools.

### 📂 [Module 2: Identity & Access Management (IAM)](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/2_identity_and_access_management.md)
*SSO federation, user provisioning, access policies, and lifecycle management.*
- Under-the-Hood SAML 2.0 and OIDC Workflows.
- IdP Administration: Okta and Microsoft Entra ID (Azure AD).
- System for Cross-domain Identity Management (SCIM) and Attribute Mapping.
- Joiner-Mover-Leaver (JML) Life Cycle Automation.
- Context-Aware Access (CAA) and Entra ID Conditional Access.

### 📂 [Module 3: Endpoint Device Management (MDM)](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/3_endpoint_device_management.md)
*Managing devices, enforcing security baselines, and automating enrollment.*
- Jamf Pro & Apple Business Manager (ABM) for macOS.
- Microsoft Intune & Windows Autopilot for Windows.
- Google Endpoint Management (Basic vs. Advanced, Company-Owned vs. BYOD).
- Device Compliance Policies, Extensions, and Application Packaging.

### 📂 [Module 4: Scripting and Automation](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/4_scripting_and_automation.md)
*Google Apps Manager (GAM), Google Apps Script, and operating system shell scripts.*
- GAM / GAM-ADV-XTD3 syntax and Service Account Configuration.
- Google Apps Script for User Offboarding, Directory Syncing, and Reporting.
- PowerShell and Bash scripts for endpoint configurations and API endpoints.

### 📂 [Module 5: L3 Scenarios & Integrated SaaS Administration](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/5_l3_scenarios_and_troubleshooting.md)
*Scenario-based interview preparation, emergency response SOPs, and SaaS administration.*
- **Incident SOP 1**: Mitigating a massive Phishing Outbreak.
- **Incident SOP 2**: Recovering from an Identity Provider (IdP) SSO Outage.
- **Incident SOP 3**: Resolving Email Migration & Delivery Failure loops.
- **Incident SOP 4**: Compromised/Lost Device Response (Wipe/Lockdown).
- **Incident SOP 5**: Repairing Broken SCIM / Provisioning Sync errors.
- **Incident SOP 6**: Inactive User Audit, Leaver OU Migration & License Reclamation (OneLogin + GAM).
- **SaaS Ecosystem Admin**: WordPress, DocuSign, ClickUp, QuickBase, Jira.

### 🌟 [Master Consolidated Migration Guide: Microsoft 365 to Google Workspace](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/microsoft_365_to_google_workspace_master_migration_guide.md)
*Unified master reference consolidating Modules 6, 7, and 8, Teams Chat, and Official Google Workspace Migration Knowledge Base (`knowledge.workspace.google.com/admin/migrate/`).*
- Account Pre-Provisioning Architecture (SCIM, GCDS, GAM CLI, Admin Console CSV).
- Complete Workload Coverage: Exchange Online, OneDrive for Business, SharePoint Online, Teams Chat.
- Azure App Registrations & Permissions (`Client ID`, `Client Secret`, `Tenant ID`, `Sharepoint host name`).
- Folder mapping, Outlook rules translation matrix, Calendar ACL mappings, Microsoft To Do tasks.
- Single MX Cutover (`1 SMTP.google.com`), SPF/DKIM/DMARC, Delta Syncs, Error Codes, GAM Scripts.
- 15 Model Answers for Senior Systems Engineer Interviews.

### 📂 [Module 6: SharePoint Online to Google Drive Migration](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/6_sharepoint_to_google_drive_migration.md)
*Enterprise-scale SharePoint to Google Workspace migration — architecture, tooling, and 10,000+ item best practices.*
- Migration Architecture: DMS Flow, OAuth Consent, Microsoft Graph API.
- Step-by-Step Procedure: Connection, CSV Mapping, Identity Mapping, Delta Migration.
- Best Practices for 10,000+ Items: Phased Batching, File Compatibility, Permission Audits.
- DMS vs. Third-Party Tools: BitTitan MigrationWiz, ShareGate Comparison.
- Shared Drive Limits & Constraints: 400K item cap, nesting depth, member limits.
- Interview Q&A: 7 Model Answers for Senior-Level SharePoint Migration Questions.

### 📂 [Module 7: Exchange Online Email, Contacts & Calendars Migration](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/7_exchange_email_calendar_contacts_migration.md)
*Migrating email, contacts, and calendar data from M365 Exchange Online to Google Workspace — enterprise best practices and coexistence planning.*
- Migration Architecture: DMS Flow, OAuth Consent, Exchange Web Services.
- Step-by-Step Procedure: Connection, CSV User Mapping, Settings Configuration, Delta Migration.
- Best Practices for 1,000+ Users: Phased Waves, API Throttling Mitigation, Mailbox Size Triage.
- Coexistence Planning: Split/Dual Delivery, Calendar Interop, MX Record Cutover Strategy.
- Tool Comparison: DMS vs. GWME vs. GWMMO vs. BitTitan MigrationWiz.
- Post-Migration DNS: MX Records, SPF, DKIM, DMARC, MTA-STS Configuration.
- Interview Q&A: 7 Model Answers for Senior-Level Exchange Migration Questions.

### 📂 [Module 8: OneDrive to Google Drive Migration](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/8_onedrive_to_google_drive_migration.md)
*Migrating user files and folders from M365 OneDrive to Google Drive — permission handling, file compatibility, and enterprise-scale strategies.*
- Migration Architecture: DMS Flow, Two-CSV Workflow (Source Users + Identity Map).
- Step-by-Step Procedure: Connection, Source CSV, Identity Mapping, Settings (Unmapped Identities, File Exclusions, Size Limits).
- Unmapped Identities Deep Dive: When to enable/disable auto-mapping for sharing permissions.
- Best Practices for 1,000+ Users: Size-Based Waves, API Throttling, File Name Compatibility.
- OneDrive vs. Google Drive Differences: Sync clients, version history, recycle bin, file locking.
- Tool Comparison: DMS vs. BitTitan vs. Movebot.
- Interview Q&A: 7 Model Answers for Senior-Level OneDrive Migration Questions.

### 📂 [Module 9: GAM Advanced Command Reference & Cheat Sheet](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/9_gam_command_reference.md)
*Production-grade reference for GAM / GAM-ADV-XTD3 based on official BNF syntax — selectors, SKUIDs, Drive/Gmail/Group playbooks, and migration pipelines.*
- BNF Syntax Rules: Placeholders, Quoting Rules, Primitives, Multithreading Tuning (`num_threads`).
- Entity Selectors: `all users`, `ou_and_children`, `group_users`, `license <SKUID>`, `queries`.
- Product & SKU Table: `wsentplus`, `wsbizstan`, `cloudidentityfree`, `vault`, `gsuiteenterprisearchived`.
- Production Playbooks: JML Offboarding, Shared Drive Lifecycle, Phishing Email Purge, 2SV Audits.
- Migration GAM Pipelines: 10,000-user pre-provisioning, post-migration ACL reconciliation.
- Interview Cheat Sheet: Top 15 Senior Admin GAM One-Liners.

### 📂 [Module 10: Google Workspace Official Migration Knowledge Base & Technical Guide](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/10_google_workspace_official_migration_knowledge_base.md)
*Consolidated reference guide extracted from official Google Workspace Admin migration documentation — covering Data Import Tool (Default & Advanced for Exchange, Teams, OneDrive, SharePoint, Dropbox), Google Workspace Migrate, GWMME, GWMMO, Azure Entra ID App Registrations, Scan Reports, Error Codes, and Delta Sync workflows.*

---

## Core Competency Checklist

Use this checklist to track your preparation progress. Mark items as completed as you review the corresponding modules.

- [ ] **Email Authentication Architecture**: Explain the difference between SPF soft-fail (`~all`) and hard-fail (`-all`), SPF lookup limits, and DMARC alignment rules.
- [ ] **SAML Assertion Flow**: Describe the precise token exchange sequence when a user attempts to log into Google Workspace via Okta SSO.
- [ ] **SCIM Integration**: Understand how attribute mapping errors occur and how to debug user lifecycle automation sync drops.
- [ ] **macOS Device Onboarding**: Explain Automated Device Enrollment (ADE) via Apple Business Manager and Jamf Pro Configuration Profiles.
- [ ] **Windows Device Onboarding**: Explain Windows Autopilot deployment profile configurations and Intune Win32 App packaging.
- [ ] **Context-Aware Access**: Define security postures based on device status, IP CIDR, geofencing, and OS version.
- [ ] **GAM Scripting**: Construct complex GAM commands for bulk audits, group additions, and calendar transfers.
- [ ] **Apps Script Automation**: Build triggers to automate tasks using Google Workspace REST APIs.
- [ ] **Litigation Holds & Vault**: Differentiate between retention policies and litigation holds, and run eDiscovery exports.
- [ ] **L3 Emergency Playbooks**: Explain break-glass account guidelines and remediation steps for security incidents.
- [ ] **SharePoint Migration**: Walk through a 10,000+ item SharePoint-to-Google-Drive migration including CSV mapping, phased batching, Delta migration, and post-migration permission audits.
- [ ] **Exchange Email Migration**: Explain the end-to-end Exchange-to-Gmail migration process including API throttling, coexistence (Split/Dual Delivery), MX record cutover, and Delta migration strategy.
- [ ] **OneDrive Migration**: Explain the OneDrive-to-Google-Drive migration process including two-CSV workflow, unmapped identities, file format exclusions, and permission preservation strategies.
- [ ] **GAM Command Mastery**: Construct BNF-compliant GAM commands for bulk JML provisioning, domain-wide phishing purges, delegated access audits, Shared Drive restrictions, and post-migration ACL verification.
- [ ] **License Reclamation & Leaver Lifecycle**: Describe the pipeline for reconciling OneLogin IdP telemetry with Google Workspace `lastLoginTime`, filtering service/technical accounts, moving users to `_Leavers` OU, and reclaiming Enterprise Plus (`1010020020`) and Cloud Search (`1010350001`) licenses via GAM.
