---
type: reference
created: 2026-06-26
timestamp: 2026-06-26
---
# Moodle — official references

This file is a permanent lookup table for official Moodle documentation, tools, version support matrix, and policy documents. **Read this whenever you're unsure about Moodle conventions, naming, security, or available tooling.**

---

## Official documentation links

| Resource | URL |
|---|---|
| Plugin types | https://moodledev.io/docs/5.0/apis/plugintypes |
| Common files | https://moodledev.io/docs/5.0/apis/commonfiles |
| Coding style guide | https://moodledev.io/general/development/policies/codingstyle |
| Frankenstyle naming | https://moodledev.io/general/development/policies/codingstyle/frankenstyle |
| Security guidelines | https://moodledev.io/general/development/policies/security |
| GitHub Actions CI | https://moodledev.io/general/development/tools/gha |
| XMLDB editor | https://moodledev.io/general/development/tools/xmldb |
| Upgrade API | https://moodledev.io/docs/5.0/guides/upgrade |
| Privacy API | https://moodledev.io/docs/5.0/apis/subsystems/privacy |
| Output API | https://moodledev.io/docs/5.0/apis/subsystems/output |
| Events system | https://moodledev.io/docs/5.0/apis/subsystems/events |
| Forms API | https://moodledev.io/docs/5.0/apis/subsystems/form |
| External services | https://moodledev.io/docs/5.0/apis/subsystems/external/writing-a-service |
| AMOS translations | https://lang.moodle.org/ |
| Moodle Plugins Directory | https://moodle.org/plugins/ |
| Plugin contribution checklist | https://moodledev.io/general/community/plugincontribution/checklist |
| Moodle Dev Kit (mdk) | https://github.com/FMCorz/mdk |
| Releases & support matrix | https://moodledev.io/general/releases |
| Deprecation policy | https://moodledev.io/docs/5.0/guides/deprecation |

---

## Moodle version matrix — check before filling in

**The numbers below are illustrative, not current — do not copy them into a real project.** Moodle ships roughly 2 major releases per year (a new LTS every ~2 years) and each carries its own PHP/database floor, so a hardcoded table goes stale within months. Before starting a new project or a major upgrade:

