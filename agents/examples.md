# Multi-agent pipeline examples

Concrete examples of how a requirement flows through Researcher → Planner → Implementor → Tester → Reviewer in a spec-driven workflow. Use these to see the **shape of each handoff** and when **loopbacks** happen. **Example 5** shows a Planner `execution_spec` with optional `technical_illustrations` (written spec remains the contract).

---

## Example 1: New feature (full chain)

**User requirement:**  
"Add a customer export endpoint that returns CSV with optional filters (date range, status)."

### Step 0: Orchestrator

- **Input:** `requirement` = (above), `category` = `new-feature` (or omitted → Researcher classifies).
- **Action:** Route to full chain. No loopback.

---

### Step 1: Researcher

**Input:** Raw requirement + repo context (e.g. "Go service, `internal/` for domain, `api/` for HTTP handlers").

**Output (handoff to Planner, full artifact envelope):**

```yaml
task_id: "T-EXPORT-001"
agent: "researcher"
version: 1

output:
  problem_summary: |
    Need an HTTP endpoint to export customers as CSV with optional query filters
    (date range, status). Backend-only; no UI in scope.

  requirements:
    - Export endpoint returns CSV of customers.
    - Optional query params: date_from, date_to, status (active/inactive/all).
    - Default: all customers if no filters. Pagination out of scope for v1.

  impacted_components:
    - api/handlers/ (new or existing export handler)
    - internal/customer/ (list/query logic)
    - config (if new route or middleware)

  dependencies: []

  edge_cases:
    - Empty result: return 200 with header-only CSV.
    - Invalid date range: 400 with clear error.
    - Invalid status value: 400.

  risks:
    - description: "Large exports could strain memory; consider streaming or limit later."
      severity: "medium"
      area: "performance"

  open_questions: []

  test_scenarios:
    - "Happy path: valid filters, non-empty CSV."
    - "No filters: all customers."
    - "Empty result, invalid params, invalid status."

  facts:
    - "Service is written in Go."

  assumptions:
    - statement: "Customer model has created_at and status fields."
      reason: "Inferred from existing list endpoint."
      risk_level: "medium"

meta:
  eval:
    requirement_completeness_score: 0.9
    risk_coverage_score: 0.8
    open_questions_blocking: false
```

---

### Step 2: Planner

**Input:** Researcher output above + repo patterns (e.g. "handlers in `api/handlers/`, use `internal/` for business logic").

**Output (handoff to Implementor):**

```yaml
task_id: "T-EXPORT-001"
agent: "planner"
version: 1

output:
  execution_spec:
    overview: "Add CSV customer export endpoint with optional filters."
    scope:
      in: ["GET /api/customers/export", "filter validation", "CSV response"]
      out: ["pagination", "UI changes"]
    functional_requirements:
      - "Endpoint returns CSV of customers."
      - "Optional filters: date_from, date_to, status(active|inactive|all)."
    business_rules:
      - "Invalid date range returns 400."
      - "Invalid status returns 400."
    edge_cases:
      - "Empty result returns 200 with header-only CSV."
    acceptance_criteria:
      - "Given status=foo, response is 400 with validation error."
      - "Given no filters, response is 200 CSV."
    test_scenarios:
      - "Happy path with filters."
      - "Invalid date range and invalid status."
    non_functional_requirements:
      - "Reuse existing auth/middleware."
      - "No new DB tables in v1."
    rollout_and_rollback:
      - "Additive route rollout behind existing auth."
      - "Rollback by removing route wiring and handler."
    open_questions: []
    implementation_plan:
      - "Add ListWithFilters to customer service."
      - "Add export handler and route."
      - "Map validation failures to 400."

meta:
  eval:
    execution_spec_complete: true
    rollback_strategy_present: true
    non_functional_constraints_covered: true
```

---

### Step 3: Implementor

**Input:** Planner output above + codebase (actual files, existing handlers, customer repo interface).

