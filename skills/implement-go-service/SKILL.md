---
name: implement-go-service
description: Implement and refactor Go service-layer use cases with explicit business rules and clean boundaries. Use when a user asks to write business logic in service layer, define transaction boundaries, orchestrate repository/external client calls via interfaces, handle idempotency, and wrap errors clearly without transport-layer concerns.
---

# Implement Go Service

Implement business logic in Go service layer with explicit rules and clear orchestration boundaries.

## Workflow

1. Identify service use case contract:
- input DTO/domain command shape
- expected output/result model
- business invariants and policy checks
2. Write business rules explicitly in service:
- validate domain preconditions and state transitions
- compute derived values required by the use case
- keep policy and decision logic in service, not in handler/repository
3. Define transaction boundary clearly:
- choose where transaction starts and ends in service method
- keep all state-changing repository calls inside the boundary
- keep external side effects ordered relative to commit strategy
4. Call dependencies through interfaces only:
- repository interface for persistence access
- external client interface for remote integrations
- avoid direct framework/transport/database driver coupling in service
5. Handle idempotency when operation can be retried:
- detect duplicate requests with stable idempotency key or equivalent guard
- return consistent result for repeated identical requests
- ensure repository writes and side effects remain safe under retries
6. Wrap and classify errors clearly:
- preserve root cause with `%w` wrapping
- add operation context in error message
- return domain-level error types/sentinels suitable for handler mapping
7. Keep observability hooks available:
- pass `context.Context` through every dependency call
- emit structured logs/metrics/traces via injected observability dependencies or helpers

## Service Layer Rules

- Service contains business rules and orchestration.
- Service does not parse HTTP/gRPC requests or build transport responses.
- Service does not execute raw SQL when architecture requires repository abstraction.
- Repository focuses on persistence operations, not domain decision logic.
- Handler maps transport concerns; service stays transport-agnostic.

## Error and Transaction Guidance

- Use typed/sentinel domain errors for expected business failures.
- Wrap infrastructure failures with operation context.
- Keep rollback behavior deterministic inside transaction scope.
- Avoid partial commits: either complete use case effects or return failure.

## Implementation Checklist

Always verify all items before finalizing:

- `business rule explicit`
- `no transport concern`
- `no direct SQL when architecture does not allow it`
- `context passed through`
- `observability hooks available`

## Guardrails

- Keep method responsibilities focused on one use case intent.
- Prevent anemic services by centralizing domain decisions in service/domain layer.
- Do not leak repository/storage models into public service contracts unless intentionally defined.
- Keep idempotency strategy explicit when retries are possible.
- Prefer deterministic outcomes and auditable error paths.
