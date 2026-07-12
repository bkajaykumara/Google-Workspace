# Google Workspace Admin: Quick Reference Q&A Handbook

This handbook provides direct, authoritative answers to the specific Google Workspace Admin interview questions and troubleshooting scenarios compiled in your query.

---

## 📑 General Settings, Priorities, & Group Behaviors

### Q1: Google Vault: Default vs. Custom Retention Rules
*   **Question:** In Google Vault, if you set a default retention rule to delete data after 1 year, but set a custom retention rule to keep data for 5 years, which one takes priority?
*   **Answer:** **The custom retention rule overrides the default retention rule.** 
    *   **The Rule:** Custom retention rules always take precedence over default retention rules, regardless of whether the custom rule's retention period is longer or shorter than the default.
    *   *Note:* If a user account or folder is placed on a **Hold**, the hold overrides all retention rules (both default and custom) and prevents any deletion.

### Q2: Policy Priority: Google Groups vs. Organizational Units (OUs)
*   **Question:** In Google Workspace, which has higher priority for settings and policies: Google Groups or OUs?
*   **Answer:** **Organizational Units (OUs) take precedence for core application settings.**
    *   **OU Scope:** App availability (turning Gmail, YouTube, or Drive ON/OFF), session lengths, and core security settings are applied strictly to OUs. You cannot apply these settings directly to a standard Google Group.
    *   **Group Exceptions:** For specific features like **Drive Trust Rules**, **Chrome Policies**, or **Email Routing bypasses**, you can use Google Groups to create exceptions or override the general OU policy.

### Q3: Retaining Data After User Deletion in Vault
*   **Question:** If you delete a user account from the Admin Console, will you still be able to search and retain their data in Google Vault?
*   **Answer:** **No.** 
    *   **The Rule:** Deleting a user account permanently purges all data associated with that account from Google Workspace, including Google Vault. Vault cannot index or retain data for a user identity that no longer exists.
    *   *Solution:* To retain a departed employee's data in Vault without paying for a full active license, assign an **Archived User (AU) license** to their suspended account instead of deleting them.

### Q4: Suspended User License Consumption
*   **Question:** If you suspend a user, is the Google Workspace license still consumed by that account?
*   **Answer:** **Yes.** Suspended users continue to consume Google Workspace licenses and will be billed at the standard rate. To free up the license, you must either delete the user account or downgrade them to an **Archived User (AU)** license tier.

### Q5: Offboarding Users Without External Server Downloads
*   **Question:** You need to retain or revoke access to some users while offboarding them, but you do not want to download their data to another external server. What is the best method?
*   **Answer:** 
    1.  **Change the password** and **reset sign-in cookies** for the user account to revoke their access instantly.
    2.  Suspend the user account to block future login attempts.
    3.  Assign an **Archived User (AU)** license to the account. This freezes the data in place (Gmail, Drive, Vault) for security and legal reviews, keeping it in Google's cloud at a lower cost without requiring local downloads or migration scripts.

### Q6: Security Groups for License Assignment
*   **Question:** Can you use Google Security Groups to apply licenses to users?
*   **Answer:** **No.** You cannot apply Google Workspace licenses directly to Google Groups. Licensing is assigned at the individual user level or auto-assigned by **Organizational Unit (OU)** under `Billing > License settings`.

### Q7: Types of Groups Available in Google Workspace
*   **Question:** What types of Google Groups can you create?
*   **Answer:**
    1.  **Email Distribution Groups:** Used to broadcast emails to multiple recipients.
    2.  **Collaborative Inboxes:** Allow teams (e.g., `support@`) to assign, track, and manage incoming group emails as tasks.
    3.  **Security Groups:** Labeled Google Groups used for applying administrative roles, resource access permissions, or whitelisting.
    4.  **Dynamic Groups:** Groups whose membership is automatically updated based on user directory attributes (e.g., department, location).

---

## 🛡️ Authentication & Access Controls

