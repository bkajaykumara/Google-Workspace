# 5. Google Workspace Senior Engineering Master 100 Interview Questions, Detailed Explanations & Follow-up Handbook

This master handbook provides exhaustive, senior-systems-engineer level answers, deep architectural explanations, Admin Console navigation paths, GAM CLI commands, and probing **Follow-up Questions & Answers** for all 100 questions across 7 core operational domains.

---

## 📧 SECTION 1: Gmail & Email Security (Q1–Q20)

### Q1: A user complains they are not receiving emails from a specific external domain. How do you investigate and fix it?
**Detailed Technical Answer:**
1. **Email Log Search (ELS) Investigation**: Navigate to **Admin Console $\rightarrow$ Reporting $\rightarrow$ Email Log Search**. Enter the recipient email, sender domain (or sender IP), and date range (up to 30 days). Inspect the message trajectory status:
   * **Delivered**: Check the recipient's Gmail inbox filters, "All Mail", Spam, or Trash. The user may have a custom filter archiving or deleting matching emails.
   * **Quarantined**: The email triggered a Content Compliance, Attachment Compliance, or Anti-Spam rule. Go to **Admin Console $\rightarrow$ Apps $\rightarrow$ Google Workspace $\rightarrow$ Gmail $\rightarrow$ Admin Quarantine** to review headers, body content, and release or drop the message.
   * **Bounced**: Click the log entry to read the exact SMTP bounce error code (e.g., `550 5.7.1 Access Denied`, `550 5.7.26 DMARC Failure`).
   * **Not Received / No Message Found**: The message never reached Google's perimeter MX servers (`1 SMTP.GOOGLE.COM`).
2. **Perimeter & DNS Diagnostics**: Run `dig MX recipientdomain.com` to confirm MX records point to Google. Execute `dig TXT senderdomain.com` to inspect the sender's SPF record. If the sender's SPF fails or lacks DMARC alignment, Google's receiving gateway may defer or drop the connection.
3. **Whitelist & Gateway Adjustment**: If the sender domain is legitimate but failing reputation checks, navigate to **Gmail $\rightarrow$ Spam, Phishing, and Malware $\rightarrow$ Email whitelist** (add sender IP) or **Inbound Gateway** (if routing through an external spam filter like Proofpoint/Mimecast).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What if the external sender claims their server received an HTTP/SMTP 421 deferral error code from Google when sending to your domain?"*
* **Senior Engineer Answer**: An SMTP 421 response code (`421 4.7.0 Try again later`) indicates Google is rate-limiting or temporarily deferring the connection because the sending IP address or domain lacks established reputation, spiked sending volume unexpectedly, or triggered rate limit thresholds. To resolve this: 1. Confirm the sender isn't listed on global blocklists (Spamhaus/Abuseat), 2. Instruct the sender to publish valid SPF and DKIM records, 3. If using an authorized third-party gateway, ensure their egress IPs are listed under **Gmail $\rightarrow$ Inbound Gateway** with **Require TLS** enabled so Google bypasses aggressive IP rate-limiting.

---

### Q2: How do you configure Gmail to quarantine all emails containing attachments larger than 10MB?
**Detailed Technical Answer:**
1. Navigate to **Admin Console $\rightarrow$ Apps $\rightarrow$ Google Workspace $\rightarrow$ Gmail $\rightarrow$ Compliance**.
2. Select the target Organizational Unit (e.g., Root `/` or `/Employees`).
3. Scroll to **Attachment compliance** and click **Configure** (or **Add Another Rule**).
4. Rule Name: `Enforce 10MB Inbound Attachment Size Limit`.
5. Under **Email messages to affect**, select **Inbound** (and optionally **Internal - Sending**).
6. Under **Add expressions**, click **Add**:
   * Select **Attachment attribute $\rightarrow$ Attachment size**.
   * Select condition **Greater than or equal to** $\rightarrow$ Enter `10485760` bytes (or `10 MB`).
7. Under **If the above expressions match**, select **Quarantine message**.
8. Select the destination quarantine: **Default Admin Quarantine**.
9. Under **Rejection notice**, optionally configure an automated bounce notice to the sender.
10. Click **Save**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you allow the Executive OU to bypass this 10MB attachment restriction while keeping it strictly enforced for all other employees?"*
* **Senior Engineer Answer**: Configure the Attachment Compliance rule at the Root OU level (`/`). Select the child OU `/Executives`, locate the inherited policy entry under **Attachment compliance**, and click **Override** (or create an explicit OU-level rule for `/Executives` setting the threshold to 25MB). Alternatively, add an **Exception Expression** within the same compliance rule: *Match expression ONLY IF User is NOT a member of group `executives-group@company.com`*.

---

### Q3: What is the difference between Spam Filtering, Content Compliance, and Objectionable Content in Gmail?
**Detailed Technical Answer:**
1. **Spam Filtering**: Automated ML/AI threat engine evaluating global sender IP reputation, domain history, header anomalies, DKIM signatures, DMARC alignment, and link destination risk.
2. **Content Compliance**: Policy engine evaluating predefined regex, custom strings, or attachment attributes (e.g., credit card numbers, SSNs, project codenames) to execute actions (Quarantine, Modify Header, Re-route, Reject).
3. **Objectionable Content**: Dictionary-based filtering tool utilizing pre-packaged or custom word lists targeting profanity, adult content, hate speech, or harassment.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"If an incoming email triggers both a Content Compliance rule (set to Quarantine) and a Spam Filter rule (set to deliver to Spam), which policy action takes precedence?"*
* **Senior Engineer Answer**: **Content Compliance rules take precedence over native Spam Filtering.** In Gmail's processing pipeline, administrative compliance rules execute prior to final inbox delivery decisions. If a Content Compliance rule matches and specifies **Quarantine message**, the email is intercepted and routed directly to the Admin Quarantine, preventing it from ever landing in the user's personal Spam folder.

---

### Q4: How would you block all emails containing executable file attachments (.exe, .bat, .msi) for the entire organization?
**Detailed Technical Answer:**
1. Open **Admin Console $\rightarrow$ Apps $\rightarrow$ Google Workspace $\rightarrow$ Gmail $\rightarrow$ Compliance**.
2. Select Root OU (`/`). Scroll to **Attachment compliance** $\rightarrow$ Click **Add Another Rule**.
3. Rule Name: `Global Block Executable File Extensions`.
4. Scope: Select **Inbound**, **Outbound**, and **Internal - Sending**.
5. Under Expressions, select **Add**:
   * Choose **Attachment attribute $\rightarrow$ Attachment file type**.
   * Select **Match any of the specified file types**.
   * Check boxes: `Executable (.exe)`, `Batch file (.bat)`, `Windows Installer (.msi)`, `Command script (.cmd)`, `PowerShell (.ps1)`, `JavaScript (.js)`, `VBScript (.vbs)`.
6. Action: Select **Reject message**.
7. Custom Rejection Notice: Enter *"Security Policy Violation: Executable attachments (.exe, .bat, .msi) are strictly prohibited."*
8. Click **Save**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do attackers bypass standard extension blocking using archived or encrypted files (.zip, .7z, .iso), and how do you mitigate this in Gmail?"*
* **Senior Engineer Answer**: Attackers embed executables inside multi-layer or encrypted archives to prevent static scanners from reading inner extensions. Mitigation: 1. Add an Attachment Compliance rule blocking `.7z`, `.rar`, `.iso`, `.img`, `.cab` extensions. 2. Enable **Safety settings $\rightarrow$ Phishing and malware $\rightarrow$ Protection against encrypted attachments** to quarantine or reject password-protected archives. 3. Require external partners to send files via shared Google Drive links governed by DLP policies.

---

### Q5: What is a Gmail routing rule? Explain the difference between Default Routing, Recipient Address Map, and Catch-All routing.
**Detailed Technical Answer:**
* **Default Routing**: Evaluates domain-level inbound and outbound traffic *before* user mailbox delivery rules trigger. Used for split delivery, dual delivery, or perimeter gateway routing.
* **Recipient Address Map**: Executes address rewriting, mapping an incoming recipient address to one or more alternate internal addresses before delivery.
* **Catch-All Routing**: Governs unrecognized recipient email addresses (`nonexistent@company.com`). Dictates whether to bounce (Recommended), discard, or forward unrecognized mail to an admin catch-all inbox.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Why is configuring a Catch-All address to forward to an admin inbox considered a major security risk?"*
* **Senior Engineer Answer**: 1. **Spam & Flooding**: Spammers generate millions of random dictionary emails (`a@company.com`, `admin@company.com`), overwhelming the catch-all inbox with spam and malware. 2. **Spear-Phishing Exposure**: Attackers send targeted spear-phishing emails to guessed addresses knowing they land in an IT admin's inbox. Best practice is setting Catch-All behavior to **Reject message** (`550 5.1.1 User unknown`).

---

