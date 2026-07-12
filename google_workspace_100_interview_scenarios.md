# Google Workspace Admin & Engineer: 100 Scenario-Based Interview Questions

This handbook contains 100 highly realistic, scenario-based interview questions with comprehensive, architectural answers. Use this repository to prepare for senior Google Workspace Administrator, Deployment Engineer, and Collaboration Architect interviews.

---

## 📑 Table of Contents
1.  **Identity, Directory, & Lifecycle Management** (Questions 1–15)
2.  **Authentication & Single Sign-On (SSO)** (Questions 16–30)
3.  **Mail Flow & Gmail Infrastructure** (Questions 31–45)
4.  **Google Drive & Collaboration Security** (Questions 46–60)
5.  **Mobile Device & Endpoint Management** (Questions 61–75)
6.  **APIs, Integrations, & Domain-Wide Delegation** (Questions 76–85)
7.  **Data Migrations & Coexistence** (Questions 86–92)
8.  **Security Center, Investigation Tool, & Auditing** (Questions 93–97)
9.  **Disaster Recovery, Billing, & Account Configuration** (Questions 98–100)

---

## 📂 Part 1: Identity, Directory, & Lifecycle Management (Q1-15)

### Q1: Syncing Custom HR Fields
*   **Scenario:** HR needs to synchronize custom fields (`WorkdayID`, `BusinessUnit`, `ManagerEmail`) from Workday to the Google Workspace directory.
*   **Question:** How do you configure the Admin Console to support these custom attributes, and how are they populated?
*   **Answer:** 
    1.  Navigate to `Directory > Users > More options > Manage custom fields`.
    2.  Click **Add Custom Field**, create a category named `HR Fields`, and add fields: `WorkdayID` (Text), `BusinessUnit` (Text), and `ManagerEmail` (Email). Set access to "Visible to organization" or "Read-only for users".
    3.  Populate these fields programmatically by making `PATCH` requests to the **Admin SDK Directory API** (`/admin/directory/v1/users/{userKey}`) containing the `customSchemas` JSON object, or by mapping them using Google Cloud Directory Sync (GCDS) or a SCIM-based provisioning integration.

### Q2: Dynamic Group Provisioning
*   **Scenario:** You need to maintain an email distribution list for all developers in North America.
*   **Question:** How do you set up a group that automatically updates its membership based on user attributes?
*   **Answer:**
    Use **Dynamic Groups** (requires Enterprise Standard or Plus).
    1.  Go to `Directory > Groups` and click **Create Group**.
    2.  Select **Dynamic Group**.
    3.  Define the membership query using attributes, e.g.:
        `user.department == 'Engineering' && user.addresses.exists(a, a.type == 'work' && a.country_code == 'US')`
    4.  Google Workspace continuously evaluates directory updates and automatically joins or removes users based on query matches.

### Q3: Secondary Domain vs. Domain Alias for Acquisition
*   **Scenario:** Your company acquires a startup (`startup.com`). They want to keep their existing email addresses but share resources.
*   **Question:** Would you configure `startup.com` as a Secondary Domain or a Domain Alias? Why?
*   **Answer:**
    Configure it as a **Secondary Domain**. 
    *   **Reasoning:** A Domain Alias automatically maps one-to-one to existing mailboxes (e.g., `user@company.com` gets `user@startup.com` as an alias). It does not allow new, independent users. Since the startup has its own employees who need separate, distinct mailboxes under `startup.com`, a Secondary Domain must be added, allowing you to create separate user accounts with the `@startup.com` suffix inside the same tenant.

### Q4: The Unmanaged Account Conflict
*   **Scenario:** You add a new employee `alice@company.com` to the directory. Google shows an error stating that the email address already exists, yet you cannot find her in your user list.
*   **Question:** What is causing this conflict, and how do you resolve it?
*   **Answer:**
    *   **Cause:** Alice created a personal Google Account using her work email (`alice@company.com`) before the organization registered the Google Workspace tenant (known as an **Unmanaged/Conflicting Account**).
    *   **Resolution:**
        1.  Use the **Transfer Tool for Unmanaged Users** under `Directory > Users > More options > Transfer tool for unmanaged users`.
        2.  Enter Alice's email address and send a transfer request.
        3.  Alice will receive an email prompting her to either transfer the account (converting it to a managed account with her existing data) or rename her personal account (e.g., to `@gmail.com`), freeing up the `alice@company.com` handle in your directory.

### Q5: Preserving Files of a Terminated Employee
*   **Scenario:** An employee leaves the company. You must delete their account to save license costs, but you cannot lose their Google Drive files.
*   **Question:** What is the correct sequence of admin actions?
*   **Answer:**
    1.  Suspend the account first to block access.
    2.  Navigate to `Directory > Users`, click the user's name, and select **Delete User**.
    3.  In the deletion dialogue, Google prompts you to **Transfer data to another user**.
    4.  Enter the target manager's or a storage archive account email, select **Drive and Docs**, check the box to include private files, and complete the deletion.
    5.  Google creates a folder in the recipient's Drive containing the transferred files, preserving their original folder structure.

### Q6: Restricting User Profile Edits
*   **Scenario:** Users are changing their names and photos in their Google Accounts, causing confusion in the corporate directory.
*   **Question:** Where do you disable this capability?
*   **Answer:**
    1.  Navigate to `Directory > Directory settings > Profile editing`.
    2.  Uncheck the boxes for **Name**, **Photo**, **Gender**, and **Pronouns**.
    3.  Click **Save**. Only administrators or directory sync tools will now be able to modify these attributes.

### Q7: Shared Contacts Deployment
*   **Scenario:** You want to deploy a shared contact list (vendors, partners) to all employees' Google Contacts directories.
*   **Question:** How is this accomplished since there is no "Shared Contacts" UI in the Admin Console?
*   **Answer:**
    *   Shared Contacts are managed programmatically via the **Domain Shared Contacts API** (part of the Admin SDK).
    *   Use a tool like GAM (Google Apps Manager) or write a Python script using the Google API Client Library. The script authenticates as a service account with Domain-Wide Delegation and pushes the contact list using the Shared Contacts endpoint. Once synced, these contacts auto-complete in Gmail and Search for all domain users.

### Q8: Organizational Unit (OU) Inheritance Override
*   **Scenario:** You have a parent OU called "Employees" where Google Chat is enabled. You create a child OU called "Contractors" and want to disable Chat for them.
*   **Question:** Explain how inheritance works in this setup and how you implement the change.
*   **Answer:**
    *   By default, child OUs inherit all settings from their parent OU.
    *   To override:
        1.  Navigate to `Apps > Google Workspace > LTI/Core Services > Google Chat`.
        2.  On the left panel, select the child OU **Contractors**.
        3.  Under Service Status, change the selection from **ON (inherited)** to **OFF**.
        4.  Click **Override** (instead of Save) to break inheritance for this setting. The status now displays as **OFF (locally applied)**.

### Q9: Security Group Membership Restrictions
*   **Scenario:** An admin attempts to add an external partner's email address to an internal Security Group. The console rejects the addition.
*   **Question:** Why did this fail, and what is the underlying security rule?
*   **Answer:**
    *   **Security Groups** (Google Groups labeled with the "Security" label) enforce strict boundaries to prevent privilege escalation.
    *   **The Rule:** A standard, external, or unmanaged account cannot join a Security Group. Only users belonging to the same organization, verified service accounts, or other internal Security Groups can be members. This prevents external actors from being accidentally granted administrative roles or access to sensitive corporate resources.

### Q10: Delegated Admin for Password Resets Only
*   **Scenario:** You need to delegate password reset authority to a Help Desk team, but they must not be allowed to create, delete, or suspend users.
*   **Question:** How do you set up this role?
*   **Answer:**
    1.  Navigate to `Account > Admin roles`.
    2.  Click **Create New Role**.
    3.  Under Privileges, select `Directory > Users > Reset Passwords`. Do **not** check "Create", "Delete", or "Manage User Attributes".
    4.  Save the role.
    5.  Assign this custom role to the Help Desk users or their Google Security Group, scoping the assignment to the target OUs they are authorized to manage.

