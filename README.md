# ia-helpers

A collection of reusable helpers for working with AI agents: skills, agent definitions, rules, generic ignore files (`.aiignore`), and other assets meant to be shared across projects.

## Contents

- `skills/` — Agent skills, one directory per skill.
  - `new-task/` — Create a task (GitHub Issue, GitLab Issue, or Redmine ticket) from predefined templates, with user validation before creation.
    - `templates/` — Task templates: `feature.md`, `bug.md`, `design.md`, `chore.md`.
    - `references/trackers.md` — Tracker-specific reference (GitHub, GitLab, Redmine).

## License

See [LICENSE](LICENSE).
