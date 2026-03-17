## Agent instructions

This repo defines a **multi-agent pipeline** (requirements → research → plan → implementation → test → review). Use these instructions when working as or with these agents. Build, test, and code-style rules live in the codebase where the pipeline runs; role contracts and handoffs are in `agents/`.

### Where to look

- **Role specs:** `agents/*-agent.md` — one file per role (Researcher, Planner, Implementor, Tester, Reviewer). Each defines responsibilities, contracts, and guardrails.
- **Orchestrator:** `agents/orchestrator-agent.md` — handoff order, loopbacks, routing, categories, validation examples.
- **Skills map:** `agents/skills-map.md` — which skills each agent uses (e.g. `implement-go-handler`, `write-unit-test-go`).
- **Examples:** `agents/examples.md` — full pipeline walkthroughs (new feature, bugfix, loopback).
- **Agent specs meta:** `agents/README.md` — standards, verification checklist, file map.
- **State model:** `agents/state-model.md` — shared state, versioning, blocking issues.
- **Evals:** `agents/evals.md` — role-level eval rubrics and metrics.
- **Observability:** `agents/observability.md` — traces, logs, run metadata.

### Runtime model

- **Entry point:** Orchestrator agent. Input is a single `task` object with:
  - `task_id` (string, stable across retries)
  - `requirement` (raw text)
  - `category` (optional; see orchestrator categories)
  - `context` (optional implementation context, e.g. repo path, stack)
  - `state` (shared state object; see **State model**)
- **Invocation shape:** All role agents receive a **JSON/YAML object**, not free-form text. The runner MUST:
  - Wrap inputs under a top-level `task` key.
  - Attach `state` from previous steps.
  - Attach previous `handoff_outputs` when present.
- **Agent outputs:** Must be valid **YAML or JSON** with:
  - Top-level `task_id`
  - Top-level `agent` (one of `researcher|planner|implementor|tester|reviewer|orchestrator`)
  - Top-level `version` (monotonic per `task_id`)
  - `output` object that matches the role’s Output Contract keys.
  - Optional `meta` object (timings, token usage, tool calls, etc.).
- **Validation:** The runner is responsible for schema validation:
  - Reject outputs that are not parseable YAML/JSON.
  - Reject outputs that miss required keys.
  - Auto-reprompt **once** with a short, schema-focused correction; on second failure, escalate to human.

### State model

Multi-agent runs share a small, explicit state object to avoid stale handoffs:

- **Required fields:**
  - `task_id` — unique per orchestrated run.
  - `requirement_version` — bumped when requirement/clarifications change.
  - `research_version` — bumped when Researcher re-runs.
  - `plan_version` — bumped when Planner re-runs.
  - `implementation_version` — bumped when Implementor re-runs.
  - `test_version` — bumped when Tester re-runs.
  - `review_version` — bumped when Reviewer re-runs.
  - `current_status` — enum; e.g. `researching|planning|implementing|testing|reviewing|blocked|complete`.
  - `blocking_issues` — list of current blocking issues (source, description, acceptance_condition).
  - `acceptance_conditions` — list of conditions to clear blocks and advance.
- **State ownership:**
  - Orchestrator is the **source of truth** for `current_status`, version fields, and `blocking_issues`.
  - Role agents may propose updates (e.g. new blocking issue) under their `output`; orchestrator applies them.
  - State is immutable per version; each update creates a new `state_version`.

### Eval and success metrics

For production use, each role should be evaluated with simple, explicit rubrics:

- **Researcher**
  - `requirement_completeness_score` (0–1)
  - `risk_coverage_score` (0–1)
  - `open_questions_blocking` (bool)
- **Planner**
  - `backward_compatibility_checked` (bool)
  - `rollback_strategy_present` (bool)
  - `non_functional_constraints_covered` (bool; latency, observability, security, scalability where relevant)
- **Implementor**
  - `changed_files_aligned_with_plan` (bool)
  - `tests_required_before_review` (bool)
- **Tester**
  - `critical_scenarios_covered` (bool)
  - `regression_risk_level` (`low|medium|high`)
- **Reviewer**
  - `severity_distribution` (count by severity)
  - `merge_readiness` (`Not ready` \| `Ready with follow-ups` \| `Ready`)

These can be stored in `output.meta.eval` per agent for downstream dashboards or offline evals.

### Observability and tracing

- **Trace unit:** One trace per `task_id`, spanning all agents and tool calls.
- **Per-agent span attributes:**
  - `agent.name`, `agent.role`
  - `task.id`, `task.category`
  - `state.version.*` (requirement/plan/implementation/etc.)
  - `token.usage.prompt`, `token.usage.completion` (if available)
  - `tools.used` (list of tool names)
  - `retry.count`
- **Logging:**
  - Structured JSON logs with `task_id`, `agent`, `step`, `severity`, `message`.
  - Log at least: invocation, validation failure, loopback decision, manual escalation.

### Tool and cost governance

- **Tool exposure:**
  - Each agent may only use the skills/tools listed in `agents/skills-map.md`, plus any explicitly configured in the runtime.
  - Tools should be tagged as `read_only` or `write` and the orchestrator/runner must enforce write access only where needed (typically Implementor and some Tester flows).
- **Per-step budgets (suggested defaults):**
  - `max_tokens` per agent call.
  - `max_retries` (default 1 auto-reprompt; beyond that, human-in-the-loop).
  - `max_external_calls` (per agent, per task).
  - `max_files_changed` (Implementor) and `max_files_read` (Researcher/Planner/Tester).

### Ambiguity and human-in-the-loop

- **Ambiguity threshold:**
  - If a role agent has blocking open questions that prevent a safe plan or implementation, it MUST:
    - Populate `open_questions` (Researcher/Planner) or `blocking_issues` (Tester/Reviewer).
    - Mark `current_status = blocked`.
    - Return control to Orchestrator to either loop back or escalate.
- **Human approval required when:**
  - Destructive DB migrations, irreversible data changes.
  - Introduction of new external integrations with side effects.
  - Addition of new runtime dependencies/packages in critical services.
  - Orchestrator should emit a `manual_handoff` artifact in these cases and stop automated delegation.

### Handoff order

Researcher → Planner → Implementor → Tester → Reviewer. Loopbacks and quality gates are in the orchestrator; no step skips required outputs.

### When stuck

- Ask a clarifying question, propose a short plan, or open a draft with notes.
- Do not push large speculative changes without confirmation.

### Safety

- Follow each role’s **Guardrails** and **Output Contract**; do not advance without required keys.
- Out-of-scope work (e.g. infra redesign, cross-system architecture) → manual handoff per orchestrator.
