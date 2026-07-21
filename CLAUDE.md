# CLAUDE.md

This file provides guidance to Claude Code when working in the KitchenLoop repository.

## What This Repo Is

This repo IS the harness — an autonomous improvement loop that you point at
your project via `kitchenloop.yaml` (copy `kitchenloop.example.yaml` and adapt
it). Everything below describes how the loop works and the rules any agent
must follow while working here.

## Development Model: Kitchen Loop over SDD

The target project is developed by an autonomous improvement loop (the Kitchen
Loop, arXiv:2603.25697) whose Execute phase routes feature work through
Spec-Driven Development (Spec Kit).

### Spec-Driven Development (features)

Feature-sized work follows: `/speckit-specify` → `/speckit-plan` →
`/speckit-tasks` → `/speckit-implement`. Specs live under `specs/`.
Plans must pass the Constitution Check before implementation.

### Kitchen Loop (the outer loop)

- Config: `kitchenloop.yaml` (spec surface dimensions, oracle commands, gates)
- Run one iteration: `./scripts/kitchenloop/kitchenloop.sh 1`
- Phases: Backlog → Ideate → Triage → Execute → Polish → Regress
- Skills: `.claude/skills/kitchenloop-*` — the Execute skill contains the
  SDD routing rules (feature-sized → Spec Kit; bug-sized → direct fix + repro test)
- State: `docs/internal/loop-state.md`, coverage in
  `.kitchenloop/coverage-matrix.yaml`, tickets in `.kitchenloop/backlog.json`
  (local provider), known-broken combos in `.kitchenloop/blocked-combos.yaml`
- Quality gate: `.kitchenloop/quality-bar.md` — includes "The Bar" (the
  project's domain red-lines). L3 integration tests are mandatory for
  integration points and money paths; see `.kitchenloop/unbeatable-tests.md`.
- Live Test & Fix gate (the Live Test & Fix rule, `.kitchenloop/quality-bar.md`):
  every SDD cycle that changes product behavior ends with live verification
  against the built compose image — QA browser journey (Playwright/Playwright
  MCP) with per-step screenshots, compose-log scan, defects fixed in the same
  cycle. Evidence lands in `.kitchenloop/evidence/` (git-ignored); the PR body
  carries the verdict in a Live Test Evidence section.
- UAT gate: protected test cards in `.kitchenloop/uat-cards/`, evaluated by the
  fresh zero-context `uat-evaluator` agent (`.claude/agents/uat-evaluator.md`)
  driving the live stack in a real browser (Playwright MCP, screenshots)
- Worktrees: loop phases run inside git worktrees under `.claude/worktrees/`
  (git-ignored) so loop work never disturbs the root repo's checked-out branch.

### Harness Rules (read before any loop work)

- **`MANDATE.md`** — the owner's standing mandate. Read it at the start of every
  loop phase. It grants autonomy per work class and lists what ALWAYS stops
  the loop (constitution/schema/money-semantics changes, gate changes, push,
  deploy). Only the owner edits it.
- **`ESCALATIONS.md`** — the one place the loop parks a question for the owner:
  one row, with the exact word the owner says to unblock it. If a decision
  isn't recorded there, the loop hasn't actually asked and must not proceed as
  if it had. Never tuck an ask into prose, a PR comment, or chat.
- **STOP sentinel** — if `.kitchenloop/STOP` exists, every loop phase refuses to
  run until the owner deletes it. Any phase may create it (with a reason
  inside) when drift metrics or stop conditions trip.
- **Protected gates** — the loop may not edit the things that judge it: the
  quality bar, the regression oracle definitions, the UAT protocol, the
  quality-sweep rules, MANDATE, or the pr-manager review stages. Changing any
  of these is the owner's call (an escalation entry), never a loop commit.
- **Evidence-backed claims** — "done" means there is something to show for it: a
  command output, a test run, a UAT card verdict. No evidence, not done.

### Rules for any agent working here

- Never weaken a test assertion, edit a protected UAT card after evaluation, or
  mark a coverage cell tested without a passing scenario — honesty is enforced
  mechanically, don't fight it.
- Money paths require L3 integration tests with state-conservation assertions
  on failure paths.
- The core domain schema may only be changed with a versioned migration note
  in the feature plan.
- No third-party API shapes outside their adapter boundary.
- Review gates check code ↔ spec ↔ architecture alignment (VIOLATION /
  MISSING-IMPL / DRIFT / EXTRA-BEHAVIOR block; SPEC-GAP files a spec-amendment
  ticket — code never silently defines the spec).

## Commands

```bash
npm test           # this repo's own tests (the coverage-deriver); CI runs the same
npm run lint       # this repo's shell+JS syntax gate
# NB: in the PROJECT the loop runs against, the oracle commands in
# kitchenloop.yaml are what gate — pnpm/Node are only the documented default there.
./scripts/kitchenloop/kitchenloop.sh 1          # one loop iteration
./scripts/kitchenloop/kitchenloop.sh 10         # overnight batch
tail -f .kitchenloop/logs/*.log                 # monitor
```
