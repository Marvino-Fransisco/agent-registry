---
name: pe-few-shot-builder
description: >
  Build few-shot prompts by designing high-quality input/output examples that
  teach the model by demonstration. Use this skill whenever the user wants to
  add examples to a prompt, asks "how many examples do I need?", wants to improve
  output consistency through demonstrations, or asks to convert a zero-shot prompt
  to few-shot. Also use when the user has bad or inconsistent model outputs and
  examples might fix it.
---

# Few-Shot Prompt Builder

Design and curate input/output examples that reliably steer model behaviour.

## When Few-Shot Wins Over Zero-Shot

- Output has a specific style, voice, or format that's easier to show than describe
- Task is domain-specific and the model's priors are weak
- Zero-shot produces high variance or inconsistent formatting
- Classification labels have nuanced boundaries
- The task involves a custom scoring rubric or persona

## How Many Examples?

| Situation | Recommended count |
|-----------|------------------|
| Format/style anchoring | 1–2 |
| Classification (clear labels) | 2–3 (one per class minimum) |
| Classification (nuanced labels) | 3–5 |
| Complex structured output | 3–5 |
| Highly variable task | 5–10 |

Prefer fewer, higher-quality examples over many mediocre ones.

## Example Quality Checklist

Each example must be:
- [ ] **Representative** — reflects real inputs the model will see
- [ ] **Correct** — the output is genuinely the right answer
- [ ] **Diverse** — covers different sub-cases, lengths, tones
- [ ] **Balanced** — for classification, cover all label classes
- [ ] **Edge-inclusive** — at least one example near a decision boundary

## Negative Examples

Include at least one negative example when:
- There's a common failure mode you want to prevent
- The model might over-generate (e.g. add disclaimers you don't want)
- The boundary between two classes is ambiguous

Format negative examples explicitly:
```
Input: [input]
Output: [what NOT to do]  ← label this clearly
Correct Output: [what to do instead]
```

## Example Format Patterns

### Pattern A — Labelled blocks (recommended for most tasks)
```
Example 1:
Input: [input]
Output: [output]

Example 2:
Input: [input]
Output: [output]
```

### Pattern B — XML tags (recommended when inputs contain markdown or code)
```xml
<example>
  <input>[input]</input>
  <output>[output]</output>
</example>
```

### Pattern C — Q&A style (for conversational tasks)
```
User: [input]
Assistant: [output]
```

## Process

1. Ask the user: task description, target output format, real sample inputs (if available).
2. Determine how many examples are needed using the table above.
3. Draft examples — diverse, representative, edge-inclusive.
4. Add at least one negative example if a common failure mode is apparent.
5. Integrate examples into a full prompt using `pe-zero-shot-builder` building blocks + examples block.
6. Show the user the full assembled prompt.
7. Suggest which test inputs to run to validate example coverage.

## Output Template

```
## Few-Shot Prompt

### System / Instruction Block
[role frame + task statement + constraints + output format]

### Examples
[2–5 examples in chosen format]

### Input Placeholder
[delimiter for live input]

### Coverage Notes
- Happy path: covered by Example [N]
- Edge case [X]: covered by Example [N]
- Negative case: [present / not present — reason]
```
