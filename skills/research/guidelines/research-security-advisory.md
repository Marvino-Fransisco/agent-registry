# Research: Security Advisory

## Purpose
Surface known vulnerabilities, security anti-patterns, and best practices for a technology or design. Produce findings that are actionable: what is the risk, how severe, and what is the fix or mitigation.

## Investigation areas

### 1. Known CVEs and advisories
```bash
# Check npm audit for JS projects
npm audit --json 2>/dev/null | jq '.vulnerabilities | keys'

# Check Python dependencies
pip-audit --format json 2>/dev/null | jq '.[].name, .[].vulns[].id'

# Check cargo (Rust)
cargo audit --json 2>/dev/null | jq '.vulnerabilities.list[].advisory.id'

# Search NVD (National Vulnerability Database) via API
curl "https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=<library>&resultsPerPage=10" \
  | jq '.vulnerabilities[].cve | {id: .id, description: .descriptions[0].value, severity: .metrics}'

# Check OSV (Open Source Vulnerabilities)
curl -X POST "https://api.osv.dev/v1/query" \
  -H "Content-Type: application/json" \
  -d '{"package": {"name": "<package>", "ecosystem": "<npm|PyPI|crates.io>"}}' \
  | jq '.vulns[] | {id, summary, severity: .severity[0].score}'
```

### 2. Authentication & Authorization review
- Is the library used for auth? Check if it implements OAuth 2.0 / OIDC correctly
- JWT: does it validate `alg` field? Vulnerable to `alg: none` attack?
- Session management: does it rotate tokens on privilege escalation?
- Does it support MFA, PKCE, refresh token rotation?

### 3. Input validation & injection risks
- SQL injection surface: does the library use parameterized queries?
- XSS: does it sanitize output or rely on the consumer to do it?
- SSRF: does it make outbound requests based on user input?
- Prototype pollution (JS libraries): check if `__proto__` or `constructor` merge is possible

### 4. Cryptography
- Does it use deprecated algorithms? (MD5, SHA1, DES, RC4, ECB mode)
- Key length — RSA < 2048 bits is considered weak; RSA 4096 or ECDSA P-256 preferred
- Does it handle entropy correctly (no `Math.random()` for secrets)?
- TLS version — is it enforcing TLS 1.2+ and rejecting TLS 1.0/1.1?

### 5. Secrets & configuration
- Does the library expose secrets in logs, error messages, or stack traces?
- Does it support environment variable injection (no hardcoded secrets)?
- Does it have a clear secrets rotation path?

## Severity classification (use CVSS standard)
| Score | Severity | Action |
|-------|----------|--------|
| 9.0–10.0 | Critical | Block adoption / patch immediately |
| 7.0–8.9 | High | Patch within 7 days, mitigate immediately |
| 4.0–6.9 | Medium | Patch within 30 days |
| 0.1–3.9 | Low | Patch in next release cycle |
| 0.0 | Informational | Best practice, not a vulnerability |

## Output format
For each finding:
```
### [CVE-XXXX-XXXXX] Short title
- **Severity**: Critical / High / Medium / Low
- **CVSS Score**: X.X
- **Affected versions**: < X.Y.Z
- **Fixed in**: X.Y.Z
- **Description**: What the vulnerability is
- **Exploit scenario**: How an attacker would use it
- **Mitigation**: Upgrade / workaround / config change
- **Reference**: [NVD link] [GitHub advisory link]
```

## Constraints
- Never dismiss a High/Critical CVE because "we don't use that feature" without explicit confirmation from the user
- Always distinguish: **vulnerability** (proven exploitable) vs **weakness** (poor practice, not yet exploited)
- Flag if a library has had repeated CVEs in the same category — indicates systemic security hygiene issues
- Do not fabricate CVE IDs — if uncertain, say "check NVD for <library>" instead of guessing
