---
name: research-library-comparison
description: >
  Use this skill to compare libraries or frameworks that solve the same problem and recommend
  the best fit for the project's constraints. Trigger when the user asks "which library should
  I use for X?", "compare A vs B vs C", "what's better between X and Y?", "should we switch from
  X to Y?", or any evaluation of multiple options for a programming task. Also trigger when a
  user is undecided between packages and wants a structured trade-off analysis with a clear
  recommendation rather than a generic pros/cons list.
---

# Research: Library Comparison

## Purpose
Give the team a clear, evidence-backed recommendation for which library to use — not a wishy-washy "it depends". It always depends, but on *specific* things. Name those things, then decide.

## Comparison criteria (always evaluate all)

| Criterion | What to measure |
|-----------|----------------|
| **Functionality** | Feature coverage vs the project's actual requirements |
| **Performance** | Benchmarks relevant to the use case (throughput, memory, cold start) |
| **Bundle size** | For frontend: minified+gzip size via bundlephobia.com |
| **API / DX** | Ergonomics, TypeScript support, autocomplete quality |
| **Documentation** | Getting started quality, API reference completeness, examples |
| **Maintenance** | Release cadence, time-to-merge PRs, number of active maintainers |
| **Ecosystem** | Plugin availability, third-party integrations, community tooling |
| **Migration cost** | How hard is it to switch away from this later? |
| **License** | MIT/Apache-2.0 = safe; GPL/AGPL = legal review required; proprietary = vendor lock-in |

## Investigation steps

```bash
# Bundle size check (frontend libraries)
curl "https://bundlephobia.com/api/size?package=<lib>@latest" | jq '{size, gzip, dependencyCount}'

# Check npm weekly downloads for comparison
curl "https://api.npmjs.org/downloads/point/last-week/<lib-a>" | jq .downloads
curl "https://api.npmjs.org/downloads/point/last-week/<lib-b>" | jq .downloads

# Check license
cat node_modules/<lib>/LICENSE 2>/dev/null || cat node_modules/<lib>/LICENSE.md 2>/dev/null

# Count open issues and recent PRs
gh repo view <owner>/<repo> --json openIssues,pullRequests

# Inspect TypeScript support
ls node_modules/<lib>/index.d.ts 2>/dev/null && echo "Has types" || echo "No bundled types"
```

## Comparison matrix format

```markdown
| | Library A | Library B | Library C |
|--|-----------|-----------|-----------|
| Weekly downloads | 2.1M | 890K | 120K |
| Bundle (gzip) | 12kb | 8kb | 32kb |
| TypeScript | ✅ built-in | ⚠️ @types | ❌ |
| Last release | 3 days ago | 2 months ago | 11 months ago |
| License | MIT | MIT | GPL-3.0 |
| Open issues | 42 | 218 | 12 |
```

## Recommendation format

State the winner, then justify with **exactly three reasons** tied to the project's constraints. Then state the **runner-up** and when to choose it instead. Keep it opinionated.

## Constraints
- Never recommend based on stars alone — a popular abandoned library is worse than a maintained niche one
- Always check license compatibility with the project's own license
- Flag if one option has a recent security advisory (link to CVE)
- Benchmark numbers must reference their source URL and test date
