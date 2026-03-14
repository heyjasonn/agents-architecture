---
name: refactor-go-code
description: Refactor Go code to improve maintainability without changing behavior. Use when a user asks to clean up structure, naming, duplication, and complexity while preserving runtime behavior, interfaces, and test outcomes.
---

# Refactor Go Code

Refactor Go code safely with behavior preservation as the primary constraint.

## Workflow

1. Establish behavior baseline:
- identify current externally observable behavior and contracts
- identify critical paths and side effects
- identify tests that define expected behavior
2. Plan safe refactor steps:
- split into small mechanical changes
- avoid mixing refactor and feature changes in the same step
- keep changes reviewable and reversible
3. Execute refactor with clear goals:
- improve naming for intent clarity
- reduce duplication via shared helpers/extracted functions
- isolate responsibilities and tighten boundaries
- simplify branching and control flow
4. Protect behavior during changes:
- preserve public interfaces unless explicitly requested
- keep error semantics and side effects consistent
- avoid altering concurrency or timing behavior unless proven equivalent
5. Validate continuously:
- run targeted tests after each logical step
- run full relevant test suite before finalizing
- ensure no unintended behavior regression

## Refactor Rules

- Prefer small, composable functions with single responsibility.
- Replace ambiguous names with domain-accurate names.
- Remove duplicated logic only when shared abstraction improves clarity.
- Flatten nested branching when equivalent guard clauses improve readability.
- Keep diffs minimal and focused on maintainability improvements.

## Required Checklist

Always verify all items before finalizing:

- `preserve behavior`
- `improve naming`
- `reduce duplication`
- `isolate responsibilities`
- `simplify branching`
- `keep tests passing`

## Guardrails

- Do not introduce new product behavior during refactor.
- Do not change API/contract behavior unless explicitly requested.
- Do not over-abstract; prefer local clarity over speculative reuse.
- Keep refactor traceable with clear before/after intent.
- If behavior equivalence is uncertain, stop and call out risk explicitly.
