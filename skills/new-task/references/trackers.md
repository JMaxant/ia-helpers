# Creation commands per tracker

## GitHub (`gh`)

```bash
gh issue create \
  --title "TITLE" \
  --body-file body.md \
  --label "label1,label2" \
  --milestone "MILESTONE_NAME" \
  --assignee "@me"
```

- Write the body to a scratchpad file rather than as an argument (escaping).
- Target repo: the current remote's; otherwise `--repo owner/name`.
- Check auth if needed: `gh auth status`.

## GitLab (`glab` or REST API)

With `glab` installed:

```bash
glab issue create --title "TITLE" --description-file body.md --label "label1,label2"
```

Without `glab`, REST API (token in `GITLAB_TOKEN`, never in plain text in the displayed command):

```bash
curl --fail-with-body -X POST \
  -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "https://gitlab.example.com/api/v4/projects/<ID_or_urlencoded_path>/issues" \
  --data-urlencode "title=TITLE" \
  --data-urlencode "description@body.md" \
  --data-urlencode "labels=label1,label2"
```

The project ID can be the URL-encoded path: `group%2Fproject`.

## Redmine

Three methods, in default preference order (overridable via `method` in the config):

### Method 1 (the simplest): CSV for manual bulk import

No authentication required; suited to creating several tasks at once. Generate a CSV file that the user will import themselves into Redmine (Project > Tasks > ... > Import, available from Redmine 4.1 onward).

```csv
subject;tracker;priority;description;estimated_hours;parent
"TITLE";"Feature";"Normal";"Task body";;
```

- `;` separator and UTF-8 encoding are recommended (confirm per instance; column mapping happens in the import wizard, so headers are free-form).
- `parent` column: ID of an existing task, or the exact `subject` of another row in the same CSV (Redmine resolves the hierarchy at import time if the option is checked in the wizard). The target project is chosen in the import wizard: remind the user of this when announcing the file.
- Add one column per optional field confirmed with the user (target version, assignee, estimate, custom fields) — see "Redmine specifics" in SKILL.md.
- Tracker/priority names must exactly match the instance's labels.
- Escape quotes in the body by doubling them (`""`); line breaks are allowed inside a quoted field.
- Write the file at the repo root or the requested location, then announce it to the user along with the import path in Redmine.

### Method 2: Redmine MCP server

If available in the session (look for `redmine` tools via ToolSearch).

### Method 3: REST API

Key in `REDMINE_API_KEY`, URL and `project_id` from `.claude/task-tracker.toml`:

```bash
curl --fail-with-body -X POST \
  -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  -H "Content-Type: application/json" \
  "$REDMINE_URL/issues.json" \
  -d @payload.json
```

`payload.json`:

```json
{
  "issue": {
    "project_id": "my-project",
    "tracker_id": 2,
    "subject": "TITLE",
    "description": "body in Textile or Markdown depending on the instance's config",
    "priority_id": 2,
    "parent_issue_id": 123,
    "fixed_version_id": 4,
    "assigned_to_id": 12,
    "custom_fields": [{ "id": 7, "value": "..." }]
  }
}
```

`parent_issue_id`, `fixed_version_id`, `assigned_to_id`, and `custom_fields` are optional — only include them once confirmed with the user. IDs (versions, users, custom fields) are fetched via the API: `GET /projects/<id>/versions.json`, `GET /users.json`, `GET /custom_fields.json`.

- `tracker_id`: map from the config's `tracker_ids` (feature/bug/design/chore).
- Careful: Redmine often uses Textile, not Markdown — check the instance's config; adapt the body's syntax accordingly (checkboxes aren't supported in Textile: use lists instead).
- The JSON response contains `issue.id`: report `$REDMINE_URL/issues/<id>`.
