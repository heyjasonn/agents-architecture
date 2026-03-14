---
name: design-api-contract
description: Design stable backend API contracts for new or changed endpoints. Use when a user asks to define request/response schema, error contract, compatibility expectations, and validation behavior for HTTP or RPC APIs.
---

# Design API Contract

Produce a concrete API contract ready for implementation and review.

## Workflow

1. Identify operation intent, actor, and trigger.
2. Define method/path (or RPC method) and payload schema.
3. Define response schema and error contract.
4. Define backward compatibility rule and versioning impact.
5. Define validation and edge-case handling expectations.

## Output Contract

### Endpoint definition

- Method/path (or RPC method), auth assumptions.

### Request contract

- Required/optional fields, validation rules.

### Response contract

- Success payload and status semantics.

### Error contract

- Stable error codes and meanings.

### Compatibility notes

- Backward compatibility and rollout constraints.

## Guardrails

- Keep contracts explicit and testable.
- Avoid breaking changes unless requirement explicitly allows.
- Keep naming and error style consistent with repo standards.