**Output (handoff to Tester):**

```yaml
task_id: "T-EXPORT-001"
agent: "implementor"
version: 1

output:
  changed_files:
    - internal/customer/list.go
    - api/handlers/customer_export.go
    - api/routes.go
  implemented_rules:
    - "Mapped invalid status/date validation to 400 per execution_spec.acceptance_criteria."
    - "Kept business logic in internal layer per execution_spec.business_rules."
  known_limitations:
    - "No pagination in v1 (in scope boundary from execution_spec)."
  areas_needing_tests:
    - "Handler: valid filters, empty result, invalid status/date."
  integration_points:
    - "GET /api/customers/export"
  tests_required_before_review: true
  migrations_applied: none
  config_changes: none
  feature_flags_used: none

meta:
  eval:
    changed_files_aligned_with_execution_spec: true
    tests_required_before_review: true
```

*(Plus the actual code changes.)*

---

### Step 4: Tester

**Input:** Implementor handoff + latest approved `execution_spec` (acceptance criteria, business rules, edge cases, non-functional requirements).

**Output (handoff to Reviewer):**

```yaml
task_id: "T-EXPORT-001"
agent: "tester"
version: 1

output:
  tests_added:
    - api/handlers/customer_export_test.go
    - internal/customer/list_test.go
  covered_scenarios:
    - "Derived from acceptance criteria: invalid status -> 400; no filters -> 200 CSV."
    - "Derived from edge cases: empty result -> header-only CSV."
  uncovered_scenarios:
    - "E2E HTTP flow (out of scope for current test layer)."
  observed_risks:
    - "Large-export performance risk persists from execution_spec.nfr."
  test_matrix:
    unit: "covered"
    integration: "partial"
    e2e: "not covered"
    concurrency: "not applicable"
    rollback/migration: "not applicable"
    performance_smoke: "not covered"
    security_regression: "covered"
  regression_risk_level: "low"

meta:
  eval:
    critical_scenarios_covered: true
    regression_risk_level: "low"
```

---

### Step 5: Reviewer

**Input:** Tester + Implementor outputs and latest approved `execution_spec`.

**Output (final):**

```yaml
task_id: "T-EXPORT-001"
agent: "reviewer"
version: 1

output:
  critical_issues: []
  improvements:
    - "optional_improvement: add max export limit; linked to execution_spec.non_functional_requirements."
  security_findings: []
  performance_findings: []
  finding_type:
    - "optional_improvement"
  merge_readiness: "Ready with follow-ups"
  blocking_reasons: []
  evidence:
    - "Spec refs: acceptance_criteria[invalid status -> 400], business_rules[validation], edge_cases[empty result]."
  recommended_owner: "Implementor"
  required_followups_after_merge:
    - "Track export latency and payload size."

meta:
  eval:
    severity_distribution:
      critical: 0
      major: 0
      minor: 1
    merge_readiness: "Ready with follow-ups"
```

**Orchestrator:** Chain complete. Merge decision = "Ready with follow-ups".

---

## Example 2: Bugfix (regression focus)

**User requirement:**  
"Fix order double-submit race in checkout — sometimes two orders are created for one click."

### Researcher output (excerpt)

```yaml
problem_summary: |
  Checkout submit can be triggered twice (e.g. double click, slow network),
  leading to duplicate orders. Need to make submit idempotent or guarded.

requirements:
  - Prevent duplicate order creation for the same checkout session/request.
  - Preserve existing behavior for legitimate retries (e.g. timeout then retry).

impacted_components:
  - checkout handler or service
  - order creation path
  - possibly payment/cart state

edge_cases:
  - Retry after timeout: should not create a second order if first succeeded.
  - Concurrent requests from same session: only one order.

test_scenarios:
  - Regression: single submit creates one order.
  - Double submit (same idempotency key/session) creates one order.
  - Two different sessions: two orders.
```

### Planner output (excerpt; execution spec)

