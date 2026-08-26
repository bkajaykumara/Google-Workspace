# 5. Google Workspace Master 100 Interview Questions & Scenario Engineering Handbook

This single master handbook contains complete, unabridged, senior-level engineering answers for **all 100 interview questions** across 7 core operational domains.

---

## 📧 SECTION 1: Gmail & Email Security (Q1–Q20)

### Q1: A user complains they are not receiving emails from a specific external domain. How do you investigate and fix it?
**Answer:**
1. **Email Log Search (ELS)**: Open Google Admin Console $\rightarrow$ **Reporting $\rightarrow$ Email Log Search**. Enter the recipient's email address and sender domain. Inspect delivery status (Delivered, Bounced, Quarantined, Spam, or Not Received).
2. **If Log Shows "Bounced" or "Quarantined"**: Read the exact SMTP error code and response. If blocked by content compliance, spam filters, or tenant blocklists, adjust rules or release the message from Admin Quarantine.
3. **If Log Shows "No Message Found"**: The email never reached Google's inbound MX servers. Verify sender SPF/DKIM records, check recipient MX records via `dig MX domain.com`, and verify external DNS propagation.
4. **Inspect User Settings**: Check user's Gmail filters, blocked addresses, or auto-forwarding rules that might bypass inbox delivery.

### Q2: How do you configure Gmail to quarantine all emails containing attachments larger than 10MB?
**Answer:**
1. Navigate to **Admin Console $\rightarrow$ Apps $\rightarrow$ Google Workspace $\rightarrow$ Gmail $\rightarrow$ Compliance**.
2. Scroll to **Attachment compliance** and click **Configure** or **Add Another Rule**.
3. Set rule scope (e.g., Inbound / Internal Sending).
4. Under expressions, add a condition: **Attachment size $\ge$ 10 MB** (10,485,760 bytes).
5. Set action: **Quarantine message** (Route to Admin Quarantine).
6. Save and apply rule to target Organizational Unit (OU).

### Q3: What is the difference between Spam Filtering, Content Compliance, and Objectionable Content in Gmail?
**Answer:**
* **Spam Filtering**: Automated ML/AI engine detecting unsolicited, malicious, or phishing emails based on IP reputation, sender scoring, and header metrics.
* **Content Compliance**: Policy engine evaluating predefined regex, custom strings, or attachment attributes (e.g., credit card numbers, confidential keywords) to execute actions (Quarantine, Modify Header, Reject).
* **Objectionable Content**: Pre-packaged word/phrase dictionaries targeting profanity, adult content, or hate speech to block or quarantine matching emails.

### Q4: How would you block all emails containing executable file attachments (.exe, .bat, .msi) for the entire organization?
**Answer:**
1. Navigate to **Gmail $\rightarrow$ Compliance $\rightarrow$ Attachment compliance** at Root OU (`/`).
2. Add expression: **Attachment file type $\rightarrow$ Match any of the specified file types**. Select `.exe`, `.bat`, `.msi`, `.cmd`, `.scr`, `.vbs`, `.ps1`.
3. Set action: **Reject message** with custom bounce message: *"Executables are blocked by security policy."*
4. Save policy across all OUs. *(Note: Google automatically blocks dangerous extensions natively, but explicit compliance rules allow custom bounce notices and audit logging).*

### Q5: What is a Gmail routing rule? Explain the difference between Default Routing, Recipient Address Map, and Catch-All routing.
**Answer:**
* **Gmail Routing Rule**: Configuration governing how inbound, outbound, or internal emails are processed, modified, or re-routed.
* **Default Routing**: Evaluates domain-level routing before user mailbox delivery (e.g., dual delivery, split routing, or perimeter gateway routing).
* **Recipient Address Map**: Replaces or rewrites incoming recipient email addresses to alternate internal addresses before delivery.
* **Catch-All Routing**: Directs emails sent to non-existent user mailboxes (`unrecognized@domain.com`) to a designated admin or support inbox rather than bouncing.

### Q6: A user is receiving phishing emails that are passing spam filters. What layers of protection do you configure?
**Answer:**
1. **Safety Settings**: Enable enhanced anti-phishing protection (**Gmail $\rightarrow$ Safety $\rightarrow$ Phishing and malware**) for unauthenticated senders, spoofed domain names, and employee display name spoofing.
2. **DMARC Enforcement**: Verify receiving server enforces DMARC alignment (`p=quarantine` or `p=reject`).
3. **Inbound Banner Alerts**: Enable external sender warning banners in Gmail.
4. **Context-Aware Access & DLP**: Block link access if redirected to untrusted IPs.
5. **Report & Purge via GAM**: Extract `Message-ID` and run `gam all users delete messages query "rfc822msgid:..." doit`.

