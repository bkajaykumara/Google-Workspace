# Module 15: Google Workspace Master 95 Platform Owner & Lead Administrator Interview Handbook

> **Target Audience**: Lead Systems Engineers, Principal Google Workspace Architects, IT Platform Owners, and Technical Directors.  
> **Overview**: Definitive interview masterclass containing 95 scenario-driven platform owner questions and model answers grouped into 13 core operational domains: Identity & Access, User Lifecycle, Mail Flow & DNS, Drive Governance, Calendar/Meet/Groups, Licensing & Storage, Device MDM, Reporting, Incident Response, Automation, Governance, Leadership, and Behavioural.

---

## 1. Identity & Access Architecture (11 Questions)

### Q1: When do you use an Organisational Unit (OU) vs. a Group?
* **Answer**: OUs carry policy, settings, and service enablement (e.g., restricting Marketplace installs). Users belong to exactly **one** OU, inheriting settings down the OU tree. Groups carry access and permissions (e.g., Shared Drive access, distribution lists). Users belong to **multiple** groups. Using OUs for access forces an unsustainable tree structure.
* **Testing**: Separation of administrative policy from access control axes.

### Q2: How do you roll out mandatory 2SV across 1,500 users without locking anyone out?
* **Answer**: Execute in stages: Enable enrolment without enforcement first and monitor completion in Reports. Communicate a clear enforcement deadline. Enforce first on a pilot OU, then broaden rollout. Utilize the new user enrolment grace period. Provide a verified identity check process for issuing backup codes. Enforce security keys harder and earlier for Super Admins. Exclude a break-glass account before enabling rules.
* **Testing**: Real-world change management vs. setting toggling.

### Q3: What is a break-glass account and how do you protect it?
* **Answer**: An emergency Super Admin account not tied to a human, designed to prevent tenant lockout during IdP/MFA outages. Exclude it from Context-Aware Access (CAA) rules. Split long, random credentials between two custodians or store in a secure safe. Store hardware FIDO2 keys physically. Configure real-time alerts for any sign-in event, and test authentication on a strict schedule.
* **Testing**: Failure-mode thinking and disaster recovery preparedness.

### Q4: How many Super Admins should a 1,500-person tenant have, and how do you keep it honest?
* **Answer**: Maintain as few as required for on-call rotas (typically 3–5). Delegate all other permissions using pre-defined or Custom Admin Roles scoped to specific OUs or privileges (e.g., Help Desk Admin). Require separate day-to-day and admin accounts. Perform quarterly admin access reviews with named approvers to prevent privilege creep.
* **Testing**: Least privilege applied to administrative functions.

### Q5: What does Context-Aware Access (CAA) give you that 2SV does not?
* **Answer**: 2SV verifies **who** is signing in; CAA evaluates **context**—whether sign-in is allowed from a compliant device, authorized IP CIDR, geofence, or supported OS state. Deploy in **Monitor Mode** first, test on a pilot OU, and exclude the break-glass account before hard enforcement. Requires Enterprise Standard/Plus or Cloud Identity Premium.
* **Testing**: Separation of authentication from zero-trust authorization.

### Q6: A third-party tool requests Drive access for your users. How do you control it?
* **Answer**: Configure **App Access Control** (`Security > API Controls > Manage Third-Party App Access`). Block unconfigured third-party apps domain-wide, then allowlist required applications by OAuth Client ID per OU. Review requested scopes (`drive.readonly` vs. `drive`). Audit existing OAuth tokens before enforcement to assess impact.
* **Testing**: Preventing OAuth shadow IT backdoors.

### Q7: Is Workspace your Identity Provider (IdP) or Service Provider (SP)?
* **Answer**: It can be either or both. Workspace as **IdP** authenticates users into third-party SaaS via SAML/OIDC. Workspace as **SP** delegates authentication to an external IdP (Okta/Entra ID). Architecture determines where lifecycle automation lives, where MFA is enforced, and what breaks during outages.
* **Testing**: Enterprise identity architecture vs. console administration.

