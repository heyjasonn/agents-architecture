## Observability and tracing for agents

This file defines the observability model for the multi-agent pipeline: traces, logs, and run metadata. Use it as the source of truth when instrumenting a runner or dashboard.

---

### 1. Trace model

- **Trace unit:** One trace per `task_id`.
- **Spans:**
  - Orchestrator span (root or near-root).
  - One span per agent invocation (Researcher, Planner, Implementor, Tester, Reviewer).
  - Nested spans for tool calls as needed.

Recommended span attributes (OpenTelemetry-style naming):

```yaml
trace:
  attributes:
    task.id: "<task_id>"
    task.category: "<category>"
    task.state.version: <int>

    agent.name: "<orchestrator|researcher|planner|implementor|tester|reviewer>"
    agent.version: <int>

    state.requirement_version: <int>
    state.research_version: <int>
    state.plan_version: <int>
    state.implementation_version: <int>
    state.test_version: <int>
    state.review_version: <int>

    tools.used: ["<tool1>", "<tool2>"]

    retry.count: <int>

    token.usage.prompt: <int>
    token.usage.completion: <int>
```

---

### 2. Logging model

- **Format:** Structured JSON logs.
- **Minimum fields:**

```json
{
  "timestamp": "2026-03-17T12:34:56Z",
  "level": "info",
  "task_id": "T-123",
  "agent": "planner",
  "event": "agent.invoked",
  "state_version": 3,
  "plan_version": 2,
  "message": "Planner invoked for task T-123"
}
```

Key events to log:

- `agent.invoked` — when a role is called.
- `agent.completed` — with summary of success/failure and key evals.
- `schema.validation_failed` — include reason and offending snippet (if safe).
- `loopback.triggered` — include from/to agents and blocking issue.
- `manual_handoff` — include reason and suggested owner.

---

### 3. Run metadata and cost tracking

Per agent invocation, record:

```yaml
run_meta:
  task_id: "<task_id>"
  agent: "<name>"
  version: <int>

  timing_ms:
    total: 1234
    model: 1100
    tools: 100

  tokens:
    prompt: 1200
    completion: 350

  tools_used:
    - name: "user-postgres.query"
      count: 2
    - name: "cursor-ide-browser.browser_snapshot"
      count: 1

  retries: 0
```

You can either:

- Embed this under `meta` in artifacts (`meta.timing_ms`, `meta.tokens`, `meta.tools_used`, `meta.retries`), and/or
- Store it separately in a run metadata store.

---

### 4. Dashboards and alerts (examples)

- **Dashboards:**
  - Token usage per `task_id` and per role.
  - Average `timing_ms.total` per role.
  - Count of `schema.validation_failed` events over time.
  - Distribution of Reviewer `merge_readiness`.

- **Alerts:**
  - High rate of `schema.validation_failed`.
  - Repeated loopbacks for the same `task_id` (potentially flapping between roles).
  - Sudden spike in `tokens.prompt` or `tokens.completion` per run.

---

### 5. Context pruning vs observability

The pipeline follows two principles:

- **Do not** push full history or logs into role prompts.
- **Do** push rich history into observability systems.

Practically:

- Handoffs pass **only**:
  - The `output` sections from previous roles.
  - Minimal additional context (e.g. file paths, API paths).
- Traces/logs store:
  - Full artifacts (if size/sensitivity allows).
  - Prompt and completion summaries or hashes (for privacy).
  - All model/tool timing and token metrics.

This keeps prompts small and predictable while still enabling deep debugging when needed.

