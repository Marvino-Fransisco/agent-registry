---
name: pe-zero-shot-builder
description: >
  Build effective zero-shot prompts from scratch — prompts with no examples,
  relying entirely on clear instruction and role framing. Use this skill whenever
  the user wants to write a prompt without examples, asks "how do I prompt for X
  without giving examples?", wants a minimal/lean prompt, or is prototyping a
  first-pass prompt for a new task. Also use when the user asks to convert an
  existing few-shot prompt into a zero-shot version.
---

# Zero-Shot Prompt Builder

Design high-quality zero-shot prompts — instructions-only, no examples.

## When Zero-Shot Works Best

- Task is well-defined and unambiguous by nature (translation, summarisation, classification with clear labels)
- Output format can be fully described in words
- The model has strong priors for the task domain
- Token budget is tight
- You're prototyping before deciding whether examples are needed

## When to Recommend Few-Shot Instead

Warn the user and suggest `pe-few-shot-builder` if:
- The task requires a specific style or voice that's hard to describe
- Output format is complex or nested
- The task is highly domain-specific or unusual
- Early testing shows high variance in outputs

## Building Blocks

A strong zero-shot prompt has these components in order:

```
[1] ROLE FRAME
[2] TASK STATEMENT
[3] CONSTRAINTS
[4] OUTPUT FORMAT
[5] INPUT PLACEHOLDER
```

### 1. Role Frame
Set the persona. Calibrate to the task:
- Simple task → minimal frame: `You are a helpful assistant.`
- Expert task → specific frame: `You are a senior tax attorney specialising in US corporate law.`
- Avoid vague frames like "You are an AI assistant" — they add no signal.

### 2. Task Statement
One clear imperative sentence. Use active voice.
- Bad: `The model should try to summarise the text in a way that captures the main points.`
- Good: `Summarise the following text in 3 bullet points, each under 20 words.`

### 3. Constraints
List explicit boundaries. Cover:
- Length (word/sentence/character count)
- Tone (formal, casual, technical)
- Scope (what to include / exclude)
- Language (if not obvious)
- Persona of the reader (who will read the output)

### 4. Output Format
Be explicit. Options:
- Plain prose
- Bullet list
- Numbered steps
- JSON schema (paste the schema)
- Markdown with specific headers
- Table with named columns

### 5. Input Placeholder
Use a clear delimiter: `<text>`, `---`, triple backticks, or XML tags.
Prefer XML tags for inputs that may themselves contain markdown.

## Process

1. Ask the user: **what task** + **what output** + **who reads it** + **any constraints**.
2. Draft the prompt using the building blocks above.
3. Annotate each block inline with a comment (e.g. `# ROLE FRAME`) so the user understands the structure.
4. Offer a clean version (no comments) after the annotated version.
5. Suggest 2–3 test inputs the user should try to validate the prompt.

## Output Template

```
## Zero-Shot Prompt

### Annotated Version
[prompt with inline # comments per block]

### Clean Version
[prompt without comments]

### Suggested Test Inputs
1. [test input — happy path]
2. [test input — edge case]
3. [test input — tricky/ambiguous]
```