```yaml
task_id: "T-CHECKOUT-002"
agent: "planner"
version: 1

output:
  execution_spec:
    business_rules:
      - "Same idempotency key must not create a duplicate order."
    acceptance_criteria:
      - "Two concurrent submits with same key create one order."
    rollout_and_rollback:
      - "No schema change required; rollback is removing idempotency guard."
    open_questions: []

meta:
  eval:
    execution_spec_complete: true
```

### Tester (excerpt; derived from execution spec)

```yaml
task_id: "T-CHECKOUT-002"
agent: "tester"
version: 1

output:
  tests_added:
    - "Concurrency: two requests with same idempotency key -> one order."
    - "Regression: single request -> one order."
  covered_scenarios:
    - "acceptance_criteria: one order per idempotency key."
  uncovered_scenarios: []
  observed_risks: []
  test_matrix:
    unit: "covered"
    integration: "covered"
    e2e: "not covered"
    concurrency: "covered"
    rollback/migration: "not applicable"
    performance_smoke: "not covered"
    security_regression: "not applicable"
  regression_risk_level: "low"

meta:
  eval:
    critical_scenarios_covered: true
```

Here the **whole chain** still runs, but each agent emphasizes: Researcher → regression scope and previous behavior; Planner → rollback-safe approach; Tester → regression + concurrency; Reviewer → residual risk and rollback posture.

---

## Example 3: Loopback (Implementor → Planner due to spec gap)

**Scenario:** Implementor starts and finds `execution_spec` says "validate status" but does not define allowed status values or response mapping for unknown values.

**Implementor output (excerpt):**

```yaml
task_id: "T-EXPORT-003"
agent: "implementor"
version: 1

output:
  changed_files: []
  implemented_rules: []
  known_limitations:
    - "Stopped: execution_spec ambiguous for status allowlist and error contract."
  areas_needing_tests: []
  integration_points: []
  tests_required_before_review: true
  migrations_applied: none
  config_changes: none
  feature_flags_used: none
  clarification_request:
    blocker: "Spec gap"
    details:
      - "execution_spec.business_rules missing explicit status enum."
      - "execution_spec.acceptance_criteria missing expected error payload for invalid status."
    requested_from: "planner"

meta:
  eval:
    halted_due_to_spec_ambiguity: true
```

**Loopback decision (Orchestrator):**  
Implementor → Planner. **Blocking issue:** `execution_spec` incomplete/ambiguous. **Acceptance condition:** Planner republishes approved `execution_spec` with explicit status allowlist and invalid-status response contract.

**Planner rerun (excerpt):**

```yaml
task_id: "T-EXPORT-003"
agent: "planner"
version: 2

output:
  execution_spec:
    business_rules:
      - "status allowlist: active|inactive|all"
    acceptance_criteria:
      - "status=foo returns 400 with code=invalid_status"
    open_questions: []

meta:
  eval:
    open_questions_blocking: false
```

Then Implementor resumes, followed by Tester (tests from acceptance criteria) and Reviewer (spec compliance check).

---

## How to use this in practice

| You want to… | Do this |
|--------------|--------|
| Run the pipeline on a new requirement | Send requirement (+ optional category) to Orchestrator; it runs Researcher → … → Reviewer and returns merge readiness. |
| Implement a single role (e.g. as a human or bot) | Read the right `*-agent.md`; take the **Input Contract** from the previous step; produce the **Output Contract** (same keys as in examples). |
| Simulate handoffs | Use the YAML shapes above as templates; replace with your requirement and repo. |
| Trigger a loopback | When an agent’s output has a blocking issue (e.g. "critical scenario untested"), Orchestrator sends back to the prior agent with **blocking issue** + **acceptance condition**; run that agent again, then continue the chain. |

For full contracts and rules, see each `agents/*-agent.md` and `orchestrator-agent.md`.

---

## Example 4: Package usage tracking for job postings

