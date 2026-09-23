---
name: User Stories & Test Scenarios
description: User stories and their test scenarios (fill in per project, separate from specs.md); progress lives in tasks.md
type: specs
created: 2026-08-22
timestamp: 2026-08-22
---

# User Stories & Test Scenarios — [FRANKENSTYLE]

> **Status:** Template. Fill this in during the design phase of your process, alongside `specs.md` — see [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development) (Phase 2) if you're following that process.

Reference document alongside [`specs.md`](specs.md): that file describes *what* gets built (done once the build starts), this file holds the test scenarios per user story that check it — during the build, during testing, and for later regression checks.

**No progress tracking in this file.** Self-test status lives in exactly one place: the score table in [`tasks.md`](tasks.md) (Phase 4c of [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development)), where each task points back here by scenario ID (e.g. "test scenario 1.1"). Tracking status in two places that have to stay in sync drifts apart sooner or later. The independent review (Phase 5) goes into your issue tracker, not a column here. If a scenario turns out infeasible during the build, mark it **⚫ Superseded (change request)** directly under that scenario's heading, with a link to the change request — don't delete it, and don't quietly rewrite it to match whatever got built instead.

---

## US 1: [Title]
- **As** [role/type of user]
- **I want** [functionality/action]
- **so that** [value/goal]

#### Test scenario 1.1: [name — happy path]
- **Given** [starting situation]
- **When** [action]
- **Then** [expected result]

#### Test scenario 1.2: [name — alternative outcome/edge case]
- **Given** [starting situation]
- **When** [action]
- **Then** [expected result]

<!--
Consider per user story (not every category applies every time):
happy path, alternative user action, config-/setting-driven variation,
error/retry path, duplicate or overlapping input. If a decision point has
multiple outcomes, give each outcome its own scenario — not one scenario
for "the" edge case. State in the Then what must *not* happen, where relevant.
-->
