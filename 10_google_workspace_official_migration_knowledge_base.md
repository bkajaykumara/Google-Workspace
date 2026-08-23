# Module 10: Google Workspace Official Migration Knowledge Base & Technical Guide

> **Source:** Consolidated from Google Workspace Official Admin Migration Documentation ([knowledge.workspace.google.com/admin/migrate/](https://knowledge.workspace.google.com/admin/migrate/))  
> **Target Audience:** Senior Systems Engineers, Enterprise Cloud Architects, and Workspace Administrators  
> **Coverage:** Data Import Tool (New DMS), Advanced Data Import (Exchange, Teams, OneDrive, SharePoint, Dropbox), Google Workspace Migrate, GWMME, GWMMO, Azure/Entra ID Application Registrations, Delta Syncs, Error Codes & Diagnostics.

---

## 1. Product Matrix: Choosing the Right Google Migration Tool

Google provides several native migration tools depending on your source environment, data scale, user count, and deployment requirements:

| Source Environment | Migration Tool | Installation / Deployment | Workloads Supported | Max Scale / Best For |
| :--- | :--- | :--- | :--- | :--- |
| **Microsoft 365 (Exchange Online)** | **Data Import Tool (New DMS)** | Cloud-native (Admin Console) | Email, Calendar, Contacts, Tasks | Turnkey cloud-to-cloud migration (Default: 1–1,000; Advanced: up to 50,000 users in 10 batches) |
| **Microsoft 365 (Teams Chat)** | **Data Import Tool (Advanced)** | Cloud-native (Admin Console + Azure App) | Teams 1:1, Group chats, Channel messages | Direct cloud import of Teams chat history & attachments |
| **Microsoft 365 (OneDrive)** | **Data Import Tool (Advanced)** | Cloud-native (Admin Console + Azure App) | Personal files, folder trees, ACL permissions | Cloud-to-cloud migration to My Drive |
| **Microsoft 365 (SharePoint Online)** | **Data Import Tool (Advanced)** | Cloud-native (Admin Console + Azure App) | Document libraries, team site files | Document libraries $\rightarrow$ Shared Drives |
| **Dropbox Business** | **Data Import Tool (Advanced)** | Cloud-native (Admin Console) | Dropbox team & personal files | Dropbox $\rightarrow$ Google Drive / Shared Drives |
| **Enterprise Multi-Tenant / Box / File Shares** | **Google Workspace Migrate** | Multi-Node GCP / On-Prem VMs | Mail, OneDrive, SharePoint, Box, File Shares | 1,000+ to 100,000+ users requiring granular telemetry |
| **On-Premises Exchange / Custom IMAP** | **GWMME** | Server / Admin Workstation | Exchange Mail, PSTs, IMAP, Public Folders | On-prem Exchange, PST files, or custom scripts |
| **Desktop Outlook / PST Files** | **GWMMO** | Client Workstation | Local PSTs, Outlook profiles | Individual end-user self-service PST imports |

---

## 2. The Data Import Tool (New Cloud-Native DMS) Architecture

The **Data Import Tool** (accessed via `Admin Console > Data > Data import & export > Data migration (new)`) is Google's modern cloud-native engine replacing legacy DMS.

### A. 2 Ways to Import Data from Exchange Online
1. **Default Data Import**:
   * Uses **Google shared API quota**.
   * Up to **1,000 Exchange Online users** at one time.
   * Direct Global Admin OAuth sign-in connection.
   * Does *not* support Microsoft In-Place Archives, M365 Group mailboxes, event attachments, calendar ACL permissions, or Outlook message rules import.
2. **Advanced Data Import**:
   * Uses **your own dedicated Azure API quota**.
   * Batches of up to **5,000 users per batch**, up to **10 batches concurrently** (5 active running simultaneously).
### B. Mandatory Prerequisite: How User Accounts Are Created Before Data Migration

> [!CRITICAL]
> **Key Architectural Rule**: Google's migration tools (**Data Import Tool**, **Google Workspace Migrate**, **GWMME**) **DO NOT create target user accounts**. They only copy data into user accounts that **already exist** and hold active Google Workspace licenses.

User accounts must be provisioned **before** starting any data import job using one of four standard enterprise methods:

| Provisioning Method | Best For / Scale | How It Works | Admin Steps |
| :--- | :--- | :--- | :--- |
| **1. Entra ID / SCIM Provisioning** | Automated Cloud Sync (Unlimited users) | Synchronizes users directly from Microsoft Entra ID (Azure AD) or Okta to Google Workspace via SCIM protocol. | Configure Google Workspace Enterprise App in Entra ID $\rightarrow$ Turn on Provisioning $\rightarrow$ Assign M365 user groups. |
| **2. Google Cloud Directory Sync (GCDS)** | On-Premises Active Directory Sync (Unlimited users) | Server tool that syncs AD DS / LDAP users, OUs, and groups into Google Workspace automatically. | Deploy GCDS VM $\rightarrow$ Connect to AD LDAP & Google Admin SDK API $\rightarrow$ Execute sync rule pass. |
| **3. GAM CLI Bulk Scripting** | Scripted Admin Control (1,000–50,000 users) | Command-line automation tool that reads an M365 user export CSV and provisions accounts in bulk. | `gam csv m365_users.csv gam create user ~Email firstname ~FirstName lastname ~LastName password ~TempPass ou /MigratedUsers` |
| **4. Admin Console Bulk CSV Upload** | One-time Batch Provisioning (10–1,000 users) | Standard Admin Console spreadsheet upload for fast manual provisioning. | `Admin Console > Directory > Users > Bulk update users` $\rightarrow$ Download template CSV $\rightarrow$ Populate users $\rightarrow$ Upload. |

---

## 3. Microsoft 365 Exchange Online Migration Deep Dive

### A. What Is Imported from Exchange Online?

#### 1. Email Data
- **Email Messages**: All messages (drafts, sent items, inbox, custom subfolders). Excludes junk email and deleted items if configured.
- **Attachments**: Up to **150 MB** per message (unless file type is blocked by Gmail security policy).
- **Message Status**: Read and unread states are preserved.
- **Importance Levels**: High Importance $\rightarrow$ `Important` label. Low/Medium Importance $\rightarrow$ imported without `Important` label.
- **Status Indicators**: Snoozed emails $\rightarrow$ custom `Snoozed` label. Archived emails $\rightarrow$ custom `Archive_` label.
- **Microsoft In-Place Archive Mailboxes** *(Advanced Method Only)*: Copies active users' In-Place Archives to a custom label (`In-Place Archive`), a specified label, or directly to **Google Vault**.
- **Shared & User Mailboxes**: Can be mapped to **Google Groups as Collaborative Inboxes** (read/unread states not imported to Groups) or mapped to a Gmail user account with delegation access enabled.
- **Microsoft 365 Group Mailboxes for Outlook** *(Advanced Method Only)*: Copies conversations and threads to Google Groups (limit 100 posts per thread).
- **Mailbox Settings**: Out of Office / Automatic Replies are automatically imported when email is selected as a data source.

#### 2. Calendar Data
- **Calendars**: Primary, secondary, and resource calendars (meeting rooms/equipment mapped via CSV).
- **Events**: All-day events, attendees with target email IDs, event status, sensitivity, and reminders.
- **Recurring Events**: Events with no end date import with an end date of **December 31, 2099**. Calendar exceptions (modified single instances) are included.
- **Microsoft Teams Meetings**: Meeting links are imported as text links (not converted to Google Meet).
- **Event Attachments** *(Advanced Method Only)*: Creates an `Imported Calendar Attachments` folder in Google Drive for each attendee, storing copies up to 150 MB.
- **Calendar Permissions (ACLs)** *(Advanced Method Only)*: User, group, and domain calendar sharing permissions are copied to Google Calendar. Shared calendars are automatically added to each user's calendar list.

#### 3. Contact Data
- **Contacts**: All non-deleted contacts (up to **27,000 contacts** per organization).
- **Contact Details**: Business and home phone numbers, plus **1 mobile phone number** per contact.
- **Categories & Folders**: Contact folders and categories become single-level labels in Google Contacts (e.g. `Folder1 - Folder2`, `Category1`).

#### 4. Task Data (Microsoft To Do)
- **Personal Tasks**: Microsoft To Do tasks $\rightarrow$ **Google Tasks** (up to 100 task lists).
- **Core Properties**: Title, notes, status, completion date, and due date (imported as all-day tasks without set times).
- **Importance**: High and low importance settings are appended as text inside the task's Note field.

---

### B. What Is NOT Imported?

| Workload | Excluded / Unsupported Items |
| :--- | :--- |
| **Email** | Messages $>150\text{ MB}$; Drafts without a `From` header; Categories; Messages in `Notes` or its subfolders; Pinned, flagged, or scheduled indicators; Shared folders; Starred folder status; Group mailbox emails without MIME headers or outside Microsoft Inbox folder. |
| **Calendar** | Color & timezone calendar settings; Additional resource calendars unsupported by Google Calendar; Events occurring *before* import start date; ICS files; Non-native attachments; Draft calendar events; Plain text formatting only for descriptions; Re-importing events modified in Google Calendar. |
| **Contacts** | Mail contacts; Global Address List (GAL); Deleted contacts/folders; Contact lists; Frequent contacts; Profile pictures; Anniversary dates; Contact initials; Links to personal webpages; Fax, pager, or extra mobile numbers (only 1 mobile number imported); Empty categories. *(Note: Starred contacts require a category named `starred` in Exchange to become Favorites in Google Contacts).* |
| **Tasks** | Recurring task settings (imported as single one-time tasks); Due date time-of-day; File attachments in Microsoft To Do; Microsoft Planner tasks. |

---

### C. Folder & System Label Mapping Logic

Exchange Online folders become Gmail labels. System reserved names are mapped to protect Gmail system labels:

| Exchange Folder Name | Gmail Label Name | Mapping Rule / Behavior |
| :--- | :--- | :--- |
| **All Mail** | `All Mail_` | Underscore added to prevent system label conflict |
| **Drafts** | `DRAFT` | System reserved label |
| **Inbox** | `INBOX` | System reserved label |
| **Sent Items / Sent Mail** | `SENT` / `Sent Mail_` | Standard sent mapping |
| **Junk Email** | `SPAM` | System reserved label |
| **Deleted Items** | `TRASH` | System reserved label |
| **Outbox** | `Outbox_` | Underscore added |
| **Archive** | `Archive_` | Underscore added |
| **Bin** | `Bin_` | Underscore added |
| **Chats** | `Chats_` | Underscore added |

> [!NOTE]
> **Folder Paths & Slashes**: Slashes in folder paths (`Travel/Asia`) become underscores (`Travel_Asia`). Folders with full paths $>225$ characters are imported without the label (messages inside are still imported to `All Mail`). Duplicate messages across multiple Outlook folders are deduplicated into a single instance in Gmail with multiple labels applied.

---

### D. Outlook Rules $\rightarrow$ Gmail Filters Translation Matrix *(Advanced Import)*

When **Import message rules as filters** is enabled in Advanced Data Import:

| Outlook Message Rule | Gmail Filter Support & Behavior |
| :--- | :--- |
| `hasError = True` | ❌ Not imported |
| `isEnabled = False` | ❌ Not imported |
| `isReadOnly = True` | ✅ Imports as standard editable Gmail filter |
| `Subject or body includes` | ✅ Imports as `Subject includes` filter (searches subject & body; partial word substrings unsupported) |
| `Message header includes` | ✅ Imports for newly incoming email messages (unsupported for existing mailbox items) |
| `Maximum / Minimum size` | ⚠️ Gmail allows single size filter only; rules with both min & max size are skipped |
| `Move to folder` | ✅ Supported for standard folders (unsupported for Draft, Sent, Spam) |
| `Categories / Flags / Sensitivity` | ❌ Not supported |
| `Received before / Received after` | ❌ Not supported |
| `Forward to / Forward as attachment / Redirect` | ❌ Not supported |
| `Pin to top` | ❌ Not supported |

---

### E. Calendar Permissions (ACL) Mapping Table *(Advanced Import)*

| Outlook Calendar Permission Role | Google Calendar ACL Role | Modification Rights |
| :--- | :--- | :--- |
| **None** | `None` | No access |
| **freeBusyRead / limitedRead** | `freeBusyReader` | Sees free/busy times only |
| **Read** | `Reader` | Sees event details |
| **Write / delegateWithoutPrivateEventAccess** | `writerWithoutPrivateAccess` | Can edit events (except private) |
| **delegateWithPrivateEventAccess** | `Writer` | Full edit access (Super Admin required for sharing changes) |
| **Custom** | ❌ Unsupported | Skipped |

---

### F. Mail & Message Count Discrepancy Causes

After a migration, users often notice differences between Outlook message counts and Gmail counts due to three architectural factors:
1. **Folder Storage vs. Deduplication**: Outlook counts separate copies of an email stored in multiple folders. Gmail deduplicates them into a single email instance with multiple labels, resulting in lower total counts in Gmail.
2. **Hidden/Archived Unread Counts**: Outlook hides unread emails in archived system folders. Gmail maps these folders to labels and includes unread counts, causing unread counts to increase in Gmail.
3. **Conversation Threading**: Gmail groups replies into conversation threads. A new incoming reply marks the entire thread as unread, whereas Outlook treats messages individually.

---

## 4. Azure/Entra ID App Registration for Advanced Import

To execute Advanced Data Import or scan batches:

1. In **Microsoft Entra admin center** (`entra.microsoft.com`), register an app: `Google Workspace Advanced Data Import`.
2. Under **API Permissions**, add Application permissions:
   * Microsoft Graph: `full_access_as_app`, `User.Read.All`, `Mail.Read`, `Calendars.Read`, `Contacts.Read`, `Chat.Read.All`, `ChannelMessage.Read.All`, `Files.Read.All`.
3. Click **Grant admin consent for [Tenant]**.
4. Generate a **Client Secret**; note the **Client ID**, **Client Secret**, and **Tenant ID**.
5. Enter these credentials in `Google Admin Console > Data > Data import & export > Data import > Advanced > New batch`.

---

## 5. Teams, OneDrive, SharePoint & Third-Party Migration

### A. Teams Chat Migration
* **1:1 Messages & Group Chats** $\rightarrow$ Google Chat 1:1 and Group Conversations.
* **Channel Messages** $\rightarrow$ Google Chat Spaces with inline attachments saved in Google Drive.

### B. OneDrive & SharePoint Online

#### 1. Workload Architecture
* **OneDrive for Business** $\rightarrow$ Google Drive (My Drive). Preserves folder hierarchies and ACL sharing.
* **SharePoint Team Site** $\rightarrow$ Target **Google Shared Drive**.
* **Document Library** $\rightarrow$ Shared Drive root folders.

#### 2. Step-by-Step SharePoint Online Advanced Data Import Setup

##### Phase 1: Create Azure App Registration (Microsoft Entra ID)
1. Sign in to **Microsoft Entra admin center** (`entra.microsoft.com`).
2. Navigate to **Applications > App registrations > New registration**.
3. Name: `Google Workspace Data Import - SharePoint`.
4. Supported Account Types: **Accounts in this organizational directory only (Single tenant)**.
5. Under **API Permissions**, click **Add a permission > Microsoft Graph > Application permissions**:
   - `Sites.FullControl.All` (or `Sites.ReadWrite.All`)
   - `Files.ReadWrite.All`
   - `User.Read.All`
   - `Group.Read.All`
6. Click **Grant admin consent for [Your Tenant Name]**.
7. Navigate to **Certificates & secrets > Client secrets > New client secret**. Copy the secret value immediately.
8. Record: **Application (Client) ID**, **Directory (Tenant) ID**, and **Client Secret**.

##### Phase 2: Create Batch in Google Admin Console
1. Go to **Google Admin Console** (`admin.google.com`).
2. Navigate to **Data > Data import & export > Data import > Advanced**.
3. Click **New batch**.
4. Enter **Batch name** (e.g. `Testing` or `SharePoint Migration Wave 1`).
5. Select **Data type**: `SharePoint Online` and click **Continue**.

##### Phase 3: Step 1 — Connect your SharePoint Account (Admin Screen)
Enter the four required credentials:
* **Client ID**: Paste Azure Application (Client) ID.
* **Client secret**: Paste Azure Client Secret value.
* **Tenant ID**: Paste Azure Directory (Tenant) ID.
* **Sharepoint host name**: Enter your SharePoint base domain without `https://` or trailing slashes (e.g., `contoso.sharepoint.com`).
Click **Connect**.

##### Phase 4: Step 2 — Select Sites to Import (CSV Upload #1)
Create a CSV file mapping source SharePoint sites to target Google Shared Drives:
```csv
Source SharePoint Site URL,Target Shared Drive ID,Managing User Email
https://contoso.sharepoint.com/sites/Marketing,0A1b2C3d4E5f6G7h8,admin@myworkspace.com
https://contoso.sharepoint.com/sites/Engineering,0B987654321ZYXWVU,admin@myworkspace.com
```
*Upload the CSV file in Step 2.* (Supports up to **5,000 SharePoint sites** per batch).

##### Phase 5: Step 3 — Map Users & Groups (CSV Upload #2)
Create an Identity Mapping CSV file to map SharePoint permission holders to Google Workspace users and Google Groups:
```csv
Source Email,Destination Email
jdoe@contoso.com,jdoe@myworkspace.com
MarketingTeam@contoso.com,marketing-group@myworkspace.com
```
*Upload the Identity Mapping CSV in Step 3, then proceed to Settings, Pre-Scan, and Execute.*

---

## 6. GWMME & GWMMO Scripting & Command Line Reference

### Headless GWMME CLI Command Example:
```powershell
& "C:\Program Files\Google\Google Workspace Migration for Microsoft Exchange\ClientMigration.exe" `
  -GlobalAdminUser "admin@domain.com" `
  -ServiceAccountKeyPath "C:\Keys\workspace-migration-sa.json" `
  -SourceType EXCHANGE_ONLINE `
  -UserMapFile "C:\Migration\UserMap.csv" `
  -StartDate "2020-01-01" `
  -MigrateEmail -MigrateCalendar -MigrateContacts `
  -NumThreads 32 `
  -LogDir "C:\Migration\Logs"
```

---

## 7. Migration Diagnostic & Troubleshooting Error Codes

| Error Code / Symptom | Cause | Solution |
| :--- | :--- | :--- |
| **Error 1004 / 401 Unauthorized** | Azure App Secret expired or missing Entra Admin Consent | Re-generate secret in Azure; click **Grant admin consent for [Tenant]**. |
| **Error 429 Too Many Requests** | Graph API rate-limiting / EWS throttling | System automatically backs off; reduce batch sizes or concurrent threads. |
| **Error 500 / General Failure** | Source mailbox unlicensed or locked | Verify active Exchange Online license; exclude corrupt folders. |
| **Message Attachment $>150\text{ MB}$** | Exceeds Gmail maximum message size | Flagged in report; migrate file via Google Drive. |
| **Folder Path $>225$ Chars** | Exceeds label character limit | Messages import into `All Mail` without label. |

---

## 8. Migration Verification & Sign-Off Checklist

- [ ] **Pre-Scan Pass**: Run batch data scan in Advanced Data Import; review item counts and warnings.
- [ ] **Primary Migration**: Complete full pass of email, calendars, contacts, tasks, and files.
- [ ] **Single MX Cutover**: Point DNS MX to `1 SMTP.google.com`.
- [ ] **Email Security**: Enable SPF (`v=spf1 include:_spf.google.com ~all`), DKIM (`google._domainkey`), and DMARC.
- [ ] **Delta Migration Pass**: Execute 48 hours post-MX cutover to catch in-flight data.
- [ ] **Exit Import**: Download log reports and click **Exit and delete import** to clean up standing client delegation.

---
*Reference: Official Google Workspace Admin Migration Documentation (Updated August 2026).*
