# ia-helpers

A collection of reusable helpers for working with AI agents: skills, agent definitions, rules, generic ignore files (`.aiignore`), and other assets meant to be shared across projects.

Assets are written in English.

## Contents

### `skills/`

Agent skills, one directory per skill. Each skill is a `SKILL.md` with YAML frontmatter (`name`, `description`, and optionally `version`, `license`, `allowed-tools`, `user-invocable`), plus optional `templates/` and `references/` subdirectories.

- `new-task/` — Create a task (GitHub Issue, GitLab Issue, or Redmine ticket) from predefined templates, with user validation before creation. Reads its configuration from `.claude/task-tracker.toml` at the repository root and offers to create it when missing.
  - `templates/` — Task templates: `feature.md`, `bug.md`, `design.md`, `chore.md`.
  - The language of a task comes from the config, then from the last tasks of the tracker, then from a question; never from the language of the request. A configured default milestone is checked against the tracker and dropped with a visible warning if it no longer exists.
  - Every task ends with a footer stating it was drafted by an AI agent and published through the account owner's token. On by default, configurable with `[disclosure]` in `.claude/task-tracker.toml`.
  - `references/trackers.md` — Tracker-specific reference (GitHub, GitLab, Redmine).
- `code-review/` — Code review focused on quality, security, and maintainability. Defines the universal review criteria (readability, design, security) and a structured report template written to `CODE_REVIEW.md`. Stack detection is dynamic: marker files (`composer.json`, `package.json`, ...) are used to infer the stack and version, confirmed interactively with the user (`AskUserQuestion`) before applying stack-specific conventions.
  - `references/` — Stack-specific conventions, one file per stack, loaded only when detected: `php.md`, `drupal.md` (loaded on top of `php.md`), `javascript.md` (includes Vue.js 3), `python.md`, `golang.md`. Add a new stack by dropping another file here (detection markers documented at the top of each file) without touching `SKILL.md`.
- `documentation/` — Writing and updating project documentation. Defines the frontmatter standard, general writing rules (structure, style, accessibility, no emoji, a mandatory glossary of scope-specific terms and acronyms), and templates for a general README, a technical/functional document, and a changelog. The writing language is not assumed: it's inferred from explicit user instructions or existing project conventions, and asked interactively (`AskUserQuestion`) otherwise.

### `agents/`

Agent definitions derived from the skills above, for runtimes that use dedicated subagents rather than invocable skills.

- `code-review.md` / `documentation.md` — Subagent definitions (not part of the Claude Code plugin) in Markdown with YAML frontmatter (`name`, `description`, `tools`), where the body is the agent's system prompt. Compatible with the Claude Code subagent format.

The agents and the matching skills carry the same content: use one or the other, not both.

### `output-styles/`

Claude Code output styles, one Markdown file per style. Each file is YAML frontmatter followed by a body appended to the end of the system prompt as `# Output Style: <name>`. Every frontmatter field is optional: `name` (defaults to the file name), `description` (shown in the `/config` picker), `keep-coding-instructions`, and `force-for-plugin` (plugins only, unused here).

- `terse.md` — Terse, evidence-first engineering voice: minimal prose, no claim without concrete evidence, assumptions surfaced rather than silently resolved, weak proposals argued rather than accepted. Sets `keep-coding-instructions: true`, which preserves Claude Code's built-in coding instructions (scope discipline, comment policy, security, UI testing) that an output style otherwise replaces; the style governs tone and rigor only, and defers code mechanics to the project's `CLAUDE.md`.

Styles are read from `~/.claude/output-styles/` for the user, and from every `.claude/output-styles/` between the working directory and the repository root for a project, the closest one winning on a name collision. Selecting a style with `/config` writes the `outputStyle` field to `.claude/settings.local.json`; because the system prompt is read once at session start, the change only applies after `/clear` or in a new session.

Claude Code also ships a built-in `Concise` style (2.1.237 and later). It only shortens responses: result first, no preamble, narration or recap. `terse` is stricter on length (fragments, no praise, no restating the request) and adds behavior `Concise` does not have: separating verified facts from assumptions and naming the evidence, challenging weak plans once before complying, full sentences in durable artifacts (commits, PRs, documentation), and a language rule for replies and files. Pick `Concise` when responses are only too long; pick `terse` for the rigor. Only one style is active at a time, so they cannot be combined.

