## State and memory model

This file defines the shared state/memory model used by the multi-agent pipeline. Treat this as the **source of truth** for how long-running tasks are tracked and resumed.

State is intentionally small and versioned; role outputs (research artifact, execution spec, implementation result, test result, review result) hold the detailed content.

---

### 1. State object shape

`state` is a single object attached to each orchestrator invocation:

```yaml
state:
  version: 1

  requirement_version: 1
  research_version: 0
  execution_spec_version: 0
  implementation_version: 0
  test_version: 0
  review_version: 0

  current_status: "researching"   # researching|planning|implementing|testing|reviewing|blocked|complete

  blocking_issues:
    - from_agent: "tester"
      description: "Invalid status returns 500, execution spec says 400"
      acceptance_condition: "Handler returns 400 with test covering invalid status"

  acceptance_conditions:
    - "All critical_issues resolved"
    - "Regression risk <= medium"
```

---

### 2. Versioning rules

- `state.version` MUST:
  - Start at `1` for a new `task_id`.
  - Increment by `1` on any change to the `state` object.
- `*_version` fields:
  - `requirement_version`: increment when the requirement or clarifications change.
  - `research_version`: increment after a successful research artifact is accepted.
  - `execution_spec_version`: increment after a successful Planner `execution_spec` is accepted.
  - `implementation_version`: increment after a successful implementation result is accepted.
  - `test_version`: increment after a successful test result is accepted.
  - `review_version`: increment after a successful review result is accepted.

These versions are used to detect stale role outputs (e.g. a test result that references an `execution_spec_version` that is no longer current).

---

### 3. Status model

`current_status` describes the high-level lifecycle of the task:

- `researching`: Researcher is producing or revising requirements.
- `planning`: Planner is producing or revising the `execution_spec`.
- `implementing`: Implementor is changing code.
- `testing`: Tester is adding/running tests.
- `reviewing`: Reviewer is assessing merge readiness.
- `blocked`: Progress is blocked by one or more `blocking_issues`.
- `complete`: Pipeline is finished; Reviewer merge_readiness is terminal.

The orchestrator is the **only writer** to `current_status`. Role agents may suggest changes via their outputs (e.g. by adding blocking issues).

---

### 4. Blocking issues and acceptance conditions

`blocking_issues` is the canonical place to record why the pipeline is blocked:

```yaml
blocking_issues:
  - from_agent: "tester"
    description: "Invalid status returns 500 instead of 400"
    acceptance_condition: "Handler returns 400 for invalid status and test added"
```

Rules:

- A role that detects a **blocking problem** (e.g. Tester, Reviewer) should:
  - Add a new item to `blocking_issues` in its `output` proposal.
  - Propose an `acceptance_condition` that, once satisfied, clears the block.
- The orchestrator:
  - Applies the proposed blocking issues to `state.blocking_issues`.
  - Sets `current_status = "blocked"` when there is at least one open blocking issue.
  - Clears specific blocking issues when their acceptance conditions are met (e.g. after a loopback fix).

`acceptance_conditions` in `state` may also include **high-level gates** (e.g. “All critical_issues resolved”), not only per-issue conditions.

---

### 5. Task identity and resumption

Every orchestrated run is identified by:

- `task_id`: Stable identifier for the task (e.g. ticket ID, hash, UUID).
- `(task_id, agent, version)`: Uniquely identifies a specific artifact version.
- `(task_id, agent, version)`: Uniquely identifies a specific role output version.

Resuming a task means:

- Reloading:
  - Latest `state` for `task_id`.
  - Latest outputs for each role (by `version`).
- Invoking the orchestrator again with:

```yaml
task_id: "<same id>"
requirement: "<possibly updated requirement>"
category: "<same or updated category>"
context: { ... }
state: <latest state>
handoff_outputs:
  researcher: <latest research artifact>
  planner: <latest execution spec artifact>
  implementor: <latest implementation result artifact>
  tester: <latest test result artifact>
  reviewer: <latest review result artifact>
```

The orchestrator then decides whether to:

- Advance to the next role.
- Loop back to a previous role with updated state.
- Escalate to human via `manual_handoff`.

---

### 6. What state is **not**

- State does **not** contain:
  - Full diffs or code content.
  - Entire prompt history.
  - Long-form reasoning logs.
- Those belong in:
  - Role outputs themselves (e.g. detailed `execution_spec`, `risks`).
  - Observability/tracing layers (e.g. logs, traces, spans).

Keeping state small and explicit makes it:

- Easy to serialize and version.
- Easy to debug.
- Safe to pass across systems without leaking large or sensitive payloads.

