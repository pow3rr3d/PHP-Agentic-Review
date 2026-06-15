# /review-php — Code Review for Claude Code

A `/review-php` command for [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) that analyzes **all changed files** (staged and unstaged) and checks their compliance with configurable development standards.

---

## Usage

```
/review-php
```

Claude analyzes the full diff (`git diff HEAD`), applies every rule defined in `STANDARDS.md`, and generates a structured report:

```
╔══════════════════════════════════════════╗
║           🔍 CODE REVIEW REPORT          ║
╚══════════════════════════════════════════╝

🌐 Standards: global (~/.claude/STANDARDS.md)
📁 Files analyzed: 2 file(s)
   • src/Service/UserService.php
   • src/Repository/UserRepository.php

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ PASS  [PHP-001]  Booleans are uppercase (TRUE / FALSE)
✅ PASS  [PHP-002]  NULL is uppercase
✅ PASS  [CLEAN-001]  No TODO in committed code
❌ FAIL  [PHP-003]  No var_dump() in committed code
         └─ line 12: `var_dump($user);`
⚠️ WARN  [STYLE-001]  PHP methods under 50 lines (non-blocking)
         └─ line 34: UserService::process() — 67 lines

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 FINAL RESULT
   ✅ Passed  : 3
   ❌ Failed  : 1
   ⚠️  Warnings: 1

→ Status: ❌ FIX BEFORE COMMIT

1 blocking issue(s) to resolve:
1. UserService.php line 12 — remove var_dump($user)

1 non-blocking warning(s):
1. UserService::process() is 67 lines — consider splitting it
```

---

## Installation

### Global installation (recommended)

First of all, clone this repository!

Available across **all your projects**:

```bash
mkdir -p ~/.claude/skills/review-php
cp SKILL.md ~/.claude/skills/review-php/SKILL.md
cp STANDARDS.md ~/.claude/STANDARDS.md
```

### Local installation (current project only)

```bash
mkdir -p .claude/skills/review-php
cp SKILL.md .claude/skills/review-php/SKILL.md
cp STANDARDS.md .claude/STANDARDS.md
```

---

## File structure

```
# Global installation
~/.claude/
├── STANDARDS.md                 ← default standards for all projects
└── skills/
    └── review-php/
        └── SKILL.md             ← command engine (do not modify)

# Per project (optional)
my-project/
└── .claude/
    └── STANDARDS.md             ← local standards (replaces or extends global)
```

---

## Managing standards per project

The command loads standards in this priority order:

| Situation | Standards loaded | Shown in report |
|---|---|---|
| No local `.claude/STANDARDS.md` | `~/.claude/STANDARDS.md` | `🌐 Standards: global` |
| Local `.claude/STANDARDS.md` | Local file only | `📁 Standards: local` |
| Local `.claude/STANDARDS.md` with `extends: global` | Global + local merged | `🔀 Standards: global + local override` |

### `extends: global` mode

Add `extends: global` as the first line of the local `STANDARDS.md` to inherit all global rules while adding or overriding specific ones:

```markdown
extends: global

# Local standards — my-symfony-project

...additional rules or overrides...
```

When two rules share the same `id`, the **local rule overrides the global one**.

---

## Configuring standards

Everything happens in `STANDARDS.md`. Each rule is a YAML block:

```yaml
id: PHP-001
label: "Booleans are uppercase (TRUE / FALSE)"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\b(true|false)\\b"
match_is_error: true
```

### Available fields

| Field | Description |
|---|---|
| `id` | Unique identifier (e.g. `PHP-001`) |
| `label` | Description shown in the report |
| `blocking` | `true` = blocks commit if the rule fails |
| `type` | Verification type: `static`, `tool`, or `semantic` |
| `file_filter` | Optional — restrict to files matching this pattern (e.g. `\.php$`) |

### The 3 verification types

#### `static` — Grep/regex on the diff

Searches for a pattern in added lines of the diff.

```yaml
id: CLEAN-001
label: "No TODO in committed code"
blocking: true
type: static
pattern: "TODO"
match_is_error: true   # true = finding the pattern is an error
                       # false = not finding the pattern is an error
```

#### `tool` — Shell command

