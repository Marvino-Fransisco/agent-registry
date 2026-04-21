---
name: research-dependency-audit
description: >
  Use this skill to audit a project's dependencies for risk: abandonment, license conflicts,
  supply chain vulnerabilities, bloat, and outdated packages. Trigger when the user asks
  "are our dependencies safe?", "which packages are outdated?", "is this package still
  maintained?", "do we have any GPL dependencies we shouldn't?", "what's our dependency
  risk exposure?", "can we reduce our bundle size?", or any request to evaluate the health,
  safety, or footprint of the project's dependency tree. Also trigger before major version
  upgrades or before a security review.
---

# Research: Dependency Audit

## Purpose
Produce a dependency health report that surfaces: security vulnerabilities, abandoned packages, license risks, duplicated packages, and unused dependencies — prioritized by severity so the team knows what to fix first.

## Audit dimensions

### 1. Security vulnerabilities
```bash
# Node.js / npm
npm audit --json | jq '.vulnerabilities | to_entries[] | {name: .key, severity: .value.severity, via: .value.via[0]}'

# Python
pip-audit --format json | jq '.[] | {name, version, vulns: [.vulns[].id]}'

# Rust
cargo audit --json | jq '.vulnerabilities.list[] | {crate: .package.name, advisory: .advisory.id, severity: .advisory.severity}'

# Go
govulncheck ./... 2>&1 | head -50

# Ruby
bundle audit check --update
```

### 2. Outdated packages
```bash
# npm - show outdated with current vs wanted vs latest
npm outdated --json | jq 'to_entries[] | {pkg: .key, current: .value.current, latest: .value.latest, type: .value.type}'

# Python
pip list --outdated --format json | jq '.[] | {name, version, latest_version}'

# Check if outdated package has a breaking major version change
# Convention: semver major bump = breaking, check changelogs
```

### 3. Abandoned packages
A package is a risk if **any two** of the following are true:
- Last release > 12 months ago
- Last commit > 6 months ago
- Open issues > 200 with no recent activity
- Maintainer account deactivated or transferred
- `deprecated` notice on npm/PyPI registry

```bash
# npm deprecation check
npm view <package> deprecated 2>/dev/null

# Check last publish date
npm view <package> time.modified

# GitHub archive check
gh repo view <owner>/<repo> --json isArchived,updatedAt
```

### 4. License audit
| License | Risk level | Notes |
|---------|-----------|-------|
| MIT, Apache-2.0, BSD-2/3 | ✅ Safe | Permissive, commercial use allowed |
| ISC, Unlicense, CC0 | ✅ Safe | Effectively public domain |
| LGPL-2.1, LGPL-3.0 | ⚠️ Review | Safe if linked dynamically; risky if statically linked |
| GPL-2.0, GPL-3.0 | ❌ Risky | Copyleft — may require open-sourcing your code |
| AGPL-3.0 | ❌ High Risk | Network copyleft — SaaS products must open source |
| CC-BY-NC, proprietary | ❌ Blocks commercial use | Requires legal review |
| No license | ❌ All rights reserved | Cannot legally use |

```bash
# npm license summary
npx license-checker --summary --production

# Python license check
pip-licenses --format markdown

# Rust
cargo license --json | jq 'group_by(.license) | .[] | {license: .[0].license, count: length}'
```

### 5. Duplicate packages (JS/TS)
```bash
# Find duplicate packages with different versions in node_modules
npm ls --all 2>/dev/null | grep -E "deduped|UNMET" | head -20

# Check bundle for duplicates (if webpack available)
npx webpack-bundle-analyzer stats.json 2>/dev/null
```

### 6. Unused dependencies
```bash
# Node.js
npx depcheck --json | jq '{unused: .dependencies, missing: .missing}'

# Python
pip-autoremove --list 2>/dev/null
```

## Risk matrix output format

```markdown
## Dependency Audit Summary — <date>

### 🔴 Critical (fix immediately)
| Package | Issue | Severity | Fix |
|---------|-------|----------|-----|
| lodash@4.17.20 | CVE-2021-23337 prototype pollution | High | Upgrade to 4.17.21 |

### 🟡 Medium (fix this sprint)
| Package | Issue | Action |
|---------|-------|--------|
| moment@2.29.4 | Abandoned (team recommends Day.js) | Replace |

### 🟢 Low (track and schedule)
| Package | Issue | Action |
|---------|-------|--------|
| uuid@7.0.3 | 2 major versions behind | Upgrade to v9 |
```

## Constraints
- Prioritize transitive (indirect) vulnerabilities too — they are equally exploitable
- A deprecated package that has no known CVE is still a risk: no future patches
- Never mark a GPL package as "low risk" in a commercial project without explicit legal sign-off
- Always separate `devDependencies` from production dependencies — dev-only vulnerabilities are lower urgency
