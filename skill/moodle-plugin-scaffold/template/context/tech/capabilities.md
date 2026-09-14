---
type: context
created: 2026-06-26
timestamp: 2026-06-26
---
# Capabilities — [FRANKENSTYLE]

This file describes capability design, role assignments, permission checks, and the safety checklist for permission-related changes.

---

## Capability structure

Define capabilities in `db/access.php`:

```php
<?php
defined('MOODLE_INTERNAL') || die();

$capabilities = [
    '[FRANKENSTYLE]:view' => [
        'captype' => 'view',
        'contextlevel' => CONTEXT_COURSE,
        'archetypes' => [
            'guest' => CAP_PREVENT,
            'student' => CAP_ALLOW,
            'teacher' => CAP_ALLOW,
            'editingteacher' => CAP_ALLOW,
            'manager' => CAP_ALLOW,
        ],
        'clonepermissionsfrom' => 'moodle/course:view',
        'riskbitmask' => RISK_XSS,
    ],
    
    '[FRANKENSTYLE]:edit' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_COURSE,
        'archetypes' => [
            'teacher' => CAP_ALLOW,
            'editingteacher' => CAP_ALLOW,
            'manager' => CAP_ALLOW,
        ],
        'clonepermissionsfrom' => 'moodle/course:update',
        'riskbitmask' => RISK_DATALOSS,
    ],
    
    '[FRANKENSTYLE]:manage' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_SYSTEM,
        'archetypes' => [
            'manager' => CAP_ALLOW,
        ],
        'clonepermissionsfrom' => 'moodle/site:config',
        'riskbitmask' => RISK_CONFIG,
    ],
];
```

---

## Capability fields

| Field | Purpose |
|---|---|
| `captype` | `view` (reading) or `write` (modifying) |
| `contextlevel` | `CONTEXT_SYSTEM`, `CONTEXT_COURSE`, `CONTEXT_MODULE`, etc. |
| `archetypes` | Default role assignments: `guest`, `student`, `teacher`, `editingteacher`, `manager` |
| `clonepermissionsfrom` | Copy permissions from an existing Moodle capability (for new role setup) |
| `riskbitmask` | Security risk level: `RISK_SPAM`, `RISK_XSS`, `RISK_PERSONAL`, `RISK_DATALOSS`, `RISK_CONFIG`, `RISK_MANAGETRUST` |

---

## Using capabilities in code

**Check single capability:**
```php
$context = \context_course::instance($courseid);
if (has_capability('[FRANKENSTYLE]:view', $context)) {
    // User can view
}
```

**Require capability (throw exception if denied):**
```php
require_capability('[FRANKENSTYLE]:edit', $context);
// If denied, throws moodle_exception
```

**Find all roles with a capability:**
```php
$roles = get_roles_with_capability('[FRANKENSTYLE]:edit', CAP_ALLOW, $context);
foreach ($roles as $role) {
    echo $role->localname;  // e.g., "Teacher", "Editor"
}
```

---

## Permission safety checklist

**BEFORE every commit that touches `db/access.php`:**

### Check 1: Role impact analysis
```
[ ] For each capability added, modified, or removed:
    [ ] Run: get_roles_with_capability('[FRANKENSTYLE]:capname', CAP_ALLOW, $context)
    [ ] Document: which roles have this capability before and after?
    [ ] Verify: no unintended role gets or loses access
```

### Check 2: User-facing capability checks
```
[ ] For each new require_capability() or has_capability() call:
    [ ] Verify student role: can view? Can edit? (expected behavior)
    [ ] Verify teacher role: can view? Can edit? (expected behavior)
    [ ] Verify editingteacher role: can view? Can edit? (expected behavior)
    [ ] Verify manager role: can view? Can edit? (expected behavior)
    [ ] Document any non-standard assignments in a code comment
```

### Check 3: Capability rename or delete
```
[ ] Never silently remove a capability from db/access.php
[ ] For rename: use upgrade_rename_capability() in db/upgrade.php
[ ] For delete: add to db/upgrade.php with $DB->delete_records('role_capabilities', ['capability' => '...'])
[ ] Bump version in version.php
```

---

## Admin-configurable allowlist for agent-driven actions

When a plugin gives an AI agent broad write access across a heterogeneous set of entities (e.g. which Moodle module/activity types the agent is allowed to create), don't hardcode the allowed set. Add a site-admin setting (`settings.php`) that bounds the palette, and validate against it both in the UI/wizard and in every external function — never client-side only.