### Q7: How do you configure Gmail to send a copy of all outgoing emails from the Finance OU to a compliance mailbox?
**Answer:**
1. Select the **/Finance** OU in **Gmail $\rightarrow$ Compliance $\rightarrow$ Content Compliance**.
2. Add rule: Scope set to **Outbound**.
3. Under Actions, select **Also deliver to $\rightarrow$ Add extra recipients**.
4. Enter compliance mailbox address (`finance-compliance@company.com`).
5. Save rule for `/Finance` OU.

### Q8: What is the purpose of Email Log Search in the Admin Console? What information can you retrieve from it?
**Answer:**
ELS tracks message lifecycle events across Google Workspace mail servers for up to 30 days.
* **Retrievable Data**: Sender/Recipient email, Date/Timestamp, Message ID, Subject line, Sender IP, TLS version, Delivery Status (Delivered, Spam, Bounced, Quarantined), Triggered Routing/Compliance Rules, and Recipient Inbox details.

### Q9: How do you configure a split delivery setup where some users use Google Workspace and others use a legacy mail server?
**Answer:**
1. Point Primary Domain **MX Records** to Google (`1 SMTP.GOOGLE.COM`).
2. Add an **Inbound Route** under **Gmail $\rightarrow$ Hosts** pointing to legacy mail server IP/FQDN over TLS.
3. Under **Gmail $\rightarrow$ Routing $\rightarrow$ Default Routing**, add a rule:
   * Select **Inbound**.
   * If recipient is unrecognized in Google Workspace Directory $\rightarrow$ **Route to legacy host**.
   * Gmail delivers matching Workspace users locally, forwarding unrecognized addresses to the legacy server.

### Q10: Explain the difference between SPF, DKIM, and DMARC. What does each one protect against?
**Answer:**
* **SPF**: Validates sender IP against authorized DNS record (`v=spf1 include:_spf.google.com ~all`). Protects against IP spoofing.
* **DKIM**: Adds cryptographic signature (`DKIM-Signature`) using 2048-bit RSA keys. Protects against message tampering in transit.
* **DMARC**: Verifies domain alignment between `From:` header and SPF/DKIM domains. Dictates policy (`p=none`, `p=quarantine`, `p=reject`). Protects against domain spoofing and phishing.

### Q11: A user's emails are landing in external recipients' spam folders. What are the first 5 things you check?
**Answer:**
1. **SPF Record**: Verify SPF contains all sending IPs/includes and passes without `PermError` (>10 DNS lookups).
2. **DKIM Key**: Verify 2048-bit DKIM key is active and DNS TXT record is validated.
3. **DMARC Alignment**: Ensure `From:` domain strictly aligns with SPF/DKIM domains.
4. **Postmaster Tools & IP Reputation**: Check domain reputation score and spam rate on Google Postmaster Tools.
5. **Content & Blacklists**: Verify sending domain/IP is not listed on Spamhaus or MXToolbox blacklists.

### Q12: What is DKIM key rotation and how do you perform it in Google Workspace?
**Answer:**
* **Purpose**: Rotating DKIM selector keys annually prevents cryptographic key compromise.
* **Procedure**:
  1. Go to **Admin Console $\rightarrow$ Gmail $\rightarrow$ Authenticate email**.
  2. Select domain and click **Generate New Record** (select 2048-bit key length and new prefix selector, e.g., `google2026`).
  3. Publish new TXT record to public DNS.
  4. Wait for DNS propagation, then click **Start Authentication** in Admin Console.

### Q13: How do you configure an inbound email gateway in Google Workspace?
**Answer:**
1. Go to **Gmail $\rightarrow$ Spam, Phishing, and Malware $\rightarrow$ Inbound gateway**.
2. Enter Gateway Egress IP CIDR ranges (e.g., Proofpoint/Mimecast IPs).
3. Check **Require TLS for connection**.
4. Enable **Message tagging / Header inspection** so Google evaluates original sender IP passed in `X-Forwarded-For` headers.

### Q14: What is the difference between a Gmail blacklist, blocked sender, and a content compliance block?
**Answer:**
* **Blacklist (Spam/IP Block)**: Drops or quarantines connection based on IP/domain reputation on global blocklists.
* **Blocked Sender**: Admin or user rule explicitly sending emails from specific addresses/domains directly to Spam or Trash.
* **Content Compliance Block**: Rule-based rejection or quarantine based on policy criteria (keywords, attachments, DLP regex).

