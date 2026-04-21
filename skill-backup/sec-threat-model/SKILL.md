---
name: sec-threat-model
description: Build structured threat models for any feature, endpoint, or system. Use when the security agent needs to identify actors, attack vectors, impact levels, and likelihood for a given scope. Trigger for any request involving "threat model", "attack surface", "what could go wrong security-wise", or when starting a new security design doc. Always run this skill first before any other security analysis skill.
---

# Threat Modeling

## Purpose

Threat modeling is the foundation of every security design. It answers:
- **Who** attacks this?
- **How** do they attack?
- **What** do they gain?
- **How likely** is it?
- **How bad** is it?

Run this before any other security analysis. Without it, recommendations lack context.

---

## Step 1 — Define the Scope

Before modeling threats, bound the scope clearly:

```
Scope: [feature name / system component]
Entry Points: [all inputs — API endpoints, UI forms, file uploads, env vars, webhooks, queues]
Trust Boundaries: [where data crosses auth zones — frontend→backend, backend→DB, service→service]
Assets: [what is being protected — PII, credentials, money, system integrity, availability]
```

---

## Step 2 — Enumerate Actors

For each actor, define their capabilities and motivation:

| Actor | Access Level | Motivation | Capability |
|-------|-------------|------------|------------|
| Anonymous user | Public endpoints only | Opportunistic | Low–Medium |
| Authenticated user | Owns their data | Privilege escalation | Medium |
| Privileged user / admin | Broad access | Insider threat | High |
| Third-party integration | OAuth / API key | Data exfiltration | Medium |
| Automated bot / scraper | Public surface | Abuse, enumeration | Medium |
| Supply chain (malicious dep) | Build environment | Code execution | High |

Only include actors that are relevant to the scope.

---

## Step 3 — Apply STRIDE

For each entry point or trust boundary, walk through STRIDE:

| Threat | Question to Ask |
|--------|----------------|
| **S**poofing | Can an attacker impersonate a legitimate user or system? |
| **T**ampering | Can data be modified in transit or at rest? |
| **R**epudiation | Can actions be denied / are they logged with integrity? |
| **I**nformation Disclosure | Can sensitive data leak? |
| **D**enial of Service | Can the feature be made unavailable? |
| **E**levation of Privilege | Can a low-privilege actor gain higher access? |

---

## Step 4 — Build the Threat Table

For each identified threat, fill in:

```markdown
## Threat Model — [Feature Name]

| ID | Actor | Vector | Target Asset | STRIDE | Impact | Likelihood | Priority |
|----|-------|--------|-------------|--------|--------|------------|----------|
| T-01 | Anonymous | Brute-force login | User credentials | S | High | High | P1 |
| T-02 | Auth user | IDOR on /api/orders/:id | Other users' data | I | High | Medium | P1 |
| T-03 | Bot | Scrape public pricing API | Business data | I | Low | High | P2 |
```

**Impact scale**: High / Medium / Low  
**Likelihood scale**: High (actively exploited pattern) / Medium (requires skill) / Low (theoretical)  
**Priority**: P1 (fix before launch) / P2 (fix within sprint) / P3 (track and accept)

---

## Step 5 — Map to Controls

For each P1 and P2 threat, name the control that mitigates it:

| Threat ID | Control | Layer | Skill to Use |
|-----------|---------|-------|-------------|
| T-01 | Rate limit + account lockout | Backend | `sec-api-hardening` |
| T-02 | Ownership check before DB query | Backend | `sec-auth-review` |
| T-03 | Authenticated endpoint or CAPTCHA | Backend/Frontend | `sec-api-hardening` |

---

## Output Template

Paste this into the design doc under `## Threat Model`:

```markdown
## Threat Model

**Scope**: [one sentence]
**Entry Points**: [list]
**Key Assets**: [list]

| ID | Actor | Vector | Asset | STRIDE | Impact | Likelihood | Priority |
|----|-------|--------|-------|--------|--------|------------|----------|
| T-01 | | | | | | | |

### Mitigations
| Threat ID | Control | Layer |
|-----------|---------|-------|
| T-01 | | |

### Accepted Risks
- [Any P3 threats intentionally accepted, with rationale]
```

---

## Common Mistakes to Avoid

- **Skipping insider threats** — authenticated users are the most common real-world attackers
- **Ignoring the supply chain** — every `npm install` or `pip install` is a trust decision
- **Treating "low likelihood" as "ignore"** — high-impact + low-likelihood = still model it
- **Modeling at too high a level** — "the API could be attacked" is not a threat; "unauthenticated POST /api/transfer bypasses auth check" is