### Q8: How do you add a SAML Application, and what usually goes wrong?
* **Answer**: Exchange metadata XML, map `NameID` to primary email, map role attributes, and scope availability by OU/Group. Common failure point: `NameID` mismatch—if the SP keys on primary email and a user's email changes, authentication breaks. Also, enabling SAML does not auto-provision target app accounts unless SCIM is configured.
* **Testing**: Practical SSO integration experience.

### Q9: With 2SV enforced, does password policy still matter?
* **Answer**: Yes. Legacy protocols, app passwords, service accounts, and recovery flows can bypass MFA if not disabled. Enforce password complexity/length, block compromised passwords, and explicitly disable **Less Secure Apps** to prevent basic authentication bypasses.
* **Testing**: Understanding MFA bypass vectors.

### Q10: How do you control session length, and what is the trade-off?
* **Answer**: Configure web session duration under **Security > Access and Data Control**. Set short sessions (e.g., 4–8 hours) for Admin OUs and standard durations for users. Trade-off: Excessively short sessions cause user habituation, training users to click through prompts without reading—exposing them to phishing. Pair short sessions with trusted device policies.
* **Testing**: Balancing security posture against human behavior risks.

### Q11: Your external IdP goes down and nobody can sign in. What do you do?
* **Answer**: Confirm outage scope. Communicate via out-of-band channels (SMS/Status Page). If pre-agreed with SecOps, temporarily bypass SSO for critical OUs using Google direct login (`admin.google.com/?loginhint=...`). Use the break-glass Super Admin account to modify SSO settings if necessary.
* **Testing**: High-stress incident composure and pre-planned fallback execution.

---

## 2. Lifecycle Engineering (8 Questions)

### Q12: Design the Joiner Flow for a new starter.
* **Answer**: Trigger from HRIS system of record (Workday/BambooHR). Provision account via SCIM/GCDS to standard naming conventions, place in department OU, add to role-based Google Groups (auto-triggering app entitlements and licenses), apply signature template, provision Drive folders, and share induction material. Target: Day-one productivity without manual IT tickets.
* **Testing**: Automated provisioning from system of record vs. manual request processing.

### Q13: Design the Leaver Flow: Order of execution?
* **Answer**: 
  1. **Suspend Account** (halts new sign-ins immediately).
  2. **Revoke Sessions & OAuth Tokens** (`gam user signout` & delete tokens; suspension alone does not end active sessions).
  3. **Transfer Drive & Calendar Ownership** to manager.
  4. **Configure Mail Forwarding / Delegation**.
  5. **Remove from Google Groups**.
  6. **Reclaim License** (only after data transfer completes).
  7. **Retain Account in Suspended State** per Vault compliance rules before deletion.
* **Testing**: Precision sequencing and recognizing that account suspension does not invalidate OAuth tokens.

### Q14: Everyone automates Joiners and Leavers. What about Movers?
* **Answer**: Movers accumulate access creep. Handle role changes via HR feed triggers: Recompute group memberships based on the new job code rather than appending groups, move the account to the new OU, and strip obsolete role-derived entitlements.
* **Testing**: Solving access accumulation gaps.

### Q15: Someone leaves. What actually happens to the files they owned?
* **Answer**: Files in **My Drive** are owned by the user account; deleting the account deletes the files unless ownership is transferred to another user. Suspending an account does not block external shared access to existing files. Files in **Shared Drives** are owned by the organization; removing a user leaves Shared Drive files completely intact.
* **Testing**: Workspace file ownership architecture vs. button clicking.

### Q16: How do you handle contractors and non-employees?
* **Answer**: Place in a dedicated `/Contractors` OU with restrictive policies (disabled Takeout, restricted external sharing, excluded from company-wide groups). Require an explicit expiration date at creation. Review contractor accounts monthly.
* **Testing**: Governance of non-HR directory populations.

### Q17: The business asks for a shared mailbox. What do you give them?
* **Answer**: Provide a **Google Group with Collaborative Inbox** (no license cost, auditable membership, no credential sharing). Issue a licensed user account only if dedicated Drive, Calendar, or third-party sign-ins are strictly required—enrolled in 2SV with credentials stored in an enterprise password manager.
* **Testing**: Pushing back against anti-pattern shared accounts.

