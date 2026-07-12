# Google Workspace Administrator Practice Handbook & Cheat Sheet

This handbook synthesizes the configuration procedures, console navigation paths, step-by-step exercises, and core security concepts required for the Google Workspace Administrator examination and practical interviews.

---

## 📂 Section 1: Security Groups, Admin Roles, & Access Control

### 1. Security Groups
*   **Definition:** A special type of Google Group used to control access to sensitive files, Shared Drives, and Admin Console roles. 
*   **Enforcement Rules:**
    *   Created by adding the permanent `"Security"` label to a Google Group.
    *   **Nesting Rule:** Only a security group in the same organization can join another security group. Any group joining must have the same or more restrictive membership settings.
    *   Prevents unauthorized membership changes, securing access propagation.

### 2. Administrator Roles
*   **Prebuilt Roles:** Super Admin, User Management Admin, Groups Admin, Help Desk Admin, Services Admin, Mobile Admin.
*   **Custom Roles:** Created for granular control (e.g., granting user password reset rights only, without profile edit rights) to enforce the **Principle of Least Privilege**.

### 3. Session Length Controls
*   **Behavior:** Forces users to re-authenticate after a specified duration (e.g., 4 hours, 24 hours, or 14 days) to prevent session hijacking.
*   **Targeting:** Elevated privilege accounts (admins, billing managers) should be forced to re-authenticate more frequently than regular users.

---

## 📱 Section 2: Mobile Device & Endpoint Management (MDM)

### 1. Basic vs. Advanced Mobile Management
*   **Basic (Agentless):** On by default. Supports screen lock/passcode enforcement, **Account Wipe** (removes work account only), and basic Android app management.
*   **Advanced (Agent-based):** Requires iOS Apple Push Notification (APNs) certificate and Android Device Policy app. Enables device encryption requirements, lock screen notification blocks, **Full Device Wipe**, and **Device Approvals**.

---

### 🛠️ Practical Exercises: Mobile Management

#### Exercise A: Importing Company-Owned Devices to Inventory
1.  Navigate to `Devices > Mobile & Endpoints > Company-owned inventory`.
2.  Click the **+ (Import)** icon.
3.  Select **Android** (or iOS) from the dropdown and download the CSV import template.
4.  Add device details (Format: `SerialNumber,AssetTag`):
    ```csv
    R84534UIF824-123-234,W123
    U83452BDU424-232-694,W442
    ```
5.  Upload the CSV file in the console and click **Import**.

#### Exercise B: Configuring Basic Mobile Management
1.  Navigate to `Devices > Mobile & Endpoints > Settings > Universal Settings`.
2.  Click **General > Mobile management**.
3.  Select **Basic (agentless)** and click **Save**.

#### Exercise C: Wiping Corporate Data from a Lost/Returned Device
*Note: A device must be assigned to a user (via their work account login) before it can be wiped.*
1.  Navigate to `Directory > Users`.
2.  Click on the target user (e.g., *Mark*).
3.  Click **Managed devices**.
4.  Select the target device, click **More (...)**, and select **Wipe Account**.
5.  *For Android:* Check **Also remove factory reset device protection** to ensure the phone can be set up fresh without prompting for the previous owner's credentials.
6.  Click **Wipe Account** to confirm. (Sync occurs within minutes; offline devices wipe immediately when they reconnect).

#### Exercise D: Enabling Device Approvals (Advanced MDM)
1.  Turn on Advanced MDM: `Devices > Mobile & Endpoints > Settings > Universal Settings > General > Mobile management`, select **Advanced** for Android/iOS, and save.
2.  Enforce Approvals: Navigate to `Universal Settings > Security > Device approvals`.
3.  Check **Require admin approval**.
4.  Under **Send approval request emails to**, input the designated IT Manager's email address (e.g., `alex.b@yourdomain.com`).
5.  Click **Save**.

---

## 🌐 Section 3: Chrome Browser Policy Management

### 1. Endpoint Verification
*   A Chrome extension that reports OS version, serial number, device ID, encryption status, and compliance state to the Admin Console.
*   Required for enforcing Context-Aware Access (Zero Trust) policies on Windows, macOS, and Linux endpoints.

### 🛠️ Practical Exercise: Chrome Browser Policies
*Goal: Display the Gemini introduction screen to users upon browser sign-in.*
1.  Navigate to `Devices > Chrome > Settings`.
2.  Under the **User & browser settings** tab, scroll to **Sign-in settings**.
3.  Click **Gemini introduction during sign-in**.
4.  Change Configuration to: **Display the Gemini introduction screen during sign-in**.
5.  Click **Save**.

---

## 📧 Section 4: Email Security Practices (SPF, DKIM, DMARC)

To prevent spoofing, phishing, and domain forgery, implement all three records in your public DNS settings:

```
  ┌────────────────────────────────────────────────────────────┐
  │                    EMAIL SECURITY TRIAD                    │
  ├─────────┬───────────────────┬──────────────────────────────┤
  │ Record  │ DNS Type & Prefix │ Primary Purpose              │
  ├─────────┼───────────────────┼──────────────────────────────┤
  │ SPF     │ TXT (@ or blank)  │ Allowlist of sending IPs     │
  ├─────────┼───────────────────┼──────────────────────────────┤
  │ DKIM    │ TXT (selector._dk)│ Cryptographic signature key  │
  ├─────────┼───────────────────┼──────────────────────────────┤
  │ DMARC   │ TXT (_dmarc)      │ Action policy on fail status │
  └─────────┴───────────────────┴──────────────────────────────┘
```

### 🛠️ Practical Exercise: DNS Record Deployment

#### Part 1: Verify & Deploy SPF
*   **Rule:** A domain must have only **one** SPF record to prevent resolution conflicts.
*   **Google Standard Record Value:**
    *   **Host:** `@` or blank
    *   **Type:** `TXT`
    *   **Value:** `v=spf1 include:_spf.google.com ~all` (where `~all` represents SoftFail; change to `-all` / HardFail after auditing).

#### Part 2: Generate & Enable DKIM
1.  Navigate to `Apps > Google Workspace > Gmail > Authenticate email`.
2.  Select your domain and click **Generate New Record** (use default prefix selector `google`).
3.  Copy the TXT record Name (`google._domainkey`) and Value.
4.  Add this `TXT` record to your DNS Registrar.
5.  Wait for propagation (up to 48 hours), return to the Admin Console, and click **Start Authentication**.

#### Part 3: Deploy DMARC Policy
1.  Create a TXT record at your DNS provider:
    *   **Host:** `_dmarc.yourdomain.com`
    *   **Value:** `v=DMARC1; p=none; rua=mailto:admin-email@yourdomain.com`
    *   *Note on Policies (`p`):* Start with `none` (monitor/report), then escalate to `quarantine` (deliver to spam), and finally `reject` (drop completely).
2.  Use the **Google Admin Toolbox** (`toolbox.googleapps.com/apps/checkmx/`) to run checks and confirm SPF, DKIM, and DMARC validity.

---

## 🛡️ Section 5: Email Content Filtering & Spam Policies

### 1. Compliance Rule Scope
Compliance rules scan email metadata, body, and attachments. Actions include **Reject**, **Modify** (add headers, alter subject, append compliance footer), or **Quarantine**.

### 🛠️ Practical Exercise: Project Compliance Filtering (Rerouting Data)
*Goal: Prevent non-executives from emailing references to "Project Jupiter". Redirect matching external emails to the security officer.*
1.  Navigate to `Apps > Google Workspace > Gmail > Compliance`.
2.  Find **Content compliance** and click **Configure** (or **Add Another Rule**).
3.  **Description:** `Secure Project Jupiter`
4.  **Email messages to affect:** Check **Outbound**.
5.  **Expressions:**
    *   Click **Add**. Change type to **Advanced content match**.
    *   **Location:** `Body`
    *   **Match type:** `Contains text`
    *   **Content:** `Jupiter` (Click Save).
6.  **Action:** Under "If the expressions match," choose **Modify message**.
7.  **Envelope Recipient:** Check **Change envelope recipient** and enter the auditor's address (e.g., `sam.m@yourdomain.com`).
8.  **Bypass Scoping (Address Lists):**
    *   Click **Show options** at the bottom.
    *   Check **Use address lists to bypass or control application of this setting**.
    *   Select **Bypass this setting for specific addresses / domains**.
    *   Click **Create or edit list** -> Add Address List named `Executive`.
    *   Add emails of bypass users: `sam.m@`, `alex.b@`, `izumi.e@`, `timothy.l@`.
    *   Save, return to compliance settings, click **Use existing list**, and select `Executive`.
9.  Click **Save** to enable the rule.

---

### 🛠️ Practical Exercise: Advanced Spam & Inbound Gateways

#### Part 1: Approved Senders & Quarantine Rules
1.  Navigate to `Apps > Google Workspace > Gmail > Spam, Phishing and Malware`.
2.  On the **Spam** row, click **Configure**.
3.  **Setting Name:** `Approved senders only`
4.  **Options:**
    *   Check: **Be more aggressive when filtering spam**.
    *   Check: **Put spam in administrative quarantine** (so admins can audit instead of delivering to user spam folders).
5.  **Bypass Rules:** Check **Bypass spam filters for messages from senders in selected lists**.
6.  Create a list, select **Bulk Add Addresses**, and input trusted domains/addresses. Save the list and assign it.
7.  Click **Save**.

#### Part 2: Inbound Gateway Configuration
*An inbound gateway is an external server that filters mail before relaying it to Google Workspace.*
1.  On the `Spam, Phishing and Malware` settings page, select the target Organizational Unit.
2.  Click **Inbound gateway** and check **Enable**.
3.  Click **Add** under **IP address/range** and input your gateway's IP block (e.g., RFC 5737 test block `192.0.2.0/24`).
4.  Click **Save**.
