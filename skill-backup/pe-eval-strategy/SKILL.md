---
name: pe-eval-strategy
description: >
  Design an evaluation strategy for a prompt — including test cases, rubrics,
  metrics, and automated or human scoring. Use this skill whenever the user wants
  to measure prompt quality, asks "how do I know if my prompt is good?", needs
  to build a regression test suite for a prompt, wants to compare two prompt
  variants, or is preparing a prompt for production and needs quality gates.
  Also use when the user is iterating on a prompt and wants objective feedback
  rather than gut feel.
---

# Prompt Evaluation Strategy

Design a rigorous eval framework to measure and improve prompt quality.

## Eval Types

| Type | When to use |
|------|-------------|
| **Exact match** | Output must equal a known string (classification labels, yes/no) |
| **Contains check** | Output must include a required phrase or field |
| **Schema validation** | Output must be valid JSON / XML matching a schema |
| **Rubric scoring** | Output is graded 1–5 on named dimensions (LLM-as-judge or human) |
| **Reference comparison** | Output is compared to a gold reference (BLEU, ROUGE, semantic similarity) |
| **Regression** | New prompt must not regress on cases where old prompt was correct |

## Test Case Design

### Minimum test set for a new prompt
- 5 happy-path cases (typical, well-formed inputs)
- 3 edge cases (boundary inputs, unusual formats)
- 2 adversarial cases (inputs designed to break the prompt)
- 1 empty/null case (empty input, missing fields)

### Test case format

```json
{
  "id": "tc-001",
  "label": "happy path — short document",
  "input": "{{document}}: The agreement was signed on 12 March 2024.",
  "expected_output": "2024-03-12",
  "assertions": [
    {"type": "exact_match", "value": "2024-03-12"},
    {"type": "not_contains", "value": "I don't know"}
  ],
  "tags": ["happy_path", "date_extraction"]
}
```

## Rubric Design (for open-ended outputs)

When exact match is impossible, grade on named dimensions:

```
## Scoring Rubric

| Dimension | Weight | 1 (poor) | 3 (acceptable) | 5 (excellent) |
|-----------|--------|----------|----------------|---------------|
| Accuracy | 40% | Major factual errors | Minor errors | Fully correct |
| Completeness | 30% | Missing key info | Mostly complete | Covers all required points |
| Format | 20% | Wrong format | Mostly correct format | Perfectly formatted |
| Conciseness | 10% | Excessive fluff | Slightly verbose | Tight and precise |

Overall score = weighted average
Pass threshold: ≥ 3.5
```

## LLM-as-Judge Prompt

Use a second model call to score outputs automatically:

```
You are an expert evaluator. Score the following model output on this task.

Task: {{task_description}}

Output to evaluate:
<output>
{{model_output}}
</output>

Score on each dimension (1–5):
- Accuracy: [score] — [one-sentence rationale]
- Completeness: [score] — [one-sentence rationale]
- Format: [score] — [one-sentence rationale]
- Conciseness: [score] — [one-sentence rationale]

Overall: [weighted score]
Pass: [yes/no] (threshold: 3.5)
```

## Metrics Summary

| Metric | What it measures | Formula |
|--------|-----------------|---------|
| Pass rate | % of test cases passing all assertions | passing / total |
| Mean rubric score | Average quality across cases | sum(scores) / N |
| Regression rate | % of previously-passing cases now failing | new_failures / old_passes |
| Consistency (std dev) | Variance across N runs of same input | std(scores per input) |

## Eval Cadence

| Stage | Eval type | Frequency |
|-------|-----------|-----------|
| Prompt draft | Manual spot-check (5–10 cases) | Every draft |
| Prompt candidate | Full test suite (automated) | Before each deployment |
| Production | Regression suite + sampling | On every change |
| A/B test | Side-by-side rubric comparison | When comparing variants |

## Process

1. Understand the prompt's task and output type.
2. Determine the appropriate eval type(s) from the table above.
3. Design the test suite: happy path + edge + adversarial + null.
4. If open-ended output: design the rubric.
5. Write the LLM-as-judge prompt (if automated scoring).
6. Define the pass threshold and regression policy.
7. Deliver the test cases in JSON and the rubric in markdown.

## Output Template

```
## Eval Strategy

### Eval Type(s)
[exact match / rubric / schema / regression / ...]

### Test Cases
[JSON array of test cases]

### Rubric (if applicable)
[dimension table with weights and score anchors]

### LLM-as-Judge Prompt (if applicable)
[judge prompt]

### Metrics & Thresholds
- Pass rate target: ≥ [X]%
- Mean rubric score target: ≥ [X]
- Regression tolerance: 0 regressions on high-priority cases

### Cadence
[when to run which evals]
```
