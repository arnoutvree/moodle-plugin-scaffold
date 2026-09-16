---
type: reference
created: 2026-06-26
timestamp: 2026-06-26
---
# Moodle platform — [FRANKENSTYLE]

This file describes Moodle-specific subsystems and implementation patterns: version metadata, events, output API, AMD modules, forms, strings, privacy, tasks, and coding standards.

---

## version.php — plugin metadata

**Required fields:**

```php
<?php
defined('MOODLE_INTERNAL') || die();

$plugin->component = '[FRANKENSTYLE]';           // Always: [PLUGIN_TYPE]_[PLUGIN_NAME]
$plugin->version = 2025052000;                    // Format: YYYYMMDDXX (XX = day sequence: 00, 01, 02...)
$plugin->requires = 2025041400;                   // Minimum Moodle version (e.g., 4.5 = 2025041400)
$plugin->release = '1.0.0';                       // Semantic version string
$plugin->maturity = MATURITY_STABLE;              // or ALPHA, BETA, RC
$plugin->copyright = '(c) 2025 [Your Organization]';
$plugin->license = 'GPL-3.0-or-later';

// Optional but recommended:
$plugin->supported = [4.5, 5.0, 5.1];             // Versions this plugin targets
```

**Rules:**
- `$plugin->version` is **YYYYMMDDXX** — increment `XX` for multiple releases the same day (00, 01, 02, ...)
- `$plugin->requires` matches the **minimum Moodle version** you support (e.g., 4.5 LTS)
- Maturity levels: `MATURITY_ALPHA` → `MATURITY_BETA` → `MATURITY_RC` → `MATURITY_STABLE`
- Never include side effects or function calls in `version.php`

---

## Events system

**Define an event in `classes/event/`:**

```php
<?php
namespace [FRANKENSTYLE]\event;

class item_created extends \core\event\base {
    protected function init() {
        $this->data['crud'] = 'c';  // c, r, u, d for create/read/update/delete
        $this->data['edulevel'] = self::LEVEL_OTHER;  // or LEVEL_TEACHING, LEVEL_LEARNING
        $this->data['objecttable'] = '[FRANKENSTYLE]_items';
    }

    public static function create_from_item($item, $context) {
        $data = [
            'context' => $context,
            'objectid' => $item->id,
        ];
        return self::create($data);
    }

    public function get_description() {
        return "User ID {$this->userid} created item {$this->objectid}.";
    }

    public function get_url() {
        return new \moodle_url('/local/[PLUGIN_NAME]/view.php', ['id' => $this->objectid]);
    }
}
```

**Trigger an event:**
```php
$item = $DB->insert_record('[FRANKENSTYLE]_items', $record);
$event = \[FRANKENSTYLE]\event\item_created::create_from_item($item, $context);
$event->trigger();
```

**Observe events in `db/events.php`:**
```php
<?php
$observers = [
    [
        'eventname' => '\[FRANKENSTYLE]\event\item_created',
        'callback' => '\[FRANKENSTYLE]\observer::on_item_created',
        'includefile' => null,
    ],
];
```

**Observer class in `classes/observer.php`:**
```php
<?php
namespace [FRANKENSTYLE];

class observer {
    public static function on_item_created(\[FRANKENSTYLE]\event\item_created $event) {
        // React to the event
        global $DB;
        $item = $DB->get_record('[FRANKENSTYLE]_items', ['id' => $event->objectid]);
        // ... do something ...
    }
}
```

---

## Hooks API (Moodle 4.4+) — prefer over legacy callbacks

For new integration points, check whether a **Hook** already exists before adding a legacy `lib.php` callback (`extend_navigation()`, `before_http_headers()`, etc.). The Hooks API is strongly typed, discoverable, and is where Moodle core is migrating legacy callbacks to — legacy callbacks still work but new code should prefer hooks when a matching hook exists.

**Listen to a core hook** — register in `db/hooks.php`:

