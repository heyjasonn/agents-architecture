---
name: create-implementation-plan
description: Create a step-by-step implementation plan from approved backend design output. Use when a user asks to convert architecture/design into an execution sequence with dependencies, file touchpoints, testing strategy, and rollout-safe ordering.
---

# Create Implementation Plan

Generate an actionable implementation sequence for backend changes.

## Workflow

1. Read design and requirement context.
2. Break implementation into ordered steps with dependencies.
3. Map each step to impacted files/components.
4. Include test strategy and validation checkpoints.
5. Include migration/rollback notes when DB changes exist.

## Output Contract

Return these sections:

### Implementation steps

- Ordered task list with dependency notes.

### File/component impact

- Expected changed files or modules per step.

### Test plan

- Unit, integration, and regression expectations.

### Risks and mitigations

- Key execution risks and safe rollback guidance.

## Guardrails

- Do not write production code.
- Do not skip testing and rollback planning.
- Keep steps concrete and directly executable.
