---
name: design-backend-solution
description: Propose backend solution design from product or technical requirements. Use when a user asks to transform a requirement, PRD, ticket, or feature request into an actionable backend design with architecture pattern selection, handler/service/repository layering, API contract definition, database schema change suggestions, and rollback strategy.
---

# Design Backend Solution

Convert raw requirements into an implementation-ready backend design document.

## Workflow

1. Parse the requirement and restate goal, actors, and boundaries.
2. Select architecture pattern that best fits the requirement:
- CRUD service with layered monolith
- Domain-oriented module boundaries
- Event-driven flow for async or cross-service consistency
- CQRS/read-write split for high read scale or complex query shape
3. Split responsibilities into `handler`, `service`, `repository` layers:
- `handler`: transport mapping, auth/context extraction, input/output validation, error mapping.
- `service`: business rules, orchestration, transaction boundary, idempotency policy.
- `repository`: persistence abstraction, query/write methods, locking and consistency behavior.
4. Define contract and payload shape:
- API endpoint/method/path
- request and response fields
- validation and error codes
- backward compatibility strategy
5. Propose schema changes only when needed:
- new tables/columns/indexes/constraints
- migration direction (forward and backward)
- data backfill approach when required
6. Define rollback strategy explicitly:
- application rollback behavior
- migration rollback safety
- feature flag or kill-switch fallback
- data integrity guardrails during rollback
7. Review edge cases and trade-offs before finalizing.

## Output Contract

Always output exactly these sections with these headings:

### Architecture overview

- State selected pattern and why it fits.
- Describe module boundaries and layer ownership.
- Name dependencies, transaction boundaries, and consistency model.

### Request flow

- Show step-by-step flow from inbound request/event to persistence and response.
- Mark validation, authorization, idempotency, and failure branches.
- Include sync/async handoff points if any.

### Data model changes

- If no DB change needed, write `No schema change required` and explain why.
- If needed, list table/column/index/constraint changes.
- Include migration and backfill notes, plus compatibility concern.

### API contract

- Specify endpoint/event contract with method/path/topic.
- Provide request and response schema (field names + types + required/optional).
- Include error model, status codes, and versioning/backward-compatibility notes.

### Edge cases

- List failure and boundary scenarios.
- Cover race conditions, retries, partial failures, duplicates, and invalid states.
- Include monitoring and alert signals for critical failures.

### Trade-offs

- Compare chosen approach with at least one plausible alternative.
- Explain impacts on complexity, scalability, consistency, and delivery speed.
- Include a clear `Rollback strategy` subsection with app rollback + migration rollback steps.

## Guardrails

- Avoid inventing product decisions as confirmed facts.
- Separate assumptions from confirmed requirements.
- Keep contracts and schema proposals concrete and testable.
- Call out risky or irreversible migrations clearly.
- Prefer incremental and reversible rollout plans.
