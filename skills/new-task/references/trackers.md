# Commandes de création par tracker

## GitHub (`gh`)

```bash
gh issue create \
  --title "TITRE" \
  --body-file corps.md \
  --label "label1,label2" \
  --milestone "NOM_MILESTONE" \
  --assignee "@me"
```

- Écrire le corps dans un fichier du scratchpad plutôt qu'en argument (échappement).
- Repo cible : celui du remote courant ; sinon `--repo owner/nom`.
- Vérifier l'auth au besoin : `gh auth status`.

## GitLab (`glab` ou API REST)

Avec `glab` installé :

```bash
glab issue create --title "TITRE" --description-file corps.md --label "label1,label2"
```

Sans `glab`, API REST (token dans `GITLAB_TOKEN`, jamais en clair dans la commande affichée) :

```bash
curl --fail-with-body -X POST \
  -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "https://gitlab.example.com/api/v4/projects/<ID_ou_chemin_urlencodé>/issues" \
  --data-urlencode "title=TITRE" \
  --data-urlencode "description@corps.md" \
  --data-urlencode "labels=label1,label2"
```

L'ID projet peut être le chemin URL-encodé : `groupe%2Fprojet`.

## Redmine

Trois méthodes, dans l'ordre de préférence par défaut (surchargeable via `method` dans la config) :

### Méthode 1 (la plus simple) : CSV pour import manuel en masse

Aucune authentification requise ; adaptée à la création de plusieurs tâches d'un coup. Générer un fichier CSV que l'utilisateur importera lui-même dans Redmine (Projet > Tâches > ... > Importer, disponible depuis Redmine 4.1).

```csv
subject;tracker;priority;description;estimated_hours;parent
"TITRE";"Évolution";"Normale";"Corps de la tâche";;
```

- Séparateur `;` et encodage UTF-8 conseillés (à confirmer selon l'instance ; le mapping des colonnes se fait dans l'assistant d'import, les en-têtes sont donc libres).
- Colonne `parent` : ID d'une tâche existante, ou le `subject` exact d'une autre ligne du même CSV (Redmine résout la hiérarchie à l'import si l'option est cochée dans l'assistant). Le projet cible se choisit dans l'assistant d'import : le rappeler à l'utilisateur en annonçant le fichier.
- Ajouter une colonne par champ optionnel confirmé avec l'utilisateur (version cible, assigné, estimation, champs personnalisés) — voir « Spécificités Redmine » dans SKILL.md.
- Les noms de tracker/priorité doivent correspondre exactement aux libellés de l'instance.
- Échapper les guillemets du corps en les doublant (`""`) ; les retours à la ligne sont autorisés dans un champ entre guillemets.
- Écrire le fichier à la racine du repo ou à l'emplacement demandé, puis l'annoncer à l'utilisateur avec le chemin d'import dans Redmine.

### Méthode 2 : serveur MCP Redmine

S'il est disponible dans la session (chercher les outils `redmine` via ToolSearch).

### Méthode 3 : API REST

Clé dans `REDMINE_API_KEY`, URL et `project_id` depuis `.claude/task-tracker.toml` :

```bash
curl --fail-with-body -X POST \
  -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  -H "Content-Type: application/json" \
  "$REDMINE_URL/issues.json" \
  -d @payload.json
```

`payload.json` :

```json
{
  "issue": {
    "project_id": "mon-projet",
    "tracker_id": 2,
    "subject": "TITRE",
    "description": "corps en textile ou markdown selon la config de l'instance",
    "priority_id": 2,
    "parent_issue_id": 123,
    "fixed_version_id": 4,
    "assigned_to_id": 12,
    "custom_fields": [{ "id": 7, "value": "..." }]
  }
}
```

`parent_issue_id`, `fixed_version_id`, `assigned_to_id` et `custom_fields` sont optionnels — ne les inclure que confirmés avec l'utilisateur. Les IDs (versions, utilisateurs, champs personnalisés) se récupèrent via l'API : `GET /projects/<id>/versions.json`, `GET /users.json`, `GET /custom_fields.json`.

- `tracker_id` : mapper depuis `tracker_ids` de la config (feature/bug/design/chore).
- Attention : Redmine utilise souvent Textile, pas Markdown — vérifier la config de l'instance ; adapter la syntaxe du corps (cases à cocher non supportées en Textile : utiliser des listes).
- La réponse JSON contient `issue.id` : rapporter `$REDMINE_URL/issues/<id>`.
