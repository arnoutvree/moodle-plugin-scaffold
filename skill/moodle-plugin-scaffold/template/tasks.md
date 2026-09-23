---
name: Tasks
description: Build order with one score per task that doubles as its status (Phase 4a-4c) — fill in per project
type: tasks
created: 2026-09-23
timestamp: 2026-09-23
---

# Tasks — [FRANKENSTYLE]

> Belongs with [`plan.md`](plan.md) (architecture, locked once approved) and [`user-stories.md`](user-stories.md) (user stories/test scenarios, Phase 2). This is the numbered build order for Phase 4a: per task the files involved, which test scenario it covers, and one verification step. Stays a living document through the build (Phase 4b); can be discarded after release — no regression value like `user-stories.md`.
>
> **Status = score (Phase 4c of [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development)).** Every task gets one 1-8 score in the table below. That score is the only status: no score yet = not started, done = the score clears its threshold. No separate status column, and no status in `user-stories.md`. A task with an objective check (automated test): threshold is always 8. A task with a judgment call (e.g. generated copy or a prompt): a lower threshold (e.g. ≥6), plus `max_iterations` and a fallback.

## 1. Status and gate

[Which phases are done? Have Gate 1 and the sign-off on this plan (4a) been recorded?]

## 2. Build order

1. **[Task name]** — files: `...`. Covers: US N / test scenario N.M. Verification: `...`

## 3. Score

| Task | What's being scored | Score | Threshold | What's needed for a higher score | Status |
|---|---|---|---|---|---|
| 1. [Task name] | [e.g. PHPUnit/Behat: test scenario N.M, happy and sad path] | — | 8 | — | ⬜ Not started |

<!--
"What's needed for a higher score" is the actual reasoning for the gap, not a
placeholder: for an objective task the missing assertion or uncovered case, for a
judgment call the evaluator's reasoning. Status follows from the score:
⬜ Not started · 🟡 Below threshold · ✅ Done · ✅ Accepted (round X/Y)
· ⚠️ Fallback (threshold not met after max_iterations).
-->
