# Module 9: GAM Advanced Command Reference & Interview Cheat Sheet

This module provides a production-grade reference for **Google Apps Manager (GAM / GAM-ADV-XTD3)** based on official Backus-Naur Form (BNF) command syntax. It is engineered for Level 3 Administrators, Systems Engineers, and interview candidates who need to demonstrate command-line mastery across user lifecycles, Drive security, email routing, domain-wide audits, and large-scale enterprise migrations.

---

## 1. GAM Architecture & Syntax Fundamentals

### Core BNF Notation Rules

GAM command structure follows modified Backus-Naur Form (BNF):

| Symbol | Meaning | Example |
| :--- | :--- | :--- |
| `<Item>` | Required parameter placeholder | `<EmailAddress>`, `<DriveFileID>`, `<OrgUnitPath>` |
| `[Item]` | Optional parameter | `[changepassword on]`, `[todrive]` |
| `(A \| B)` | Choice between alternatives | `(user <UserItem> \| group <GroupItem>)` |
| `Item*` | Zero or more occurrences | `(matchfield <FieldName> <Pattern>)*` |
| `Item+` | One or more occurrences | `<Digit>+` |
| `~Column` | CSV column variable replacement | `~Email`, `~OrgUnitPath`, `~Password` |

### Quoting & Escaping Rules

When parameters contain spaces, commas, or quotes:
- **No special characters**: `gam user user@domain.com show drivefileacl 0Bxyz`
- **Spaces in strings**: Surround with double quotes: `gam user user@domain.com add sendas user@domain.com "Finance Team"`
- **Nested quotes / Commas in CSV lists**: `" 'item 1', 'item, 2', \"item'3\" "`
- **Regular Expressions**: Use Python `re` syntax: `gam all users print filelist query "mimeType != 'application/vnd.google-apps.folder'"`

---

## 2. Entity Selectors (How to Target Users, Groups, and OUs)

GAM allows targeting single entities, subsets, or the entire tenant using rich selector syntax:

### A. User Selectors (`<UserTypeEntity>`)

| Selector | BNF Pattern | Description |
| :--- | :--- | :--- |
| **Single User** | `gam user <EmailAddress>` | Targets one specific user |
| **All Active Users** | `gam all users` | Targets all active (non-suspended, non-archived) users |
| **All Suspended Users** | `gam all users_susp` | Targets only suspended users (ideal for cleanup) |
| **All Archived Users** | `gam all users_arch` | Targets Google Vault / Archived User SKU accounts |
| **Specific Org Unit** | `gam ou <OrgUnitPath>` | Targets users directly in that OU (excludes sub-OUs) |
| **OU and Sub-OUs** | `gam ou_and_children <OrgUnitPath>` | Recursively targets an OU and all nested child OUs |
| **Group Members** | `gam group_users <GroupEmail> [members] [managers] [owners] end` | Targets members/managers/owners of a Google Group |
| **Directory Query** | `gam query <QueryUser>` | Queries directory attributes: `gam query "orgUnitPath='/Sales' isSuspended=false"` |
| **License Filter** | `gam license <SKUID>` | Targets all users assigned a specific SKU (e.g., `wsentplus`) |
| **CSV Input Loop** | `gam csv <FileName.csv> gam <Command>` | Loops through CSV, replacing `~Header` with row values |

### B. ChromeOS & Mobile Device Selectors

| Selector | BNF Pattern | Description |
| :--- | :--- | :--- |
| **All Chrome Devices** | `gam all cros` | Targets all enrolled ChromeOS hardware |
| **ChromeOS by OU** | `gam cros_ou_and_children <OrgUnitPath>` | Targets ChromeOS devices within a specific OU subtree |
| **ChromeOS by Serial** | `gam cros_sn <SerialNumberList>` | Targets specific device serial numbers |
| **Mobile Devices** | `gam print mobile query "<QueryMobile>"` | Queries enrolled Android/iOS endpoints |

---

## 3. Product & SKU Reference (SKUID Table)

When assigning, upgrading, or auditing Google Workspace licenses via GAM, use these standardized SKU aliases:

| SKU Name | GAM Alias | Numeric SKUID | Description |
| :--- | :--- | :--- | :--- |
| **Workspace Enterprise Plus** | `wsentplus` / `enterprise` | `1010020020` | Full enterprise suite + DLP + Vault + S/MIME |
| **Workspace Enterprise Standard**| `wsentstan` | `1010020026` | Enterprise storage + recording + standard security |
| **Workspace Business Plus** | `wsbizplus` | `1010020025` | 5 TB storage + Vault + Advanced Endpoint Management |
| **Workspace Business Standard** | `wsbizstan` | `1010020028` | 2 TB storage + 150-participant Meet + Shared Drives |
| **Workspace Business Starter** | `wsbizstarter` | `1010020027` | 30 GB storage + basic services |
| **Workspace Frontline Starter** | `wsflw` | `1010020030` | Frontline mobile/collaboration suite |
| **Cloud Identity Free** | `cloudidentityfree` / `identity` | `1010010001` | SSO/IAM identity without Workspace core apps |
| **Cloud Identity Premium** | `cloudidentitypremium` | `1010050001` | BeyondCorp, Context-Aware Access, advanced MDM |
| **Google Vault (Add-on)** | `vault` | `Google-Vault` | Standalone archiving/eDiscovery license |
| **Archived User (Enterprise)** | `gsuiteenterprisearchived` | `1010340001` | Data retention for offboarded Enterprise users |
| **Gemini Enterprise (Add-on)** | `geminient` / `duetai` | `1010470001` | Generative AI add-on for Workspace |

---

## 4. Production GAM Playbooks by Domain

### A. User Lifecycle Automation (Joiner-Mover-Leaver)

#### 1. Bulk Provisioning with Specific OU and License
```bash
# Provision 1,000+ users from CSV with randomized passwords, force change at login, and assign Enterprise Plus SKU
gam csv new_hires.csv gam create user ~Email firstname ~FirstName lastname ~LastName password ~TempPassword org ~OrgUnitPath changepassword on

# Bulk assign Enterprise Plus license
gam csv new_hires.csv gam user ~Email add license wsentplus
```

#### 2. Department Mover (Transfer OU, Update Manager, Clear Old Group Memberships)
```bash
# Move user to new Organizational Unit
gam user jdoe@company.com update ou "/Sales/EMEA"

# Update reporting manager and department attributes in Directory
gam user jdoe@company.com relation manager smitchell@company.com department "EMEA Sales" title "Senior Account Executive"

# Remove user from all groups in former department
gam user jdoe@company.com delete groups
```

#### 3. Complete Leaver / Offboarding Pipeline (Zero-Trust Deprovisioning)
```bash
# Step 1: Wipe all active web sessions & OAuth tokens
gam user leaver@company.com signout
gam user leaver@company.com delete tokens

# Step 2: Suspend account, randomize password, move to Offboarded OU
gam user leaver@company.com update suspended on password "Disabled#994!xK" org "/Offboarded_Users"

# Step 3: Remove from all Google Groups and wipe mobile devices
gam user leaver@company.com delete groups
gam user leaver@company.com delete mobile

# Step 4: Transfer entire Drive and Calendar ownership to manager
gam user leaver@company.com transfer drive manager@company.com
gam user leaver@company.com transfer calendar manager@company.com

#### 4. Production Playbook: Inactive User Audit, Leaver OU Move & License Reclamation
```bash
# Step 1: Move inactive users (>90 days no login) to '_Leavers' OU
gam csv cleanup.csv gam update user ~email org "_Leavers" >> cleanupresults.csv

# Step 2: Strip expensive Enterprise Plus SKU (1010020020) to reclaim license costs
gam csv cleanup.csv gam user ~email delete license "1010020020" >> cleanupresults.csv

# Step 3: Strip standalone Cloud Search SKU (1010350001)
gam csv cleanup.csv gam user ~email delete license "1010350001" >> cleanupresults.csv
```
*Note*: Service accounts (`svc_*`) and technical accounts (`ta_*`) must be filtered out of `cleanup.csv` prior to execution. Always use `>>` append redirection to preserve audit logs across all three phases.

---

### B. Google Drive & Shared Drive Administration

#### 1. Shared Drive Lifecycle Management
```bash
# Create a new Shared Drive and capture its ID
gam create teamdrive "Q4 Marketing Campaigns"

# Add members with specific Shared Drive roles (manager, contentmanager, contributor, commenter, reader)
gam add drivefileacl <SharedDriveID> user director@company.com role manager
gam add drivefileacl <SharedDriveID> group marketing-team@company.com role contentmanager
gam add drivefileacl <SharedDriveID> user contractor@agency.com role reader