This is a separate decision from the capability check: the capability decides **who** may use the plugin at all; the allowlist decides **what** the agent may touch within that session. Keep both checks — one doesn't substitute for the other.

## Capability choice rule of thumb

Start by reusing an **existing core capability** (e.g. `moodle/course:manageactivities`) rather than defining a custom one immediately. Add a custom capability only once a task needs finer-grained access than the core capability provides — for example, to separate "AI-driven building" from "manual activity management". Adding a custom capability later is always possible without a breaking change; replacing a core capability with a custom one too early risks touching existing role configurations for no reason.

## Risk bitmask for AI-written content

A plugin that writes content on behalf of an agent almost always needs at least:

| Risk | When it applies |
|---|---|
| `RISK_XSS` | Writes agent-generated HTML into course content |
| `RISK_SPAM` | Can generate a large amount of content/activities in one action (bulk creation) |
| `RISK_DATALOSS` | Consider for functions that overwrite or delete existing content |

Choose these deliberately when defining the capability — don't leave the risk mask minimal just because the content was authored "by AI" rather than typed by the user.

---

## Upgrade patterns

**Rename a capability:**
```php
// db/upgrade.php
if ($oldversion < 2025070100) {
    upgrade_rename_capability(
        '[FRANKENSTYLE]:oldcap',
        '[FRANKENSTYLE]:newcap'
    );
    upgrade_plugin_savepoint(true, 2025070100, '[PLUGIN_TYPE]', '[PLUGIN_NAME]');
}
```

**Delete a capability:**
```php
// db/upgrade.php
if ($oldversion < 2025070100) {
    $DB->delete_records('role_capabilities', [
        'capability' => '[FRANKENSTYLE]:unusedcap'
    ]);
    upgrade_plugin_savepoint(true, 2025070100, '[PLUGIN_TYPE]', '[PLUGIN_NAME]');
}
```

**Add a new capability:**
Just add it to `db/access.php` — no upgrade step needed. Moodle handles it automatically.

---

## Risk bitmasks explained

| Mask | Meaning |
|---|---|
| `RISK_SPAM` | User could spam other users |
| `RISK_XSS` | User input displayed to others (XSS risk) |
| `RISK_PERSONAL` | Access to personal data (email, preferences) |
| `RISK_DATALOSS` | Ability to delete or modify data permanently |
| `RISK_CONFIG` | Ability to change system configuration |
| `RISK_MANAGETRUST` | Ability to manage roles or permissions (dangerous!) |

Choose the **highest applicable risk**. For example, a "view user profile" capability is `RISK_PERSONAL`.

---

## Archetype reference

**Standard Moodle roles:**

| Archetype | Used for |
|---|---|
| `guest` | Unauthenticated visitors (limited access) |
| `student` | Course participants (view content, submit work) |
| `teacher` | Non-editing teachers (limited instructor role) |
| `editingteacher` | Full teaching capability (create content, manage grades) |
| `manager` | System administrators (all permissions) |

**Strategy:**
- `view` capabilities: grant to `student` and above
- `edit` capabilities: grant to `editingteacher` and above
- `manage` (system-level): grant only to `manager`

---

## Example: complete capability set

```php
$capabilities = [
    // View a report (read-only)
    '[FRANKENSTYLE]:viewreport' => [
        'captype' => 'view',
        'contextlevel' => CONTEXT_COURSE,
        'archetypes' => [
            'editingteacher' => CAP_ALLOW,
            'manager' => CAP_ALLOW,
        ],
        'clonepermissionsfrom' => 'moodle/course:view',
        'riskbitmask' => RISK_XSS,
    ],
    
    // Create or edit items (write)
    '[FRANKENSTYLE]:edit' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_COURSE,
        'archetypes' => [
            'editingteacher' => CAP_ALLOW,
            'manager' => CAP_ALLOW,
        ],
        'clonepermissionsfrom' => 'moodle/course:update',
        'riskbitmask' => RISK_DATALOSS,
    ],
    
    // Delete items (high risk)
    '[FRANKENSTYLE]:delete' => [
        'captype' => 'write',
        'contextlevel' => CONTEXT_COURSE,
        'archetypes' => [
            'manager' => CAP_ALLOW,
        ],
        'clonepermissionsfrom' => 'moodle/course:delete',
        'riskbitmask' => RISK_DATALOSS,
    ],
];
```

---