### Q6: A user is receiving phishing emails that are passing spam filters. What layers of protection do you configure?
**Detailed Technical Answer:**
1. **Gmail Enhanced Safety Settings**: Enable **Gmail $\rightarrow$ Safety $\rightarrow$ Phishing and malware** settings for employee display name spoofing, domain spoofing, unauthenticated senders, and shortener warning banners.
2. **DMARC Enforcement**: Publish DMARC `p=quarantine` or `p=reject` record and enable **Gmail $\rightarrow$ Safety $\rightarrow$ Reject unauthenticated emails failing DMARC**.
3. **External Sender Warning Banners**: Enable amber warning banners on all inbound external emails.
4. **Automated Purge via GAM**: Extract `Message-ID` and run:
   ```bash
   gam all users delete messages query "rfc822msgid:123456789@phisher.com" doit
   ```

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you prevent 'Display Name Spoofing' where an attacker uses a free Gmail account named 'CEO John Doe <attacker@gmail.com>'?"*
* **Senior Engineer Answer**: Enable **Gmail $\rightarrow$ Safety $\rightarrow$ Protect against employee display name spoofing**. Input the names and addresses of executives. When an external email matches an executive's display name, Gmail displays a prominent amber warning banner or moves the message to Spam/Quarantine.

---

### Q7: How do you configure Gmail to send a copy of all outgoing emails from the Finance OU to a compliance mailbox?
**Detailed Technical Answer:**
1. Open **Admin Console $\rightarrow$ Apps $\rightarrow$ Google Workspace $\rightarrow$ Gmail $\rightarrow$ Compliance**.
2. Select target OU: **/Finance** $\rightarrow$ **Content compliance** $\rightarrow$ Click **Configure**.
3. Name: `BCC Outbound Finance Mail to Compliance Archive`.
4. Scope: Check **Outbound** and **Internal - Sending**.
5. Expressions: Select **Match ALL** $\rightarrow$ Add expression **Any content**.
6. Actions: Under **Also deliver to**, check **Add extra recipients** $\rightarrow$ Add `finance-compliance-archive@company.com`.
7. Click **Save**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you ensure the sender in the Finance OU cannot see or tamper with this compliance copy delivery rule?"*
* **Senior Engineer Answer**: Content Compliance rules execute on Google's backend MTA layer, completely decoupled from the user's Gmail client. Users have zero visibility into Content Compliance rules, and extra recipients added via "Also deliver to" are injected at the MTA layer (like an envelope BCC), leaving no trace in the sender's local Sent Items folder or headers.

---

### Q8: What is the purpose of Email Log Search in the Admin Console? What information can you retrieve from it?
**Detailed Technical Answer:**
**Email Log Search (ELS)** tracks message lifecycle events across Google Workspace mail servers for the past 30 days.

#### Retrievable Telemetry Fields:
* **Metadata**: Date/Time (UTC), Message-ID, Subject, Sender/Recipient email & IP address, File size.
* **Authentication**: SPF result, DKIM signature verification, DMARC alignment status.
* **Encryption**: TLS version (TLS 1.2/1.3) and cipher suite.
* **Status & Policies**: Delivered, Bounced, Quarantined, Spam, Rejected, and triggered Compliance/DLP rule names.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you investigate mail delivery logs older than 30 days?"*
* **Senior Engineer Answer**: Configure **BigQuery Log Export** under **Reporting $\rightarrow$ BigQuery Export**. Connect a GCP Project and BigQuery dataset to continuously stream Gmail System Logs, enabling SQL queries across historical log data spanning multiple years.

---

### Q9: How do you configure a split delivery setup where some users use Google Workspace and others use a legacy mail server?
**Detailed Technical Answer:**
1. **Set MX Records**: Point public DNS MX records to Google (`1 SMTP.GOOGLE.COM`).
2. **Add Legacy Host**: Go to **Gmail $\rightarrow$ Hosts $\rightarrow$ Add Route**. Name: `Legacy Exchange Host`, IP/FQDN: `mail.company.com:25`, Security: **Require TLS**.
3. **Configure Default Routing**: Go to **Gmail $\rightarrow$ Routing $\rightarrow$ Default Routing $\rightarrow$ Add Rule**. Scope: **Inbound**. If recipient is unrecognized in Directory $\rightarrow$ **Modify message $\rightarrow$ Change route $\rightarrow$ Legacy Exchange Host**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you prevent mail loops (`554 5.4.14 Hop count exceeded`) in a split delivery architecture?"*
* **Senior Engineer Answer**: Ensure the legacy Exchange server does NOT route unrecognized mail back to Google. Exchange must be authoritative for its local mailboxes only, or use specific mail contacts (`user@gsuite.company.com`) to route outbound Google mail back without creating circular loops.

---

### Q10: Explain the difference between SPF, DKIM, and DMARC. What does each one protect against?
**Detailed Technical Answer:**
* **SPF**: Validates sender IP against authorized DNS record (`v=spf1 include:_spf.google.com ~all`). Protects against **IP Spoofing**.
* **DKIM**: Adds cryptographic signature (`DKIM-Signature`) using 2048-bit RSA keys. Protects against **Message Tampering in Transit**.
* **DMARC**: Enforces domain alignment between `From:` header and SPF/DKIM domains. Dictates policy (`p=none/quarantine/reject`). Protects against **Exact Domain Spoofing & BEC**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can an email pass both SPF and DKIM checks but still FAIL DMARC validation?"*
* **Senior Engineer Answer**: **Yes.** DMARC requires **Domain Alignment**. If an attacker sends from a server (`attacker.com`) with valid SPF and DKIM for `attacker.com`, but sets the visible header to `From: CEO <ceo@company.com>`, DMARC compares `company.com` against `attacker.com`. Because domains do not align, **DMARC FAILS**.

---

### Q11: A user's emails are landing in external recipients' spam folders. What are the first 5 things you check?
**Detailed Technical Answer:**
1. **SPF Record**: Check DNS via `dig TXT company.com`. Ensure `include:_spf.google.com` exists and DNS lookups $\le 10$.
2. **DKIM Signature**: Verify 2048-bit DKIM key is active in Admin Console and TXT record matches `google._domainkey`.
3. **DMARC Alignment**: Confirm DMARC TXT record exists and `From:` domain matches SPF/DKIM domains.
4. **Postmaster Tools Reputation**: Inspect Postmaster Tools for Domain Reputation (High/Medium/Low/Bad) and Spam Rate (<0.10%).
5. **Blacklists & Content**: Check IP/domain against Spamhaus/MXToolbox blocklists, and check body content for spam trigger words.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"If domain reputation drops to 'Bad' on Postmaster Tools, what immediate steps do you take?"*
* **Senior Engineer Answer**: 1. Audit Email Log Search for compromised accounts sending bulk outbound spam. 2. Enforce strict outbound rate limits and block unauthenticated SMTP relay scripts. 3. Stop emailing unverified lead lists and enforce double opt-in. 4. Move DMARC to `p=reject`. 5. Gradually warm outbound sending volume over 2-4 weeks until reputation metrics recover.

---

### Q12: What is DKIM key rotation and how do you perform it in Google Workspace?
**Detailed Technical Answer:**
1. Open **Gmail $\rightarrow$ Authenticate email $\rightarrow$ Select domain $\rightarrow$ Generate New Record**.
2. Select 2048-bit key length and enter a new selector prefix (e.g., `google2026`).
3. Add generated TXT record (`google2026._domainkey`) to public DNS.
4. Wait 24-48 hours for DNS propagation, then click **Start Authentication** in Admin Console.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens to emails sent JUST BEFORE clicking 'Start Authentication'?"*
* **Senior Engineer Answer**: They deliver normally. Using a new selector prefix (`google2026`), the old selector (`google._domainkey`) remains active in DNS. Receiving MTAs fetch the specific selector declared inside the message header (`s=google` vs `s=google2026`). Keep the old DNS TXT record online for 7 days post-rotation before deleting.

---

### Q13: How do you configure an inbound email gateway in Google Workspace?
**Detailed Technical Answer:**
1. Open **Gmail $\rightarrow$ Spam, Phishing, and Malware $\rightarrow$ Inbound gateway**.
2. Enter Gateway Egress IP CIDR ranges (e.g., Proofpoint/Mimecast IPs).
3. Check **Automatically detect external IP** (extracts client IP from `X-Forwarded-For` header).
4. Check **Require TLS for connections from gateway IPs**.
5. Check **Reject messages that are not from gateway IPs**.
6. Save policy.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens if an internal user emails another internal user in the same domain after configuring an Inbound Gateway?"*
* **Senior Engineer Answer**: Internal mail between users in the same domain delivers natively within Google's internal transport layer without routing out to the internet or third-party gateway. The "Reject messages not from gateway IPs" policy applies strictly to inbound SMTP connections arriving from external internet IPs at Google's public border MX servers.

---

