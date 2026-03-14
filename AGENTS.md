# Agent instructions

This repo defines a **multi-agent pipeline** (requirements → research → plan → implementation → test → review). Use these instructions when working as or with these agents. Build, test, and code-style rules live in the codebase where the pipeline runs; role contracts and handoffs are in `agents/`.

## Where to look

- **Role specs:** `agents/*-agent.md` — one file per role (Researcher, Planner, Implementor, Tester, Reviewer). Each defines responsibilities, contracts, and guardrails.
- **Orchestrator:** `agents/orchestrator-agent.md` — handoff order, loopbacks, routing, categories, validation examples.
- **Skills map:** `agents/skills-map.md` — which skills each agent uses (e.g. `implement-go-handler`, `write-unit-test-go`).
- **Examples:** `agents/examples.md` — full pipeline walkthroughs (new feature, bugfix, loopback).

## Handoff order

Researcher → Planner → Implementor → Tester → Reviewer. Loopbacks and quality gates are in the orchestrator; no step skips required outputs.

## When stuck

- Ask a clarifying question, propose a short plan, or open a draft with notes.
- Do not push large speculative changes without confirmation.

## Safety

- Follow each role’s **Guardrails** and **Output Contract**; do not advance without required keys.
- Out-of-scope work (e.g. infra redesign, cross-system architecture) → manual handoff per orchestrator.
