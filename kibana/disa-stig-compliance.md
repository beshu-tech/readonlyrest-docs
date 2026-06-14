---
description: >-
  DISA STIG compliance analysis for deployments using the ReadonlyREST
  plugin as the authentication and authorization enforcement point.
---

# DISA STIG Compliance — ReadonlyREST Kibana Plugin

This document answers DISA STIG (Web Server Security Requirements Guide) compliance questions for deployments where the ReadonlyREST Kibana plugin serves as the authentication and authorization enforcement point.

---

## 1. Session Management

### V-206351 — Server-side session management

**Status: Fully satisfied**

ReadonlyREST stores all session state on the server side. The client-side cookie contains only an encrypted session identifier — no session data is stored in the cookie itself.

Two storage backends are available:
- **In-memory** (default): suitable for single Kibana instance deployments.
- **Elasticsearch index** (recommended for HA): sessions are persisted in a dedicated index shared across all Kibana nodes. Enabled via `readonlyrest_kbn.store_sessions_in_index: true`.

---

### V-206396 — Invalidate session identifiers upon logout or session termination

**Status: Fully satisfied**

On logout, ReadonlyREST deletes the session record from the server-side store and clears the client cookie in the same operation. Once a session is deleted, any subsequent request presenting the old cookie is rejected and redirected to the login page.

For multi-tab browser scenarios, a background probe running in each tab detects the logout event and immediately redirects all open tabs to login.

---

### V-206397 — Cookie security settings (HttpOnly, Secure flags; SameSite)

**Status: Fully satisfied**

ReadonlyREST sets the following flags on every session cookie:

| Flag | Default | Notes |
|------|---------|-------|
| `HttpOnly` | `true` (always) | Cannot be disabled. |
| `Secure` | `true` when TLS is active | Automatically enabled when Kibana is configured with SSL, or when `readonlyrest_kbn.cookies.secure: true` is set explicitly (required for NGINX SSL termination). |
| `SameSite` | `Lax` | Configurable to `Strict` (recommended for management interfaces) or `None`. |

All cookie attributes are configurable in `kibana.yml`:

```yaml
readonlyrest_kbn.cookieName: rorCookie
readonlyrest_kbn.cookiePass: <minimum-32-character-secret>
readonlyrest_kbn.cookies.sameSite: strict   # lax | strict | none
readonlyrest_kbn.cookies.secure: true       # explicit override
```

For NGINX reverse-proxy deployments where SSL is terminated at the proxy, set `readonlyrest_kbn.cookies.secure: true` to ensure the `Secure` flag is applied to cookies even though Kibana runs over plain HTTP internally.

---

### V-206398 — Accept only system-generated session identifiers

**Status: Fully satisfied**

ReadonlyREST rejects any session identifier it did not create through two independent checks:

1. The cookie is encrypted with HAPI Iron (AES-256 + HMAC-SHA256) using a secret key known only to the ReadonlyREST deployment. Any tampered or externally crafted cookie fails decryption.
2. Even a structurally valid identifier must exist as an active record in the server-side session store. Identifiers not present in the store are rejected.

---

### V-206399 — Session ID generation using FIPS 140-2 approved RNG *(HIGH)*

**Status: Conditionally satisfied — depends on the Node.js runtime**

ReadonlyREST generates session IDs using the Node.js cryptographic random number generator (`crypto.randomBytes()`), which uses the operating system CSPRNG. When the Kibana process is started with a FIPS-validated Node.js build or with the `--enable-fips` flag, this call automatically uses the FIPS-validated OpenSSL DRBG, satisfying the requirement.

ReadonlyREST inherits the cryptographic posture of the Kibana runtime — enabling FIPS mode is an infrastructure and deployment concern, not a ReadonlyREST configuration option.

---

### V-206400 — Non-reproducible session identifiers

**Status: Fully satisfied**

Session IDs are UUID v4 values generated from 122 bits of independent random entropy per call. The same RNG call never produces the same output.

---

### V-206401 — Session ID length ≥ 128 bits

**Status: Fully satisfied**

UUID v4 is a 128-bit value. ReadonlyREST uses it as the session key.

---

### V-206402 — Session ID character set (A–Z, a–z, 0–9 minimum)

**Status: Fully satisfied**

UUID v4 is represented using hexadecimal characters (`0–9`, `a–f`), which satisfies the minimum alphanumeric requirement.

---

### V-206403 — Session ID entropy ≥ 50% of ID length

**Status: Fully satisfied**

UUID v4 carries 122 bits of random entropy in a 128-bit value — 95% entropy density, well above the 50% threshold.

