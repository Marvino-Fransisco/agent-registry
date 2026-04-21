# OWASP Top 10 Audit Checklist (2021 Edition)

Use this as a structured walkthrough for any security review. For each category, assess pass / fail / N/A and link to the relevant finding in the security design doc.

Reference: https://owasp.org/Top10/

---

## A01:2021 — Broken Access Control

> Moving up from #5 to #1. 94% of applications tested had some form of broken access control.

**What to check**:
- [ ] Every endpoint verifies authentication (not just public routes)
- [ ] Every resource access verifies ownership (`userId = currentUser.id`)
- [ ] Admin functions behind separate role guard, not just `requireAuth`
- [ ] Directory listing disabled on web server
- [ ] CORS policy restricts `Access-Control-Allow-Origin` to known domains (not `*` for credentialed requests)
- [ ] API does not expose more data than the UI shows (e.g., full object when only name is displayed)
- [ ] HTTP method enforcement: `GET /orders` cannot `DELETE` orders

**CWE mapping**: CWE-22, CWE-284, CWE-285, CWE-639  
**→ Deep dive**: `sec-auth-review`

---

## A02:2021 — Cryptographic Failures

> Previously "Sensitive Data Exposure". Root cause is weak/missing cryptography.

**What to check**:
- [ ] Passwords hashed with bcrypt/argon2id/scrypt (not MD5/SHA1/SHA256)
- [ ] Data in transit: HTTPS enforced everywhere (no HTTP fallback)
- [ ] HSTS header present: `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- [ ] PII and sensitive data encrypted at rest (column-level or disk-level)
- [ ] No sensitive data (PII, tokens) in URLs — use request body or headers
- [ ] Encryption keys stored separately from encrypted data
- [ ] `Math.random()` never used for security-sensitive values — use `crypto.randomBytes()`
- [ ] TLS 1.2 minimum; 1.3 preferred; SSLv3/TLS 1.0/1.1 disabled

**CWE mapping**: CWE-259, CWE-327, CWE-331

---

## A03:2021 — Injection

> SQL, NoSQL, OS command, LDAP, XSS all fall here.

**What to check**:
- [ ] All database queries use parameterized statements or safe ORM APIs
- [ ] No `eval()`, `exec()`, `system()` with user input
- [ ] No string concatenation for shell commands
- [ ] User input sanitized before rendering as HTML
- [ ] ORM raw query methods (`sequelize.literal`, `knex.raw`) audited
- [ ] LDAP queries sanitized if applicable
- [ ] Template engines use auto-escaping (Jinja2, Handlebars, Blade do by default)

**CWE mapping**: CWE-77, CWE-78, CWE-89, CWE-564  
**→ Deep dive**: `sec-input-validation`

---

## A04:2021 — Insecure Design

> New in 2021. Design-level flaws that cannot be patched out — they require redesign.

**What to check**:
- [ ] Threat model completed before implementation (not retrofitted)
- [ ] Rate limiting designed in from the start (not added after abuse occurs)
- [ ] Business logic flaws: can a user apply a discount coupon multiple times?
- [ ] Account recovery flow cannot be used to take over another user's account
- [ ] Negative quantity / negative price scenarios handled in e-commerce
- [ ] Pagination implemented — endpoints cannot return all records with no limit
- [ ] Sensitive operations require re-authentication (password change, 2FA disable, payment)

**CWE mapping**: CWE-73, CWE-183, CWE-209  
**→ Deep dive**: `sec-threat-model`

---

## A05:2021 — Security Misconfiguration

> The most commonly found issue. Includes default credentials, verbose errors, open cloud storage.

**What to check**:
- [ ] Default credentials changed on all components (databases, admin panels, cloud services)
- [ ] Error responses show generic messages to clients — no stack traces, DB errors, or paths
- [ ] Development features disabled in production (`DEBUG=false`, no `/swagger` in prod unless intended)
- [ ] Unnecessary features/services disabled (ports, services, pages)
- [ ] Security headers present (CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy)
- [ ] S3 / Blob storage buckets not publicly readable unless intentional CDN assets
- [ ] Cloud security groups / firewall rules: only necessary ports open
- [ ] XML external entity processing disabled (if using XML parsers)

**CWE mapping**: CWE-16, CWE-611  
**→ Deep dive**: `sec-csp-policy`, `sec-api-hardening`

---

## A06:2021 — Vulnerable and Outdated Components

> Using components with known vulnerabilities.

**What to check**:
- [ ] `npm audit` / `pip-audit` run — zero high/critical findings
- [ ] All direct dependencies are up to date
- [ ] Base Docker images pinned and recently updated
- [ ] OS-level packages updated (no unpatched CVEs on server OS)
- [ ] End-of-life runtimes not used (Node.js 14, Python 2, PHP 7.x, etc.)
- [ ] Dependabot or Renovate configured for automated updates

**CWE mapping**: CWE-1035, CWE-1104  
**→ Deep dive**: `sec-dependency-audit`

---

## A07:2021 — Identification and Authentication Failures

> Previously "Broken Authentication".

**What to check**:
- [ ] No default/weak passwords permitted
- [ ] Brute force protection: rate limiting + lockout on login and OTP endpoints
- [ ] Session IDs are cryptographically random (not predictable)
- [ ] Sessions invalidated server-side on logout
- [ ] JWT algorithm pinned — `alg: none` rejected
- [ ] Password reset tokens single-use and short-lived (≤ 15min)
- [ ] Account enumeration prevented (consistent error messages and timing)
- [ ] MFA available for privileged accounts

**CWE mapping**: CWE-255, CWE-259, CWE-287, CWE-330  
**→ Deep dive**: `sec-auth-review`

---

## A08:2021 — Software and Data Integrity Failures

> New in 2021. CI/CD pipeline compromises, insecure deserialization, unsigned updates.

**What to check**:
- [ ] CI/CD pipeline uses pinned action versions (not `@main` or `@latest`)
- [ ] No deserialization of untrusted data in non-safe languages (Java, PHP, Python pickle)
- [ ] Subresource Integrity (SRI) on third-party scripts and stylesheets
- [ ] Signed commits or artifact signing for release pipeline
- [ ] Webhooks verified with HMAC signatures
- [ ] Software update mechanism uses signed packages

```html
<!-- SRI example -->
<script
  src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.21/lodash.min.js"
  integrity="sha512-..."
  crossorigin="anonymous">