### Q11: Locating Suspended Accounts Programmatically
*   **Scenario:** An auditor wants a list of all suspended user accounts in the directory to verify deprovisioning.
*   **Question:** How do you generate this list using Google Apps Manager (GAM)?
*   **Answer:**
    Run the following GAM command in the terminal:
    `gam print users query "isSuspended=true" fields id name primaryEmail suspendedReason`
    This queries the Admin SDK Directory API using the `isSuspended` parameter and outputs a CSV file containing the IDs, names, emails, and suspension reasons (e.g., admin-initiated vs. login-failure-initiated) of all suspended accounts.

### Q12: Enforcing Directory Visibility Restrictions
*   **Scenario:** You want to hide contractors from the global Directory so that regular employees cannot search for or find them in Gmail autocomplete.
*   **Question:** How do you configure Directory sharing policies to segment visibility?
*   **Answer:**
    1.  Navigate to `Directory > Directory settings > Sharing settings`.
    2.  Select **Custom directories**.
    3.  Create two directories: One for Employees (containing all users) and one for Contractors (containing only their OU).
    4.  Assign the custom directories: Set the Employees OU to use the global custom directory, and set the Contractors OU to see only their custom directory.
    5.  Save. Autocomplete and Directory searches will now respect these boundaries.

### Q13: Custom Administrative Scopes
*   **Scenario:** You have regional offices in London and New York. You want the London Help Desk admin to reset passwords **only** for users physically located in the London office.
*   **Question:** How do you configure this scoping boundary?
*   **Answer:**
    1.  Create an OU structure: `Root > United Kingdom > London Office`. Move all London employees into this OU.
    2.  Navigate to `Account > Admin roles` and select the Help Desk custom role.
    3.  Click **Assign role**.
    4.  Instead of assigning the role to the "Root" organization, select the specific child OU **London Office**.
    5.  Assign it to the London Help Desk admin. They will now only be able to perform administrative actions on users nested within that specific OU branch.

### Q14: Recovery Information Lockdown
*   **Scenario:** To prevent SIM-swapping and social engineering attacks, you want to block admins and users from adding personal recovery phone numbers to their accounts.
*   **Question:** How do you disable this setting?
*   **Answer:**
    1.  Navigate to `Security > Authentication > Account Recovery`.
    2.  Locate the **Recovery information** section.
    3.  Turn **OFF** the setting: "Allow admins and users to add recovery email information to their account" and "Allow admins and users to add recovery phone information to their account".
    4.  Click **Save**.

### Q15: GCDS Sync Latency Mitigations
*   **Scenario:** Your company syncs Active Directory to Google using GCDS every 6 hours. When an employee is terminated in AD, they remain active in Google for up to 6 hours.
*   **Question:** How do you mitigate this security gap?
*   **Answer:**
    1.  Implement a real-time webhook or automation script in Active Directory.
    2.  When an account status is updated to disabled in AD, trigger an API call directly to the Google Admin SDK Directory API:
        `PUT https://admin.googleapis.com/admin/directory/v1/users/{userKey}`
        with payload: `{"suspended": true}`.
    3.  This suspends the Google account within seconds, while GCDS will later clean up and align the status during its scheduled delta sync.

---

## 🔐 Part 2: Authentication & Single Sign-On (SSO) (Q16-30)

### Q16: Resolving SAML Error 403: app_not_configured_for_user
*   **Scenario:** Users trying to sign in to Google Workspace via an external Identity Provider (Okta) receive the error "403: app_not_configured_for_user" on a Google landing page.
*   **Question:** What is the root cause, and how do you resolve it?
*   **Answer:**
    *   **Cause:** This error occurs during an **IdP-Initiated** login flow. The user authenticated at Okta and Okta posted the SAML assertion to Google's ACS URL, but the user is not assigned to the Google Workspace application within the Okta admin portal.
    *   **Resolution:** Log into the Okta Admin Console, locate the Google Workspace app integration, and assign the affected users or their Okta groups to the application.

### Q17: Mitigating SAML SSO Outages (Break-Glass Accounts)
*   **Scenario:** Your primary IdP (Azure AD/Entra ID) is down. All employees, including administrators, are locked out of Google Workspace because SSO redirection fails.
*   **Question:** How do you log into the Admin Console to disable SSO, and how should you have architected this?
*   **Answer:**
    *   **Immediate Action:** Navigate directly to the SAML bypass URL:
        `https://admin.google.com/a/yourdomain.com/ServiceLogin?skipSso=true`
        Log in using the email address and password of a local Super Admin account, bypassing the SSO redirect.
    *   **Architectural Prevention:** Always maintain at least two "Break-Glass" Super Admin accounts that do not use the corporate SSO domain. These accounts must have complex passwords, hardware security keys (FIDO2) enrolled, and be excluded from SSO profiles in the Admin Console.

### Q18: Custom SAML SSO Profiles
*   **Scenario:** You want your regular employees to authenticate via Okta, but your external contractors must authenticate using Microsoft Entra ID.
*   **Question:** How do you configure different SSO rules for different sets of users?
*   **Answer:**
    Use **SSO Profiles** (Multiple IdP support).
    1.  Navigate to `Security > Authentication > SSO with third-party IdP`.
    2.  Click **Add SAML Profile** and configure the profile for Entra ID (upload metadata, entity ID, login URL). Do the same for Okta.
    3.  Go to **Manage SSO profile assignments**.
    4.  Select the **Contractors** OU on the left and assign them to the Entra ID SSO profile.
    5.  Select the **Employees** OU and assign them to the Okta SSO profile.
    6.  Save the assignments.

### Q19: Post-SSO Verification Settings
*   **Scenario:** Even though users authenticate successfully at your IdP, Google prompts them for an additional security question or 2SV prompt when they sign in from a new IP.
*   **Question:** Where do you disable or modify this behavior?
*   **Answer:**
    1.  Navigate to `Security > Authentication > Login challenges`.
    2.  Under **Post-SSO verification**, locate the settings.
    3.  Select **Don’t ask users for additional verifications from Google** if they log in via your SSO profile.
    4.  *Note:* Only select this if your IdP already enforces strict risk-based MFA, as this setting transfers trust entirely to the IdP.

### Q20: Multi-Party Approval for SSO Changes
*   **Scenario:** To prevent a compromised administrator from hijacking the domain by changing the third-party SSO settings, you want to require a second administrator's approval for any SSO modifications.
*   **Question:** How do you configure this?
*   **Answer:**
    1.  Navigate to `Security > Multi-party approval settings`.
    2.  Turn **ON** the setting: "Require multi-party approval for sensitive admin actions".
    3.  Locate the rule for **SSO with third-party IDPs** and ensure it is checked/enabled.
    4.  *Under the hood:* If an admin tries to change the SSO endpoint or upload a certificate, Google puts the request in a "Pending" state and emails all other Super Admins. The changes do not take effect until another authorized admin approves the request.

### Q21: Troubleshooting SAML Clock Skew
*   **Scenario:** Users are redirected to Okta, log in, but are redirected back to a Google error page showing: `SAML Assertion expired / Invalid timestamp`.
*   **Question:** What is the cause, and how do you resolve it?
*   **Answer:**
    *   **Cause:** **Clock Skew**. The system clocks of the IdP (Okta) and Google are out of sync by more than the allowed threshold (usually 5 minutes). The assertion's `IssueInstant` or `NotOnOrAfter` values are evaluated as expired by Google.
    *   **Resolution:** Verify that the IdP servers are synchronized with a reliable Network Time Protocol (NTP) pool. If you cannot modify server time immediately, check if the IdP has a setting to increase the allowed clock skew leeway.

### Q22: FIDO2 Passwordless Enforcements
*   **Scenario:** You want to eliminate passwords entirely for your users, allowing them to sign in using passkeys (touch ID, face unlock, or security keys) on their devices.
*   **Question:** Where do you enable this?
*   **Answer:**
    1.  Navigate to `Security > Authentication > Passwordless`.
    2.  Turn **ON** the setting: "Allow users to skip their password and authenticate with a passkey".
    3.  Under **Passkeys Restriction**, select your allowed platforms (e.g., allow on any device or restrict to corporate-owned hardware).
    4.  Users will now be prompted to register their device's passkey, allowing them to bypass password prompts on subsequent logins.

### Q23: 2-Step Verification (2SV) Enrollment Grace Period
*   **Scenario:** You enforce "Security Key Only" 2SV for the entire organization. New hires are locked out on their first day because they cannot log in to register their key.
*   **Question:** What setting prevents this lockout loop?
*   **Answer:**
    *   Configure the **New user enrollment period** under `Security > Authentication > 2-Step Verification`.
    *   Set this to an appropriate duration (e.g., 1 week). This creates a temporary bypass for newly created accounts, allowing them to log in with a password alone during their first week, register their FIDO2 key, and complete the enrollment before the enforcement policy applies to their account.