---

### V-206414 — Absolute session timeout ≤ 8 hours

**Status: Partially satisfied — requires configuration; note architectural limitation**

ReadonlyREST enforces a configurable session timeout via `readonlyrest_kbn.session_timeout_minutes` (default: 4320 minutes / 3 days). For STIG compliance this must be set to 480 minutes (8 hours) or less:

```yaml
readonlyrest_kbn.session_timeout_minutes: 480
```

**Limitation:** ReadonlyREST's timeout is a sliding inactivity window — each user action resets the clock. There is no hard absolute cap on total session lifetime from the moment of login. A continuously active user will not be forcibly logged out after 8 hours. ReadonlyREST creates and manages its own session independently of the IdP after the initial authentication, so there is no external control point that can enforce an absolute lifetime on an active ReadonlyREST session. Strict absolute session lifetime enforcement is a known limitation of the current ReadonlyREST implementation.

---

### V-206415 — Inactive/idle session timeout

**Status: Fully satisfied — requires configuration**

ReadonlyREST terminates sessions that have been idle for longer than the configured timeout. The same `session_timeout_minutes` setting controls the idle window. Background cleanup runs automatically and removes expired sessions from the store. A client-side probe (every 30 seconds) independently detects session expiry in the browser and triggers immediate logout.

Recommended configuration:

```yaml
readonlyrest_kbn.session_timeout_minutes: 30   # example: 30-minute idle timeout
```

---

## 2. Session IP Binding

### V-264360 — Restrict management sessions to consistent inbound source IP
### V-264361 — Restrict user sessions to consistent inbound source IP

**Status: Not implemented**

ReadonlyREST does not bind sessions to an originating IP address. A session token is valid regardless of the IP from which it is presented.

IP session binding is a known limitation of the current ReadonlyREST implementation.

---

## 3. Authorization & Access Control

### V-206355 — Enforce approved authorizations for logical access (RBAC)

**Status: Fully satisfied**

ReadonlyREST is the RBAC enforcement point for Kibana. Access control is driven by the ReadonlyREST ACL configuration, which maps authenticated identities (users, SAML attributes, OIDC claims, group memberships) to permissions. The following are enforced per session:

- **Kibana tenants / spaces:** each user or group is confined to a specific Kibana space, isolating dashboards and saved objects.
- **Elasticsearch index access:** users can only query the indices permitted by their ACL block.
- **API path restrictions:** specific Kibana API endpoints can be allowed or denied per user or group.
- **Access level:** read-only, read-write, or admin access within Kibana is configurable per ACL block.

---

### V-206394 — Prohibit anonymous user access / prevent unauthorized changes

**Status: Fully satisfied**

Every request passing through ReadonlyREST requires a valid authenticated session. Unauthenticated requests are redirected to the login page before reaching Kibana. The only paths that bypass authentication are health-check endpoints (`/status`, `/api/status` by default), which expose no user data and allow no modifications. These whitelisted paths are configurable.

---

### V-264342 — Individual authentication before shared account access

**Status: Fully satisfied at the plugin level**

ReadonlyREST requires each session to be established through an individual authentication event — credentials, a SAML assertion, or an OIDC token exchange. Every session is tied to a specific authenticated identity and carries its own server-side record.

Whether multiple people share the same IdP credentials is outside ReadonlyREST's scope — that is an identity provider concern.

---

## 4. Authentication

### V-222523 — Multi-Factor Authentication for privileged accounts *(CAT I)*

**Status: Conditionally satisfied — depends on IdP configuration**

ReadonlyREST does not implement MFA natively. Authentication is fully delegated to external identity providers via SAML or OIDC. When the IdP enforces MFA, that requirement is satisfied before ReadonlyREST issues a session — ReadonlyREST neither bypasses nor weakens IdP-side MFA policies.

Deployments not using SAML or OIDC have no MFA enforcement point at the ReadonlyREST Kibana plugin layer. MFA for such deployments requires migrating to SAML or OIDC with an IdP that enforces MFA.

---

### V-222543 — Plaintext credential transmission *(CAT I)*

**Status: Conditionally satisfied — requires TLS configuration**

The exposure of credentials in transit depends on the authentication method in use:

- **SAML / OIDC deployments:** ReadonlyREST does not handle raw credentials directly — authentication relies on SAML assertions or OIDC token exchanges. Only session tokens are transmitted between the browser and Kibana, and these are exposed if TLS is not configured.
- **Basic auth deployments (`auth_key`):** Username and password are transmitted from the browser on every request. Without TLS, credentials are exposed in plaintext on every authenticated request.

