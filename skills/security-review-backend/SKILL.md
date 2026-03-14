---
name: security-review-backend
description: Perform security-focused backend review for Go services. Use when a user asks to evaluate auth/authz, input validation, secret handling, injection risks, data exposure, and operational safeguards before merge.
---

# Security Review Backend

Run a targeted security review with actionable findings.

## Workflow

1. Identify trust boundaries and sensitive assets.
2. Verify auth/authz behavior for affected paths.
3. Inspect input validation and sanitization.
4. Inspect secret handling, logging redaction, and data exposure.
5. Inspect data access and injection risk.
6. Report findings with severity and fix direction.

## Output Contract

### Critical findings

- Exploitable or high-risk issues with evidence.

### Important findings

- Medium-risk concerns and mitigations.

### Hardening recommendations

- Non-blocking improvements.

### Security readiness

- `Not ready`, `Ready with follow-ups`, or `Ready` with rationale.

## Guardrails

- Provide evidence for every finding.
- Prioritize real exploitability over theoretical concerns.
- Avoid generic advice disconnected from code behavior.