**User requirement:**  
"Track package usage for recruiter job postings. Show purchased/used/remaining counts, increment used on job post creation, auto-calculate remaining, and allow manual used-count adjustment via modal with hard limits."

### Step 0: Orchestrator

- **Input:** requirement above, `category` = `new-feature`.
- **Action:** Route to full chain with data-change awareness (counter persistence + update API).

---

### Step 1: Researcher

**Output (handoff to Planner, excerpt):**

```yaml
task_id: "T-PACKAGE-USAGE-001"
agent: "researcher"
version: 1

output:
  problem_summary: |
    Recruiters buy packages with service quotas, but current usage tracking only
    covers job posting usage partially and does not expose purchased/used/remaining
    consistently. Need reliable counters and safe manual correction.

  requirements:
    - "Expose purchased_count, used_count, remaining_count for package services."
    - "On recruiter job-post creation, increment used_count for service=job_posting."
    - "remaining_count is derived as purchased_count - used_count."
    - "Recruiter can manually adjust used_count through modal action."

  business_constraints:
    - "used_count must be <= purchased_count."
    - "remaining_count must never be negative."
    - "used_count must never be negative."

  impacted_components:
    - "package subscription domain model"
    - "job-post creation use case"
    - "recruiter package usage API"
    - "recruiter UI package usage modal"

  edge_cases:
    - "Concurrent job creations on same package should not over-consume."
    - "Manual adjustment over purchased_count should be rejected with 400."
    - "Adjustment below 0 should be rejected with 400."

  risks:
    - description: "Race conditions can make used_count inconsistent without row locking/atomic update."
      severity: "high"
      area: "correctness"
    - description: "Legacy data may have invalid counters; migration/backfill needed."
      severity: "medium"
      area: "data-quality"

  open_questions: []

meta:
  eval:
    requirement_completeness_score: 0.96
    risk_coverage_score: 0.9
    open_questions_blocking: false
```

---

### Step 2: Planner

**Output (handoff to Implementor, execution spec excerpt):**

```yaml
task_id: "T-PACKAGE-USAGE-001"
agent: "planner"
version: 1

output:
  execution_spec:
    overview: "Add package service usage tracking with hard constraints and recruiter manual adjustment."
    scope:
      in:
        - "Persist purchased_count and used_count per recruiter package service"
        - "Auto-increment used_count on job-post creation"
        - "Read model exposing purchased/used/remaining"
        - "Manual used_count adjustment endpoint for recruiter modal"
      out:
        - "Pricing/package purchase flow redesign"
        - "Non-job-posting service fulfillment workflows"
    functional_requirements:
      - "GET package usage returns purchased_count, used_count, remaining_count."
      - "POST job creation consumes 1 usage unit for job_posting service."
      - "PATCH usage adjustment updates used_count when constraints pass."
    business_rules:
      - "remaining_count = purchased_count - used_count (derived, not stored)."
      - "used_count must be between 0 and purchased_count inclusive."
      - "If package has no remaining units, job creation returns 409 insufficient_quota."
      - "Manual adjustment above purchased_count or below 0 returns 400 validation error."
    data_model:
      table: "recruiter_package_services"
      columns:
        - "id"
        - "recruiter_id"
        - "package_id"
        - "service_code"               # e.g. job_posting, featured_job, branding
        - "purchased_count"            # integer, >= 0
        - "used_count"                 # integer, >= 0, <= purchased_count
        - "updated_at"
    acceptance_criteria:
      - "Given purchased=10 and used=3, API shows remaining=7."
      - "Creating job post increments used from 3 -> 4 and remaining 7 -> 6."
      - "If used==purchased, job post creation returns 409 and does not create job."
      - "Modal adjustment to 8 (<= purchased) succeeds and remaining recalculates."
      - "Modal adjustment to 11 when purchased=10 fails with 400."
    test_scenarios:
      - "Happy path: usage listing and job-post consumption."
      - "Boundary: used == purchased."
      - "Validation: manual adjust below 0 and above purchased."
      - "Concurrency: parallel job creation requests do not oversubscribe."
    non_functional_requirements:
      consistency: "strong for counter updates (single transaction + row lock/atomic update)"
      observability:
        tracing: true
        logs:
          - "usage.consume.success"
          - "usage.consume.rejected_no_remaining"
          - "usage.adjust.rejected_validation"
      latency_p95_ms: 300
    rollout_and_rollback:
      rollout:
        - "DB migration adds purchased_count/used_count with defaults."
        - "Backfill existing package rows."
        - "Enable recruiter UI modal after API deploy."
      rollback:
        - "Disable adjustment endpoint behind feature flag."
        - "Keep migration (additive), stop calling consumption logic."
    open_questions: []
    implementation_plan:
      - "Add domain service: ConsumeUsage(recruiterID, serviceCode, units=1)."
      - "Wrap job-post creation + usage consumption in one transaction."
      - "Add DTO/view model to return purchased/used/remaining."
      - "Add manual adjust command with validation."
      - "Add unit + integration tests for constraints and concurrency."

meta:
  eval:
    execution_spec_complete: true
    rollback_strategy_present: true
    non_functional_constraints_covered: true
```

