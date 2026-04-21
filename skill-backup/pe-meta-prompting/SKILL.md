---
name: pe-meta-prompting
description: >
  Use a model to generate, improve, or optimise prompts — prompting about prompts.
  Use this skill whenever the user wants Claude to write a prompt for them from
  a high-level goal, asks "generate a prompt that does X", wants to automate
  prompt drafting, build a prompt generator, or use an LLM to iterate on prompt
  quality. Also use when the user wants to build a system where one model writes
  prompts for another model.
---

# Meta-Prompting

Use a model to generate or improve prompts. The model writes the prompt; you validate and refine it.

## What is Meta-Prompting?

Meta-prompting is when you instruct a model to produce a prompt (rather than a final answer). The output of the meta-prompt is itself a prompt, which is then used to drive another model call.

```
User Goal
   ↓
[Meta-Prompt] → Model → Draft Prompt
                              ↓
                       [Draft Prompt] → Model → Final Output
```

## When to Use Meta-Prompting

- You have a high-level goal but don't know how to phrase the prompt
- You want to rapidly generate prompt variants for A/B testing
- You're building a pipeline where prompts must be generated dynamically per user input
- You want a model to specialise a generic prompt template for a specific domain
- You need a prompt in a language or format you're not familiar with

## Meta-Prompt Design Patterns

### Pattern 1 — Goal-to-Prompt
Give the model a goal; it returns a ready-to-use prompt.

```
You are an expert prompt engineer.

Given the following task goal, write a high-quality system prompt that
a language model can use to accomplish the task reliably.

The prompt must include:
- A clear role frame
- An explicit task statement
- Output format specification
- At least one edge case instruction

Task goal: {{task_goal}}

Return only the prompt. No explanation, no preamble.
```

### Pattern 2 — Prompt Improver
Give the model a draft prompt; it returns an improved version.

```
You are an expert prompt engineer. Your job is to improve the following
prompt without changing its intent.

Improvements to make:
- Increase clarity and remove ambiguity
- Add missing constraints
- Improve output format specification
- Remove redundant phrasing

Return only the improved prompt. No explanation.

Original prompt:
<prompt>
{{original_prompt}}
</prompt>
```

### Pattern 3 — Prompt Variants Generator
Generate N variants for A/B testing.

```
You are an expert prompt engineer.

Generate {{n}} distinct variants of the following prompt for A/B testing.
Each variant should differ in: [tone / persona / instruction style / format spec].

Label each variant VARIANT_1, VARIANT_2, etc.

Base prompt:
<prompt>
{{base_prompt}}
</prompt>
```

### Pattern 4 — Domain Specialiser
Take a generic prompt and specialise it for a domain.

```
You are an expert prompt engineer specialising in {{domain}}.

Adapt the following general-purpose prompt to work specifically for
{{domain}} use cases. Preserve the structure but replace generic language
with domain-specific terminology, constraints, and examples.

General prompt:
<prompt>
{{general_prompt}}
</prompt>
```

## Validation Step (always required)

A meta-prompt output is a **draft**, not a final prompt. Always:

1. Run the generated prompt on 2–3 test inputs.
2. Check with `pe-prompt-review` for quality issues.
3. Iterate: if output is poor, revise the meta-prompt, not just the generated prompt.

## Process

1. Clarify the user's goal: are they generating from scratch, improving an existing prompt, or building a dynamic pipeline?
2. Select the appropriate pattern above.
3. Fill in the meta-prompt template with the user's context.
4. Run it (or ask the user to run it) and capture the output.
5. Review the generated prompt with `pe-prompt-review`.
6. Iterate until the generated prompt scores ≥ 4/5.

## Output Template

```
## Meta-Prompt

### Pattern Used
[Goal-to-Prompt / Prompt Improver / Variants / Domain Specialiser]

### Meta-Prompt
[full meta-prompt to run]

### Generated Prompt (after running)
[output of the meta-prompt]

### Review Notes
[quick quality check — what works, what needs refinement]

### Next Step
[run test inputs / apply pe-prompt-review / iterate]
```
