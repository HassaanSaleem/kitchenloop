---
title: MANDATE — the owner's standing mandate
owner: the owner
last_validated: YYYY-MM-DD
---

The owner writes this file; the loop only reads it — at the start of every
phase — and never edits it. Keep it to a handful of lines: it says what the
loop may do on its own and what it must stop for, nothing more. It is policy,
not a task list. (Running state lives in docs/internal/loop-state.md; the work
queue in .kitchenloop/backlog.json; open questions in ESCALATIONS.md.)

1. The loop may run autonomously: ideate scenarios, triage findings, execute
   backlog tickets through SDD, and merge PRs that pass every gate (lint + tests
   + codex alignment review + UAT card + regression oracle).
2. ALWAYS STOP for: constitution amendments; core domain schema version
   bumps/migrations; changes to money-path semantics; changes to the loop's own
   gates (quality bar, oracle, UAT protocol, sweep rules, pr-manager review
   stages `scripts/pr-manager/prompts/`,
   `docs/architecture/system-architecture.md`, this file); pushes to `main`
   outside the gated pr-manager merge pipeline, pushes to new remotes, and
   force-pushes; deploys. (Feature-branch pushes for PR creation are authorized.)
3. Spec gaps (SPEC-GAP findings) file tickets; spec amendments go through the
   speckit flow with the owner's review — code never silently defines the spec.
4. Quality sweep every 3rd iteration; dead code is removed only via ticketed PRs.
5. When blocked, record the question as a row in ESCALATIONS.md and move on to
   other work. If it isn't recorded there, the loop has not really asked — so
   it must not act as though the owner already answered.