Runs a command from the project root. Success if exit code is 0. If the tool is not installed, the rule becomes a non-blocking `⚠️ WARN`.

```yaml
id: PHPSTAN-001
label: "0 PHPStan errors (level configured in phpstan.neon)"
blocking: true
type: tool
file_filter: "\.php$"
command: "vendor/bin/phpstan analyse --no-progress --error-format=table"
command_success: exit_0
```

#### `semantic` — Analysis by Claude

Claude analyzes the diff itself based on the prompt instructions. The prompt must return a JSON `{"found": bool, "occurrences": [...]}`.

```yaml
id: SEC-001
label: "No hardcoded API key or token"
blocking: true
type: semantic
prompt: |
  Analyze the diff below. Look for potential secrets: API keys, tokens,
  passwords, hardcoded credentials (not in .env or example config files).
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: description"]}.
```

---

## Default standards

| ID | Rule | Type | Blocking |
|---|---|---|---|
| `PHP-001` | Booleans uppercase (`TRUE` / `FALSE`) | static | ✅ |
| `PHP-002` | `NULL` uppercase | static | ✅ |
| `PHP-003` | No `var_dump()` | static | ✅ |
| `PHP-004` | No `print_r()` | static | ✅ |
| `PHP-005` | No `die()` / `exit()` | static | ⚠️ |
| `PHP-006` | No `dd()` / `dump()` (Symfony) | static | ✅ |
| `PHPSTAN-001` | 0 PHPStan errors | tool | ✅ |
| `TEST-001` | All unit tests pass (PHPUnit) | tool | ✅ |
| `TEST-002` | New code has corresponding test file | semantic | ⚠️ |
| `CLEAN-001` | No `TODO` | static | ✅ |
| `CLEAN-002` | No `FIXME` | static | ✅ |
| `CLEAN-003` | No `HACK` | static | ⚠️ |
| `CLEAN-004` | No commented-out code | semantic | ⚠️ |
| `CLEAN-005` | No magic numbers | semantic | ⚠️ |
| `CLEAN-006` | No deeply nested code (max 3 levels) | semantic | ⚠️ |
| `CLEAN-007` | Methods have a single responsibility | semantic | ⚠️ |
| `CLEAN-008` | No functions with more than 3 parameters | semantic | ⚠️ |
| `CLEAN-009` | No negative or double-negative conditionals | semantic | ⚠️ |
| `CLEAN-010` | Meaningful names (no abbreviations) | semantic | ⚠️ |
| `SOLID-S-001` | SRP — No mixed responsibilities | semantic | ⚠️ |
| `SOLID-O-001` | OCP — No large switch/if-elseif chains | semantic | ⚠️ |
| `SOLID-L-001` | LSP — Overridden methods respect parent contract | semantic | ⚠️ |
| `SOLID-I-001` | ISP — No bloated interfaces | semantic | ⚠️ |
| `SOLID-D-001` | DIP — No direct instantiation of concrete dependencies | static | ✅ |
| `SOLID-D-002` | DIP — Dependencies typed against interfaces | semantic | ⚠️ |
| `SEC-001` | No hardcoded secrets | semantic | ✅ |
| `SEC-002` | No SQL injection via concatenation | static | ✅ |
| `STYLE-001` | PHP methods under 50 lines | semantic | ⚠️ |

---

## Adding a rule

1. Open `~/.claude/STANDARDS.md` (or `.claude/STANDARDS.md` in your project)
2. Copy an existing YAML block of the same type
3. Assign a unique `id`
4. For `type: static`: test your regex on [regex101.com](https://regex101.com) in PHP mode
5. For `type: tool`: make sure the command runs from the project root
6. For `type: semantic`: the prompt must return `{"found": bool, "occurrences": [...]}`

---

## Behavior

- The command **never modifies any file** — review only
- All changes are analyzed (`git diff HEAD`), both staged and unstaged
- Results are output in three fixed groups: all PASS first, then FAIL, then WARN
- Within each group, rules are sorted by line number ascending
- A `blocking: true` rule that fails sets the final status to ❌
- A missing `type: tool` command gives a non-blocking `⚠️ WARN`, never a fatal error
