---
name: se-markdown-format
description: Format software engineering outputs (implementation notes, technical decisions, code explanations) in clean, consistent markdown. Use this skill whenever the software engineer agent produces written output that will be read by other developers — such as implementation summaries, ADRs, inline documentation, or PR descriptions. Trigger on any "document this", "write up", "PR description", "ADR", or "explain this decision" request.
---

# Markdown Format

Produce developer-facing documentation that is clear, scannable, and consistent.

## Document Types

### Implementation Summary
Used after completing a feature. Captures what was built and why.

```markdown
# [Feature Name] — Implementation Summary

## What Was Built
[2-3 sentences describing what the feature does]

## Key Decisions
- **[Decision]**: [Why this approach was chosen over alternatives]
- **[Decision]**: [Why]

## Files Changed
| File | Change |
|---|---|
| `src/services/user.service.ts` | Added createUser method |
| `src/repos/user.repo.ts` | Added save() |

## Testing
- Unit: [what's covered]
- Integration: [what's covered]
- Edge cases: [what's covered]

## Known Limitations / Follow-ups
- [ ] [thing to address later]
```

### Architecture Decision Record (ADR)
Used for significant technical decisions.

```markdown
# ADR-[number]: [Short title]

**Status**: Accepted / Proposed / Deprecated
**Date**: YYYY-MM-DD

## Context
[Why does this decision need to be made?]

## Decision
[What was decided?]

## Consequences
**Positive**:
- [benefit]

**Negative**:
- [tradeoff]

## Alternatives Considered
- **[Alternative]**: Rejected because [reason]
```

### PR Description
```markdown
## Summary
[What does this PR do in 1-2 sentences?]

## Changes
- [change 1]
- [change 2]

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manually tested: [scenarios]

## Notes for Reviewer
[Anything to pay special attention to]
```

## Formatting Rules

- **Headers**: Use `##` for sections, `###` for subsections — never skip levels
- **Code**: Always use fenced code blocks with language tag (` ```typescript `, ` ```sql `)
- **Lists**: Use `-` for unordered, `1.` for ordered steps
- **Tables**: Use for comparisons and file change lists
- **Bold**: For key terms and decisions — not for general emphasis
- **Line length**: Keep prose lines under 100 characters
- **Tone**: Direct, factual, no filler phrases ("essentially", "basically", "it's worth noting")
