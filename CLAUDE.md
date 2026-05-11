# Coordinated Agent Team — Claude Instructions

This repository contains a **prompt-driven multi-agent system** for autonomous software delivery.
It defines clear agent roles, a deterministic workflow, strict I/O contracts, and artifact-based coordination.

## How This System Works

Agents are defined as markdown files in `.github/agents/`. Each agent has a YAML frontmatter block
specifying its `name`, `tools`, and `model`. The Orchestrator controls the state machine and dispatches work.

All runtime artifacts are written to `.agents-work/<session>/` (gitignored).

## Full Workflow

```
INTAKE -> DESIGN -> APPROVE_DESIGN -> PLAN -> REVIEW_STRATEGY -> IMPLEMENT_LOOP -> INTEGRATE -> RELEASE -> DONE
```

Lean workflow (trivial/low-risk tasks):
```
INTAKE_LEAN -> IMPLEMENT_LOOP -> INTEGRATE -> DONE
```

## Agent Roster

| # | Agent | Responsibility |
|---|---|---|
| 00 | Orchestrator | Controls state machine, dispatches work, enforces gates |
| 01 | SpecAgent | Produces `spec.md`, `acceptance.json`, and initial session artifacts |
| 02 | Architect | Designs architecture and ADRs |
| 03 | Planner | Builds `tasks.yaml` with dependencies and checks |
| 04 | Coder | Implements scoped tasks |
| 05 | Reviewer | Structured code review and risk analysis |
| 06 | QA | Test planning, execution, acceptance validation |
| 07 | Security | Security assessment and decision-trigger findings |
| 08 | Integrator | Build/CI integration and release readiness |
| 09 | Docs | README/report updates and delivery documentation |
| 10 | Designer | UI/UX design specs |
| 11 | Researcher | Evidence-based technical research |

## Key Files

- `.github/agents/CONTRACT.md` — global I/O contract, status model, artifact requirements, hard gates
- `.github/agents/WORKFLOW.md` — state machine, dispatch rules, lean mode, repair loops
- `.github/agents/DISPATCH-REFERENCE.md` — mandatory dispatch template and pre-dispatch validation
- `.github/agents/00-orchestrator.md` through `11-researcher.md` — role-specific instructions

## Artifact Model

All session artifacts live under `.agents-work/YYYY-MM-DD_<short-slug>/`:

| Artifact | Purpose |
|---|---|
| `spec.md` | Feature specification and acceptance criteria |
| `acceptance.json` | Machine-readable acceptance criteria |
| `tasks.yaml` | Task list with dependencies and status |
| `status.json` | Session-level state: current state, decisions, assumptions |
| `report.md` | Final delivery report |
| `architecture.md` | Architecture decisions (full mode only) |

Task status lifecycle: `not-started` → `in-progress` → `implemented` → `completed` (or `blocked`)

## Quality Gates — Workflow Cannot Progress When

- Required artifacts are missing or invalid
- `APPROVE_DESIGN` not passed (full mode)
- `REVIEW_STRATEGY` not chosen (full mode)
- Reviewer, QA, or Security returns `BLOCKED`
- Cross-task final review (`task.id: meta`) returns `BLOCKED`
- Build/CI is red

## Using This System in a Project

1. Copy `.github/agents/` into your project root
2. Add `.agents-work/` to `.gitignore`
3. Optionally create `.github/copilot-instructions.md` with project coding standards
4. Start: `@orchestrator Build X with constraints Y (project_type: web|api|cli|lib|mixed)`

## Precedence

`CONTRACT.md` > agent specs > `WORKFLOW.md` / `DISPATCH-REFERENCE.md` > `copilot-instructions.md`

## Demo Projects

- `demo-greeting` — lightweight greeting card
- `demo-click-message` — minimal click-to-message static demo
- `demo-pomidoro` — Pomodoro timer with distraction journal
- `demo-traffic-simulator` — traffic simulation on Canvas
- `demo-function-plot-cli` — Python terminal app for plotting f(x)

## Maintenance Rules

1. Keep `CONTRACT.md` as canonical source for schema and status definitions.
2. Keep `DISPATCH-REFERENCE.md` in sync with `00-orchestrator.md` dispatch rules.
3. Update `WORKFLOW.md` and role files together to avoid drift.
4. Re-check lean/full mode consistency after every rule change.
5. Preserve canonical agent names used in dispatch and `recommended_agent`.
