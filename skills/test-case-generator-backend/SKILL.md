---
name: test-case-generator-backend
description: Generate backend test case checklists from feature requirements. Use when a user asks to convert a feature, PRD, ticket, or API spec into structured test scenarios covering happy path, validation, auth/authz, concurrency, idempotency, timeout/retry, data consistency, and audit/logging.
---

# Test Case Generator Backend

Generate implementation-ready backend test checklists from feature requirements.

## Workflow

1. Parse feature requirement:
- identify actors, entrypoints (API/event/job), and expected outcomes
- identify data entities, side effects, and integrations
- identify assumptions and missing details
2. Derive coverage matrix by risk area:
- happy path
- validation
- auth/authz
- concurrency
- idempotency
- timeout/retry
- data consistency
- audit/logging
3. Generate concrete test scenarios:
- define preconditions
- define input/action
- define expected result and observable assertions
- define negative/failure variants where relevant
4. Classify scenario type:
- unit
- integration
- end-to-end/API contract
- non-functional/resilience
5. Prioritize by impact and failure risk:
- critical (must-run)
- high
- medium
- low
6. Flag gaps and dependencies:
- required test data setup
- external stubs/mocks needed
- monitoring/log assertions required

## Output Contract

Always return exactly these sections with these headings:

### Feature summary

- Restate feature intent, actor, and core flow.
- Note assumptions and open questions.

### Test checklist

- Provide checklist grouped by categories in this order:
1. Happy path
2. Validation
3. Auth/AuthZ
4. Concurrency
5. Idempotency
6. Timeout/Retry
7. Data consistency
8. Audit/Logging
- Each test case must include: `ID`, `Scenario`, `Preconditions`, `Input/Action`, `Expected Result`, `Type`, `Priority`.

### Coverage gaps

- List missing requirements or ambiguous behavior that block reliable test design.
- List suggested clarifications.

### Recommended execution order

- Suggest run order: smoke -> critical path -> high-risk edge/failure -> full regression.

## Scenario Design Rules

- Make scenarios observable and verifiable through API responses, DB state, events, or logs.
- Prefer behavior-level assertions over implementation details.
- Include both positive and negative assertions for security and consistency-sensitive flows.
- Include race/retry conditions when concurrent or distributed updates are possible.
- Include traceability hints (requirement clause or risk area) when possible.

## Guardrails

- Do not invent product decisions as confirmed facts.
- Keep assumptions explicit and separate from confirmed behavior.
- Avoid duplicate test cases with equivalent coverage intent.
- Ensure each category has at least one scenario; state `Not applicable` only with reason.
- Keep wording concise and executable by QA/engineers.