# Restrict Shared Drive permissions (disallow external sharing, disallow non-members reading)
gam update teamdrive <SharedDriveID> restrictions:domainUsersOnly true restrictions:driveMembersOnly true
```

#### 2. Domain-Wide Drive Security Audits
```bash
# Audit all files in the domain shared with 'anyoneWithLink' or 'anyone' (Public exposures)
gam all users print filelist query "visibility='anyoneWithLink' or visibility='anyoneCanFind'" fields id,name,owners,permissions todrive

# Audit files shared externally outside the corporate domain
gam all users print filelist query "readableByMe and not (domain='company.com')" fields id,name,owners,permissions > external_shares.csv

# Strip external sharing permissions in bulk from compromised user's files
gam user compromised@company.com delete drivefileacl 0Bxyz anyoneWithLink
```

#### 3. Orphaned File Recovery & Bulk File Ownership Transfer
```bash
# Find files owned by a suspended user that are not in a Shared Drive
gam user suspended_user@company.com print filelist query "'suspended_user@company.com' in owners" fields id,name > user_files.csv

# Bulk transfer file ownership from CSV
gam csv user_files.csv gam user new_owner@company.com claimdrivefile ~id
```

---

### C. Gmail & Mail Flow Security Administration

#### 1. Delegated Mailbox Access (Executive & Legal Audits)
```bash
# Add executive assistant as a delegate to an executive mailbox
gam user executive@company.com add delegate assistant@company.com

# Audit all active delegates domain-wide
gam all users print delegates todrive

# Revoke a delegate immediately
gam user executive@company.com delete delegate assistant@company.com
```

#### 2. Auto-Forwarding & Compliance Audits
```bash
# Audit all active email forwarding rules across the entire domain
gam all users print forwardingaddresses todrive

# Audit all Gmail filters created across all users (detecting malicious exfiltration rules)
gam all users print filters todrive

# Delete an unauthorized external forwarding rule
gam user victim@company.com delete forwardingaddress attacker@external.com
```

#### 3. Send-As Aliases & Out of Office (Vacation) Enforcements
```bash
# Add a verified corporate send-as address for a support department
gam user agent@company.com add sendas support@company.com "Company Support" default true

# Enforce an emergency vacation auto-responder for an unexpectedly absent employee
gam user absent@company.com vacation on subject "Out of Office Notice" message "I am currently out of office. For urgent inquiries, contact team@company.com." startdate 2026-08-19 enddate 2026-08-26
```

#### 4. Incident Response: Domain-Wide Malicious Email Purge
```bash
# Search and permanently purge phishing emails across all user mailboxes by Message-ID or Subject
gam all users delete messages query "from:phishing@evil.com subject:'Urgent Payroll Update'" doit

# Purge email by header Message-ID (fastest, most precise incident response)
gam all users delete messages query "rfc822msgid:CA+123456789@mail.attacker.com" doit
```

---

### D. Security, 2SV, and Identity Governance

#### 1. Two-Step Verification (2SV / MFA) Audits
```bash
# Audit 2SV enforcement status across all Super Admins
gam query "isAdmin=true" print users fields primaryEmail,isEnrolledIn2Sv,is2SvEnforced

# Generate backup verification codes for a locked-out executive
gam user executive@company.com print backupcodes
```

#### 2. Third-Party OAuth App & Token Audits
```bash
# Print all third-party apps granted OAuth tokens across all users
gam all users print tokens todrive

# Revoke a malicious / unapproved third-party app across the entire organization by Client ID
gam all users delete token clientid 9876543210-abcdef.apps.googleusercontent.com
```

#### 3. Context-Aware Access & Security Key Enforcement
```bash
# List all registered FIDO2 / WebAuthn Hardware Security Keys for a user
gam user admin@company.com print securitykeys

# Audit mobile device compliance status across the organization
gam print mobile fields resourceId,email,model,os,compromisedStatus,devicePasswordStatus
```

---

## 5. Enterprise Migration & Provisioning GAM Pipelines

### A. Pre-Migration Directory Provisioning Pipeline

For a 10,000-user migration from Microsoft 365, execute this multi-stage GAM batch pipeline:

```bash
# Stage 1: Create Organizational Units from CSV
gam csv ous.csv gam create ou ~OrgUnitPath