An output style applies to the main conversation only. Subagents such as the `agents/` definitions above run their own system prompt and do not inherit it.

### Root files

- `.claude-plugin/plugin.json` and `marketplace.json` — Claude Code plugin manifest and marketplace catalog (see Usage below). The repository is both: the marketplace lists a single plugin whose source is the repository root.
- `.editorconfig` — Shared editor settings (UTF-8, LF, 2-space indent, trailing whitespace preserved in Markdown to keep hard breaks).
- `.markdownlint-cli2.jsonc` — Markdown lint rules (see Development below).
- `lefthook.yml` — Git hooks config (pre-commit Markdown lint).
- `Taskfile.dist.yml` — Template for local dev commands (see Development below).
- `package.json` — npm scripts and dev dependencies backing the Taskfile and CI.
- `.github/workflows/quality.yml` — CI job running the Markdown lint and `claude plugin validate --strict` on push to `main` and on pull requests.
- `LICENSE` — MIT.

## Usage

Two ways to install, depending on the runtime.

### Claude Code plugin

```text
/plugin install ia-helpers --marketplace JMaxant/ia-helpers
```

Or in two steps: `/plugin marketplace add JMaxant/ia-helpers`, then `/plugin install ia-helpers@ia-helpers`. Update with `claude plugin update ia-helpers@ia-helpers`; the installed version follows the `version` field of `.claude-plugin/plugin.json`.

The plugin ships the three skills (`code-review`, `documentation`, `new-task`) and the `Terse` output style, which Claude Code lists as `ia-helpers:Terse`. It does not ship the `agents/` definitions, which duplicate the skills for runtimes without skill support: use the symlink recipe below for those. It is Claude Code only.

Names are stable: renaming a skill or the output style breaks existing installations, so it is treated as a breaking change. Anthropic does not review plugins and `plugin update` pulls whatever the repository holds, so the content is kept to Markdown only, with no hooks, MCP servers or executables; the risk is limited to the instructions themselves. In particular, `new-task` drives `gh` with your token and `code-review` writes `CODE_REVIEW.md`.

### Symlinks

For any other runtime, or to pin and adapt your own copy, the assets can be copied or symlinked into the configuration directory of the target tool. For Claude Code:

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
| `task qa` | `npm run lint:md` | Runs the Markdown lint on all files. |
| `task qa:fix` | `npm run lint:md:fix` | Auto-fixes what can be (currently Markdown lint issues). |
| — | `npm run lint:plugin` | Runs `claude plugin validate --strict` on the plugin (see below). |

The pre-commit hook (via `lefthook.yml`) runs the same Markdown lint on staged `*.md` files only, so most issues are caught before they ever reach CI.

Plugin validation needs the Claude Code CLI (`claude`). Run it inside a sandbox (a container, a devcontainer, an agent sandbox), never on the host: a `claude` installed on the host runs with your user's access to your home directory, credentials and other projects, and an agent sandbox limits what can leak out of it. CI runs it on every push to `main` and on pull requests, at the version pinned in `.github/workflows/quality.yml`, so it does not have to be run locally at all.

## Contributing

- Follow `.editorconfig`.
- One directory per skill, containing a `SKILL.md` with a `description` explicit enough for an agent to decide on its own when to trigger the skill.
- Keep supporting material in `templates/` (content to fill in) and `references/` (documentation loaded on demand) so the main `SKILL.md` stays short.
- When a skill and an agent cover the same subject, keep both in sync.
- One Markdown file per output style in `output-styles/`, named in kebab-case, with a `description` written for the `/config` picker. Set `keep-coding-instructions: true` unless the style deliberately drops Claude Code's software engineering instructions. Keep it to tone, rigor, and response format: project conventions belong in `CLAUDE.md`, and reusable workflows in a skill.
- Run `task qa` (or `npm run lint:md`) before committing; the pre-commit hook already checks staged Markdown files, `task qa` covers all of them. Plugin validation runs in CI.

## AI-generated tracker items

Trackers attribute API calls to the token owner, so a task created by an agent looks hand-written. `new-task` therefore discloses the agent's involvement in the body of every task it creates, shows that text in the draft the user approves, and lets a project turn it off or reword it. Commits keep their `Co-Authored-By` trailer.

## License

See [LICENSE](LICENSE).
