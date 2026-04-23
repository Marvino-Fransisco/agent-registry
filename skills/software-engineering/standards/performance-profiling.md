

# Performance Profiling

Find what's slow, understand why, then fix it — in that order.

## Performance Checklist

### Algorithmic Complexity
- What is the Big O of this algorithm?
- Can nested loops be flattened with a hash map?
- Are there repeated computations that could be cached (memoisation)?
- Is sorting happening unnecessarily inside a loop?

Common complexity traps:
| Pattern | Complexity | Fix |
|---|---|---|
| Nested loops over same data | O(n²) | Use hash map → O(n) |
| Linear search in hot path | O(n) | Index or sort → O(log n) |
| Repeated expensive computation | O(n×k) | Memoize → O(n) |
| Sorting inside loop | O(n² log n) | Sort once outside |

### Memory
- Are large objects held in memory longer than needed?
- Are there unbounded caches or collections that grow forever?
- Is there object creation happening in tight loops (GC pressure)?
- Are streams/generators used instead of loading everything into memory?

### I/O
- Are there sequential I/O calls that could be parallelised?
- Is there unnecessary serialisation/deserialisation?
- Are responses paginated or streamed instead of loaded fully?

### Data Structures
| Need | Use |
|---|---|
| Fast lookup by key | HashMap / Dict |
| Ordered + fast lookup | TreeMap / SortedDict |
| Fast membership check | HashSet |
| FIFO queue | Deque |
| Priority ordering | Heap / PriorityQueue |
| Prefix search | Trie |

## Analysis Process

1. **Measure first** — don't optimise based on intuition
   - Identify the hot path (what runs most often?)
   - Estimate current complexity
2. **Isolate the bottleneck**
   - Is it CPU (algorithm), memory, or I/O?
3. **Apply the fix**
   - Reduce complexity tier first (O(n²) → O(n))
   - Then reduce constant factors (avoid repeated work)
   - Then consider caching/memoisation
4. **Verify improvement**
   - Measure again after the fix

## Output Format

```
## Performance Analysis for [function/module]

### Current Complexity
Time: O(?) — [explanation]
Space: O(?) — [explanation]

### Bottlenecks Found

🔴 [description] — O(n²) nested loop
→ Fix: [description] — reduces to O(n)

🟠 [description] — repeated DB call in loop
→ Fix: batch query outside loop

### Optimised Code
[implementation]

### After Fix
Time: O(?) — [explanation]
Space: O(?) — [explanation]
```
