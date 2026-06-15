# 📋 Standards globaux — /review

Ces règles s'appliquent à **tous tes projets** par défaut.

Pour un projet spécifique, tu as deux options :
- Créer `.claude/STANDARDS.md` dans le projet → remplace entièrement ces règles
- Créer `.claude/STANDARDS.md` avec `extends: global` en en-tête → ajoute des règles par-dessus celles-ci

---

## 🔴 PHP — Typage & Conventions

```yaml
id: PHP-001
label: "Les booléens sont en majuscules (TRUE / FALSE)"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\b(true|false)\\b"
match_is_error: true
```

```yaml
id: PHP-002
label: "NULL est en majuscules"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\bnull\\b"
match_is_error: true
```

```yaml
id: PHP-003
label: "Pas de var_dump() dans le code commité"
blocking: true
type: static
file_filter: "\.php$"
pattern: "var_dump\\s*\\("
match_is_error: true
```

```yaml
id: PHP-004
label: "Pas de print_r() dans le code commité"
blocking: true
type: static
file_filter: "\.php$"
pattern: "print_r\\s*\\("
match_is_error: true
```

```yaml
id: PHP-005
label: "Pas de die() ou exit() sans raison explicite"
blocking: false
type: static
file_filter: "\.php$"
pattern: "\\b(die|exit)\\s*\\("
match_is_error: true
```

---

## 🔴 Qualité — Analyse statique

```yaml
id: PHPSTAN-001
label: "0 erreur PHPStan (niveau configuré dans phpstan.neon)"
blocking: true
type: tool
file_filter: "\.php$"
command: "vendor/bin/phpstan analyse --no-progress --error-format=table"
command_success: exit_0
```

---

## 🟡 Code propre — Marqueurs temporaires

```yaml
id: CLEAN-001
label: "Pas de TODO dans le code commité"
blocking: true
type: static
pattern: "TODO"
match_is_error: true
```

```yaml
id: CLEAN-002
label: "Pas de FIXME dans le code commité"
blocking: true
type: static
pattern: "FIXME"
match_is_error: true
```

```yaml
id: CLEAN-003
label: "Pas de HACK dans le code commité"
blocking: false
type: static
pattern: "HACK"
match_is_error: true
```

```yaml
id: CLEAN-004
label: "Pas de code commenté (blocs suspects)"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyse le diff ci-dessous. Repère les blocs de code PHP commenté
  (lignes en // ou /* */ qui semblent être du code désactivé, pas de la documentation).
  Signale chaque occurrence avec le numéro de ligne approximatif.
  Réponds UNIQUEMENT par un JSON: {"found": true/false, "occurrences": ["ligne X: ..."]}.
```

---

## 🟡 Sécurité — Basique

```yaml
id: SEC-001
label: "Pas de clé API ou token en dur dans le code"
blocking: true
type: semantic
prompt: |
  Analyse le diff ci-dessous. Cherche des secrets potentiels : clés API, tokens,
  mots de passe, credentials en dur dans le code (pas dans des fichiers .env ou de config exemples).
  Réponds UNIQUEMENT par un JSON: {"found": true/false, "occurrences": ["ligne X: description"]}.
```

```yaml
id: SEC-002
label: "Pas de requête SQL construite par concaténation (injection)"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\$[a-zA-Z_]+\\s*\\.\\s*[\"'].*SELECT|INSERT|UPDATE|DELETE"
match_is_error: true
```

---

## 🟢 Style & lisibilité

```yaml
id: STYLE-001
label: "Les méthodes PHP font moins de 50 lignes"
blocking: false
type: semantic
file_filter: "\.php$"
prompt: |
  Analyse le diff ci-dessous. Identifie les méthodes PHP ajoutées ou modifiées
  qui font plus de 50 lignes de code (hors commentaires et lignes vides).
  Réponds UNIQUEMENT par un JSON: {"found": true/false, "occurrences": ["NomDeLaMethode: X lignes"]}.
```
