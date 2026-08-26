# 5. Google Workspace Master Interview Questions & Scenario Handbook

This single master interview handbook consolidates all technical questions, architectural scenarios, troubleshooting playbooks, and candidate evaluation benchmarks across all Google Workspace administration domains.

---

## 🎯 Section 1: Enterprise IAM, SSO, SAML 2.0, OAuth 2.0 & OIDC (Questions 1–42)

### Q1: What is Single Sign-On (SSO)?
**Answer:**
Single Sign-On (SSO) is an architectural pattern enabling an authenticated principal (user) to obtain access to multiple independent software applications using a single set of credentials, authenticated once at a centralized Identity Provider (IdP).

### Q2: Is SSO a protocol?
**Answer:**
No. SSO is a user experience model and business outcome, not a protocol. SSO is implemented using protocols such as SAML 2.0, OIDC, Kerberos, or WS-Federation.

### Q3: What is an Identity Provider (IdP)?
**Answer:**
An Identity Provider is a centralized system responsible for authenticating user credentials, evaluating access policies, and issuing signed identity assertions or tokens. (Examples: OneLogin, Okta, Entra ID). In our architecture, **OneLogin** is the IdP.

### Q4: What is a Service Provider (SP)?
**Answer:**
A Service Provider (Relying Party) is an application or SaaS platform that relies on an external IdP to authenticate users. In our architecture, **Google Workspace** is the SP.

### Q5: What is Federation?
**Answer:**
Federation is a cryptographically backed trust relationship established between an IdP and an SP, where the SP trusts authentication decisions signed by the IdP's private key.

### Q6: What is SAML 2.0?
**Answer:**
SAML 2.0 (Security Assertion Markup Language) is an XML-based authentication standard used primarily for enterprise federated SSO. SAML answers: **"Who is the user, and was their authentication verified?"**

### Q7: What problem does SAML solve?
**Answer:**
SAML eliminates local password storage in SaaS apps, centralizes access revocation, reduces credential sprawl, and protects against password phishing.

### Q8: Explain the SAML 2.0 SP-Initiated authentication flow.
**Answer:**
1. User requests access to `mail.google.com`.
2. Google Workspace detects domain SSO and redirects user (HTTP 302) to OneLogin with a `SAMLRequest`.
3. User authenticates at OneLogin (Password + WebAuthn FIDO2 MFA).
4. OneLogin generates a signed XML `SAMLResponse` (Assertion).
5. User browser POSTs `SAMLResponse` to Google ACS URL (`https://www.google.com/a/company.com/acs`).
6. Google verifies XML signature using OneLogin's public X.509 certificate and grants a session.

### Q9: What is a SAML Assertion & what does it contain?
**Answer:**
A SAML Assertion is a cryptographically signed XML document containing: User Identity (`NameID`), Issuer, Expiration conditions, Audience URI (`google.com/a/domain.com`), and custom user attributes (`FirstName`, `Department`, `EmployeeID`).

### Q10: What attributes can a SAML assertion transmit?
**Answer:**
NameID (primary email), First Name, Last Name, Department, Employee ID, Job Title, Manager Email, and Group Memberships.

### Q11: How does Google Workspace trust OneLogin?
**Answer:**
Via three core SAML configurations: Sign-in Page URL, Sign-out Page URL, and the public X.509 Verification Certificate.

### Q12: How do you confirm your organization uses SAML?
**Answer:**
In Google Admin Console navigate to **Security > Authentication > SSO with third-party IdP** to view active third-party SSO profiles containing URLs and X.509 certificates.

### Q13: What happens if the SAML X.509 certificate expires?
**Answer:**
All user authentication fails immediately because Google cannot verify assertion signatures. *Mitigation*: Upload the new certificate to Google's Dual Certificate Window before rotating private keys in OneLogin.

### Q14: What happens if the Sign-in URL is incorrect?
**Answer:**
Google cannot redirect users to OneLogin, resulting in connection timeouts or HTTP 404 errors during login attempts.