### Q8: Requirements for Configuring Context-Aware Access (CAA)
*   **Question:** What are the prerequisites and requirements before you can create and enforce Context-Aware Access levels?
*   **Answer:**
    1.  **Licensing:** Users must have a Google Workspace **Enterprise** edition (Enterprise Standard/Plus) or a **Cloud Identity Premium** license.
    2.  **Endpoint Verification:** The Endpoint Verification extension must be force-installed in Chrome browsers, and the Native Helper app must be deployed to Windows/macOS endpoints.
    3.  **Context-Aware Access Enabled:** Turn on CAA globally under `Security > Access and data control > Context-Aware Access`.

### Q9: SSO (Single Sign-On) Configuration Steps
*   **Question:** How do you configure Single Sign-On (SSO) with a third-party Identity Provider (IdP)?
*   **Answer:**
    1.  Navigate to `Security > Authentication > Set up single sign-on (SSO) with a third party IdP`.
    2.  Check **Set up SSO with third-party identity provider**.
    3.  Input the **Sign-in page URL** and **Sign-out page URL** provided by your IdP (e.g., Okta, Entra ID).
    4.  Upload the IdP’s public **Verification Certificate** (x.509 format) to verify SAML signatures.
    5.  Download Google's metadata file or copy the Google ACS URL, and configure the corresponding Google Workspace integration app inside the IdP console.

---

## 📧 Mail Flow & Gateway Troubleshoots

### Q10: Contractor Mail Marked as Spam
*   **Question:** A critical external contractor is sending emails to your domain, but they are consistently landing in spam. What actions do you take?
*   **Answer:**
    *   **Temporary Fix:** Navigate to `Apps > Google Workspace > Gmail > Spam, Phishing and Malware`. Locate **Email Allowlist** and input the contractor mail server's sending IP addresses. This prevents Gmail from marking their mail as spam, though users may still see safety warning banners if the contractor's DNS records are invalid.
    *   **Long-Term Fix:** Add the contractor’s domain/email addresses to your **Approved Senders List** under your custom Spam settings, and instruct the contractor to fix their SPF/DKIM records.

### Q11: Calendar Invites Failing on Group Distribution Lists (DLs)
*   **Question:** Users send calendar invitations to a group email (e.g., `team@company.com`), but the members of the group are not receiving the invites, and they are not appearing on their calendars. Why?
*   **Answer:**
    1.  **Access Permissions:** The Google Group settings (configured in `groups.google.com`) might be restricting external senders from sending mail to the group. If the calendar invite originates from an external domain, the email is blocked.
    2.  **Auto-Accept Setting:** In the Admin Console under `Apps > Google Workspace > Calendar > Sharing settings`, check if **Allow users to invite Google Groups to events** is disabled.
    3.  **Subscription Delivery Settings:** Group members might have configured their group subscriptions to digest mode, preventing real-time receipt of direct `.ics` calendar invitation emails.

### Q12: Quarantine Message Lifespan (Gmail)
*   **Question:** How many days will a quarantined message remain in the administrative queue before it is automatically deleted?
*   **Answer:** **30 days.** If an administrator does not release, reject, or delete a message from the quarantine queue within 30 days, Google Workspace automatically purges it.

### Q13: Troubleshooting Split Delivery Mail Delivery Failure
*   **Question:** You configured split delivery to route mail between Google and an on-premises Exchange server, but emails are not being delivered. How do you troubleshoot this?
*   **Answer:**
    1.  **Check Email Log Search:** Search the logs to see where Gmail routed the message and the SMTP response code returned by the destination host.
    2.  **Verify Host Configurations:** Go to `Gmail > Routing > Hosts`. Ensure the IP address or hostname of your Exchange server is correct and accessible over port 25 or 587.
    3.  **Check for Loops:** If the Exchange server does not recognize the recipient domain as local, it will query public MX records and send the mail back to Google, causing an infinite loop. Verify that the destination Exchange server has the domain set as an "Internal Relay" and not "Authoritative".

