---
name: moodle-plugin-scaffold
description: |
  Scaffold a new Moodle plugin project: verifies the current Moodle/PHP support matrix, then generates both the CLAUDE.md context layer (identity, rules, specs, tech/platform/workflow docs including Hooks API and moodle-plugin-ci setup) and the actual plugin code skeleton (version.php, lib.php, db/access.php, lang file, privacy provider) in one pass.

  Use whenever the user wants to start a new Moodle plugin project, or says something like "new Moodle plugin project", "scaffold a Moodle plugin", "set up a plugin from the Moodle template".

created: 2026-09-14
---

# Moodle plugin scaffold — set up a new project

This skill sets up a new Moodle plugin project from the template bundled in this
skill's own `template/` folder — no external repo to clone. The result is a
fully filled-in `CLAUDE.md` context structure with current Moodle/PHP version
data (never copied placeholders), plus a working code skeleton to build on.

**Universal rule:** ask one thing at a time, wait for the answer before continuing.

---

## Phase 0 — intake

Ask these questions **one at a time**:

1. **Plugin type** (`local`, `mod`, `block`, `auth`, `theme`, `report`, `tool`, `enrol`, ...) — see the "Plugin types and required files" table in `template/context/software/moodle.md` if the user is unsure.
2. **Plugin name** (short, lowercase, no underscores needed beyond the frankenstyle composition — e.g. `translatecourse`)
   → derive: `FRANKENSTYLE` = `[type]_[name]` (e.g. `local_translatecourse`), `PLUGIN_MAPNAME` = the folder the plugin lives in (e.g. `local/translatecourse`)
   → show the derived values and ask for confirmation.
3. **One-sentence description** of the problem this plugin solves (for `specs.md`'s Overview).
4. **Target roles** — which Moodle roles (student/teacher/editingteacher/manager/custom) use the plugin, and which capabilities does each get?
5. **Author/organization** — for the `@copyright` header and `version.php`.

---

## Phase 1 — verify the version matrix (never fill in from memory)

Never fill in a Moodle or PHP version from training data — it goes stale fast (Moodle ships ~2 majors/year). Always:

1. `WebFetch` `https://moodledev.io/general/releases` for the current (LTS) version and its PHP/database support window.
2. Determine whether the candidate floor is already **security-only** or **EOL** — if so, pick a newer version as the floor instead.
3. Summarize: "I'll target Moodle [X] LTS+ and PHP [Y]+, based on [source/date]. Correct?" — wait for confirmation.

If WebFetch fails or isn't available: say so explicitly and ask for the versions directly instead of guessing.

---

## Phase 2 — copy and fill in the context structure

1. Copy from this skill's own `template/` folder into the new project folder:
   - `CLAUDE.md`, `identity.md`, `rules.md`, `intent.md`, `specs.md`, `user-stories.md`
   - the full `context/` folder (`tech/`, `platform/`, `software/`, `workflows/`)
   - **not** the `SKILL.md` itself, no `.git/`, no `.DS_Store`
   - The new project folder gets its **own, fresh** git history — never bring along this skill's own `.git/`.

2. Replace, in **all** copied files:
   - `[FRANKENSTYLE]` → the derived frankenstyle name
   - `[PLUGIN_TYPE]` → the plugin type
   - `[PLUGIN_NAME]` → the short plugin name
   - `[PLUGIN_MAPNAME]` → the full folder path
   - the Moodle/PHP placeholders in `CLAUDE.md` and `context/software/moodle.md` → the values confirmed in Phase 1
   - `intent.md`'s Problem and Target users sections → the description and roles given in Phase 0
   - `specs.md`'s Capabilities section → the given roles/capabilities (Overview stays a reference to `intent.md`, don't fill it in separately — that lets the two drift apart)

3. Set `created:` to today in every file; leave `type:` unchanged.

4. Grep-check for remaining placeholders (`grep -rn '\[FRANKENSTYLE\]\|\[PLUGIN_\|\[FILL IN\]'`) and report what's deliberately still open (features, user flows, data model in `specs.md`).

---

## Phase 3 — generate the plugin code skeleton

Unlike a pure context-scaffolding tool, this skill also generates the actual starting code, using the structure documented in `context/tech/architecture.md` (folder layout), `context/platform/moodle.md` (version.php, events, output, privacy) and `context/tech/capabilities.md` (db/access.php) — all already filled in for this project by Phase 2, so generate from those rather than re-deriving the patterns from scratch.

Generate, inside `[PLUGIN_MAPNAME]/`:

- `version.php` — `$plugin->component`, `$plugin->version` (today's date, `YYYYMMDDXX`), `$plugin->requires`, `$plugin->release = '0.1.0'`, `$plugin->maturity = MATURITY_ALPHA`, `$plugin->copyright`, `$plugin->license`.
- `lib.php` — only if the plugin type needs a callback (e.g. `[FRANKENSTYLE]_extend_navigation()`); otherwise skip it, an empty `lib.php` is dead weight.
- `lang/en/[FRANKENSTYLE].php` — at minimum `$string['pluginname']`.
- `db/access.php` — one capability entry per capability named in Phase 0/`specs.md`, with `archetypes` set from the target roles given in Phase 0.
- `classes/privacy/provider.php` — the null provider by default; upgrade this once the plugin actually stores personal data (see the Privacy API section of `context/platform/moodle.md`).
- `.gitignore` and an initial `README.md` stub (plugin name, one-line description, link back to `CLAUDE.md`).

Skip `db/install.xml`/`db/upgrade.php` and any `classes/` beyond the privacy provider until the actual feature work defines what tables/classes are needed — a skeleton with an empty database schema invites drift between the schema and the real design in `specs.md`.

---

## Phase 4 — summary

Close with:

1. Full path to the new project folder.
2. List of created/filled-in files, context layer and code skeleton together.
3. Which sections in `specs.md` are still open (features, user flows, data model, capability details) — expected, not an error.
4. A reminder that no code beyond the skeleton should exist yet — feature work starts from `specs.md`/`user-stories.md`, following whatever spec-driven process the project uses (pairs well with [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development)).

---

## When NOT to use this skill

- For changes to an **existing** plugin project.
- For WordPress or Laravel projects — use an equivalent scaffold for that platform instead.
