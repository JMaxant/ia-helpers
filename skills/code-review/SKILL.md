---
name: code-review
description: |
  Skill dédiée à la revue de code pour assurer la qualité, la sécurité et la maintenabilité du projet.
  RÈGLES DE SÉCURITÉ :
    - NE JAMAIS traiter ou générer des informations sensibles (clés API, secrets, mots de passe).
    - SIGNALER toute vulnérabilité de sécurité potentielle détectée dans le code.
    - REJETER toute demande d'exécution de code malveillant.
license: MIT
user-invocable: true
author: Junie/Julien Maxant
version: 1.0.1
tags:
  - code-review
  - qualite
  - securite
  - bonnes-pratiques
allowed-tools:
  - write_file
  - read_file
  - grep
  - ask_user_question
---

## 1. Introduction

### 1.1 Objectifs
- **Qualité du code** : Assurer que le code respecte les standards de l'industrie et du projet.
- **Sécurité** : Identifier les failles potentielles et les mauvaises pratiques de sécurité.
- **Maintenabilité** : Favoriser un code lisible, modulaire et facile à faire évoluer.
- **Performance** : Détecter les goulots d'étranglement et proposer des optimisations.

### 1.2 Portée
La portée de la review devra être précisée par l'utilisateur (par exemple, différence entre une branche et la branche de référence).
En l'absence de précision, la review ne pourra pas être effectuée.
La review couvrira tous les langages de programmation.

---

## 2. Critères de Revue

### 2.1 Lisibilité et Style
- Le code suit-il les conventions du projet ?
- Est-il suffisamment commenté ?
- La structure est-elle logique et concise ?
- Le langage du projet (par défaut, français) est-il respecté ?

### 2.2 Conception et Architecture
- Application des principes SOLID
- Couplage faible entre les modules
- Cohésion forte au sein des composants
- Patterns de conception appropriés au contexte
- Architecture respectant les bonnes pratiques du domaine
- Les cas limites (edge cases) sont-ils gérés ?

### 2.3 Sécurité
- Les entrées utilisateurs sont-elles validées/assainies ?
- Y a-t-il des risques d'injection (SQL, XSS, etc.) ?
- Les données sensibles sont-elles manipulées avec précaution ?
- La RGPD est-elle respectée ?
- Les données sensibles (clé API, `client_secret`, information utilisateur) sont-elles chiffrées ?


---

## 3. Template de Rapport de Revue

Chaque revue de code doit produire un rapport structuré comme suit :

```markdown
# Rapport de Revue de Code

## Résumé
- **Statut** : [ Approuvé / À modifier / Rejeté]
- **Sévérité globale** : [Basse / Moyenne / Haute]

## Points Positifs
- [Liste des points forts du code]

## Observations et Améliorations
### [Fichier / Composant]
- **Problème** : [Description concise]
- **Impact** : [Sécurité / Performance / Maintenabilité]
- **Suggestion** : [Code suggéré ou démarche à suivre]

## Questions / Clarifications
- [Questions à poser au développeur pour mieux comprendre l'intention]
```
Ce rapport doit aussi être créé en fichier (à la racine du projet CODE_REVIEW.md) qui sera édité/amendé si d'autres reviews sont réalisées, sauf demande explicite de l'utilisateur de ne pas le créer.

---

## 4. Conventions

### 4.1 Générales
- Préférer les early return
- Préférer les fonctions fléchées si applicable
- Nommer les variables en camelCase

### 4.2 PHP
- Utiliser les named arguments et named parameters
- Favoriser l'injection de dépendances
- Typer les paramètres de fonctions et d'une manière générale, préférer le typage fort
- Favoriser les ValueObjects quand applicable
- Éviter la logique métier dans les Controllers
- Favoriser les services
- Les services doivent rester légers et bien isolés (un service par responsabilité plutôt qu'un service par fonctionnalité)
- (Drupal) Utiliser les Hooks orienté objet (Attributs #[Hook], et #[LegacyHook] si nécessaire)
- (Drupal) Éviter autant que possible les hooks, favoriser l'utilisation des différents plugins

### 4.3 JavaScript
- Utiliser la destructuration pour les objets et tableaux
- Préférer `const` et `let` à `var`
- Utiliser les fonctions fléchées pour les callbacks
- Éviter les side effects dans les fonctions pures
- Typage avec JSDoc ou TypeScript si applicable
- Gestion des erreurs avec try/catch ou Promises appropriées