### Q14: What is the difference between a Gmail blacklist, blocked sender, and a content compliance block?
**Detailed Technical Answer:**
* **Blacklist (IP Block)**: Transport layer block based on global threat intelligence (Spamhaus/Google Reputation). Drops SMTP connections before message data transmission.
* **Blocked Sender**: Admin/user rule sending emails from specific addresses/domains (`spammer@badsite.com`) directly to Spam or Trash.
* **Content Compliance Block**: Policy inspection rule evaluating regex, file attachments, or DLP attributes to reject or quarantine messages regardless of sender reputation.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can an end-user's personal 'Blocked Sender' list override an Admin's 'Email Whitelist'?"*
* **Senior Engineer Answer**: **No.** Admin Console policies take absolute precedence over end-user personal settings. Adding a domain or IP to the Admin Console **Email Whitelist** or **Approved Sender List** (with *Bypass Spam Filters* enabled) forces inbox delivery, overriding end-user personal blocked sender lists.

---

### Q15: How do you configure email confidential mode? What are its limitations?
**Detailed Technical Answer:**
* **Configuration**: Enable under **Gmail $\rightarrow$ User settings $\rightarrow$ Confidential mode**.
* **Features**: Prevents forwarding, copying, printing, downloading; sets expiration dates; requires SMS passcode.
* **Limitations**: Does NOT prevent screenshots or physical photos; external non-Gmail recipients receive web links (`https://mail.google.com/confidential...`); Google Vault still archives confidential messages for legal discovery.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can an admin automatically enforce Confidential Mode for all outbound emails containing credit card numbers?"*
* **Senior Engineer Answer**: Direct automatic triggering of Confidential Mode via Content Compliance is not natively supported. However, you can configure a **DLP Rule** detecting Credit Card Numbers set to **Block message** with a custom bounce notice: *"Credit card numbers detected. Please re-send this message using Gmail Confidential Mode."*

---

### Q16: A user has enabled auto-forwarding of all emails to a personal Gmail account. How do you detect and disable this?
**Detailed Technical Answer:**
1. **Audit & Detect**: Check Admin Audit Log for event `Email forwarding address added`, or run GAM:
   ```bash
   gam all users print forwarding > forwarding_audit.csv
   ```
2. **Disable Tenant-Wide**: Go to **Gmail $\rightarrow$ End User Access $\rightarrow$ Automatic forwarding** $\rightarrow$ Uncheck **Automatic forwarding** (Set to **Disabled**).
3. **Purge via GAM**:
   ```bash
   gam user alex.smith@company.com delete forwardingaddress personal@gmail.com
   ```

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What if the user created a custom Gmail FILTER ('has:attachment $\rightarrow$ Forward to personal@gmail.com') instead of using main Auto-Forwarding settings?"*
* **Senior Engineer Answer**: Disabling **Automatic forwarding** in **End User Access** disables both native account forwarding AND filter-based auto-forwarding to external domains. To audit and purge user filters specifically:
```bash
gam all users print filters > user_filters.csv
gam user alex.smith@company.com delete filter id <filter_id>
```

---

### Q17: How do you set up email archiving (Vault) for all users in a specific OU?
**Detailed Technical Answer:**
1. Log into **Google Vault** (`vault.google.com`).
2. Go to **Retention $\rightarrow$ Custom Rules $\rightarrow$ Create**.
3. Service: **Gmail** $\rightarrow$ Scope: **Specific organizational unit** (`/Finance`).
4. Retention Period: **Fixed time period** (`2555` days for 7 years).
5. Action: **Keep only messages sent/received within retention period and purge expired messages**.
6. Save rule.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What is the difference between a Vault Retention Rule and a Vault Litigation Hold?"*
* **Senior Engineer Answer**: Retention Rules govern lifecycle cleanup (purging data older than X years). Litigation Holds are explicit legal overrides placed on specific users/OUs during audits/lawsuits. **Litigation Holds completely override Retention Rules.** Even if a retention rule dictates deleting a 7-year-old email, a Litigation Hold preserves data indefinitely until removed.

---

### Q18: What is Postmaster Tools and what information does it provide that the Admin Console doesn't?
**Detailed Technical Answer:**
**Google Postmaster Tools** (`postmaster.google.com`) provides machine-learning delivery telemetry for domain owners sending to Gmail recipients.

#### Unique Metrics Provided:
* **Domain Reputation**: High, Medium, Low, or Bad classification.
* **IP Reputation**: Reputation score of egress SMTP server IPs.
* **User-Reported Spam Rate**: Percentage of emails marked "Report Spam" by users (Must remain <0.10%).
* **Authentication Metrics**: Graph metrics for SPF, DKIM, and DMARC pass/fail rates.
* **TLS Rate & Delivery Errors**: Encryption percentages and connection drops.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you authorize your IT team to view Postmaster Tools data for `company.com`?"*
* **Senior Engineer Answer**: 1. Log into `postmaster.google.com`, 2. Add domain `company.com`, 3. Copy generated TXT verification code (`google-site-verification=...`), 4. Publish TXT code in public DNS, 5. Click Verify. Once verified, grant view permissions to IT admin accounts via Postmaster Tools User Management.

---

### Q19: How would you configure Gmail to reject all emails that fail DMARC alignment?
**Detailed Technical Answer:**
1. **Public DNS Record**: Publish TXT record for `_dmarc.company.com`:
   ```text
   v=DMARC1; p=reject; rua=mailto:dmarc-reports@company.com; pct=100
   ```
2. **Admin Console Enforcement**: Go to **Gmail $\rightarrow$ Safety $\rightarrow$ Spoofing and authentication $\rightarrow$ Unauthenticated email** $\rightarrow$ Enable **Protect against unauthenticated senders** $\rightarrow$ Action: **Reject message**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens if a legitimate third-party vendor (Mailchimp/HubSpot) sends email on your behalf after setting `p=reject` without configuring DKIM/SPF?"*
* **Senior Engineer Answer**: Emails are **hard-bounced and rejected** (`550 5.7.26`). To prevent disruption, admins MUST audit DMARC `rua` XML reports during a `p=none` monitoring phase to identify third-party senders, publish SPF `include` mechanisms, and align 2048-bit DKIM keys BEFORE setting `p=reject`.

---

### Q20: Explain the difference between p=none, p=quarantine, and p=reject in a DMARC record. When would you use each?
**Detailed Technical Answer:**
* `p=none` (**Monitoring**): Collects aggregate reports without affecting mail delivery. Used during initial deployment (0-90 days).
* `p=quarantine` (**Soft Enforcement**): Routes unauthenticated mail failing DMARC to Spam or Admin Quarantine. Used when >95% of email streams are authenticated.
* `p=reject` (**Hard Enforcement**): Rejects unauthenticated mail at the SMTP gateway (`550 5.7.26`). Used in mature zero-trust environments.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What does the `pct=20` tag do when added to a DMARC policy (`p=quarantine; pct=20`)?"*
* **Senior Engineer Answer**: The `pct=` tag applies the policy action (`quarantine`/`reject`) to only 20% of unauthenticated messages, while the remaining 80% fallback to `p=none`. Admins use `pct=10`, `pct=25`, `pct=50` to execute a gradual rollout across large enterprises before enforcing 100% policy application (`pct=100`).

---

## 🛡️ SECTION 2: Security & Context-Aware Access (Q21–Q40)

### Q21: What is Context-Aware Access (CAA) and how does it implement Zero Trust in Google Workspace?
**Detailed Technical Answer:**
Context-Aware Access (CAA) evaluates dynamic contextual attributes (device compliance, IP CIDR, geofence, Chrome management state) using Common Expression Language (CEL) at request time to grant or deny access to specific Google apps without relying on static perimeter firewalls.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you prevent locking out all administrators when deploying a newly created CAA policy?"*
* **Senior Engineer Answer**: 1. Assign Access Levels to a test OU (`/IT-Pilots`) first. 2. Use **Monitor Mode** to log matches to Audit logs without actively blocking traffic. 3. Explicitly exclude the `/Admins` OU holding emergency break-glass accounts from restrictive endpoint device Access Levels.

---

### Q22: A user is working from a personal laptop. How do you ensure they can access Gmail but NOT Google Drive?
**Detailed Technical Answer:**
1. Create Access Level `Require_Managed_Device` in **Security $\rightarrow$ Context-Aware Access**:
   ```cel
   device.chrome.management_state == ChromeManagementState.CHROME_MANAGEMENT_STATE_BROWSER_MANAGED &&
   device.is_managed_device == true
   ```
2. In **Assign Access Levels**, select **Google Drive and Docs** $\rightarrow$ Assign `Require_Managed_Device`.
3. Select **Gmail** $\rightarrow$ Leave **Unassigned**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can the user bypass this restriction by opening Google Docs links embedded inside Gmail emails on their personal laptop?"*
* **Senior Engineer Answer**: **No.** Google Docs, Sheets, Slides, and Forms are components of the **Google Drive and Docs** application service container in CAA. Opening a Google Doc link (`docs.google.com/...`) triggers CAA evaluation against the Drive app policy, blocking access on unmanaged devices regardless of link origin.

---

