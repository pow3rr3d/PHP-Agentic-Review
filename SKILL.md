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

**Rules are always output in two groups, in this order:**
1. All PASS rules first
2. Then all FAIL rules
3. Then all WARN rules

**Each PASS line — exact format:**
```
✅ PASS  [ID]  Label
```
- Two spaces between `PASS` and `[ID]`
- Two spaces between `]` and the label
- No sub-lines for PASS

**Each FAIL line — exact format:**
```
❌ FAIL  [ID]  Label
         └─ line N: `code excerpt`
```
- Two spaces between `FAIL` and `[ID]`
- Two spaces between `]` and the label
- Sub-line indented with exactly 9 spaces then `└─`
- Always include line number and code excerpt for `static` rules
- For `tool` rules: include the command output summary instead of a line reference
- Multiple sub-lines allowed, each on its own line with the same indentation

**Each WARN line — exact format:**
```
⚠️ WARN  [ID]  Label (non-blocking)
         └─ line N: description
```
- Two spaces between `WARN` and `[ID]`
- Two spaces between `]` and the label
- Always append ` (non-blocking)` to the label
- Sub-line indented with exactly 9 spaces then `└─`
- Multiple sub-lines allowed

---

### Footer block

Separator line, then:
```
📊 FINAL RESULT
   ✅ Passed  : N
   ❌ Failed  : N
   ⚠️  Warnings: N
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