1. Check [moodledev.io/general/releases](https://moodledev.io/general/releases) for the current release, the current LTS, and their PHP/MySQL/MariaDB/PostgreSQL requirements.
2. Check which listed versions are already **security-only** or **EOL** — never target one of those as a floor for new development.
3. Fill in the table below for this specific project and keep it updated at each major bump.

| Version | PHP min | PHP max | MySQL min | MariaDB min | PostgreSQL min | Status | Support until |
|---|---|---|---|---|---|---|---|
| [fill in — current LTS] | | | | | | | |
| [fill in — current release, if different] | | | | | | | |

**Recommendation:** target the current LTS release as the floor unless there's a specific reason to support an older one.

**Doc links above point at a specific major's docs path** (`moodledev.io/docs/5.0/...`). When that major goes EOL, re-check whether the docs site has moved that content to a newer version path — Moodle's docs are versioned per major, and old paths sometimes redirect or go stale.

---

## Plugin types and required files

| Type | Directory | Required files | Entry point |
|---|---|---|---|
| `local` | `local/[name]/` | `version.php`, `lib.php` (optional) | `lib.php` hooks |
| `mod` | `mod/[name]/` | `version.php`, `lib.php`, `mod_form.php` | Activity instance |
| `block` | `blocks/[name]/` | `version.php`, `block_[name].php` | Block rendering |
| `auth` | `auth/[name]/` | `version.php`, `auth.php` | Login flow |
| `theme` | `theme/[name]/` | `version.php`, `config.php` | Rendering |
| `report` | `report/[name]/` | `version.php`, `index.php` | Report page |
| `tool` | `admin/tool/[name]/` | `version.php`, `cli/*.php` | CLI or admin |
| `enrol` | `enrol/[name]/` | `version.php`, `lib.php` | Enrollment logic |
| `filter` | `filter/[name]/` | `version.php`, `filter.php` | Text filtering hook |
| `tiny` | `lib/editor/tiny/plugins/[name]/` | `version.php`, `classes/plugininfo.php` | TinyMCE editor button/menu |
| `qtype` | `question/type/[name]/` | `version.php`, `questiontype.php`, `question.php` | Quiz question type |
| `qbank` | `question/bank/[name]/` | `version.php`, `classes/plugin_feature.php` | Question bank feature (column/action/report) |

**Notes on the less common types above:**

- **`filter`** — class `filter_[name]` extends `moodle_text_filter`, implements `filter($text, array $options)`. Runs on every rendered piece of content site-wide, so keep the regex/replace logic cheap and cache aggressively (`cache/definitions.php`) — a slow filter is a global performance hit, not a local one.
- **`tiny`** — Moodle 4.1+ TinyMCE integration (not the legacy `atto`/`tinymce` plugin types). `classes/plugininfo.php` extends `\editor_tiny\plugin` and implements the relevant interfaces (`is_enabled`, `get_plugin_configuration_for_context`, etc.); JS lives in `amd/src/` per the standard AMD module pattern. Check `moodledev.io/docs/5.0/apis/subsystems/tiny` for the current interface set — it's changed across 4.x releases.
- **`qtype`** — `questiontype.php` extends `question_type`, `question.php` extends `question_definition` (or a behaviour-specific subclass). Grading logic must be deterministic and side-effect-free — it can run in a sandboxed/replayed context (e.g. regrade). Never trust submitted response data without validating against the question's own defined answer structure first.
- **`qbank`** — `classes/plugin_feature.php` extends `\core_question\local\bank\plugin_features_base`; register the specific feature (extra column, bulk action, report) via the relevant `get_*_classes()` method. These run inside the question bank's existing list/filter UI, so match its UX conventions rather than introducing new patterns.

---

## Security checklist

Always follow:

1. **Input validation** — use `required_param()`, `optional_param()`, `validate_parameters()`
2. **Output escaping** — use `s()`, `format_string()`, `format_text()`, Mustache
3. **SQL injection prevention** — use `$DB` DML, named parameters, no concatenation
4. **CSRF protection** — use `sesskey()`, `confirm_sesskey()` for state changes
5. **Capability checks** — use `require_capability()`, `has_capability()`
6. **Credentials** — never store in code, use `get_config()` / `set_config()`
7. **Deprecation** — check https://moodledev.io/docs/5.0/guides/deprecation before using functions

---

## Maturity levels

| Level | When to use |
|---|---|
| `MATURITY_ALPHA` | Experimental, breaking changes expected |
| `MATURITY_BETA` | Feature-complete but untested in production |
| `MATURITY_RC` | Release candidate, likely stable |
| `MATURITY_STABLE` | Production-ready, backward-compatible |

---

## Version numbering convention

**`$plugin->version` format: `YYYYMMDDXX`**

- `YYYY` = year (e.g., 2025)
- `MM` = month (01-12)
- `DD` = day (01-31)
- `XX` = sequence within the day (00, 01, 02, ...)

Examples:
- `2025052000` = May 20, 2025, release 00
- `2025052001` = May 20, 2025, release 01 (second release that day)

**`$plugin->release` format: semantic versioning (e.g., `1.0.0`, `2.1.3`)**

---

## Frankenstyle naming

Format: `[TYPE]_[NAME]`

Examples:
- `local_translatecourse` (local plugin)
- `mod_customquiz` (activity module)
- `block_dashboard` (block)
- `report_engagement` (report)
- `auth_saml2` (authentication)

**Rules:**
- Always lowercase
- Underscores between type and name
- Use this prefix for classes, tables, capabilities, functions, constants

---

## PHPDoc requirements

```php
<?php
/**
 * MyClass description.
 *
 * @package   [FRANKENSTYLE]
 * @copyright (c) 2025 [Organization]
 * @license   http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */
class MyClass {
    /**
     * Do something important.
     *
     * @param int $userid The user ID
     * @param string $action The action to perform
     * @return bool True if successful
     * @throws moodle_exception If the action fails
     */
    public function do_something($userid, $action) {
        // ...
    }
}
```

---

## Testing infrastructure

**PHPUnit:** Unit and integration tests
```bash
vendor/bin/phpunit --filter [FRANKENSTYLE]
```

**Behat:** Acceptance tests (browser-based)
```bash
php admin/tool/behat/cli/run.php --tags=@[FRANKENSTYLE]
```

**Linting:** Code quality
```bash
mdk phpcs [FRANKENSTYLE]
# or
./vendor/bin/phpcs [plugin-dir]
```

---

## Moodle Dev Kit (mdk)

Optional command-line tool for plugin development:
```bash
mdk init                    # Set up dev environment
mdk run                     # Start Moodle
mdk phpcs                   # Lint code
mdk phpunit                 # Run tests
```

See: https://github.com/FMCorz/mdk

---

## Plugin submission checklist

Before uploading to moodle.org/plugins:

- [ ] All code follows Moodle PHP Coding Standards
- [ ] Every PHP file starts with `defined('MOODLE_INTERNAL') || die();` (except entry points)
- [ ] `lang/en/[FRANKENSTYLE].php` is complete and contains all user-facing strings
- [ ] No external network requests without user/admin opt-in
- [ ] Privacy API implemented (at least null provider)
- [ ] Version number matches `YYYYMMDDXX` format
- [ ] GPLv3 license header in all files
- [ ] Tests exist (PHPUnit minimum)
- [ ] Plugin works with supported Moodle versions (verify with `$plugin->requires` and `$plugin->supported`)
- [ ] No TODO, FIXME, or debug code left in production files

---
