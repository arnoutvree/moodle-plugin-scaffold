---
type: context
created: 2026-06-26
updated: 2026-10-12
timestamp: 2026-06-26
---
# Testing — [FRANKENSTYLE]

This file describes the testing strategy for this plugin: PHPUnit unit tests, Behat acceptance tests, and test data generation.

---

## PHPUnit setup

**Initialization** (once per Moodle install). First set `$CFG->phpunit_prefix` and `$CFG->phpunit_dataroot` to values that differ from the live site in `config.php`; PHPUnit then runs in its own tables and data directory and does not touch the site data. Run `init.php` again after adding a new test class or generator; on Moodle 5.3 and later `util.php --upgrade` brings the test environment up to date without a full re-initialisation.
```bash
php public/admin/tool/phpunit/cli/init.php    # Moodle 5.1 and later (code lives in public/)
php admin/tool/phpunit/cli/init.php           # Moodle 5.0 and earlier
```

**Test file structure:**
```
[PLUGIN_MAPNAME]/tests/
├── [feature]_test.php              # Unit test file (suffix: _test)
├── generator/
│   └── lib.php                     # Test data generator (optional)
└── fixtures/
    └── [data].json                 # Test fixtures (optional)
```

**Minimal test class:**
```php
<?php
// This file is part of Moodle - https://moodle.org/
// (full GPL v3 header as in every other PHP file of the plugin)

namespace [FRANKENSTYLE];

/**
 * Test class for [feature].
 *
 * @package   [FRANKENSTYLE]
 * @copyright [YEAR] [YOUR ORG]
 * @license   http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 * @covers    \[FRANKENSTYLE]\[class_under_test]
 */
#[\PHPUnit\Framework\Attributes\CoversClass(\[FRANKENSTYLE]\[class_under_test]::class)]
final class [feature]_test extends \advanced_testcase {
    public function test_something_works(): void {
        $this->resetAfterTest();
        
        // Arrange
        $user = $this->getDataGenerator()->create_user();
        
        // Act
        $result = some_function($user->id);
        
        // Assert
        $this->assertTrue($result);
    }

    public function test_user_without_capability_is_denied(): void {
        $this->resetAfterTest();
        $this->setUser($this->getDataGenerator()->create_user());

        $this->expectException(\required_capability_exception::class);
        some_function_that_requires_a_capability();
    }
}
```

No `defined('MOODLE_INTERNAL') || die();` in a test file that only holds a class: the sniff `moodle.Files.MoodleInternal.MoodleInternalNotNeeded` rejects it (see `moodle-pitfalls.md`). The namespace is the component namespace (`[FRANKENSTYLE]`), not `...\tests`.

**Key points:**
- Extend `\advanced_testcase` for tests that need DB reset
- Extend `\basic_testcase` only for pure logic that touches no Moodle API at all; when in doubt use `\advanced_testcase`
- Always call `$this->resetAfterTest()` in tests touching the database
- Use data generator: `$this->getDataGenerator()->create_user()`, `create_course()`, etc.
- Use assertions: `$this->assertTrue()`, `$this->assertSame()`, `$this->assertEquals()`; expect failures with `$this->expectException(...)` (call it before the line that throws)
- Switch user with `$this->setUser()`, `$this->setAdminUser()` or `$this->setGuestUser()` to test allowed and denied cases

**Supporting Moodle 4.5 (PHPUnit 9.6) and 5.x (PHPUnit 11.x) at once:** put the PHPUnit attributes (`#[CoversClass(...)]`, `#[CoversNothing]`, `#[DataProvider('...')]`) **and** the doc-comment tags (`@covers`, `@dataProvider`) on the same class or method. PHPUnit 9.6 ignores attributes, so with attributes only the data providers hand over no arguments (`ArgumentCountError`) and the phpcs sniff `moodle.PHPUnit.TestCaseCovers` still looks for `@covers`. PHPUnit 11.x ignores the doc-comment tags as soon as an attribute is present, so both together give no deprecations.

**Keep test classes small.** phpmd flags `TooManyPublicMethods`/`TooManyMethods` in test classes with many scenarios. Decide at the start: either one test class per scenario group (small classes, no suppression), or one class per specification section (scenario numbering stays readable) with `@SuppressWarnings` in the class docblock. In the second case `moodle.Commenting.ValidTags.Invalid` rejects that tag, so wrap the class header in `phpcs:disable`/`phpcs:enable`.

**Run tests:** (Moodle 5.1 and later: plugin paths start with `public/`, for example `public/[PLUGIN_MAPNAME]/tests/[feature]_test.php`)
```bash
# All tests for this plugin (the testsuite exists after init.php/util.php --buildconfig)
vendor/bin/phpunit --testsuite [FRANKENSTYLE]_testsuite

# Single test class (path relative to the Moodle root)
vendor/bin/phpunit [PLUGIN_MAPNAME]/tests/[feature]_test.php

# Single test method
vendor/bin/phpunit --filter '[FRANKENSTYLE]\\[feature]_test::test_something_works'

# Show deprecations and notices from your own code (PHPUnit 10+, so Moodle 5.x; not available on 9.6 / Moodle 4.5)
vendor/bin/phpunit --testsuite [FRANKENSTYLE]_testsuite --display-notices --display-deprecations
```

