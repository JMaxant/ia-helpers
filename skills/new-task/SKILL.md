---
name: new-task
description: Créer une tâche (GitHub Issue, GitLab Issue ou ticket Redmine) à partir de templates prédéfinis (feature, bug, conception, chore), avec validation avant création. Utiliser dès que l'utilisateur veut créer/rédiger une issue, un ticket ou une tâche de gestion de projet.
---

# new-task — création de tâches templatisées

Rédige une tâche selon un template, la fait valider, puis la crée dans le tracker du projet.

## Étape 0 — Initialiser la configuration (si absente)

Si `.claude/task-tracker.toml` n'existe pas à la racine du repo, ou si l'utilisateur demande explicitement l'initialisation, proposer de le créer en le questionnant (AskUserQuestion) :

1. **Tracker** : github | gitlab | redmine (pré-sélectionner celui inféré depuis `git remote get-url origin` s'il y en a un).
2. **Défauts** : labels par défaut, milestone par défaut (proposer les valeurs existantes du projet : `gh label list`, milestones ouvertes...).
3. **Si Redmine** : URL de l'instance, `project_id`, méthode de création (csv | mcp | rest — voir `references/trackers.md`), et le mapping `tracker_ids` (demander les IDs ou les récupérer via l'API si une clé est déjà disponible).

Écrire le fichier, le montrer à l'utilisateur, puis continuer avec la tâche demandée. Si l'utilisateur décline, continuer sans config (inférence + questions ponctuelles) sans re-proposer dans la session.

## Étape 1 — Identifier le tracker

1. Lire `.claude/task-tracker.toml` à la racine du repo s'il existe (format ci-dessous).
2. Sinon, inférer depuis `git remote get-url origin` : `github.com` → GitHub, `gitlab` → GitLab.
3. Si ambigu ou hors repo git, demander à l'utilisateur (AskUserQuestion).

Format de `.claude/task-tracker.toml` :

```toml
tracker = "github"          # github | gitlab | redmine

[defaults]
labels = []                  # labels/étiquettes appliqués par défaut
milestone = ""               # milestone par défaut (optionnel)

[redmine]
url = "https://redmine.example.com"
project_id = "mon-projet"
method = "csv"               # csv (défaut, import manuel) | mcp | rest
tracker_ids = { feature = 2, bug = 1, design = 2, chore = 2 }
# Clé API (méthode rest) : variable d'environnement REDMINE_API_KEY, jamais dans ce fichier.
```

## Étape 2 — Choisir le template

Types disponibles (fichiers dans `templates/`) :

- `feature.md` — évolution, nouvelle fonctionnalité
- `bug.md` — anomalie
- `design.md` — tâche de conception (ADR, étude, maquette)
- `chore.md` — maintenance, dépendances, doc, CI

Déduire le type depuis la demande ; demander seulement si vraiment ambigu.

## Étape 3 — Rédiger

Remplir le template avec les informations fournies et le contexte du repo. Contraintes de rédaction :

- Titre : impératif, ≤ 70 caractères, sans ponctuation finale. Pour un bug, décrire le symptôme, pas la solution.
- Ne jamais laisser un placeholder `<...>` dans le résultat : demander l'information manquante ou supprimer la section si elle est marquée optionnelle.
- Critères d'acceptation : vérifiables, un par ligne, cases à cocher.
- Pas d'emoji. Concision.
- Respecter les conventions du projet si documentées (CLAUDE.md, CONTRIBUTING) : préfixes de titre, labels obligatoires, liens vers milestone.

## Étape 4 — Valider

Toujours montrer le brouillon complet (titre, corps, labels, milestone, assigné) à l'utilisateur et attendre son accord explicite avant toute création. Poser la validation via AskUserQuestion en plaçant le brouillon complet dans le champ `preview` de l'option de création — du texte affiché avant l'appel d'outil peut ne pas être visible de l'utilisateur. Prévoir une option « Corriger d'abord ». Appliquer ses corrections et re-présenter si les changements sont substantiels.

## Étape 5 — Créer

Commandes détaillées par tracker : voir `references/trackers.md`. Résumé :

- **GitHub** : `gh issue create` (prérequis : `gh` authentifié).
- **GitLab** : `glab issue create` si installé, sinon API REST avec `GITLAB_TOKEN`.
- **Redmine** : selon `method` de la config — par défaut génération d'un CSV pour import manuel en masse (le plus simple, aucune authentification requise) ; sinon serveur MCP Redmine si disponible dans la session, ou API REST avec `REDMINE_API_KEY`.

**Spécificités Redmine — confirmer les rattachements avant création.** Ne jamais deviner : avant de générer le CSV ou le payload, faire confirmer interactivement (AskUserQuestion) :

- le **projet cible** (le `project_id` de la config peut ne pas être le bon : sous-projets fréquents) ;
- la **tâche parente** éventuelle (ID d'une tâche existante, ou une tâche du même lot en cas de création en masse) ;
- les **champs optionnels** pertinents pour cette instance : version cible, assigné, estimation, catégorie, champs personnalisés. Ne proposer que ceux que l'utilisateur ou la config mentionnent ; s'il en cite de nouveaux, proposer de les ajouter à `.claude/task-tracker.toml` pour les fois suivantes.

Pour une création en masse, présenter un tableau récapitulatif (sujet, tracker, parent, projet) et le faire valider en bloc plutôt que tâche par tâche.

Après création, rapporter l'URL (ou l'ID Redmine) à l'utilisateur. En cas d'échec (auth, droits), rapporter l'erreur exacte et fournir le corps rédigé pour création manuelle — ne pas perdre la rédaction.
