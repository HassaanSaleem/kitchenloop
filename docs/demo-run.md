# Demo run — KitchenLoop on relay-notes

A live capture of this harness driving the
[`kitchenloop-demo`](https://github.com/HassaanSaleem/kitchenloop-demo)
seed (`relay-notes` — a tiny zero-dependency note API). Nothing here is staged:
these are the loop's own actions, in order. The demo host has no Docker and the
product is API-only, so the **live-stack / L3-boot gates record `SKIPPED`**; the
gates that *did* run are the deterministic oracle, the quality bar, the Claude
`pr-auditor`, the Codex tribunal, and the zero-context UAT evaluator.

## The headline — a genuine autonomous merge (PR #3)

With the full review tribunal enabled (`reviewers.codex.enabled: true`), one
iteration drained a bug ticket, implemented the fix with tests, passed **every**
gate, and **merged itself to `main`**
([PR #3](https://github.com/HassaanSaleem/kitchenloop-demo/pull/3), merge commit
`596393b`):

| Gate | Result |
| --- | --- |
| Claude `pr-auditor` (Stage 3) | APPROVE |
| **Codex Stage 4a** (alignment review) | **APPROVE — "no blocking issues… matches the README-documented fixes… no new runtime dependencies or route surface"** |
| Zero-context UAT evaluator (Haiku, sealed card) | **PASS** |
| **Codex Stage 8** (final merge-safety) | **APPROVE** |
| Regression oracle (post-merge, on `main`) | green |

The Codex verdicts are real — the loop wrote them to `/tmp/codex-4a-3.txt` and
`/tmp/codex-8-3.txt` from actual `codex exec` runs. PR #3 shipped 221 insertions
across `src/` and `tests/` (195 lines of new tests) fixing the "one malformed
note 500s all of `/search`" bug the loop had found itself.

## How it got there

The first iterations, run **before** the tribunal was enabled, are worth keeping
because they show the gates *catching* things rather than waving them through.

**Iteration 1 (full loop, tribunal off).** Ideate ran as a synthetic user and
found a real bug unaided:

> a note created without a `title` (which the API doesn't validate) permanently
> breaks `/search` with a **500 for every note**.

It documented 3 product bugs + 2 passing behaviors, and — deriving the coverage
matrix — found a bug in the harness's **own** tooling:

> a real bug in `derive-coverage.mjs` … Since `scripts/` changes are on
> MANDATE's ALWAYS-STOP list, I won't fix it myself — this goes to ESCALATIONS.md.

That is the discipline working: the loop refused to touch its own protected code
and escalated instead. The run also exposed **three real harness bugs** (an
over-strict YAML-indent parser, a `grep -c` double-print in `escalation_count`,
and an over-broad protected-path entry that blocked legitimate merges) — all
fixed in commit
[`09d26c3`](https://github.com/HassaanSaleem/kitchenloop-demo/commit/09d26c3).

**Iteration 2 (drain, tribunal off).** Execute implemented a fix and earned a
SHA-pinned UAT `PASS`, but the merge gate **correctly refused**: a follow-up
commit had landed *after* the UAT ran, so the verdict no longer covered the head
commit —

```
RESULT: NOT_MERGEABLE: head moved past UAT'd commit — UAT certifies eaf1d31
but head is f04c45c (un-UAT'd change)
```

A merge pipeline that checks *tested-SHA == merged-SHA* is the opposite of a
rubber stamp. (That PR was later closed and its branch deleted; PR #3 is the
clean, tribunal-reviewed merge that replaced it.)

## What the run demonstrates, end to end

- A synthetic user that finds **real, reproducible bugs** (the `/search` 500)
- SDD execute producing a **real fix with new tests**
- A **configurable review tribunal**: the Claude `pr-auditor` always, plus a
  genuine cross-vendor **Codex** review (Stages 4a + 8) — no model reviews its
  own work
- A **zero-context UAT evaluator** returning a SHA-pinned `PASS`
- Gates that both **pass honestly** (PR #3) and **refuse honestly** (the
  protected-path escalation in iter 1, the stale-UAT refusal in iter 2)
- `main` kept green throughout, with a clean autonomous merge to close it out
