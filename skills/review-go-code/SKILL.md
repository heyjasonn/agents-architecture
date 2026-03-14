---
name: review-go-code
description: Review Go code with senior-level engineering rigor. Use when a user asks for code review, PR review, or quality assessment focused on correctness, simplicity, performance, readability, error handling, concurrency safety, testability, security, and observability.
---

# Review Go Code

Review Go code like a senior engineer with risk-first prioritization and actionable feedback.

## Review Workflow

1. Build context:
- identify changed files and affected behavior
- identify service boundaries, data flow, and critical execution paths
- identify assumptions and missing context
2. Evaluate systematically across review dimensions:
- correctness
- simplicity
- performance
- readability
- error handling
- concurrency safety
- testability
- security
- observability
3. Prioritize findings by production risk:
- user impact
- data integrity risk
- reliability/latency risk
- maintainability and operational burden
4. Verify test posture:
- check if existing/new tests cover main behavior and failure paths
- flag missing tests for risky code paths
5. Determine merge readiness:
- blocked if unresolved critical issues remain
- conditionally ready if only non-blocking improvements remain

## Review Dimensions

### Correctness

- Check for logic errors, edge-case breaks, and unintended behavior changes.
- Verify state transitions, invariants, and boundary conditions.

### Simplicity

- Prefer straightforward control flow and minimal abstraction.
- Flag unnecessary indirection or premature generalization.

### Performance

- Check algorithmic complexity, allocations, hot-path efficiency, and I/O usage.
- Flag avoidable repeated work, N+1 access patterns, and unbounded operations.

### Readability

- Check naming clarity, cohesion, function size, and code organization.
- Flag confusing branching and hidden side effects.

### Error Handling

- Check explicit handling of expected failure modes.
- Verify context-rich error wrapping and stable error semantics.

### Concurrency Safety

- Check race risk, lock/channel misuse, deadlock potential, and cancellation handling.
- Verify goroutine lifecycle management and resource cleanup.

### Testability

- Check dependency boundaries, interface seams, determinism, and mockability.
- Flag code that is difficult to isolate or verify.

### Security

- Check input validation, auth/authz assumptions, secret handling, and injection risks.
- Flag sensitive data exposure in logs or responses.

### Observability

- Check logging quality, metrics coverage, tracing propagation, and diagnosability.
- Ensure failure paths are observable for incident response.

## Output Contract

Always return exactly these sections with these headings:

### Critical issues

- List blockers that can cause incorrect behavior, data loss/corruption, security exposure, severe reliability impact, or unsafe rollout.
- Include file/location, impact, and concrete fix direction.

### Important improvements

- List non-blocking but high-value changes improving maintainability, performance, robustness, or test coverage.
- Include rationale and suggested change.

### Nice-to-have

- List optional refinements (style/clarity/refactoring) with lower urgency.

### Merge readiness

- Provide one status: `Not ready`, `Ready with follow-ups`, or `Ready`.
- Explain the decision in 1-3 concise bullets.

## Guardrails

- Prioritize findings over summary.
- Do not invent defects without evidence in code.
- Keep recommendations actionable and specific.
- Distinguish confirmed issues from assumptions.
- If no issues are found, state that explicitly and mention residual risk/testing gaps.