### Q15: How do you configure email confidential mode? What are its limitations?
**Answer:**
* **Configuration**: Enabled under **Gmail $\rightarrow$ User settings $\rightarrow$ Confidential mode**.
* **Features**: Disables forwarding, copying, printing, downloading; sets expiration dates; requires SMS passcode.
* **Limitations**: Does not prevent recipients from taking screenshots or physical camera photos; third-party email clients render web links rather than inline text.

### Q16: A user has enabled auto-forwarding of all emails to a personal Gmail account. How do you detect and disable this?
**Answer:**
1. **Detection**: Run a report in **Reporting $\rightarrow$ Audit $\rightarrow$ Admin / User Logins** or via GAM: `gam all users print forwarding`.
2. **Disable Tenant-Wide**: Go to **Gmail $\rightarrow$ End User Access $\rightarrow$ Automatic forwarding** and set to **Disabled**.
3. **GAM Remediation**: `gam user alex.smith@company.com delete forwardingaddress personal@gmail.com`.

### Q17: How do you set up email archiving (Vault) for all users in a specific OU?
**Answer:**
1. Log into **Google Vault** (`vault.google.com`).
2. Navigate to **Retention $\rightarrow$ Custom Rules $\rightarrow$ Create**.
3. Select **Service: Gmail**, specify target **OU** (e.g., `/Finance`).
4. Set retention period (e.g., **Indefinitely** or **7 Years / 2555 Days**).
5. Action: **Keep forever** or **Purge after retention period**.

### Q18: What is Postmaster Tools and what information does it provide that the Admin Console doesn't?
**Answer:**
Google Postmaster Tools provides external delivery analytics for domain owners sending to Gmail users:
* **Metrics**: Domain Reputation, IP Reputation, Spam Rate (user-reported), SPF/DKIM/DMARC Success Rates, Encryption (TLS) percentage, and Delivery Errors.

### Q19: How would you configure Gmail to reject all emails that fail DMARC alignment?
**Answer:**
1. Publish DNS DMARC TXT Record: `v=DMARC1; p=reject; rua=mailto:dmarc-reports@company.com; pct=100`.
2. In Google Admin Console, go to **Gmail $\rightarrow$ Safety $\rightarrow$ Spoofing and authentication**, and enable **Reject unauthenticated email failing DMARC**.

### Q20: Explain the difference between p=none, p=quarantine, and p=reject in a DMARC record. When would you use each?
**Answer:**
* `p=none` (**Monitoring**): Collects aggregate reports without affecting mail delivery. Used during initial deployment.
* `p=quarantine` (**Soft Enforcement**): Routes unauthenticated mail to recipient Spam or Admin Quarantine. Used during policy validation.
* `p=reject` (**Hard Enforcement**): Rejects unauthenticated mail at the SMTP gateway (`550 5.7.26`). Used in mature zero-trust environments.

---

## 🛡️ SECTION 2: Security & Context-Aware Access (Q21–Q40)

### Q21: What is Context-Aware Access (CAA) and how does it implement Zero Trust in Google Workspace?
**Answer:**
CAA evaluates dynamic contextual attributes (device compliance, IP CIDR, geofence, Chrome management state) using Common Expression Language (CEL) at request time to grant or deny access to specific Google apps without relying on static perimeter firewalls.

### Q22: A user is working from a personal laptop. How do you ensure they can access Gmail but NOT Google Drive?
**Answer:**
1. Create Access Level **"Corporate_Managed_Device_Only"**: `device.chrome.management_state == ChromeManagementState.CHROME_MANAGEMENT_STATE_BROWSER_MANAGED`.
2. Go to **Security $\rightarrow$ Context-Aware Access $\rightarrow$ Assign Access Levels**.
3. Under **Google Drive**, assign **"Corporate_Managed_Device_Only"**.
4. Leave **Gmail** unassigned or assign a basic MFA access level.

### Q23: What is Endpoint Verification and how is it different from Mobile Device Management (MDM)?
**Answer:**
* **Endpoint Verification**: Lightweight Chrome extension reporting OS, encryption status, serial number, and screen lock to Admin Console for CAA policy evaluation.
* **MDM (Advanced Endpoint Management)**: Full management agent enforcing password profiles, app deployment, remote wipe, and device configuration profiles.

### Q24: How do you block access to Google Workspace from a specific country or region?
**Answer:**
1. Create a CAA Access Level in Advanced Mode using CEL:
   ```cel
   !origin.region_code.in(["RU", "CN", "IR", "KP"])
   ```
