---
type: architecture
created: 2026-06-26
timestamp: 2026-06-26
---
# Architecture — [FRANKENSTYLE]

This file describes the complete folder structure of the `[PLUGIN_MAPNAME]` plugin, entry points, bootstrap sequence, and naming conventions.

---

## Plugin folder location and structure

The plugin code lives in a single folder:
```
[PLUGIN_MAPNAME]/
├── version.php                        # Mandatory: version metadata
├── lib.php                            # Optional: callbacks only (create if needed)
├── settings.php                       # Optional: admin settings
├── classes/
│   ├── output/
│   │   ├── renderer.php               # Renderer subclass (extends plugin_renderer_base)
│   │   └── [name]_renderable.php      # Renderable + templatable classes
│   ├── event/
│   │   └── [name]_created.php         # Event classes (optional)
│   ├── task/
│   │   ├── scheduled_task.php         # Scheduled task (optional)
│   │   └── adhoc_task.php             # Adhoc task (optional)
│   ├── external/
│   │   └── [name].php                 # External function classes (optional)
│   └── privacy/
│       └── provider.php               # Privacy API (mandatory since Moodle 3.4)
├── db/
│   ├── access.php                     # Capabilities (mandatory if plugin grants permissions)
│   ├── install.xml                    # Table definitions (create via XMLDB editor)
│   ├── install.php                    # Post-install hook (optional)
│   ├── upgrade.php                    # Version migrations (mandatory if install.xml exists)
│   ├── events.php                     # Event observers (optional)
│   ├── services.php                   # Web service definitions (optional)
│   └── tasks.php                      # Scheduled task registration (optional)
├── lang/
│   └── en/
│       └── [FRANKENSTYLE].php         # English language file (mandatory)
├── templates/
│   └── [name].mustache                # Mustache templates (optional)
├── amd/
│   └── src/
│       └── [name].js                  # AMD modules (optional, build to amd/build/)
├── tests/
│   ├── [name]_test.php                # PHPUnit tests
│   ├── behat/
│   │   └── [name].feature             # Behat feature files
│   └── generator/
│       └── lib.php                    # Test data generator (optional)
└── pix/
    ├── icon.svg                       # Plugin icon (scalable)
    └── icon.png                       # Plugin icon (fallback)
```

---

## Core entry points and load sequence

1. **version.php** — Moodle reads this first. Must contain `$plugin->component` matching Frankenstyle, `$plugin->version`, `$plugin->requires`.
2. **lib.php** — Loaded automatically if it exists. Contains callbacks like `[FRANKENSTYLE]_extend_navigation()`, `[FRANKENSTYLE]_get_string_manager()`, etc.
3. **settings.php** — Loaded only when admin visits Settings > Plugins > [Type]. Use for admin configuration UI.
4. **db/install.xml** — Processed once during plugin installation. Tables are created and versioned.
5. **db/upgrade.php** — Runs every time Moodle boots and detects a version mismatch. Always wraps changes in `if ($oldversion < YYYYMMDDXX)` blocks.

---

## Namespace and naming conventions

All classes live in **PSR-4 namespaces** matching Frankenstyle:

```php
// Example for local_translatecourse:
namespace local_translatecourse;
namespace local_translatecourse\output;
namespace local_translatecourse\event;
namespace local_translatecourse\task;
namespace local_translatecourse\external;
namespace local_translatecourse\privacy;
```

All **functions outside classes** are prefixed with Frankenstyle:

```php
// lib.php example
function local_translatecourse_extend_navigation(\navigation_node $navnode) { ... }
function local_translatecourse_pluginfile($course, $cm, $context, $filearea, $args, $forcedownload, array $options = []) { ... }
```

All **database tables** use Frankenstyle prefix (Moodle adds `mdl_` automatically):

```
install.xml: <TABLE NAME="local_translatecourse_courses">
Query result: $DB->get_record('local_translatecourse_courses', ...)
```

All **capabilities** use Frankenstyle prefix:

```php
// db/access.php
'[FRANKENSTYLE]:view' => [
    'riskbitmask' => RISK_XSS,
    'captype' => 'view',
    'contextlevel' => CONTEXT_COURSE,
    'archetypes' => [
        'editingteacher' => CAP_ALLOW,
        'student' => CAP_ALLOW,
    ],
],
```

---

## When code is loaded

| File | When loaded |
|---|---|
| `version.php` | Always (Moodle detects all plugins) |
| `lib.php` | Always (if it exists) |
| `settings.php` | Only when admin visits admin UI |
| `classes/privacy/provider.php` | When privacy policy is generated or data deletion runs |
| `classes/event/*.php` | Only when events are triggered or observers listen |
| `classes/task/*.php` | Only when scheduled tasks or adhoc tasks run |
| `db/access.php` | When permission checks happen (every page load via role/capability cache) |
| `db/install.xml` | Once during install; never again unless table deleted |
| `db/upgrade.php` | Every boot if `version.php` version > database version |
| `amd/src/*.js` | Only if module is loaded via `$PAGE->requires->js_call_amd()` |
| `templates/*.mustache` | Only when code calls `$OUTPUT->render()` with renderable object |

---

## Heads-up: `/public` doc-root and Routing Engine (Moodle 5.1+)

Newer Moodle majors introduced a `/public` web doc-root reorganization and a new Routing Engine that changes how requests reach entry-point pages. **Don't assume the classic `require_once('../../config.php')` entry-point pattern is unchanged** if this plugin targets Moodle 5.1 or later — verify against the current [Upgrade API docs](https://moodledev.io/docs/5.0/guides/upgrade) and the deprecation policy page for the specific target version before relying on file-layout assumptions in this document written for older majors.

---

## Wizard entry point with navigation toggles

For plugins whose primary interface is a standalone UI page (e.g. a wizard) rather than a block or activity page, a common structure is:

1. **Entry-point page** — `pages/index.php`, a standard Moodle page with `require_once('../../../config.php')`, `require_login()`, and a capability check before rendering.
2. **Navigation hooks in `lib.php`** register links to that page instead of hardcoding them into a theme or menu:
   - `[FRANKENSTYLE]_extend_settings_navigation()` — adds a link under Course Administration.
   - `[FRANKENSTYLE]_extend_navigation_course()` — adds a link into the course navigation.
3. **Per-link admin toggle** — gate each navigation hook behind its own admin setting (e.g. `show_in_admin`, `show_in_course_nav`) so visibility can be changed without a code deploy, independent of the capability check that still gates the page itself.

This keeps "is the feature visible here" (admin setting) separate from "is the user allowed to use it" (capability) — both checks apply, but they're configured in different places for different reasons.

---

## Key bootstrap principle

Moodle loads `config.php` → discovers all plugins via `version.php` → auto-includes `lib.php` if present → processes requests. Your plugin receives control via:

1. **Callbacks** in `lib.php` (if you implement them)
2. **Entry-point pages** you create (e.g., `index.php`, `view.php`)
3. **Web services** you register in `db/services.php`
4. **Scheduled tasks** in `db/tasks.php`
5. **Events** you observe in `db/events.php`

Never use global `include` or `require` to load plugin code outside these mechanisms — Moodle's autoloader handles it.

---
