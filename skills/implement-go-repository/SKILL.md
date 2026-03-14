---
name: implement-go-repository
description: Implement and refactor Go repository layer for reliable database access across libraries (database/sql, gorm, query builders). Use when a user asks to add or update repository methods with clear query intent, parameterized statements, transaction-aware execution, DB model to domain model mapping, and consistent handling for not-found, duplicate, and constraint errors.
---

# Implement Go Repository

Implement DB access code in Go repository layer with correctness, performance awareness, and clear error semantics.

## Workflow

1. Define repository method contract:
- input parameters and expected domain return type
- query intent (get/list/create/update/delete/upsert)
- transaction requirement for the operation
2. Write clear data-access statements for the chosen library:
- keep query/filter intent explicit and readable (SQL, ORM clauses, or builder expressions)
- select only needed columns
- keep filtering, ordering, and limit behavior deterministic
3. Use parameterized query always:
- bind parameters via placeholders supported by driver/query builder
- avoid string interpolation for dynamic values
- sanitize dynamic query fragments through allowlist strategy when unavoidable
4. Make repository transaction-aware:
- support execution with a transaction-aware abstraction (for example `database/sql`, `gorm`, or query-builder transaction handles)
- ensure methods join caller transaction when provided
- keep commit/rollback ownership at service transaction boundary
5. Map DB model and domain model explicitly:
- scan DB fields into persistence struct with correct Go types
- transform persistence struct to domain model (and reverse for writes)
- keep storage-only fields hidden from domain contract unless required
6. Handle database error categories consistently:
- `not found` -> domain-level not found error
- `duplicate key` -> domain-level already exists/conflict error
- `constraint violation` -> domain-level invalid state/precondition error
- wrap unexpected DB errors with operation context
7. Respect query timeout and context:
- call context-aware DB APIs (`QueryContext`, `ExecContext`, etc.)
- use request/service context for cancellation and deadline propagation

## Query and Performance Rules

- Avoid N+1 query patterns where batch query or join can satisfy the use case.
- Consider index usage for filter/sort/join predicates; flag missing indexes in proposal.
- Keep projection minimal to reduce scan and network overhead.
- Prefer deterministic pagination strategy for list queries.

## Error Mapping Guidance

- Keep driver-specific error parsing centralized in helper/adaptor when possible.
- Preserve root cause with `%w` wrapping.
- Return stable domain errors so service/handler can map consistently.

## Implementation Checklist

Always verify all items before finalizing:

- `avoid N+1 queries when possible`
- `indexes considered`
- `query timeout respected`
- `scan fields with correct types`
- `unit/integration test coverage`

## Guardrails

- Do not place business rules in repository methods.
- Do not leak transport concerns (HTTP/gRPC status/code) into repository.
- Do not execute raw SQL in other layers if architecture mandates repository ownership.
- Do not hardcode repository design to a single DB library; follow project-selected abstractions.
- Keep transaction semantics explicit and predictable.
- Include tests for success path, not found, duplicate/constraint error, and timeout/cancellation behavior.
