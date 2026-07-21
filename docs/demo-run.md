# Demo run — KitchenLoop on relay-notes

A live capture of this harness driving the
[`kitchenloop-demo`](https://github.com/HassaanSaleem/kitchenloop-demo)
seed (`relay-notes` — a tiny zero-dependency note API). Nothing here is staged:
these are the loop's own actions, in order.

## Iteration 1 — full loop (`kitchenloop.sh 1`)

**Ideate** ran as a synthetic user against the API and found a real bug on its own:

> a note created without a `title` (which the API doesn't validate) permanently
> breaks the `/search` endpoint with a **500 for every note**, not just the
> malformed one.

It wrote a scenario documenting **3 product bugs + 2 passing behaviors**, then —
while deriving the coverage matrix — found a bug in the harness's **own** tooling:

> a real bug in `scripts/kitchenloop/derive-coverage.mjs` … the list-item regex
> requires exactly 4-space indent, while standard YAML nests list items at 6.
> Since `scripts/` changes are on MANDATE's ALWAYS-STOP list, I won't fix it
> myself — this goes to ESCALATIONS.md.

That is the discipline working exactly as designed: the loop **would not touch
its own protected code**, and escalated instead.

**Triage → Execute** turned the findings into tickets and opened PR #1 (5
tickets). **Polish** (the pr-manager) then held the line: PR #1 touched a
protected path, so it was marked `NOT_MERGEABLE` and an escalation (`PRM-PROT-1`)
was filed rather than merged. **Regress** confirmed `main` stayed green
throughout. Exit 0.

## Three harness bugs the run surfaced — fixed

The first iteration paid for itself by exposing three real defects (commit
[`09d26c3`](https://github.com/HassaanSaleem/kitchenloop-demo/commit/09d26c3)):

1. **`derive-coverage.mjs`** — accept YAML list items at 4-*or-more* space
   indent (the loop found this one itself).
2. **`escalation_count()`** — `grep -c … || echo 0` double-printed `0\n0` on
   no-match (grep -c prints `0` *and* exits 1), breaking a later integer test.
3. **`PROTECTED_PATHS`** — dropped `.kitchenloop/uat-cards/`: the execute phase
   legitimately authors a new card per ticket, and tamper-proofing sealed cards
   is the UAT gate's freeze check, not the merge guard's. (This was what blocked
   PR #1.)

## Iteration 2 — drain + merge (`--mode dev-only`)

On the fixed harness, **Execute** (Opus) drained a backlog ticket, implemented
the fix, wrote a UAT card, and ran the **zero-context `uat-evaluator`** (Haiku,
in an isolated worktree) — which returned:

```
Verdict: PASS
Commit under test: eaf1d31 (evaluator pinned its checkout to this SHA)
```

That is the anti-gaming design in action: the evaluator sees no implementation
context and pins its checkout to a specific commit.

**The merge then correctly did *not* happen** — and this is the most important
result in the whole run. A follow-up fix (`f04c45c`, "return 400 not 500 for
malformed POST body") landed *after* the UAT ran on `eaf1d31`, so the merge gate
refused:

```
RESULT: NOT_MERGEABLE: head moved past UAT'd commit — UAT certifies eaf1d31
but head is f04c45c (un-UAT'd change); needs fresh execute-phase UAT + re-prep
```

The gate verifies that the UAT verdict covers the **exact commit being merged**.
It will not merge code whose verification is one commit stale. A merge pipeline
that checks *tested-SHA == merged-SHA* is the opposite of a rubber stamp — it is
precisely the property that makes an autonomous loop safe to leave running.

## What this run demonstrates, live

- A synthetic user that finds **real, reproducible bugs** (the `/search` 500)
- SDD execute producing a **real fix with new tests**
- A **zero-context UAT evaluator** returning a SHA-pinned `PASS`
- Every gate holding: **protected-path** guard + escalation, the **UAT-covers-head**
  merge gate, the **STOP** sentinel, **worktree isolation**, and rejection-SHA
  backpressure
- `main` kept green end to end

The one thing a single clean iteration would add — a fully green auto-merge —
requires the execute phase's UAT to be the final step with no post-UAT commit;
here a late fix invalidated the verdict and the gate, correctly, refused. In
sustained operation the loop re-enters execute for that ticket, re-runs UAT on
the final code, and merges. The
[open PR](https://github.com/HassaanSaleem/kitchenloop-demo/pulls) and its
`PASS` UAT comment are left visible as evidence.