2. Assign this Access Level across all Core Google Apps.

### Q25: What is the Alert Center in Google Workspace? Name 5 types of alerts you would configure.
**Answer:**
Centralized threat dashboard in Admin Console.
* **5 Critical Alerts**: 1. Super Admin Password Reset, 2. Suspicious Login Activity, 3. User Suspended for Abuse, 4. DLP Rule Violation, 5. High-Risk OAuth App Grant.

### Q26: How do you configure a CAA policy to allow access only from corporate-managed devices?
**Answer:**
1. Go to **Security $\rightarrow$ Access and data control $\rightarrow$ Context-Aware Access**.
2. Create Level: `device.is_managed_device == true && device.encryption_status == EncryptionStatus.ENCRYPTED`.
3. Assign Level to Root OU across all Apps.

### Q27: What is BeyondCorp and how does Google Workspace implement its principles?
**Answer:**
BeyondCorp is Google's Zero Trust architecture shifting access control from network perimeters to individual users and device security postures evaluated per request via Context-Aware Access.

### Q28: A Super Admin account has been compromised. What are your first 5 steps?
**Answer:**
1. **Reset Password & Revoke Session Tokens**: Force password reset and execute immediate session signout via Admin Console.
2. **Revoke 2SV / Security Keys**: Remove registered 2SV devices and register a new FIDO2 Hardware Key.
3. **Audit Admin Activity Logs**: Inspect **Admin Audit Log** for unauthorized role grants, DNS edits, or SSO policy changes.
4. **Revoke OAuth Grants**: Purge all third-party app tokens granted by the admin.
5. **Isolate & Report**: Inspect secondary recovery email/phone for unauthorized changes and notify SOC.

### Q29: How do you enforce 2-Step Verification (2SV) for specific OUs while keeping it optional for others?
**Answer:**
1. Go to **Security $\rightarrow$ Authentication $\rightarrow$ 2-Step Verification**.
2. Select target OU (e.g., `/Finance`).
3. Set **Enforcement $\rightarrow$ Turn on enforcement**. Select enforcement date and allowed methods (Security Key / Google Prompt).
4. Select `/Contractors` OU and set Enforcement to **Off**.

### Q30: What is the difference between basic and advanced mobile device management in Google Workspace?
**Answer:**
* **Basic MDM**: Requires no client agent; enforces basic passcode, screen lock, and selective account wipe.
* **Advanced MDM**: Requires Device Policy App; enforces work profiles, app management, complete device wipe, and CAA device compliance rules.

### Q31: How do you perform a security audit of all third-party apps connected to Google Workspace via OAuth?
**Answer:**
1. Go to **Security $\rightarrow$ Access and data control $\rightarrow$ API Controls $\rightarrow$ App Access Control**.
2. Inspect installed app list, requested OAuth scopes, user counts, and risk scores.
3. Change untrusted high-risk apps to **Blocked**.

### Q32: A user's account shows login activity from two different countries within 30 minutes. How do you respond?
**Answer:**
1. **Triggered Incident**: Impossible Travel alert fires in Alert Center.
2. **Immediate Action**: Execute GAM command: `gam user alex.smith@company.com signout` and suspend account.
3. **Investigation**: Inspect Login Audit Log for source IPs, user-agent strings, and MFA verification logs.
4. **Remediation**: Force password reset and re-enroll 2SV.

### Q33: What is Google Workspace's approach to data encryption at rest and in transit?
**Answer:**
* **In Transit**: Encrypted using TLS 1.2/1.3 with Perfect Forward Secrecy across external and internal Google connections.
* **At Rest**: Encrypted using AES-256 or AES-128; file data is split into chunks, encrypted with unique data keys, and master keys are managed via FIPS 140-2 validated KMS. Support for Client-Side Encryption (CSE).

### Q34: How do you configure an activity-based alert rule for when a user downloads more than 100 Drive files in one hour?
**Answer:**
1. Go to **Security $\rightarrow$ Security Center $\rightarrow$ Investigation Tool**.
2. Data source: **Drive log events**. Condition: **Event == Download**, Group by **Actor**, Threshold **> 100 in 1 hour**.
3. Click **Create Rule**, set Action to **Send notification to Alert Center** and email SOC admins.

### Q35: What is the purpose of Security Investigation Tool in Google Workspace? How is it different from the Audit Log?
**Answer:**
* **Audit Log**: Static event history log viewer.
* **Security Investigation Tool**: Interactive security engine allowing cross-log querying, automated triage, and direct remediation actions (e.g., bulk file deletion, token revocation, user suspension directly from query results).