# Stage 2: Bulk provision user accounts into target OUs with randomized passwords
gam csv m365_users.csv gam create user ~Email firstname ~FirstName lastname ~LastName password ~TempPassword org ~OrgUnitPath changepassword on

# Stage 3: Assign corresponding Workspace licenses
gam csv m365_users.csv gam user ~Email add license ~SKUID

# Stage 4: Create Google Groups to match M365 Security Groups
gam csv groups.csv gam create group ~GroupEmail name ~GroupName description ~Description

# Stage 5: Populate Group Memberships in bulk
gam csv group_members.csv gam update group ~GroupEmail add member user ~MemberEmail
```

### B. Post-Migration Verification & Permission Remediation

```bash
# Step 1: Verify all 10,000 users exist and are active
gam all users print users fields primaryEmail,orgUnitPath,isSuspended > post_migration_users.csv

# Step 2: Audit Shared Drive permissions to confirm M365 SharePoint mapping
gam all users print teamdrives fields id,name,restrictions > active_shared_drives.csv

# Step 3: Check for orphaned or unmapped Drive permissions
gam all users print filelist query "mimeType != 'application/vnd.google-apps.folder'" fields id,name,permissions todrive

# Step 4: Bulk-transfer calendar resource bookings to Google Workspace Resource Calendars
gam csv calendar_resources.csv gam create resource ~ResourceEmail name ~ResourceName type ~Type capacity ~Capacity
```

---

## 6. Interview Quick Reference: Top 15 Senior Admin One-Liners

| # | Operational Scenario | Exact GAM Command |
| :--- | :--- | :--- |
| **1** | **Mass Phishing Purge** | `gam all users delete messages query "subject:'Urgent Action Required'" doit` |
| **2** | **Wipe Compromised User** | `gam user compromised@co.com signout && gam user compromised@co.com delete tokens` |
| **3** | **Bulk Provision from CSV** | `gam csv users.csv gam create user ~Email firstname ~First lastname ~Last password ~Pass org ~OU changepassword on` |
| **4** | **Assign Enterprise SKU** | `gam csv users.csv gam user ~Email add license wsentplus` |
| **5** | **Audit External Drive Shares** | `gam all users print filelist query "visibility='anyoneWithLink'" fields id,name,owners todrive` |
| **6** | **Audit Auto-Forwarding** | `gam all users print forwardingaddresses todrive` |
| **7** | **Emergency Out-of-Office** | `gam user user@co.com vacation on subject "OOO" message "Out until Monday" startdate 2026-08-19 enddate 2026-08-25` |
| **8** | **Add Mailbox Delegate** | `gam user manager@co.com add delegate assistant@co.com` |
| **9** | **Wipe Mobile Device** | `gam user user@co.com wipe mobile <DeviceID>` |
| **10** | **Audit 2SV Super Admins** | `gam query "isAdmin=true" print users fields primaryEmail,isEnrolledIn2Sv` |
| **11** | **Revoke Malicious OAuth App** | `gam all users delete token clientid <ClientID>` |
| **12** | **Transfer Drive Ownership** | `gam user leaver@co.com transfer drive manager@co.com` |
| **13** | **Create Shared Drive** | `gam create teamdrive "Finance Shared Drive"` |
| **14** | **Add Shared Drive Manager** | `gam add drivefileacl <DriveID> user admin@co.com role manager` |
| **15** | **Export Group Members** | `gam print group-members group all-employees@co.com todrive` |

---

## 7. Performance & Rate Limit Optimization (GAM Tuning)

When running GAM operations across **10,000+ users**, unoptimized commands will trigger Google API `429 Too Many Requests` rate limit errors. Apply these enterprise optimizations:

### 1. Enable Multithreading (`gam.cfg`)
Configure `gam.cfg` to adjust the number of concurrent API worker threads:
```ini
# Recommended for large batch operations
num_threads = 25
auto_batch_min = 50
```

### 2. Stream Output to Google Sheets (`todrive`)
Instead of redirecting large standard output to local flat files (`> output.csv`), append `todrive` to upload results directly to Google Drive as an auto-formatted Google Sheet:
```bash
gam all users print filelist query "visibility='anyone'" todrive
```

### 3. Use Target Selectors Instead of `all users`
Never run `gam all users` when targeting a specific subset:
- ❌ `gam all users print users query "orgUnitPath='/Sales'"` (makes 10,000 API calls)
- ✅ `gam ou_and_children "/Sales" print users` (queries only the relevant OU subtree)
