---
type: process
created: 2026-06-26
timestamp: 2026-06-26
---
# Release process — [FRANKENSTYLE]

This file describes the complete release workflow: version numbering, pre-release checklist, testing, packaging, and distribution.

---

## Version numbering

Use **two numbers together:**

1. **`$plugin->version` (database):** `YYYYMMDDXX` format
   - Updated for every release
   - Moodle uses this to detect upgrades
   - Format: 2025052000, 2025052001, etc.

2. **`$plugin->release` (human-readable):** semantic version
   - Format: `1.0.0`, `1.1.0`, `2.0.0-beta1`
   - Both must be updated for every release

**Example:**
```php
$plugin->version = 2025052000;  // Database version
$plugin->release = '1.0.0';     // Human version
```

---

## Complete pre-release checklist

### Code quality

```
[ ] Every check in the Definition of done passes on the exact commit you release
      (single list: context/tech/testing.md, "Definition of done"; run via moodle-plugin-ci)

[ ] All PHPUnit tests pass
      vendor/bin/phpunit --testsuite [FRANKENSTYLE]_testsuite

[ ] All Behat tests pass (if tests exist)
      php admin/tool/behat/cli/run.php --tags=@[FRANKENSTYLE]

[ ] Code style passes with 0 errors
      moodle-plugin-ci phpcs

[ ] No debug output left in code
      - No var_dump(), print_r(), console.log()
      - No error_log() outside \debugging() calls
      - No TODO/FIXME comments in production files
```

### Documentation

```
[ ] lang/en/[FRANKENSTYLE].php has all strings
      - Check for missing translations
      - Verify string keys are lowercase

[ ] CHANGELOG or version history updated
      - Document breaking changes, new features, bug fixes

[ ] README.md or plugin description updated
      - Clear feature list
      - Installation instructions
      - Known limitations (if any)
```

### Metadata

```
[ ] version.php updated
      - $plugin->version bumped (YYYYMMDDXX)
      - $plugin->release updated (semver)
      - $plugin->requires matches minimum Moodle version
      - $plugin->maturity set correctly (ALPHA/BETA/RC/STABLE)

[ ] No hardcoded plugin names in version.php
      - Use $plugin->component = '[FRANKENSTYLE]';
```

### Functionality

```
[ ] $plugin->requires matches minimum Moodle version
      - Test upgrade from that version to current

[ ] Privacy API implemented
      - At least null_provider or full implementation
      - Verified via tool_dataprivacy

[ ] Capabilities correctly assigned
      - Tested: student, teacher, editingteacher, manager roles
      - No unintended permission grants

[ ] Database tables (if applicable)
      - Upgrade script tested from previous version
      - No orphaned columns or tables
```

### Compliance

```
[ ] GPL v3 header in all PHP files
      ```php
      // This file is part of Moodle - https://moodle.org/
      // Moodle is free software: you can redistribute it and/or modify
      // it under the terms of the GNU General Public License as published by
      // the Free Software Foundation, either version 3 of the License, or
      // (at your option) any later version.
      ```

[ ] No credentials, API keys, or secrets in code
      - Check all `.php`, `.js`, `.sql` files
      - No hardcoded passwords or tokens

[ ] External network calls require user/admin consent
      - No automatic API calls without opt-in setting
```

---

## Creating a release

### Step 1: Prepare version numbers

```bash
# Update version.php
# OLD:
# $plugin->version = 2025051000;
# $plugin->release = '0.9.0';

# NEW:
# $plugin->version = 2025052000;
# $plugin->release = '1.0.0';
```

### Step 2: Run full test suite

```bash
# Unit tests
vendor/bin/phpunit --testsuite [FRANKENSTYLE]_testsuite

# Acceptance tests (if applicable)
php admin/tool/behat/cli/run.php --tags=@[FRANKENSTYLE]

# Lint
./vendor/bin/phpcs [PLUGIN_MAPNAME]/
```

### Step 3: Commit and tag

```bash
git add .
git commit -m "Release v1.0.0

- Feature X implemented
- Bug Y fixed
- Documentation updated

Prepare for release via CLAUDE.md release checklist."

git tag -a v1.0.0 -m "Release 1.0.0"
git push origin main
git push origin v1.0.0
```

### Step 4: Generate zip package

**The zip must have the plugin mapname as root, NOT the Frankenstyle name.**

```bash
cd /path/to/parent/directory/of/plugin

# Good: results in translatecourse/ at zip root
zip -r [PLUGIN_NAME]-[version].zip [PLUGIN_MAPNAME]/ \
  --exclude "*.DS_Store" \
  --exclude "*/__pycache__/*" \
  --exclude "*.git*" \
  --exclude "*node_modules*" \
  --exclude "*/.env*" \
  --exclude "*tests/*" \
  --exclude "*.log" \
  --exclude "*/.idea/*" \
  --exclude "*/.vscode/*" \
  --exclude "*.Spotlight-V100*" \
  --exclude "*.Trashes*" \
  --exclude "*.codegraph*"

# Example result: translatecourse-1.0.0.zip
# When extracted, creates: translatecourse/ (NOT local_translatecourse/)
```

**Why:** Moodle installer expects the folder inside the zip to match the plugin component name. The installer will rename it to `local_translatecourse/` automatically.

### Step 5: Verify the zip

```bash
# Extract to a temporary location and verify structure
mkdir /tmp/testzip
cd /tmp/testzip
unzip /path/to/[PLUGIN_NAME]-[version].zip

# Should see:
# [PLUGIN_MAPNAME]/
#   version.php
#   lib.php
#   classes/
#   ...
```