```php
<?php
// db/hooks.php
$callbacks = [
    [
        'hook' => \core\hook\navigation\primary_extend::class,
        'callback' => '\[FRANKENSTYLE]\hook_callbacks::extend_primary_navigation',
        'priority' => 0,
    ],
];
```

```php
<?php
// classes/hook_callbacks.php
namespace [FRANKENSTYLE];

class hook_callbacks {
    public static function extend_primary_navigation(\core\hook\navigation\primary_extend $hook): void {
        $hook->add_item(...);
    }
}
```

**Define a custom hook** for other plugins to listen to (in `classes/hook/`):

```php
<?php
namespace [FRANKENSTYLE]\hook;

#[\core\attribute\label('Allows plugins to react after an item is created')]
#[\core\attribute\tags('[FRANKENSTYLE]')]
class item_created {
    public function __construct(
        public readonly int $itemid,
    ) {}
}
```

Dispatch it like any other hook:

```php
$hook = new \[FRANKENSTYLE]\hook\item_created($itemid);
\core\di::get(\core\hook\manager::class)->dispatch($hook);
```

**Rules:**
- Check the current list of available core hooks before writing a new `lib.php` callback — a hook may already cover the same integration point
- Hooks are discovered via `db/hooks.php`, not autoloading magic — always register there
- Prefer hooks for new plugin-to-plugin extension points too, not just core integration

---

## Output API — rendering templates

**Create a renderable class:**

```php
<?php
namespace [FRANKENSTYLE]\output;

class item_view implements \renderable, \templatable {
    private $item;

    public function __construct($item) {
        $this->item = $item;
    }

    public function export_for_template(\renderer_base $output) {
        return [
            'id' => $this->item->id,
            'name' => format_string($this->item->name),
            'description' => format_text($this->item->description),
            'candelete' => has_capability('[FRANKENSTYLE]:delete', $output->get_context()),
        ];
    }
}
```

**Render it in your page:**

```php
$item = $DB->get_record('[FRANKENSTYLE]_items', ['id' => $itemid]);
$renderable = new \[FRANKENSTYLE]\output\item_view($item);
echo $OUTPUT->render($renderable);
```

**Mustache template `templates/item_view.mustache`:**

```mustache
<div class="[FRANKENSTYLE]-item">
    <h3>{{name}}</h3>
    <p>{{description}}</p>
    {{#candelete}}
        <a href="delete.php?id={{id}}">{{#str}}delete, [FRANKENSTYLE]{{/str}}</a>
    {{/candelete}}
</div>
```

---

## AMD modules (JavaScript)

**Source file `amd/src/mymodule.js`:**

```javascript
define(['jquery', 'core/notification'], function($, Notification) {
    return {
        init: function() {
            $('#mybutton').on('click', function() {
                console.log('Button clicked!');
            });
        }
    };
});
```

**Build** (convert ES5 to AMD):
```bash
npx grunt amd
```

**Load in your page:**

```php
$PAGE->requires->js_call_amd('[FRANKENSTYLE]/mymodule', 'init', []);
```

**Or in a template:**

```mustache
{{#js}}
require(['[FRANKENSTYLE]/mymodule'], function(Module) {
    Module.init();
});
{{/js}}
```

---

## Forms (moodleform)

**Create a form in `classes/form/edit_item.php`:**

```php
<?php
namespace [FRANKENSTYLE]\form;

class edit_item extends \moodleform {
    protected function definition() {
        $mform = $this->_form;

        $mform->addElement('text', 'name', get_string('name', '[FRANKENSTYLE]'));
        $mform->setType('name', PARAM_TEXT);
        $mform->addRule('name', null, 'required', null, 'client');

        $mform->addElement('textarea', 'description', get_string('description', '[FRANKENSTYLE]'));
        $mform->setType('description', PARAM_RAW);

        $this->add_action_buttons(true, get_string('save', '[FRANKENSTYLE]'));
    }

    public function validation($data, $files) {
        $errors = parent::validation($data, $files);
        if (strlen($data['name']) < 3) {
            $errors['name'] = 'Name must be at least 3 characters';
        }
        return $errors;
    }
}
```

**Use the form:**

