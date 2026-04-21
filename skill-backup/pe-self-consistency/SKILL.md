---
name: pe-self-consistency
description: >
  Improve answer reliability by sampling multiple reasoning paths and aggregating
  the most consistent answer. Use this skill whenever the user wants to reduce
  model variance, asks about self-consistency decoding, wants majority-vote
  aggregation across multiple model calls, needs higher accuracy on hard reasoning
  tasks, or is debugging a prompt where the model sometimes gets the right answer
  but not always. Also use when a single model call is unreliable but the user
  can afford multiple calls.
---

# Self-Consistency Prompting

Sample multiple independent reasoning chains and aggregate the most consistent final answer.

## What is Self-Consistency?

Instead of relying on a single model call, you run the same prompt N times (with temperature > 0), collect all final answers, and return the most frequent one (majority vote).

```
Prompt → Run × N (temp > 0)
  ↓         ↓         ↓
Answer_1  Answer_2  Answer_3  ...
         ↓
   [Majority Vote]
         ↓
   Final Answer
```

Self-consistency consistently outperforms single-pass CoT on arithmetic, commonsense, and symbolic reasoning tasks.

## When to Use Self-Consistency

| Use | Avoid |
|-----|-------|
| Hard reasoning (maths, logic, multi-hop) | Simple factual lookup |
| High-stakes decisions needing reliability | Latency-critical paths |
| Model is right ~50–70% of the time | Model is already >90% accurate |
| You can afford N × cost per call | Token budget is tight |

## Recommended Settings

| Parameter | Value |
|-----------|-------|
| Temperature | 0.5–1.0 (must be > 0 for diversity) |
| N samples | 5–10 (diminishing returns after ~20) |
| Aggregation | Majority vote on extracted final answer |

## Prompt Design for Self-Consistency

The underlying prompt should use CoT (see `pe-chain-of-thought`) so each sample produces a distinct reasoning path. The key is **answer extraction** — you need to reliably parse the final answer from each sample.

### Recommended answer extraction patterns

```
# Option A — XML tag
Return your final answer inside <answer> tags.
<answer>[answer]</answer>

# Option B — Explicit label
End with: Final answer: [answer]

# Option C — JSON suffix
After your reasoning, output: {"answer": "..."}
```

## Aggregation Strategies

### Majority Vote (default)
Count the most frequent extracted answer. Break ties randomly or flag for human review.

```python
from collections import Counter

answers = ["A", "B", "A", "A", "C"]
winner = Counter(answers).most_common(1)[0][0]
# → "A"
```

### Weighted Vote
Weight by reasoning length or confidence signals (longer, more detailed reasoning = higher weight). Use only if you have a calibrated confidence signal.

### Threshold Vote
Only return an answer if it appears in ≥ K/N samples. Otherwise, flag as uncertain.

```python
threshold = 0.6  # 60% agreement required
votes = Counter(answers)
top_answer, top_count = votes.most_common(1)[0]
if top_count / len(answers) >= threshold:
    return top_answer
else:
    return "uncertain"
```

## Implementation Sketch

```python
import anthropic
from collections import Counter

def self_consistency(prompt: str, n: int = 7, temperature: float = 0.7) -> str:
    client = anthropic.Anthropic()
    answers = []
    for _ in range(n):
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            temperature=temperature,
            messages=[{"role": "user", "content": prompt}]
        )
        raw = response.content[0].text
        answer = extract_answer(raw)   # implement per your extraction pattern
        answers.append(answer)
    return Counter(answers).most_common(1)[0][0]
```

## Process

1. Confirm the user has a CoT prompt (or build one with `pe-chain-of-thought`).
2. Add a clear answer extraction pattern to the prompt.
3. Choose N and temperature based on accuracy vs. cost tradeoff.
4. Choose aggregation strategy (default: majority vote).
5. Provide the implementation sketch above adapted to the user's stack.
6. Suggest how to handle ties and low-confidence cases.

## Output Template

```
## Self-Consistency Setup

### Base Prompt (with CoT + answer extraction)
[prompt]

### Sampling Config
- N: [number of samples]
- Temperature: [value]
- Aggregation: [majority vote / weighted / threshold]

### Answer Extraction Logic
[regex / tag parser / JSON parse — with code snippet]

### Aggregation Code
[Python/JS snippet for majority vote]

### Uncertainty Handling
[what to do when no answer meets threshold]
```
