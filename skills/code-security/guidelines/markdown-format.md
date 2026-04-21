# Security Design Document Formatting

## Purpose

This skill governs the structure, tone, and formatting of all security design documents produced by the code-security-agent. Every `[feature-name]-security-design.md` must follow this template exactly — enabling consistent review, easy Notion import, and clear communication with engineers.

---

## Document Template

```markdown
# [Feature Name] — Security Design

**Date**: YYYY-MM-DD  
**Author**: Code Security Agent  
**Status**: Draft | Under Review | Approved  
**Scope**: [One-sentence description of what is being secured]  
**Related Design**: [link to corresponding backend or frontend design doc, if any]

---

## Table of Contents

1. [Threat Model](#threat-model)
2. [Vulnerabilities Identified](#vulnerabilities-identified)
3. [Security Controls](#security-controls)
4. [Authorization Matrix](#authorization-matrix)
5. [Data Exposure Controls](#data-exposure-controls)
6. [Security Headers](#security-headers)
7. [Dependency Audit](#dependency-audit)
8. [Logging & Monitoring](#logging--monitoring)
9. [Testing Checklist](#testing-checklist)
10. [Residual Risk](#residual-risk)
11. [References](#references)

---

## Threat Model

**Entry Points**: [list all inputs to the feature]  
**Key Assets**: [what is being protected]  

| ID | Actor | Vector | Asset | STRIDE | Impact | Likelihood | Priority |
|----|-------|--------|-------|--------|--------|------------|----------|
| T-01 | | | | | High/Med/Low | High/Med/Low | P1/P2/P3 |

### Mitigations

| Threat ID | Control | Layer |
|-----------|---------|-------|
| T-01 | | Backend/Frontend/Infra |

### Accepted Risks

- [P3 threats intentionally accepted with rationale]

---

## Vulnerabilities Identified

> One section per vulnerability. Severity follows CVSS convention.

### [VULN-001] — [Short Title]

- **Severity**: Critical | High | Medium | Low | Informational
- **Layer**: Frontend | Backend | Infrastructure | Configuration
- **OWASP Category**: [e.g. A01:2021 – Broken Access Control]
- **CWE**: [e.g. CWE-639 – Authorization Bypass Through User-Controlled Key]
- **Description**: [Clear explanation of the vulnerability — what, where, how it can be exploited]
- **Proof of Concept**:
  ```
  [Request/payload/code that demonstrates the issue]
  ```
- **Impact**: [What an attacker gains]
- **Recommendation**: [Specific, actionable fix]
- **Test Case**: [How to verify the fix works]
- **Status**: Open | In Progress | Resolved

---

## Security Controls

Controls to implement or verify, grouped by layer.

### Backend

- [ ] Control description — maps to VULN-XXX or threat T-XX
- [ ] ...

### Frontend

- [ ] Control description
- [ ] ...

### Infrastructure / Configuration

- [ ] Control description
- [ ] ...

---

## Authorization Matrix

| Endpoint | Method | Anonymous | Authenticated (own) | Authenticated (other) | Admin |
|----------|--------|-----------|--------------------|-----------------------|-------|
| | GET | 401 | 200 | 403/404 | 200 |

---

## Data Exposure Controls

### API Response Audit

| Endpoint | Fields Returned | Sensitive Fields Excluded |
|----------|----------------|--------------------------|
| | | |

### Logging Policy

- **Logged**: [fields]
- **Never logged**: [fields]
- **Masked**: [fields and masking pattern]

### PII Retention

| Data Type | Retention | Deletion |
|-----------|-----------|---------|
| | | |

---

## Security Headers

| Header | Value | Status |
|--------|-------|--------|
| Content-Security-Policy | | ✅ / ❌ / ⚠️ |
| Strict-Transport-Security | | |
| X-Frame-Options | | |
| X-Content-Type-Options | | |
| Referrer-Policy | | |
| Permissions-Policy | | |

---

## Dependency Audit

**Date**: YYYY-MM-DD  
**Tool**: npm audit / pip-audit / govulncheck

| Package | Version | CVE | Severity | Action |
|---------|---------|-----|----------|--------|
| | | | | Upgrade to X.X.X |

**Automation**: Dependabot / Renovate configured: ✅ / ❌

---

## Logging & Monitoring

| Event | Log Level | Fields Logged | Alert Threshold |
|-------|-----------|--------------|----------------|
| Failed login | WARN | userId/IP, timestamp | >5 failures/15min |
| 403 Forbidden | WARN | userId, endpoint, timestamp | >20/min |
| 500 Error | ERROR | errorId, path, userId | >10/min |

---

## Testing Checklist

Security test cases — each maps to a vulnerability or control.

### Authentication & Authorization
- [ ] Unauthenticated request to protected endpoint returns `401`
- [ ] Authenticated user cannot access another user's resource (returns `403` or `404`)
- [ ] Admin endpoint returns `403` for non-admin authenticated user
- [ ] JWT with `alg: none` is rejected
- [ ] Expired JWT is rejected

### Input Validation
- [ ] SQL injection payload in string fields returns `400` (not `500`)
- [ ] XSS payload in text input is escaped on render
- [ ] Oversized payload (>limit) returns `413`
- [ ] Invalid UUID in path param returns `400`

### Rate Limiting
- [ ] Login endpoint blocks after 5 failures
- [ ] `429 Too Many Requests` returned with `Retry-After` header

### Data Exposure
- [ ] API response does not include `passwordHash`, tokens, or internal flags
- [ ] Error response contains no stack trace or DB error message
- [ ] Source maps not accessible in production

### Headers
- [ ] CSP header present on all responses
- [ ] HSTS header present on HTTPS responses
- [ ] `X-Powered-By` header absent

---

## Residual Risk

| Risk | Reason Not Mitigated | Accepted By | Review Date |
|------|---------------------|-------------|------------|
| | | | |

---

## References

- OWASP: [link to relevant OWASP page]
- CVE(s): [CVE-XXXX-XXXXX]
- CWE(s): [CWE-XXX]
- Internal: [link to backend design doc, architecture diagram, etc.]
```

---

## Formatting Rules

1. **Severity levels**: Always one of — `Critical`, `High`, `Medium`, `Low`, `Informational`
2. **VULN IDs**: Sequential `VULN-001`, `VULN-002`, etc.
3. **Threat IDs**: Sequential `T-01`, `T-02`, etc.
4. **Status fields**: `Open`, `In Progress`, `Resolved` — nothing else
5. **Checkboxes**: `- [ ]` for pending, `- [x]` for complete
6. **Tables**: All tables use pipe syntax and have headers
7. **Code blocks**: Always specify language (`js`, `bash`, `sql`, `http`)
8. **Tone**: Technical and precise — no vague language ("might", "could potentially", "seems like")
9. **Recommendations**: Always actionable — never "use better security" or "add encryption"

## Severity Definitions

| Severity | Description | SLA |
|----------|-------------|-----|
| **Critical** | Exploitable remotely without auth, leads to full system compromise or data breach | Fix before deploy |
| **High** | Exploitable with low effort, significant data or privilege impact | Fix within 3 days |
| **Medium** | Requires specific conditions or limited impact | Fix within sprint |
| **Low** | Defense-in-depth issue, low exploitability | Track and fix opportunistically |
| **Informational** | Best practice gap, no direct exploitability | Document and accept or backlog |

---

## File Naming

```
designs/[kebab-case-feature-name]-security-design.md

Examples:
  designs/user-auth-security-design.md
  designs/payment-flow-security-design.md
  designs/file-upload-security-design.md
  designs/admin-panel-security-design.md
```
