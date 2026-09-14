---
type: architecture
created: 2026-06-26
timestamp: 2026-06-26
---
# Database schema — [FRANKENSTYLE]

This file describes the database design, DML patterns, table naming, and upgrade strategy for `[FRANKENSTYLE]`.

---

## DML API — never raw SQL

Always use the Moodle DML `$DB` global. Never use PDO, mysqli, or raw SQL.

```php
// Read single record
$record = $DB->get_record('[FRANKENSTYLE]_items', ['userid' => $userid]);

// Read multiple records
$records = $DB->get_records('[FRANKENSTYLE]_items', ['status' => 'pending']);

// Read with SQL (use named params, never concatenation)
$sql = 'SELECT * FROM {[FRANKENSTYLE]_items} WHERE userid = :userid AND status = :status';
$records = $DB->get_records_sql($sql, ['userid' => $userid, 'status' => 'pending']);

// Count
$count = $DB->count_records('[FRANKENSTYLE]_items', ['status' => 'active']);

// Insert
$record = new \stdClass();
$record->userid = $userid;
$record->status = 'new';
$record->id = $DB->insert_record('[FRANKENSTYLE]_items', $record);

// Update
$record->status = 'updated';
$DB->update_record('[FRANKENSTYLE]_items', $record);

// Delete
$DB->delete_records('[FRANKENSTYLE]_items', ['id' => $id]);
```

---

## Settings API

Store admin and plugin settings in `config_plugins`:

```php
// Get setting
$value = get_config('[FRANKENSTYLE]', 'setting_name');

// Set setting
set_config('setting_name', $value, '[FRANKENSTYLE]');
```

**Never** write directly to the `config` table. Always use `get_config()` / `set_config()`.

---

## Table naming and install.xml

**Table names:**
- Moodle automatically prefixes tables with `mdl_`
- In `install.xml`, omit the prefix; write only `[FRANKENSTYLE]_tablename`
- In PHP code, use `$DB->get_record('[FRANKENSTYLE]_tablename', ...)`
- Max length: 28 chars after `mdl_` (total table name limit)

**Creating tables:**
- Always use the **XMLDB Editor** (Site Admin → Development → XMLDB editor)
- Never hand-write XML
- Define all indexes, foreign keys, and defaults in XMLDB
- Example snippet:
  ```xml
  <TABLE NAME="[FRANKENSTYLE]_items">
    <FIELDS>
      <FIELD NAME="id" TYPE="int" LENGTH="10" NOTNULL="true" SEQUENCE="true" />
      <FIELD NAME="userid" TYPE="int" LENGTH="10" NOTNULL="true" />
      <FIELD NAME="status" TYPE="char" LENGTH="20" NOTNULL="false" />
      <FIELD NAME="timecreated" TYPE="int" LENGTH="10" NOTNULL="true" />
    </FIELDS>
    <KEYS>
      <KEY NAME="primary" TYPE="primary" FIELDS="id" />
      <KEY NAME="userid" TYPE="foreign" FIELDS="userid" REFTABLE="user" REFFIELDS="id" />
    </KEYS>
    <INDEXES>
      <INDEX NAME="status_idx" UNIQUE="false" FIELDS="status" />
    </INDEXES>
  </TABLE>
  ```

---

## Upgrade strategy — db/upgrade.php

```php
<?php
defined('MOODLE_INTERNAL') || die();

function xmldb_[FRANKENSTYLE]_upgrade($oldversion) {
    global $DB;
    $dbman = $DB->get_manager();

    if ($oldversion < 2025060100) {
        // Create new table
        $table = new \xmldb_table('[FRANKENSTYLE]_items');
        // (Fields defined in install.xml or via xmldb_field calls)
        if (!$dbman->table_exists($table)) {
            $dbman->create_table($table);
        }
        upgrade_plugin_savepoint(true, 2025060100, '[PLUGIN_TYPE]', '[PLUGIN_NAME]');
    }

    if ($oldversion < 2025070100) {
        // Add column
        $table = new \xmldb_table('[FRANKENSTYLE]_items');
        $field = new \xmldb_field('newcolumn', XMLDB_TYPE_TEXT, 'medium', null, false);
        if (!$dbman->field_exists($table, $field)) {
            $dbman->add_field($table, $field);
        }
        upgrade_plugin_savepoint(true, 2025070100, '[PLUGIN_TYPE]', '[PLUGIN_NAME]');
    }

    if ($oldversion < 2025080100) {
        // Rename capability (if applicable)
        upgrade_rename_capability('capability_old_name', '[FRANKENSTYLE]:new_name');
        upgrade_plugin_savepoint(true, 2025080100, '[PLUGIN_TYPE]', '[PLUGIN_NAME]');
    }

    return true;
}
```

**Rules:**
- Never call plugin functions from `upgrade.php` — use only DML directly
- Always wrap version checks in `if ($oldversion < YYYYMMDDXX)` blocks
- Call `upgrade_plugin_savepoint(true, VERSION, '[PLUGIN_TYPE]', '[PLUGIN_NAME]')` after each step
- Match each savepoint version to the corresponding version in `version.php`
- Test upgrades from every supported Moodle version to the latest

---

## Common queries

### Paginated results
```php
$sql = 'SELECT * FROM {[FRANKENSTYLE]_items} WHERE userid = :userid ORDER BY timecreated DESC';
$records = $DB->get_records_sql($sql, ['userid' => $userid], $start, $limit);
```

### Aggregate
```php
$sql = 'SELECT COUNT(*) as cnt FROM {[FRANKENSTYLE]_items} WHERE status = :status';
$result = $DB->get_record_sql($sql, ['status' => 'done']);
echo $result->cnt;
```

### Join
```php
$sql = 'SELECT i.*, u.firstname, u.lastname
        FROM {[FRANKENSTYLE]_items} i
        JOIN {user} u ON i.userid = u.id
        WHERE i.status = :status';
$records = $DB->get_records_sql($sql, ['status' => 'pending']);
```

---

## What never to modify

- Core Moodle tables (`user`, `course`, `role_capabilities`, etc.)
- Tables created by other plugins
- The `config` table — always use `get_config()` / `set_config()`
- The `version` field in `version.php` except when planning a release

---