### Q23: What is Endpoint Verification and how is it different from Mobile Device Management (MDM)?
**Detailed Technical Answer:**
* **Endpoint Verification**: Passive Chrome extension gathering OS, encryption status (`ENCRYPTED`), screen lock, and serial number for CAA CEL evaluation.
* **MDM (Advanced Management)**: Active control agent enforcing passcode profiles, app deployment, remote wipe, and device payload policies.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Is Endpoint Verification alone sufficient to remotely wipe a compromised Windows laptop?"*
* **Senior Engineer Answer**: **No.** Endpoint Verification is a read-only telemetry extension. To execute remote device wipes on Windows desktop assets, the device must be enrolled in **GCPW (Google Credential Provider for Windows)** with Advanced Desktop Management enabled, or managed via an enterprise MDM like Microsoft Intune.

---

### Q24: How do you block access to Google Workspace from a specific country or region?
**Detailed Technical Answer:**
Create a CAA Access Level in Advanced Mode using CEL:
```cel
!origin.region_code.in(["RU", "CN", "IR", "KP", "SY"])
```
Assign this Access Level across all Core Google Apps for target OUs.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What if a legitimate employee travels to a blocked country on business? How do you grant emergency access safely?"*
* **Senior Engineer Answer**: 1. Create group `geofence-exceptions@company.com`. 2. Update CEL rule:
```cel
!origin.region_code.in(["RU", "CN", "IR", "KP"]) ||
request.auth.claims.groups.contains("geofence-exceptions@company.com")
```
3. Add user to exception group with 7-day expiration time via GAM, and require FIDO2 Hardware Keys.

---

### Q25: What is the Alert Center in Google Workspace? Name 5 types of alerts you would configure.
**Detailed Technical Answer:**
Centralized threat dashboard in Admin Console.
* **5 Critical Alerts**: 1. Super Admin Password Reset, 2. Suspicious Login Activity, 3. User Suspended for Abuse, 4. DLP Rule Violation, 5. High-Risk OAuth App Grant.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you automate sending Alert Center notifications directly into a PagerDuty or Slack channel?"*
* **Senior Engineer Answer**: Navigate to **Security $\rightarrow$ Alerts $\rightarrow$ Manage Rules**, edit target rules, and add the webhook email address for PagerDuty or Slack (`#soc-alerts@company.pagerduty.com`). Alternatively, use the **Google Workspace Alert Center REST API** to stream alert payloads into Splunk/Microsoft Sentinel.

---

### Q26: How do you configure a CAA policy to allow access only from corporate-managed devices?
**Detailed Technical Answer:**
Create Access Level in **Security $\rightarrow$ Context-Aware Access**:
```cel
device.is_managed_device == true &&
device.encryption_status == EncryptionStatus.ENCRYPTED &&
device.screen_lock_enabled == true
```
Assign level across All Core Apps for target OUs.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How does Google Workspace verify that a Windows or Mac device is 'Corporate Managed'?"*
* **Senior Engineer Answer**: Verified through **Device Certificate Binding** and **Endpoint Verification**. The device must be enrolled in Google Endpoint Management (holding a certificate pushed via Jamf/Intune), enrolled in GCPW, or imported into the Admin Console Company-Owned Devices inventory database via serial number matching.

---

### Q27: What is BeyondCorp and how does Google Workspace implement its principles?
**Detailed Technical Answer:**
BeyondCorp is Google's Zero Trust architecture shifting access control from network perimeters to individual users and device security postures evaluated per request via Context-Aware Access, Identity-Aware Proxy (IAP), and FIDO2 2SV.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How does BeyondCorp handle traditional corporate office networks?"*
* **Senior Engineer Answer**: BeyondCorp treats local corporate office networks as completely untrusted public internet connections. Employees inside corporate headquarters undergo the exact same device posture verification, 2SV authentication, and CAA CEL evaluations as remote employees working from home or coffee shops.

---

### Q28: A Super Admin account has been compromised. What are your first 5 steps?
**Detailed Technical Answer:**
1. **Execute Emergency Session Signout & Lock Account**:
   ```bash
   gam user compromised.admin@company.com signout
   ```
   Suspend account in Admin Console.
2. **Revoke 2SV & Reset Password**: Remove 2SV security keys, backup codes, recovery phone/email, and reset password.
3. **Audit Admin Activity Logs**: Inspect **Admin Audit Log** for unauthorized role grants, secondary domain additions, or modified SAML SSO profiles.
4. **Revoke OAuth Tokens**: Run `gam user compromised.admin@company.com delete tokens`.
5. **Isolate & Report**: Inspect secondary recovery email/phone settings for persistence backdoors and notify SOC.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you prevent a compromised Super Admin from deleting audit logs to hide their tracks?"*
* **Senior Engineer Answer**: Google Workspace Audit Logs are append-only and cryptographically immutable; even Super Admins cannot edit, alter, or delete native Admin Audit logs or Security Investigation Tool records. Furthermore, continuous streaming to **BigQuery Log Export** ensures logs are stored in a separate GCP project governed by strict IAM permissions.

---

### Q29: How do you enforce 2-Step Verification (2SV) for specific OUs while keeping it optional for others?
**Detailed Technical Answer:**
1. Go to **Security $\rightarrow$ Authentication $\rightarrow$ 2-Step Verification**.
2. Select target OU (`/Finance`) $\rightarrow$ Set **Enforcement $\rightarrow$ Turn on enforcement**. Select enforcement date and allowed methods (Security Key / Google Prompt).
3. Select exception OU (`/Contractors`) $\rightarrow$ Set **Enforcement $\rightarrow$ Turn off enforcement**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens when a new hire logs in for the first time in an OU where 2SV is enforced?"*
* **Senior Engineer Answer**: Configure a **New user enrollment period** (Grace Period) of 1 to 7 days under 2SV settings. This grants new hires a temporary window to log in with an initial password and complete 2SV registration. Alternatively, generate a 24-hour **Backup Verification Code** via Admin Console to satisfy 2SV on first login.

---

### Q30: What is the difference between basic and advanced mobile device management in Google Workspace?
**Detailed Technical Answer:**
Basic MDM requires no client agent and enforces basic screen lock/passcode and selective account wipe. Advanced MDM requires Device Policy App / Android Work Profile, enforcing work profile container isolation, full device wipe, app force-installation, and CAA device compliance rules.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens to personal photos on a BYOD Android phone when an admin executes an 'Account Wipe' in Advanced MDM?"*
* **Senior Engineer Answer**: An **Account Wipe** selectively purges only the corporate Work Profile container (Gmail, Drive, corporate apps, and enterprise encryption keys). All personal data, personal photos, personal apps, and SMS messages residing in the personal profile remain completely untouched.

---

### Q31: How do you perform a security audit of all third-party apps connected to Google Workspace via OAuth?
**Detailed Technical Answer:**
Go to **Security $\rightarrow$ Access and data control $\rightarrow$ API Controls $\rightarrow$ App Access Control**. Review third-party apps, requested OAuth scopes, user counts, and risk scores. Change untrusted high-risk apps (`gmail.readonly`, `drive`) to **Blocked**, revoking all current bearer tokens.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you prevent users from installing unapproved third-party Marketplace apps in the future?"*
* **Senior Engineer Answer**: Go to **Apps $\rightarrow$ Google Workspace Marketplace apps $\rightarrow$ Settings**. Under **User installs**, select **Do not allow users to install any app from Google Workspace Marketplace** (or *Allow users to install only allowed apps from Marketplace*).

---

### Q32: A user's account shows login activity from two different countries within 30 minutes. How do you respond?
**Detailed Technical Answer:**
1. Alert Center triggers `Suspicious Login` (Impossible Travel).
2. Execute immediate containment:
   ```bash
   gam user alex.smith@company.com signout
   gam user alex.smith@company.com suspend
   ```
3. Audit Login Audit Log comparing IPs, ASNs, User-Agent strings, and 2SV verification methods.
4. Purge OAuth tokens (`gam user alex.smith@company.com delete tokens`), reset password, un-suspend account, and enforce FIDO2 re-enrollment.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What if the user claims they were using a legitimate corporate VPN that routed traffic through an overseas egress node?"*
* **Senior Engineer Answer**: Verify the overseas IP address against your corporate VPN provider's known egress IP CIDR blocks. If the IP belongs to your corporate VPN pool, add that CIDR range to **Security $\rightarrow$ Access and data control $\rightarrow$ Whitelisted IP ranges** or update CAA CEL policy to treat corporate VPN egress IPs as trusted origin IPs.

---

### Q33: What is Google Workspace's approach to data encryption at rest and in transit?
**Detailed Technical Answer:**
* **In Transit**: Encrypted using TLS 1.2/1.3 with Perfect Forward Secrecy (PFS). SMTP connections enforce MTA-STS.
* **At Rest**: Files uploaded to Drive or Gmail are split into sub-file chunks. Each chunk is encrypted with individual AES-256/128 keys, and master keys are stored in FIPS 140-2 validated KMS. Support for Client-Side Encryption (CSE).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What is Client-Side Encryption (CSE) and how does it differ from standard Google encryption at rest?"*
* **Senior Engineer Answer**: With standard encryption, Google manages the KMS encryption keys. With **Client-Side Encryption (CSE)**, data is encrypted on the client browser using customer-managed cryptographic keys (hosted on Azure Key Vault, HashiCorp Vault, or Thales HSM) *before* data streams to Google. Google servers store only encrypted blobs and have zero technical capability to decrypt customer files.

