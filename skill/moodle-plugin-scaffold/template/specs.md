---
name: Project Specifications
description: Requirements, features, and architecture for this plugin (to be filled in per project)
type: specs
created: 2026-06-29
timestamp: 2026-06-29
---

# Specs — [FRANKENSTYLE]

> **Status:** Template. Fill `intent.md` in first — this file picks up where that one ends.

## Overview

See [`intent.md`](intent.md) for the problem statement, target users and constraints.
This file covers only the what and how: features, user flows, data model and capabilities below.

---

## Features (MVP)

- [ ] Feature 1 — [description]
- [ ] Feature 2 — [description]
- [ ] Feature 3 — [description]

---

## User Stories & Test Scenarios

See [`user-stories.md`](user-stories.md) — a separate file, since it stays live during build and test (a progress table per scenario), while this file is done once the build starts.

---

## User Flows

### Flow 1: [Name]

```mermaid
graph TD
  A[Start] --> B{Decision?}
  B -->|Yes| C[Action]
  B -->|No| D[Alternative]
  C --> E[End]
  D --> E
```

---

## Data Model

```mermaid
erDiagram
  COURSE ||--o{ TRANSLATION : has
  TRANSLATION {
    int id PK
    int courseid FK
    string language
    string content
  }
```

---

## Capabilities

| Capability | Description | Default roles |
|---|---|---|
| `[FRANKENSTYLE]:view` | [description] | [roles] |
| `[FRANKENSTYLE]:create` | [description] | [roles] |
| `[FRANKENSTYLE]:manage` | [description] | [roles] |

---

## Out of Scope

- [What won't this plugin do?]
- [External dependencies excluded]

---

## Testing Strategy

- **PHPUnit tests:** [What logic needs testing?]
- **Behat scenarios:** [What user workflows need testing?]

Concrete test scenarios (Given/When/Then) and their progress live in [`user-stories.md`](user-stories.md), not here.

