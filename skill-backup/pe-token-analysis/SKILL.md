---
name: pe-token-analysis
description: >
  Analyse a prompt's token usage, identify waste, and compress it without losing
  quality. Use this skill whenever the user wants to reduce prompt length, asks
  about token costs, is hitting context window limits, wants to optimise a prompt
  for latency or cost, or asks "how many tokens does my prompt use?". Also use
  when a prompt is verbose and the user wants a leaner version.
---

# Token Analysis

Measure, audit, and reduce prompt token consumption without sacrificing quality.

## Token Estimation (without running an API call)

Rough heuristics (English text, standard tokeniser):
- 1 token ≈ 4 characters ≈ 0.75 words
- 100 words ≈ 133 tokens
- 1 page of prose ≈ 400–600 tokens
- Code/JSON: more tokens per word (punctuation tokenises separately)
- Repeated words/phrases: tokeniser does not deduplicate

To count precisely, use the model's tokeniser:
- Claude: `anthropic.count_tokens(text)` (Python SDK)
- OpenAI-compatible: `tiktoken.encoding_for_model("gpt-4").encode(text)`

## Token Audit

For a given prompt, measure and categorise token spend:

| Block | Token count | % of total | Notes |
|-------|-------------|------------|-------|
| Role frame | N | X% | |
| Task instruction | N | X% | |
| Constraints | N | X% | |
| Output format | N | X% | |
| Few-shot examples | N | X% | Often the largest block |
| Input placeholder | N | X% | Variable — not in static count |
| **Total (static)** | **N** | **100%** | |

Flag any block consuming > 30% of the static token budget for compression review.

## Common Token Waste Patterns

| Pattern | Example | Fix |
|---------|---------|-----|
| Restating the same rule twice | "Be concise. Keep responses short and to the point." | Remove duplicate |
| Hedging phrases | "Please try to", "You should attempt to" | Use direct imperatives |
| Unnecessary preamble | "Your task is to help the user with the following..." | Remove — just state the task |
| Verbose examples | Examples with long explanatory prose | Trim to the minimum that demonstrates the pattern |
| Over-specified format | Describing every field in a JSON schema in prose | Use the schema itself, not prose |
| Politeness filler | "Thank you for providing this information." | Remove entirely |
| Negative + positive restatement | "Don't be verbose. Be concise." | Pick one formulation |

## Compression Techniques

### 1. Remove preamble
```
# Before (12 tokens)
Your task is to summarise the provided text.

# After (5 tokens)
Summarise the provided text.
```

### 2. Merge related constraints
```
# Before (18 tokens)
Keep your response short. Use a maximum of 100 words.
Do not use bullet points. Write in plain prose.

# After (11 tokens)
Respond in plain prose, ≤ 100 words.
```

### 3. Use structured format for rules
```
# Before (prose rules — 40 tokens)
Make sure you do not include any personal opinions. Always cite your sources.
Never make up information. Keep the tone professional and neutral.

# After (rules list — 22 tokens)
Rules:
- No personal opinions
- Cite sources for every claim
- Professional, neutral tone
- No fabrication
```

### 4. Compress few-shot examples
Keep examples minimal — demonstrate format, not explain it.
Remove all prose explanation from inside examples.

### 5. Externalise large context
If a reference document is always the same, store it separately and load via retrieval. Don't bake it into the static prompt.

## Token Budget Planning

For a target context window, allocate tokens across:

| Component | Recommended allocation |
|-----------|----------------------|
| System prompt (static) | 10–20% of context window |
| Dynamic context / documents | 40–60% |
| Conversation history | 10–20% |
| Output (max_tokens) | 10–20% |

Example: 100k context window → ~15k system, ~50k context, ~15k history, ~20k output.

## Process

1. Take the user's prompt.
2. Estimate token count per block using the heuristics above.
3. Run the waste audit — flag patterns from the table.
4. Apply compression techniques in order of impact (highest token saving first).
5. Compare before/after token counts.
6. Produce the compressed prompt and a savings summary.
7. Use `pe-prompt-review` to confirm compression didn't degrade quality.

## Output Template

```
## Token Analysis

### Token Count by Block
[audit table]

### Total: [N] tokens (before) → [N] tokens (after) — [X]% reduction

### Waste Found
1. [Pattern]: [location in prompt] — saves ~N tokens
2. ...

### Compressed Prompt
[optimised prompt]

### Quality Check
[confirm no instruction was dropped, only phrasing compressed]
```
