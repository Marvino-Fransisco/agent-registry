---
name: pe-prompt-review
description: >
  Review, audit, and critique any prompt or system prompt for quality issues.
  Use this skill whenever the user asks to review, audit, check, critique, analyse,
  or grade a prompt — even if they just say "what's wrong with this prompt?" or
  "is this prompt good?". Also triggers when the user pastes a prompt and asks
  for feedback, improvements, or a second opinion.
---

# Prompt Review

Conduct a structured audit of a prompt and return actionable, prioritised feedback.

## Review Dimensions

Evaluate the prompt across these 8 dimensions. Score each 1–5 (5 = excellent).

| # | Dimension | What to look for |
|---|-----------|-----------------|
| 1 | **Clarity** | Is the task unambiguous? Could a different model interpret it differently? |
| 2 | **Persona / Role** | Is there a role frame? Is it calibrated to the task complexity? |
| 3 | **Constraints** | Are format, length, tone, and scope explicitly stated? |
| 4 | **Output Format** | Is the expected output schema or structure defined? |
| 5 | **Few-Shot Examples** | Are examples present when needed? Do they cover edge cases? |
| 6 | **Edge Cases** | Does the prompt handle nulls, off-topic inputs, refusals, ambiguity? |
| 7 | **Token Efficiency** | Any redundant phrasing, filler, or over-explanation? |
| 8 | **Safety** | Any vectors for hallucination, jailbreak, or harmful output? |

## Output Format

Return the review in this structure:

```
## Prompt Review

### Scores
| Dimension       | Score | Note |
|-----------------|-------|------|
| Clarity         | X/5   | ...  |
| Persona         | X/5   | ...  |
| ...             | ...   | ...  |
**Overall: X/5**

### Critical Issues (must fix)
- [Issue]: [why it matters] → [fix]

### Suggestions (nice to have)
- [Suggestion]: [rationale]

### Revised Prompt (optional, if issues are significant)
[rewritten version]
```

## Process

1. Read the full prompt carefully — do not skim.
2. Score each dimension independently before writing notes.
3. Identify the **top 3 critical issues** (score ≤ 2 in any dimension, or systemic problems).
4. List suggestions separately from critical issues.
5. If overall score < 3, offer a revised prompt.
6. If overall score ≥ 4, affirm what works before listing suggestions.

## Principles

- Be specific: cite the exact phrase or section causing the issue.
- Be constructive: every criticism must include a fix or direction.
- Prioritise: don't list 10 equal-weight items — rank by impact.
- Stay model-agnostic unless the user specifies a target model.
