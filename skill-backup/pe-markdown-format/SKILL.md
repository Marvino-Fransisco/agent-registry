---
name: pe-markdown-format
description: >
  Format, structure, and polish prompts or prompt documentation using clean
  markdown conventions. Use this skill whenever the user wants to format a prompt
  as a markdown document, convert a raw prompt into a well-structured .md file,
  write a prompt spec or prompt library entry, or produce readable documentation
  for a prompt. Also use when the user asks to "clean up" or "document" a prompt.
---

# Prompt Markdown Format

Produce clean, readable markdown documents for prompts, prompt specs, and prompt libraries.

## When to Use Markdown Formatting

- Storing prompts in a repository or docs system
- Writing prompt specs for team review
- Creating a prompt library entry
- Documenting a prompt for handoff to engineers or stakeholders
- Producing a changelog entry for a revised prompt

## Prompt Document Structure

Use this standard structure for a standalone prompt markdown file:

```markdown
---
name: [short-slug]
version: [semver e.g. 1.0.0]
model: [target model or "model-agnostic"]
temperature: [float]
tags: [comma-separated: classification, summarisation, etc.]
author: [name or team]
updated: [YYYY-MM-DD]
---

# [Prompt Name]

> [One-sentence description of what this prompt does and when to use it]

## Task

[Clear description of the task the prompt performs]

## System Prompt

```
[full system prompt text]
```

## User Message Template

```
[user message template with {{variables}}]
```

## Variables

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `var1` | string | yes | ... |

## Examples

### Example 1 — [label e.g. happy path]
**Input:**
```
[sample input]
```
**Output:**
```
[expected output]
```

### Example 2 — [label e.g. edge case]
...

## Eval Criteria

- [ ] [Criterion 1 — e.g. "Output is valid JSON"]
- [ ] [Criterion 2 — e.g. "Summary is under 150 words"]
- [ ] [Criterion 3 — e.g. "No hallucinated citations"]

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | YYYY-MM-DD | Initial version |
```

## Markdown Conventions

- Use `#` for prompt name, `##` for major sections — no deeper than `###`
- Wrap all prompt text in fenced code blocks (triple backticks) — preserves whitespace
- Use `{{variable}}` syntax for placeholders inside code blocks
- Use a YAML front-matter block for metadata
- Tables for variables, changelog, and eval criteria
- Blockquote (`>`) for one-liner description at the top
- Checkboxes (`- [ ]`) for eval criteria — easy to tick during review

## Versioning Convention

Follow semver semantics adapted for prompts:
- **Patch** (1.0.x): Typo fix, minor wording tweak, no behaviour change
- **Minor** (1.x.0): New variable, new example, refined instructions — backward compatible
- **Major** (x.0.0): Changed task, new output format, persona overhaul — breaking change

## Process

1. Take the user's raw prompt or prompt details.
2. Ask for metadata if missing (name, model, author, date).
3. Assemble the full markdown document using the structure above.
4. Fill in the eval criteria based on the task.
5. Add a changelog row for the current version.
6. Present the finished `.md` file.

## Output

Deliver the result as a single fenced markdown code block the user can copy directly into a `.md` file, followed by any notes on missing information they should fill in.