### Q18: Your company acquires another tenant. How do you approach the consolidation?
* **Answer**: Define end-state architecture (tenant merge vs. federated coexistence). Inventory target users, groups, Shared Drives, OAuth apps, and mail routing. Sequence: Move identity (SCIM/SSO) first, migrate data second, cut over domain DNS MX records, and maintain legacy domain as alias with split delivery routing.
* **Testing**: Enterprise M&A project architecture skills.

---

## 3. Mail Flow, Security & Authentication (9 Questions)

### Q19: Explain SPF, DKIM, and DMARC alignment.
* **Answer**: **SPF** authorizes sending IP addresses against the envelope sender (`Return-Path`). **DKIM** cryptographically signs the message header to verify integrity. **DMARC** requires alignment between the visible `From:` header domain and the SPF/DKIM domains, specifying receiver policies (`p=none/quarantine/reject`) and aggregate reporting (`rua`).
* **Testing**: Understanding header alignment vs. reciting acronyms.

### Q20: How do you move DMARC from p=none to p=reject without losing legitimate mail?
* **Answer**: Start at `p=none` and analyze RUA aggregate reports to identify all legitimate sending services (marketing, ticketing, ERP). Fix DKIM alignment for each sender (signing on subdomains if necessary). Advance to `p=quarantine; pct=10`, gradually stepping up to `pct=100`, then enforce `p=reject`.
* **Testing**: Practical email deliverability governance.

### Q21: A partner says your email goes to spam. How do you diagnose it?
* **Answer**: Obtain raw email headers from the recipient. Check `Authentication-Results` for SPF, DKIM, and DMARC alignment failures. Inspect sending IP/domain against RBL blocklists. Check if inbound relay servers modified message headers. Analyze DMARC aggregate reports for that domain.
* **Testing**: Diagnostic methodology under technical ambiguity.

### Q22: What is Content Compliance used for, and where does it bite you?
* **Answer**: Content Compliance scans envelope headers, subject lines, or body text to quarantine, modify, redirect, or reject mail (e.g., blocking credit cards). **Where it bites**: Rules execute independently with **no priority ordering**—you cannot write a narrow override rule below a broad rule. Solution: Use narrow regex match conditions rather than stacking rules.
* **Testing**: Advanced knowledge of Gmail rule engine limitations.

### Q23: Split Delivery vs. Dual Delivery: When do you use each?
* **Answer**: **Split Delivery** delivers email to either Workspace or legacy server based on user mailbox location (user exists in one system). **Dual Delivery** delivers a copy to both systems simultaneously (transition state during migrations). Dual delivery should be timeboxed due to double storage and split reply-thread risks.
* **Testing**: Migration mail routing strategy.

### Q24: How do you run a mail quarantine that people actually review?
* **Answer**: Tune rules to minimize false positives. Assign explicit owners and SLAs to quarantine queues. Configure automated digest notifications to users where policy permits. Monitor queue depth metrics weekly.
* **Testing**: Operational queue governance.

### Q25: What does an external sender banner buy you, and what are its limits?
* **Answer**: Provides a visual warning on external inbound mail to highlight social engineering and impersonation attempts. **Limits**: User habituation (banner fatigue renders it invisible over time) and zero protection against compromised internal accounts.
* **Testing**: Realistic assessment of low-friction security controls.

### Q26: Malicious zero-day attachment not in signature databases: What helps?
* **Answer**: **Security Sandbox** (executes attachments in a dynamic virtual sandbox to detect malicious behavior; Enterprise Plus). Enable advanced Gmail protections for encrypted attachments, unverified scripts, and anomalous file extensions. Enable link protection to evaluate URLs at click-time.
* **Testing**: Dynamic threat prevention vs. static signature scanning.

---

## 4. Drive & Data Governance (7 Questions)

### Q27: Shared Drive vs. My Drive: Why does it matter for governance?
* **Answer**: **My Drive** files are owned by individual accounts; files are at risk of deletion during user offboarding. **Shared Drives** are owned by the organization; file ACLs and assets persist independently of user employment lifecycle.
* **Testing**: Connecting storage architecture to lifecycle data risk.