---

## Behat acceptance testing

**Initialization** (once per Moodle install). Set `$CFG->behat_wwwroot`, `$CFG->behat_prefix` and `$CFG->behat_dataroot` in `config.php` first. Run `init.php` again after adding a feature file or step definition.
```bash
php public/admin/tool/behat/cli/init.php     # Moodle 5.1 and later
php admin/tool/behat/cli/init.php            # Moodle 5.0 and earlier
```

**Feature file structure:**
```
[PLUGIN_MAPNAME]/tests/behat/
├── [feature].feature               # Gherkin scenario
└── behat_[FRANKENSTYLE].php        # Step definitions (if custom steps needed)
```

**Example feature file:**
```gherkin
@[FRANKENSTYLE]
Feature: User can translate courses
  # Add the @javascript tag only to scenarios that really need a browser with JavaScript:
  # those are slower and less stable.
  In order to support multilingual content
  As a teacher
  I want to translate a course to another language

  Background:
    Given the following "courses" exist:
      | fullname | shortname |
      | My Course | testcourse |

  Scenario: Translate a course title
    Given I am on the "My Course" "course" page logged in as "admin"
    When I click on "Translate" "button"
    And I set the field "Language" to "Spanish"
    And I set the field "Translation" to "Mi Curso"
    And I press "Save"
    Then I should see "Translation saved successfully"
```

**Step definitions** (if you need custom steps beyond core Moodle):
```php
<?php
require_once(__DIR__ . '/../../../../lib/behat/behat_base.php');

/**
 * Step definitions for [FRANKENSTYLE]
 * @package   [FRANKENSTYLE]
 * @copyright [YEAR] [YOUR ORG]
 * @license   http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */
class behat_[FRANKENSTYLE] extends behat_base {
    /**
     * @Given I do something plugin-specific
     */
    public function i_do_something_plugin_specific(): void {
        // Step implementation
    }
}
```

**Run Behat tests:** (Moodle 5.0 and earlier: leave out `public/`)
```bash
# All scenarios tagged with plugin
php public/admin/tool/behat/cli/run.php --tags=@[FRANKENSTYLE]

# Specific feature file
php public/admin/tool/behat/cli/run.php [PLUGIN_MAPNAME]/tests/behat/[feature].feature

# Specific scenario
php public/admin/tool/behat/cli/run.php --tags=@[FRANKENSTYLE] --name="scenario name"
```

---

## Test data generation

For complex test data, create a **generator class:**

```php
<?php
// tests/generator/lib.php
// The class name must be exactly <component>_generator and must not be namespaced:
// get_plugin_generator('[FRANKENSTYLE]') looks for that class in this file.

class [FRANKENSTYLE]_generator extends \component_generator_base {
    public function create_item($options = []): \stdClass {
        global $DB;
        
        $defaults = [
            'userid' => get_admin()->id,
            'name' => 'Test Item',
            'status' => 'active',
        ];
        
        $record = (object)(array_merge($defaults, $options));
        $record->timecreated = time();
        $record->id = $DB->insert_record('[FRANKENSTYLE]_items', $record);
        
        return $record;
    }
}
```

Use it in tests:
```php
$item = $this->getDataGenerator()->get_plugin_generator('[FRANKENSTYLE]')->create_item([
    'userid' => $user->id,
    'name' => 'Custom Item',
]);
```

---

## Fixtures per test scenario (traceability back to specs.md)

Each test scenario in `user-stories.md` (Given/When/Then) gets its own generator method, identifiable by the scenario ID — so test code can be traced back to the scenario without grepping, and the task's score in `tasks.md` updated once the test passes (see [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development), Phase 2c/4c, if you're following that process).

**Naming:** `scenario_<number>_<slug>()`, e.g. `scenario_1_1_enrolment_still_active()` for test scenario 1.1.

**Set up via a generator/direct database insertion, never via UI navigation.** Both a PHPUnit fixture (generator method) and a Behat `Background:` follow the same rule: the Given steps set up the state directly, not by clicking through the UI the scenario is actually testing. Moodle's own Behat guidance is explicit about this: "The setup is not what you are really testing here" — Given steps should therefore prefer direct steps (setting config, creating users/courses) over step-by-step navigation through the interface.

**Pattern:** the scenario method sets up only the Given state and returns the involved objects; the test itself performs the When and asserts the Then — including what must *not* happen.

```php
<?php
// tests/generator/lib.php (class <component>_generator, no namespace)

class [FRANKENSTYLE]_generator extends \component_generator_base {

    /**
     * Test scenario 1.1 — Active enrolment, no completion yet (user-stories.md, US1)
     * Given: an active course enrolment with no completion record
     */
    public function scenario_1_1_enrolment_still_active(): \stdClass {
        $course = $this->create_item(['shortname' => 'testcourse', 'status' => 'active']);
        $enrolment = $this->create_enrolment(['userid' => 2, 'courseid' => $course->id]);

        return (object) compact('course', 'enrolment');
    }
}
```

