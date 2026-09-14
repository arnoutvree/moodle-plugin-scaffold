---
name: Strict Rules for [FRANKENSTYLE]
description: Non-negotiable constraints and boundaries for this plugin
type: rules
created: 2026-06-29
timestamp: 2026-06-29
---

# Rules — [FRANKENSTYLE]

## Code & Security

- **Every PHP file** (except entry points) must start with: `defined('MOODLE_INTERNAL') || die();`
- **Never trust user input** — always use `required_param()` / `optional_param()`
- **Never write raw SQL** — always use Moodle DML API (`$DB`)
- **Never echo raw HTML with user data** — use Mustache templates + `format_string()` / `s()`
- **Every state-changing POST** requires `sesskey()` / `confirm_sesskey()`
- **Store settings** in `config_plugins` via `get_config()` / `set_config()`, never in `config`

## Capabilities & Access

- **Always check roles before commit:** Use `get_roles_with_capability()` to verify no unintended access loss/gain
- **Document capability changes** explicitly in commit messages
- **Upgrading capabilities?** Use `upgrade_rename_capability()` or explicit `$DB->delete_records()` in `db/upgrade.php`
  - Adding capabilities is safe
  - Renaming/deleting capabilities affects existing roles — add deprecation step if unsure
- **Every new `require_capability()` / `has_capability()`** must pass the role check (student, teacher, editingteacher, manager still have expected access)

## Database & Upgrades

- **Upgrade paths must be stable** — test rollback, document breaking changes
- **Never silently drop or rename columns** — use explicit migration steps
- **Every version bump** needs corresponding `db/upgrade.php` handler

## Theme Integration

- **Respect the installed Moodle theme** — don't hardcode colors, fonts, or layout
- **Use Moodle's theming API** (theme-aware CSS classes, settings, colors)
- **Test in default and a custom theme** before considering UI "done"

## Testing

- **PHPUnit required** for all business logic (DML queries, capability checks, data transforms)
- **Behat required** for user-facing workflows (forms, navigation, feature steps)
- **Test coverage** should reflect importance (critical paths ≥80%, nice-to-haves ≥50%)

## Uncertainty Rule

**If you are not >95% certain what to do, always ask clarifying questions first.** Do not guess. This applies to:
- Ambiguous feature requirements
- Capability implications
- Upgrade strategy
- Theme integration decisions
- Test coverage scope

Examples of when to ask:
- "Is this capability for teachers only, or teachers+managers?"
- "Should this setting migrate from old format, or reset for all users?"
- "Does this UI element need custom styling, or can it use standard Moodle components?"
