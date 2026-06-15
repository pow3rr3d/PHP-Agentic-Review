---
name: review-php
command: /review-php
description: Analyzes all changed files (staged + unstaged) and checks compliance with project standards. Loads local standards (.claude/STANDARDS.md) first, falls back to global standards (~/.claude/STANDARDS.md).
allowed-tools: Bash(git diff *), Bash(git status *), Bash(phpstan *), Bash(grep *), Bash(find *), Bash(cat *), Bash(test *)
---

# /review-php — Pre-commit Code Review

## Step 1 — Retrieve all changes

```bash
git status
git diff HEAD
```

If no changes are detected, output exactly this and stop:
```
⚠️ No changes found. Modify some files before using /review-php.
```

## Step 2 — Load standards (fallback: local → global)

```bash
test -f .claude/STANDARDS.md && cat .claude/STANDARDS.md
test -f ~/.claude/STANDARDS.md && cat ~/.claude/STANDARDS.md
```

- If `.claude/STANDARDS.md` exists → use this file only
- Otherwise → use `~/.claude/STANDARDS.md`
- If neither exists → output `❌ No STANDARDS.md found.` and stop
- If local file starts with `extends: global` → merge both files (global first, local overrides by id)

## Step 3 — Run all checks silently

Run every rule in the loaded standards. Collect all results before producing any output. Do not stream partial results.

## Step 4 — Output the report

Output the report using **this exact format, character by character**. No deviations.

---

### Header block

```
╔══════════════════════════════════════════╗
║           🔍 CODE REVIEW REPORT          ║
╚══════════════════════════════════════════╝
```

One blank line, then the standards source on one line:
- `🌐 Standards: global (~/.claude/STANDARDS.md)`
- `📁 Standards: local (.claude/STANDARDS.md)`
- `🔀 Standards: global + local override (.claude/STANDARDS.md)`

One blank line, then the file list:
```
📁 Files analyzed: N file(s)
   • path/to/file.php
   • path/to/other.php
```

---

### Separator

Always this exact string, on its own line, before and after the results block:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Results block

**Sorting rules — mandatory:**
- All PASS lines first, then all FAIL lines, then all WARN lines
- Within the PASS group: sort by rule ID alphabetically (PHP-001, PHP-002, CLEAN-001, ...)
- Within the FAIL group: sort by line number ascending
- Within the WARN group: sort by line number ascending

**Each line uses one of these three templates. Copy the template exactly, replace the placeholders:**

PASS template:
```
✅ PASS  [PHP-001]  Label of the rule
```

FAIL template (static):
```
❌ FAIL  [PHP-001]  Label of the rule
         └─ line 42: `$x = null;`
```

FAIL template (tool — no line number):
```
❌ FAIL  [PHPSTAN-001]  Label of the rule
         └─ command output summary
```

WARN template:
```
⚠️ WARN  [CLEAN-010]  Label of the rule (non-blocking)
         └─ line 42: description
```

Rules:
- The space pattern in the templates is the source of truth — reproduce it character for character
- `(non-blocking)` is always appended to WARN labels, never to PASS or FAIL labels
- Multiple `└─` sub-lines are allowed for the same rule, one per line, same indentation
- Never add section headers between result lines

---

### Footer block

Separator line, then:
```
📊 FINAL RESULT
   ✅ Passed  : N
   ❌ Failed  : N
   ⚠️ Warnings: N
```

One blank line, then status — exactly one of:
```
→ Status: ✅ READY TO COMMIT
```
```
→ Status: ❌ FIX BEFORE COMMIT
```

If status is ❌, add one blank line then a summary block:
```
N blocking issue(s) to resolve:
1. file.php line N — description of fix
2. file.php line N — description of fix
```
Blocking issues must be listed **in ascending line number order**.

If there are also warnings, add one blank line then:
```
N non-blocking warning(s):
1. description
```

---

## Strict output rules

- **Never** add section headers (no `🔴 PHP`, no `🟡 Clean code`, etc.) inside the results block
- **Never** mix PASS / FAIL / WARN — always three separate groups in order
- **Never** output anything between the header block and the first separator
- **Never** output anything between the last separator and the footer block
- **Never** modify any file
- **Never** stream partial output — complete all checks first, then output the full report at once
- A `blocking: true` rule that fails → status is ❌
- A missing or crashed `type: tool` → always ⚠️ WARN, never ❌ FAIL
