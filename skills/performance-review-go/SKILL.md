---
name: performance-review-go
description: Review and optimize Go application performance with evidence-driven analysis. Use when a user asks to diagnose or improve CPU/memory efficiency, database latency, allocation overhead, concurrency bottlenecks, caching opportunities, and benchmark strategy.
---

# Performance Review Go

Review Go code and architecture for performance bottlenecks, then propose safe and measurable optimizations.

## Workflow

1. Define performance objective and scope:
- identify workload profile (throughput, latency, batch, background)
- identify SLO/SLA or target metrics
- identify critical paths and high-traffic endpoints/jobs
2. Analyze CPU and memory hotspots:
- inspect likely hot loops, expensive serialization/parsing, and repeated computations
- identify memory growth patterns, GC pressure, and large object retention
- prioritize hotspots by expected user impact
3. Review data access latency:
- identify slow DB queries, chatty access patterns, and N+1 risks
- evaluate index usage, query shape, and over-fetching
- separate DB time from application processing time
4. Identify allocation reduction opportunities:
- locate unnecessary allocations in hot paths
- reduce transient object churn, avoid avoidable conversions/copies
- consider pooling/reuse only when complexity is justified
5. Review concurrency bottlenecks:
- analyze lock contention, worker saturation, queue backlog, and blocking I/O
- review goroutine lifecycle, fan-out/fan-in pressure, and cancellation handling
- identify backpressure and bounded parallelism needs
6. Evaluate caching suitability:
- identify read-heavy and recomputation-heavy paths suitable for cache
- define cache key, TTL/invalidation strategy, and consistency trade-offs
- assess memory cost and stale-data risk
7. Propose benchmark and validation plan:
- define micro-benchmarks for hot functions
- define integration/load benchmark scenarios for critical flows
- define before/after metrics and acceptance thresholds

## Output Contract

Always return exactly these sections with these headings:

### Findings

- Summarize observed or likely bottlenecks by area.
- Rank by impact and confidence.

### Optimization proposals

- For each proposal include: `Problem`, `Change`, `Expected impact`, `Risk/Trade-off`, `Validation method`.

### Benchmark proposal

- Provide benchmark matrix covering micro and end-to-end/load scenarios.
- Include baseline metrics, target metrics, and pass criteria.

### Rollout notes

- Provide safe rollout strategy, guardrail metrics, and rollback trigger conditions.

## Required Checklist

Always verify all items before finalizing:

- `CPU/memory hotspots`
- `DB latency`
- `allocation reduction`
- `concurrency bottleneck`
- `caching suitability`
- `benchmark proposal`

## Guardrails

- Do not optimize without measurable bottleneck evidence.
- Prefer low-risk/high-impact optimizations first.
- Preserve correctness and maintainability; avoid premature micro-optimizations.
- State uncertainty explicitly when runtime/profile data is missing.
- Recommend instrumentation/profiling when confidence is low.
