# 2. Identity, Access, and Device Management Master Engineering Guide

This master guide covers Enterprise IAM, Single Sign-On (SSO), SAML 2.0, OAuth 2.0, OpenID Connect (OIDC), SCIM automated user provisioning (JML), Endpoint Device Management (MDM), Netskope CASB steering, and Context-Aware Access (CAA).

---

## 🏛️ 1. Enterprise IAM Topology (OneLogin + Google Workspace + Netskope)

In a modern zero-trust enterprise deployment:
*   **Identity Provider (IdP)**: OneLogin (Central source of truth for authentication and identity lifecycle).
*   **Service Provider (SP)**: Google Workspace (Resource domain for Gmail, Drive, Docs, Calendar, Meet).
*   **User Lifecycle Provisioning**: OAuth 2.0 REST automation connecting OneLogin to Google Admin SDK Directory API.
*   **Device Security Steering**:
    *   *Managed Devices*: Direct access to Google Workspace.
    *   *Unmanaged / BYOD Devices*: Steered through **Netskope Reverse Proxy (CASB)** for real-time DLP, download prevention, and document watermarking.

```mermaid
graph TD
    User([User Sign-In Request]) --> DeviceCheck{Is Device Managed & Compliant?}
    
    DeviceCheck -->|Yes: MDM Cert Verified| DirectGW[Direct Connection to Google Workspace]
    DeviceCheck -->|No: Personal / Unmanaged| ProxyNetskope[Routed via Netskope Reverse Proxy CASB]
    
    DirectGW --> SAMLAuth[OneLogin SAML 2.0 SP-Init Authentication]
    ProxyNetskope --> InlineDLP{Inline Security Inspection}
    InlineDLP -->|File Download Attempt| BlockDownload[❌ Block Download & Log Security Event]
    InlineDLP -->|Web Document View| AllowWatermark[✅ Allow View with Overlay Watermark]
```

---

## 🔐 2. SAML 2.0 Authentication Engineering

Security Assertion Markup Language (SAML 2.0) handles federated enterprise authentication.