```php
public function test_scenario_1_1_enrolment_still_active(): void {
    $this->resetAfterTest();
    $this->getDataGenerator()->get_plugin_generator('[FRANKENSTYLE]')
        ->scenario_1_1_enrolment_still_active();

    // When
    process_pending_completions();

    // Then — including "and not"
    $this->assertEquals('in_progress', get_status_for(2));
    $this->assertNotEquals('completed', get_status_for(2));
}
```

**Traceability table** — illustrates the fixture ↔ test pattern. The authoritative progress lives in the score table in `tasks.md` (Phase 4c) — update that once a scenario is implemented here, not this standalone illustration:

| Test scenario (user-stories.md) | Fixture | Test |
|---|---|---|
| 1.1 Enrolment still active | `scenario_1_1_enrolment_still_active()` | `[feature]_test.php::test_scenario_1_1_enrolment_still_active` |

The same principle applies to a Behat scenario (user-facing workflow), but via a `Background:` block with concrete data instead of a generator method — the Given lines in the `.feature` file itself are then the fixture.

**Gherkin discipline:** use `Given`, `When` and `Then` exactly once per scenario; additional steps within the same phase go via `And`/`But`. This is Moodle's own Behat convention and ties directly into the "one scenario per outcome" rule — a scenario with a second `When` is effectively testing two things at once.

---

## Definition of done (for the agent)

A task that changes code is only done when the checks below have been **run and passed**, and your reply states the result of each one. Saying "done" without output is not allowed. If a check still fails after two fix attempts, stop and report it.

| Check | When | Blocks |
|---|---|---|
| `moodle-plugin-ci phplint` | always, for PHP changes | yes |
| `moodle-plugin-ci phpcs` (fix first with `phpcbf`) | always, for code changes | yes, 0 errors |
| `moodle-plugin-ci validate` | always, especially after changing `version.php` or metadata | yes |
| `moodle-plugin-ci savepoints` | when `db/upgrade.php` or `version.php` changed | yes |
| `moodle-plugin-ci phpmd` | after larger changes or refactors; ignore parser errors on valid PHP | no, review the findings |
| `moodle-plugin-ci phpdoc` | for new or changed functions and classes | yes |
| PHPUnit | for new functionality, bug fixes and refactors | yes |
| Behat | for new or changed user flows | yes |
| `moodle-plugin-ci grunt` / `mustache` | when `amd/src/`, CSS/SCSS or `templates/` changed | yes |

**This is the only authoritative list.** The pre-release checklist and the CI workflow point here; they do not keep a list of their own.

**Where it runs:** all `moodle-plugin-ci` commands expect the Moodle checkout that `moodle-plugin-ci install` creates (in CI) or a local test site; use that, not a bare git clone. On a bare clone only `php -l` and a standalone `phpcs --standard=moodle` (from `moodlehq/moodle-cs`) work. Report such a limited check as incomplete (see below).

**No test environment or CI tools available?** Do not skip checks silently. Report which checks could not run and why, run at least `php -l` on the changed files, and say the result is only partly validated. A skipped check is never a passed check.

**Order** (fast to slow, so you find problems early): `phplint` and `validate`, then `phpcs`, then `phpmd`, then `phpunit`, then `behat`.

There is no fixed allowed-warnings threshold: state any deliberately accepted warning and its reason in your reply, or add a `phpcs:ignore` with an explanation.

**Test against more than one Moodle version:** run the plugin against at least the lowest and highest version it supports (see `requires` in `version.php`), preferably 4.5 and 5.x.

---

## Coverage expectations

**Coverage is a signal, not a gate.** One rule for all documents:

- **Required (blocks):** every test scenario in `user-stories.md` has a test, sad path included, and every component in the table below has at least one test.
- **Target (does not block):** 80% line coverage of the non-trivial code in `classes/`. Measure it with `moodle-plugin-ci phpunit --coverage-text` (needs pcov or xdebug). A lower number is a reason to look at which paths are untested, not to refuse.
- **Never:** write tests just to reach the number. A test that asserts nothing does not count.

| Component | Test type |
|---|---|
| Service classes (managers, processors) | PHPUnit |
| External API functions | PHPUnit |
| Database operations | PHPUnit with `resetAfterTest()` |
| User-facing workflows | Behat (e.g., "admin translates a course") |
| Permission checks | PHPUnit (test both allowed and denied cases) |
| Events and observers | PHPUnit |

Dead code (unused imports, conditions that can never be reached) needs no coverage; remove it instead.

---

## CI

Use `moodlehq/moodle-plugin-ci` rather than invoking PHPUnit/Behat directly in CI — it handles spinning up a full Moodle install against your plugin first, which bare `vendor/bin/phpunit` cannot do standalone. Run the checks from the table above in CI. See [context/workflows/release-process.md](../workflows/release-process.md) (CI/CD section) for the complete workflow.

---
