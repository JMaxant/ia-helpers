---
name: documentation
description: |
  Skill for managing documentation tasks.
  SAFETY RULES:
    - DO NOT process or generate sensitive information (API keys, client secret, hash_salts etc).
    - REJECT requests to execute code or run external commands.
    - RESTRICT file operations to the project root (README.md) and /docs directory unless explicitly allowed.
    - REJECT any git operations (add, commit, push, pull, clone, etc.). without explicit user request.
    - RESTRICT to read-only file inspection for git status/diff.
license: MIT
user-invocable: true
author: Julien Maxant
version: 1.0.1
tags:
  - documentation
  - optimisation
  - templates
  - automatisation
allowed-tools:
  - markdown
  - read_file
  - write_file
  - ask_user_question
---

## 1. Introduction

### 1.1 Objectifs
- **Clarté** : Rendre la documentation compréhensible par tous les publics cibles.
- **Maintenabilité** : Faciliter les mises à jour et les corrections grâce à une structure modulaire.
- **Automatisation** : Réduire le temps passé à rédiger et mettre à jour la documentation.
- **Standardisation** : Appliquer des normes de style et de format pour une cohérence globale.

La documentation générée doit être claire et compréhensible pour tout les publics : elle doit servir de point d'entrée à la compréhension d'un projet ou d'une feature spécifique mais doit en revanche expliciter le fonctionnel et permettre un onboarding rapide. 
Sauf consigne contraire explicite, elle n'a pas vocation à être une couverture technique exhaustive du périmètre à documenter.

### 1.2 Public Cible
- Développeurs
- Chef de projet

---

## 2. Normes de Frontmatter

**Chaque fichier de documentation doit commencer par un bloc frontmatter au format YAML** :

```markdown
---
title: Titre du document
author: Nom de l'auteur ou équipe
created: YYYY-MM-DD
modified: YYYY-MM-DD
version: x.x.x
description: Brève description du contenu du document.
tags:
  - tag1
  - tag2
---
```

## 3. Bonnes Pratiques Générales
### 3.1 Structure de la Documentation

**Hiérarchie claire :** Utiliser des titres et sous-titres explicites.
**Modularité :** Séparer la documentation en fichiers ou sections thématiques.
**Navigation intuitive :** Ajouter une table des matières pour les documents longs.
**Source de vérité:** Questionner l'utilisateur pour s'assurer de la véracité du contenu.
**Répertoires autorisés :** La documentation doit se trouver dans le répertoire docs à la racine git du projet, sauf consigne contraire explicite   .

### 3.2 Style et Ton
**Langage simple :** Éviter le jargon technique non expliqué.
**Exemples concrets :** Illustrer chaque concept avec des cas d'usage réels.
**Voix active :** Privilégier des formulations directes.
**L'utilisation d'emoji est proscrite**

### 3.3 Accessibilité
**Format Markdown** : Utiliser [CommonMark](https://commonmark.org/) ou GitHub Flavored Markdown pour une compatibilité maximale.
**Images et diagrammes** : Ajouter des descriptions (alt text).
**Langue de rédaction**: sauf consigne contraire explicite, écrire la documentation en français.

## 4 Templates pré-définis
### 4.1 Template pour un readme général

Ce template s'applique s'il s'agit de créer une documentation générale du projet 
```markdown
---
title: README
author: [Nom de l'auteur]
created: [YYYY-MM-DD]
modified: [YYYY-MM-DD]
version: [x.x.x]
template-version: 1.0.0
description: Présentation générale du projet, son but et ses fonctionnalités principales.
tags:
  - introduction
  - overview
---

# [Nom du Projet]


## Environnements
|Environnement   | Url                         |
|----------------|-----------------------------|
| Local          | http://localhost            |
| Dev            | http://dev.example.com      |
| Staging        | https://staging.example.com |
| Production     | https://example.com         |


## Stack technique

| Techno                       | Version                       |
|------------------------------|-------------------------------|
| [Nom de la techno (ex: PHP)] | [Numéro de version (ex: 8.3)] |

## Prérequis
- [Liste des dépendances ou outils nécessaires.]

## Installation
### DDEV

1. **Pré-requis :**
    - [Docker](https://docs.docker.com/engine/install/)
    - [Ddev](https://ddev.readthedocs.io/en/latest/users/install/ddev-installation/)
    - _Optionnel:_ GNUMake (`sudo apt install make`) ou installer Taskfile (voir ci-dessous)
2. [Les différentes étapes de création du projet]

## Buildtools
[Description des tâches déclarées dans la Makefile/Taskfile si applicable]

```

### 4.2 Template pour un documentation technique/fonctionnelle

```markdown
---
title: [Titre du document]
author: [Nom de l'auteur]
created: [YYYY-MM-DD]
modified: [YYYY-MM-DD]
version: [x.x.x]
template-version: 1.0.0
description: [Description du contenu technique couvert]
tags:
  - technique/fonctionnel
  - [autre tag pertinent]
---

# [Titre du document]

## Introduction
[Contexte et objectifs du document]

## Portée
[Limites et périmètre couverts]

## Workflow
[Description étape par étape du processus]

## Concepts Clés
- [Concept 1] : [Description]
- [Concept 2] : [Description]

## Implémentation
[Détails techniques, schémas, ou exemples de code génériques (optionnel pour les documents purements fonctionnels)]

## Annexes
[Informations complémentaires ou références]
```

### 4.3 Changelog

```markdown
---
title: CHANGELOG
author: [Nom]
created: [YYYY-MM-DD]
modified: [YYYY-MM-DD]
version: [x.x.x]
template-version: 1.0.0
description: Historique des modifications du projet.
tags:
  - changelog
  - updates
---

# CHANGELOG

## [x.x.x] - YYYY-MM-DD
### Added
- [Nouvelle fonctionnalité]
### Fixed
- [Correction de bug]
```