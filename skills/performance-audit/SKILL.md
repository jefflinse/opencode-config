---
name: performance-audit
description: Structured performance investigation workflow covering profiling, benchmarking, bottleneck identification, and optimization verification
---

## What This Skill Does

Defines the workflow for investigating and resolving performance issues — from establishing baselines through profiling, diagnosis, optimization, and verification. Ensures performance improvements are measured, not assumed.

## When To Use

Use this when:
- An endpoint or operation is slower than expected
- Memory usage is growing unexpectedly
- The system struggles under load
- Before a performance-critical release (establish baselines)
- After infrastructure changes that may affect performance

Do NOT use this when:
- The system is down (use incident-response first, profile later)
- The issue is a functional bug (use bug-triage)
- You're optimizing without evidence of a problem (premature optimization)

## Workflow

### Phase 1: Define the Problem
**Agent**: performance-profiler (read-only assessment)

1. What is slow? (specific endpoint, operation, query, or user flow)
2. How slow is it? (current numbers: latency p50/p95/p99, throughput, memory usage)
3. How fast should it be? (target: SLA, user expectation, or "faster than now")
4. When did it get slow? (always, after a deploy, under load, gradually)
5. What is the workload? (request rate, data volume, concurrency)

**Gate**: Problem definition must be agreed upon before profiling begins. "Make it faster" is not a problem definition.

### Phase 2: Establish Baseline
**Agent**: performance-profiler

1. Run benchmarks or load tests against the current code
2. Record baseline metrics: latency (p50, p95, p99), throughput, memory, CPU
3. Identify the profiling approach:
   - **Go**: `go test -bench=. -benchmem -count=5`, pprof CPU/heap profiles
   - **Node.js**: clinic.js doctor/flamegraph, `--prof`, heap snapshots
   - **Java**: JFR recording, async-profiler flamegraphs, GC logs
4. Document the exact commands and environment for reproducibility

**Acceptance criteria**: Baseline numbers recorded with exact reproduction steps.

### Phase 3: Profile and Diagnose
**Agent**: performance-profiler

1. Run the appropriate profiling tools for the stack
2. Analyze profiles to identify the top bottlenecks:
   - CPU-bound: which functions consume the most time?
   - Memory-bound: which allocations dominate?
   - I/O-bound: which external calls are slow?
   - Concurrency: where is contention?
3. Produce a ranked list of findings with estimated impact
4. For each finding, provide specific file/function references and a recommended fix

**Gate**: Findings must be reviewed before optimization begins. Prioritize by impact — fix the biggest bottleneck first.

### Phase 4: Optimize
**Agent**: builder, ts-builder, or java-builder (depending on stack)

1. Address findings one at a time, starting with the highest-impact bottleneck
2. After each optimization:
   - Run the full test suite to verify no behavioral regression
   - Run the benchmark to measure the improvement
3. Commit each optimization separately with before/after numbers in the commit message
4. If an optimization doesn't measurably improve performance, revert it

**Critical rule**: Never sacrifice correctness for performance. If an optimization makes tests fail, it's wrong.

### Phase 5: Verify
**Agent**: performance-profiler

1. Run the full benchmark suite against the optimized code
2. Compare against Phase 2 baselines
3. Verify the target performance is met (or document how close we got)
4. Check for regressions in other metrics (e.g., faster but uses more memory)
5. Run under realistic load to verify improvements hold at scale

**Acceptance criteria**: Measurable improvement documented with before/after numbers.

### Phase 6: Document and Ship
**Agent**: git-workflow

1. Squash into logical commits with performance numbers in messages
2. Create PR with performance comparison data
3. Note any follow-up optimizations identified but not implemented

## Principles

- **Measure, don't guess**: No optimization without profiling data
- **One change at a time**: Each optimization is a separate commit with measured impact
- **Biggest bottleneck first**: Don't optimize a function that takes 1% of total time
- **Correctness over speed**: If tests fail, the optimization is wrong
- **Know when to stop**: If the target is met, stop optimizing
- **Reproducible benchmarks**: Anyone should be able to reproduce your numbers with the documented commands