### Q28: How do you find and fix external file exposure across a domain?
* **Answer**: Run automated audit queries using GAM (`gam all users print filelist query "visibility='anyoneWithLink'"`) or third-party tools (GAT+). Schedule recurring relative-date reports to review and bulk-remediate external sharing ACLs automatically.
* **Testing**: Scalable data loss remediation.

### Q29: What does Workspace DLP actually let you do?
* **Answer**: Scans Drive content and Gmail traffic for predefined detectors (SSN, PCI) or custom regex patterns. Actions: Block sharing, warn user, quarantine message, or log event. Deploy in **Audit-Only Mode** first to measure false-positive rates before hard blocking.
* **Testing**: Phased DLP enforcement strategy.

### Q30: Target Audiences vs. Default Sharing Options.
* **Answer**: Target Audiences create scoped, team-based default sharing choices (e.g., "Share with Finance Department") rather than forcing users to choose between "Private" and "Everyone in Organization".
* **Testing**: Designing secure default behaviors.

### Q31: How do you prevent bulk data export via Google Takeout?
* **Answer**: Disable Google Takeout service across all user OUs in Admin Console. Audit leaver accounts for Takeout download events prior to departure.
* **Testing**: Blocking exfiltration vectors.

### Q32: Data Regions: What can you tell Legal?
* **Answer**: Data Regions enforce geographic storage policies at rest (e.g., EU/US) for covered core service data (Gmail, Drive, Docs). **Limits**: Does not cover metadata, audit logs, or data in transit. Requires Enterprise Plus.
* **Testing**: Precision regarding regulatory compliance scope.

---

## 5. Device & Mobile Management (4 Questions)

### Q33: Fundamental MDM vs. Advanced MDM vs. Enterprise MDM.
* **Answer**: **Fundamental MDM** provides agentless inventory, basic passcodes, and remote account wipe. **Advanced MDM** adds device policies and Android Enterprise Work Profiles. **Full Enterprise MDM** (Jamf/Intune/Kandji) manages macOS/Windows OS builds, patching, and configuration profiles. Workspace consumes device compliance states via Context-Aware Access.
* **Testing**: Tool boundary recognition.

### Q34: How does device state affect document access?
* **Answer**: Via **Context-Aware Access (CAA)**. Workspace evaluates MDM compliance signals (disk encryption, screen lock, OS version) as mandatory conditions for accessing Gmail/Drive.
* **Testing**: Zero-trust identity and endpoint integration.

### Q35: What is Chrome Browser Management?
* **Answer**: Manages browser profiles, extension allowlisting, and security policies across corporate, personal, and contractor endpoints without requiring full machine MDM enrollment.
* **Testing**: Securing the browser as a distinct attack surface.

### Q36: Stolen laptop workflow: First 60 minutes.
* **Answer**: Sign user out of all sessions and revoke OAuth tokens. Issue remote wipe via MDM. Confirm full disk encryption (BitVault/FileVault) is active. Audit locally cached files. Mark asset stolen in inventory, and report to compliance if regulated data was present.
* **Testing**: Comprehensive incident response handling.

---

## 6. Reporting & Analytics (7 Questions)

### Q37: Recurring reports: What detail do people get wrong?
* **Answer**: Using a **fixed date range** instead of a **relative date window** (e.g., "Last 7 Days"). Fixed ranges generate identical stale data on every scheduled run.
* **Testing**: Hands-on reporting operational experience.

### Q38: Admin Console reports are insufficient at scale. What next?
* **Answer**: Stream audit logs to **Google Cloud BigQuery** for long-term retention beyond 6 months, custom SQL joins across log types, and Looker Studio dashboarding. Use **Security Investigation Tool (SIT)** for real-time query and action capabilities.
* **Testing**: Enterprise log analytics architecture.

### Q39: Core Audit Logs breakdown.
* **Answer**: **Admin** (config changes), **Login** (auth/suspicious sign-ins), **Drive** (access/sharing/deletions), **Gmail** (message events), **OAuth Token** (third-party app grants), **Groups** (membership changes), **Devices** (endpoint compliance).
* **Testing**: Log investigation instinct.