### Q14: Soft Fail vs. Hard Fail in SPF DNS Records
*   **Question:** Explain the difference between `~all` and `-all` in an SPF record.
*   **Answer:**
    *   **Soft Fail (`~all`):** Recommends that the receiving server accept the email but flag it as suspicious (e.g., routing it to the Spam folder or adding a header warning) if it originates from an IP not listed in the SPF record.
    *   **Hard Fail (`-all`):** Recommends that the receiving server reject/drop the email entirely if the sending IP is not authorized in the SPF record.

---

## 💻 Vault, Storage, & Identity Sync

### Q15: Is Google Vault a Backup Tool?
*   **Question:** Can you use Google Vault as a backup? What type of data does it store, and how is it used?
*   **Answer:**
    *   **No, Google Vault is NOT a backup tool.** It is an archiving, eDiscovery, and compliance tool. 
    *   **The Difference:** A backup tool is designed to restore data back to its original state/location (e.g., restoring a deleted calendar event directly to a user's calendar). Vault preserves data in place for legal holds and search/export. 
    *   **Data Stored:** Gmail, Google Drive files, classic Hangouts and Google Chat messages, Google Groups messages, Google Meet recordings, and Voice logs.
    *   **How it is used:** Admins use Vault to place legal holds, perform keyword searches across the entire domain, and export matching data for legal counsel.

### Q16: How Google Cloud Directory Sync (GCDS) Works
*   **Question:** What is GCDS, and how does it synchronize data under the hood?
*   **Answer:**
    *   **Definition:** GCDS is a desktop-based sync utility (run on-premises or on an VM) used to synchronize users, groups, aliases, and profiles from an LDAP server (like Microsoft Active Directory) to Google Workspace.
    *   **Sync Flow:** GCDS reads the LDAP directory and queries your Google Workspace directory. It performs a comparison in memory, generates a list of differences, and makes API writes directly to Google Workspace via the Admin SDK to create, update, or suspend accounts.
    *   *Note:* GCDS is a **one-way sync** (LDAP to Google). It never writes changes back to your Active Directory.

### Q17: Difference Between My Drive and Shared Drive
*   **Question:** What is the primary difference in file ownership and permissions between My Drive and Shared Drive?
*   **Answer:**
    *   **My Drive:** Files are owned by the **individual creator**. If the employee leaves the company and their account is deleted, the files are deleted, causing data loss. Sharing permissions are applied directly at the file or folder level.
    *   **Shared Drive:** Files are owned by the **organization/tenant**. Even if the file creator leaves and their account is deleted, the file remains in the Shared Drive, ensuring business continuity. Membership permissions are applied at the Shared Drive level, guaranteeing consistent access policies for all nested files.

### Q18: DNS & HTTP: What Happens When You Click `example.com`?
*   **Question:** Explain the backend network process when a user clicks a URL like `example.com`.
*   **Answer:**
    1.  **Browser Cache Check:** The browser checks its local cache for the IP address of `example.com`. If missing, it queries the OS DNS resolver.
    2.  **DNS Resolution:** The resolver queries the local recursive DNS server (provided by the ISP or public resolvers like `8.8.8.8`). If the resolver does not have the record, it queries the Root Name Server (`.`), the TLD Name Server (`.com`), and finally the Authoritative Name Server for `example.com` to retrieve the `A` record containing the IP address (e.g., `93.184.216.34`).
    3.  **TCP Handshake:** The browser initiates a TCP 3-Way Handshake (SYN -> SYN-ACK -> ACK) with the web server IP on port 80 (HTTP) or 443 (HTTPS).
    4.  **TLS Handshake:** For HTTPS, the client and server negotiate keys, verify the SSL certificate, and establish an encrypted session.
    5.  **HTTP Request/Response:** The browser sends an `HTTP GET /` request. The server processes the request and sends back the HTML webpage response.
