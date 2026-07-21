# ESCALATIONS — open questions the loop has parked for the owner

Each row is one decision the loop could not make for itself. It adds a row,
keeps working on everything else, and waits; the owner replies whenever, by
saying the word(s) in **Say**. Once handled, the row is deleted — the history
survives in the loop state and the git log. No rows means the loop currently
needs nothing from a human.

Rules for the loop:
- If the loop stops for the owner, it goes here — not into prose, a PR comment,
  or chat. An unrecorded question has not really been asked.
- Put one short context paragraph directly under the row it belongs to: what
  happened, the recommendation, and what stays blocked meanwhile.
- Never rewrite or reinterpret an existing row's **Say** word; add a new row.

| ID | Say | Question | Recommendation | Since | Blocks |
|----|-----|----------|----------------|-------|--------|
| ✅ E-001 (resolved) | *(done)* | Example: waive the UAT-card gate for a docs-only PR (markdown-only change, no product behavior)? | RESOLVED — waived by the owner: no browser journey applies, every other merge gate green; merged. | 2026-01-01 | done |
