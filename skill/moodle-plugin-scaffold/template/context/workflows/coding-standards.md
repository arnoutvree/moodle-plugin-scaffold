---
type: process
created: 2026-06-26
timestamp: 2026-06-26
---
# Coding standards — [FRANKENSTYLE]

This file describes the mandatory coding style, naming conventions, documentation, and commit discipline for this plugin.

---

## PHP — Moodle Coding Standards

All PHP code follows the **Moodle PHP Coding Standards** (PSR-12 derivative):
https://moodledev.io/general/development/policies/codingstyle

### Formatting

```php
<?php
// Always start with open tag + docblock
// Never include closing tag at end of file

defined('MOODLE_INTERNAL') || die();

// Indentation: 4 spaces (never tabs)
// Brace on same line:

if ($condition) {
    do_something();
} else {
    do_something_else();
}

// Class brace on new line:
class MyClass {
    // ...
}

// Method brace on same line:
public function do_something() {
    // ...
}

// Long lines: break at 120 chars (soft limit, not hard)
$result = some_function_with_long_name(
    $argument1,
    $argument2,
    $argument3
);
```

### Naming conventions

| Element | Format | Example |
|---|---|---|
| **Classes** | PascalCase | `class ItemRenderer { }` |
| **Namespaces** | lowercase with backslash | `namespace [FRANKENSTYLE]\output;` |
| **Methods** | camelCase | `public function render_item() { }` |
| **Properties** | camelCase | `private $item_list;` |
| **Variables** | snake_case | `$course_id`, `$user_name` |
| **Constants** | UPPERCASE | `const MAX_ITEMS = 100;` |
| **Functions** | snake_case | `function [FRANKENSTYLE]_extend_navigation() { }` |
| **Database tables** | `[FRANKENSTYLE]_tablename` | `mdl_[FRANKENSTYLE]_items` |
| **Capabilities** | `[FRANKENSTYLE]:action` | `[FRANKENSTYLE]:view`, `[FRANKENSTYLE]:edit` |

### Spacing and brackets

```php
// Array syntax: use [] not array()
$items = [1, 2, 3];
$config = ['key' => 'value'];

// No space before parentheses in control structures
if ($x) { }      // Good
if ($x) { }      // Good
if( $x ) { }     // Bad

// Space after comma in lists
foo($a, $b, $c);  // Good
foo($a,$b,$c);    // Bad

// No space inside brackets
$arr[0];          // Good
$arr[ 0 ];        // Bad
```

---

## JavaScript — AMD modules

All JavaScript uses **Asynchronous Module Definition (AMD)** via Moodle's loader.

### Source file structure

```javascript
// File: amd/src/mymodule.js
define(['jquery', 'core/ajax'], function($, Ajax) {
    // Module definition
    
    return {
        init: function() {
            console.log('Module initialized');
        },
        
        doSomething: function(param) {
            return param * 2;
        }
    };
});
```

### Build process

```bash
# AMD modules in amd/src/ are built to amd/build/
npx grunt amd
# or
npm run amd

# Built output is minified and ready for production
```

### Naming

- Module names: lowercase with underscores (`my_module.js`)
- Functions: camelCase (`doSomething()`)
- Constants: UPPERCASE

### No globals

```javascript
// Bad: creates global
var myGlobal = 5;

// Good: scoped to module
define(['jquery'], function($) {
    let myScoped = 5;
    return { getValue: () => myScoped };
});
```

---

## CSS / SCSS — BEM naming

All styles prefixed with plugin name to avoid conflicts with Moodle core.

```css
/* BEM: Block__Element--Modifier */

.[FRANKENSTYLE]-container { }
.[FRANKENSTYLE]-container__title { }
.[FRANKENSTYLE]-container__title--emphasized { }
.[FRANKENSTYLE]-container__item { }
.[FRANKENSTYLE]-container__item--disabled { }
```

---

## Mustache templates

All user-facing text via language strings. No hardcoded text.