### Q36: How do you set up a Hardware Security Key (FIDO2) enforcement policy for Admin accounts only?
**Answer:**
1. Select `/Admins` OU in **Security $\rightarrow$ Authentication $\rightarrow$ 2-Step Verification**.
2. Under **Allowed 2SV methods**, select **Only Security Key**.
3. Enforce policy for `/Admins` OU.

### Q37: What is an Access Transparency log and who can view it?
**Answer:**
Logs actions taken by Google staff when accessing customer data for administrative/support obligations. Accessible to Super Admins with Enterprise Plus licenses under Audit Logs.

### Q38: How do you configure Google Workspace to block access from Tor exit nodes or anonymous VPN IPs?
**Answer:**
Use Context-Aware Access CEL rule matching threat intelligence IP ranges or integrate an inline CASB (Netskope) to inspect and drop unverified proxy sessions.

### Q39: What is the difference between a Super Admin, a Delegated Admin, and a read-only Admin?
**Answer:**
* **Super Admin**: Complete, unrestricted root access to all tenant configurations and data.
* **Delegated Admin**: Role-Based Access Control (RBAC) privileges scoped to specific tasks/OUs (e.g., User Management Admin).
* **Read-Only Admin**: Access restricted to viewing audit logs and reports without write/modify permissions.

### Q40: How do you configure an automatic response to suspend a user account when suspicious activity is detected?
**Answer:**
Configure an Alert Center Rule with automated remediation triggers or integrate Google Workspace System Event Webhooks with a serverless Apps Script / Python SDK automation pipeline executing GAM suspension.

---

## 👥 SECTION 3: User & OU Management (Q41–Q55)

### Q41: You need to onboard 500 users at once. What are the different methods available and which would you recommend?
**Answer:**
* **Methods**: Admin Console CSV import, GAM CLI (`gam csv users.csv gam create user ...`), SCIM IdP provisioning, Directory API SDK.
* **Recommendation**: **SCIM via IdP (Entra ID / OneLogin)** for automated lifecycle management; **GAM CLI** for one-time bulk manual imports.

### Q42: What is the difference between suspending a user and deleting a user in Google Workspace? What happens to their data?
**Answer:**
* **Suspending**: Disables login instantly; preserves all data (Drive, Mail, Calendar); continues license consumption unless reassigned.
* **Deleting**: Purges account; data is permanently deleted after 20 days unless transferred during deletion workflow.

### Q43: How do you transfer all Drive content from a departing employee to their manager?
**Answer:**
1. **Admin Console**: Go to **Users $\rightarrow$ Select User $\rightarrow$ Delete User $\rightarrow$ Transfer Data** (enter Manager Email).
2. **GAM CLI**:
   ```bash
   gam user leaver@company.com transfer drive manager@company.com keep-shares
   ```

### Q44: A user is being offboarded. List all the steps you would take to properly offboard them in Google Workspace.
**Answer:**
1. Reset password & execute global signout (`gam user <email> signout`).
2. Revoke OAuth tokens and 2SV security keys.
3. Transfer Drive files and Calendar events to manager/successor.
4. Move user to `/_Leavers` OU and suspend account.
5. Reclaim Enterprise license (assign Cloud Identity Free license).
6. Verify Google Vault Litigation Hold status.

### Q45: How do you configure a catch-all email address in Google Workspace?
**Answer:**
Go to **Gmail $\rightarrow$ Routing $\rightarrow$ Catch-all address**, select **Forward unrecognized emails to:** `catchall@company.com`.

### Q46: What is the difference between a Google Group and a Shared Mailbox in Google Workspace?
**Answer:**
* **Google Group**: Mailing list/collaborative inbox entity (no separate license fee).
* **Shared Mailbox**: Implemented in Workspace via a licensed user account with delegated inbox permissions or Collaborative Group.

### Q47: How do you configure a user alias? Can an alias receive calendar invites?
**Answer:**
Added in **Users $\rightarrow$ User Details $\rightarrow$ Alternate email addresses**. Yes, aliases receive calendar invites directly in the primary inbox.

### Q48: How do you bulk update user attributes (department, job title, phone number) using the Admin Console?
**Answer:**
Go to **Users $\rightarrow$ Bulk update users**, download user CSV, update columns (`Department`, `Title`, `Phone`), and upload updated CSV.

