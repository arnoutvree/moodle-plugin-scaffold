---
type: architecture
created: 2026-06-26
timestamp: 2026-06-26
---
# API patterns — [FRANKENSTYLE]

This file describes external services, AJAX endpoints, web service definitions, and error handling patterns for this plugin.

---

## External services (web services)

External services let other systems call your plugin's functions via REST or XMLRPC.

**Define in `classes/external/`:**

```php
<?php
namespace [FRANKENSTYLE]\external;

class get_items extends \core_external\external_api {
    
    public static function execute_parameters() {
        return new \core_external\external_function_parameters([
            'courseid' => new \core_external\external_value(
                PARAM_INT, 
                'ID of the course'
            ),
        ]);
    }
    
    public static function execute($courseid) {
        global $DB;
        
        // Validate parameters
        ['courseid' => $courseid] = self::validate_parameters(
            self::execute_parameters(),
            ['courseid' => $courseid]
        );
        
        // Check capability
        $context = \context_course::instance($courseid);
        self::validate_context($context);
        require_capability('[FRANKENSTYLE]:view', $context);
        
        // Fetch data
        $items = $DB->get_records('[FRANKENSTYLE]_items', ['courseid' => $courseid]);
        
        // Return with structure
        return self::execute_returns($items);
    }
    
    public static function execute_returns() {
        return new \core_external\external_multiple_structure(
            new \core_external\external_single_structure([
                'id' => new \core_external\external_value(PARAM_INT, 'Item ID'),
                'name' => new \core_external\external_value(PARAM_TEXT, 'Item name'),
            ])
        );
    }
}
```

**Register in `db/services.php`:**

```php
<?php
$services = [
    '[FRANKENSTYLE]_api' => [
        'functions' => [
            '[FRANKENSTYLE]_get_items',
        ],
        'requiredcapability' => '[FRANKENSTYLE]:view',
        'restrictedusers' => 0,
        'enabled' => 1,
    ],
];

$functions = [
    '[FRANKENSTYLE]_get_items' => [
        'classname' => '[FRANKENSTYLE]\\external\\get_items',
        'methodname' => 'execute',
        'classpath' => 'local/[PLUGIN_NAME]/classes/external/get_items.php',
        'description' => 'Get items from a course',
        'type' => 'read',
        'ajax' => true,
        'loginrequired' => true,
        'capabilities' => '[FRANKENSTYLE]:view',
    ],
];
```

---

## AJAX calls from JavaScript

In your AMD module, use `core/ajax`:

```javascript
// amd/src/mymodule.js
define(['jquery', 'core/ajax', 'core/notification'], function($, Ajax, Notification) {
    return {
        init: function() {
            $('#mybutton').on('click', function() {
                Ajax.call([{
                    methodname: '[FRANKENSTYLE]_get_items',
                    args: {
                        courseid: 5
                    },
                    done: function(response) {
                        console.log('Items:', response);
                    },
                    fail: Notification.exception
                }]);
            });
        }
    };
});
```

Then load in your page or template:

```php
$PAGE->requires->js_call_amd('[FRANKENSTYLE]/mymodule', 'init');
```

---

## Error handling

Always throw `moodle_exception` with a language string, never raw PHP exceptions:

```php
// Correct
if (!$item) {
    throw new \moodle_exception('itemnotfound', '[FRANKENSTYLE]');
}

// Wrong
if (!$item) {
    throw new \Exception('Item not found');
}
```

In your language file:

```php
$string['itemnotfound'] = 'Item not found';
```

**Common exceptions:**

```php
// No permissions
throw new \required_capability_exception($context, '[FRANKENSTYLE]:view', 'nopermission', '[FRANKENSTYLE]');

// Invalid parameter
throw new \invalid_parameter_exception('Invalid course ID');

// Moodle exception with custom message
throw new \moodle_exception('error:customcode', '[FRANKENSTYLE]', '', $a = ['var' => 'value']);
```

---

## Parameter validation

Always validate with `validate_parameters()`:

```php
public static function execute($courseid, $status = 'all') {
    $params = self::validate_parameters(
        self::execute_parameters(),
        ['courseid' => $courseid, 'status' => $status]
    );
    // Use $params, never raw input
}
```

**Common parameter types:**

| Type | Purpose |
|---|---|
| `PARAM_INT` | Integer |
| `PARAM_FLOAT` | Float/decimal |
| `PARAM_TEXT` | Plain text (no HTML) |
| `PARAM_RAW` | Raw text (careful!) |
| `PARAM_ALPHA` | Letters only |
| `PARAM_ALPHANUM` | Letters and numbers |
| `PARAM_BOOL` | Boolean |
| `PARAM_EMAIL` | Email address |
| `PARAM_URL` | URL |

---

## AI-agent-friendly external functions

When external functions are the primary interface for an AI agent (not just a JS frontend), a few extra patterns pay off:

**Normalized content model.** If the plugin lets an agent create/read/update content across several underlying entity types (e.g. different Moodle module types), define one shared read/write shape instead of a separate schema per type:

```
{ id, type, name,
  intro:    {text, format},
  content:  {text, format},
  subitems: [...],           // nested structure (chapters, pages, ...)
  files:    [...],           // draftitemid references
  settings: {...},           // type-specific extras
  visible }
```

A single "get content" function produces this shape; a single "update content" function consumes it. This turns create/improve/translate into the same operation applied to different fields, so the agent (and the plugin) doesn't need N bespoke schemas.

**Tiered entity coverage.** Roll out support for entity/module types in tiers, ordered by how their content is represented — not by feature importance:

| Tier | Nature of content | Ship first? |
|---|---|---|
| 1 | Plain HTML/text the agent can generate directly | Yes — v1 |
| 2 | Settings + short intro/description | v2 |
| 3 | Uploaded file/package (binary, not agent-generatable as text) | v3 |
| 4 | Substructure built from many small records (e.g. quiz questions) | v3 |

Tier 3/4 types usually need a different write path entirely (file upload API, bulk import) rather than the create/update functions used for Tier 1/2 — call this out explicitly so it isn't assumed to be "just another modname".

**Namespace migration note.** On Moodle 4.2+, external function base classes live under `core_external\` (`core_external\external_api`, `external_function_parameters`, `external_value`, `external_single_structure`, ...), not the old global class names. If you copy an older example or skeleton, check it isn't still using the pre-4.2 global classes with `defined('MOODLE_INTERNAL')`.

---

## External integrations bypassing the standard web-service token flow

The patterns above assume Moodle's own web-service token/capability model (`db/services.php`). If a plugin instead exposes a lightweight custom endpoint outside that flow — e.g. a small REST handler that isn't registered as a Moodle web service at all — it doesn't inherit token/capability checks automatically. Add your own: rate limiting per API key, audit logging (key + IP + endpoint), and HTTPS-enforcing for any key sent via query string (query strings end up in server/proxy logs, unlike headers or POST bodies).

---

## REST vs XMLRPC

Both are configured in `db/services.php`. To call via REST:

```bash
curl -X POST http://moodle.local/webservice/rest/server.php \
  -d "wstoken=YOUR_TOKEN" \
  -d "wsfunction=[FRANKENSTYLE]_get_items" \
  -d "courseid=5" \
  -d "moodlewsrestformat=json"
```

---
