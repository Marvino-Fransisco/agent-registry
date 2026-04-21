---
name: pe-prompt-diff
description: >
  Show a clear before/after diff of prompt changes and explain the rationale
  for each change. Use this skill whenever the user wants to compare two versions
  of a prompt, asks "what changed?", wants to review a prompt revision, needs to
  document prompt changes for a changelog, or is about to apply changes to a
  prompt and wants explicit approval. Also use proactively before making any
  non-trivial edit to an existing prompt.
---

# Prompt Diff

Show structured before/after comparisons of prompt changes with rationale.

## When to Produce a Diff

Always produce a diff before applying changes when:
- Modifying an existing system prompt
- Changing few-shot examples
- Altering output format instructions
- Adjusting temperature, model, or sampling settings
- Rewriting any instruction that changes model behaviour

Never silently apply changes. Show the diff first, wait for approval.

## Diff Format

### Inline Diff (for short prompts or small changes)

Use `- ` for removed lines, `+ ` for added lines, ` ` (space) for unchanged context:

```diff
  You are a helpful assistant.

- Summarise the following text in 5 bullet points.
+ Summarise the following text in exactly 5 bullet points, each under 20 words.

  Text: {{text}}

- Return your answer as plain text.
+ Return your answer as a markdown bullet list.
```

### Block Diff (for large or structural changes)

Show old and new blocks side-by-side with a change summary:

```
### Change 1 — Task instruction [lines 3–4]

**Before:**
Summarise the text in bullet points.

**After:**
Summarise the following text in exactly 5 bullet points, each under 20 words.
Return your answer as a markdown bullet list.

**Rationale:** Added explicit count (5) and length constraint (20 words) to
reduce variance. Added format spec (markdown list) to anchor the output shape.

**Risk:** Low — additive constraint, no intent change.
```

### Settings Diff (for model/parameter changes)

```
### Settings Changes

| Setting | Before | After | Rationale |
|---------|--------|-------|-----------|
| model | claude-haiku-4-5 | claude-sonnet-4-6 | Need better reasoning for complex classification |
| temperature | 1.0 | 0.3 | Reduce variance — output should be deterministic |
| max_tokens | 256 | 512 | Previous limit was truncating long outputs |
```

## Change Classification

Label every change with one of:

| Label | Meaning |
|-------|---------|
| `additive` | Adds constraint or info — low risk of regression |
| `restrictive` | Narrows scope or removes permission — may break edge cases |
| `format` | Changes output format only — pipeline impact possible |
| `breaking` | Changes intent, persona, or task — high regression risk |
| `fix` | Corrects a bug in the prompt — expected to improve, may affect edge cases |

## Approval Gate

After every diff, end with one of:

**For low-risk changes:**
> These are additive changes. Shall I apply them?

**For format changes:**
> This changes the output format. Check whether your downstream parser handles the new format before I apply this.

**For breaking changes:**
> ⚠️ This is a breaking change — it alters the core task/intent. Run your full eval suite before deploying. Shall I apply this change?

## Changelog Entry

After approval and application, produce a changelog entry:

```markdown
## Changelog

### v1.1.0 — [YYYY-MM-DD]

**Changes:**
- [additive] Added word count constraint to bullet points (≤ 20 words each)
- [format] Output changed from plain text to markdown bullet list

**Rationale:** Reduce output variance and improve downstream parsing reliability.

**Risk:** Low. No intent change. Downstream parsers should handle markdown list.

**Tested on:** [N test cases — pass rate X%]
```

## Process

1. Receive the original prompt and the proposed changes (or generate changes from a review).
2. Choose diff format: inline (small changes), block (structural), settings (parameter changes).
3. Label each change with its classification.
4. Write the rationale for each change (one sentence minimum).
5. Present the diff and wait for explicit approval.
6. After approval, produce the changelog entry.

## Output Template

```
## Prompt Diff

### Summary
[N changes: X additive, Y format, Z breaking]

### Diff
[inline or block diff]

### Settings Changes (if any)
[settings table]

### Approval
[approval gate message appropriate to the highest-risk change]
```
