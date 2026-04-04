## Evals and review rubric

This file defines how to evaluate the quality of each agent’s output. It provides a simple, explicit rubric that can be used both:

- Online (per-run metrics in `meta.eval`), and
- Offline (batch evaluation on stored artifacts).

All scores/flags below are **advisory**; they do not replace the required output keys in each role spec.

---

### 1. Evaluation model

Each agent may populate a `meta.eval` object alongside its `output`:

```yaml
meta:
  eval:
    # Role-specific keys (see below)
```

Runners can:

- Log these values for dashboards.
- Aggregate them across runs for quality tracking.
- Attach them to traces for debugging.

---

### 2. Researcher rubric

```yaml
meta:
  eval:
    requirement_completeness_score: 0.0-1.0
    risk_coverage_score: 0.0-1.0
    open_questions_blocking: true|false
```

- **`requirement_completeness_score`**
  - 0.9–1.0: Requirements are concrete, testable, and cover main flows + key edge cases.
  - 0.6–0.8: Mostly complete, some gaps but non-blocking.
  - < 0.6: Significant gaps; not ready for Planner.
- **`risk_coverage_score`**
  - Measures how well `risks` cover performance, security, data correctness, and operational risk.
- **`open_questions_blocking`**
  - `true`: Planner should not publish `execution_spec`; orchestrator should loop back or escalate.

---

### 3. Planner rubric

```yaml
meta:
  eval:
    execution_spec_complete: true|false
    rollback_strategy_present: true|false
    non_functional_constraints_covered: true|false
```

- **`execution_spec_complete`**
  - `true`: `execution_spec` includes functional requirements, business rules, edge cases, acceptance criteria, test scenarios, and rollout/rollback.
  - Optional `technical_illustrations` / `implementation_sketches` do not satisfy completeness by themselves; they are supplementary only and must not replace those written sections.
- **`rollback_strategy_present`**
  - `true`: `execution_spec.rollout_and_rollback` is present and realistic.
- **`non_functional_constraints_covered`**
  - `true`: `execution_spec.non_functional_requirements` covers latency, observability, security, and scalability where relevant.

---

### 4. Implementor rubric

```yaml
meta:
  eval:
    changed_files_aligned_with_execution_spec: true|false
```

- **`changed_files_aligned_with_execution_spec`**
  - `true`: `changed_files` correspond to components and constraints described in `execution_spec`.
  - If `false`: orchestrator may send a loopback to Planner or require manual review.

---

### 5. Tester rubric

```yaml
meta:
  eval:
    critical_scenarios_covered: true|false
    regression_risk_level: "low"|"medium"|"high"
```

- **`critical_scenarios_covered`**
  - `true`: All critical flows identified in `execution_spec.acceptance_criteria`, `execution_spec.business_rules`, and `execution_spec.edge_cases` are in `covered_scenarios`.
  - `false`: Orchestrator should treat as a potential block before Reviewer.
- **`regression_risk_level`**
  - `low`: Minimal behavior change or strong regression coverage.
  - `medium`: Some gaps but acceptable for non-critical paths.
  - `high`: Significant gaps; likely not safe to merge without additional work.

---

### 6. Reviewer rubric

```yaml
meta:
  eval:
    severity_distribution:
      critical: <int>
      high: <int>
      medium: <int>
      low: <int>
    merge_readiness: "Not ready"|"Ready with follow-ups"|"Ready"
```

- **`severity_distribution`**
  - Simple histogram of findings by severity; used for dashboards.
- **`merge_readiness`**
  - Mirrors `output.merge_readiness` for easy filtering.

---

### 7. Usage patterns

- **Per-run monitoring**
  - Emit `meta.eval` for each role and push to your metrics store or logs.
  - Build simple panels:
    - Average `requirement_completeness_score` over time.
    - Distribution of `regression_risk_level`.
    - Ratio of `Not ready` vs `Ready` in Reviewer.

- **Batch evaluation**
  - Sample stored artifacts.
  - Re-score or sanity-check `meta.eval` with human raters.
  - Use discrepancies to refine prompts and guardrails.

- **Gating**
  - Evals are **inputs** to gating, not hard rules.
  - For strict environments, you may:
    - Reject merges when `regression_risk_level = "high"`.
    - Require human sign-off when `critical > 0` in `severity_distribution`.