```mustache
<!-- Good: translatable -->
<h3>{{#str}}title, [FRANKENSTYLE]{{/str}}</h3>
<p>{{description}}</p>

<!-- Bad: hardcoded text -->
<h3>Title (Not Translatable)</h3>
<p>{{description}}</p>

<!-- Conditional with string -->
{{#candelete}}
    <button>{{#str}}delete, [FRANKENSTYLE]{{/str}}</button>
{{/candelete}}
```

---

## PHPDoc — mandatory for classes and public methods

```php
<?php
/**
 * ItemRenderer class - renders course items to HTML.
 *
 * @package   [FRANKENSTYLE]
 * @copyright (c) 2025 Your Organization
 * @license   http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */
class ItemRenderer {
    
    /**
     * Render a single item.
     *
     * @param object $item The item object
     * @param context $context The context object
     * @return string HTML output
     * @throws moodle_exception If item not found
     */
    public function render_item($item, $context) {
        // ...
    }
    
    /**
     * Get item count.
     *
     * @return int Number of items
     */
    private function get_item_count() {
        // Private methods don't need PHPDoc unless complex
    }
}
```

### Minimum documentation

| Element | Requires PHPDoc |
|---|---|
| Classes | Yes |
| Public methods | Yes |
| Protected methods | Yes (complex logic) |
| Private methods | No (unless complex) |
| Properties | No (obvious names are enough) |
| Global functions | Yes |

---

## Code clarity & comments

Code should be readable and understandable by others without convoluted constructions.

### Principles

- **Simplicity over cleverness:** code should be easy to follow. No obscure constructions, no "clever" tricks.
- **Functional naming:** functions, variables, and classes should make clear what they do.
- **Comments where it matters:** put comments where the **WHY** isn't obvious from the code itself.
  - No comments that repeat what the code does (`// increment counter` above `$count++`)
  - Do comment on **why** something is needed, or a hidden constraint

### Examples

**Good — clear and simple:**
```php
// Get all courses where the user is enrolled and has not yet completed
public function get_incomplete_enrollments($user_id) {
    $sql = "SELECT c.* FROM {course} c
            JOIN {enrol} e ON e.courseid = c.id
            WHERE e.userid = ? AND c.completion_enabled = 1";
    return $DB->get_records_sql($sql, [$user_id]);
}
```

**Bad — unclear and overengineered:**
```php
// Bizarre indirection, unclear logic
private function _gtcie($uid) {
    $cs = [];
    foreach (self::$_ecdata as $e) {
        if (($e['u'] ?? null) == $uid && !($e['c'] ?? 0)) {
            $cs[] = $e['cid'];
        }
    }
    return $cs;
}
```

---

## Linting

**PHP:**
```bash
# Check style
./vendor/bin/phpcs [PLUGIN_MAPNAME]/

# Auto-fix some issues
./vendor/bin/phpcbf [PLUGIN_MAPNAME]/
```

**JavaScript:**
```bash
npm run lint
# or
npx eslint amd/src/
```

---

## Static analysis — moodle-plugin-ci (the standard toolchain)

Don't hand-roll separate phpcs/phpstan/phpmd invocations — [`moodlehq/moodle-plugin-ci`](https://github.com/moodlehq/moodle-plugin-ci) bundles the whole Moodle-specific quality toolchain (Moodle Coding Standard via phpcs, PHP Mess Detector, PHPDoc checker, PHPStan/Psalm-equivalent static checks, PHPUnit, Behat, grunt/eslint) and is what Moodle core's own CI and most maintained community plugins use.

```bash
# Install once
composer create-project -n --no-dev --ignore-platform-reqs moodlehq/moodle-plugin-ci ci ^4

# Run the full suite against this plugin
moodle-plugin-ci phplint
moodle-plugin-ci phpcs
moodle-plugin-ci phpmd
moodle-plugin-ci phpdoc
moodle-plugin-ci validate
moodle-plugin-ci phpunit
moodle-plugin-ci behat
```

Start with `phpcs`/`phplint`/`validate` passing cleanly before adding `phpunit`/`behat` to a CI gate — see [context/workflows/release-process.md](release-process.md) for a ready-to-use GitHub Actions workflow.