---

### Q34: How do you configure an activity-based alert rule for when a user downloads more than 100 Drive files in one hour?
**Detailed Technical Answer:**
1. Go to **Security $\rightarrow$ Security Center $\rightarrow$ Investigation Tool**.
2. Data Source: **Drive log events** $\rightarrow$ Condition: **Event == Download**.
3. Grouping & Thresholds: Click **Create Rule** $\rightarrow$ Threshold: **Download Count > 100** within **1 Hour** grouped by **Actor**.
4. Actions: Check **Send to Alert Center** and email SOC admins. Save rule.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you automatically suspend a user account when this mass download alert rule triggers?"*
* **Senior Engineer Answer**: Integrate the Investigation Tool alert with a **GCP Cloud Function / Lambda Webhook**. When the threshold triggers, the alert payload posts to the webhook script, which authenticates via Admin SDK Directory API Service Account and executes account suspension (`users.update(suspended=True)`).

---

### Q35: What is the purpose of Security Investigation Tool in Google Workspace? How is it different from the Audit Log?
**Detailed Technical Answer:**
* **Audit Log**: Read-only static log viewer listing historical events.
* **Security Investigation Tool**: Interactive SOAR workbench allowing cross-log querying, automated alert rule creation, and direct bulk remediation actions (deleting phishing emails, revoking OAuth tokens, suspending accounts directly from query results).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can you save search queries in the Investigation Tool for fellow SOC analysts to re-use?"*
* **Senior Engineer Answer**: **Yes.** Click **Save** in the top right of the Investigation Tool query builder, name the search (e.g., `Daily_External_Drive_Shares`), and set sharing permissions to **Shared with domain admins**.

---

### Q36: How do you set up a Hardware Security Key (FIDO2) enforcement policy for Admin accounts only?
**Detailed Technical Answer:**
1. Select `/Admins` OU in **Security $\rightarrow$ Authentication $\rightarrow$ 2-Step Verification**.
2. Allowed 2SV methods: Select **Only Security Key** (FIDO2 WebAuthn keys like YubiKey/Titan).
3. Enforcement: Select **Turn on enforcement**. Save policy.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens if an admin loses their YubiKey and cannot satisfy the FIDO2 prompt?"*
* **Senior Engineer Answer**: Another Super Admin generates a temporary 24-hour **Backup Verification Code** via Admin Console (**Users $\rightarrow$ Admin Account $\rightarrow$ Security $\rightarrow$ Get Backup Verification Codes**). The locked-out admin enters a backup code to satisfy 2SV and register a replacement FIDO2 key.

---

### Q37: What is an Access Transparency log and who can view it?
**Detailed Technical Answer:**
Access Transparency logs capture every instance where Google staff or support engineers access customer data at rest during support operations. Accessible to Super Admins holding Enterprise Plus licenses under **Reporting $\rightarrow$ Audit $\rightarrow$ Access Transparency**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What information is captured inside an Access Transparency log entry?"*
* **Senior Engineer Answer**: 1. Target resource reference (e.g., specific Gmail message ID or Drive file ID), 2. Exact UTC timestamp, 3. Google staff member justification code (e.g., `CASE_NUMBER_12345`), 4. Action executed (e.g., `view_file_metadata`), 5. Google office location of the engineer.

---

### Q38: How do you configure Google Workspace to block access from Tor exit nodes or anonymous VPN IPs?
**Detailed Technical Answer:**
Create a CAA Access Level in Advanced Mode matching threat intelligence IP ranges:
```cel
!inIpRange(origin.ip, ["185.220.100.0/22", "198.51.100.0/24"])
```
Alternatively, route unmanaged web traffic through **Netskope CASB Reverse Proxy** which maintains real-time threat intelligence feeds dropping Tor exit nodes and anonymous VPN endpoints.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Why is blocking Tor exit nodes critical for enterprise Google Workspace security?"*
* **Senior Engineer Answer**: Tor exit nodes obscure attacker origin IPs, allowing cybercriminals to execute credential stuffing attacks, password spraying, and session hijacking without exposing their real geographic IP or triggering location-based security alerts.

---

### Q39: What is the difference between a Super Admin, a Delegated Admin, and a read-only Admin?
**Detailed Technical Answer:**
* **Super Admin**: Unrestricted root privileges across all domain configurations, user data, security policies, and domain setups.
* **Delegated Admin**: Role-Based Access Control (RBAC) privileges scoped to specific tasks/OUs (e.g., *User Management Admin for /Sales OU*).
* **Read-Only Admin**: Access restricted to viewing audit logs, reports, and settings without write/modify permissions.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Why is assigning Super Admin roles to day-to-day IT Helpdesk staff considered a major security violation?"*
* **Senior Engineer Answer**: Super Admin accounts bypass OU restrictions, access all Google Vault legal data, modify SAML SSO setups, and create backdoor accounts. The Principle of Least Privilege mandates assigning Helpdesk staff custom **Delegated Admin Roles** scoped strictly to user password resets and group management for specific child OUs.

---

### Q40: How do you configure an automatic response to suspend a user account when suspicious activity is detected?
**Detailed Technical Answer:**
Configure an Alert Center Rule (e.g., *DLP Exfiltration Threshold Exceeded*) with automated remediation triggers, or subscribe to Google Workspace System Event Webhooks via Admin SDK Push Notifications API. Send payloads to a GCP Cloud Function running GAM commands:
```bash
gam user <email> update suspended true
```

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you prevent automated suspension scripts from accidentally suspending C-level executive accounts during false-positive alerts?"*
* **Senior Engineer Answer**: Add an explicit OU check or attribute filter inside the automation script logic:
```python
if user_ou in ["/Executives", "/Admins"]:
    send_urgent_slack_alert(user) # Notify SOC without suspending
else:
    suspend_user_via_gam(user) # Automate suspension
```

---

## 👥 SECTION 3: User & OU Management (Q41–Q55)

### Q41: You need to onboard 500 users at once. What are the different methods available and which would you recommend?
**Detailed Technical Answer:**
* **Methods**: 1. Admin Console CSV Import, 2. GAM CLI Script (`gam csv users.csv gam create user ~Email firstname ~FirstName lastname ~LastName ou ~OU`), 3. SCIM Provisioning via IdP (Entra ID/OneLogin), 4. Directory API Python SDK script.
* **Recommendation**: **SCIM via IdP** for automated enterprise lifecycle governance; **GAM CLI** for one-time offline manual migrations.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you assign licenses to these 500 new users automatically during onboarding?"*
* **Senior Engineer Answer**: Enable **Automatic License Assignment** under **Billing $\rightarrow$ License settings**. Select target OU (`/Employees`) and choose the default license SKU (e.g., *Google Workspace Enterprise Plus*). Any user created in or moved to `/Employees` is automatically provisioned a license.

---

### Q42: What is the difference between suspending a user and deleting a user in Google Workspace? What happens to their data?
**Detailed Technical Answer:**
* **Suspending**: Disables login immediately; preserves all data (Drive, Gmail, Vault holds); account continues consuming a paid license unless reassigned to Cloud Identity Free.
* **Deleting**: Purges account identity; data is queued for permanent deletion after 20 days unless transferred to a successor during deletion workflow.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"If a suspended user account is assigned a Cloud Identity Free license, can they still receive emails sent to their Gmail address?"*
* **Senior Engineer Answer**: **No.** Cloud Identity Free licenses do NOT include Gmail mailbox services. Moving a suspended user to Cloud Identity Free removes their Gmail license, causing incoming emails sent to that address to bounce back to senders (`550 5.1.1 User unknown` or `550 5.2.1 Mailbox disabled`).

---

### Q43: How do you transfer all Drive content from a departing employee to their manager?
**Detailed Technical Answer:**
* **Admin Console**: Go to **Users $\rightarrow$ Delete User $\rightarrow$ Transfer Data**, check **Drive and Docs**, enter Manager Email $\rightarrow$ Click **Assign & Delete**.
* **GAM CLI (Pre-Deletion Transfer)**:
  ```bash
  gam user leaver@company.com transfer drive manager@company.com keep-shares
  ```

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Where does the transferred Drive content appear inside the Manager's Google Drive?"*
* **Senior Engineer Answer**: Google Drive creates a dedicated root folder in the Manager's "My Drive" named `leaver@company.com (Transferred Data)`. Inside this folder, the entire directory tree and file hierarchy owned by the departing employee is preserved intact, along with existing file sharing permissions (`keep-shares`).

---

