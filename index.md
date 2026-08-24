# Google Workspace Administrator Interview Study Guide

Welcome to your study guide. This repository is designed to help you bridge the gap between mid-level Google Workspace Administration (3.7 years) and the senior systems engineer role (4–8 years of experience) described in the job qualifications. 

Use this guide to study architectural concepts, review terminal commands, analyze scripting examples, and walk through real-world Incident Response Playbooks (Level 3 Support).

---

## Guide Index

### 🏆 [SINGLE COMPLETE MASTER HANDBOOK (ALL MODULES CONSOLIDATED)](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/google_workspace_complete_master_handbook.md)
*The complete, unabridged, single master reference document consolidating ALL 10 project modules, Level 3 Incident SOPs, IAM Architecture, MDM, Scripting, GAM BNF Syntax, Apps Scripting, PowerShell/Bash scripts, and the complete Microsoft 365 Migration Framework into one authoritative guide.*

---

### 🌟 [Master Consolidated Migration Guide: Microsoft 365 to Google Workspace](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/microsoft_365_to_google_workspace_master_migration_guide.md)
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

### 🎓 [Module 11: Professional Google Workspace Administrator Certification & Top 55 Q&A Handbook](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/11_certification_exam_and_interview_qa.md)
*Comprehensive handbook for the Professional Google Workspace Administrator Certification exam & technical interviews — featuring exam format ($125 USD, 2 hrs, Kryterion), Admin Console control tower architecture, Top 55 Q&As (Basic, Intermediate, Advanced Scenario, Behavioral), indispensable admin tools matrix, and the official 7-step preparation strategy.*

### 📂 [Module 12: G Suite & Google Workspace Core Operational Interview Handbook](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/12_gsuite_and_workspace_core_interview_handbook.md)
*Operational interview guide covering core G Suite/Workspace architecture, Google Workspace vs. Microsoft 365 comparison, email migration workflows, user provisioning, Apps Script automation, Google Sheets advantages & conditional formatting, email aliases, Google Groups collaborative inboxes, third-party app governance, and service troubleshooting.*

### 💻 [Module 13: Google Workspace Developer APIs, Python SDKs & Custom Add-ons Handbook](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/13_google_workspace_developer_apis_and_sdk_handbook.md)
*Developer reference guide for Google Workspace APIs — featuring complete Python SDK scripts for Drive file upload & Calendar event scheduling (with Google Meet link generation), OAuth 2.0 web application authentication architecture flow, custom Workspace add-on lifecycle (`appsscript.json`), Reports API audit querying, and SAML 2.0 SSO setup.*

### 📋 [Module 14: G Suite & Google Workspace Senior Analyst Hiring, Evaluation & Interview Rubric](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/14_gsuite_analyst_hiring_and_evaluation_handbook.md)
*Hiring manager and assessor framework — featuring 10 scenario interview questions with strong answer criteria & red flags, 4-category weighted evaluation rubric (Technical 35%, Security 30%, Automation 20%, Communication 15%), senior vs. mid-level differentiation matrix, core enterprise technology benchmarks, and hiring FAQs.*

### 🏆 [Module 15: Google Workspace Master 95 Platform Owner & Lead Administrator Interview Handbook](file:///c:/Users/ajayk/OneDrive/Desktop/Google%20Workspace%20adminstration/15_google_workspace_master_95_interview_questions_handbook.md)
*Masterclass containing 95 scenario-driven platform owner questions and model answers across 13 operational domains — Identity & Access, User Lifecycle, Mail Flow, Drive Governance, Calendar/Meet/Groups, Licensing/Storage, Device MDM, Reporting, Incidents, Automation, Governance, Leadership, and Behavioural.*

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
