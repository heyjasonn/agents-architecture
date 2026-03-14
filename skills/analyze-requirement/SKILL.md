---
name: analyze-requirement
description: Analyze business or technical requirements and convert them into implementation-ready engineering tasks. Use when starting a new task or receiving a Jira ticket, PRD, technical spec, or requirement note that must be clarified into scope, assumptions, system impact, risks, missing information, and an execution plan.
---

# Analyze Requirement

Convert raw requirements into a clear, implementation-ready technical analysis. Keep outputs concise, explicit, and actionable for engineering planning.

## Workflow

1. Read the input artifact fully.
2. Extract business goal, user roles, and expected outcomes.
3. Summarize the requirement in plain language.
4. Identify scope boundaries:
   - List in-scope items.
   - List out-of-scope items.
5. State assumptions explicitly.
6. Analyze technical impact across:
   - API
   - Database
   - Service/module boundaries
   - Event/queue/integration
   - Authentication/authorization
   - Configuration/feature flags/environments
7. Identify risks and unknowns:
   - Implementation risk
   - Operational risk
   - Data/security/compliance risk
   - Missing information that blocks delivery
8. Propose implementation steps in execution order.

## Output Contract

Always return exactly these sections with these headings:

### Problem summary

- Summarize the problem and desired outcome.
- Mention key actor(s), trigger(s), and value.

### Functional requirements

- List concrete behaviors the system must support.
- Prefer testable statements.

### Non-functional requirements

- Capture performance, reliability, security, observability, compliance, and rollout constraints.
- If not specified, write "Not specified" and keep the assumption in Open questions.

### Technical impact

- API: new endpoints/changes, contracts, backward compatibility.
- DB: schema changes, migrations, indexing, data lifecycle.
- Service: touched modules, dependency changes, side effects.
- Event: producers/consumers, payload contracts, idempotency.
- Auth: role/permission changes, token/session effects.
- Config: env vars, feature flags, defaults, deployment impact.

### Open questions

- List missing details required to estimate or implement safely.
- Mark blockers clearly.

### Proposed implementation steps

- Provide an ordered task list from design to rollout.
- Include testing, migration, monitoring, and rollback readiness where relevant.

## Guardrails

- Do not invent product decisions as facts.
- Separate confirmed facts from assumptions.
- Keep scope discipline; do not silently include out-of-scope items.
- Call out requirement conflicts explicitly.
- Flag risky changes early, especially around data integrity and auth.