### Q24: OAuth 2.0 Scopes Whitelisting
*   **Scenario:** You want to block all third-party applications from accessing Google APIs, but you must allow **Slack** to access users' Google Calendars.
*   **Question:** How do you set this up?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > API controls`.
    2.  Click **Manage Third-Party App Access**.
    3.  Under settings, configure the default rule for unconfigured apps to **Blocked**.
    4.  Click **Add App > OAuth App Name or Client ID**.
    5.  Search for Slack's client ID, select the application, check the scopes it requests, and set the trust level to **Trusted** or **Limited** (limiting it to calendar scopes only). Click Save.

### Q25: Mitigating Token Theft (Session Control)
*   **Scenario:** An attacker steals an admin's OAuth refresh token via a phishing cookie-stealing malware.
*   **Question:** How do you terminate the attacker's session immediately, and how do you reduce the validity of future tokens?
*   **Answer:**
    1.  **Immediate Remediation:** Navigate to `Directory > Users`, select the admin's account, and click **Security**. Scroll to **Sign-in cookies** and click **Reset**. This invalidates all active session cookies and OAuth access tokens.
    2.  **Prevention:** Navigate to `Security > Access and data control > Google session control`. Under **Session control**, change the Web session duration from "14 days" to **4 hours** or **8 hours**, forcing re-authentication and decreasing the blast radius of stolen tokens.

### Q26: Re-authenticating GCP Admins More Frequently
*   **Scenario:** You want regular Workspace users to keep their sessions active for 7 days, but GCP Project Owners must re-authenticate every 1 hour.
*   **Question:** How do you configure this?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > Google Cloud session control`.
    2.  Set the session duration for the Google Cloud Platform console and Google Cloud SDK to **1 hour**.
    3.  Ensure the target GCP administrators are nested in their own OU, and apply this setting specifically to their OU branch, while leaving the root organization set to the default session duration.

### Q27: Restricting Less Secure Apps (LSA)
*   **Scenario:** Some users are trying to sync their mail using legacy clients that do not support modern OAuth authentication (e.g., Outlook 2013).
*   **Question:** How do you block these insecure protocols globally?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > Less secure apps`.
    2.  Select **Disable access to less secure apps (Recommended)**.
    3.  Click **Save**. This blocks all legacy IMAP/POP clients that rely only on basic username and password authentication, forcing the use of modern OAuth clients.

### Q28: Client-Side Encryption (CSE) Key Management
*   **Scenario:** Your company requires Client-Side Encryption for sensitive Google Docs.
*   **Question:** How does Google Workspace handle the encryption keys, and who controls them?
*   **Answer:**
    With CSE, the data is encrypted **before** it reaches Google's servers. The customer controls the encryption keys, which are hosted on an external key management service (KMS) or an identity provider that integrates with Google's CSE API. Google never has access to the private keys or the decrypted content, ensuring total confidentiality but disabling server-side indexing/search.

### Q29: SAML NameID Format Verification
*   **Scenario:** During an SSO setup, authentication fails with: `SAML NameID missing or incorrect format`.
*   **Question:** What does Google Workspace require the NameID to contain?
*   **Answer:**
    Google Workspace requires the SAML assertion's `<Subject>` block to contain a `<NameID>` element. By default, this value must map exactly to the user's primary email address (e.g., `user@company.com`) or their primary alias, formatted as `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.

### Q30: Advanced Protection Program Enrollment
*   **Scenario:** You have high-risk executives whom you want to enroll in Google's Advanced Protection Program.
*   **Question:** Where is this configured, and what strict rules are automatically applied to enrolled accounts?
*   **Answer:**
    1.  Navigate to `Security > Authentication > Advanced Protection Program`.
    2.  Check the box to **Enable user enrollment**.
    3.  **Enforced Rules:** Enrolled users are restricted to using physical security keys (FIDO2) for login, all non-vetted third-party OAuth access is automatically blocked, and account recovery is highly restricted (requiring administrator intervention) to prevent social engineering.

---

## 📧 Part 3: Mail Flow & Gmail Infrastructure (Q31-45)

### Q31: Debugging SPF "Too Many DNS Lookups"
*   **Scenario:** You add multiple third-party senders to your SPF record. Suddenly, your emails start failing SPF validation with a `PermError: Too many DNS lookups`.
*   **Question:** What is causing this, and how do you resolve it?
*   **Answer:**
    *   **Cause:** The SPF RFC limit allows a maximum of **10 nested DNS lookups** to prevent denial-of-service loops during SPF validation. Every `include:` statement in your record causes one or more DNS queries.
    *   **Resolution:** Flatten your SPF record. Remove unnecessary `include:` statements, replace them with direct IP ranges (`ip4:1.2.3.4/24`), or use an SPF flattening service. If you must use multiple third-party services, consider setting up subdomains (e.g., `marketing.company.com`) with their own dedicated, short SPF records.

### Q32: Multiple SPF Records Conflict
*   **Scenario:** A domain has two TXT records for SPF:
    Record 1: `v=spf1 include:_spf.google.com ~all`
    Record 2: `v=spf1 include:mailgun.org ~all`
*   **Question:** What is the impact of this setup on email delivery?
*   **Answer:**
    *   **Impact:** Having multiple SPF records on a single domain is an invalid configuration. Receiving mail servers will evaluate the setup as a **PermError** (Permanent Error) and will typically reject or flag all emails from your domain.
    *   **Resolution:** Merge the records into a single TXT record:
        `v=spf1 include:_spf.google.com include:mailgun.org ~all`

