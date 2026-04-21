# Research: Markdown Format

## Purpose
Enforce a consistent, scannable, decision-ready format across all research reports. A good research report answers the key question in the first 30 seconds of reading (TL;DR), then provides depth for those who need it.

## Canonical report template

```markdown
# [Topic] Research
> **Date**: YYYY-MM-DD | **Researcher**: AI Agent | **Status**: Draft / Final

---

## TL;DR
One paragraph. State the conclusion first. What should the team do, and why?
No hedging. No "it depends" without immediately explaining what it depends on.

---

## Context
- **Why this research was triggered**: <user request or problem statement>
- **Current stack**: <relevant technologies already in use>
- **Key constraint**: <the single most important constraint driving the decision>
- **Decision needed by**: <urgency — blocks what?>

---

## Scope
**In scope**: What this research covers.
**Out of scope**: What this research explicitly does NOT cover (and why).

---

## Options Evaluated
1. **Option A** — <one-line description>
2. **Option B** — <one-line description>
3. **Option C** — <one-line description>
> Options NOT evaluated (and why): ...

---

## Comparison Matrix

| Criterion | Option A | Option B | Option C |
|-----------|----------|----------|----------|
| Performance | ✅ Fast | ⚠️ Medium | ❌ Slow |
| DX | ✅ | ✅ | ⚠️ |
| Maintenance | ✅ Active | ⚠️ Slow | ❌ Abandoned |
| License | MIT | MIT | GPL |
| Migration cost | Low | Medium | High |
| **Overall** | ⭐ Recommended | Runner-up | Not recommended |

---

## Deep Dive

### Option A — <Name>
**What it is**: ...
**Strengths**: ...
**Weaknesses**: ...
**Security**: ...
**Performance**: ...
**Community**: ...
**License**: ...

### Option B — <Name>
...

### Option C — <Name>
...

---

## Trade-off Analysis

| | Option A | Option B | Option C |
|--|----------|----------|----------|
| **Gains** | ... | ... | ... |
| **Loses** | ... | ... | ... |
| **Best for** | ... | ... | ... |
| **Avoid if** | ... | ... | ... |

---

## Recommendation

> **Adopt Option A.**

**Justification**:
1. <First reason — tied to a specific project constraint>
2. <Second reason — backed by a specific data point>
3. <Third reason — addresses the key risk>

**Runner-up**: Option B — choose this instead if <specific condition>.

**Do not use**: Option C — because <clear reason>.

---

## Migration / Adoption Path

> Estimated effort: X hours / X days
> Risk: Low / Medium / High

- [ ] Step 1: ...
- [ ] Step 2: ...
- [ ] Step 3: ...
- [ ] Step 4: Validate with <test or metric>
- [ ] Step 5: Remove old approach

**Rollback plan**: <How to revert if something goes wrong>

---

## Open Questions

- [ ] <Question that couldn't be answered from available sources>
- [ ] <Question requiring a sandbox test or vendor contact>
- [ ] <Question that depends on a future team decision>

---

## References

| Source | URL | Date accessed |
|--------|-----|---------------|
| Official docs | https://... | YYYY-MM-DD |
| CVE advisory | https://... | YYYY-MM-DD |
| Benchmark | https://... | YYYY-MM-DD |
| Engineering blog | https://... | YYYY-MM-DD |
```

---

## Writing style rules

### Do
- **State conclusions before evidence** — busy engineers read top-to-bottom, stop early
- **Use tables** for comparisons — never prose lists for multi-attribute comparisons
- **Use checkboxes** for action items — makes the report actionable
- **Quantify** — "3x faster" not "much faster"; "CVE-2024-1234 CVSS 9.8" not "critical vulnerability"
- **Flag opinion vs fact** — prefix opinions with `[Opinion]`, facts with a source reference
- **Use callout quotes** (`>`) for the final recommendation — it should stand out

### Don't
- Never write "it depends" as a conclusion — always resolve the dependency
- Never use more than 3 heading levels (`###`) — deeper nesting is unreadable
- Never leave the Comparison Matrix empty — a research report without a matrix is incomplete
- Never list references without dates — tech content ages, dates tell readers how stale a source is
- Never use passive voice in the Recommendation section — "we recommend" not "it is recommended"

## Badge conventions (optional, for visual scanning)

```markdown
✅ — meets requirement / no issue
⚠️ — partial / conditional / requires attention  
❌ — fails / risk / do not use
⭐ — recommended choice
🔴 — critical / block adoption
🟡 — medium risk / fix this sprint
🟢 — low risk / track and schedule
```

## Constraints
- The TL;DR must be the **first content section** — never bury the conclusion
- The Recommendation section must be **opinionated** — vague conclusions are not recommendations
- Every row in the References table must have a **date accessed** — no exceptions
- Research reports go in `researches/` folder as `[kebab-case-topic]-research.md`