### Q44: A user is being offboarded. List all the steps you would take to properly offboard them in Google Workspace.
**Detailed Technical Answer:**
1. Reset password & execute global signout (`gam user <email> signout`).
2. Revoke OAuth tokens and 2SV security keys.
3. Transfer Drive files and Calendar events to manager.
4. Move user to `/_Leavers` OU and set status to **Suspended**.
5. Reclaim Enterprise license (assign Cloud Identity Free license).
6. Verify Google Vault Litigation Hold status.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens to secondary Google Services like YouTube brand channels or Google Analytics accounts owned by the offboarded user?"*
* **Senior Engineer Answer**: Deleting or suspending the user without transferring secondary service ownership causes loss of admin control over YouTube channels, Google Ads accounts, and Google Analytics properties. Offboarding SOPs must mandate transferring Brand Channel ownership and GCP Project IAM roles to a shared administrative group before suspending the primary user account.

---

### Q45: How do you configure a catch-all email address in Google Workspace?
**Detailed Technical Answer:**
Go to **Apps $\rightarrow$ Google Workspace $\rightarrow$ Gmail $\rightarrow$ Routing $\rightarrow$ Catch-all address**. Select **Forward unrecognized emails to:** `catchall@company.com` or set to **Bounce**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can Catch-All rules cause false-positive spam delivery issues?"*
* **Senior Engineer Answer**: Yes. Spammers harvest domains and blast emails to thousands of guessed addresses. Catch-all inboxes ingest all directory harvesting traffic, increasing spam scores. Setting Catch-All to **Bounce** (`550 User Unknown`) is the security best practice.

---

### Q46: What is the difference between a Google Group and a Shared Mailbox in Google Workspace?
**Detailed Technical Answer:**
* **Google Group**: Free collaborative inbox/distribution list entity without a paid license.
* **Shared Mailbox**: Licensed user account shared among multiple users via delegated inbox access.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"When should you choose a Delegated Shared Mailbox over a Collaborative Google Group?"*
* **Senior Engineer Answer**: Use a **Delegated Shared Mailbox** when users require full Gmail features (sent folder synchronization, custom signatures, drafts, labels, and mobile app access). Use a **Collaborative Group** when you need simple ticket assignment without consuming a paid Workspace license.

---

### Q47: How do you configure a user alias? Can an alias receive calendar invites?
**Detailed Technical Answer:**
Added in **Users $\rightarrow$ User Details $\rightarrow$ Alternate email addresses**. Yes, aliases natively receive calendar invites directly in the primary inbox.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can a user SEND calendar invites using their email alias?"*
* **Senior Engineer Answer**: By default, calendar invites sent from Google Calendar display the primary email address. To send invites from an alias, the user must add the alias under **Gmail Settings $\rightarrow$ Accounts $\rightarrow$ Send mail as**, and select the alias address when sending calendar responses or event creations.

---

### Q48: How do you bulk update user attributes (department, job title, phone number) using the Admin Console?
**Detailed Technical Answer:**
Go to **Users $\rightarrow$ Bulk update users** $\rightarrow$ Download CSV $\rightarrow$ Update columns (`Department`, `Title`) $\rightarrow$ Upload CSV.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens if you leave the 'Password' column blank in the bulk CSV update file?"*
* **Senior Engineer Answer**: Leaving the Password column blank in an update CSV instructs Google Workspace to leave the existing user passwords untouched. It will only update the modified metadata attributes (`Department`, `Title`, `Phone`).

---

### Q49: What is the difference between an Organizational Unit (OU) and a Group in Google Workspace? When would you use each for policy application?
**Detailed Technical Answer:**
* **OU**: Hierarchical organizational tree structure defining baseline setting inheritance.
* **Group**: Cross-OU entity used for flexible policy overrides (CAA rules, App access).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"If an OU setting BLOCKS YouTube access, but a Group setting ALLOWS YouTube access, which wins for a user in both?"*
* **Senior Engineer Answer**: **Group-based policy settings take precedence over OU settings.** Group-based policy overrides allow granting specific application access or CAA privileges to cross-functional group members regardless of their inherited OU restrictions.

---

### Q50: A manager wants to see their team member's calendar. How do you configure Calendar delegation?
**Detailed Technical Answer:**
In Calendar Settings $\rightarrow$ **Share with specific people** $\rightarrow$ Add Manager email $\rightarrow$ Grant **Make changes and manage sharing**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How can an admin configure calendar delegation globally via GAM without asking the user to log into their web browser?"*
* **Senior Engineer Answer**: Use GAM CLI:
```bash
gam calendar user@company.com add editor manager@company.com
```

---

### Q51: How do you configure a "no-reply" email address that can send but not receive emails?
**Detailed Technical Answer:**
Create `noreply@company.com` group/user, add Routing Rule to reject inbound mail with custom auto-response.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you allow automated web applications (like WordPress/Jira) to send mail from `noreply@company.com` without hardcoding passwords?"*
* **Senior Engineer Answer**: Configure **SMTP Relay Service** in **Gmail $\rightarrow$ Routing $\rightarrow$ SMTP relay service**. Whitelist the application server's static egress IP address and require TLS. The application sends mail through `smtp-relay.gmail.com` on port 587/465 without password authentication.

---

### Q52: What are the different types of Google Groups (mailing list, forum, collaborative inbox, Q&A forum)?
**Detailed Technical Answer:**
Mailing List (Distribution), Collaborative Inbox (Topic tracking), Web Forum (Discussions), Q&A Forum (Question voting).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you prevent external spammers from emailing internal Google Groups?"*
* **Senior Engineer Answer**: Open Group Settings $\rightarrow$ **Access permissions $\rightarrow$ Posting permissions**. Change posting permission from *Public/Anyone on the web* to **Entire organization** or **Group members only**.

---

### Q53: How do you restore a deleted user and their data? What is the restore time limit?
**Detailed Technical Answer:**
Go to **Users $\rightarrow$ Recently deleted $\rightarrow$ Select User $\rightarrow$ Restore**. Time limit: **20 days** post-deletion.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens if you attempt to restore a deleted user on day 21?"*
* **Senior Engineer Answer**: On day 21, the user account and associated data (Gmail, Drive files, Calendar events) are permanently purged from Google Workspace storage. Recovery is impossible unless the data was preserved in **Google Vault** prior to account deletion.

---

### Q54: How do you configure a shared drive and set member permissions (Viewer, Commenter, Contributor, Content Manager, Manager)?
**Detailed Technical Answer:**
Create Shared Drive $\rightarrow$ Add Members $\rightarrow$ Assign roles: Viewer (Read), Commenter (Comment), Contributor (Edit files), Content Manager (Manage files/folders), Manager (Admin).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can a 'Contributor' delete a subfolder inside a Shared Drive?"*
* **Senior Engineer Answer**: **No.** Contributors can create, edit, and upload files, but they CANNOT delete files or folders. Deleting files or moving folders requires **Content Manager** or **Manager** permissions.

---

### Q55: What is Directory Sharing and how do you control what user information is visible in the Global Address List (GAL)?
**Detailed Technical Answer:**
Configured in **Directory $\rightarrow$ Directory settings $\rightarrow$ Profile editing**. Controls custom field visibility across GAL.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you hide contractor accounts from appearing in the Global Address List (GAL) autocompletion?"*
* **Senior Engineer Answer**: Move contractor accounts to `/Contractors` OU. Go to **Directory $\rightarrow$ Directory settings $\rightarrow$ Sharing settings**, select `/Contractors` OU, and set **Directory sharing** to **Do not share** (or create a Custom Directory Visibility Map excluding contractors).

---

## 💾 SECTION 4: Google Drive & Data Protection (Q56–Q70)

### Q56: What is the difference between My Drive and Shared Drives from an admin governance perspective?
**Detailed Technical Answer:**
My Drive files are user-owned (lost if user deleted without transfer); Shared Drive files are organization-owned (persist indefinitely).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What is the 400,000 item limit on Shared Drives?"*
* **Senior Engineer Answer**: A single Shared Drive can hold a maximum of 400,000 total files, folders, and trashed items combined. Exceeding this limit prevents users from uploading or creating new files until items are permanently purged.

---

### Q57: How do you configure Drive to prevent users from downloading, printing, or copying files shared with them?
**Detailed Technical Answer:**
In file sharing settings, check **Disable options to download, print, and copy for commenters and viewers**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can IRM restrictions be enforced tenant-wide via policy?"*
* **Senior Engineer Answer**: Admins can enforce global Drive sharing restrictions under **Apps $\rightarrow$ Drive and Docs $\rightarrow$ Sharing settings**, preventing external users from downloading or copying shared content across designated OUs.

---

### Q58: What are Drive Trust Rules and how do they differ from standard sharing settings?
**Detailed Technical Answer:**
Trust Rules provide granular condition-based policies. Precedence rule: **Most restrictive setting always wins**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens when you enable 'Enforce Trust Rules Exclusively'?"*
* **Senior Engineer Answer**: Legacy Drive sharing settings are completely disabled and replaced by Trust Rules as the single authoritative policy engine for the domain.

---

### Q59: How do you find all files in your organization that have been shared publicly ("anyone with the link")?
**Detailed Technical Answer:**
Use Security Investigation Tool: **Drive log events $\rightarrow$ Visibility == Public on the web / Anyone with link**, or GAM CLI:
```bash
gam user all show fileaccess filter "visibility == 'anyoneWithLink'"
```

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you bulk-revoke public access for all returned files using GAM?"*
* **Senior Engineer Answer**:
```bash
gam user all delete driveaccess anyoneWithLink
```

