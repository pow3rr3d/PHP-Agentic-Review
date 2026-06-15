---
name: review
command: /review-php
description: Analyse le code stagé (git add) et vérifie la conformité aux standards du projet. Charge les standards locaux (.claude/STANDARDS.md) en priorité, sinon les standards globaux (~/.claude/STANDARDS.md).
allowed-tools: Bash(git diff --staged *), Bash(git status *), Bash(phpstan *), Bash(grep *), Bash(find *), Bash(cat *), Bash(test *)
---

# /review — Code Review avant commit

## Objectif
Analyser **uniquement les fichiers stagés** (`git add`) et vérifier leur conformité aux standards de développement.

## Étape 1 — Récupérer les fichiers stagés

```bash
git diff --staged --name-only
git diff --staged
```

Si aucun fichier n'est stagé, affiche :
> ⚠️ Aucun fichier stagé. Lance `git add <fichiers>` avant d'utiliser /review.

Et stoppe l'exécution.

## Étape 2 — Charger les standards (fallback local → global)

Cherche les standards dans cet ordre de priorité :

```bash
# 1. Standards locaux au projet (priorité haute)
test -f .claude/STANDARDS.md && cat .claude/STANDARDS.md

# 2. Sinon, standards globaux
test -f ~/.claude/STANDARDS.md && cat ~/.claude/STANDARDS.md
```

Règles de chargement :
- Si `.claude/STANDARDS.md` existe dans le projet courant → utilise **uniquement** ce fichier
- Sinon → utilise `~/.claude/STANDARDS.md`
- Si aucun fichier n'existe → affiche une erreur et stoppe

En tête du rapport, indique toujours la source chargée :
- `📁 Standards : locaux (.claude/STANDARDS.md)`
- `🌐 Standards : globaux (~/.claude/STANDARDS.md)`

### Standards hybrides (local + global)

Si le fichier local contient la directive suivante en en-tête :

```yaml
extends: global
```

Alors charge **les deux fichiers** : les règles globales d'abord, les règles locales ensuite. En cas de conflit d'`id`, la règle locale écrase la règle globale. Indique dans le rapport :
- `🔀 Standards : globaux + override local (.claude/STANDARDS.md)`

## Étape 3 — Analyser chaque standard

Pour chaque règle définie dans le fichier de standards chargé, applique la vérification sur le diff stagé.

Les vérifications sont de 3 types :

**`type: static`** — Grep/regex sur les lignes ajoutées du diff (lignes commençant par `+`, hors `+++`)
- Si `match_is_error: true` → trouver le pattern = erreur
- Si `match_is_error: false` → ne pas trouver le pattern = erreur
- Si `file_filter` est défini → n'analyser que les fichiers dont le nom matche ce pattern

**`type: tool`** — Exécution d'une commande shell depuis la racine du projet
- `command_success: exit_0` → succès si code de retour = 0
- Si la commande n'existe pas ou échoue à s'exécuter → `⚠️ WARN` (non bloquant) avec le message d'erreur

**`type: semantic`** — Analyse LLM du diff
- Applique le `prompt` défini dans la règle sur le contenu du diff
- Le prompt doit retourner un JSON : `{"found": true/false, "occurrences": ["description..."]}`
- `found: true` = la règle a détecté un problème

## Étape 4 — Produire le rapport

```
╔══════════════════════════════════════════╗
║           🔍 CODE REVIEW REPORT          ║
╚══════════════════════════════════════════╝

🌐 Standards : globaux (~/.claude/STANDARDS.md)   ← ou locaux, ou hybrides
📁 Fichiers analysés : X fichier(s)
   • src/Service/MyService.php
   • ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ PASS  [PHP-001]  Les booléens sont en majuscules (TRUE / FALSE)
❌ FAIL  [PHP-002]  NULL est en majuscules
         └─ Détail : ligne 42 — `if ($value === null)`
⚠️ WARN  [STYLE-001]  Méthodes < 50 lignes (non bloquant)
         └─ Suggestion : MyService::process() — 67 lignes

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 RÉSULTAT FINAL
   ✅ Réussis  : X
   ❌ Échoués  : X  ← BLOQUANT
   ⚠️  Warnings : X

→ Statut : ✅ PRÊT À COMMIT  |  ❌ À CORRIGER AVANT COMMIT
```

## Règles de comportement

- Ne **jamais** modifier de fichier automatiquement — review uniquement
- Les standards marqués `blocking: true` font passer le statut final en ❌
- Toujours afficher le numéro de ligne et l'extrait de code pour les erreurs `static`
- Si un outil (`type: tool`) n'est pas installé → `⚠️ WARN` non bloquant, jamais une erreur fatale
