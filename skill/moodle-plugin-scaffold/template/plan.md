---
name: Plan
description: Architecture and build decisions for Phase 4a — file structure, data model, integration points, security & authorization, open decisions and spikes (fill in per project)
type: plan
created: 2026-09-23
timestamp: 2026-09-23
---

# Plan — [FRANKENSTYLE]

> **Status:** Template. Fill this in during Phase 4a of [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development), after `specs.md`/`user-stories.md` are approved (Gate 1). This isn't part of the spec — that stays `specs.md` and `user-stories.md`. This file translates those two into architecture and build decisions; the numbered build order lives separately in [`tasks.md`](tasks.md).

## 1. Overview

[One or two sentences: what gets built technically. Points to `specs.md`/`user-stories.md`, doesn't repeat the functional scope.]

### Status and gate

[Which phases are done? Has Gate 1 (approval) been recorded? Without a recorded approval, Phase 4b doesn't start.]

## 2. Architecture

### 2.1 File structure

[Proposed folder/class layout, new or changed.]

```
[PLUGIN_MAPNAME]/
├── version.php
├── ...
```

### 2.2 Data model

[Tables, fields, types, indexes — the technical side of `specs.md`'s functional Data Model section. "No changes" is a valid answer.]

### 2.3 Integration points

[Hooks/callbacks/events (Moodle Hooks API) — the technical side of `specs.md`'s Capabilities/Hooks section. Which core APIs get reused instead of built from scratch?]

### 2.4 Security & authorization

[Concrete capability names (table: capability / context level / archetypes / riskbitmask), authorization checks.]

### 2.5 External dependencies

[APIs/packages, sync vs. async, rate-limit approach. "N/A" if there are none.]

## 3. Open decisions

[Decisions that need confirming before Phase 4b — with a proposal and the reasoning behind it, not just the question.]

## 4. Spikes

[One per unconfirmed technical risk at implementation level (like Phase 1a, but after design). "N/A" if there are none.]

## What this plan doesn't do

- It doesn't replace `specs.md`/`user-stories.md` — if they contradict, `specs.md` wins.
- It holds no build order or progress — that lives in [`tasks.md`](tasks.md).
