extends: global

# 📋 Local standards — my-symfony-project

These rules are added on top of the global standards (~/.claude/STANDARDS.md).
Rules sharing the same `id` override the corresponding global rule.

---

## 🔴 Symfony — Project-specific conventions

```yaml
id: SYM-001
label: "Services are declared in snake_case in services.yaml"
blocking: true
type: static
file_filter: "services\.yaml$"
pattern: "[A-Z]"
match_is_error: true
```

```yaml
id: SYM-002
label: "No dependency injection via public properties (use constructor instead)"
blocking: true
type: semantic
file_filter: "\.php$"
prompt: |
  Analyze the diff. Look for dependency injections via public properties
  (e.g. `public MyService $service` without a constructor).
  Reply ONLY with a JSON: {"found": true/false, "occurrences": ["line X: ..."]}.
```

---

## 🟡 Global rule override

```yaml
# Make CLEAN-001 non-blocking for this project (overrides the global rule)
id: CLEAN-001
label: "TODO in code (warning only for this project)"
blocking: false
type: static
pattern: "TODO"
match_is_error: true
```
