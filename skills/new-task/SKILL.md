---
name: new-task
description: Create a task (GitHub Issue, GitLab Issue, or Redmine ticket) from predefined templates (feature, bug, design, chore), with validation before creation. Use as soon as the user wants to create/draft an issue, a ticket, or a project-management task.
---

# new-task — templated task creation

Draft a task from a template, get it validated, then create it in the project's tracker. A single request can cover several tasks at once (e.g. "create a ticket for each of these 3 bugs"): in that case, draft the whole batch and get it validated as a block (see Step 4), rather than task by task.

## Step 0 — Initialize the configuration (if missing)

If `.claude/task-tracker.toml` doesn't exist at the repo root, or if the user explicitly asks for initialization, offer to create it by asking (AskUserQuestion):

1. **Tracker**: github | gitlab | redmine (pre-select the one inferred from `git remote get-url origin` if there is one).
2. **Defaults**: default labels, default milestone (suggest the project's existing values: `gh label list`, open milestones...).
3. **If Redmine**: instance URL, `project_id`, creation method (csv | mcp | rest — see `references/trackers.md`), and the `tracker_ids` mapping (ask for the IDs or fetch them via the API if a key is already available).

Write the file, show it to the user, then continue with the requested task. If the user declines, continue without config (inference + one-off questions) without re-offering it during the session.

## Step 1 — Identify the tracker

1. Read `.claude/task-tracker.toml` at the repo root if it exists (format below).
2. Otherwise, infer from `git remote get-url origin`: `github.com` → GitHub, `gitlab` → GitLab.
3. If ambiguous or outside a git repo, ask the user (AskUserQuestion).

Format of `.claude/task-tracker.toml`:

```toml
tracker = "github"          # github | gitlab | redmine

[defaults]
labels = []                  # default labels/tags applied
milestone = ""               # default milestone (optional)

[redmine]
url = "https://redmine.example.com"
project_id = "my-project"
method = "csv"               # csv (default, manual import) | mcp | rest
tracker_ids = { feature = 2, bug = 1, design = 2, chore = 2 }
# API key (rest method): environment variable REDMINE_API_KEY, never in this file.
```

## Step 2 — Choose the template

Available types (files in `templates/`):

- `feature.md` — evolution, new feature
- `bug.md` — defect
- `design.md` — design task (ADR, study, mockup)
- `chore.md` — maintenance, dependencies, docs, CI

Infer the type from the request; ask only if genuinely ambiguous.

## Step 3 — Draft

Fill in the template with the information provided and the repo's context. Drafting constraints:

- Title: imperative mood, ≤ 70 characters, no trailing punctuation. For a bug, describe the symptom, not the fix.
- Never leave a `<...>` placeholder in the result: ask for the missing information or remove the section if it's marked optional.
- Acceptance criteria: verifiable, one per line, checkboxes.
- No emoji. Be concise.
- Follow the project's conventions if documented (CLAUDE.md, CONTRIBUTING): title prefixes, required labels, links to a milestone.

## Step 4 — Validate

Always show the full draft (title, body, labels, milestone, assignee) to the user and wait for their explicit approval before any creation.

**Single task**: display the full draft as plain response text (never truncated, unlike the `preview` field), then ask the create/fix decision via AskUserQuestion with a short summary in `preview` (title + labels/milestone, not the whole body) and a "Fix first" option. Apply the corrections and re-present if the changes are substantial.

**Multiple tasks (batch)**: do not re-display each draft in full in the conversation. Draft the whole batch into a single file (format below), present a compact summary table in the conversation (subject, tracker, labels, milestone, parent if applicable) pointing to that file for the full body, then validate the batch as a block via a single AskUserQuestion ("Create the batch" / "Fix first"). Expand a specific draft in full only if the user asks for it.

**Batch draft file**: write the batch to a markdown file at the root of the current directory (not a scratchpad directory — a non-technical project manager wouldn't know where to look), named `tasks-YYYY-MM-DD-HH-mm.md` (generation date and time, 24h, hyphens as separators — the `H:i` format becomes `HH-mm` because `:` is invalid in a filename on Windows). Tell the user its path explicitly. This same file is then used as the source for creation (Step 5), so each draft only needs to be written once.

## Step 5 — Create

Detailed commands per tracker: see `references/trackers.md`. Summary:

- **GitHub**: `gh issue create` (prerequisite: `gh` authenticated). For a batch, use `--body-file` pointing to the body already drafted in the Step 4 file (extracted per task), rather than passing the body inline again or regenerating it.
- **GitLab**: `glab issue create` if installed, otherwise the REST API with `GITLAB_TOKEN`. Same principle of reusing the file for the body in case of a batch.
- **Redmine**: depending on the config's `method` — by default, generating a CSV for manual bulk import (the simplest, no authentication required), which is then the same file as the one from Step 4 (make its format `.csv` instead of `.md` from Step 4 onward if Redmine is the target); otherwise the Redmine MCP server if available in the session, or the REST API with `REDMINE_API_KEY`.

**Redmine specifics — confirm attachments before creation.** Never guess: before generating the CSV or the payload, confirm interactively (AskUserQuestion):

- the **target project** (the config's `project_id` may not be the right one: sub-projects are common);
- the possible **parent task** (ID of an existing task, or a task from the same batch in case of bulk creation);
- the **optional fields** relevant to this instance: target version, assignee, estimate, category, custom fields. Only offer the ones the user or the config mentions; if new ones are cited, offer to add them to `.claude/task-tracker.toml` for next time.

For bulk creation, the Step 4 summary table should also include the target project and the parent task so it can be validated at a glance.

After creation, report the URL (or the Redmine ID) to the user. On failure (auth, permissions), report the exact error and provide the drafted body for manual creation — never lose the drafting work.
