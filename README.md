# ia-helpers

A collection of reusable helpers for working with AI agents: skills, agent definitions, rules, generic ignore files (`.aiignore`), and other assets meant to be shared across projects.

Assets are written in English.

## Contents

### `skills/`

Agent skills, one directory per skill. Each skill is a `SKILL.md` with YAML frontmatter (`name`, `description`, and optionally `version`, `license`, `allowed-tools`, `user-invocable`), plus optional `templates/` and `references/` subdirectories.

- `new-task/` — Create a task (GitHub Issue, GitLab Issue, or Redmine ticket) from predefined templates, with user validation before creation. Reads its configuration from `.claude/task-tracker.toml` at the repository root and offers to create it when missing.
  - `templates/` — Task templates: `feature.md`, `bug.md`, `design.md`, `chore.md`.
  - `references/trackers.md` — Tracker-specific reference (GitHub, GitLab, Redmine).
- `code-review/` — Code review focused on quality, security, and maintainability. Defines the universal review criteria (readability, design, security) and a structured report template written to `CODE_REVIEW.md`. Stack detection is dynamic: marker files (`composer.json`, `package.json`, ...) are used to infer the stack and version, confirmed interactively with the user (`AskUserQuestion`) before applying stack-specific conventions.
  - `references/` — Stack-specific conventions, one file per stack, loaded only when detected: `php-drupal.md`, `javascript.md` (includes Vue.js 3), `python.md`, `golang.md`. Add a new stack by dropping another file here (detection markers documented at the top of each file) without touching `SKILL.md`.
- `documentation/` — Writing and updating project documentation. Defines the frontmatter standard, general writing rules (structure, style, accessibility, no emoji), and templates for a general README, a technical/functional document, and a changelog. The writing language is not assumed: it's inferred from explicit user instructions or existing project conventions, and asked interactively (`AskUserQuestion`) otherwise.

### `agents/`

Agent definitions derived from the skills above, for runtimes that use dedicated subagents rather than invocable skills.

- `code-review.md` / `documentation.md` — Subagent definitions in Markdown with YAML frontmatter (`name`, `description`, `tools`), where the body is the agent's system prompt. Compatible with the Claude Code subagent format.
- `code-review.toml` / `documentation.toml` — TOML descriptors (`display_name`, `description`, `safety`, `skills`) that bind an agent to its skill, for runtimes reading agent registries such as `~/.vibe/agents/`.

The `.md` agents and the matching skills carry the same content: pick whichever format the target runtime supports, not both.

### Root files

- `.editorconfig` — Shared editor settings (UTF-8, LF, 2-space indent, trailing whitespace preserved in Markdown to keep hard breaks).
- `LICENSE` — MIT.

## Usage

These assets are meant to be copied or symlinked into the configuration directory of the target tool. For Claude Code:

```bash
# Skills — globally for all projects
ln -s "$PWD/skills/new-task" ~/.claude/skills/new-task

# Skills — for a single project
ln -s "$PWD/skills/code-review" /path/to/project/.claude/skills/code-review

# Agents
ln -s "$PWD/agents/documentation.md" ~/.claude/agents/documentation.md
```

Symlinking keeps the installed copies in sync with this repository; copy instead when a project needs to pin or adapt its own version.

### Containerized agents

When the agent runs inside a container or a sandbox (Docker, DDEV, a devcontainer, an agent sandbox, or anything similar), it only sees the paths exposed to that container. A symlink into the host home directory resolves to nothing, and a home-level installation is lost whenever the container is rebuilt on an ephemeral filesystem. Three options, from the most local to the most permanent:

- **Commit the assets to the project.** Copy the skills and agents into the project's `.claude/skills/` and `.claude/agents/`. Since the project directory is normally mounted into the container, they are available with no extra setup, and the whole team gets the same versions.
- **Mount this repository into the container.** Expose it as a volume, then symlink or copy from the mount point to the configuration directory the agent reads inside the container. The assets stay in sync with the repository, at the cost of one mount to declare per environment.
- **Bake them into the image.** Copy them in at build time when the environment is rebuilt often and the versions can be pinned.

Whichever option applies, check where the agent actually looks for its skills and agents inside the container, since the configuration directory may differ from the host one.

## Contributing

- Follow `.editorconfig`.
- One directory per skill, containing a `SKILL.md` with a `description` explicit enough for an agent to decide on its own when to trigger the skill.
- Keep supporting material in `templates/` (content to fill in) and `references/` (documentation loaded on demand) so the main `SKILL.md` stays short.
- When a skill and an agent cover the same subject, keep both in sync.

## License

See [LICENSE](LICENSE).