```php
$form = new \[FRANKENSTYLE]\form\edit_item(null, ['itemid' => $itemid]);

if ($form->is_cancelled()) {
    redirect('/path/to/list.php');
} else if ($data = $form->get_data()) {
    // Process form submission
    $DB->update_record('[FRANKENSTYLE]_items', $data);
    redirect('/path/to/view.php?id=' . $data->id);
} else {
    // Display form
    $form->display();
}
```

---

## Strings (i18n)

**All user-facing text in `lang/en/[FRANKENSTYLE].php`:**

```php
<?php
$string['pluginname'] = 'My Plugin';
$string['name'] = 'Name';
$string['description'] = 'Description';
$string['delete'] = 'Delete';
$string['deleteconfirm'] = 'Are you sure you want to delete this item?';
$string['error:itemnotfound'] = 'Item not found';
$string['privacy:metadata'] = 'This plugin stores item data.';
```

**Use strings:**

```php
// In PHP
echo get_string('pluginname', '[FRANKENSTYLE]');

// In templates
{{#str}}pluginname, [FRANKENSTYLE]{{/str}}

// With parameters
echo get_string('welcome', '[FRANKENSTYLE]', ['name' => $username]);
```

---

## Privacy API

**Implement in `classes/privacy/provider.php`:**

```php
<?php
namespace [FRANKENSTYLE]\privacy;

class provider implements
    \core_privacy\local\metadata\null_provider,
    \core_privacy\local\request\plugin_provider
{
    public static function get_reason(): string {
        return 'privacy:metadata';
    }

    public static function get_contexts_for_userid(int $userid): \core_privacy\local\request\contextlist {
        return new \core_privacy\local\request\contextlist();
    }

    public static function export_user_data(\core_privacy\local\request\approved_contextlist $contextlist) {
        // Export user data
    }

    public static function delete_data_for_user(\core_privacy\local\request\approved_contextlist $contextlist) {
        // Delete user data
    }
}
```

**Minimum: null provider** (plugin stores no personal data):
```php
class provider implements \core_privacy\local\metadata\null_provider {
    public static function get_reason(): string {
        return 'privacy:reason_no_data';
    }
}
```

---

## Scheduled and adhoc tasks

**Define a scheduled task in `classes/task/cleanup.php`:**

```php
<?php
namespace [FRANKENSTYLE]\task;

class cleanup extends \core\task\scheduled_task {
    public function get_name() {
        return get_string('task:cleanup', '[FRANKENSTYLE]');
    }

    public function execute() {
        global $DB;
        // Delete old items
        $DB->delete_records('[FRANKENSTYLE]_items', [
            'status' => 'deleted',
        ]);
        mtrace('Cleanup complete');
    }
}
```

**Register in `db/tasks.php`:**

```php
<?php
$tasks = [
    [
        'classname' => '[FRANKENSTYLE]\\task\\cleanup',
        'blocking' => 0,
        'minute' => '0',
        'hour' => '2',
        'day' => '*',
        'month' => '*',
        'dayofweek' => '*',
    ],
];
```

---

## Coding standards

**PHP: Moodle PHP Coding Standards**
- Spaces, not tabs
- Brace on same line: `if (...) {`
- Class names: `ClassName`, method names: `methodName()`, constants: `CONSTANT`
- Always use `defined('MOODLE_INTERNAL') || die();` at the top of PHP files, except entry points, side-effect-free class/interface/trait files, and `db/install.php`/`db/upgrade.php` (nearly all Moodle core plugins omit the check there)
- Add PHPDoc to all classes and public methods

**JS: AMD + ESLint**
- Lint: `npm run lint` (if configured)
- Use `define()` for AMD modules
- Avoid `var`, use `let` and `const`

**CSS: BEM-style naming**
```css
.[FRANKENSTYLE]-container { }
.[FRANKENSTYLE]-container__title { }
.[FRANKENSTYLE]-container--disabled { }
```

**Mustache: use language strings**
```mustache
{{#str}}key, [FRANKENSTYLE]{{/str}}
```

---