### Q40: Tuning Alert Center to prevent alert fatigue.
* **Answer**: Route high-fidelity alerts (compromised accounts, DLP blocks) to active SecOps channels; route low-fidelity alerts to a review queue. Prune rules that produce zero actionable outcomes.
* **Testing**: Operational alert hygiene.

### Q41: Monthly executive reporting metrics.
* **Answer**: Trend lines for 2SV and MDM compliance %, reclaimed license cost savings ($), unresolved access review exceptions with age, and security incident MTTR. Avoid raw ticket counts without context.
* **Testing**: Executive communication skills.

### Q42: Proving 2SV enforcement completeness.
* **Answer**: Reconcile actual user enrolment data against the complete directory user list and explicit exclusion lists—do not rely solely on policy toggle configurations.
* **Testing**: Effective audit evidence validation.

### Q43: Reporting on license utilization.
* **Answer**: Join license assignment with last login timestamp **and** active workload usage (Gmail/Drive activity). Group by department and budget holder.
* **Testing**: Data-driven SaaS cost optimization.

---

## 7. Incident Response (9 Questions)

### Q44: Compromised account walkthrough.
* **Answer**: 
  1. **Contain**: Suspend account, terminate sessions, revoke OAuth tokens and app passwords (`gam user signout`).
  2. **Investigate**: Audit Login logs, SIT query, inspect Gmail forwarding rules/filters, review Drive sharing changes.
  3. **Remediate**: Reset password, re-enroll 2SV, restore account access.
  4. **Prevent**: Update DMARC/spam rules, review OAuth app grants, export audit trail.
* **Testing**: Structured incident containment and remediation steps.

### Q45: Mass phishing campaign response.
* **Answer**: Execute domain-wide message search and hard purge in SIT or via GAM (`gam all users delete messages query ...`). Query login logs to identify users who clicked or submitted credentials; contain those accounts immediately. Block sender IP/domain, and issue transparent user comms.
* **Testing**: Campaign-level incident response scaling.

### Q46: Senior leaver data exfiltration investigation.
* **Answer**: Coordinate with Legal and HR before taking action. Place a **Google Vault Hold** on mail and Drive data. Audit Drive download logs, external file shares, personal account transfers, and forwarding rules. Present factual timestamps and log events without drawing legal conclusions.
* **Testing**: Legal chain of custody and forensic boundaries.

### Q47: Accidental public folder exposure (3,000 files).
* **Answer**: Remove public sharing ACL at root folder level immediately. Verify inherited permissions updated on child items. Audit Drive log to identify external IP access during exposure window. Inspect root cause (default sharing settings/missing DLP) to prevent recurrence.
* **Testing**: Immediate containment and root-cause resolution.

### Q48: Bulk Shared Drive file deletion in progress.
* **Answer**: Contain immediately by suspending the offending account to halt active deletions. Audit logs to determine if cause is malicious, sync client error, or human mistake. Restore items from Shared Drive Trash / Admin Restore window. Audit manager ACL permissions.
* **Testing**: Prioritizing containment over diagnostic analysis.

### Q49: User intentionally shared confidential file externally for business need.
* **Answer**: Contain external exposure immediately. Inform HR of protocol breach. Investigate why a secure, sanctioned path was not available for the business requirement to address systemic workflow friction.
* **Testing**: Balancing security enforcement with systemic business process analysis.

### Q50: How do you define an Incident vs. a Problem?
* **Answer**: An **Incident** is an active disruption affecting confidentiality, integrity, or availability, or a failed control requiring an immediate structured response. A **Problem** is the underlying root cause of one or more incidents.
* **Testing**: Operational ITIL incident management depth.

---

## 8. Automation & Engineering (8 Questions)

### Q51: Supported tool vs. Custom script decision criteria.
* **Answer**: Use supported native/partner tools first for audit trails, role-based access control, and team maintainability. Write custom scripts only when no supported API path exists. Maintain scripts in version control with least-privilege service accounts.
* **Testing**: Engineering judgment and avoiding single-point-of-failure scripts.

