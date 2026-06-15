# /review — Code Review pour Claude Code

Commande `/review` pour [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) qui analyse le code stagé (`git add`) avant chaque commit et vérifie sa conformité à des standards de développement configurables.

---

## Fonctionnement

```
git add src/MonFichier.php
/review
```

Claude analyse le diff stagé, applique chaque règle définie dans `STANDARDS.md`, et produit un rapport structuré :

```
╔══════════════════════════════════════════╗
║           🔍 CODE REVIEW REPORT          ║
╚══════════════════════════════════════════╝

🌐 Standards : globaux (~/.claude/STANDARDS.md)
📁 Fichiers analysés : 2 fichier(s)
   • src/Service/UserService.php
   • src/Repository/UserRepository.php

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ PASS  [PHP-001]  Les booléens sont en majuscules (TRUE / FALSE)
❌ FAIL  [PHP-002]  NULL est en majuscules
         └─ Détail : ligne 42 — `if ($value === null)`
✅ PASS  [CLEAN-001]  Pas de TODO dans le code commité
⚠️ WARN  [STYLE-001]  Les méthodes PHP font moins de 50 lignes
         └─ Suggestion : UserService::process() — 67 lignes

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 RÉSULTAT FINAL
   ✅ Réussis  : 3
   ❌ Échoués  : 1
   ⚠️  Warnings : 1

→ Statut : ❌ À CORRIGER AVANT COMMIT
```

---

## Installation

### Installation globale (recommandée)

Disponible dans **tous tes projets** :

```bash
mkdir -p ~/.claude/skills/review
cp SKILL.md ~/.claude/skills/review/SKILL.md
cp STANDARDS.md ~/.claude/STANDARDS.md
```

### Installation locale (projet uniquement)

```bash
mkdir -p .claude/skills/review
cp SKILL.md .claude/skills/review/SKILL.md
cp STANDARDS.md .claude/STANDARDS.md
```

---

## Structure des fichiers

```
# Installation globale
~/.claude/
├── STANDARDS.md                 ← standards par défaut pour tous les projets
└── skills/
    └── review/
        └── SKILL.md             ← moteur de la commande (ne pas modifier)

# Par projet (optionnel)
mon-projet/
└── .claude/
    └── STANDARDS.md             ← standards locaux (remplace ou étend le global)
```

---

## Gestion des standards par projet

La commande charge les standards selon cette priorité :

| Situation | Standards chargés | Affiché dans le rapport |
|---|---|---|
| Pas de `.claude/STANDARDS.md` local | `~/.claude/STANDARDS.md` | `🌐 Standards : globaux` |
| `.claude/STANDARDS.md` local | Fichier local uniquement | `📁 Standards : locaux` |
| `.claude/STANDARDS.md` avec `extends: global` | Global + local fusionnés | `🔀 Standards : globaux + override local` |

### Mode `extends: global`

Ajoute `extends: global` en première ligne du `STANDARDS.md` local pour hériter des règles globales tout en les complétant ou les surchargeant :

```markdown
extends: global

# Standards locaux — mon-projet-symfony

...règles supplémentaires ou overrides...
```

En cas de conflit d'`id`, la règle **locale écrase la règle globale**.

---

## Configurer les standards

Tout se passe dans `STANDARDS.md`. Chaque règle est un bloc YAML :

```yaml
id: PHP-001
label: "Les booléens sont en majuscules (TRUE / FALSE)"
blocking: true
type: static
file_filter: "\.php$"
pattern: "\\b(true|false)\\b"
match_is_error: true
```

### Champs disponibles

| Champ | Description |
|---|---|
| `id` | Identifiant unique (ex: `PHP-001`) |
| `label` | Description affichée dans le rapport |
| `blocking` | `true` = bloque le commit si la règle échoue |
| `type` | Type de vérification : `static`, `tool`, ou `semantic` |
| `file_filter` | Optionnel — restreindre aux fichiers matchant ce pattern (ex: `\.php$`) |

### Les 3 types de vérification

#### `static` — Grep/regex sur le diff

Recherche un pattern dans les lignes ajoutées du diff.

```yaml
id: CLEAN-001
label: "Pas de TODO dans le code commité"
blocking: true
type: static
pattern: "TODO"
match_is_error: true   # true = trouver le pattern est une erreur
                       # false = ne pas trouver le pattern est une erreur
```

#### `tool` — Commande shell

Exécute une commande depuis la racine du projet. Succès si le code de retour est 0. Si l'outil n'est pas installé, la règle passe en `⚠️ WARN` non bloquant.

```yaml
id: PHPSTAN-001
label: "0 erreur PHPStan"
blocking: true
type: tool
file_filter: "\.php$"
command: "vendor/bin/phpstan analyse --no-progress --error-format=table"
command_success: exit_0
```

#### `semantic` — Analyse par Claude

Claude analyse lui-même le diff selon les instructions du prompt. Le prompt doit retourner un JSON `{"found": bool, "occurrences": [...]}`.

```yaml
id: SEC-001
label: "Pas de clé API ou token en dur"
blocking: true
type: semantic
prompt: |
  Analyse le diff ci-dessous. Cherche des secrets potentiels : clés API, tokens,
  mots de passe, credentials en dur (pas dans des fichiers .env ou de config exemples).
  Réponds UNIQUEMENT par un JSON: {"found": true/false, "occurrences": ["ligne X: description"]}.
```

---

## Standards inclus par défaut

| ID | Règle | Type | Bloquant |
|---|---|---|---|
| `PHP-001` | Booléens en majuscules (`TRUE` / `FALSE`) | static | ✅ |
| `PHP-002` | `NULL` en majuscules | static | ✅ |
| `PHP-003` | Pas de `var_dump()` | static | ✅ |
| `PHP-004` | Pas de `print_r()` | static | ✅ |
| `PHP-005` | Pas de `die()` / `exit()` | static | ⚠️ |
| `PHPSTAN-001` | 0 erreur PHPStan | tool | ✅ |
| `CLEAN-001` | Pas de `TODO` | static | ✅ |
| `CLEAN-002` | Pas de `FIXME` | static | ✅ |
| `CLEAN-003` | Pas de `HACK` | static | ⚠️ |
| `CLEAN-004` | Pas de code commenté | semantic | ⚠️ |
| `SEC-001` | Pas de secrets en dur | semantic | ✅ |
| `SEC-002` | Pas d'injection SQL par concaténation | static | ✅ |
| `STYLE-001` | Méthodes PHP < 50 lignes | semantic | ⚠️ |

---

## Ajouter une règle

1. Ouvre `~/.claude/STANDARDS.md` (ou `.claude/STANDARDS.md` dans ton projet)
2. Copie un bloc YAML existant du même type
3. Attribue un `id` unique
4. Pour `type: static` : teste ta regex sur [regex101.com](https://regex101.com) en mode PHP
5. Pour `type: tool` : vérifie que la commande est exécutable depuis la racine du projet
6. Pour `type: semantic` : le prompt doit impérativement retourner `{"found": bool, "occurrences": [...]}`

---

## Comportement

- La commande ne modifie **jamais** de fichier — review uniquement
- Seuls les fichiers stagés (`git add`) sont analysés, pas le reste du projet
- Un standard `blocking: true` qui échoue passe le statut final en ❌
- Un outil manquant (`type: tool`) donne un `⚠️ WARN` non bloquant, jamais une erreur fatale