### Q33: Restricting Gmail Outbound Relay
*   **Scenario:** You want to ensure that emails sent from your corporate apps (hosted on AWS) can use Google's SMTP relay, but you want to restrict access so only your AWS server's static IP can authenticate.
*   **Question:** How do you configure the SMTP Relay service?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Gmail > Routing`.
    2.  Locate the **SMTP relay service** setting and click **Configure**.
    3.  Under **Allowed senders**, select **Only addresses in my domains**.
    4.  Under **Authentication**, check the box for **Only accept mail from the specified IP addresses**.
    5.  Input your AWS server's static IP address or CIDR range.
    6.  Check **Require TLS encryption** for secure transit. Click Save.

### Q34: DKIM Key Selector Conflicts
*   **Scenario:** You already use Google Workspace with DKIM using the default selector `google`. You now need to configure a third-party email system (SendGrid) to send emails using DKIM for the same domain.
*   **Question:** How do you avoid key selector conflicts?
*   **Answer:**
    *   **Resolution:** Use a different **DKIM selector**. 
    *   DNS records locate DKIM keys using the selector prefix (e.g., `selector._domainkey.company.com`). SendGrid must be configured to generate and sign mail using its own selector (e.g., `sendgrid` or `s1`), creating a DNS TXT record at `sendgrid._domainkey.company.com`. Google will continue to sign Workspace mail using `google._domainkey.company.com`.

### Q35: Split Delivery Configuration
*   **Scenario:** During a phased migration, some users are in Google Workspace and others remain on an on-premises Exchange server. External emails must land in Gmail first. If the recipient does not exist in Google, the mail must automatically route to the Exchange server.
*   **Question:** How do you configure this?
*   **Answer:**
    Configure **Split Delivery** using Gmail routing:
    1.  Navigate to `Apps > Google Workspace > Gmail > Routing`.
    2.  Scroll to **Routing** and click **Configure**.
    3.  **Email messages to affect:** Check **Inbound**.
    4.  Under "If the above expressions match," set the action to **Modify message**.
    5.  Scroll to **Route**, check **Change route**, and select a custom mail route pointing to your Exchange server's host (MX/IP).
    6.  Under **Options**, select **Perform this action only for non-recognized addresses**.
    7.  Click Save. This checks Google's directory first; if the user is missing, it forwards the email to Exchange.

### Q36: Inbound Mail Gateway Setup
*   **Scenario:** Your company deploys an external email gateway (Proofpoint) for spam filtering. All inbound emails route through Proofpoint before entering Gmail.
*   **Question:** How do you configure Gmail to trust this gateway and prevent Gmail's internal filters from marking the gateway's IP as a source of spam?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Gmail > Spam, Phishing and Malware`.
    2.  Locate the **Inbound gateway** setting and click **Configure**.
    3.  Check **Enable**.
    4.  Add the IP addresses or CIDR blocks of your Proofpoint gateway servers.
    5.  Check **Automatically detect external IP** (recommmended so Gmail looks at the original sender's IP in the mail header, not the gateway IP, for SPF checks).
    6.  Click Save.

### Q37: Preventing Phishing via Internal Address Spoofing
*   **Scenario:** Hackers are sending external phishing emails with the display name and sender address of your CEO (`ceo@company.com`) to employees.
*   **Question:** How do you block this using Gmail security settings?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Gmail > Safety`.
    2.  Under **Spoofing and authentication**, check **Protect against spoofing of employee names** and **Protect against spoofing of domain names**.
    3.  Choose your action: Select **Move to Spam** or **Put in administrative quarantine** to intercept emails that claim to be internal but fail authentication (SPF/DKIM/DMARC).
    4.  Enable safety warnings to alert users when a sender has a similar name to an executive but uses an external address.

### Q38: Restricting Attachment File Types
*   **Scenario:** Your company wants to block all inbound and outbound emails that contain executable files (`.exe`, `.bat`, `.scr`) to prevent malware.
*   **Question:** How do you set up this rule?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Gmail > Compliance`.
    2.  Find **Attachment compliance** and click **Configure**.
    3.  Check **Inbound** and **Outbound**.
    4.  Under expressions, click **Add**. Select **File type match** and choose the restricted extensions (e.g., Executable files).
    5.  Under actions, select **Reject message** and enter a custom rejection notice (e.g., "Attachments containing executable code violate corporate security policies"). Save the rule.

### Q39: Dual Delivery for Archiving
*   **Scenario:** Legal requirements dictate that all incoming employee emails must land in their Gmail inboxes and simultaneously be archived in a separate on-premises archiving appliance.
*   **Question:** How do you set this up?
*   **Answer:**
    Configure **Dual Delivery**:
    1.  Navigate to `Apps > Google Workspace > Gmail > Routing`.
    2.  Scroll to **Routing** and click **Configure**.
    3.  Set target to **Inbound**.
    4.  Under actions, select **Modify message**.
    5.  Scroll to **Also deliver to**, check the box, click **Add**, and input the route/destination email server of your archiving appliance.
    6.  Save the rule. Incoming mail will now be delivered to both Gmail and the archive in parallel.

### Q40: Managing Admin Quarantines
*   **Scenario:** You configured a spam filter to send suspicious emails to an administrative quarantine.
*   **Question:** How do you access this quarantine, and what actions can you perform on the held messages?
*   **Answer:**
    *   **Access:** Go to the Admin Console and navigate to `Apps > Google Workspace > Gmail > Manage Quarantines` or visit `admin.google.com/ac/quarantine`.
    *   **Actions:** The administrator can view the headers and content of quarantined emails, and choose to:
        *   **Release:** Deliver the email to the user's inbox.
        *   **Reject:** Block the email and notify the sender.
        *   **Drop:** Silently delete the email without notifying the sender.

### Q41: DMARC Policy Enforcement Strategies
*   **Scenario:** You want to implement a strict DMARC policy, but you are afraid of blocking legitimate marketing emails that might not be configured correctly.
*   **Question:** What phased approach should you take?
*   **Answer:**
    1.  **Phase 1 (none):** Set the TXT record to `p=none; rua=mailto:dmarcreports@company.com`. Analyze the XML reports for several weeks to identify legitimate third-party services that fail validation.
    2.  **Phase 2 (quarantine):** Once SPF/DKIM are fixed for all senders, update the policy to `p=quarantine; pct=25;` (enforcing spam routing on only 25% of failing mail). Gradually increase `pct` to 100%.
    3.  **Phase 3 (reject):** Change the record to `p=reject;` to fully drop all unauthorized emails claiming to be from your domain.

### Q42: Restricting Internal Communication via Compliance Rules
*   **Scenario:** You want to block employees in the "Underage Interns" OU from sending emails to anyone outside the organization.
*   **Question:** How do you configure this restriction?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Gmail > Compliance`.
    2.  Find **Restrict Delivery** and click **Configure**.
    3.  Add a description: `Block External Outbound for Interns`.
    4.  Under allowed list, select your internal domain only.
    5.  Under actions, select **Reject message**.
    6.  Apply this rule specifically to the **Underage Interns** child OU by selecting it on the left panel before saving.

### Q43: Stripping Attachments for Contractors
*   **Scenario:** You want contractors to receive emails, but you want to strip any attachments containing Microsoft Office documents (`.docx`, `.xlsx`) to prevent data leakage.
*   **Question:** How do you write this rule?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Gmail > Compliance`.
    2.  Find **Attachment compliance** and select the **Contractors** OU. Click Configure.
    3.  Select **Inbound**.
    4.  Add expression: **File type match** > select Office documents.
    5.  Under actions, select **Modify message**.
    6.  Check **Strip attachments from message** and optionally append a warning text to the message body. Save.

### Q44: Appending Compliance Footers
*   **Scenario:** Corporate legal requires a specific confidentiality footer appended to the bottom of all outbound emails sent by the Sales department.
*   **Question:** Where and how is this configured?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Gmail > Compliance`.
    2.  Select the **Sales** OU on the left panel.
    3.  Locate the **Compliance footer** setting and click **Configure**.
    4.  Enter the required legal disclaimer text in the editor.
    5.  Save. Gmail will now automatically append this footer to all outgoing external emails sent by users in the Sales OU.

### Q45: Troubleshooting Latent Outbound Email Blocks
*   **Scenario:** Users in the Sales department report that their outgoing emails are being delayed or failing with a `550 Blocked by Content Compliance Policy` error.
*   **Question:** Where do you investigate the exact cause of this block?
*   **Answer:**
    1.  Navigate to the **Investigation Tool** in the Admin Console.
    2.  Select **Gmail log events** as the data source.
    3.  Filter by **Sender** (a Sales user) and check the events list.
    4.  Click on the failed event. Look at the `Compliance Rules Triggered` metadata. This tells you the exact rule name (e.g., "Outbound Credit Card Block") that flagged the email, allowing you to edit the rule or educate the user.

---

## 📂 Part 4: Google Drive & Collaboration Security (Q46-60)

### Q46: Restricting External Folder Sharing
*   **Scenario:** You want users to be able to share individual Google Drive files externally, but you must prevent them from sharing entire Drive folders externally.
*   **Question:** How do you configure this constraint?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Drive and Docs > Sharing settings`.
    2.  Set the default **Link sharing** option to restrict access.
    3.  Locate the policy for **Folder sharing**.
    4.  Set the policy to **Allow users to share files externally but restrict external folder sharing**. This allows file collaboration while preventing users from accidentally leaking entire directory structures.

### Q47: Shared Drive Creator Restrictions
*   **Scenario:** You want to prevent regular users from creating new Shared Drives to keep the storage structured, restricting creation to IT staff only.
*   **Question:** Where do you configure this?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Drive and Docs > Sharing settings`.
    2.  Locate the **Shared Drive creation** settings.
    3.  Select the root OU, and uncheck **Prevent users from creating new shared drives**.
    4.  Select a child OU (e.g., "Regular Employees") and check the box to **Prevent users in this OU from creating new shared drives**, while leaving it enabled for the IT Staff OU.

### Q48: Data Loss Prevention (DLP) for Credit Cards
*   **Scenario:** You must prevent any user from sharing Google Sheets containing customer credit card numbers with anyone outside the organization.
*   **Question:** How do you configure a DLP rule to enforce this?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > Data protection`.
    2.  Click **Create Rule > New Rule**.
    3.  Under detector, select the prebuilt detector for **Credit Card Number**.
    4.  Set the condition: **File shared externally**.
    5.  Set the action: **Block external sharing** and **Send email alert to Super Admins**.
    6.  Activate the rule. Google will now automatically scan documents and revoke external sharing if a credit card pattern is detected.

### Q49: Restricting Shared Drive Download/Copy
*   **Scenario:** You have a Shared Drive called "Board of Directors" containing confidential financial reports. You must prevent members with View or Comment access from downloading, copying, or printing files.
*   **Question:** How do you enforce this restriction?
*   **Answer:**
    1.  Navigate to Google Drive, right-click the **Board of Directors** Shared Drive, and select **Shared Drive settings**.
    2.  Check the box: **Prevent viewers and commenters from downloading, copying, and printing files**.
    3.  Click Apply. This restricts data extraction for anyone who is not a Contributor, Content Manager, or Manager.

### Q50: Whitelisting External Domains for Drive Sharing
*   **Scenario:** Your company frequently partners with `partnercompany.com`. You want to allow employees to share Drive files with this domain, but block sharing with all other external domains.
*   **Question:** How do you set up this sharing boundary?
*   **Answer:**
    1.  Navigate to `Directory > Target audiences` (or use Trust Rules).
    2.  Create a **Whitelisted Domain List** under `Apps > Google Workspace > Drive and Docs > Sharing settings > Whitelisted domains`.
    3.  Add `partnercompany.com` to the list.
    4.  Under Drive sharing options, select **On - Files can be shared with whitelisted domains only**. Save the setting.

### Q51: Drive Trust Rules vs. Sharing Settings
*   **Scenario:** You need a complex sharing policy: the Marketing department can share files externally with anyone, but the Finance department can only share files internally.
*   **Question:** Why are standard sharing settings insufficient, and how do you use Trust Rules?
*   **Answer:**
    *   Standard sharing settings apply globally or by OU, but they lack granular context.
    *   **Trust Rules** (requires Enterprise editions) allow you to write detailed rules:
        *   Navigate to `Security > Access and data control > Trust Rules`.
        *   Create Rule 1: Scope to Finance OU > Restriction: Sharing to "External Users" is blocked.
        *   Create Rule 2: Scope to Marketing OU > Restriction: Sharing to "Anyone" is allowed.
    *   Trust Rules override general sharing settings and allow segmenting access by file type, sender, and receiver.

### Q52: Handling Deleted Files in Google Vault
*   **Scenario:** An employee deleted a critical Google Sheet from their Drive and emptied their trash. The company is now in a legal dispute, and you must recover the file.
*   **Question:** How do you retrieve this file using Google Vault?
*   **Answer:**
    1.  Log into Google Vault (`vault.google.com`).
    2.  Create a new **Matter** for the legal case.
    3.  Go to **Holds** and check if a Drive hold was active. If not, go to **Search**.
    4.  Select **Drive** as the service, enter the user's email, and add a search term or file name query.
    5.  Because Google Vault retains files based on your tenant's **Retention Rules** (even if the user purged their trash), the file will appear in the search results.
    6.  Select the file and click **Export** to download the document.

### Q53: Shared Drive Membership Limits
*   **Scenario:** You attempt to add a Google Group containing 60,000 users directly as a member of a Shared Drive, but the action fails.
*   **Question:** What limit have you hit, and how should you grant access?
*   **Answer:**
    *   **Limit:** Shared Drives have a direct member limit of **600 individuals** (or groups counted as members). However, groups with massive memberships can overload permission syncing.
    *   **Resolution:** Instead of adding the group directly to the Shared Drive membership, assign permissions using **Google Groups** mapped to target audience configurations, or divide the users into smaller target OUs and assign access via folders/drives, or share files via links restricted to the domain target audience.

### Q54: Finding Externally Shared Files (Investigation Tool)
*   **Scenario:** You need to generate a report of all Google Drive files currently shared with external users.
*   **Question:** How do you query this in the Investigation Tool?
*   **Answer:**
    1.  Open the **Investigation Tool**.
    2.  Select **Drive log events** as the data source.
    3.  Add conditions:
        *   `Visibility` = `Shared externally`
        *   `Date` = Within the target audit range.
    4.  Click Search. The results will display all files, owners, external collaborators, and access levels, allowing you to bulk-revoke permissions directly from the search results panel.

### Q55: Offboarding Drive File Transfers
*   **Scenario:** A manager wants to transfer ownership of all files created by a departed employee to a replacement team member, but they only want to transfer files created after a specific date.
*   **Question:** Can the standard Google Workspace account deletion flow do this? How else is it achieved?
*   **Answer:**
    *   **Limitation:** The default deletion flow is all-or-nothing; it transfers *all* Drive files.
    *   **Resolution:** Use the **Investigation Tool** or **GAM**:
        1.  In the Investigation Tool, search for Drive files where `Owner` = `departeduser@company.com` AND `Created Time` > `YYYY-MM-DD`.
        2.  Select all files in the results grid.
        3.  Click **Actions > Change Owner**.
        4.  Enter the email of the replacement employee.
        5.  Execute. This selectively transfers ownership of the target cohort.

### Q56: Preventing Google Forms External Submissions
*   **Scenario:** Employees are creating Google Forms that collect internal feedback, but the forms are accidentally exposed to the public internet.
*   **Question:** How do you force all newly created Google Forms to require domain sign-in by default?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Drive and Docs > Sharing settings`.
    2.  Scroll down to **Google Forms settings**.
    3.  Check the box: **Require users to sign in to their work account to access forms**.
    4.  Save. External users will now be blocked from accessing forms unless the form creator explicitly turns off the restriction in the individual form's settings.

### Q57: Auditing Google Drive Storage Giants
*   **Scenario:** Your tenant is running out of pooled storage. You need to identify which users are consuming the most storage in their personal Google Drive.
*   **Question:** Where do you find this information in the Admin Console?
*   **Answer:**
    1.  Navigate to `Storage` (the last menu item on your sidebar).
    2.  The Dashboard shows your total usage. Scroll to **Users using the most storage**.
    3.  Click **View all users** to see a ranked list of accounts with their specific Drive, Gmail, and Photos storage consumption.
    4.  You can select a user to view their files or apply a storage limit policy to their OU.

### Q58: Applying Storage Limits to specific OUs
*   **Scenario:** To control costs, you want to limit contractors to 15 GB of total storage, while keeping employees on unlimited storage.
*   **Question:** Where and how do you configure storage limits?
*   **Answer:**
    1.  Navigate to `Storage > Storage settings` (or via `Account > Settings > Storage`).
    2.  On the left panel, select the child OU **Contractors**.
    3.  Enable **User storage limit** and set the threshold to **15 GB**.
    4.  Ensure the root organization remains unrestricted or set to a higher tier. Click Save.

### Q59: Managing Visitor Sharing (PIN-Based Collaborators)
*   **Scenario:** You need to collaborate on Drive files with external partners who do not have Google Accounts.
*   **Question:** How do you enable secure collaboration for these users without forcing them to create Gmail accounts?
*   **Answer:**
    *   Enable **Visitor Sharing**.
    *   Go to `Apps > Google Workspace > Drive and Docs > Sharing settings`.
    *   Under sharing options, select **Allow users to share files with visitors (non-Google accounts)**.
    *   *How it works:* When an employee shares a file with a visitor's email, the visitor receives a link. Clicking it sends a temporary PIN to their inbox to verify their identity before granting secure, browser-based edit access.

### Q60: Blocking Google Docs Add-Ons
*   **Scenario:** You want users to write documents, but you want to prevent them from installing unapproved third-party add-ons (like grammar check utilities) inside Google Docs.
*   **Question:** How do you block this integration?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Drive and Docs > Features and Applications`.
    2.  Locate **Google Workspace Marketplace Add-ons**.
    3.  Select the target OU and change the setting to **Disable users from installing Google Workspace Marketplace Add-ons**.
    4.  Click Save.

---

## 📱 Part 5: Mobile Device & Endpoint Management (Q61-75)

### Q61: Mandatory iOS Push Certificate Renewal
*   **Scenario:** Your company manages 300 iPhones. The APNs (Apple Push Notification service) certificate is set to expire next week.
*   **Question:** What happens if the certificate expires before renewal, and what is the rule for renewal?
*   **Answer:**
    *   **Impact of Expiration:** If the certificate expires, Google Workspace MDM loses connection to Apple's push servers. You will be unable to sync policies, push apps, or wipe devices. You will have to re-enroll all 300 devices from scratch.
    *   **Rule for Renewal:** You must renew the certificate using the **exact same Apple ID** that was used to create it originally. Changing the Apple ID during renewal breaks the cryptographic chain, rendering existing device profiles invalid and requiring total re-enrollment.

### Q62: Bypassing iOS Screen Locks during Setup
*   **Scenario:** When deploying iOS MDM, you want to streamline the onboarding experience. You want to skip screens like Apple Pay setup and Siri configuration during the initial device boot.
*   **Question:** Where do you configure this?
*   **Answer:**
    1.  Navigate to `Devices > Mobile & Endpoints > Settings > iOS Settings > Apple device enrollment`.
    2.  Edit the enrollment profile.
    3.  Under **Setup Assistant Settings**, uncheck the boxes for screens you want to bypass (e.g., Apple Pay, Siri, Zoom, True Tone).
    4.  Save the profile.

### Q63: Android Enterprise System App Management
*   **Scenario:** You enroll Android devices in Advanced MDM. Users report that basic system apps (like the default camera or calendar) are missing from their Work Profiles.
*   **Question:** Why did this happen, and how do you restore them?
*   **Answer:**
    *   **Reason:** Google's Android Enterprise protocol automatically blocks all non-critical system applications by default within the Work Profile container to prevent data leaks.
    *   **Resolution:** 
        1.  Navigate to `Devices > Mobile & Endpoints > Settings > Android Settings > System Apps`.
        2.  Click **Manage System Apps**.
        3.  Search for the camera or system calendar package name, and select **Enable**. Save the setting to push the apps back to the work containers.

### Q64: Wiping Company Data vs. Device Wipe
*   **Scenario:** An employee leaves the company. They use their personal Android phone under Advanced MDM.
*   **Question:** If you click "Wipe Device", what is the risk, and what action should you perform instead?
*   **Answer:**
    *   **The Risk:** Clicking **Wipe Device** executes a factory reset, erasing the entire phone, including personal photos, apps, and accounts. This exposes the company to legal liability.
    *   **The Correct Action:** Perform an **Account Wipe** (or "Wipe Account") instead. This surgically removes only the corporate Workspace account, work profile container, and managed corporate applications, leaving the user's personal data completely untouched.

### Q65: Restricting Chrome Browser Extensions
*   **Scenario:** You want to prevent users from installing malicious or unapproved extensions in their Chrome Browsers.
*   **Question:** What policy configurations enforce this?
*   **Answer:**
    1.  Navigate to `Devices > Chrome > Settings > Users & browsers`.
    2.  Under **Apps and extensions**, configure the **Extension install sources** or use the **App and extension management** panel.
    3.  Set the default block policy: Set the **Block status** for the root store to **Block all other apps and extensions**.
    4.  Add approved extensions to the whitelist and set their policy to **Allow install**.

### Q66: Endpoint Verification Extension Deployment
*   **Scenario:** You need to enforce Context-Aware Access on all Windows/macOS endpoints, which requires the Endpoint Verification extension.
*   **Question:** How do you push the extension to all users' Chrome browsers automatically?
*   **Answer:**
    1.  Navigate to `Devices > Chrome > Settings > Users & browsers`.
    2.  Scroll to **Apps & extensions** and click the management panel.
    3.  Add the **Endpoint Verification** extension (ID: `jhifjgcakbdfacnlanmjofjhilhpggoc`) from the Chrome Web Store list.
    4.  Change its installation policy from "Allow install" to **Force install**.
    5.  Save. Chrome will silently install the extension on all endpoints when users log in.

### Q67: Blocking Unmanaged Chrome Browser Sign-ins
*   **Scenario:** To prevent data leaks, you want to prevent employees from signing in to their personal Gmail accounts inside the Google Chrome browser on work laptops.
*   **Question:** What policy setting forces this constraint?
*   **Answer:**
    1.  Navigate to `Devices > Chrome > Settings > Users & browsers`.
    2.  Locate the policy **Restrict sign-in to pattern** (`RestrictSigninToPattern`).
    3.  Set the regex pattern to `.*@yourcompany.com` (matching your verified domains).
    4.  Save. Chrome will block sign-ins from any account not matching the pattern.

### Q68: Forcing Chrome OS Device Enrollment
*   **Scenario:** A company buys 100 Chromebooks. You want to ensure that even if a user factory-resets a Chromebook, it will force-enroll back into the corporate directory upon startup.
*   **Question:** What setting must be configured?
*   **Answer:**
    1.  Navigate to `Devices > Chrome > Settings > Device Settings`.
    2.  Locate the **Forced re-enrollment** policy.
    3.  Select **Force device to re-enroll into this domain after wipe**.
    4.  Save. The Chromebook's hardware ID is registered with Google's servers; upon boot, it contacts Google and automatically forces corporate enrollment.

### Q69: Enterprise Wi-Fi Certificate Push
*   **Scenario:** You want your enrolled Android and iOS devices to automatically connect to the corporate secure Wi-Fi network without users entering the password.
*   **Question:** Where do you upload the network profile and security certificates?
*   **Answer:**
    1.  Navigate to `Devices > Networks`.
    2.  Click **Create Wi-Fi profile**.
    3.  Input the SSID, security type (e.g., WPA2 Enterprise), and upload the root CA certificate.
    4.  Scope the network profile to the OUs containing the mobile devices. Google MDM will silently push the network configuration and certificate to all enrolled endpoints.

### Q70: Geofencing Workspace Access via CAA
*   **Scenario:** Due to compliance, you must block users from accessing Google Workspace if they are physically located outside of the United States.
*   **Question:** How do you configure a Context-Aware Access level to enforce this?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > Context-Aware Access > Access Levels`.
    2.  Click **Create Access Level**.
    3.  Define the condition using the **IP Subnet** or **Geographic location** parameter:
        Set `Country` = `United States`.
    4.  Save. Assign this Access Level to your target applications (e.g., Gmail, Drive).

### Q71: Managing App Purchases via Managed Google Play
*   **Scenario:** You want to purchase and distribute a paid Android app to 50 employees' work profiles.
*   **Question:** How do you authorize and license this application?
*   **Answer:**
    1.  Navigate to `Apps > Web and mobile apps`.
    2.  Click **Add app > Search from Google Play**.
    3.  Approve the app under your organization's Managed Google Play Account.
    4.  Configure the licensing/distribution model, assign it to the target OU, and set the installation type to **Force install**.

### Q72: Endpoint Verification Helper App Deployment
*   **Scenario:** You deployed the Endpoint Verification Chrome extension on Windows laptops, but the Admin Console is still not reporting key hardware details like the device serial number or encryption status.
*   **Question:** What missing component is causing this, and how do you resolve it?
*   **Answer:**
    *   **Cause:** The Chrome extension alone cannot query local OS metrics (like disk encryption or serial numbers) due to browser sandbox limits. It requires the **Endpoint Verification Native Helper App** to be installed on the operating system.
    *   **Resolution:** Package and deploy the native helper installer (available from Google's documentation) to your endpoints using a software distribution tool (like SCCM, Intune, or Jamf).

### Q73: Troubleshooting iOS Profile Enrollment Fails
*   **Scenario:** A user trying to enroll their iPhone in Google MDM receives a `Profile Installation Failed - Connection to server could not be established` error on Safari.
*   **Question:** How do you troubleshoot this error?
*   **Answer:**
    1.  Verify that Apple's APNs certificate is valid and not expired in your Admin Console.
    2.  Ensure Safari is being used (third-party iOS browsers like Chrome or Firefox cannot install configuration profiles).
    3.  Check if the device has a pre-existing profile from a different MDM provider (only one management profile is allowed on iOS; remove existing profiles in *Settings > General > VPN & Device Management*).

### Q74: Enforcing Screen Lock Complexity
*   **Scenario:** You want to require all mobile users to have a minimum 6-digit numeric passcode or an alphanumeric password on their phones.
*   **Question:** Where do you configure this?
*   **Answer:**
    1.  Navigate to `Devices > Mobile & Endpoints > Settings > Universal Settings > General > Password requirements`.
    2.  Enable **Require password**.
    3.  Under passcode settings:
        *   Set **Minimum length** to `6`.
        *   Set **Password quality** to `Numeric` (or Alphanumeric).
    4.  Save the changes.

### Q75: Chrome Kiosk Mode Configuration
*   **Scenario:** You want to configure a set of Chromebooks to run in Kiosk Mode, displaying only a single web page (your company's check-in portal) with no browser address bar or settings menu.
*   **Question:** Where and how is this configured?
*   **Answer:**
    1.  Navigate to `Devices > Chrome > Devices`.
    2.  Switch the view to **Kiosk settings**.
    3.  Add the target URL of the check-in portal.
    4.  Configure the devices to automatically launch the kiosk app on boot.
    5.  Scope this configuration to the specific OU containing your kiosk Chromebooks.

---

## 📂 Part 6: APIs, Integrations, & Domain-Wide Delegation (Q76-85)

### Q76: Implementing Domain-Wide Delegation (DWD)
*   **Scenario:** A third-party archiving tool (Smarsh) needs to access and archive all users' Gmail mailboxes. The integration requires a GCP Service Account.
*   **Question:** How do you authorize this service account to access all mailboxes without giving it the Super Admin role?
*   **Answer:**
    Configure **Domain-Wide Delegation (DWD)**:
    1.  Navigate to `Security > Access and data control > API controls`.
    2.  Under Domain-wide delegation, click **Manage Domain Wide Delegation**.
    3.  Click **Add new**.
    4.  Enter the service account's **Client ID** (obtained from the GCP Console).
    5.  In the **OAuth Scopes** field, input the exact scopes required (e.g., `https://www.googleapis.com/auth/gmail.readonly`).
    6.  Click Authorize. The service account can now use Google APIs to impersonate any user in the domain for those scopes.

### Q77: Restricting API Access (Untrusted Apps)
*   **Scenario:** A user wants to use a mobile app that integrates with their Google Calendar. The app is not on your whitelisted applications list.
*   **Question:** How does the user experience this restriction, and how can you selectively authorize it?
*   **Answer:**
    *   **User Experience:** When attempting to log in, the user sees an error screen: `Access blocked: Your institution's admin has not reviewed this app`.
    *   **Admin Authorization:**
        1.  Go to `Security > Access and data control > API controls > Manage Third-Party App Access`.
        2.  Click **Add App > OAuth App Name or Client ID**.
        3.  Search for the blocked application, select it, and change its configuration to **Limited** or **Trusted**. Click Save.

### Q78: Revoking Compromised OAuth Tokens Globally
*   **Scenario:** A popular third-party PDF editor app is compromised by hackers. 50 of your users have previously authorized this app to access their Google Drives.
*   **Question:** How do you revoke this app's access for all users immediately?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > API controls > Manage Third-Party App Access`.
    2.  Locate the compromised application in your configured list.
    3.  Select the app, change its access configuration from "Trusted/Limited" to **Blocked**.
    4.  This instantly invalidates all active OAuth refresh tokens held by the application across your entire domain, shutting down data access immediately.

### Q79: Auditing Admin SDK Activity
*   **Scenario:** You suspect an administrator is abusing their privileges by querying the Directory API to extract employee personal phone numbers.
*   **Question:** Where do you audit these API calls?
*   **Answer:**
    1.  Navigate to the **Reports** section of the Admin Console, or open the **Investigation Tool**.
    2.  Select **Admin log events** as the data source.
    3.  Filter by **Event** = `API Call` or specifically query the Directory API method names (e.g., `users.list` or `users.get`).
    4.  Analyze the event log, which displays the admin's email, timestamp, parameters passed, and IP address.

### Q80: Google Workspace Marketplace Installation Controls
*   **Scenario:** You want to allow users to browse the Google Workspace Marketplace, but prevent them from installing any app unless the app has been reviewed and approved by IT.
*   **Question:** How do you configure this in the Admin Console?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace Marketplace apps > Settings`.
    2.  Under **Manage Marketplace Apps**, select **Allow users to install only white-listed applications from the Marketplace**.
    3.  To whitelist an app, find the app in the Marketplace, select **Admin Install**, but configure it to be available for installation rather than pushing it automatically to all users.

### Q81: Exporting Reports API Data to BigQuery
*   **Scenario:** You need to retain Workspace audit logs for 3 years for compliance. Google's default retention for reports is 6 months.
*   **Question:** How do you automate the export of audit data?
*   **Answer:**
    Set up the **BigQuery Export** integration:
    1.  Navigate to `Reporting > BigQuery Export` (or under Account Settings).
    2.  Click **Configure**.
    3.  Enter your GCP Project ID and select the BigQuery dataset where logs should be stored.
    4.  Google Workspace will now push all audit logs (Admin, Gmail, Drive, Login events) to BigQuery daily, where they can be retained indefinitely.

### Q82: Identifying Service Accounts with DWD (Audit)
*   **Scenario:** An auditor wants a list of all GCP service accounts that currently have Domain-Wide Delegation authorization on the tenant.
*   **Question:** Where is this list exported?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > API controls`.
    2.  Click **Manage Domain Wide Delegation**.
    3.  The console displays a table listing all authorized Client IDs, the associated Service Account emails, and the specific OAuth Scopes delegated to each client. You can copy or download this list.

### Q83: Restricting API Access to Google Calendar
*   **Scenario:** You want to restrict external API integrations from querying your meeting room resources, while allowing them to query individual user calendars.
*   **Question:** How do you segment this access?
*   **Answer:**
    1.  Navigate to `Security > Access and data control > API controls > Manage Google Services`.
    2.  Locate **Google Calendar**.
    3.  Set the service access to **Restricted**.
    4.  Under calendar settings, specifically restrict resource calendar visibility to internal users only, preventing external OAuth clients from reading resource availability.

### Q84: Enforcing App Access Control for Scripting (App Sheet/Apps Script)
*   **Scenario:** Users are writing Google Apps Scripts that access external web services. You want to block these scripts from running external fetch calls.
*   **Question:** Where is this blocked?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Drive and Docs > Features and Applications`.
    2.  Locate **Apps Script settings**.
    3.  Configure policies to disable external API fetch options or restrict execution scopes, enforcing compliance over script behaviors.

### Q85: Automating User Provisioning via SCIM Token Expirations
*   **Scenario:** Your Okta-to-Google user provisioning fails. The error log says `Unauthorized / Token Expired`.
*   **Question:** How do you resolve this?
*   **Answer:**
    *   **Cause:** The OAuth credential token used by Okta to authenticate to Google's Directory API has expired or been revoked.
    *   **Resolution:** Log into the Google Admin Console under the administrative account, regenerate the provisioning token, copy the token secret, paste it into Okta's provisioning settings, and test the integration to restart sync.

---

## 📂 Part 7: Data Migrations & Coexistence (Q86-92)

### Q86: Selecting the Correct Migration Tool for File Servers
*   **Scenario:** You need to migrate 20 TB of unstructured file server data (folders, permissions, active directories) to Google Shared Drives.
*   **Question:** Which Google tool is designed for this, and what is the architecture?
*   **Answer:**
    *   **Tool:** **Google Workspace Migrate**.
    *   **Architecture:** It requires deploying a multi-node infrastructure:
        1.  A dedicated Windows Server running the Google Workspace Migrate platform.
        2.  One or more Node databases (SQL/CouchDB) to track file chunks and metadata.
        3.  The tool connects to your on-premises file servers, parses NTFS permissions, maps them to Google's sharing model, and uploads the data in parallel to Google Drive/Shared Drives.

### Q87: Basic IMAP Migration (Data Migration Service)
*   **Scenario:** You are migrating 100 mailboxes from a generic IMAP provider to Gmail. You want a simple, built-in tool that doesn't require server installations.
*   **Question:** What tool do you choose, and what is the required configuration?
*   **Answer:**
    *   **Tool:** **Data Migration Service (DMS)**, which is built directly into the Admin Console.
    *   **Configuration:**
        1.  Go to `Data > Data migrations`.
        2.  Select **Email** as the migration type.
        3.  Select **IMAP** as the source, input the IMAP server host address, and choose an admin credential for authentication.
        4.  Upload a CSV containing your source email addresses, target Google email addresses, and source passwords.
        5.  Click Start. DMS pulls the email data directly over the web.

### Q88: GWMME vs. GWMMO
*   **Scenario:** A manager has 50 GB of archived emails stored in local PST files on their desktop. They need these imported into Gmail.
*   **Question:** Do you use GWMME or GWMMO? Why?
*   **Answer:**
    *   Use **GWMMO (Google Workspace Migration for Microsoft Outlook)**.
    *   **Reasoning:** GWMMO is a client-side utility designed for individual users to run on their local Windows desktops. It imports data (PSTs, calendar, contacts) directly from local Outlook profiles into their personal Google account. GWMME (Google Workspace Migration for Microsoft Exchange) is a server-side utility designed for admins to perform bulk migrations from Exchange servers or folders.

### Q89: Coexistence Calendar Interop Setup
*   **Scenario:** During a phased migration, users on Google Calendar need to view free/busy availability for users still on Microsoft Exchange, and vice versa.
*   **Question:** What feature do you configure, and what is the architecture?
*   **Answer:**
    Configure **Calendar Interop**:
    1.  Navigate to `Apps > Google Workspace > Calendar > Calendar Interop`.
    2.  Set up the **Exchange web services (EWS)** endpoint URL in Google Workspace, providing credentials for an Exchange service account.
    3.  In Microsoft Exchange, configure an availability address space pointing to Google's interop endpoint (`gcal.yourdomain.com`).
    4.  Workspace and Exchange will now query each other's free/busy lookup APIs in real-time when scheduling meetings.

### Q90: Handling PST Files at Scale (GWMME)
*   **Scenario:** You have 5,000 PST files stored on a central network share. You need to import them into Google Workspace.
*   **Question:** What is the migration flow using GWMME?
*   **Answer:**
    1.  Install the **GWMME command-line utility** on a central migration server.
    2.  Create a mapping CSV file associating the local PST file path (e.g., `\\share\pst\user1.pst`) with the corresponding Google email address.
    3.  Run GWMME in command-line mode using a batch script, pointing it to the mapping file, your authorized Google service account key file, and the target scopes.
    4.  GWMME runs the imports in parallel, uploading the mail, calendar, and contact data to Gmail.

### Q91: Tenant-to-Tenant Workspace Migrations
*   **Scenario:** Your company merges with another company that already uses Google Workspace. You want to merge their Google Workspace tenant into your tenant.
*   **Question:** Can you merge them directly via the Admin Console? What tool or partner solution is required?
*   **Answer:**
    *   **Direct Merge:** No, there is no built-in native tool in the Admin Console to merge two live Google Workspace tenants.
    *   **Resolution:** You must use an enterprise migration platform like **CloudM** or **BitTitan MigrationWiz**. You configure API access/Service Accounts on both tenants, map the users, migrate the Drive and Mail data over the web, and then update the DNS records to redirect the custom domains to the primary target tenant.

### Q92: Mitigating Mail Loops during Migrations
*   **Scenario:** During split delivery setup, a user on Google Workspace sends an email to a user on Exchange, but the email loops back and forth infinitely, generating errors.
*   **Question:** What configuration mistake caused this, and how do you resolve it?
*   **Answer:**
    *   **Cause:** The routing rule in Google Workspace points to Exchange, and the routing rule in Exchange points back to Google Workspace for the same email address. Neither server recognizes the user as local, creating a mail loop.
    *   **Resolution:** Ensure that the destination routing profile in Google Workspace points to a **non-routing subdomain** (e.g., `user@exchange.company.com`) or a direct IP that does not cause MX record queries, and configure Exchange to accept mail for that subdomain as local.

---

## 🛡️ Part 8: Security Center, Investigation Tool, & Auditing (Q93-97)

### Q93: Phishing Remediations (Investigation Tool)
*   **Scenario:** A dangerous phishing email containing malware has successfully landed in 500 employee inboxes.
*   **Question:** How do you locate and delete this email from all inboxes immediately?
*   **Answer:**
    1.  Navigate to the **Investigation Tool** under `Security`.
    2.  Set data source to **Gmail messages**.
    3.  Add conditions to match the phishing email: `Subject` = `Target Subject` OR `Sender` = `phisher@malicious.com` OR `Attachment SHA256 Hash` = `MalwareHash`.
    4.  Click Search to retrieve the list of matching emails.
    5.  Select all messages, click **Actions > Delete Messages**.
    6.  Confirm. Google Workspace will delete the emails from the users' inboxes and trash folders in bulk.

### Q94: Tracking Unauthorized Admin Actions
*   **Scenario:** An administrator changed the password for the CFO's account without authorization.
*   **Question:** Where do you find the log showing which admin performed this action and when?
*   **Answer:**
    1.  Navigate to the **Investigation Tool** or **Admin Reports**.
    2.  Set data source to **Admin log events**.
    3.  Add conditions:
        *   `Event` = `Change User Password`
        *   `Target User` = `cfo@company.com`
    4.  Search. The log will display the executing admin's email, their IP address, and the exact timestamp of the password modification.

### Q95: Google Workspace Security Health Score
*   **Scenario:** You want to audit your tenant configuration against Google's security best practices (e.g., checking if password policies are strong, or if less secure apps are blocked).
*   **Question:** Where in the console do you find a structured checklist and recommendations?
*   **Answer:**
    Navigate to `Security > Security health`. The console displays a list of security settings with colored icons (Green = Secure, Yellow = Warning, Red = Action Required). You can review Google's recommendations for each setting and click **Update** to apply best practices directly.

### Q96: Detecting Mass File Downloads (Security Alert)
*   **Scenario:** You want to be alerted immediately if a user downloads more than 100 documents from a Shared Drive within 10 minutes.
*   **Question:** How do you configure this alert?
*   **Answer:**
    1.  Navigate to `Rules` (or `Security > Rules`).
    2.  Click **Create Rule > Activity**.
    3.  Select **Drive log events** as the source.
    4.  Set condition: `Event` = `Download` and specify threshold: `100 events` in `10 minutes`.
    5.  Set action: **Send email alert to administrators** and **Send alert to Alert Center**.
    6.  Save the rule.

### Q97: Auditing Google Meet Attendance & Quality
*   **Scenario:** An executive complains that their Google Meet calls are frequently dropping.
*   **Question:** Where do you audit the call quality, network packet loss, and participants list?
*   **Answer:**
    1.  Navigate to `Apps > Google Workspace > Google Meet > Meet Quality Tool`.
    2.  Search for the meeting ID or the executive's email address.
    3.  Select the target meeting. The tool displays graphs showing CPU utilization, jitter, packet loss, bandwidth, audio/video levels, and participant join/leave times, allowing you to isolate network bottlenecks on the client or provider side.

---

## 📂 Part 9: DR, Billing, & Account Configuration (Q98-100)

### Q98: Enterprise Billing License Assignments
*   **Scenario:** Your company has 1,000 users. You purchased 900 Enterprise Standard licenses and 100 Business Starter licenses. You want to ensure new hires do not automatically consume the expensive Enterprise licenses unless explicitly assigned.
*   **Question:** How do you manage this licensing behavior?
*   **Answer:**
    1.  Navigate to `Billing > License settings`.
    2.  Locate the default **Auto-assignment** settings.
    3.  Disable auto-assignment for **Enterprise Standard**.
    4.  Enable auto-assignment for **Business Starter** (or set it to "Disabled" globally).
    5.  When creating a new user, manually select and assign the target license, or use an OU-based licensing template to push specific licenses automatically to specific Organizational Units.

### Q99: Disaster Recovery DNS Switchovers
*   **Scenario:** Your company's primary domain DNS is hosted on GoDaddy. A catastrophic outage impacts GoDaddy's nameservers, rendering your DNS records inaccessible.
*   **Question:** How do you maintain mail flow continuity for Gmail, and what backup records should you have configured?
*   **Answer:**
    *   **Continuous Mail Flow:** Gmail itself is unaffected because it runs on Google's infrastructure. However, external senders cannot resolve your MX records, so mail will queue on their servers (usually for 48–72 hours).
    *   **Mitigation:** You must register a secondary, backup DNS provider (like Cloudflare or AWS Route 53) using secondary nameservers. In an emergency, login to your domain registrar and update the primary nameservers to point to the backup DNS provider where identical MX, SPF, and DKIM records are pre-configured.

### Q100: Multi-Domain Architecture Directory Visibility
*   **Scenario:** Your parent company owns `subsidiaryA.com` and `subsidiaryB.com`, both configured as secondary domains inside the same Google Workspace tenant. You want to prevent Subsidiary A users from viewing Subsidiary B users in the Directory directory search.
*   **Question:** How do you isolate the directories of two domains sharing the same tenant?
*   **Answer:**
    1.  Go to `Directory > Directory settings > Sharing settings`.
    2.  Set the default directory visibility to **Custom directories**.
    3.  Create two custom directories:
        *   `Directory A`: Add a search filter matching user emails containing `@subsidiaryA.com`.
        *   `Directory B`: Add a search filter matching user emails containing `@subsidiaryB.com`.
    4.  Assign the custom directories: Set the Subsidiary A OU to see only `Directory A`, and the Subsidiary B OU to see only `Directory B`.
    5.  Save. This isolates the internal contact search, completing the directory segregation.
