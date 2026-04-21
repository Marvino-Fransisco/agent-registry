# Research: Performance Benchmark

## Purpose
Find or produce credible benchmark data, interpret it in the context of the project's actual load profile, and give a verdict on whether the technology can meet the performance requirements.

## Key performance dimensions

| Dimension | Metric | Tool |
|-----------|--------|------|
| **Throughput** | Requests/sec, msgs/sec, rows/sec | wrk, hey, k6, pgbench |
| **Latency** | p50, p95, p99, p999 (not average) | wrk2, k6, Gatling |
| **Memory** | RSS, heap usage under load | /proc/meminfo, heaptrack, pmap |
| **CPU** | % utilization at target RPS | top, perf, py-spy |
| **Cold start** | Time to first byte from idle | curl timing, Lambda metrics |
| **Concurrency** | Max concurrent connections before degradation | wrk -c, k6 VU ramp |
| **Scalability** | Does performance scale linearly with resources? | horizontal scale test |

## Investigation steps

### Step 1: Find existing benchmarks
Prioritize in this order:
1. Official benchmarks from the project (be skeptical — they cherry-pick)
2. Independent benchmarks from credible sources:
   - [TechEmpower Web Framework Benchmarks](https://www.techempower.com/benchmarks/) for web frameworks
   - [DB-Engines](https://db-engines.com) for databases
   - [benchmarksgame.alioth.debian.org](https://benchmarksgame-team.pages.debian.net/benchmarksgame/) for language runtimes
   - Engineering blogs from Cloudflare, Discord, Shopify, Uber, etc.
3. Community benchmarks (GitHub repos with reproducible methodology)

### Step 2: Validate benchmark conditions
Check if the benchmark conditions match the project:
- Same or similar hardware tier?
- Realistic payload size (not just `{"ok": true}`)?
- Warm vs cold test? (Always prefer warm for steady-state comparison)
- Connection pooling enabled/disabled?
- Same concurrency level as expected production traffic?
- Network latency accounted for?

### Step 3: Run micro-benchmarks if needed
```bash
# HTTP throughput test
wrk -t12 -c400 -d30s --latency http://localhost:3000/api/health

# k6 ramp test
cat <<'EOF' > /tmp/bench.js
import http from 'k6/http';
import { check } from 'k6';
export const options = {
  stages: [
    { duration: '30s', target: 100 },
    { duration: '1m', target: 100 },
    { duration: '10s', target: 0 },
  ],
};
export default function () {
  const res = http.get('http://localhost:3000/api/health');
  check(res, { 'status 200': (r) => r.status === 200 });
}
EOF
k6 run /tmp/bench.js

# Memory profile (Node.js)
node --max-old-space-size=512 --expose-gc -e "
  const used = process.memoryUsage();
  console.log(JSON.stringify(used));
"

# Postgres query timing
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) SELECT ...;
```

### Step 4: Interpret results
Always report:
- **p99 latency** (not average — averages hide tail latency pain)
- **Throughput at target p99** (not peak throughput — sustainable throughput at SLA)
- **Degradation curve** — does it cliff or gracefully degrade under overload?
- **Memory footprint** at target concurrency

## Output format
```
### Performance Finding: <technology>
- **Test type**: Throughput / Latency / Memory / Scalability
- **Source**: [URL or "measured by researcher on <date>"]
- **Hardware**: <instance type or CPU/RAM spec>
- **Conditions**: <concurrency, payload, warm/cold, pooling>
- **Results**:
  - Throughput: X req/s
  - p50 latency: Xms | p95: Xms | p99: Xms
  - Memory at load: X MB RSS
- **Verdict vs project SLA**: ✅ Meets / ⚠️ Marginal / ❌ Does not meet
- **Caveat**: <what conditions could change this result>
```

## Constraints
- **Never report average latency as the primary metric** — always use percentiles
- Benchmarks from 2+ years ago are suspect — runtime performance changes with versions
- Always note if the benchmark was run on the same language runtime/version as the project
- Synthetic benchmarks do not replace production profiling — recommend production tracing as validation
- Flag if a vendor's benchmark has suspiciously round numbers or no methodology published
