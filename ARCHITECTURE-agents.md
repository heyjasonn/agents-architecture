## Agents runtime contract

Source-of-truth for how the orchestrator and role agents exchange data at runtime. This file defines the concrete YAML/JSON shapes expected by a runner implementation.

All examples below are written as YAML for readability, but the same structure MUST be accepted/produced as JSON.

---

### 1. Task envelope (input to orchestrator)

Minimal shape the runner sends to the orchestrator:

```yaml
task_id: "<string>"               # Stable across the whole run
requirement: "<raw spec text>"    # User requirement or ticket text
category: "<new-feature|bugfix|db-change|external-integration|other|null>"
context:                          # Optional, runtime-specific
  repo_root: "<path>"
  stack: "go-backend"
  env: "dev"                      # or "staging" / "prod-sim"
state: {}                         # See State model below
handoff_outputs:                  # Optional, may be empty on first call
  researcher: <ResearcherArtifact?>
  planner: <PlannerArtifact?>
  implementor: <ImplementorArtifact?>
  tester: <TesterArtifact?>
  reviewer: <ReviewerArtifact?>
```

The orchestrator may add more context fields as long as they are namespaced (e.g. `runtime.*`, `tools.*`).

---

### 2. Shared state model

State is a small, versioned object the orchestrator updates between steps:

```yaml
state:
  version: 1                      # Monotonic; bump on any state change

  requirement_version: 1
  research_version: 0
  plan_version: 0
  implementation_version: 0
  test_version: 0
  review_version: 0

  current_status: "researching"   # researching|planning|implementing|testing|reviewing|blocked|complete

  blocking_issues:
    - from_agent: "tester"
      description: "Invalid status returns 500, plan says 400"
      acceptance_condition: "Handler returns 400 with test covering invalid status"

  acceptance_conditions:
    - "All critical_issues resolved"
    - "Regression risk <= medium"
```

Only the orchestrator should mutate `state`; role agents propose changes via their artifacts.

---

### 3. Common artifact envelope (output of each agent)

All role outputs share the same top-level envelope:

```yaml
task_id: "<string>"
agent: "<researcher|planner|implementor|tester|reviewer|orchestrator>"
version: 1                        # Monotonic per (task_id, agent)

output: {}                        # Agent-specific keys (see below)

meta:                             # Optional metadata for observability/eval
  eval: {}                        # Role-specific scores, booleans, etc.
  timing_ms:
    total: 1234
  tokens:
    prompt: 1200
    completion: 350
  tools_used:
    - name: "user-postgres.query"
      count: 2
  retries: 0
```

The runner MUST validate:

- `task_id` matches the current task.
- `agent` is one of the allowed roles.
- `version` is an integer and strictly greater than the previous version for that agent.
- `output` is an object containing the required keys for that role (see below).

---

### 4. Role-specific output schemas

#### 4.1 Researcher artifact (`<ResearcherArtifact>`)

```yaml
task_id: "<string>"
agent: "researcher"
version: 1

output:
  problem_summary: "<one-paragraph scope>"

  requirements:
    - "<testable requirement 1>"
    - "<testable requirement 2>"

  impacted_components:
    - "api/handlers/"
    - "internal/customer/"

  dependencies:
    - "postgres"
    - "payment-provider-X"

  edge_cases:
    - "Empty result set"
    - "Invalid status parameter"

  risks:
    - description: "Large exports may be slow"
      severity: "medium"          # low|medium|high
      area: "performance"

  open_questions:
    - "Is pagination required for v1?"

  test_scenarios:
    - "Happy path: valid filters"
    - "Empty result: still 200 with header-only CSV"

  facts:
    - "Service is written in Go"
    - "Existing customer listing endpoint uses paging"

  assumptions:
    - statement: "Customer model has created_at and status fields"
      reason: "Inferred from existing list endpoint"
      risk_level: "medium"

  risk_summary:
    total: 3
    by_severity:
      low: 1
      medium: 2
      high: 0

meta:
  eval:
    requirement_completeness_score: 0.9      # 0–1
    risk_coverage_score: 0.8                 # 0–1
    open_questions_blocking: false
```

---

#### 4.2 Planner artifact (`<PlannerArtifact>`)

```yaml
task_id: "<string>"
agent: "planner"
version: 1

output:
  architecture_overview: |
    New GET handler parses query params, calls domain ListWithFilters,
    streams CSV to response using existing auth middleware.

  request_flow:
    - "Parse query (date_from, date_to, status)"
    - "Validate parameters"
    - "Call customer.ListWithFilters(ctx, filters)"
    - "Stream CSV to response"

  api_contract:
    method: "GET"
    path: "/api/customers/export"
    query_params:
      date_from: { type: "string", format: "date", required: false }
      date_to: { type: "string", format: "date", required: false }
      status: { type: "string", enum: ["active", "inactive", "all"], required: false }
    responses:
      "200":
        content_type: "text/csv"
      "400":
        reason: "Validation error"
      "500":
        reason: "Internal error"

  db_changes:
    migrations: []                 # or list of migration specs

  implementation_steps:
    - "Add ListWithFilters to customer service"
    - "Add export handler and route"
    - "Wire validation and CSV writer"

  constraints:
    - "Reuse existing auth middleware"
    - "No new tables"

  assumptions:
    - "Customer repository already supports filtering by status"

  non_functional_requirements:
    latency_p50_ms: 200
    latency_p95_ms: 500
    observability:
      tracing: true
      metrics: true
    security:
      auth_required: true
      pii_in_csv: true

  backward_compatibility:
    strategy: "Additive change; no existing endpoints modified"
    notes:
      - "No change to existing customer list API"

  rollback_strategy:
    approach: "Deploy behind feature flag; disable flag to rollback"
    db_rollback_required: false

meta:
  eval:
    backward_compatibility_checked: true
    rollback_strategy_present: true
    non_functional_constraints_covered: true
```

