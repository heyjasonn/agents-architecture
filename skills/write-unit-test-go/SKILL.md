---
name: write-unit-test-go
description: Write and refactor Go unit tests with clear behavior-focused coverage. Use when a user asks to add or improve tests using table-driven style, happy/edge/error case coverage, interface-based dependency mocking, and assertions that verify business behavior instead of overfitting to implementation details.
---

# Write Unit Test Go

Write maintainable Go unit tests that validate behavior, not incidental implementation.

## Workflow

1. Identify unit under test and behavior contract:
- define observable inputs and outputs
- list business rules and expected side effects
- identify dependency boundaries to mock
2. Build table-driven test cases:
- create a test case struct with name, input, setup, expected result, expected error
- include happy path, edge cases, and error cases
- use descriptive case names for fast diagnosis
3. Mock dependencies via interfaces:
- replace external collaborators with mocks/fakes/stubs at interface boundary
- define mock expectations only for behavior needed by each case
- avoid real network/DB/file calls in unit tests
4. Execute test cases consistently:
- iterate with `t.Run(tc.name, func(t *testing.T) { ... })`
- keep setup, exercise, and assert phases explicit
- isolate case state to avoid cross-test contamination
5. Assert business behavior:
- assert domain output, returned errors, and expected interactions that represent business rules
- avoid asserting private/internal implementation details unless contractually significant
- keep assertions deterministic and readable
6. Validate test quality:
- ensure coverage across success, boundary, and failure paths
- ensure tests fail for meaningful regressions and avoid brittle snapshots

## Testing Rules

- Prefer table-driven tests for variant inputs and expected outcomes.
- Cover happy path, edge cases, and error cases in each relevant use case.
- Mock dependencies through interfaces, not concrete types.
- Assert externally visible behavior and business outcomes.
- Keep tests fast, deterministic, and independent.

## Suggested Test Case Matrix

For each use case, include at least:

- one happy path case
- one boundary/edge case
- one dependency error case
- one validation or business-rule violation case (if applicable)

## Guardrails

- Do not test transport/framework concerns in service/domain unit tests.
- Do not over-mock internal implementation steps that are not part of behavior contract.
- Do not rely on global state, wall-clock timing, or non-deterministic ordering without control.
- Keep fixture setup minimal and local to each test case.
- Prefer explicit expected errors (sentinel/type/message contract) over vague assertions.