---

## PHP 8.2 / 8.3 compatibility — common breakage points

Moodle's own PHP floor moves forward with each LTS (see [context/software/moodle.md](../software/moodle.md)). Watch for these when writing new code or upgrading an existing plugin to a newer Moodle/PHP floor:

- **Dynamic properties are deprecated (PHP 8.2+).** Declare all class properties explicitly; Moodle core classes that still rely on dynamic properties are being migrated over time, but plugin code shouldn't add new instances of the pattern.
- **`${var}` string interpolation is deprecated (8.2+)** — use `{$var}` instead.
- **`utf8_encode()`/`utf8_decode()` are deprecated (8.2+)** — use `mb_convert_encoding()` or Moodle's `core_text` class.
- Prefer PHP `enum` (8.1+) over class-constant groups for fixed value sets in new code (e.g. status values), where Moodle core conventions don't already dictate a different pattern.
- Run `moodle-plugin-ci phpcs` with the coding standard matched to the plugin's actual minimum-supported Moodle/PHP combination — it catches most of these automatically.

---

## Commit discipline

### Commit message format

```
Short summary (under 70 chars)

Detailed explanation of what changed and why.
Explain the reasoning, not the mechanics.

If related to an issue, mention it:
Fixes #123
Relates to #456
```

### Examples

**Good:**
```
Add external function to fetch course items

Students and teachers need to retrieve a list of items
from their course via the REST API. This adds a new
external function and registers it in db/services.php.
```

**Bad:**
```
Update function

Added stuff
```

### What to include in commits

- [x] Feature code (classes, functions)
- [x] Associated language strings
- [x] PHPUnit tests
- [x] Behat tests (if UI-affecting)
- [x] `version.php` bump (if database or capability change)
- [x] `db/upgrade.php` step (if schema or capability change)

### What NOT to commit

- [ ] IDE config files (.idea/, .vscode/)
- [ ] OS files (.DS_Store)
- [ ] node_modules/
- [ ] build outputs (amd/build/)
- [ ] test fixtures or temp data
- [ ] .env files or credentials

---

## Testing discipline

### PHPUnit

```php
<?php
namespace [FRANKENSTYLE]\tests;

class item_manager_test extends \advanced_testcase {
    public function test_create_item(): void {
        $this->resetAfterTest();
        
        $manager = new \[FRANKENSTYLE]\item_manager();
        $item = $manager->create_item(['name' => 'Test']);
        
        $this->assertIsObject($item);
        $this->assertEquals('Test', $item->name);
    }
}
```

**Minimum coverage:** 80% of non-trivial code paths.

### Behat

```gherkin
@[FRANKENSTYLE] @plugin
Feature: Manage course items

  Scenario: Teacher creates an item
    Given I log in as "teacher1"
    When I visit the items page
    And I click "Create item"
    And I fill "Name" with "New Item"
    And I click "Save"
    Then I should see "Item created successfully"
```

---

## Dead code removal

If a function, class, or import becomes unused:

```php
// Old: used once, now removed
function old_function_no_longer_needed() { }

// Action: delete it entirely
// Don't leave it with a comment or rename it
```

**Exception:** If you discover dead code unrelated to your task, mention it in a comment but don't delete it unless explicitly asked.

---

## Deprecation and migration

When deprecating a function:

```php
/**
 * Get item list.
 *
 * @deprecated since version 2.0.0, use get_items() instead.
 * @return array|null
 */
function legacy_get_item_list() {
    debugging('legacy_get_item_list() is deprecated, use get_items()', DEBUG_DEVELOPER);
    return get_items();
}
```

Always provide a migration path and give users at least one minor version to update.

---

## Code review checklist

Before pushing, ask:

- [ ] Does this follow Moodle Coding Standards?
- [ ] Are all user-facing strings in lang files?
- [ ] Is there PHPDoc on public methods?
- [ ] Are there tests?
- [ ] Did I verify role/capability impact?
- [ ] Is the commit message clear?
- [ ] Are there any debug statements left?
- [ ] Does this work with the minimum supported Moodle version?

---
