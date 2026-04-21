---
name: pe-chain-of-thought
description: >
  Design prompts that elicit step-by-step reasoning from a model using
  chain-of-thought (CoT) techniques. Use this skill whenever the user wants the
  model to reason through a problem, asks about CoT prompting, wants to reduce
  reasoning errors, needs the model to "show its work", or is working on tasks
  involving maths, logic, multi-step decisions, or complex classification. Also
  use when model outputs are correct on easy cases but wrong on hard ones.
---

# Chain-of-Thought Prompting

Design prompts that elicit explicit step-by-step reasoning before the model produces a final answer.

## Why CoT Works

Models perform better on complex tasks when they reason through intermediate steps. CoT trades tokens for accuracy — the model "thinks aloud" before committing to an answer.

## CoT Variants

### 1. Zero-Shot CoT
Add a reasoning trigger phrase. No examples needed.

```
[task instruction]

Think step by step before giving your final answer.
```

Common trigger phrases (pick one):
- `Think step by step.`
- `Let's work through this carefully.`
- `Reason through this before answering.`
- `Break this down into steps.`

Best for: general reasoning, maths, logic puzzles, when you have no examples.

---

### 2. Few-Shot CoT
Provide examples that include explicit reasoning chains.

```
Example:
Q: [question]
A: Let me think through this step by step.
   Step 1: [reasoning]
   Step 2: [reasoning]
   Step 3: [reasoning]
   Final answer: [answer]

Q: {{question}}
A:
```

Best for: domain-specific reasoning, custom rubrics, nuanced classification.

---

### 3. Structured CoT (Scratchpad)
Force reasoning into a labelled scratchpad before the answer.

```
For each question, first fill in the <scratchpad> with your reasoning,
then give your final answer in <answer>.

<scratchpad>
[your step-by-step reasoning here]
</scratchpad>

<answer>
[final answer only]
</answer>
```

Best for: when you want to parse reasoning and answer separately in a pipeline.

---

### 4. Self-Ask CoT
Model decomposes the question into sub-questions and answers each.

```
Break this problem into sub-questions. Answer each sub-question,
then use those answers to answer the original question.

Format:
Sub-question 1: [question]
Answer: [answer]

Sub-question 2: [question]
Answer: [answer]

...

Final answer: [synthesised answer]
```

Best for: complex multi-hop questions, research tasks, comparative analysis.

---

### 5. ReAct (Reason + Act)
Interleave reasoning with tool use or action steps.

```
For each step, output:
Thought: [your reasoning about what to do next]
Action: [the action to take, e.g. search, calculate, look up]
Observation: [what you found]

Repeat until you have enough information, then output:
Final Answer: [answer]
```

Best for: agentic tasks, tool-using pipelines, retrieval-augmented generation.

---

## Answer Extraction

When using CoT, the final answer may be buried in reasoning. Use one of these patterns to extract it cleanly:

- XML tags: `<answer>...</answer>` — parse with regex or tag parser
- Delimiter: `Final answer:` — split on this string
- JSON suffix: end CoT with `{"answer": "..."}` — parse as JSON

## When NOT to Use CoT

- Very simple tasks (CoT adds tokens without benefit)
- Latency-critical production pipelines (CoT is slower)
- Output must be a single token (yes/no, label) — CoT before the label is fine; CoT after is wasteful

## Process

1. Ask: what task? what's failing — wrong answers, inconsistency, or opaque reasoning?
2. Choose the appropriate CoT variant.
3. Draft the prompt with CoT trigger + answer extraction pattern.
4. If using few-shot CoT, draft 2–3 examples with explicit reasoning chains.
5. Specify how the final answer will be parsed in the pipeline.
6. Suggest test cases that range from easy to hard to validate the CoT benefit.

## Output Template

```
## Chain-of-Thought Prompt

### Variant
[Zero-Shot / Few-Shot / Scratchpad / Self-Ask / ReAct]

### Prompt
[full prompt with CoT trigger and answer extraction]

### Examples (if few-shot)
[2–3 examples with reasoning chains]

### Answer Extraction
[how to parse the final answer from the output]

### Test Cases
[3 cases: easy / medium / hard — to validate CoT benefit]
```
