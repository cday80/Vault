# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two things share this working tree:

1. **An Obsidian vault** (`.obsidian/`). Commits with the message `vault backup: <timestamp>` are produced by Obsidian's Git plugin and are not meaningful change history — ignore them when looking for intent. `.obsidian/workspace.json` churns constantly and almost never warrants a manual commit.
2. **The Coordinated Agent Team pack** (`agents/`). This is a prompt-driven multi-agent system distributed *as source* from here — it is designed to be **copied into other projects** at `.github/agents/`. The pack is documentation/prompts only; there is no build, no tests, no lint, no package manager.

## Path convention — important

The pack lives at **`agents/`** in this repository, but the agent prompts themselves reference **`.github/agents/`** because that is the *destination* path inside a consuming project. When editing an agent file, do not "fix" the `.github/agents/...` references to point at `agents/...` — those paths are correct for the deployed layout, even though they don't resolve here.

Concretely: if you see `.github/agents/DISPATCH-REFERENCE.md` inside `agents/00-orchestrator.md`, that path is intentional. Treat the pack as a redistributable artifact.

## Architecture of the agent pack

The pack is a deterministic state machine implemented as natural-language prompts. The "interpreter" is the LLM; the "runtime state" is files under `.agents-work/<session>/`.

State machine (full mode):
```
INTAKE -> DESIGN -> APPROVE_DESIGN -> PLAN -> REVIEW_STRATEGY -> IMPLEMENT_LOOP -> INTEGRATE -> RELEASE -> DONE
```

Lean mode (low-risk tasks): `INTAKE_LEAN -> IMPLEMENT_LOOP -> INTEGRATE -> DONE`.

Repair / branch states: `ASK_USER`, `FIX_REVIEW`, `FIX_TESTS`, `FIX_SECURITY`, `FIX_BUILD`, `BLOCKED`.

The Orchestrator (`00-orchestrator.md`) is the only agent that touches the state machine. All other agents are single-shot: they receive a JSON task, return a JSON result, and write artifacts. The Orchestrator decides the next transition based on the returned `status` and `gates`.

Agents are split into roles 00–11 (orchestrator, spec, architect, planner, coder, reviewer, qa, security, integrator, docs, designer, researcher). The full roster, default model assignments, and tool configuration are documented in `README.md` and in each agent's YAML frontmatter.

## Files that are load-bearing (read these before changing rules)

These four files form a tight contract. A change to one almost always requires a coordinated change to the others — drift here breaks the whole system.

- `agents/CONTRACT.md` — canonical I/O schema, status enum, artifact requirements, hard gates. Source of truth.
- `agents/WORKFLOW.md` — state transitions, dispatch rules, lean-mode behavior, repair-loop budgets.
- `agents/DISPATCH-REFERENCE.md` — mandatory dispatch template, `context_files` matrix, pre-dispatch validation checklist. The Orchestrator is required to re-read this before every dispatch.
- `agents/00-orchestrator.md` — uses the above; must stay consistent with `DISPATCH-REFERENCE.md`.

**Precedence when guidance conflicts:** `CONTRACT.md` > role specs (`00-…` to `11-…`) > `WORKFLOW.md` / `DISPATCH-REFERENCE.md` > the consuming project's `copilot-instructions.md`.

## Runtime artifacts (in consuming projects, not here)

Sessions live at `.agents-work/YYYY-MM-DD_<slug>/`. Core files: `spec.md`, `acceptance.json`, `tasks.yaml`, `status.json`, `report.md`. Conditional: `architecture.md` (full mode only), `adr/`, `design-specs/`, `research/`, `approve-design-history.jsonl`.

Task status lifecycle: `not-started` → `in-progress` → `implemented` → `completed` (or `blocked`).

`status.json` holds session-level state (`current_state`, retries, decisions, assumptions, known issues); it is the recovery anchor — interrupted sessions resume by re-reading it.

## Hard gates (these block progression)

- Required artifacts missing or invalid
- (full mode) `APPROVE_DESIGN` not approved by user
- (full mode) `REVIEW_STRATEGY` not chosen by user
- Reviewer / QA / Security returns `BLOCKED`
- Cross-task final review (`task.id: meta`) returns `BLOCKED`
- Security `NEEDS_DECISION` pending an `ASK_USER` resolution
- Build/CI red

Repair loops have a 3-attempt budget per task per loop type. `APPROVE_DESIGN` `changes-requested` is intentionally uncapped.

## What "editing this repo" usually means

There are no build/test commands. Work here is almost always editing prompts. When changing the pack:

1. Identify which load-bearing files are affected (see list above).
2. Update them as a group — if `CONTRACT.md` changes the schema, the role files and `DISPATCH-REFERENCE.md` likely also need touching.
3. Verify lean-mode and full-mode flows still describe consistent behavior.
4. Preserve the canonical agent names — they are referenced by the dispatch template and by `recommended_agent` fields.
5. The demo projects mentioned in `README.md` (`demo-greeting`, `demo-click-message`, `demo-pomidoro`, `demo-traffic-simulator`, `demo-function-plot-cli`) are **not present** in this working tree. If you need to reason about a demo, check tags/branches or ask before assuming.
