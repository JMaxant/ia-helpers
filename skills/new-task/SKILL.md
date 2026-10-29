---
name: new-task
version: 1.2.0
description: Create a task (GitHub Issue, GitLab Issue, or Redmine ticket) from predefined templates (feature, bug, design, chore), with validation before creation. Use as soon as the user wants to create/draft an issue, a ticket, or a project-management task.
---

# new-task — templated task creation

Draft a task from a template, get it validated, then create it in the project's tracker. A single request can cover several tasks at once (e.g. "create a ticket for each of these 3 bugs"): in that case, draft the whole batch and get it validated as a block (see Step 4), rather than task by task.

## Step 0 — Initialize the configuration (if missing)

If `.claude/task-tracker.toml` doesn't exist at the repo root, or if the user explicitly asks for initialization, offer to create it by asking (AskUserQuestion):

1. **Tracker**: github | gitlab | redmine (pre-select the one inferred from `git remote get-url origin` if there is one).
2. **Defaults**: default labels (suggest the project's existing ones: `gh label list`), and the default language (suggest the one inferred from the last tasks, see "Language"). Leave `milestone` empty unless the project keeps one milestone open over a long period (see "Milestone"). The AI disclosure footer (see "Disclosure footer") is on by default and is not asked about; mention it in the summary of the file you show, and change it only if the user wants it off or reworded.
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
milestone = ""               # default milestone (optional, validated at use time, see "Milestone")
language = ""                # language of titles and bodies, e.g. "fr"; empty = inferred, see "Language"

[disclosure]
enabled = true               # append the AI disclosure footer to every task (default: true)
text = ""                    # custom wording; empty = the default wording, see "Disclosure footer"

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
- Follow the project's conventions if documented (CLAUDE.md, CONTRIBUTING): title prefixes, required labels, links to a milestone. They do not decide the language (see below).
- Write in the language resolved below, headings of the template included.
- Unless disabled in the config, end the body with the disclosure footer (next section).

## Language

Resolve the language of the title and body before drafting, in this order, and stop at the first step that gives an answer:

1. **Config**: `defaults.language`, when non-empty (a name or an ISO code).
2. **Existing tasks**: read the last five tasks of the tracker, open or closed (commands in `references/trackers.md`), and take the language most of their titles and bodies are written in. Leave out the disclosure footer when judging, since its default wording is English.
3. **Ask once** (AskUserQuestion), then offer to save the answer as `defaults.language`. This covers an empty tracker and a split one.

Do not infer the language from anything else: not the language of the user's request, not a rule such as "write the documentation in English" that is scoped to `docs/`, not the README or the commit messages. A task drafted in English in a tracker where every other task is in another language is the failure this section prevents.

Translate the template's headings into the resolved language. Label, milestone and tracker names stay exactly as the tracker defines them. The disclosure footer is not translated: set `[disclosure].text` for a localized wording.

## Milestone

`defaults.milestone` drifts: a milestone is closed or renamed on the tracker and the config keeps pointing at it, which makes the creation command fail. Therefore:

- **Off by default.** Step 0 leaves it empty unless the project keeps one milestone open over a long period. Per-release milestones belong to the task, not to the config.
- **Validated while drafting**, before Step 4, against the tracker's open milestones (commands in `references/trackers.md`). A milestone found is shown in the draft. A milestone not found is dropped from the draft with a visible warning in Step 4 (`Default milestone "X" not found on the tracker: not applied`), and the creation proceeds without it. Never let the creation command fail on it, and never drop it silently.
- **After a warning**, offer once to fix or clear `defaults.milestone` in the config.
- **A milestone the user names for one task** that does not exist is not dropped: ask whether to pick another one or create the task without it.

## Disclosure footer

The tracker attributes every API call to the owner of the token, so a task drafted by an agent appears under the user's name with nothing to tell a reader that an agent was involved. The footer states it.

- **Default**: enabled. `[disclosure].enabled = false` turns it off for the project.
- **Wording, defined here once**: `Drafted by an AI agent ({agent}) and published through the account owner's token, after their review.` `{agent}` is the name of the agent running the skill (for example `Claude Code`). A non-empty `[disclosure].text` replaces the default wording and may use `{agent}`.
- **Placement**: the last thing in the body, after a blank line and a `---` line. Title, labels and milestone are left untouched.
- **Validation**: the footer is part of the draft shown in Step 4, for a single task and in the batch file alike, so the user approves the exact text that will be published. If the user asks to drop it for one task, drop it for that task only and leave the config alone unless they ask to change it.
- **Out of scope**: git commit attribution (the `Co-Authored-By` trailer) and comments on existing tasks, which this skill does not create.

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
