# Moodle Plugin Scaffold

An AI coding agent skill that sets up a new **Moodle plugin** project in one pass: it verifies the current Moodle/PHP support matrix (never guessed from training data), fills in a full `CLAUDE.md` context layer — identity, rules, a spec template, and reference docs on architecture, capabilities, the database, testing, coding standards and release — and generates the actual starting code skeleton (`version.php`, `lib.php`, `db/access.php`, the language file, a privacy provider).

The template itself lives inside this repo (`skill/moodle-plugin-scaffold/template/`), so there's no separate project to clone or keep in sync — one skill, self-contained.

It's a single markdown instruction file plus its bundled template, so it works with whatever AI coding agent you use — natively as a [Claude Code](https://claude.com/product/claude-code) Skill, or pasted into any other agent's system prompt / rules file / custom instructions.

## What you get

- **`CLAUDE.md`** — the entry point: plugin identity table, a "when to read which file" index, and the behavioral rules an AI agent should follow while working on the plugin (Moodle-first, simplicity-first, surgical changes, absolute prohibitions around security/capabilities/database changes).
- **`identity.md` / `rules.md`** — the AI's expertise/focus/approach for this specific project, and its non-negotiable constraints.
- **`intent.md` / `specs.md` / `user-stories.md`** — a problem statement → spec → living test-scenario-progress chain, ready to fill in (pairs well with [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development) if you want the full spec-driven process around it).
- **`context/tech/architecture.md`** — the complete plugin folder structure, entry points, load order, naming conventions.
- **`context/tech/capabilities.md`** — capability design, role archetypes, the permission-safety checklist for anything touching `db/access.php`.
- **`context/tech/database-schema.md`** — DML patterns, `install.xml`/`upgrade.php` conventions.
- **`context/tech/api-patterns.md`** — external services, AJAX, error handling, AI-agent-friendly external function design.
- **`context/tech/testing.md`** — PHPUnit/Behat setup, and a fixture-naming convention that ties test code back to specific test scenarios.
- **`context/platform/moodle.md`** — version.php, Events, the Hooks API, Output API, AMD, Forms, Privacy API, scheduled tasks.
- **`context/software/moodle.md`** — a permanent lookup table of official Moodle docs, the version support matrix (filled in live, not hardcoded), plugin types.
- **`context/workflows/coding-standards.md`** — PHP/JS/CSS style, PHPDoc requirements, commit discipline, `moodle-plugin-ci`.
- **`context/workflows/release-process.md`** — version numbering, the pre-release checklist, zip packaging, a ready-to-use GitHub Actions CI workflow.
- A generated **code skeleton** — not just the planning docs — so the project is immediately buildable.

## Install

Clone this repo:

```bash
git clone https://github.com/arnoutvree/moodle-plugin-scaffold.git
```

**Claude Code:** symlink the skill folder into its user-level skills directory:

```bash
ln -s "$(pwd)/moodle-plugin-scaffold/skill/moodle-plugin-scaffold" ~/.claude/skills/moodle-plugin-scaffold
```

**Any other AI coding agent** (Cursor, Windsurf, Copilot, a custom agent, etc.): point it at [`skill/moodle-plugin-scaffold/SKILL.md`](skill/moodle-plugin-scaffold/SKILL.md) directly, or copy its contents (and the `template/` folder it references) into whatever instruction-file convention that agent uses.

## Usage

From wherever you keep your projects:

```
new Moodle plugin project
```

In Claude Code you can also call it explicitly:

```
/moodle-plugin-scaffold
```

It asks a handful of questions one at a time (plugin type, name, roles, description), verifies the Moodle/PHP version matrix live, then generates the project.

## Related

- [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development) — the spec-driven process (problem → spec → approval → build → independent review → release) this scaffold's `intent.md`/`specs.md`/`user-stories.md` are designed to feed into.
- [`moodle-plugin-vibe-review`](https://github.com/arnoutvree/moodle-plugin-vibe-review) — reviews the code this scaffold (and everything built on top of it) produces, against Moodle's coding, security and API conventions.

## License

MIT — see [LICENSE](LICENSE).
