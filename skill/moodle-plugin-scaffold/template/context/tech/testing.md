---
type: context
created: 2026-06-26
timestamp: 2026-06-26
---
# Testing — [FRANKENSTYLE]

This file describes the testing strategy for this plugin: PHPUnit unit tests, Behat acceptance tests, and test data generation.

---

## PHPUnit setup

**Initialization** (once per Moodle install):
```bash
php admin/tool/phpunit/cli/init.php
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
namespace [FRANKENSTYLE]\tests;

defined('MOODLE_INTERNAL') || die();

/**
 * Test class for [feature].
 * @package   [FRANKENSTYLE]
 * @copyright (c) [YEAR] [YOUR ORG]
 * @license   http://www.gnu.org/copyleft/gpl.html GNU GPL v3 or later
 */
class [feature]_test extends \advanced_testcase {
    public function test_something_works(): void {
        $this->resetAfterTest();
        
        // Arrange
        $user = $this->getDataGenerator()->create_user();
        
        // Act
        $result = some_function($user->id);
        
        // Assert
        $this->assertTrue($result);
    }
}
```

**Key points:**
- Extend `\advanced_testcase` for tests that need DB reset
- Extend `\basic_testcase` for pure unit tests (no DB)
- Always call `$this->resetAfterTest()` in tests touching the database
- Use data generator: `$this->getDataGenerator()->create_user()`, `create_course()`, etc.
- Use assertions: `$this->assertTrue()`, `$this->assertEquals()`, `$this->assertThrows()`

**Run tests:**
```bash
# All tests for this plugin
vendor/bin/phpunit --filter [FRANKENSTYLE]

# Single test class
vendor/bin/phpunit [FRANKENSTYLE]/tests/[feature]_test.php

# Single test method
vendor/bin/phpunit --filter [FRANKENSTYLE]/[feature]_test::test_something_works
```

---

## Behat acceptance testing

**Initialization** (once per Moodle install):
```bash
php admin/tool/behat/cli/init.php
```

**Feature file structure:**
```
[PLUGIN_MAPNAME]/tests/behat/
├── [feature].feature               # Gherkin scenario
└── behat_[FRANKENSTYLE].php        # Step definitions (if custom steps needed)
```

**Example feature file:**
```gherkin
@[FRANKENSTYLE] @plugin
Feature: User can translate courses
  In order to support multilingual content
  As a teacher
  I want to translate a course to another language

  Background:
    Given the following "courses" exist:
      | fullname | shortname |
      | My Course | testcourse |

  Scenario: Translate a course title
    Given I log in as "admin"
    And I navigate to course "My Course"
    When I click on "Translate" "button"
    And I fill the field "Language" with "Spanish"
    And I fill the field "Translation" with "Mi Curso"
    And I click on "Save" "button"
    Then I should see "Translation saved successfully"
```

**Step definitions** (if you need custom steps beyond core Moodle):
```php
<?php
require_once(__DIR__ . '/../../../../lib/behatlib.php');

/**
 * Step definitions for [FRANKENSTYLE]
 * @package   [FRANKENSTYLE]
 * @copyright (c) [YEAR] [YOUR ORG]
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

**Run Behat tests:**
```bash
# All scenarios tagged with plugin
php admin/tool/behat/cli/run.php --tags=@[FRANKENSTYLE]

# Specific feature file
php admin/tool/behat/cli/run.php [PLUGIN_MAPNAME]/tests/behat/[feature].feature

# Specific scenario
php admin/tool/behat/cli/run.php --tags=@[FRANKENSTYLE] --name="scenario name"
```

---

## Test data generation

For complex test data, create a **generator class:**

```php
<?php
// tests/generator/lib.php
namespace [FRANKENSTYLE]\tests;

class generator extends \component_generator_base {
    public function create_item($options = []): \stdClass {
        global $DB;
        
        $defaults = [
            'userid' => 2,  // Admin user
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

Each test scenario in `user-stories.md` (Given/When/Then) gets its own generator method, identifiable by the scenario ID — so test code can be traced back to the scenario without grepping, and its status updated there once the test passes (see [`moodle-plugin-development`](https://github.com/arnoutvree/moodle-plugin-development), Phase 2b/4a, if you're following that process).

**Naming:** `scenario_<number>_<slug>()`, e.g. `scenario_1_1_enrolment_still_active()` for test scenario 1.1.

**Set up via a generator/direct database insertion, never via UI navigation.** Both a PHPUnit fixture (generator method) and a Behat `Background:` follow the same rule: the Given steps set up the state directly, not by clicking through the UI the scenario is actually testing. Moodle's own Behat guidance is explicit about this: "The setup is not what you are really testing here" — Given steps should therefore prefer direct steps (setting config, creating users/courses) over step-by-step navigation through the interface.

**Pattern:** the scenario method sets up only the Given state and returns the involved objects; the test itself performs the When and asserts the Then — including what must *not* happen.

```php
<?php
// tests/generator/lib.php
namespace [FRANKENSTYLE]\tests;

class generator extends \component_generator_base {

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

**Traceability table** — illustrates the fixture ↔ test pattern. The authoritative, full progress (status per scenario: open/in progress/passed/failed) lives in `user-stories.md`, alongside `specs.md` — update that table once a scenario is implemented here, not this standalone illustration:

| Test scenario (user-stories.md) | Fixture | Test |
|---|---|---|
| 1.1 Enrolment still active | `scenario_1_1_enrolment_still_active()` | `[feature]_test.php::test_scenario_1_1_enrolment_still_active` |

The same principle applies to a Behat scenario (user-facing workflow), but via a `Background:` block with concrete data instead of a generator method — the Given lines in the `.feature` file itself are then the fixture.

**Gherkin discipline:** use `Given`, `When` and `Then` exactly once per scenario; additional steps within the same phase go via `And`/`But`. This is Moodle's own Behat convention and ties directly into the "one scenario per outcome" rule — a scenario with a second `When` is effectively testing two things at once.

---

## Coverage expectations

| Component | Test type |
|---|---|
| Service classes (managers, processors) | PHPUnit |
| External API functions | PHPUnit |
| Database operations | PHPUnit with `resetAfterTest()` |
| User-facing workflows | Behat (e.g., "admin translates a course") |
| Permission checks | PHPUnit (test both allowed and denied cases) |
| Events and observers | PHPUnit |

**Minimum coverage target:** 80% of non-trivial code paths. Dead code (unused imports, unexecuted conditions) doesn't need coverage.

---

## CI/testing in GitHub Actions

Use `moodlehq/moodle-plugin-ci` rather than invoking PHPUnit/Behat directly in CI — it handles spinning up a full Moodle install against your plugin first, which bare `vendor/bin/phpunit` cannot do standalone. See [context/workflows/release-process.md](../workflows/release-process.md) (CI/CD section) for the complete workflow.

---