TLS is mandatory for STIG compliance regardless of the authentication method.

TLS is enabled in `kibana.yml`:

```yaml
server.ssl.enabled: true
server.ssl.key: /path/to/server.key
server.ssl.certificate: /path/to/server.crt
server.ssl.supportedProtocols: ["TLSv1.2", "TLSv1.3"]
```

Or using a PKCS#12 keystore:

```yaml
server.ssl.enabled: true
server.ssl.keystore.path: /path/to/server.p12
server.ssl.keystore.password: <keystore-password>
server.ssl.supportedProtocols: ["TLSv1.2", "TLSv1.3"]
```

---

## 5. Transport Security

### V-222596 — TLS protocol version enforcement *(CAT I)*

**Status: Conditionally satisfied — requires configuration**

When TLS is enabled, ReadonlyREST enforces a minimum protocol baseline: TLSv1.0, SSLv2, and SSLv3 are always disabled. TLSv1.1, TLSv1.2, and TLSv1.3 are permitted by default.

DISA STIG requires a minimum of TLS 1.2. Restrict the allowed protocols via the standard Kibana SSL setting in `kibana.yml`:

```yaml
server.ssl.enabled: true
server.ssl.supportedProtocols: ["TLSv1.2", "TLSv1.3"]
```

ReadonlyREST respects this setting and applies the configured protocol restrictions across all TLS connections.

---

### V-222571 — Cryptographic algorithms *(CAT I)*

**Status: Fully satisfied**

ReadonlyREST uses only modern, approved cryptographic algorithms internally:

| Usage | Algorithm |
|---|---|
| Session cookie encryption | AES-256 (HAPI Iron — AES-CBC + HMAC-SHA256) |
| Session and tenancy data encryption | AES (CryptoJS) |
| License token verification | ES512 (ECDSA with SHA-512) |

No deprecated algorithms are present in the ReadonlyREST codebase. MD5, SHA-1, DES, and RC4 are not used.

---

## 6. Audit & Logging

### V-222452 — Failed login attempt logging *(CAT II)*

**Status: Conditionally satisfied — requires audit configuration**

The ReadonlyREST Elasticsearch plugin provides formal structured audit logging for all access control decisions, including rejected authentication attempts. When enabled, FORBIDDEN events are written to a timestamped Elasticsearch index (`readonlyrest_audit-YYYY-MM-DD`) with structured fields including `final_state`, `logged_user`, `presented_identity`, `error_type`, and `error_message`.

Enable audit logging in `readonlyrest.yml`:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
      - type: index
```

The Kibana plugin additionally logs rejected attempts at `INFO` level in the Kibana application log (e.g. "Could not login in: …"), visible in standard production deployments.

---

### V-222463 — Privileged user action logging *(CAT II)*

**Status: Conditionally satisfied — requires audit configuration**

The ReadonlyREST Elasticsearch plugin routes all requests — including privileged admin API calls — through the same ACL evaluation and audit pipeline. When audit logging is enabled, every request is written to the audit index. See the [audit log documentation](https://docs.readonlyrest.com/elasticsearch/audit) for the full list of available fields.

Enable audit logging in `readonlyrest.yml`:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
      - type: index
```

For maximum field coverage including the request path and ACL history, use `FullAuditLogSerializer`:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
      - type: index
        serializer: tech.beshu.ror.audit.instances.FullAuditLogSerializer
```

---

### V-222507 — Audit log integrity *(CAT II)*

**Status: Partially satisfied — access control covered, tamper-detection outside scope**

ReadonlyREST writes audit events to a timestamped Elasticsearch index (`readonlyrest_audit-YYYY-MM-DD`). The ACL can be used to prevent unauthorized modification or deletion of those indices. Add the following rule to `readonlyrest.yml`, placed **after** any existing allow rules for the Kibana server user (ACL blocks are evaluated top-to-bottom; the first matching rule wins):

```yaml
- name: "Protect audit index"
  type: forbid
  indices: ["readonlyrest_audit-*"]
  actions: ["indices:data/write/*", "indices:admin/delete"]