### Q49: What is the difference between an Organizational Unit (OU) and a Group in Google Workspace? When would you use each for policy application?
**Answer:**
* **OU**: Hierarchical tree structure defining baseline setting inheritance.
* **Group**: Cross-OU entity used for flexible policy overrides (e.g., CAA rules, App access).

### Q50: A manager wants to see their team member's calendar. How do you configure Calendar delegation?
**Answer:**
In Calendar Settings $\rightarrow$ **Share with specific people**, add Manager email and grant **Make changes and manage sharing** permissions.

### Q51: How do you configure a "no-reply" email address that can send but not receive emails?
**Answer:**
Create `noreply@company.com` group/address, configure Gmail Routing rule to drop all incoming messages to `noreply@` with an automated rejection response.

### Q52: What are the different types of Google Groups (mailing list, forum, collaborative inbox, Q&A forum)?
**Answer:**
* **Mailing List**: Distribution group forwarding emails to members.
* **Collaborative Inbox**: Topic assignment and resolution tracking group.
* **Web Forum**: Web-based discussion board.
* **Q&A Forum**: Question voting and answer marking group.

### Q53: How do you restore a deleted user and their data? What is the restore time limit?
**Answer:**
Go to **Users $\rightarrow$ Recently deleted**, select user, click **Restore**. **Time Limit**: **20 days** post-deletion.

### Q54: How do you configure a shared drive and set member permissions (Viewer, Commenter, Contributor, Content Manager, Manager)?
**Answer:**
1. Create Shared Drive in Drive.
2. Add members with roles:
   * **Viewer**: Read-only access.
   * **Commenter**: Read + comment access.
   * **Contributor**: Read/write/edit files (cannot delete/move folders).
   * **Content Manager**: Full file/folder management.
   * **Manager**: Full administrative member & drive settings control.

### Q55: What is Directory Sharing and how do you control what user information is visible in the Global Address List (GAL)?
**Answer:**
Configured in **Directory $\rightarrow$ Directory settings $\rightarrow$ Profile editing**. Controls custom field visibility across GAL.

---

## 💾 SECTION 4: Google Drive & Data Protection (Q56–Q70)

### Q56: What is the difference between My Drive and Shared Drives from an admin governance perspective?
**Answer:**
* **My Drive**: Files owned by individual users; deleted if owner account is deleted without data transfer.
* **Shared Drive**: Files owned by the domain organization; data persists regardless of individual member offboarding.

### Q57: How do you configure Drive to prevent users from downloading, printing, or copying files shared with them?
**Answer:**
In file sharing settings, check **Viewers and commenters can see the option to download, print, and copy = DISABLED**, or enforce tenant-wide via Information Rights Management (IRM) policies.

### Q58: What are Drive Trust Rules and how do they differ from standard sharing settings?
**Answer:**
Trust Rules provide granular, condition-based Drive sharing policies overriding legacy OU toggles. Precedence rule: **Most restrictive setting always wins**.

### Q59: How do you find all files in your organization that have been shared publicly ("anyone with the link")?
**Answer:**
Run Security Investigation Tool query: **Data Source: Drive log events $\rightarrow$ Visibility == Public on the web / Anyone with link**, or execute GAM command: `gam user all show fileaccess filter "visibility == 'anyoneWithLink'"`.

### Q60: What is Google Vault and what types of data can it preserve, search, and export?
**Answer:**
Governance tool preserving Gmail, Drive, Groups, Chat, and Meet recordings for legal compliance.

### Q61: How do you configure a Vault retention rule for all Gmail data for a specific OU for 7 years?
**Answer:**
Create Custom Retention Rule in Vault: **Service: Gmail $\rightarrow$ Scope: OU (/Finance) $\rightarrow$ Duration: 2555 Days $\rightarrow$ Action: Purge after retention**.

### Q62: A legal hold has been placed on a specific user. How do you configure this in Google Vault?
**Answer:**
Create Matter in Vault $\rightarrow$ **Holds $\rightarrow$ Create $\rightarrow$ Service: Gmail/Drive $\rightarrow$ Target: Specific User (`alex.smith@company.com`)**. (Litigation Hold overrides retention rules indefinitely).

### Q63: What is the difference between a DLP Audit rule and a DLP Block rule?
**Answer:**
* **Audit Rule**: Logs matching event silently to Admin log without blocking user action.
* **Block Rule**: Prevents file sharing/email transmission and displays security policy modal.

### Q64: How would you configure DLP to detect and block sharing of credit card numbers in Google Drive?
**Answer:**
Create DLP Rule $\rightarrow$ **Detector: Credit Card Number $\rightarrow$ Action: Block external sharing $\rightarrow$ Alert: Notify SOC**.