---

### Step 3: Implementor

**Output (handoff to Tester, excerpt):**

```yaml
task_id: "T-PACKAGE-USAGE-001"
agent: "implementor"
version: 1

output:
  changed_files:
    - "internal/packageusage/service.go"
    - "internal/packageusage/repository.go"
    - "internal/jobposting/create_usecase.go"
    - "api/handlers/recruiter_package_usage_handler.go"
    - "api/handlers/recruiter_package_usage_adjust_handler.go"
    - "db/migrations/20260331_add_recruiter_package_usage.sql"
  implemented_rules:
    - "remaining computed in read model as purchased-used."
    - "job create path consumes usage in same transaction."
    - "manual adjustment validates [0..purchased_count]."
  known_limitations:
    - "Admin bulk correction endpoint not included in this scope."
  areas_needing_tests:
    - "Concurrent consume requests on one package row."
  integration_points:
    - "GET /api/recruiter/packages/:packageId/usage"
    - "PATCH /api/recruiter/packages/:packageId/usage/job-posting"
    - "POST /api/recruiter/jobs"
  tests_required_before_review: true

meta:
  eval:
    changed_files_aligned_with_execution_spec: true
```

---

### Step 4: Tester

**Output (handoff to Reviewer, excerpt):**

```yaml
task_id: "T-PACKAGE-USAGE-001"
agent: "tester"
version: 1

output:
  tests_added:
    - "internal/packageusage/service_test.go"
    - "internal/jobposting/create_usecase_test.go"
    - "api/handlers/recruiter_package_usage_adjust_handler_test.go"
  covered_scenarios:
    - "consume increments used_count by 1 when remaining > 0"
    - "consume rejected when used_count == purchased_count"
    - "manual adjustment within range succeeds"
    - "manual adjustment out of range fails with 400"
    - "remaining derived correctly after each update"
    - "parallel consume does not exceed purchased_count"
  uncovered_scenarios:
    - "cross-service package reporting page E2E"
  observed_risks:
    - description: "Need production monitoring on usage.consume.rejected_no_remaining spikes."
      severity: "low"
  test_matrix:
    unit: "covered"
    integration: "covered"
    e2e: "partial"
    concurrency: "covered"
    rollback/migration: "partial"
    performance_smoke: "covered"
    security_regression: "covered"
  regression_risk_level: "low"
```

---

### Step 5: Reviewer

**Output (final, excerpt):**

