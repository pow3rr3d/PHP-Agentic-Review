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

```yaml
id: PHP-006
label: "No dd() or dump() (Symfony debug helpers) in committed code"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\b(dd|dump)\\s*\\("
match_is_error: true
```

---

## 🔴 Quality — Static analysis & Tests

```yaml
id: PHPSTAN-001
label: "0 PHPStan errors (level configured in phpstan.neon)"
blocking: true
type: tool
file_filter: "\.php$"
command: "vendor/bin/phpstan analyse --no-progress --error-format=table"
command_success: exit_0
```

```yaml
id: TEST-001
label: "All unit tests pass (PHPUnit)"
blocking: true
type: tool
command: "vendor/bin/phpunit --testsuit=unit --no-coverage"
command_success: exit_0
```

```yaml
id: TEST-002
label: "New code has corresponding test file"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for added or modified PHP classes that are NOT
  test classes themselves (not in a /tests/ or /Test/ directory, not suffixed with Test.php)
  and check whether a corresponding test file is mentioned anywhere in the diff.
  Flag classes that introduce new behavior but have no visible test counterpart.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["ClassName: no test file found in diff"]}.
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

```yaml
id: CLEAN-005
label: "No magic numbers (unnamed numeric literals)"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for magic numbers: raw numeric literals used directly
  in logic (e.g. `if ($age > 18)`, `sleep(3600)`) that should be named constants.
  Ignore obvious exceptions: 0, 1, array indexes, HTTP status codes with a comment.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: value — suggestion"]}.
```

```yaml
id: CLEAN-006
label: "No deeply nested code (max 3 levels)"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for added or modified code blocks with more than
  3 levels of nesting (if/foreach/while/try inside each other).
  Suggest early returns or extracted methods where applicable.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: description"]}.
```

```yaml
id: CLEAN-007
label: "Functions and methods have a single responsibility (do one thing)"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for added or modified methods that clearly do more
  than one thing: mixing data fetching + transformation + persistence, or business
  logic + formatting, etc.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["MethodName: description of multiple responsibilities"]}.
```

```yaml
id: CLEAN-008
label: "No functions with more than 3 parameters"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for added or modified functions/methods with more
  than 3 parameters. Suggest a DTO, array, or value object when relevant.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["MethodName: X parameters"]}.
```

```yaml
id: CLEAN-009
label: "No negative or double-negative conditionals"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for negative or double-negative boolean conditions
  that hurt readability (e.g. `!isNotValid()`, `if (!$user->isNotActive())`).
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: original → suggestion"]}.
```

```yaml
id: CLEAN-010
label: "Methods and variables have meaningful names (no abbreviations)"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for added variables, parameters, or methods with
  cryptic or abbreviated names (e.g. $tmp, $x, $mgr, $d, process2(), handleStuff()).
  Ignore well-known conventions ($i in loops, $e for exceptions).
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: name — suggestion"]}.
```

---

## 🟠 S.O.L.I.D — Detectable violations

```yaml
id: SOLID-S-001
label: "SRP — Class does not mix unrelated responsibilities"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for added or modified classes that mix unrelated
  responsibilities: e.g. a Service that also handles HTTP responses, a Repository
  that sends emails, an Entity that contains business logic and persistence.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["ClassName: description of mixed responsibilities"]}.
```

```yaml
id: SOLID-O-001
label: "OCP — No large switch/if-elseif chains that should use polymorphism"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for switch statements or if/elseif chains with 3+
  branches dispatching behavior based on a type, status, or role — a pattern that
  should typically use polymorphism or a strategy pattern instead.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: description — suggestion"]}.
```

```yaml
id: SOLID-L-001
label: "LSP — Overridden methods do not change expected behavior or throw unexpected exceptions"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for overridden methods in child classes that:
  - throw exceptions not declared in the parent
  - return types incompatible with the parent contract
  - ignore or silently drop parent behavior without clear justification
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["ClassName::method — description"]}.
```

```yaml
id: SOLID-I-001
label: "ISP — No interface with too many unrelated methods"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for added or modified interfaces with more than
  5 methods that cover unrelated concerns — a sign they should be split into
  smaller, more focused interfaces.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["InterfaceName: X methods — suggested split"]}.
```

```yaml
id: SOLID-D-001
label: "DIP — No direct instantiation of concrete dependencies (use injection)"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\bnew\\s+[A-Z][a-zA-Z]+(Service|Repository|Manager|Handler|Client)\\b"
match_is_error: true
```

```yaml
id: SOLID-D-002
label: "DIP — Dependencies are type-hinted against interfaces, not concrete classes"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff below. Look for constructor parameters or method arguments
  type-hinted against concrete classes (e.g. UserRepository, MailerService) instead
  of interfaces (e.g. UserRepositoryInterface, MailerInterface).
  Ignore scalar types, DTOs, and value objects.
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["ClassName::__construct — param $x should use InterfaceName"]}.
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
