---
name: sec-dependency-audit
description: Audit project dependencies for known vulnerabilities, supply chain risks, outdated packages, and insecure lockfile practices. Use when reviewing package.json, requirements.txt, Gemfile, go.mod, Cargo.toml, Dockerfile, or any dependency manifest. Trigger for mentions of CVE, npm audit, pip-audit, outdated packages, supply chain, dependency vulnerabilities, or when doing a pre-launch security review. Covers Node.js, Python, Go, Ruby, Rust, and Docker base images.
---

# Dependency Security Audit

## Why Dependencies Are a Top Risk

The average production application has 100–1000 transitive dependencies. Each one is:
- Code you didn't write
- Code you may never read
- Code that can be compromised at any point (maintainer account takeover, malicious update, typosquat)

Dependency attacks are now among the most common supply chain vectors (Log4Shell, node-ipc, event-stream, xz-utils).

---

## Audit by Ecosystem

### Node.js / npm

```bash
# Check for known CVEs
npm audit

# Auto-fix non-breaking patches
npm audit fix

# Show only high/critical
npm audit --audit-level=high

# Full dependency tree
npm ls --all

# Check for outdated packages
npm outdated
```

**Interpreting audit output**:
- **Critical** — exploitable remotely, fix immediately
- **High** — significant risk, fix before next release
- **Moderate** — fix within sprint
- **Low** — track, fix opportunistically

**Lockfile rules**:
- [ ] `package-lock.json` or `yarn.lock` committed to version control
- [ ] CI runs `npm ci` (not `npm install`) — uses lockfile exactly
- [ ] `npm install` never run in production — only `npm ci`
- [ ] No `^` or `~` ranges in `dependencies` for production apps (pin exact versions)

---

### Python / pip

```bash
# Audit installed packages
pip install pip-audit --break-system-packages
pip-audit

# Or use safety (requires free account)
pip install safety
safety check -r requirements.txt

# Check outdated
pip list --outdated

# Generate lockfile from requirements.in
pip-compile requirements.in --generate-hashes
```

**Checklist**:
- [ ] `requirements.txt` uses pinned versions (`==`) not ranges (`>=`)
- [ ] Hash verification enabled: `pip install --require-hashes -r requirements.txt`
- [ ] `pip-audit` runs in CI pipeline
- [ ] Virtual environment used — no global package installs

---

### Go

```bash
# Check for vulnerabilities
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...

# List dependencies
go list -m all

# Check outdated
go list -m -u all
```

**Checklist**:
- [ ] `go.sum` committed — provides hash verification of all deps
- [ ] `govulncheck` in CI pipeline
- [ ] Only import packages from trusted, maintained sources

---

### Ruby / Bundler

```bash
bundle audit check --update
bundler-audit check
```

**Checklist**:
- [ ] `Gemfile.lock` committed
- [ ] `bundle audit` in CI

---

### Docker Base Images

```dockerfile
# ❌ Floating tag — you don't know what you'll get tomorrow
FROM node:20

# ❌ Latest — worst possible choice
FROM node:latest

# ✅ Pinned to digest — immutable
FROM node:20.11.0-alpine3.19@sha256:a1234567...

# ✅ Acceptable — pinned minor version
FROM node:20.11.0-alpine3.19
```

**Checklist**:
- [ ] Base images pinned to specific version tag (not `latest`)
- [ ] Prefer `alpine` or `distroless` variants — smaller attack surface
- [ ] Run `docker scout cves <image>` or `trivy image <image>` in CI
- [ ] Base image updated at least monthly (automated via Dependabot/Renovate)
- [ ] Multi-stage builds — build tools not present in final image

---

## Supply Chain Risk Assessment

Beyond known CVEs, evaluate each critical dependency:

| Risk Factor | Questions to Ask |
|-------------|-----------------|
| **Maintainer count** | Is this a one-person project? What if they abandon it? |
| **Last publish date** | Unmaintained for 2+ years? Likely unpatched vulnerabilities |
| **Download count** | Low downloads = less community security scrutiny |
| **Typosquatting** | Does the package name look like a common typo of a popular package? |
| **Permissions requested** | Does an npm `postinstall` script run? Does it make network calls? |
| **Ownership changes** | Has the package changed maintainers recently? |

### High-Risk Package Patterns

Flag for manual review any package that:
- Has `preinstall`, `postinstall`, or `install` scripts (runs arbitrary code on `npm install`)
- Requests filesystem or network access in a non-obvious way
- Was recently transferred to a new owner
- Has fewer than 1000 weekly downloads but is a direct dependency
- Is a fork of a popular package with a slightly different name

---

## Automated Dependency Management

Recommend setting up automated PR generation for dependency updates:

### Dependabot (GitHub)

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Renovate

```json
// renovate.json
{
  "extends": ["config:base"],
  "schedule": ["every weekend"],
  "automerge": true,
  "automergeType": "pr",
  "requiredStatusChecks": ["ci"]
}
```

---

## CI Integration Template

Add to your CI pipeline:

```yaml
# GitHub Actions example
- name: Dependency Audit
  run: |
    npm audit --audit-level=high
    # Fail pipeline on high/critical findings
```

```yaml
# Python
- name: pip-audit
  run: |
    pip install pip-audit
    pip-audit -r requirements.txt --fail-on-vuln
```

---

## Output Format for Design Doc

```markdown
## Dependency Risk Assessment

**Audit Date**: YYYY-MM-DD  
**Tool Used**: npm audit / pip-audit / govulncheck  
**Findings**:

| Package | Version | CVE | Severity | Status |
|---------|---------|-----|----------|--------|
| lodash | 4.17.20 | CVE-2021-23337 | High | → Upgrade to 4.17.21 |
| axios | 0.21.1 | CVE-2021-3749 | Medium | → Upgrade to 0.21.4 |

**Supply Chain Notes**:
- [any manual review flags]

**Automation**:
- [ ] Dependabot/Renovate configured
- [ ] Audit step in CI pipeline
```