---

### Q60: What is Google Vault and what types of data can it preserve, search, and export?
**Detailed Technical Answer:**
Governance tool preserving Gmail, Drive, Groups, Chat, and Meet recordings for legal compliance.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Does Google Vault preserve data in real-time if a user creates and immediately deletes a file within 5 seconds?"*
* **Senior Engineer Answer**: **Yes.** Vault operates at the backend storage layer. Even if a user creates, edits, and permanently deletes an email or Drive file immediately, Vault captures and preserves the data payload.

---

### Q61: How do you configure a Vault retention rule for all Gmail data for a specific OU for 7 years?
**Detailed Technical Answer:**
Vault $\rightarrow$ **Custom Rules $\rightarrow$ Create $\rightarrow$ Service: Gmail $\rightarrow$ OU: /Finance $\rightarrow$ Duration: 2555 Days $\rightarrow$ Action: Purge**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens to emails older than 7 years after saving this rule?"*
* **Senior Engineer Answer**: If the action is set to **Purge**, Vault automatically queued background jobs to permanently delete all matching emails older than 2555 days across the target OU.

---

### Q62: A legal hold has been placed on a specific user. How do you configure this in Google Vault?
**Detailed Technical Answer:**
Vault $\rightarrow$ **Matters $\rightarrow$ Create $\rightarrow$ Holds $\rightarrow$ Service: Gmail/Drive $\rightarrow$ User: `alex.smith@company.com`**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Does the targeted user know they have been placed under a Vault Legal Hold?"*
* **Senior Engineer Answer**: **No.** Vault Legal Holds are completely silent and transparent to end users. No notification or UI indicator is displayed inside the user's Gmail or Drive interface.

---

### Q63: What is the difference between a DLP Audit rule and a DLP Block rule?
**Detailed Technical Answer:**
Audit Rule logs silently; Block Rule prevents sharing and alerts user/SOC.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Why should you run new DLP rules in 'Audit Mode' before changing them to 'Block Mode'?"*
* **Senior Engineer Answer**: Running in Audit Mode allows admins to review false-positive match rates in audit logs and refine regex expressions before enforcing active blocks that could break legitimate business workflows.

---

### Q64: How would you configure DLP to detect and block sharing of credit card numbers in Google Drive?
**Detailed Technical Answer:**
DLP Rule $\rightarrow$ **Detector: Credit Card Number $\rightarrow$ Action: Block external sharing $\rightarrow$ Alert: SOC Notification**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Does DLP scan images and PDF scans for credit card numbers?"*
* **Senior Engineer Answer**: **Yes**, if **Optical Character Recognition (OCR)** is enabled in Drive DLP settings. OCR extracts text from image files (`.png`, `.jpg`) and scanned PDFs for evaluation.

---

### Q65: How do you configure Drive to prevent external sharing for all users except the Sales team?
**Detailed Technical Answer:**
Root OU Drive Sharing = **OFF**. Override `/Sales` OU Drive Sharing = **ON**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you restrict the Sales team to sharing externally ONLY with whitelisted partner domains?"*
* **Senior Engineer Answer**: In `/Sales` OU Drive Sharing settings, check **Whitelisted domains** and enter approved partner domain names (`partner.com`).

---

### Q66: What are Google Drive Labels and how are they used in DLP policies?
**Detailed Technical Answer:**
Taxonomy tags (`Confidential`, `PCI-Data`) applied manually/automatically to Drive files, triggering DLP policies.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can Drive Labels automatically lock a file from being shared externally?"*
* **Senior Engineer Answer**: **Yes.** Configure a DLP rule: *If Drive Label == 'Confidential' $\rightarrow$ Block external sharing*.

---

### Q67: A user accidentally permanently deleted an important Drive file 35 days ago. Can it be recovered? How?
**Detailed Technical Answer:**
Admin Console restore window is **25 days post-trash**. Past 35 days, recovery is **ONLY possible via Google Vault** if an active retention rule/hold was active.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you export the file from Vault to restore it to the user's Drive?"*
* **Senior Engineer Answer**: Search Vault Matter for file $\rightarrow$ Click **Export Results** $\rightarrow$ Download ZIP archive $\rightarrow$ Re-upload file to user's Drive via GAM or web interface.

---

### Q68: What is the difference between Drive activity reports and Drive audit logs?
**Detailed Technical Answer:**
Activity Reports provide usage summaries; Audit Logs detail raw event streams (`File Create`, `ACL Change`, `Download`).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Which log source tracks file downloads executed via Google Drive Desktop Client sync?"*
* **Senior Engineer Answer**: **Drive Audit Logs** capture sync downloads executed via Drive Desktop Client under event `Sync Download`.

---

### Q69: How do you configure a Shared Drive to prevent members from moving files out of it?
**Detailed Technical Answer:**
In Shared Drive Settings, check **Prevent members from moving files out of this shared drive**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Why is this setting important for data exfiltration prevention?"*
* **Senior Engineer Answer**: Prevents rogue internal members from moving company files out of a protected Shared Drive into their personal "My Drive" where external sharing rules might be less restrictive.

---

### Q70: What are the different levels of Drive sharing permissions and what can each level do?
**Detailed Technical Answer:**
Viewer (Read), Commenter (Comment), Editor (Edit), Manager (Full Control).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can an Editor permanently delete a file in My Drive?"*
* **Senior Engineer Answer**: An Editor can move a file to Trash, but only the file **Owner** can permanently purge the file from Trash in My Drive.

---

## 🔐 SECTION 5: SSO, SAML, OAuth & Identity (Q71–Q80)

### Q71: A user's SSO login is failing with a "SAML Response signature invalid" error. How do you troubleshoot?
**Detailed Technical Answer:**
Check X.509 cert in Google matches IdP public key, verify ACS URL (`https://www.google.com/a/domain.com/acs`), and check NTP clock drift (>300s).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you capture raw SAML XML payload responses for analysis?"*
* **Senior Engineer Answer**: Use browser Developer Tools (Network Tab) or Chrome extension **SAML Chrome Panel** to capture Base64-encoded `SAMLResponse` POST data, decode Base64 to XML, and inspect signatures.

---

### Q72: What is the difference between SP-initiated and IdP-initiated SSO? Which one does Google Workspace use by default?
**Detailed Technical Answer:**
SP-Initiated starts at `mail.google.com` (Google default); IdP-Initiated starts at IdP portal tile.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Why is SP-Initiated SSO considered more secure against CSRF attacks?"*
* **Senior Engineer Answer**: SP-Initiated flows issue a unique, cryptographically signed `SAMLRequest` containing `InResponseTo` IDs preventing unauthorized assertion injection.

---

### Q73: Can you configure different SSO profiles for different OUs in Google Workspace? How?
**Detailed Technical Answer:**
Yes, using **Partial SSO (Manage SSO Profiles)** assigned per OU.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How does Partial SSO enable zero-downtime IdP migrations?"*
* **Senior Engineer Answer**: Allows creating a new IdP SSO profile and assigning it wave-by-wave to OUs (`/IT-Pilots` first) while keeping remaining users on the legacy IdP.

---

### Q74: What happens when a Super Admin tries to login via SSO?
**Detailed Technical Answer:**
Can use SSO, but local password fallback exists to prevent lockout during IdP outages.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What is the best practice for Emergency Break-Glass Super Admin accounts?"*
* **Senior Engineer Answer**: Reside in a dedicated `/Admins` OU assigned to **SSO Profile: None (Use Google Credentials)** with FIDO2 keys.

---

### Q75: How do you audit which third-party OAuth apps have access to user data in Google Workspace?
**Detailed Technical Answer:**
Inspect **Security $\rightarrow$ API Controls $\rightarrow$ App Access Control**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you revoke app access for a single suspicious OAuth client ID?"*
* **Senior Engineer Answer**: Select App in App Access Control $\rightarrow$ Change status to **Blocked**.

---

### Q76: How would you revoke all OAuth tokens for a compromised user?
**Detailed Technical Answer:**
GAM Command: `gam user <email> delete tokens` or Admin Console Reset Tokens.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Does revoking OAuth tokens log out active browser web sessions?"*
* **Senior Engineer Answer**: Token revocation revokes API bearer tokens and mobile app sync. Web sessions are revoked via **Sign out all sessions** (`gam user <email> signout`).

---

### Q77: What is the difference between SAML 2.0 and OIDC for enterprise SSO? When would you choose one over the other?
**Detailed Technical Answer:**
SAML = XML enterprise federation; OIDC = JSON JWT for modern/mobile apps.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Why is OIDC preferred for mobile applications?"*
* **Senior Engineer Answer**: OIDC uses lightweight JSON (native to mobile) and supports PKCE (RFC 7636) securing code exchanges against interception.

---

