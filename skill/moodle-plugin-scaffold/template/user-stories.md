---
name: User Stories & Test Scenarios
description: Living progress table for user stories and test scenarios (fill in per project, separate from specs.md)
type: specs
created: 2026-08-22
timestamp: 2026-08-22
---

# User Stories & Test Scenarios — [FRANKENSTYLE]

> **Status:** Template. Fill this in during the design phase of your process, alongside `specs.md` — see [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development) (Phase 2) if you're following that process.

Living counterpart to [`specs.md`](specs.md): that file describes *what* gets built (done once the build starts), this file tracks *whether it actually works* — during the build, during testing, and for later regression checks.

## Progress

| # | Test scenario | Status (self-test) | Test | Verification (independent) |
|---|---|---|---|---|
| 1.1 | [description] | ⬜ Open | — | ⬜ Not yet verified |
| 1.2 | [description] | ⬜ Open | — | ⬜ Not yet verified |

**Status legend (self-test):** ⬜ Open (not yet built) · 🟡 In progress · 🟢 Passed · 🔴 Failed · ⚫ Dropped (change request)

**Status legend (verification):** ⬜ Not yet verified · 🟢 Verified by [name/agent] · 🔴 Rejected by [name/agent] — reason

The `Test` column fills in during the build with the implemented Behat scenario or `_test.php` method, following the fixture naming in [`context/tech/testing.md`](context/tech/testing.md).

**Status vs. Verification:** the `Status` column is the result of the automated test written during the build (the same party that builds, tests) — the `Verification` column is a second, independent look: another AI agent (without the builder's context/assumptions) or a human, checking the scenario apart from its own test. Keep this explicitly separate from "tested" — a green self-test doesn't prove nobody shared the same blind spot.

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
