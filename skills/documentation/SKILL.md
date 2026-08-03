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
version: 1.1.0
tags:
  - documentation
  - optimization
  - templates
  - automation
allowed-tools:
  - Read
  - Write
  - AskUserQuestion
---

## 1. Introduction

### 1.1 Objectives
- **Clarity**: Make the documentation understandable by all target audiences.
- **Maintainability**: Ease updates and corrections through a modular structure.
- **Automation**: Reduce the time spent writing and updating documentation.
- **Standardization**: Apply style and format standards for overall consistency.

The generated documentation must be clear and understandable for all audiences: it should serve as an entry point for understanding a project or a specific feature, but it must primarily explain the functional side and enable fast onboarding.
Unless explicitly instructed otherwise, it is not meant to be an exhaustive technical coverage of the scope being documented.

### 1.2 Target Audience
- Developers
- Project managers

---

## 2. Choosing the writing language

Before writing anything, determine the language the documentation should be written in:

1. If the user explicitly states a language (in the request, or in project conventions such as CLAUDE.md/CONTRIBUTING), use it without asking.
2. Otherwise, check for an existing convention: the language already used in the project's existing documentation (`docs/`, `README.md`) or in `CLAUDE.md`/`CONTRIBUTING`.
3. If no convention can be inferred, ask the user (AskUserQuestion) which language to write in before drafting any content. Do not default silently to a language.
4. Keep a single language per document; do not mix languages within the same file unless explicitly requested.

---

## 3. Frontmatter Standards

**Every documentation file must start with a YAML frontmatter block**:

```markdown
---
title: Document title
author: Author or team name
created: YYYY-MM-DD
modified: YYYY-MM-DD
version: x.x.x
description: Brief description of the document's content.
tags:
  - tag1
  - tag2
---
```

## 4. General Best Practices
### 4.1 Documentation Structure

**Clear hierarchy:** Use explicit titles and subtitles.
**Modularity:** Split documentation into thematic files or sections.
**Intuitive navigation:** Add a table of contents for long documents.
**Source of truth:** Question the user to confirm the accuracy of the content.
**Allowed directories:** Documentation must live in the `docs` directory at the project's git root, unless explicitly instructed otherwise.

### 4.2 Style and Tone
**Simple language:** Avoid unexplained technical jargon.
**Concrete examples:** Illustrate each concept with real use cases.
**Active voice:** Favor direct phrasing.
**Emoji use is prohibited.**

### 4.3 Accessibility
**Markdown format**: Use [CommonMark](https://commonmark.org/) or GitHub Flavored Markdown for maximum compatibility.
**Images and diagrams**: Add descriptions (alt text).
**Writing language**: as determined in section 2.

## 5. Predefined Templates
### 5.1 General README Template

This template applies when creating general project documentation.
```markdown
---
title: README
author: [Author name]
created: [YYYY-MM-DD]
modified: [YYYY-MM-DD]
version: [x.x.x]
template-version: 1.0.0
description: General overview of the project, its purpose, and its main features.
tags:
  - introduction
  - overview
---

# [Project Name]


## Environments
|Environment     | Url                         |
|----------------|-----------------------------|
| Local          | http://localhost            |
| Dev            | http://dev.example.com      |
| Staging        | https://staging.example.com |
| Production     | https://example.com         |


## Tech Stack

| Tech                          | Version                       |
|------------------------------|-------------------------------|
| [Tech name (e.g. PHP)]        | [Version number (e.g. 8.3)]   |

## Prerequisites
- [List of required dependencies or tools.]

## Installation
### DDEV

1. **Prerequisites:**
    - [Docker](https://docs.docker.com/engine/install/)
    - [Ddev](https://ddev.readthedocs.io/en/latest/users/install/ddev-installation/)
    - _Optional:_ GNUMake (`sudo apt install make`) or install Taskfile (see below)
2. [The various project setup steps]

## Build tools
[Description of the tasks declared in the Makefile/Taskfile if applicable]

```

### 5.2 Technical/Functional Documentation Template

```markdown
---
title: [Document title]
author: [Author name]
created: [YYYY-MM-DD]
modified: [YYYY-MM-DD]
version: [x.x.x]
template-version: 1.0.0
description: [Description of the technical content covered]
tags:
  - technical/functional
  - [other relevant tag]
---

# [Document title]

## Introduction
[Context and objectives of the document]

## Scope
[Boundaries and scope covered]

## Workflow
[Step-by-step description of the process]

## Key Concepts
- [Concept 1]: [Description]
- [Concept 2]: [Description]

## Implementation
[Technical details, diagrams, or generic code examples (optional for purely functional documents)]

## Appendix
[Additional information or references]
```

### 5.3 Changelog

```markdown
---
title: CHANGELOG
author: [Name]
created: [YYYY-MM-DD]
modified: [YYYY-MM-DD]
version: [x.x.x]
template-version: 1.0.0
description: History of the project's changes.
tags:
  - changelog
  - updates
---

# CHANGELOG

## [x.x.x] - YYYY-MM-DD
### Added
- [New feature]
### Fixed
- [Bug fix]
```
