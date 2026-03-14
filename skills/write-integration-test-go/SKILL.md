---
name: write-integration-test-go
description: Write and refactor Go integration tests for real infrastructure boundaries. Use when a user asks to test integration across database, queue, or external service stubs with explicit data seeding, cleanup strategy, deterministic execution, and verification at real I/O boundaries.
---

# Write Integration Test Go

Write reliable integration tests that validate behavior across real I/O boundaries with controlled environments.

## Workflow

1. Define integration scope and boundary:
- identify system under test and real I/O components (DB, queue, external stub)
- define what must be real vs what may remain stubbed
- keep focus on cross-component contract and behavior
2. Prepare deterministic test environment:
- start isolated dependencies (container/local test instance/in-memory equivalent approved by architecture)
- configure fixed test settings (timezone, random seed, IDs where feasible)
- disable flaky background jobs unless explicitly part of the scenario
3. Seed test data explicitly:
- insert only required fixtures with clear intent
- keep seed data minimal, readable, and scenario-specific
- record seeded identifiers used in assertions
4. Execute integration flow through real boundaries:
- call public API/use case entrypoint that triggers DB/queue/external interactions
- verify persistence, produced/consumed messages, and stubbed external effects
- assert behavior contract, not internal implementation sequence
5. Apply cleanup logic:
- clean created data/resources after each test (or rollback per test transaction if supported)
- reset queue topics/subscriptions and external stub state
- ensure tests are isolated and repeatable when rerun
6. Validate determinism and failure handling:
- avoid timing races with explicit waits/polls and bounded timeout
- assert stable outcomes for success and failure scenarios
- include at least one error-path integration case per critical boundary

## Integration Testing Rules

- Cover real I/O boundaries that unit tests cannot validate.
- Keep seed data and cleanup logic explicit in test code.
- Keep tests deterministic and independently runnable.
- Use external stubs/fakes only where real third-party systems are impractical.
- Prefer observable outcome assertions (DB state, queue payload, response/status, side effects).

## Required Checklist

Always verify all items before finalizing:

- `seed data is explicit`
- `cleanup logic`
- `deterministic`
- `cover real IO boundary`

## Guardrails

- Do not over-mock components that are part of the integration contract.
- Do not share mutable global test state across cases.
- Do not rely on test execution order.
- Keep timeout and retry settings explicit to avoid flaky tests.
- Add failure diagnostics (logs, IDs, payload snippets) that help debug CI failures quickly.
