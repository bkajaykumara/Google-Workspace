# 4. GAM CLI, Scripting, Developer APIs, and L3 Incident Operations Master Engineering Guide

This master guide provides a comprehensive operational reference for Google Apps Manager (GAM / GAM-ADV-XTD3), custom Python SDK scripts, Google Apps Script automation, PowerShell/Bash endpoint administration, and Level 3 Emergency Incident Response Playbooks.

---

## ⚡ 1. GAM & GAM-ADV-XTD3 Master Command Reference

GAM (Google Apps Manager) is a command-line tool for managing Google Workspace domains via Admin SDK REST APIs.

### BNF Selectors & Quoting Rules
*   **User Selectors**: `user <email>`, `all users`, `ou <OU_Name>`, `ou_and_children <OU_Name>`, `group <group_email>`.
*   **Multithreading (`num_threads`)**: Execute commands concurrently across large user populations (`gam csv users.csv gam user ~primaryEmail update user ... num_threads 20`).

### Essential GAM Production One-Liners

```bash
# 1. Bulk User Offboarding Pipeline
gam user alex.smith@company.com deprecate retain-license transfer drive backup-admin@company.com keep-shares transfer calendar backup-admin@company.com delete-calendar-events move /_Leavers signout

# 2. Domain-Wide Phishing Email Purge
gam all users delete messages query "rfc822msgid:123456789@phishing.com" doit

# 3. Audit 2SV Status Across Root OU
gam print users ou / fields primaryEmail,isEnrolledIn2Sv,is2SvEnforced > 2sv_audit.csv

# 4. Create Shared Drive & Assign Permissions
gam create shareddrive "Engineering Docs" admin alex.smith@company.com
gam add acl shareddrive "Engineering Docs" user sara.jones@company.com role fileorganizer

# 5. Reclaim Inactive Enterprise Plus Licenses
gam user alex.smith@company.com update license wsentplus cloudidentityfree
```

---

## 💻 2. Developer APIs & Python SDK Automation

### Complete Python SDK Script: Upload File to Google Drive

```python
import os.path
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

SCOPES = ['https://www.googleapis.com/auth/drive.file']

def upload_file_to_drive(file_path, mime_type):
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file('credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())

    service = build('drive', 'v3', credentials=creds)
    file_metadata = {'name': os.path.basename(file_path)}
    media = MediaFileUpload(file_path, mimetype=mime_type)
    
    file_obj = service.files().create(body=file_metadata, media_body=media, fields='id, webViewLink').execute()
    print(f"Uploaded File ID: {file_obj.get('id')}")
    print(f"Web Link: {file_obj.get('webViewLink')}")

if __name__ == '__main__':
    upload_file_to_drive('report.pdf', 'application/pdf')
```

---

## 📜 3. Google Apps Script & OS Shell Scripts

### Google Apps Script: Auto-Transfer Inactive User Files

```javascript
function transferInactiveUserFiles() {
  var targetEmail = "leaver.user@company.com";
  var adminEmail = "storage.admin@company.com";
  
  var files = DriveApp.searchFiles("'" + targetEmail + "' in owners and trashed = false");
  while (files.hasNext()) {
    var file = files.next();
    file.setOwner(adminEmail);
  }
  Logger.log("Transferred ownership of files from " + targetEmail + " to " + adminEmail);
}
```

---

## 🚨 4. Level 3 Emergency Incident Response SOPs

### SOP 1: Mitigating a Massive Phishing Outbreak
1.  **Identify Message ID**: Extract `Message-ID` or unique subject/sender from Email Log Search (ELS).
2.  **Execute GAM Purge**:
    ```bash
    gam all users delete messages query "subject:\"URGENT: Password Reset Required\"" doit
    ```
3.  **Revoke Suspicious OAuth Tokens**:
    ```bash
    gam all users delete tokens appid <malicious_app_id>
    ```

### SOP 2: Recovering from Total IdP SSO Outage
1.  **Access Admin Console Directly**: Log into `admin.google.com` using a **Break-Glass Emergency Super Admin Account** (`admin-breakglass@company.com`) residing in an OU with Third-Party SSO disabled.
2.  **Execute SSO Bypass**: Navigate to **Security > Authentication > SSO with third-party IdP**, edit assignment for Root OU, and set **SSO Profile assignment** to **None (Use Google Credentials)**.
3.  **Restore SSO**: Once OneLogin/Okta recovers, revert assignment to Third-Party SAML SSO Profile.

### SOP 3: Compromised / Lost Device Emergency Response
1.  **Wipe Device via GAM**:
    ```bash
    gam user alex.smith@company.com wipe device <device_id>
    ```
2.  **Sign Out Active User Sessions & Revoke OAuth Tokens**:
    ```bash
    gam user alex.smith@company.com signout
    gam user alex.smith@company.com delete tokens
    ```

### SOP 4: Repairing Broken SCIM / Provisioning Sync Errors
1.  **Debug HTTP 409 Conflict**: User account already exists in Google Workspace with primary email or alias. Rename or merge duplicate account.
2.  **Debug HTTP 400 Bad Request**: Invalid attribute mapping (e.g., null value passed to required `name.familyName` field). Update missing fields in IdP directory.
