---
type: navigation
created: 2026-06-26
updated: 2026-08-17
timestamp: 2026-08-17
---
# CLAUDE.md — [FRANKENSTYLE]

Moodle [PLUGIN_TYPE] plugin. All code must follow Moodle coding standards and target the **current LTS release or later** (PHP version per that release's support matrix).

> **Version policy:** don't hardcode a Moodle/PHP floor from memory — check [moodledev.io/general/releases](https://moodledev.io/general/releases) for the current LTS and its PHP/database requirements before filling in the table below. Moodle ships ~2 majors per year and PHP support windows shift with each one; a version pinned at template-creation time goes stale fast. See also [context/software/moodle.md](context/software/moodle.md).

## Plugin identity

| Property     | Value                  |
|---|---|
| Plugin type  | `[PLUGIN_TYPE]`        |
| Plugin name  | `[PLUGIN_NAME]`        |
| Frankenstyle | `[FRANKENSTYLE]`       |
| Moodle       | [FILL IN: current LTS or later, verify at moodledev.io/general/releases] |
| PHP minimum  | [FILL IN: verify against the chosen Moodle version's support matrix] |

---

## Universal conventions

If this plugin's design and delivery follows a spec-driven process (written spec, approval gate, independent review before release), keep that process's own instruction file as the authority above this one on *process* — this file stays authoritative on Moodle/PHP conventions. See [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development) for a ready-made version of that process.

---

## When to read which file

**Database or upgrade changes?** → [context/tech/database-schema.md](context/tech/database-schema.md)  
**Capabilities or role permissions?** → [context/tech/capabilities.md](context/tech/capabilities.md)  
**Frontend, templates, or AMD modules?** → [context/platform/moodle.md](context/platform/moodle.md) (Output API section)  
**Replacing a legacy `lib.php` callback?** → [context/platform/moodle.md](context/platform/moodle.md) (Hooks API section) — check whether a Hook already replaces it before adding a new callback  
**Testing (PHPUnit/Behat)?** → [context/tech/testing.md](context/tech/testing.md)  
**Static analysis / CI (moodle-plugin-ci, PHPStan)?** → [context/workflows/coding-standards.md](context/workflows/coding-standards.md) (Static analysis section)  
**Writing or approving a spec?** → [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development) (Phase 2 and 5)  
**Ready to release?** → [context/workflows/release-process.md](context/workflows/release-process.md)  
**General rules & conventions?** → [context/workflows/coding-standards.md](context/workflows/coding-standards.md)

---

## All context files (load as needed)

| File | Purpose |
|---|---|
| [identity.md](identity.md) | AI expertise, focus, and approach for this project |
| [rules.md](rules.md) | Strict boundaries and non-negotiable constraints |
| [intent.md](intent.md) | Problem statement, target users, known constraints — fill in before specs.md |
| [specs.md](specs.md) | Project requirements, features, user flows, data model |
| [context/tech/architecture.md](context/tech/architecture.md) | Plugin folder structure, entry points, namespacing |
| [context/tech/database-schema.md](context/tech/database-schema.md) | Database design, DML patterns, upgrade strategy |
| [context/tech/api-patterns.md](context/tech/api-patterns.md) | External services, AJAX, authentication, error handling |
| [context/tech/testing.md](context/tech/testing.md) | PHPUnit setup, Behat, test data generation |
| [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development) | Spec-driven process: writing test scenarios, getting them approved, translating them to Behat/PHPUnit (Phase 2 and 5) |
| [context/tech/capabilities.md](context/tech/capabilities.md) | Capability design, role safety checks, permission patterns |
| [context/platform/moodle.md](context/platform/moodle.md) | Moodle subsystems: Output API, Events, AMD, Forms, Privacy |
| [context/software/moodle.md](context/software/moodle.md) | Official Moodle references, version matrix, tools |
| [context/workflows/release-process.md](context/workflows/release-process.md) | Release checklist, version numbering, zip packaging |
| [context/workflows/coding-standards.md](context/workflows/coding-standards.md) | PHP style, naming, documentation, commit discipline |

---

## Behavioral guidelines — always follow these

**MOODLE FIRST — never rebuild what core already provides**
- Before building any UI widget, endpoint, or utility: check whether Moodle core already has it. If it does, use it — even when a custom version would better match a design mockup.
- Applies especially to: file upload/management (`form_filemanager`/filepicker with `accepted_types` — never a hand-rolled dropzone), form elements and validation, modals (`core/modal`), notifications/toasts (`core/notification`, `core/toast`), string management (`core/str`), AJAX (`core/ajax`), templates (`core/templates`), user/course pickers.
- A design handoff shows *what* the interface should do, not *how* to implement it — map its widgets onto core components first, and only build custom when core genuinely has no equivalent.
- Custom rebuilds of core features are a bug, not a style choice: they miss server-side validation, accessibility, theming and future core improvements.

**THINK FIRST, CODE SECOND**
- State assumptions explicitly before implementing. Name multiple interpretations, don't choose silently. Push back if a simpler approach exists.
- If anything is unclear: stop, name what's confusing, ask one focused question.
- If less than 80% confident in intent or execution: ask for context and goal first. Implement only after sufficient clarity.

**SIMPLICITY FIRST**
- Write minimal code solving the problem — nothing speculative.
- No functions, abstractions, or configurable options beyond what's asked.
- No error handling for scenarios that cannot occur.
- Ask yourself: "Would a senior engineer find this over-engineered?" If yes, simplify.

**SURGICAL CHANGES**
- Touch only what's asked — don't "improve" surrounding code, comments, or formatting.
- Never refactor working code. Match existing style, even if you'd do it differently.
- Be careful modifying callbacks in `lib.php` outside task scope.
- Be careful modifying capabilities in `db/access.php` that touch existing roles.
- Be careful modifying `db/upgrade.php` steps already running in production.
- If your change leaves an import, variable, or function unused: delete it. Don't delete pre-existing dead code unless explicitly asked.
- If you spot unrelated dead code or a potential problem: mention it in a comment — don't repair it.

**TARGETED EXECUTION**
- Translate each task into a testable goal before implementing:
  - "Add external function" → "Write PHPUnit test calling the function and verifying output, then implement"
  - "Add capability" → "Define which archetypes get access, what risk mask applies, then implement and verify with `get_roles_with_capability()`"
- For multi-step tasks: state a short plan with one verification per step:
  ```
  1. [Step] → verify: [check]
  2. [Step] → verify: [check]
  ```
- Vague success criteria ("make it work") signal a need to ask first, not guess.

---

## Absolute prohibitions — enforce these always

**PROJECT STRUCTURE**
- Never write code outside the plugin subfolder `[PLUGIN_MAPNAME]/`
- Never place files in project root (except the released zip)
- Never rename the plugin subfolder — it must exactly match the second part of Frankenstyle (e.g., `translatecourse` for `local_translatecourse`)

**MOODLE INTEGRITY**
- Never modify Moodle core files
- Never use deprecated Moodle functions without explicit instruction (check https://moodledev.io/docs/5.0/guides/deprecation)
- Never overwrite tables, capabilities, or settings not created by this plugin
- Always start every PHP file with `defined('MOODLE_INTERNAL') || die();`, except entry-point pages (`require_once('../../config.php')`), files without side-effects (a lone class/interface/trait — the official Moodle exception), and `db/install.php`/`db/upgrade.php` (install/upgrade-only files; nearly all Moodle core plugins omit the check there)

**DATABASE (Moodle DML)**
- Never write raw SQL — always use the `$DB` global (`get_record`, `get_records_sql`, `insert_record`, etc.)
- Always use named parameters or `?` placeholders in `*_sql()` calls — never string concatenation
- Never delete tables or fields without a migration step in `db/upgrade.php` and version bump in `version.php`
- Always define tables in `db/install.xml` via XMLDB editor, never by hand

**CAPABILITIES & ACCESS**
- Never modify `db/access.php` without checking impact on existing roles via `get_roles_with_capability()`
- Always call `require_capability()` or `require_login()` before state-changing actions
- Capability rename or delete requires `upgrade_rename_capability()` or explicit delete in `db/upgrade.php` — never silent

**SECURITY** — see also the [OWASP Top 10](https://owasp.org/www-project-top-ten/) for the broader risk framework this maps onto
- Never store credentials, API keys, or passwords in code or version control
- Always use `sesskey()` / `confirm_sesskey()` for every state-changing POST
- Always read user input via `required_param()` / `optional_param()` with explicit `PARAM_*` type — never direct `$_GET` / `$_POST`
- If a value can contain digits (e.g. `savecheck2`), use `PARAM_ALPHANUMEXT` — `PARAM_ALPHA` silently strips digits, so a value like `savecheck2` arrives corrupted (`savecheck`) and never matches
- Always escape output via `s()`, `format_string()`, `format_text()`, or Mustache
- Never use `eval()`, `base64_decode()` on user input, or similar constructs

**TESTING — SAD PATH**
- Every new capability check, validation, or external function gets at least one negative PHPUnit scenario alongside the happy path: `$this->expectException(required_capability_exception::class)` for missing permissions, a validation error for invalid input, `MUST_EXIST`/null handling for records that don't exist
- Assert on the specific exception type (`$this->expectException(...)`), never just "doesn't fail with a fatal error"
- See [context/tech/testing.md](context/tech/testing.md) for the technical test syntax

**DEPENDENCIES**
- Never add a new composer or npm package without asking first — justify: which problem it solves, whether Moodle core or an existing dependency already covers it (see MOODLE FIRST above), its license, size, maintenance status, and supply-chain risk.
- Default to no. A package must earn its place, not save a few lines of code.

**RELEASES & DELIVERY**
- Always generate a zip file as part of release
- Filename always: `[PLUGIN_NAME]-[version].zip` (e.g., `local_translatecourse-1.2.0.zip`)
- Zip root is always the plugin subfolder with name **without** type prefix (e.g., `translatecourse/`, not `local_translatecourse/`) — otherwise Moodle rejects install
- Always exclude Mac files, .git, node_modules, tests/, .env, .idea/, .vscode/, logs, .codegraph
- Always bump `$plugin->version` (format `YYYYMMDDXX`) and update `$plugin->release` (semver) before release

---
