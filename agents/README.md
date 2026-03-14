# Agent specs

Pipeline role definitions for Researcher, Planner, Implementor, Tester, Reviewer, and Orchestrator. Each `*-agent.md` is the source of truth for that role.

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
| `researcher-agent.md` | Parse spec → requirements, impacted components, risks |
| `planner-agent.md` | Research → API/DB plan, implementation steps |
| `implementor-agent.md` | Plan → code + change report |
| `tester-agent.md` | Implementor output → tests, coverage, risks |
| `reviewer-agent.md` | Tester output → findings, merge readiness |
| `orchestrator-agent.md` | Routing, categories, loopbacks, quality gates |
| `skills-map.md` | Agent → skills mapping (e.g. `analyze-requirement`, `implement-go-handler`) |
| `examples.md` | Full pipeline walkthroughs |

Root entrypoint for agents: repo root `AGENTS.md`.

**Examples:** See [examples.md](examples.md) for full pipeline walkthroughs (new feature, bugfix, and a loopback).
