# AGENTS.md Audit Checklist

> Quarterly review template to keep your AGENTS.md accurate, effective, and secure.

---

## When to Audit

- **Quarterly** (minimum) — schedule a calendar reminder
- **After major dependency updates** — new framework versions, new tools
- **After project restructuring** — directory changes, architecture changes
- **When agents repeatedly make mistakes** — sign of stale or missing rules
- **When team conventions change** — new patterns, new standards

---

## The Audit Checklist

### ✅ Section 1: File Health

| Check | Status | Notes |
|-------|--------|-------|
| File exists at repository root | ☐ | |
| File is named `AGENTS.md` (exact case) | ☐ | |
| File is committed to git (not ignored) | ☐ | |
| File size is under 200 lines | ☐ | Current: ___ lines |
| No hidden Unicode characters | ☐ | Run scanner |
| CODEOWNERS entry exists for AGENTS.md | ☐ | |
| Last updated within 90 days | ☐ | Last: _________ |

### ✅ Section 2: Setup Commands

| Check | Status | Notes |
|-------|--------|-------|
| Install command works | ☐ | Run: `__________` |
| Dev command works | ☐ | Run: `__________` |
| Test command works | ☐ | Run: `__________` |
| Build command works | ☐ | Run: `__________` |
| Lint command works | ☐ | Run: `__________` |
| Single-file test command works | ☐ | Run: `__________` |
| Package manager matches lockfile | ☐ | PM: __________ |
| "NEVER use [alt PM]" included | ☐ | |

### ✅ Section 3: Stack & Versions

| Check | Status | Notes |
|-------|--------|-------|
| Framework versions are current | ☐ | |
| Language version matches project | ☐ | |
| "NOT used" list is accurate | ☐ | |
| No deprecated tools mentioned | ☐ | |

### ✅ Section 4: Project Structure

| Check | Status | Notes |
|-------|--------|-------|
| All listed directories exist | ☐ | |
| No missing important directories | ☐ | |
| Directory annotations are accurate | ☐ | |
| No individual files listed (only dirs) | ☐ | |

### ✅ Section 5: Conventions

| Check | Status | Notes |
|-------|--------|-------|
| All conventions are still current | ☐ | |
| No contradictory conventions | ☐ | |
| Each convention is specific and testable | ☐ | |
| Convention count is 5-10 (not bloated) | ☐ | Count: ___ |
| No vague rules ("clean code", etc.) | ☐ | |

### ✅ Section 6: Boundaries

| Check | Status | Notes |
|-------|--------|-------|
| Boundary section exists | ☐ | |
| All boundaries use "NEVER" (strong language) | ☐ | |
| Protected directories/files are correct | ☐ | |
| Secrets boundary included | ☐ | |
| No soft language ("avoid", "try not to") | ☐ | |
| Boundary count is 5-10 | ☐ | Count: ___ |

### ✅ Section 7: Testing

| Check | Status | Notes |
|-------|--------|-------|
| Test framework matches actual usage | ☐ | |
| Test command is correct | ☐ | |
| Coverage targets are current | ☐ | |
| Testing patterns match team standard | ☐ | |

### ✅ Section 8: Security

| Check | Status | Notes |
|-------|--------|-------|
| No credentials in AGENTS.md | ☐ | |
| No internal URLs or IPs | ☐ | |
| Safety rules present | ☐ | |
| Indirect injection rules present | ☐ | |

### ✅ Section 9: Cross-Tool Compatibility

| Check | Status | Notes |
|-------|--------|-------|
| No tool-specific instructions in AGENTS.md | ☐ | |
| Tool-specific files reference AGENTS.md | ☐ | |
| No duplicate rules between files | ☐ | |
| Tested with team's primary AI tool | ☐ | Tool: _________ |

### ✅ Section 10: Monorepo (if applicable)

| Check | Status | Notes |
|-------|--------|-------|
| Root AGENTS.md is lean (< 60 lines) | ☐ | |
| Per-package AGENTS.md files are current | ☐ | |
| No contradictions between root and packages | ☐ | |
| Cross-package import boundaries stated | ☐ | |

---

## Audit Summary

```
Date: _______________
Auditor: _______________

Total checks: ___
Passed: ___
Failed: ___
N/A: ___

Action items:
1. _______________________________________________
2. _______________________________________________
3. _______________________________________________
4. _______________________________________________
5. _______________________________________________

Next audit scheduled: _______________
```

---

## Quick Fix Priority

When audit reveals issues, fix in this order:

| Priority | What to Fix | Why |
|----------|------------|-----|
| 🔴 P0 | Broken commands | Agent can't build/test at all |
| 🔴 P0 | Missing security rules | Risk of secret commits |
| 🟡 P1 | Stale stack versions | Generates deprecated code |
| 🟡 P1 | Wrong directory structure | Files created in wrong places |
| 🟡 P1 | Contradictory rules | Unpredictable agent behavior |
| 🟢 P2 | Missing conventions | Inconsistent code style |
| 🟢 P2 | Missing testing section | No tests generated |
| ⚪ P3 | Verbose content | Reduced compliance |
| ⚪ P3 | Missing "NOT used" list | Agent uses wrong tools |

---

## Automating the Audit

### CI Check (GitHub Actions)

```yaml
name: AGENTS.md Audit
on:
  schedule:
    - cron: '0 9 1 */3 *'  # First day of every quarter at 9 AM
  workflow_dispatch:

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check file exists
        run: test -f AGENTS.md

      - name: Check file size
        run: |
          lines=$(wc -l < AGENTS.md)
          echo "AGENTS.md: $lines lines"
          [ "$lines" -lt 300 ] || echo "::warning::AGENTS.md exceeds 300 lines"

      - name: Check required sections
        run: |
          for section in "Setup" "Conventions" "Boundaries"; do
            grep -q "## $section" AGENTS.md || echo "::error::Missing section: $section"
          done

      - name: Check for hidden Unicode
        run: |
          if grep -P '[\x{200B}-\x{200F}\x{202A}-\x{202E}\x{2066}-\x{2069}]' AGENTS.md; then
            echo "::error::Hidden Unicode characters detected"
            exit 1
          fi

      - name: Check freshness
        run: |
          last_modified=$(git log -1 --format=%ci -- AGENTS.md)
          echo "Last modified: $last_modified"
          days_ago=$(( ($(date +%s) - $(date -d "$last_modified" +%s)) / 86400 ))
          [ "$days_ago" -lt 90 ] || echo "::warning::AGENTS.md not updated in $days_ago days"

      - name: Verify commands exist in package.json
        run: |
          if [ -f package.json ]; then
            grep -oP 'pnpm \K\w+' AGENTS.md | while read script; do
              jq -e ".scripts.\"$script\"" package.json > /dev/null 2>&1 || \
                echo "::warning::Script '$script' in AGENTS.md not found in package.json"
            done
          fi
```

---

*This checklist is part of the [AGENTS.md Education & Learning Center](../README.md).*
