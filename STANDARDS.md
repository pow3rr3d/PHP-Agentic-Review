# 📋 Global standards — /review-php

These rules apply to **all your projects** by default.

For a specific project, you have two options:
- Create `.claude/STANDARDS.md` in the project → fully replaces these rules
- Create `.claude/STANDARDS.md` with `extends: global` at the top → adds rules on top of these

---

## 🔴 PHP — Typing & Conventions

```yaml
id: PHP-001
label: "Booleans are uppercase (TRUE / FALSE)"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\b(true|false)\\b"
match_is_error: true
```

```yaml
id: PHP-002
label: "NULL is uppercase"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\bnull\\b"
match_is_error: true
```

```yaml
id: PHP-003
label: "No var_dump() in committed code"
blocking: true
type: static
file_filter: "\.php$"
pattern: "var_dump\\s*\\("
match_is_error: true
```

```yaml
id: PHP-004
label: "No print_r() in committed code"
blocking: true
type: static
file_filter: "\.php$"
pattern: "print_r\\s*\\("
match_is_error: true
```

```yaml
id: PHP-005
label: "No die() or exit() without explicit reason"
blocking: false
type: static
file_filter: "\.php$"
pattern: "\\b(die|exit)\\s*\\("
match_is_error: true
```

---

## 🔴 Quality — Static analysis

```yaml
id: PHPSTAN-001
label: "0 PHPStan errors (level configured in phpstan.neon)"
blocking: true
type: tool
file_filter: "\.php$"
command: "vendor/bin/phpstan analyse --no-progress --error-format=table"
command_success: exit_0
```

---

## 🟡 Clean code — Temporary markers

```yaml
id: CLEAN-001
label: "No TODO in committed code"
blocking: true
type: static
pattern: "TODO"
match_is_error: true
```

```yaml
id: CLEAN-002
label: "No FIXME in committed code"
blocking: true
type: static
pattern: "FIXME"
match_is_error: true
```

```yaml
id: CLEAN-003
label: "No HACK in committed code"
blocking: false
type: static
pattern: "HACK"
match_is_error: true
```

```yaml
id: CLEAN-004
label: "No commented-out code (suspicious blocks)"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for commented-out PHP code blocks
  (lines with // or /* */ that look like disabled code, not documentation).
  Report each occurrence with the approximate line number.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: ..."]}.
```

---

## 🟡 Security — Basics

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

```yaml
id: SEC-002
label: "No SQL query built by concatenation (injection risk)"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\$[a-zA-Z_]+\\s*\\.\\s*[\"'].*SELECT|INSERT|UPDATE|DELETE"
match_is_error: true
```

---

## 🟢 Style & readability

```yaml
id: STYLE-001
label: "PHP methods under 50 lines"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Identify added or modified PHP methods
  that exceed 50 lines of code (excluding comments and blank lines).
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["MethodName: X lines"]}.
```
