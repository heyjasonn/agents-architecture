## Skills map: agents → skills

Each agent invokes one or more **skills** to fulfill its responsibilities. Skills are discrete capabilities (e.g. Cursor skills, MCP tools, or internal procedures). This map defines which skills each agent uses.

### Summary (by agent)

| Agent | Skills |
|-------|--------|
| **Researcher** | `analyze-requirement`, `test-case-generator-backend` |
| **Planner** | `design-backend-solution`, `design-api-contract`, `design-db-migration`, `create-implementation-plan` |
| **Implementor** | `implement-go-handler`, `implement-go-service`, `implement-go-repository`, `implement-external-integration` |
| **Tester** | `write-unit-test-go`, `write-integration-test-go` |
| **Reviewer** | `review-go-code`, `security-review-backend`, `performance-review-go` |

### Permissions (per skill)

| Skill | Agent(s) | Mode | Side effects | Approval required |
|-------|----------|------|--------------|-------------------|
| `analyze-requirement` | Researcher | read-only | none | no |
| `test-case-generator-backend` | Researcher | read-only | none | no |
| `design-backend-solution` | Planner | read-only | none | no |
| `design-api-contract` | Planner | read-only | none | no |
| `design-db-migration` | Planner | read-only | none | no |
| `create-implementation-plan` | Planner | read-only | none | no |
| `implement-go-handler` | Implementor | write | code changes | sometimes (new endpoints or breaking changes) |
| `implement-go-service` | Implementor | write | code changes | sometimes (business logic changes) |
| `implement-go-repository` | Implementor | write | code + data access changes | yes (when affecting persistence/queries) |
| `implement-external-integration` | Implementor | write | external side effects (HTTP, webhooks, etc.) | yes (new integration or side effects) |
| `write-unit-test-go` | Tester | write | code changes (tests only) | no |
| `write-integration-test-go` | Tester | write | code + external systems in test env | sometimes (when touching real infra) |
| `review-go-code` | Reviewer | read-only | none | no |
| `security-review-backend` | Reviewer | read-only | none | no |
| `performance-review-go` | Reviewer | read-only | none | no |

## By agent

### Researcher

- **analyze-requirement** — Parse ticket/spec into concrete requirements, impacted components, edge cases, risks.
- **test-case-generator-backend** — Produce initial test scenarios from requirements and backend scope.

### Planner

- **design-backend-solution** — Architecture overview, request flow, constraints.
- **design-api-contract** — Endpoints, payloads, status codes.
- **design-db-migration** — Schema changes, migration strategy, rollback.
- **create-implementation-plan** — Ordered implementation steps and assumptions.
- Optional `output.execution_spec.technical_illustrations` / `implementation_sketches` (lightweight, explanatory snippets) are defined in `planner-agent.md`, not as separate skills here.

### Implementor

- **implement-go-handler** — HTTP/gRPC handlers; transport layer only.
- **implement-go-service** — Business logic, use cases.
- **implement-go-repository** — Data access, repositories.
- **implement-external-integration** — Adapters for external APIs, webhooks, etc.

### Tester

- **write-unit-test-go** — Unit tests for handlers, services, repositories.
- **write-integration-test-go** — Integration tests (DB, HTTP, external mocks).

### Reviewer

- **review-go-code** — Correctness, style, maintainability.
- **security-review-backend** — Security findings and recommendations.
- **performance-review-go** — Performance findings and recommendations.

---

## Usage

- **Orchestrator / runner:** When invoking an agent, pass the agent’s skill list so the runtime can load the right skills (e.g. from `.cursor/skills/` or an MCP server).
- **Adding skills:** Add the skill to the agent(s) that should use it and document it in this file. Keep skill names kebab-case and scoped (e.g. `design-db-migration` not `design`).
- **Tech-agnostic variants:** This map is Go/backend-oriented. For other stacks, add parallel skills (e.g. `implement-ts-handler`, `write-unit-test-ts`) and optionally a separate skills map or a `stack` dimension in this file.
