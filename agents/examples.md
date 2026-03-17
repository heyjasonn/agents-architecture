# Multi-agent pipeline examples

Concrete examples of how a requirement flows through Researcher → Planner → Implementor → Tester → Reviewer. Use these to see the **shape of each handoff** and when **loopbacks** happen.

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
architecture_overview: |
  New GET handler under existing API. Handler parses query, calls domain
  ListWithFilters, domain returns rows, handler writes CSV to response.

request_flow:
  - Parse query (date_from, date_to, status).
  - Validate (dates parseable, status in allowlist).
  - Call customer.ListWithFilters(ctx, filters).
  - Write CSV header + rows to http.ResponseWriter.
  - On error: return 400/500 as appropriate.

api_contract:
  GET /api/customers/export
  Query: date_from (optional), date_to (optional), status (optional: active|inactive|all)
  Response: 200 CSV (Content-Type: text/csv), 400 on validation error, 500 on internal error.

db_changes: none

implementation_steps:
  1. Add ListWithFilters to customer service (repo + use case if present).
  2. Add export handler in api/handlers; wire route.
  3. Validate query params; map to filters; call ListWithFilters; write CSV.

constraints:
  - Reuse existing auth/middleware for the API.
  - No new tables; use existing customer table.

assumptions:
  - Customer model has created_at and status (or equivalent).
```

---

### Step 3: Implementor

**Input:** Planner output above + codebase (actual files, existing handlers, customer repo interface).

**Output (handoff to Tester):**

```yaml
changed_files:
  - internal/customer/list.go          # ListWithFilters
  - api/handlers/customer_export.go    # new handler
  - api/routes.go                     # register GET /api/customers/export

implemented_rules:
  - Handler only does HTTP; logic in internal/customer.
  - Reused existing validation pattern from other handlers.

known_limitations:
  - No pagination; large result sets could be slow (noted in Researcher risks).

areas_needing_tests:
  - Handler: valid filters, no filters, empty result, invalid params.
  - ListWithFilters: filter logic unit tests.

integration_points:
  - HTTP client calls GET /api/customers/export (for E2E if applicable).
```

*(Plus the actual code changes.)*

---

### Step 4: Tester

**Input:** Implementor handoff + plan + requirement intent.

**Output (handoff to Reviewer):**

```yaml
tests_added:
  - api/handlers/customer_export_test.go  # table-driven: valid, empty, invalid params
  - internal/customer/list_test.go        # ListWithFilters with various filters

covered_scenarios:
  - Happy path with filters; no filters; empty result; invalid date_from; invalid status.

uncovered_scenarios:
  - E2E from HTTP (if not in scope for this repo).

observed_risks:
  - None new; large-export risk already documented.
```

---

### Step 5: Reviewer

**Input:** Tester output + implementor/planner outputs for traceability.

**Output (final):**

```yaml
critical_issues: []

improvements:
  - Consider adding a max-limit (e.g. 10k rows) and return 400 if exceeded.

security_findings: []
performance_findings: []

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

### Planner output (excerpt)

```yaml
# Focus: rollback-safe, minimal-risk
request_flow:
  - Before creating order: check idempotency key (e.g. checkout_session_id + intent).
  - If key already used and order exists: return existing order (idempotent).
  - If key used but order failed: allow retry (or explicit failure).
  - Create order only when key not seen or retry allowed.

# Rollback: no schema change required; additive idempotency check. If we add
# a table for keys, make it additive; migration reversible.
```

### Tester (excerpt)

```yaml
tests_added:
  - Concurrency test: two requests with same idempotency key → one order.
  - Regression: single request → one order.
  - Different keys → two orders.

observed_risks:
  - None; rollback = remove check and redeploy (no DB rollback needed if no new table).
```

Here the **whole chain** still runs, but each agent emphasizes: Researcher → regression scope and previous behavior; Planner → rollback-safe approach; Tester → regression + concurrency; Reviewer → residual risk and rollback posture.

---

## Example 3: Loopback (Tester → Implementor)

**Scenario:** Tester finds that "invalid status value" returns 500 instead of 400, and the plan said "400 on validation error".

**Tester output (excerpt):**

```yaml
uncovered_scenarios:
  - Invalid status (e.g. status=foo): currently 500; requirement and plan say 400.

observed_risks:
  - Validation gap: status enum not validated in handler.
```

**Loopback decision (Orchestrator):**  
Tester → Implementor. **Blocking issue:** Critical scenario (invalid status) untested/incorrect. **Acceptance condition:** Handler returns 400 for invalid `status` and test added.

**Implementor:** Adds validation and test; produces new handoff. Tester runs again; then Reviewer.

---

## How to use this in practice

| You want to… | Do this |
|--------------|--------|
| Run the pipeline on a new requirement | Send requirement (+ optional category) to Orchestrator; it runs Researcher → … → Reviewer and returns merge readiness. |
| Implement a single role (e.g. as a human or bot) | Read the right `*-agent.md`; take the **Input Contract** from the previous step; produce the **Output Contract** (same keys as in examples). |
| Simulate handoffs | Use the YAML shapes above as templates; replace with your requirement and repo. |
| Trigger a loopback | When an agent’s output has a blocking issue (e.g. "critical scenario untested"), Orchestrator sends back to the prior agent with **blocking issue** + **acceptance condition**; run that agent again, then continue the chain. |

For full contracts and rules, see each `agents/*-agent.md` and `orchestrator-agent.md`.