### Step 6: Upload to Moodle Plugins Directory (optional)

If distributing via **moodle.org/plugins**:

1. Go to https://moodle.org/plugins/
2. Log in or create an account
3. Create plugin project (first time) or update existing one
4. Upload zip file
5. Fill in changelog and release notes
6. Submit for review

**Before submitting, verify the plugin contribution checklist:** https://moodledev.io/general/community/plugincontribution/checklist

### Step 7: Document the release

Add to CHANGELOG or release history:

```
## Version 1.0.0 — 2025-05-20

### New Features
- Feature X added
- Feature Y added

### Bug Fixes
- Fixed issue #1
- Fixed issue #2

### Breaking Changes
(none)

### Requires
- Moodle 4.5 LTS or higher
- PHP 8.1 or higher
```

---

## CI/CD (GitHub Actions via moodle-plugin-ci)

Automate the pre-release checklist above with [`moodlehq/moodle-plugin-ci`](https://github.com/moodlehq/moodle-plugin-ci) instead of running each tool by hand. This is the same toolchain Moodle core itself uses. Which checks must run, and which of them block, is decided in [context/tech/testing.md](../tech/testing.md) (Definition of done); the workflow below runs that list.

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-22.04
    services:
      mysql:
        image: mysql:8.4            # same engine as production
        env:
          MYSQL_ALLOW_EMPTY_PASSWORD: "true"
        ports: ["3306:3306"]
        options: >-
          --health-cmd="mysqladmin ping" --health-interval=10s
          --health-timeout=5s --health-retries=5
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: 'postgres'
          POSTGRES_HOST_AUTH_METHOD: 'trust'
        ports: ["5432:5432"]
        options: >-
          --health-cmd="pg_isready" --health-interval=10s
          --health-timeout=5s --health-retries=5
    strategy:
      fail-fast: false
      matrix:
        include:
          # Lowest and highest Moodle branch from $plugin->requires / $plugin->supported.
          # Check branch names and PHP versions at moodledev.io/general/releases.
          - { php: '8.2', moodle-branch: 'MOODLE_405_STABLE', database: 'mysqli' }
          - { php: '8.3', moodle-branch: '[HIGHEST_MOODLE_BRANCH]', database: 'mysqli' }
          - { php: '8.3', moodle-branch: '[HIGHEST_MOODLE_BRANCH]', database: 'pgsql' }
    steps:
      - uses: actions/checkout@v4
        with:
          path: plugin

      - uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          extensions: pgsql, mysqli, zip, gd, intl, soap
          ini-values: max_input_vars=5000
          coverage: none            # use pcov in a separate run to measure coverage
          tools: composer

      - name: Install moodle-plugin-ci
        run: |
          composer create-project -n --no-dev --ignore-platform-reqs moodlehq/moodle-plugin-ci ci ^4
          echo $(cd ci; pwd)/bin >> $GITHUB_PATH
          echo $(cd ci; pwd)/vendor/bin >> $GITHUB_PATH

      - name: Install Moodle + plugin
        run: moodle-plugin-ci install -vvv --plugin ./plugin --db-host=127.0.0.1
        env:
          DB: ${{ matrix.database }}
          MOODLE_BRANCH: ${{ matrix.moodle-branch }}

      # Fast to slow, same order as the Definition of done. phpmd advises and does not block.
      - run: moodle-plugin-ci phplint
      - run: moodle-plugin-ci validate
      - run: moodle-plugin-ci phpcs --max-warnings 0
      - run: moodle-plugin-ci phpmd
        continue-on-error: true
      - run: moodle-plugin-ci phpdoc --max-warnings 0
      - run: moodle-plugin-ci savepoints
      - run: moodle-plugin-ci mustache
      - run: moodle-plugin-ci grunt --max-lint-warnings 0
      - run: moodle-plugin-ci phpunit --fail-on-warning
      - run: moodle-plugin-ci behat --profile chrome --auto-rerun 2
```

Avetica hosts plugins on GitLab first: use the same commands, services and matrix in `.gitlab-ci.yml`. Only the wrapper (runner, services syntax) differs, not the list of checks.

Run the matrix against the Moodle branches this plugin actually declares support for in `version.php`'s `$plugin->supported` — not every branch Moodle has ever shipped.

---

## Distribution methods

| Method | Setup | Cost | Audience |
|---|---|---|---|
| **moodle.org/plugins** | Free, automated via web form | Free | All Moodle administrators |
| **GitHub releases** | Git tag + zip upload | Free | Developers, tech-savvy admins |
| **Internal repository** | Private Git/zip store | Free (hosting) | Your organization only |
| **Package manager** | Custom integration | Varies | Specific platforms |

---

## Versioning strategy

**For a new plugin:**
- Start with `1.0.0-beta` (MATURITY_BETA)
- After feedback, release `1.0.0` (MATURITY_STABLE)

**For subsequent releases:**
- **Patch** (1.0.1): bug fixes, no feature additions
- **Minor** (1.1.0): new features, backward-compatible
- **Major** (2.0.0): breaking changes, significant refactor

**For long-term support:**
- If targeting multiple Moodle versions, increment patch for each Moodle release
  - Example: 1.0.0 for Moodle 4.5, 1.0.1 for Moodle 5.0, etc.

---

## Rollback procedure

If a release introduces a critical bug:

1. **Immediately create a hotfix** on a new branch from the broken release tag
2. **Fix the bug** and test thoroughly
3. **Bump patch version** (e.g., 1.0.1)
4. **Tag and release** the hotfix
5. **Notify users** of the critical update

---
