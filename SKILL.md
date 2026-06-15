---
name: review-php
command: /review-php
description: Analyzes all changed files (staged + unstaged) and checks compliance with project standards. Loads local standards (.claude/STANDARDS.md) first, falls back to global standards (~/.claude/STANDARDS.md).
allowed-tools: Bash(git diff *), Bash(git status *), Bash(phpstan *), Bash(grep *), Bash(find *), Bash(cat *), Bash(test *)
---

# /review-php — Pre-commit Code Review

## Goal
Analyze **all changed files** (staged and unstaged) and check their compliance with the development standards defined in `STANDARDS.md`.

## Step 1 — Retrieve all changes

Run the following commands to get every modified file and their diff:

```bash
git status
git diff HEAD
```

If no changes are detected, display:
> ⚠️ No changes found. Modify some files before using /review-php.

Then stop execution.

## Step 2 — Load standards (fallback: local → global)

Look for standards in this priority order:

```bash
# 1. Project-level local standards (high priority)
test -f .claude/STANDARDS.md && cat .claude/STANDARDS.md

# 2. Otherwise, global standards
test -f ~/.claude/STANDARDS.md && cat ~/.claude/STANDARDS.md
```

Loading rules:
- If `.claude/STANDARDS.md` exists in the current project → use **this file only**
- Otherwise → use `~/.claude/STANDARDS.md`
- If neither file exists → display an error and stop

Always indicate the source at the top of the report:
- `📁 Standards: local (.claude/STANDARDS.md)`
- `🌐 Standards: global (~/.claude/STANDARDS.md)`

### Hybrid standards (local + global)

If the local file starts with the following directive:

```yaml
extends: global
```

Then load **both files**: global rules first, local rules second. If two rules share the same `id`, the local rule overrides the global one. Indicate in the report:
- `🔀 Standards: global + local override (.claude/STANDARDS.md)`

## Step 3 — Analyze each standard

**Analyze all changed lines** returned by `git diff HEAD` — both staged and unstaged changes.

For each rule defined in the loaded standards file, apply the check against the full diff.

There are 3 verification types:

**`type: static`** — Grep/regex on added lines of the diff (lines starting with `+`, excluding `+++`)
- If `match_is_error: true` → finding the pattern = error
- If `match_is_error: false` → not finding the pattern = error
- If `file_filter` is set → only analyze files whose name matches this pattern

**`type: tool`** — Shell command executed from the project root
- `command_success: exit_0` → success if exit code = 0
- If the command is missing or fails to run → `⚠️ WARN` (non-blocking) with the error message

**`type: semantic`** — LLM analysis of the diff
- Apply the `prompt` defined in the rule to the full diff content
- The prompt must return a JSON: `{"found": true/false, "occurrences": ["description..."]}`
- `found: true` = the rule detected a problem

## Step 4 — Generate the report

```
╔══════════════════════════════════════════╗
║           🔍 CODE REVIEW REPORT          ║
╚══════════════════════════════════════════╝

🌐 Standards: global (~/.claude/STANDARDS.md)   ← or local, or hybrid
📁 Files analyzed: X file(s)
   • src/Service/MyService.php
   • ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ PASS  [PHP-001]  Booleans are uppercase (TRUE / FALSE)
❌ FAIL  [PHP-002]  NULL is uppercase
         └─ Detail: line 42 — `if ($value === null)`
⚠️ WARN  [STYLE-001]  PHP methods under 50 lines (non-blocking)
         └─ Suggestion: MyService::process() — 67 lines

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 FINAL RESULT
   ✅ Passed  : X
   ❌ Failed  : X  ← BLOCKING
   ⚠️  Warnings: X

→ Status: ✅ READY TO COMMIT  |  ❌ FIX BEFORE COMMIT
```

## Behavior rules

- **Never** modify any file automatically — review only
- Standards marked `blocking: true` set the final status to ❌
- Always display the line number and code excerpt for `static` errors
- If a `type: tool` command is not installed → `⚠️ WARN` non-blocking, never a fatal error