### SP-Initiated SAML 2.0 Authentication Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as User Browser
    participant SP as Google Workspace (SP)
    participant IdP as OneLogin (IdP)
    
    User->>SP: 1. Request access (https://mail.google.com)
    Note over SP: Detects SSO domain configuration
    SP-->>User: 2. Redirect (HTTP 302) with SAMLRequest payload
    User->>IdP: 3. GET /trust/saml2/http-post/sso?SAMLRequest=...
    Note over IdP: Authenticates user (Password + WebAuthn FIDO2 MFA)
    IdP-->>User: 4. Generate & sign SAMLResponse XML with IdP Private Key
    User->>SP: 5. POST SAMLResponse to ACS URL (https://www.google.com/a/company.com/acs)
    Note over SP: Decrypts & verifies signature using uploaded X.509 Certificate,<br/>validates Entity ID & assertion timestamps
    SP-->>User: 6. Issue Google Session Cookie & load Inbox
```

### Core SAML Configuration Parameters:
*   **ACS URL (Assertion Consumer Service)**: `https://www.google.com/a/yourdomain.com/acs`
*   **Entity ID (Audience URI)**: `google.com/a/yourdomain.com`
*   **NameID Format**: `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`
*   **X.509 Certificate Rotation**: Google supports **Dual Certificate Window** uploading up to 2 public X.509 certificates. Upload the new certificate to Google prior to rotating private keys in OneLogin to prevent user lockout.

---

## 🔑 3. OAuth 2.0 Authorization Framework

OAuth 2.0 (RFC 6749) grants third-party applications scoped API permissions without exposing user passwords.

```mermaid
sequenceDiagram
    autonumber
    actor User as Resource Owner (User)
    participant Client as Provisioning Engine (OneLogin)
    participant AuthServer as Google OAuth Server
    participant ResourceServer as Admin SDK Directory API
    
    Client->>AuthServer: 1. Authenticate with Service Account Key
    AuthServer-->>Client: 2. Issue Bearer Access Token (Valid 3600s)
    Client->>ResourceServer: 3. POST /admin/directory/v1/users (Authorization: Bearer <Token>)
    ResourceServer-->>Client: 4. Return HTTP 201 User Created
```

*   **Access Token vs. Refresh Token**: Access tokens grant short-lived API permissions (typically 1 hour). Refresh tokens are long-lived credentials used to request new Access Tokens.
*   **Admin SDK Scopes**:
    *   `https://www.googleapis.com/auth/admin.directory.user` (Full user management).
    *   `https://www.googleapis.com/auth/admin.directory.group` (Group lifecycle).

---

## 🪪 4. OpenID Connect (OIDC) & Modern Authentication

OIDC extends OAuth 2.0 to add a standardized identity authentication layer.

$$\text{OIDC} = \text{OAuth 2.0 (Authorization)} + \text{ID Token JWT (Authentication)}$$

### JWT ID Token Structure (`Header.Payload.Signature`):

```json
// Header
{ "alg": "RS256", "typ": "JWT" }

// Payload (Decoded Claims)
{
  "iss": "https://accounts.google.com",
  "aud": "1092837465012-client_id.apps.googleusercontent.com",
  "sub": "108237491827364510293",
  "hd": "company.com",
  "email": "alex.smith@company.com",
  "email_verified": true,
  "iat": 1787696100,
  "exp": 1787699700
}
```

---

## 📊 5. Protocol Comparison Matrix

| Technical Vector | SAML 2.0 | OAuth 2.0 | OpenID Connect (OIDC) |
| :--- | :--- | :--- | :--- |
| **Primary Purpose** | Enterprise Authentication / SSO | Delegated Authorization | Authentication + Authorization |
| **Data Format** | XML (`<saml2:Assertion>`) | JSON / Opaque Bearer Token | JSON Web Token (JWT) |
| **API Access** | ❌ No native API token model | ✅ Native API bearer token | ✅ Native API bearer token |
| **Mobile & SPA** | Poor (Requires Browser Redirects) | Excellent | Excellent (supports PKCE RFC 7636) |
| **Target Use Case** | Legacy & SaaS Enterprise SSO | Third-party API Access | Modern SaaS, Mobile & Single Page Apps |

---

## 🔄 6. SCIM & Joiner-Mover-Leaver (JML) Lifecycle Automation

SCIM (System for Cross-domain Identity Management) automates provisioning between HR systems, IdPs, and Google Workspace.

```mermaid
graph TD
    HR["HR System (Workday / BambooHR)"] --> IdP["IdP (OneLogin / Okta)"]
    
    subgraph JML Processes
        IdP -->|SCIM Create| Joiner["Joiner: Provision user account, assign license, add to default groups"]
        IdP -->|SCIM Update| Mover["Mover: Update department, manager, and group memberships"]
        IdP -->|SCIM Suspend| Leaver["Leaver: Disable account, revoke OAuth tokens, transfer Drive files"]
    end
    
    Joiner --> GW["Google Workspace Admin SDK API"]
    Mover --> GW
    Leaver --> GW
```

---

## 📱 7. Endpoint Device Management (MDM) & Context-Aware Access (CAA)

### Endpoint Management Strategy
*   **macOS Onboarding**: Automated Device Enrollment (ADE) via Apple Business Manager (ABM) pushing Jamf Pro MDM profiles.
*   **Windows Onboarding**: Windows Autopilot enrolling devices into Microsoft Intune.
*   **Google Endpoint Management**: Enforces device encryption, screen lock, remote wipe, and enterprise app management across BYOD and Company-Owned devices.

### Context-Aware Access (CAA) CEL Policy Engine
Context-Aware Access evaluates access requests dynamically using Common Expression Language (CEL).

```cel
// CEL Access Level Example: Require Corporate Managed Chrome + Egress IP Range
device.chrome.management_state == ChromeManagementState.CHROME_MANAGEMENT_STATE_BROWSER_MANAGED &&
inIpRange(origin.ip, ["203.0.113.0/24", "198.51.100.12/32"])
```

---

## 🔗 Official Google Support References
- [Set up SSO via SAML for Google Workspace](https://support.google.com/a/answer/60224)
- [Manage SAML Certificates](https://support.google.com/a/answer/6245100)
- [Set up Context-Aware Access](https://support.google.com/a/answer/9368756)
- [Google Identity Platform OAuth 2.0](https://developers.google.com/identity/protocols/oauth2)
- [Google OpenID Connect Documentation](https://developers.google.com/identity/openid-connect/openid-connect)