### Q65: How do you configure Drive to prevent external sharing for all users except the Sales team?
**Answer:**
Set Root OU Drive Sharing to **OFF (Restricted)**. Override `/Sales` OU Drive Sharing to **ON (Allow External Sharing)**.

### Q66: What are Google Drive Labels and how are they used in DLP policies?
**Answer:**
Taxonomy tags (e.g., `Confidential`, `PCI-Data`) applied manually/automatically to Drive files, triggering DLP policies.

### Q67: A user accidentally permanently deleted an important Drive file 35 days ago. Can it be recovered? How?
**Answer:**
Admin Console Drive Data Restore window is **25 days post-trash**. Past 35 days, recovery is **ONLY possible via Google Vault** if an active retention rule or litigation hold preserved the file.

### Q68: What is the difference between Drive activity reports and Drive audit logs?
**Answer:**
* **Activity Reports**: Aggregated usage trends/charts.
* **Audit Logs**: Granular raw event stream (`File Create`, `ACL Change`, `Download`).

### Q69: How do you configure a Shared Drive to prevent members from moving files out of it?
**Answer:**
In Shared Drive Settings, check **Prevent members from moving files out of this shared drive**.

### Q70: What are the different levels of Drive sharing permissions and what can each level do?
**Answer:**
Viewer (Read), Commenter (Read+Comment), Editor (Read/Write/Delete), Manager (Full Admin).

---

## 🔐 SECTION 5: SSO, SAML, OAuth & Identity (Q71–Q80)

### Q71: A user's SSO login is failing with a "SAML Response signature invalid" error. How do you troubleshoot?
**Answer:**
1. Verify X.509 certificate in Google matches IdP public key.
2. Check ACS URL (`https://www.google.com/a/domain.com/acs`) and Audience URI (`google.com/a/domain.com`).
3. Check NTP server clock drift (>300s).

### Q72: What is the difference between SP-initiated and IdP-initiated SSO? Which one does Google Workspace use by default?
**Answer:**
* **SP-Initiated**: User visits `mail.google.com` first, gets redirected to IdP. (Default in Google Workspace).
* **IdP-Initiated**: User clicks app tile inside IdP portal (OneLogin) directly.

### Q73: Can you configure different SSO profiles for different OUs in Google Workspace? How?
**Answer:**
Yes. Using **Partial SSO (Manage SSO Profiles)**, assign distinct SAML SSO profiles per OU.

### Q74: What happens when a Super Admin tries to login via SSO?
**Answer:**
Can authenticate via SSO, but Google maintains local password fallback to prevent admin lockout during IdP outages.

### Q75: How do you audit which third-party OAuth apps have access to user data in Google Workspace?
**Answer:**
Inspect **Security $\rightarrow$ API Controls $\rightarrow$ App Access Control**.

### Q76: How would you revoke all OAuth tokens for a compromised user?
**Answer:**
Execute GAM command: `gam user <email> delete tokens`, or click **Reset OAuth Tokens** in Admin Console user security tab.

### Q77: What is the difference between SAML 2.0 and OIDC for enterprise SSO? When would you choose one over the other?
**Answer:**
* **SAML 2.0**: XML enterprise federation for legacy/SaaS web apps.
* **OIDC**: JSON JWT layer for mobile apps, SPAs, and API token integration.

### Q78: How do you configure Google Workspace as a Service Provider (SP) with OneLogin as the Identity Provider (IdP)?
**Answer:**
Set Sign-in URL, Sign-out URL, and upload OneLogin X.509 certificate under **Security $\rightarrow$ SSO with third-party IdP**.

### Q79: What is SCIM provisioning and how does it complement SSO in a Google Workspace + OneLogin setup?
**Answer:**
SSO handles login authentication; SCIM automates real-time Joiner-Mover-Leaver user creation, updates, and suspensions via REST APIs.

### Q80: What certificates are exchanged between Google Workspace and an IdP during SAML setup, and why?
**Answer:**
IdP uploads its public X.509 Certificate to Google Workspace so Google can cryptographically verify incoming SAML assertion signatures.

---

## 📱 SECTION 6: Mobile & Endpoint Management (Q81–Q90)

### Q81: What is the difference between Basic Mobile Management and Advanced Mobile Device Management in Google Workspace?
**Answer:**
Basic enforces passcodes without client agents; Advanced requires Device Policy App for complete management, app distribution, and CAA rules.