### Q15: What is OAuth 2.0?
**Answer:**
OAuth 2.0 (RFC 6749) is an authorization framework allowing third-party applications to obtain limited access to HTTP API resources on behalf of a user without sharing passwords. OAuth answers: **"What resources can this application access?"**

### Q16: Is OAuth 2.0 an authentication protocol?
**Answer:**
No. OAuth 2.0 was designed for authorization (delegated API access), issuing access tokens representing permissions rather than user identity.

### Q17: What problem does OAuth 2.0 solve?
**Answer:**
OAuth eliminates the anti-pattern of sharing primary user passwords with third-party applications.

### Q18: Provide an example of OAuth 2.0 in action.
**Answer:**
A photo printing site requests access to Google Photos. User logs into Google, reviews consent screen (`photoslibrary.readonly`), approves access, and Google issues an access token to the printing app.

### Q19: What is an Access Token?
**Answer:**
An Access Token is a credential string presented in `Authorization: Bearer <token>` HTTP headers to call protected REST APIs. It contains scopes, expiration time, and audience data.

### Q20: Does an Access Token identify a user?
**Answer:**
No. Access tokens represent permissions granted to a client application, not user identity.

### Q21: What is an OAuth Scope?
**Answer:**
A scope defines specific permission boundaries granted to an access token (e.g., `https://www.googleapis.com/auth/admin.directory.user`).

### Q22: How is OAuth 2.0 utilized in your enterprise?
**Answer:**
OneLogin uses OAuth 2.0 bearer tokens to authenticate against Google Admin SDK Directory REST APIs for automated Joiner-Mover-Leaver (JML) account creation, updates, and suspensions.

### Q23: What is OIDC?
**Answer:**
OpenID Connect (OIDC) is an identity layer built on top of OAuth 2.0. OIDC answers: **"Who is the user AND what can the application access?"**

### Q24: Why was OIDC created?
**Answer:**
To standardize identity verification across OAuth 2.0 implementations by introducing standard ID Tokens (JWT) and UserInfo endpoints.

### Q25: What does OIDC add to OAuth 2.0?
**Answer:**
OIDC adds ID Tokens (JWT), standardized identity scopes (`openid`, `profile`, `email`), and discovery endpoints (`/.well-known/openid-configuration`).

### Q26: What is an ID Token?
**Answer:**
An ID Token is a cryptographically signed JSON Web Token (JWT) containing user identity claims (`iss`, `sub`, `aud`, `exp`, `email`, `hd`). Structure: `Header.Payload.Signature`.

### Q27: What is the difference between an Access Token and an ID Token?
**Answer:**
Access Tokens are used by APIs for authorization (`scope`). ID Tokens are consumed by client applications for authentication (`identity`).

### Q28: What protocol powers "Sign in with Google"?
**Answer:**
OpenID Connect (OIDC).

### Q29: Why is OIDC preferred for modern applications over SAML?
**Answer:**
OIDC uses lightweight JSON (native to mobile/SPAs), supports PKCE (RFC 7636) for mobile app security, and combines identity proof with API access tokens in a single flow.

### Q30 & Q31: Compare SAML, OAuth 2.0, and OIDC.
*   **SAML 2.0**: XML-based enterprise authentication & SSO.
*   **OAuth 2.0**: Delegated API authorization via access tokens.
*   **OIDC**: Modern JSON identity authentication + API authorization.

### Q32: Describe your organization's authentication architecture.
**Answer:**
User $\rightarrow$ OneLogin IdP (SAML 2.0 SP-Init) $\rightarrow$ Google Workspace SP.

### Q33: Describe your provisioning architecture.
**Answer:**
OneLogin $\rightarrow$ OAuth 2.0 Bearer Token $\rightarrow$ Google Admin SDK Directory API (automated user creation, updates, suspensions, group sync).

### Q34: Explain managed device access in your environment.
**Answer:**
Managed devices (holding Jamf/Intune MDM certificates) connect directly to Google Workspace endpoints.

### Q35: Explain unmanaged device access in your environment.
**Answer:**
Unmanaged/BYOD devices are steered through **Netskope Reverse Proxy (CASB)** for real-time DLP inspection, download prevention, and document watermarking.