---

#### 4.3 Implementor artifact (`<ImplementorArtifact>`)

```yaml
task_id: "<string>"
agent: "implementor"
version: 1

output:
  changed_files:
    - "internal/customer/list.go"
    - "api/handlers/customer_export.go"
    - "api/routes.go"

  implemented_rules:
    - "Handler only parses/validates HTTP and delegates to domain layer"
    - "Reused existing validation helpers"

  known_limitations:
    - "No pagination; large exports may be slow"

  areas_needing_tests:
    - "Handler invalid status param returns 400"

  integration_points:
    - "HTTP GET /api/customers/export"

  tests_required_before_review:
    value: true
    explanation: "Unit tests for handler + domain added; E2E deferred"

  migrations_applied:
    - id: "20260317_add_export_index"
      description: "Index on customers.created_at for export"
      status: "applied"

  config_changes:
    - key: "customers.export.max_rows"
      default: 10000
      scope: "service"

  feature_flags_used:
    - name: "customers_export_enabled"
      default: false
      rollout_plan: "Canary 5% → 25% → 100%"

meta:
  eval:
    changed_files_aligned_with_plan: true
```

---

#### 4.4 Tester artifact (`<TesterArtifact>`)

```yaml
task_id: "<string>"
agent: "tester"
version: 1

output:
  tests_added:
    - "api/handlers/customer_export_test.go"
    - "internal/customer/list_test.go"

  covered_scenarios:
    - "Happy path with filters"
    - "No filters"
    - "Empty result"
    - "Invalid date_from"

  uncovered_scenarios:
    - "E2E HTTP → DB path with real DB"

  observed_risks:
    - description: "Large exports not covered by performance test"
      severity: "medium"

  test_matrix:
    unit:
      covered: true
    integration:
      covered: true
    e2e:
      covered: false
    concurrency:
      covered: false
    rollback_migration:
      covered: false
    performance_smoke:
      covered: false
    security_regression:
      covered: false

  regression_risk_level: "medium"   # low|medium|high

meta:
  eval:
    critical_scenarios_covered: true
    regression_risk_level: "medium"
```

---

#### 4.5 Reviewer artifact (`<ReviewerArtifact>`)

```yaml
task_id: "<string>"
agent: "reviewer"
version: 1

output:
  critical_issues:
    - id: "CSV-VAL-001"
      description: "Invalid status value returns 500 instead of 400"
      severity: "high"
      file: "api/handlers/customer_export.go"

  improvements:
    - "Consider adding max row limit and returning 400 if exceeded"

  security_findings: []

  performance_findings:
    - "No performance tests for large exports; monitor p95 latency after rollout"

  merge_readiness: "Ready with follow-ups"   # Not ready|Ready with follow-ups|Ready

  blocking_reasons:
    - "CSV-VAL-001 must be fixed before merge"

  evidence:
    - type: "test"
      reference: "api/handlers/customer_export_test.go:InvalidStatus"
    - type: "behavior"
      reference: "Manual curl: status=foo returns 500"

  recommended_owner:
    - "implementor"

  required_followups_after_merge:
    - "Add performance smoke test for 10k-row export"
    - "Monitor p95 latency for export endpoint for 7 days"

meta:
  eval:
    severity_distribution:
      critical: 0
      high: 1
      medium: 1
      low: 0
    merge_readiness: "Ready with follow-ups"
```

---

### 5. Orchestrator control artifacts

The orchestrator may emit small control artifacts to signal special states.

#### 5.1 Manual handoff

```yaml
task_id: "<string>"
agent: "orchestrator"
version: 1

output:
  type: "manual_handoff"
  reason: "Out-of-scope infra redesign"
  suggested_owner: "platform-team"
  notes:
    - "Requirement describes service mesh migration; outside pipeline scope"
```

#### 5.2 Loopback decision (logical, not necessarily a separate artifact)

Loopbacks are expressed in `state.blocking_issues` and by re-invoking the previous agent with updated `state` and `handoff_outputs`. A runner may optionally persist a dedicated loopback record:

```yaml
task_id: "<string>"
agent: "orchestrator"
version: 2

output:
  type: "loopback"
  from_agent: "tester"
  to_agent: "implementor"
  blocking_issue: "Invalid status returns 500 instead of 400"
  acceptance_condition: "Handler returns 400 for invalid status and test added"
```

---

### 6. Implementation notes for runners

- Treat this file as the **reference schema** for serialization/deserialization.
- Prefer generating types from it (e.g. TypeScript types, Go structs, JSON Schemas).
- Enforce:
  - Strict JSON/YAML parsing.
  - Presence of required keys per role.
  - Version monotonicity per `(task_id, agent)`.
- Keep prompts short by:
  - Passing only `output` sections + minimal context to role prompts.
  - Storing full envelopes and history in a separate store (DB, S3, etc.) for observability.

