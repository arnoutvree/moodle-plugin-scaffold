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
// This file is part of Moodle - https://moodle.org/
// (full GPL v3 header as in every other PHP file of the plugin)

// NOTE: no MOODLE_INTERNAL check here: Behat may load this file before config.php is included.

require_once(__DIR__ . '/../../../../lib/behat/behat_base.php');   // the path Moodle core uses in its own step files; it also holds when the code lives in public/

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

**Plugin entities in Behat.** To use `Given the following "[FRANKENSTYLE] > items" exist:` in a `Background:`, add `tests/generator/behat_[FRANKENSTYLE]_generator.php` with a class `behat_[FRANKENSTYLE]_generator extends behat_generator_base` that defines `get_creatable_entities()` (see "Writing acceptance tests" on moodledev.io).

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

## Testing by topic

What to test per kind of component, with the Moodle helper to use. Every item follows the same rule: a happy path, at least one negative test, and the boundaries (empty array, `null`, zero records, a string at its maximum length).

**Write the test to fail first.** Before you trust a new test, see it fail: write it before the implementation, or break the implementation on purpose. A test you have never seen fail may test nothing. Never weaken an assertion, delete a test or skip one to get a green run, and never hide a runtime error in an empty `try`/`catch`.

**Denied access is tested in PHPUnit, not in Behat.** Behat marks every error page as a failed step, so it cannot assert that an action was refused.

```php
$this->setUser($this->getDataGenerator()->create_user());   // setAdminUser() / setGuestUser() / setUser(null) also exist
$this->expectException(\required_capability_exception::class);
\[FRANKENSTYLE]\manager::delete_item($itemid);
```

### Scheduled and adhoc tasks
- Adhoc: queue the task with `\core\task\manager::queue_adhoc_task()`, run it with `$this->runAdhocTasks(\[FRANKENSTYLE]\task\my_task::class)` (or `run_all_adhoc_tasks()`), then assert the result. Both are protected helpers of `advanced_testcase`.
- Scheduled: create the task object and call `execute()`.
- Test the failure path as well: what the task does when the record is gone or the external call fails.

### Events, messages and email
- Events: `$sink = $this->redirectEvents();` ... `$sink->get_events()` ... `$sink->close();`
- Messages: call `$this->preventResetByRollback()` first (messages are sent after the transaction closes), then `$sink = $this->redirectMessages();`
- Email: `unset_config('noemailever');` then `$sink = $this->redirectEmails();`

### Privacy provider
Extend `\core_privacy\tests\provider_testcase`, not `advanced_testcase`: the plain test case does not reset the privacy content writer between tests. Test the metadata (every table that holds user data is described), the export for a user, and the deletion for one user and for a whole context. The test file is `tests/privacy/provider_test.php`.

### External functions (web services)
Call the function as a user with and without the capability, pass the result through `\external_api::clean_returnvalue()`, and test invalid parameters. Check the context validation: a user without access to the course must be refused.

### Forms
Call the form's `validation($data, $files)` method with invalid input and assert the returned errors, or submit through `moodleform::mock_submit()`.

### Renderers, templates and JavaScript
- Test the data a renderable or template gets (`export_for_template()`) in PHPUnit; `moodle-plugin-ci mustache` lints the templates.
- `moodle-plugin-ci grunt` checks the JavaScript (lint and an up-to-date `amd/build/`). Behaviour that needs a browser goes in a Behat scenario tagged `@javascript`.

### External services and mocking
Tests never call a real external service, not even in your own environment. Mock the HTTP client with Moodle's dependency injection container, which is reset between tests:

```php
[$client, $mock, $history] = $this->get_mocked_http_client();   // protected helper of advanced_testcase; check the signature for your Moodle version
$mock->append(new \GuzzleHttp\Psr7\Response(200, [], json_encode(['name' => 'Colin'])));
// ...run the code that gets \core\http_client from \core\di::get(), then assert on $history
```

Without the helper: build a Guzzle `MockHandler`, wrap it in a `HandlerStack` and register `new \core\http_client(['handler' => $stack])` with `\core\di::set(\core\http_client::class, ...)`. Test the failure cases too: an error status, a timeout, malformed JSON, an empty answer.

### AI features
The model is never part of an automated test: use a stub class that implements the same interface as your provider gateway and returns fixed answers (or throws). Test your own guard code, which is deterministic. Write at least one test for every risk that applies to the feature:

| Risk | What the test asserts |
|---|---|
| Prompt injection | Text from a user, or retrieved content, cannot override the system instruction (sanitiser, prompt builder). |
| Out of scope (jailbreak) | A question outside the intended role gets a refusal or redirect from your guard, not a normal answer. |
| Data leakage | The system prompt, tool definitions and other users' data are not in the output. |
| Tool misuse | Only relevant when the model can call tools: a call outside the allowed scope is refused. |
| Unbounded input | An over-long or repeated input hits the limit or timeout. |

### Upgrade steps
`moodle-plugin-ci savepoints` checks the structure. Test the content: install the previous release, create data, upgrade, check that the data and settings are intact, and that a second upgrade changes nothing.

### Databases and themes
- Run the tests on the production database engine and on PostgreSQL (the CI matrix does). Use the `$DB` methods and not raw SQL; raw SQL has to work on both.
- Check the UI in the default theme and in a custom theme by hand (see `rules.md`).

### Data providers and flaky tests
- A data provider only returns data. It must not create records: it runs before the database reset.
- Tag a Behat scenario `@javascript` only when it needs JavaScript; those are slower and less stable. Moodle's runner `run.php --auto-rerun` runs failed scenarios once more and `vendor/bin/behat --rerun` repeats only the failed ones. A scenario that needs a rerun to pass is a problem to fix, not to hide.

### Not covered yet
Accessibility checks and JavaScript unit tests (Jest) are not described here: no verified Moodle source yet. Check the Moodle developer documentation before adding them.

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

**Without a generator method:** when the Given state is short, a private helper in the test class is fine. The link to the scenario is the scenario ID, not the generator: put the ID in the class docblock ("test scenarios 4.1, 4.1b") and in the test name (`test_scenario_4_1_...`). A generator method per scenario is the recommended form for long or reusable setup, not a requirement.

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

**Make a new test fail first** (red-first): write it before the implementation, or break the implementation on purpose, and state in your reply that you saw it fail. Never weaken an assertion, delete a test or skip one to get a green run.

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
| Scheduled and adhoc tasks | PHPUnit |
| Privacy provider | PHPUnit (`provider_testcase`) |
| Form validation | PHPUnit |
| Code that calls an external service | PHPUnit with a mocked HTTP client or stub |
| AI guard code (sanitiser, limits, permissions) | PHPUnit, one test per applicable risk |

Dead code (unused imports, conditions that can never be reached) needs no coverage; remove it instead.

---

## CI

Use `moodlehq/moodle-plugin-ci` rather than invoking PHPUnit/Behat directly in CI — it handles spinning up a full Moodle install against your plugin first, which bare `vendor/bin/phpunit` cannot do standalone. Run the checks from the table above in CI. See [context/workflows/release-process.md](../workflows/release-process.md) (CI/CD section) for the complete workflow.

---