### Q78: How do you configure Google Workspace as a Service Provider (SP) with OneLogin as the Identity Provider (IdP)?
**Detailed Technical Answer:**
Set Sign-in URL, Sign-out URL, and upload X.509 certificate under **SSO with third-party IdP**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What is the Entity ID URL for Google Workspace?"*
* **Senior Engineer Answer**: `google.com/a/domain.com` (or `google.com`).

---

### Q79: What is SCIM provisioning and how does it complement SSO in a Google Workspace + OneLogin setup?
**Detailed Technical Answer:**
SSO authenticates at login; SCIM syncs accounts, profiles, and suspensions via REST APIs.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens if SCIM provisioning fails for a new hire?"*
* **Senior Engineer Answer**: The user can authenticate at IdP via SSO, but Google Workspace returns `550 User Unknown` because the account was never created via API.

---

### Q80: What certificates are exchanged between Google Workspace and an IdP during SAML setup, and why?
**Detailed Technical Answer:**
IdP uploads public X.509 Certificate to Google Workspace for signature validation.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Does Google Workspace send its private key to the IdP?"*
* **Senior Engineer Answer**: **No.** Private keys are held securely by the issuing party and never transmitted.

---

## 📱 SECTION 6: Mobile & Endpoint Management (Q81–Q90)

### Q81: What is the difference between Basic Mobile Management and Advanced Mobile Device Management in Google Workspace?
**Detailed Technical Answer:**
Basic enforces passcodes without client agents; Advanced requires Device Policy App for full management, app distribution, and CAA.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Which mode is required to enforce Work Profile isolation on Android?"*
* **Senior Engineer Answer**: **Advanced Mobile Management**.

---

### Q82: How do you remotely wipe a lost Android device that is enrolled in Google Workspace MDM?
**Detailed Technical Answer:**
Admin Console $\rightarrow$ **Devices $\rightarrow$ Wipe Device / Wipe Account**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What is the difference between 'Wipe Device' and 'Wipe Account'?"*
* **Senior Engineer Answer**: Wipe Device performs a complete factory reset; Wipe Account removes corporate data only.

---

### Q83: How do you configure a policy to require a screen lock PIN on all managed mobile devices?
**Detailed Technical Answer:**
Go to **Devices $\rightarrow$ Password requirements**, set minimum PIN length (6 digits).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What happens if a user disables screen lock on their managed phone?"*
* **Senior Engineer Answer**: Advanced MDM blocks corporate app access until screen lock is re-enabled.

---

### Q84: What happens to Google Workspace data on a mobile device when a user is offboarded?
**Detailed Technical Answer:**
Account Wipe removes corporate work profile data while leaving personal BYOD data intact.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can an offboarded user access offline cached emails?"*
* **Senior Engineer Answer**: Executing Account Wipe purges the local SQLite database container and encryption keys, destroying offline cached data.

---

### Q85: How do you block jailbroken or rooted devices from accessing Google Workspace?
**Detailed Technical Answer:**
Enable **Block compromised devices** in Advanced MDM settings.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How does Advanced MDM detect rooted Android devices?"*
* **Senior Engineer Answer**: Uses Google Play Integrity API / SafetyNet Attestation checking OS kernel integrity.

---

### Q86: What is Endpoint Verification and what information does it report to the Admin Console?
**Detailed Technical Answer:**
Chrome extension reporting OS version, encryption status, screen lock, and serial number.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Does Endpoint Verification track real-time GPS location?"*
* **Senior Engineer Answer**: **No.** Endpoint Verification collects hardware posture telemetry only, not GPS tracking data.

---

### Q87: How do you configure a policy that allows only managed devices to access Google Drive?
**Detailed Technical Answer:**
Create CAA Access Level `device.is_managed_device == true` and assign to Drive.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can unmanaged users access Gmail under this policy?"*
* **Senior Engineer Answer**: Yes, if Gmail is left unassigned in CAA settings.

---

### Q88: What is the difference between a corporate-owned device and a BYOD device in Google Workspace management?
**Detailed Technical Answer:**
Corporate-owned allows full wipe and serial tracking; BYOD isolates work profile data.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you import corporate serial numbers?"*
* **Senior Engineer Answer**: Upload CSV under **Devices $\rightarrow$ Company-owned devices**.

---

### Q89: How do you push a Chrome extension to all managed Chrome browsers across the organization?
**Detailed Technical Answer:**
Go to **Devices $\rightarrow$ Chrome $\rightarrow$ Apps & extensions $\rightarrow$ Force install**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can users uninstall a Force-Installed extension?"*
* **Senior Engineer Answer**: **No.** Force-installed extensions cannot be disabled or uninstalled by end users.

---

### Q90: What is Chrome Enterprise and how does it differ from Chrome Browser Cloud Management?
**Detailed Technical Answer:**
Chrome Enterprise includes OS management/licensing; CBCM manages cloud browser policies across desktop OS.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Is CBCM free for Google Workspace customers?"*
* **Senior Engineer Answer**: **Yes.** CBCM is included without additional license cost.

---

## 📊 SECTION 7: Reporting, Audit & Compliance (Q91–Q100)

### Q91: What are the different types of audit logs available in Google Workspace and what does each one track?
**Detailed Technical Answer:**
Admin (settings), Drive (file ops), Login (auth), SAML (SSO), Token (OAuth), Groups (members).

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Which log records 2SV setting changes?"*
* **Senior Engineer Answer**: **Admin Audit Log**.

---

### Q92: How do you configure an automated weekly report of all admin activity changes in the domain?
**Detailed Technical Answer:**
Create Saved Query in Investigation Tool $\rightarrow$ **Create Alert Rule $\rightarrow$ Weekly Email**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What format are alert emails sent in?"*
* **Senior Engineer Answer**: HTML formatted email with summary table and link to Investigation Tool.

---

### Q93: How do you export Google Workspace audit logs to BigQuery for long-term retention and analysis?
**Detailed Technical Answer:**
Go to **Reporting $\rightarrow$ BigQuery Export**, authorize GCP Project, enable streaming.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How do you query BigQuery logs using SQL?"*
* **Senior Engineer Answer**: Use BigQuery Studio running SQL queries on `activity_logs` tables.

---

### Q94: What is the difference between the Reports section and the Security Investigation Tool in the Admin Console?
**Detailed Technical Answer:**
Reports provides static graphs; Investigation Tool provides interactive querying and remediation.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Which tool allows bulk file deletion?"*
* **Senior Engineer Answer**: **Security Investigation Tool**.

---

### Q95: How do you set up an alert that fires when a new Super Admin account is created?
**Detailed Technical Answer:**
Alert Center Rule: **Event == Admin Privilege Grant $\rightarrow$ Privilege == Super Admin**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can alert emails be sent to external SOC addresses?"*
* **Senior Engineer Answer**: **Yes**, by specifying external email addresses in rule action settings.

---

### Q96: A compliance team needs a report of all files shared externally in the last 90 days. How do you generate this?
**Detailed Technical Answer:**
Investigation Tool: **Drive log events $\rightarrow$ Visibility == Shared Externally $\rightarrow$ Time: 90 days $\rightarrow$ Export**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Where does the exported CSV land?"*
* **Senior Engineer Answer**: Automatically saved to the Admin's Google Drive root folder.

---

### Q97: How do you monitor and report on users who have not enabled 2-Step Verification?
**Detailed Technical Answer:**
Go to **Reporting $\rightarrow$ User reports $\rightarrow$ Security $\rightarrow$ Filter: 2SV Enrolled == False**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"Can you export this 2SV status list via GAM?"*
* **Senior Engineer Answer**:
```bash
gam print users fields primaryEmail,isEnrolledIn2Sv > 2sv_audit.csv
```

---

### Q98: What is Google Workspace's data retention policy for audit logs natively, and how do you extend it?
**Detailed Technical Answer:**
Native retention is **6 months** (180 days). Extend by streaming to **BigQuery**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"How far back can Email Log Search (ELS) query?"*
* **Senior Engineer Answer**: **30 days**.

---

### Q99: How do you investigate a suspected data breach where an insider may have exfiltrated data via Google Drive?
**Detailed Technical Answer:**
Query Investigation Tool for Actor $\rightarrow$ Inspect Downloads, External Shares, Ownership Changes, and OAuth Tokens.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What action do you take during investigation?"*
* **Senior Engineer Answer**: Immediately suspend user, revoke OAuth tokens, and preserve Google Vault data.

---

### Q100: A regulatory body requires you to prove that emails were never tampered with in transit. Which Google Workspace features and logs would you present as evidence?
**Detailed Technical Answer:**
Present **ELS TLS Logs**, **DKIM Headers**, **MTA-STS Enforcement Logs**, and **Google Vault Cryptographic Hash Manifests**.

**Follow-up Question & Answer:**
* **Interviewer Follow-up Question**: *"What cryptographic hash algorithm does Google Vault export manifest use to verify data integrity?"*
* **Senior Engineer Answer**: Google Vault exports include an `XML/CSV` manifest containing **SHA-256 cryptographic hashes** for every exported message and attachment file, proving data integrity and chain-of-custody compliance.