```

This satisfies the separation-of-privilege aspect of AU-9: no regular user can alter or delete audit data, while the Kibana server user — already permitted by its own allow rule above — retains the access it requires.

Tamper detection (log signing, hash chaining) and audit log backup are outside ReadonlyREST scope and must be addressed at the Elasticsearch/infrastructure layer.

---

## 7. HTTP Security Headers

### V-222602 — Content-Security-Policy *(CAT II)*

**Status: Partially satisfied — architectural limitation in Kibana**

`Content-Security-Policy` is managed by Kibana, not ReadonlyREST. Kibana 7.x and 8.x do not set a CSP header by default — configure it explicitly in `kibana.yml`.

Kibana's frontend architecture hardcodes `'unsafe-inline'` in `style-src` regardless of configuration — this cannot be removed without breaking the UI. Additionally, Kibana 7.x and 8.x hardcode `'unsafe-eval'` in `script-src`; this was removed in Kibana 9.x. The configuration below restricts what operators can control:

Kibana 7.9 – 7.13 (only `csp.rules` is available):
```yaml
csp.rules:
  - "script-src 'self'"
  - "style-src 'self'"
  - "object-src 'none'"
```

Kibana 7.14+ and 8.x (per-directive settings):
```yaml
csp.script_src: ["'self'"]
csp.style_src: ["'self'"]
csp.object_src: ["'none'"]   # 8.x only; not available in 7.x
```

The resulting effective policy will always contain `'unsafe-inline'` in `style-src` (all versions) and `'unsafe-eval'` in `script-src` (Kibana 7.x and 8.x). These weaken the CSP and should be documented as accepted risks in the system's security assessment.

---

### X-Frame-Options — Clickjacking prevention *(CAT II)*

**Status: Outside ReadonlyREST scope — requires Kibana configuration**

`X-Frame-Options` is managed by Kibana, not ReadonlyREST. Configure it explicitly in `kibana.yml`:

```yaml
server.customResponseHeaders:
  X-Frame-Options: "DENY"
```


---

### Strict-Transport-Security (HSTS) *(CAT II)*

**Status: Outside ReadonlyREST scope — requires Kibana configuration**

`Strict-Transport-Security` is managed by Kibana, not ReadonlyREST. Configure it explicitly in `kibana.yml` (requires TLS to be enabled):

```yaml
server.customResponseHeaders:
  Strict-Transport-Security: "max-age=31536000; includeSubDomains"
```


---

## Summary

| STIG Control | Requirement | CAT | Status |
|---|---|---|---|
| V-206351 | Server-side session state | II | ✅ Fully satisfied |
| V-206396 | Invalidate on logout | II | ✅ Fully satisfied |
| V-206397 | HttpOnly, Secure, SameSite | II | ✅ Fully satisfied |
| V-206398 | Only system-generated SIDs | II | ✅ Fully satisfied |
| V-206399 | FIPS 140-2 RNG | I | ⚠️ Depends on Node.js FIPS mode |
| V-206400 | Non-reproducible SIDs | II | ✅ Fully satisfied |
| V-206401 | SID ≥ 128 bits | II | ✅ Fully satisfied |
| V-206402 | SID charset A–Z, a–z, 0–9 | II | ✅ Fully satisfied |
| V-206403 | Entropy ≥ 50% of SID length | II | ✅ Fully satisfied |
| V-206414 | Absolute timeout ≤ 8 hours | II | ⚠️ Sliding timeout only; requires configuration |
| V-206415 | Idle/inactivity timeout | II | ✅ Satisfied — requires configuration |
| V-264360 | IP binding — management sessions | II | 🔴 Not implemented |
| V-264361 | IP binding — user sessions | II | 🔴 Not implemented |
| V-206355 | RBAC logical access control | II | ✅ Fully satisfied |
| V-206394 | No anonymous access | II | ✅ Fully satisfied |
| V-264342 | Individual auth before shared access | II | ✅ Satisfied at plugin level |
| V-222523 | MFA for privileged accounts | I | ⚠️ Depends on IdP — not available without SAML/OIDC |
| V-222543 | Plaintext credential transmission | I | ⚠️ Requires TLS configuration |
| V-222596 | TLS protocol version (min. 1.2) | I | ⚠️ Conditionally satisfied — requires configuration |
| V-222571 | Cryptographic algorithms (no deprecated) | I | ✅ Fully satisfied |
| V-222452 | Failed login attempt logging | II | ⚠️ Conditionally satisfied — requires audit configuration |
| V-222463 | Privileged user action logging | II | ⚠️ Conditionally satisfied — requires audit configuration |
| V-222507 | Audit log integrity | II | ⚠️ Partially satisfied — access control via ACL; tamper-detection outside scope |
| V-222602 | Content-Security-Policy | II | ⚠️ Partially satisfied — style-src 'unsafe-inline' is a Kibana architectural limitation |

**Legend:**
- ✅ — satisfied (by default or via documented configuration)
- ⚠️ — requires configuration; control is not met if skipped
- 🔴 — not implemented