</script>
```

**CWE mapping**: CWE-345, CWE-353, CWE-426, CWE-494

---

## A09:2021 — Security Logging and Monitoring Failures

> Without logging, breaches go undetected. Average time to detect a breach is 207 days.

**What to check**:
- [ ] Auth events logged: login, logout, failed login, password reset, MFA events
- [ ] Access control failures logged (403s)
- [ ] High-value transactions logged with enough context to reconstruct
- [ ] Logs stored separately from application (not just in app container)
- [ ] Logs include: timestamp, user ID, IP, action, resource, outcome
- [ ] No sensitive data in logs (passwords, tokens, full PAN)
- [ ] Alert on suspicious patterns: brute force, impossible travel, mass data access
- [ ] Log retention policy defined (typically 90 days hot, 1 year cold)

**CWE mapping**: CWE-117, CWE-223, CWE-532, CWE-778  
**→ Deep dive**: `sec-data-exposure-review`

---

## A10:2021 — Server-Side Request Forgery (SSRF)

> New in 2021. Attacker causes server to make requests to internal/external resources.

**What to check**:
- [ ] Any feature accepting a URL as input (webhook, image import, link preview, PDF generator) is audited for SSRF
- [ ] URL inputs validated against allowlist of permitted domains (not denylist of internal IPs — those are bypassed)
- [ ] DNS rebinding mitigated: resolve and validate IP at request time, not just at input time
- [ ] Requests from server never reach cloud metadata endpoint: `169.254.169.254`
- [ ] Internal services not reachable from SSRF vector (firewall blocks `10.x`, `192.168.x`, `172.16.x`)

```js
// ❌ Vulnerable to SSRF
const html = await fetch(req.body.url).then(r => r.text());

// ✅ Domain allowlist
const url = new URL(req.body.url);
const ALLOWED_DOMAINS = ['api.example.com', 'cdn.example.com'];
if (!ALLOWED_DOMAINS.includes(url.hostname)) throw new Error('Domain not allowed');
```

**CWE mapping**: CWE-918

---

## Audit Summary Template

```markdown
## OWASP Top 10 Audit — [Feature/System Name]

**Date**: YYYY-MM-DD  
**Auditor**: [name]

| # | Category | Status | Notes |
|---|----------|--------|-------|
| A01 | Broken Access Control | ✅ Pass / ❌ Fail / ⚠️ Partial | |
| A02 | Cryptographic Failures | | |
| A03 | Injection | | |
| A04 | Insecure Design | | |
| A05 | Security Misconfiguration | | |
| A06 | Vulnerable Components | | |
| A07 | Auth Failures | | |
| A08 | Integrity Failures | | |
| A09 | Logging Failures | | |
| A10 | SSRF | | |

**Findings requiring action**:
[link to VULN-XXX entries in design doc]
```