### Q52: Domain-Wide Delegation service account risks.
* **Answer**: Service accounts with Domain-Wide Delegation can impersonate any user for assigned OAuth scopes. Mitigate risk: Restrict OAuth scopes to absolute minimum, store key files securely outside repositories, rotate keys, and log API activity.
* **Testing**: Managing high-risk administrative API credentials.

### Q53: What makes an automation safe to re-run?
* **Answer**: **Idempotency**—running the script multiple times produces the exact same state as running it once. Implement pre-execution state checks, dry-run reporting modes, API rate-limiting handling, and clear error logging.
* **Testing**: Production-grade scripting practices.

### Q54: Bulk job API rate-limiting (HTTP 429) resolution.
* **Answer**: Implement **exponential backoff with jitter** retries. Batch API requests where supported. Build resumable execution state logic so failed runs resume without duplicating completed work.
* **Testing**: Scalable API automation engineering.

### Q55: Testing a script that impacts 1,500 accounts.
* **Answer**: Run read-only dry-run pass and diff output against expected results across full population. Execute live test on pilot OU (including admin account). Execute phased rollout waves. Ensure a documented rollback script exists prior to execution.
* **Testing**: Operational risk management.

---

## 9. Governance & Compliance (8 Questions)

### Q56: Running an enterprise access review.
* **Answer**: Extract entitlements from directory sources (Groups, Shared Drives, Admin roles, OAuth apps). Present entitlement data to resource business owners (not IT). Timebox review window, enforce automatic revocation upon non-response, and record approvals for compliance audits.
* **Testing**: Practical access certification execution.

### Q57: Evidencing controls for auditors.
* **Answer**: Provide historical execution logs, scheduled report run histories, policy change logs, and exception review records over time—single point-in-time screenshots are insufficient evidence.
* **Testing**: Audit readiness and control operationalization.

### Q58: Mapping Workspace controls to ISO 27001.
* **Answer**: 
  * **A.9 Access Control**: OUs, Groups, SAML SSO.
  * **A.9.4 Password/MFA**: 2SV enforcement, FIDO2 security keys.
  * **A.9.2 Privileged Access**: Delegated Admin Roles, Super Admin hygiene.
  * **A.8.2 Data Classification/DLP**: Drive sharing rules, DLP detectors.
  * **A.12.4 Logging**: Admin/Login audit logs, SIT, Alert Center.
* **Testing**: Aligning technical controls with security frameworks.

### Q59: Reducing SaaS license spend without business disruption.
* **Answer**: Audit user last sign-in and active workload usage (Gmail/Drive). Reclaim dormant licenses automatically via leaver workflows. Present usage data to department budget owners before revoking access.
* **Testing**: Commercial awareness and IT financial governance.

### Q60: Vault Retention vs. Backup.
* **Answer**: **Vault** is an eDiscovery and legal compliance archive that retains data against deletion; it is not designed for point-in-time state recovery. **Backup** provides rapid point-in-time state restoration of mailboxes and file structures following corruption or ransomware.
* **Testing**: Dispelling common compliance assumptions.

### Q61: Governance process for new third-party SaaS integrations.
* **Answer**: Evaluate requested OAuth scopes, SAML/SCIM capabilities, vendor security posture, and functional overlap with existing tools. Approve via Admin Console App Access Control by OAuth Client ID.
* **Testing**: Gating third-party software risks.

---

## 10. Leadership, Strategy & Behavioural (8 Questions)

### Q62: First 90 days as Workspace Lead.
* **Answer**: 
  * **Days 1–30**: Observe and map identity flows, OU structures, security posture, and stakeholder requirements.
  * **Days 31–60**: Remediate quick wins (e.g., automated license reclamation, security misconfigurations).
  * **Days 61–90**: Present strategic 12-month roadmap aligned with business objectives.
* **Testing**: Strategic onboarding methodology for leadership roles.

### Q63: Prioritizing competing urgent demands.
* **Answer**: Rank by cost of non-occurrence: Active data security incidents / outages first $\rightarrow$ external compliance deadlines / renewals second $\rightarrow$ technical debt-reducing automation third. Make queue visibility transparent to stakeholders.
* **Testing**: Workload prioritization under pressure.

---
*Reference: Official Google Workspace Platform Owner & Lead Administrator Interview Masterclass.*
