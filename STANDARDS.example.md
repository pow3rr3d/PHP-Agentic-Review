extends: global

# 📋 Standards locaux — mon-projet-symfony

Ces règles s'ajoutent aux standards globaux (~/.claude/STANDARDS.md).
Les règles avec le même `id` écrasent la règle globale correspondante.

---

## 🔴 Symfony — Conventions spécifiques

```yaml
id: SYM-001
label: "Les services sont déclarés en snake_case dans services.yaml"
blocking: true
type: static
file_filter: "services\.yaml$"
pattern: "[A-Z]"
match_is_error: true
```

```yaml
id: SYM-002
label: "Pas d'injection par propriété publique (utiliser le constructeur)"
blocking: true
type: semantic
file_filter: "\.php$"
prompt: |
  Analyse le diff. Cherche des injections de dépendances via propriétés publiques
  (ex: `public MyService $service` sans constructeur).
  Réponds UNIQUEMENT par un JSON: {"found": true/false, "occurrences": ["ligne X: ..."]}.
```

---

## 🟡 Override d'une règle globale

```yaml
# Ici on rend CLEAN-001 non-bloquant pour ce projet (override de la règle globale)
id: CLEAN-001
label: "TODO dans le code (warning seulement pour ce projet)"
blocking: false
type: static
pattern: "TODO"
match_is_error: true
```