### Q36: Why is Netskope not required for managed devices?
**Answer:**
Managed devices already meet enterprise Zero-Trust requirements via enforced disk encryption, EDR software, mTLS device certificates, and corporate management policies.

### Q37: What is Context-Aware Access (CAA)?
**Answer:**
Google Workspace's Zero-Trust policy engine evaluating dynamic access signals (IP range, device posture, location, Chrome management state) using Common Expression Language (CEL).

### Q38: How does Google Workspace determine whether to steer traffic to Netskope?
**Answer:**
Via SAML assertion device trust attributes, Context-Aware Access rules, and alternate SSO Profile endpoints pointing to Netskope reverse proxy nodes.

### Q39: Troubleshooting — Users receive "Invalid SAML Response".
**Answer:**
Check: 1. X.509 certificate expiry, 2. Audience URI / Entity ID mismatch (`google.com/a/domain.com`), 3. ACS URL mismatch (`https://www.google.com/a/domain.com/acs`), 4. NameID format (`emailAddress`), 5. Clock drift (>300s).

### Q40: Troubleshooting — Total OneLogin IdP Outage.
**Answer:**
Log into `admin.google.com` using a Break-Glass Emergency Super Admin account (`admin-breakglass@company.com`) residing in an OU with Third-Party SSO disabled. Change Root OU SSO Profile assignment to **None (Use Google Credentials)** until OneLogin recovers.

### Q41: Troubleshooting — Users repeatedly prompted to authenticate (Session Loop).
**Answer:**
Check OneLogin vs. Google session duration mismatches, Chrome SameSite third-party cookie restrictions, and Netskope proxy SSL decryption token drops.

### Q42: Rapid Recall One-Line Definitions Cheat Sheet:
*   **SSO**: Single Sign-On user experience.
*   **SAML**: XML-based enterprise authentication protocol.
*   **OAuth**: Delegated authorization framework.
*   **OIDC**: Authentication layer on top of OAuth 2.0.
*   **IdP**: System responsible for authenticating users.
*   **SP**: Application trusting the IdP.
*   **SAML Assertion**: Signed XML identity proof.
*   **Access Token**: Delegated API permission credential.
*   **ID Token**: Signed JWT identity proof.
*   **Context-Aware Access**: Zero-Trust CEL access policy engine.

---

## ✉️ Section 2: Core Engineering & Security Scenarios

### Q43: How do SPF, DKIM, and DMARC work together?
**Answer:**
SPF verifies sender IP authorization; DKIM verifies message integrity via cryptographic signatures; DMARC enforces domain alignment (`From:` header matches SPF/DKIM domain) and specifies recipient policy (`none`, `quarantine`, `reject`).

### Q44: What is the Trust Rules precedence rule when conflicting with legacy Drive sharing settings?
**Answer:**
**The most restrictive setting always takes precedence.** If legacy Drive sharing allows external sharing but a Trust Rule blocks it, the share is **BLOCKED**.

---

## 📦 Section 3: Microsoft 365 Migration Scenarios

### Q45: What is the difference between Default Data Import and Advanced Data Import?
**Answer:**
Default Data Import uses shared Google API quota for up to 1,000 users. Advanced Data Import utilizes dedicated Azure App Registrations for enterprise migrations up to 50,000 users.

### Q46: How do you handle SharePoint document library limits when migrating to Shared Drives?
**Answer:**
Google Shared Drives enforce a 400,000 total item limit and 20 subfolder depth limit. Large SharePoint document libraries must be flattened and split into multiple focused Shared Drives during pre-migration planning.

---

## 🛠️ Section 4: GAM CLI & Incident Response Scenarios

### Q47: How do you execute a domain-wide emergency purge of a phishing email?
**Answer:**
```bash
gam all users delete messages query "rfc822msgid:phish123@attacker.com" doit
```

### Q48: How do you debug a SCIM HTTP 409 Conflict error?
**Answer:**
HTTP 409 indicates the primary email or alias already exists in Google Workspace. Search Google Directory via GAM (`gam info user <email>`), resolve alias conflicts, or merge accounts.
