# Agent specs

Pipeline role definitions for Researcher, Planner, Implementor, Tester, Reviewer, and Orchestrator. Planner produces the execution spec (optional `technical_illustrations` / `implementation_sketches` are explanatory only); downstream agents execute against the latest approved execution spec and its explicit written sections.

## Standards used

- **AGENTS.md (Agentic AI Foundation / Linux Foundation):** Plain Markdown, under ~150 lines per file, actionable instructions, no required schema.
- **Best practices (agentsmd.io, CodingAgents.md):** Clear Do/Don't guardrails, explicit input/output contracts, "when stuck" escape hatch, safety boundaries, concrete examples.
- **Consistency:** Role agents share the same section order: **Skills** → Responsibilities → Input Contract → Output Contract (table) → Guardrails → Exit Criteria. Frontmatter: `name`, `description` (third-person, when to use). Skills link to [skills-map.md](skills-map.md).

## Verification checklist

Use this when adding or editing agent specs. Standards: AGENTS.md (Agentic AI Foundation), agentsmd.io, ASDLC — under ~150 lines, command-first, explicit contracts, no vague directives.

| Check | Role agents | Orchestrator | Root AGENTS.md |
|-------|-------------|--------------|----------------|
| Under ~150 lines | ✓ | ✓ | ✓ |
| Frontmatter `name` + `description` | ✓ | ✓ | n/a |
| Skills section (links to skills-map) | ✓ | n/a | n/a |
| Explicit output keys (table) | ✓ | ✓ | n/a |
| Guardrails as Do/Don't (no vague "be careful") | ✓ | ✓ | ✓ |
| Exit criteria stated | ✓ | ✓ | n/a |
| Handoff order / loopbacks defined | n/a | ✓ | ✓ (points to orchestrator) |
| "When stuck" or escape hatch | n/a | n/a | ✓ |
| No contradictory priorities | ✓ | ✓ | ✓ |

## File map

| File | Purpose |
|------|--------|
| `researcher-agent.md` | Clarify requirement → constraints, impacted components, risks |
| `planner-agent.md` | Research output → execution spec (optional lightweight code illustrations per planner spec) |
| `implementor-agent.md` | Execution spec → code + implementation result |
| `tester-agent.md` | Execution spec + implementation result → test result, risks |
| `reviewer-agent.md` | Execution spec + test result → review findings, merge readiness |
| `orchestrator-agent.md` | Routing, categories, loopbacks, quality gates |
| `skills-map.md` | Agent → skills mapping (e.g. `analyze-requirement`, `implement-go-handler`) |
| `examples.md` | Full pipeline walkthroughs |

Root entrypoint for agents: repo root `AGENTS.md`.

**Examples:** See [examples.md](examples.md) for full pipeline walkthroughs (new feature, bugfix, loopback, and an execution spec with optional `technical_illustrations`).