```yaml
task_id: "T-PACKAGE-USAGE-001"
agent: "reviewer"
version: 1

output:
  critical_issues: []
  improvements:
    - "Add audit trail table for manual used_count adjustments (who/when/reason)."
  security_findings: []
  performance_findings: []
  merge_readiness: "Ready with follow-ups"
  blocking_reasons: []
  evidence:
    - "Constraint tests verify used_count <= purchased_count and remaining >= 0."
    - "Job creation integration test confirms consumption and rollback behavior."
  recommended_owner:
    - "implementor"
  required_followups_after_merge:
    - "Add analytics dashboard for package usage burn-down per recruiter."
```

---

## Example 5: Planner execution spec with optional code illustrations

**User requirement:**  
"After a subscription renewal succeeds, emit a `subscription.renewed` domain event so analytics and search indexers can react. Duplicates for the same billing period must not produce duplicate downstream effects."

This example shows a **Planner-only** handoff shape: the **written** `execution_spec` sections are the downstream contract; `technical_illustrations` is optional, explanatory, and not a substitute for explicit requirements.

**Output (handoff to Implementor, full artifact envelope):**

```yaml
task_id: "T-SUB-RENEW-EVT-001"
agent: "planner"
version: 1

output:
  execution_spec:
    overview: "Publish a renewal event after successful subscription renewal with idempotency per renewal attempt."
    scope:
      in: ["renewal success path", "event publish to internal broker", "idempotency by renewal_id"]
      out: ["payment processor UI", "customer email notifications"]
    functional_requirements:
      - "On successful renewal transaction completion, publish subscription.renewed including subscription_id, renewal_id, and renewed_at."
      - "Publishing uses the platform event bus; consumers are out of scope except as noted in scope."
    business_rules:
      - "The same renewal_id must not produce more than one subscription.renewed emission (idempotent publisher)."
      - "renewed_at must reflect the billing-system completion timestamp, not client clock."
    edge_cases:
      - "Broker temporarily unavailable: publisher retries with backoff; no silent drop of a successful renewal."
      - "Replay of the same renewal_id after success: no second event."
    acceptance_criteria:
      - "Given renewal_id R1 applied once, exactly one subscription.renewed with renewal_id R1 is observed on the bus."
      - "Given two different renewal_ids for the same subscription in different periods, two distinct events are emitted."
    test_scenarios:
      - "Happy path: renewal completes, one event emitted."
      - "Duplicate publish attempt for same renewal_id: second is a no-op."
      - "Broker failure: event appears after retry without duplicate side effects."
    non_functional_requirements:
      - "Emit within 5s p95 of renewal commit; structured logs with renewal_id and subscription_id."
    rollout_and_rollback:
      - "Feature-flag the publisher; rollback disables flag (no schema removal in v1)."
    open_questions: []
    implementation_plan:
      - "Add outbox or idempotency store keyed by renewal_id before publish."
      - "Wire renewal success handler to publisher."
      - "Add integration test: single emission + retry semantics."

    technical_illustrations:
      - purpose: "Example payload shape for integrators (not a schema registry contract)."
        snippet_type: "event_payload_example"
        snippet: |
          {
            "type": "subscription.renewed",
            "renewal_id": "ren_01H...",
            "subscription_id": "sub_01H...",
            "renewed_at": "2026-04-02T12:00:00Z"
          }
        notes: "Illustrative JSON only. Binding field requirements and idempotency rules are in functional_requirements and business_rules above."
        constraints: "Implementor must not treat keys or formatting as authoritative without matching written spec."

      - purpose: "Pseudocode sketch for idempotent publish (not production logic)."
        snippet_type: "pseudocode"
        snippet: |
          on renewal_success(renewal_id):
            if already_emitted(renewal_id): return ok
            mark_pending_emit(renewal_id)
            publish(build_event(renewal_id))  // see business_rules for duplicate rules
            mark_emitted(renewal_id)
        notes: "Explanatory flow only; transaction boundaries and storage belong to implementation_plan and repo standards."
        constraints: "If this sketch conflicts with written edge_cases or business_rules, the written spec wins."

meta:
  eval:
    execution_spec_complete: true
    rollback_strategy_present: true
    non_functional_constraints_covered: true
```