### Q82: How do you remotely wipe a lost Android device that is enrolled in Google Workspace MDM?
**Answer:**
Admin Console $\rightarrow$ **Devices $\rightarrow$ Select Device $\rightarrow$ Wipe Device / Wipe Account**.

### Q83: How do you configure a policy to require a screen lock PIN on all managed mobile devices?
**Answer:**
Go to **Devices $\rightarrow$ Password requirements**, set Minimum PIN length (e.g., 6 digits) and enforce across OUs.

### Q84: What happens to Google Workspace data on a mobile device when a user is offboarded?
**Answer:**
Executing **Account Wipe** removes enterprise work profile data while preserving user personal photos/data on BYOD devices.

### Q85: How do you block jailbroken or rooted devices from accessing Google Workspace?
**Answer:**
Enable **Block compromised devices** in Advanced Mobile Management settings.

### Q86: What is Endpoint Verification and what information does it report to the Admin Console?
**Answer:**
Chrome extension reporting OS version, encryption status, screen lock, and serial number for CAA evaluation.

### Q87: How do you configure a policy that allows only managed devices to access Google Drive?
**Answer:**
Create CAA Access Level `device.is_managed_device == true` and assign to Google Drive.

### Q88: What is the difference between a corporate-owned device and a BYOD device in Google Workspace management?
**Answer:**
Corporate-owned devices allow full wipe and serial tracking; BYOD isolates corporate data inside containerized work profiles.

### Q89: How do you push a Chrome extension to all managed Chrome browsers across the organization?
**Answer:**
Go to **Devices $\rightarrow$ Chrome $\rightarrow$ Apps & extensions**, add extension ID, set Installation policy to **Force install**.

### Q90: What is Chrome Enterprise and how does it differ from Chrome Browser Cloud Management?
**Answer:**
Chrome Enterprise includes OS management/licensing; CBCM provides cloud browser policy management across desktop platforms.

---

## 📊 SECTION 7: Reporting, Audit & Compliance (Q91–Q100)

### Q91: What are the different types of audit logs available in Google Workspace and what does each one track?
**Answer:**
Admin (settings changes), Drive (file ops), Login (auth events), SAML (SSO events), Token (OAuth grants), Groups (membership changes).

### Q92: How do you configure an automated weekly report of all admin activity changes in the domain?
**Answer:**
Create Saved Query in Security Investigation Tool $\rightarrow$ **Create Alert Rule $\rightarrow$ Set Frequency: Weekly $\rightarrow$ Email Admin**.

### Q93: How do you export Google Workspace audit logs to BigQuery for long-term retention and analysis?
**Answer:**
Go to **Reporting $\rightarrow$ BigQuery Export**, authorize GCP Project, and enable continuous log streaming.

### Q94: What is the difference between the Reports section and the Security Investigation Tool in the Admin Console?
**Answer:**
Reports provides static graphs/summaries; Investigation Tool offers interactive cross-log querying and direct remediation actions.

### Q95: How do you set up an alert that fires when a new Super Admin account is created?
**Answer:**
Alert Center Rule: **Event == Admin Privilege Grant $\rightarrow$ Privilege == Super Admin $\rightarrow$ Action: Send Immediate Alert**.

### Q96: A compliance team needs a report of all files shared externally in the last 90 days. How do you generate this?
**Answer:**
Security Investigation Tool: **Data source: Drive log events $\rightarrow$ Visibility == Shared Externally $\rightarrow$ Time: Last 90 days $\rightarrow$ Export to Sheets**.

### Q97: How do you monitor and report on users who have not enabled 2-Step Verification?
**Answer:**
Go to **Reporting $\rightarrow$ User reports $\rightarrow$ Security $\rightarrow$ Filter: 2-Step Verification Enrolled == False**.

### Q98: What is Google Workspace's data retention policy for audit logs natively, and how do you extend it?
**Answer:**
Native retention is **6 months** (180 days). Extended indefinitely by streaming logs to **Google Cloud BigQuery** or a third-party SIEM.

### Q99: How do you investigate a suspected data breach where an insider may have exfiltrated data via Google Drive?
**Answer:**
Query Security Investigation Tool for Actor $\rightarrow$ Inspect File Downloads, External Shares, Ownership Transfers, and OAuth App Grants.

### Q100: A regulatory body requires you to prove that emails were never tampered with in transit. Which Google Workspace features and logs would you present as evidence?
**Answer:**
Present **ELS TLS Logs** (TLS 1.2/1.3 PFS verification), **DKIM Cryptographic Headers** (`d=` domain validation logs), **MTA-STS Enforcement Logs**, and **Google Vault Cryptographic Export Hash Manifests**.
