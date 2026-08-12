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

### `output-styles/`

Claude Code output styles, one Markdown file per style. Each file is YAML frontmatter followed by a body appended to the end of the system prompt as `# Output Style: <name>`. Every frontmatter field is optional: `name` (defaults to the file name), `description` (shown in the `/config` picker), `keep-coding-instructions`, and `force-for-plugin` (plugins only, unused here).

- `terse.md` — Terse, evidence-first engineering voice: minimal prose, no claim without concrete evidence, assumptions surfaced rather than silently resolved, weak proposals argued rather than accepted. Sets `keep-coding-instructions: true`, which preserves Claude Code's built-in coding instructions (scope discipline, comment policy, security, UI testing) that an output style otherwise replaces; the style governs tone and rigor only, and defers code mechanics to the project's `CLAUDE.md`.

Styles are read from `~/.claude/output-styles/` for the user, and from every `.claude/output-styles/` between the working directory and the repository root for a project, the closest one winning on a name collision. Selecting a style with `/config` writes the `outputStyle` field to `.claude/settings.local.json`; because the system prompt is read once at session start, the change only applies after `/clear` or in a new session.

An output style applies to the main conversation only. Subagents such as the `agents/` definitions above run their own system prompt and do not inherit it.

### Root files

- `.editorconfig` — Shared editor settings (UTF-8, LF, 2-space indent, trailing whitespace preserved in Markdown to keep hard breaks).
- `.markdownlint-cli2.jsonc` — Markdown lint rules (see Development below).
- `lefthook.yml` — Git hooks config (pre-commit Markdown lint).
- `Taskfile.dist.yml` — Template for local dev commands (see Development below).
- `package.json` — npm scripts and dev dependencies backing the Taskfile and CI.
- `.github/workflows/markdown-lint.yml` — CI job running the Markdown lint on push to `main` and on pull requests.
- `LICENSE` — MIT.

## Usage

These assets are meant to be copied or symlinked into the configuration directory of the target tool. For Claude Code:

```bash
# Create the target directories first: ln -s fails on a missing parent, and
# Claude Code only creates these once the matching feature has been used.
mkdir -p ~/.claude/skills ~/.claude/agents ~/.claude/output-styles

# Skills — globally for all projects
ln -s "$PWD/skills/new-task" ~/.claude/skills/new-task

# Skills — for a single project
mkdir -p /path/to/project/.claude/skills
ln -s "$PWD/skills/code-review" /path/to/project/.claude/skills/code-review

# Agents
ln -s "$PWD/agents/documentation.md" ~/.claude/agents/documentation.md

# Output styles — Claude Code only, globally or per project
ln -s "$PWD/output-styles/terse.md" ~/.claude/output-styles/terse.md
ln -s "$PWD/output-styles/terse.md" /path/to/project/.claude/output-styles/terse.md
```

Select the installed output style with `/config`; it takes effect on the next session.

Symlinking keeps the installed copies in sync with this repository; copy instead when a project needs to pin or adapt its own version.

### Containerized agents

When the agent runs inside a container or a sandbox (Docker, DDEV, a devcontainer, an agent sandbox, or anything similar), it only sees the paths exposed to that container. A symlink into the host home directory resolves to nothing, and a home-level installation is lost whenever the container is rebuilt on an ephemeral filesystem. Three options, from the most local to the most permanent:

- **Commit the assets to the project.** Copy the skills, agents, and output styles into the project's `.claude/skills/`, `.claude/agents/`, and `.claude/output-styles/`. Since the project directory is normally mounted into the container, they are available with no extra setup, and the whole team gets the same versions. Check that the project's `.gitignore` does not exclude `.claude/` wholesale, which would silently drop them.
- **Mount this repository into the container.** Expose it as a volume, then symlink or copy from the mount point to the configuration directory the agent reads inside the container. The assets stay in sync with the repository, at the cost of one mount to declare per environment.
- **Bake them into the image.** Copy them in at build time when the environment is rebuilt often and the versions can be pinned.

Whichever option applies, check where the agent actually looks for its skills and agents inside the container, since the configuration directory may differ from the host one.

## Development

Requires Node.js and, optionally, [Task](https://taskfile.dev) (`task`) to run the commands below without typing the underlying `npm`/`npx` calls.

Setup (once per clone): copy the Taskfile template, then run the setup task.

```bash
cp Taskfile.dist.yml Taskfile.yml   # git-ignored, safe to customize locally
task setup                          # npm install + registers the pre-commit git hook
```

Without `task`, the same setup is `npm install` (its `prepare` script registers the git hook).

Available tasks (`task --list`, or run the underlying npm script directly):

| Task | npm script equivalent | What it does |
| --- | --- | --- |
| `task qa` | `npm run lint:md` | Runs every quality check on all files — what CI runs. |
| `task qa:fix` | `npm run lint:md:fix` | Auto-fixes what can be (currently Markdown lint issues). |

The pre-commit hook (via `lefthook.yml`) runs the same Markdown lint on staged `*.md` files only, so most issues are caught before they ever reach CI.

## Contributing

- Follow `.editorconfig`.
- One directory per skill, containing a `SKILL.md` with a `description` explicit enough for an agent to decide on its own when to trigger the skill.
- Keep supporting material in `templates/` (content to fill in) and `references/` (documentation loaded on demand) so the main `SKILL.md` stays short.
- When a skill and an agent cover the same subject, keep both in sync.
- One Markdown file per output style in `output-styles/`, named in kebab-case, with a `description` written for the `/config` picker. Set `keep-coding-instructions: true` unless the style deliberately drops Claude Code's software engineering instructions. Keep it to tone, rigor, and response format: project conventions belong in `CLAUDE.md`, and reusable workflows in a skill.
- Run `task qa` (or `npm run lint:md`) before committing; the pre-commit hook already checks staged Markdown files, `task qa` covers everything.

## License

See [LICENSE](LICENSE).
