---
name: design-db-migration
description: Design safe database migrations for production systems. Use when a user asks to plan schema evolution with forward migration, backward compatibility, backfill strategy, rollout/rollback plan, and traffic impact analysis while minimizing downtime and lock risk.
---

# Design DB Migration

Design migration plans that are safe to deploy under live traffic and reversible when needed.

## Workflow

1. Assess change scope and risk:
- identify affected tables, columns, indexes, constraints, and query paths
- identify read/write traffic patterns and critical SLAs
- classify migration as metadata-only, data rewrite, or long-running backfill
2. Design forward migration steps:
- split into small, deployable phases
- prefer additive changes first (new nullable column/table/index) before destructive changes
- avoid risky one-shot DDL on hot tables when phased approach is possible
3. Ensure backward compatibility:
- keep old and new schemas readable/writable during transition window
- support dual-read/dual-write or compatibility adapters when needed
- defer dropping old columns/constraints until all app versions are compatible
4. Define backfill strategy:
- choose online batched backfill with bounded batch size and pacing
- make backfill resumable and idempotent
- define verification checks, progress metrics, and stop conditions
5. Define rollout and rollback plan:
- rollout: migration order, app release order, feature-flag gates, monitoring checkpoints
- rollback: app rollback compatibility, reversible migration steps, fallback behavior
- document irreversible operations explicitly and place them last
6. Evaluate impact on existing traffic:
- estimate lock duration, write amplification, replication lag, and query latency impact
- plan off-peak execution windows if needed
- include mitigation for elevated error rate or latency during rollout

## Output Contract

Always return exactly these sections with these headings:

### Forward migration

- List ordered migration phases and exact schema actions.
- Mark online-safe vs high-risk operations.

### Backward compatibility

- Explain how old and new application versions coexist safely.
- Specify when destructive cleanup can happen.

### Backfill strategy

- Define batch policy, throttling, idempotency, and verification approach.
- Include resume/retry behavior.

### Rollout and rollback plan

- Provide deployment order across app and DB changes.
- Include rollback triggers and exact rollback actions.

### Impact on existing traffic

- Explain expected effects on read/write latency, lock behavior, and throughput.
- Include monitoring signals and abort conditions.

## Safety Checklist

Always verify all items before finalizing:

- `zero/minimal downtime`
- `nullable/default strategy`
- `lock risk considered`
- `index creation strategy considered`

## Guardrails

- Prefer expand-and-contract migration patterns for high-traffic systems.
- Avoid immediate destructive changes in the same release as additive schema changes.
- Keep migration scripts deterministic, observable, and retry-aware.
- Call out engine-specific caveats (PostgreSQL/MySQL/etc.) when they affect lock behavior.
- Require post-migration validation before cleanup phase.
