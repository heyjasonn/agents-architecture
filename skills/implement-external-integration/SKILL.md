---
name: implement-external-integration
description: Implement and refactor integrations with external services (payment, notification, CRM, object storage, queues) using resilient and secure patterns. Use when a user asks to build or update external client adapters with interface abstraction, retry strategy, timeout/circuit-breaker controls, request/response log redaction, idempotency handling, and partial-failure recovery.
---

# Implement External Integration

Implement external service integrations with clear boundaries, reliability controls, and safe observability.

## Workflow

1. Define integration boundary and interface abstraction:
- create service-facing interface for required capabilities
- keep provider-specific SDK details inside adapter/client implementation
- support provider swap or test doubles without changing business layer
2. Define request/response contract mapping:
- map domain input to provider request model explicitly
- map provider response to internal domain DTO
- normalize provider-specific statuses and error codes to internal error model
3. Set resilience policy:
- specify retry strategy (max attempts, backoff, jitter, retryable error classes)
- enforce timeout budget per call and per operation
- apply circuit breaker or equivalent fail-fast control when architecture supports it
4. Implement secure logging and redaction:
- log request/response metadata needed for debugging and audit
- redact secrets, tokens, PII, payment data, and sensitive payload fields
- avoid raw payload logs unless explicitly permitted and sanitized
5. Handle partial failure scenarios:
- identify which side effects can succeed independently and which must be compensated
- document fallback behavior (queue for retry, degraded mode, manual reconciliation)
- keep failure outcomes deterministic and observable
6. Implement idempotency where duplicate execution is possible:
- support idempotency keys or dedup strategy for retried/replayed requests
- ensure repeated calls do not create duplicate side effects
7. Include observability hooks:
- emit structured logs, key metrics (latency, success/failure, retries), and traces
- propagate context and correlation IDs across integration boundaries

## Integration Rules

- Access external services via interface-based adapters.
- Keep business logic in service/domain layer, not inside transport or SDK wrappers.
- Keep secret/config loading in secure configuration path (env/secret manager), never hardcoded.
- Keep retry policy explicit; avoid unbounded retries.
- Respect provider rate limits and error contracts.

## Failure and Fallback Guidance

- Classify errors as retryable vs non-retryable.
- Distinguish upstream unavailability, timeout, validation, and auth errors.
- Define fallback behavior per integration path and state expected user/system impact.
- If eventual consistency is used, define reconciliation or dead-letter handling.

## Implementation Checklist

Always verify all items before finalizing:

- `no secret hardcode`
- `idempotency considered`
- `fallback documented`
- `metrics/logging included`

## Guardrails

- Do not leak secrets or full sensitive payloads in logs.
- Do not couple business layer directly to provider SDK types.
- Do not hide retry/timeout behavior; keep policies explicit in code/config.
- Do not ignore partial failure paths; define recovery action.
- Add tests for success path, timeout, retry exhaustion, non-retryable failures, and fallback behavior.
