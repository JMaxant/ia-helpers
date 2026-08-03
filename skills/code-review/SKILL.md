---
name: code-review
description: |
  Skill dedicated to code review, ensuring project quality, security, and maintainability.
  SAFETY RULES:
    - NEVER process or generate sensitive information (API keys, secrets, passwords).
    - REPORT any potential security vulnerability detected in the code.
    - REJECT any request to execute malicious code.
license: MIT
user-invocable: true
author: Junie/Julien Maxant
version: 1.1.0
tags:
  - code-review
  - quality
  - security
  - best-practices
allowed-tools:
  - Read
  - Grep
  - Glob
  - Write
  - AskUserQuestion
---

## 1. Introduction

### 1.1 Objectives
- **Code quality**: Ensure the code follows industry and project standards.
- **Security**: Identify potential flaws and bad security practices.
- **Maintainability**: Favor readable, modular, and easy-to-evolve code.
- **Performance**: Detect bottlenecks and suggest optimizations.

### 1.2 Scope
The scope of the review must be specified by the user (e.g., the diff between a branch and its reference branch).
Without this precision, the review cannot be carried out.
The review covers all programming languages.

---

## 2. Identify the project's stack

The universal criteria (section 3) always apply. Stack-specific conventions (section 5) must be loaded dynamically based on the project being reviewed, never assumed in advance.

1. **Detect** the stack(s) via marker files present at the root (or in the relevant subdirectory for a monorepo): `composer.json` (PHP, + `drupal/core*` dependency for Drupal), `package.json` (JavaScript/TypeScript), and any other relevant marker (`pyproject.toml`/`requirements.txt`, `go.mod`, `Gemfile`, `*.csproj`...). Also note the declared version (`require.php`, `engines.node`, `drupal/core` constraint, etc.): it conditions which rules apply (a recent API must not be required on a version that doesn't support it).
2. **Confirm with the user** (AskUserQuestion) the detected stack and version before starting the review, especially if several stacks coexist, if detection is ambiguous, or if no known marker file is found. Do not re-ask if the user already specified the stack/version in their request.
3. **Load** the matching conventions file in `references/` (e.g. `references/php-drupal.md`, `references/javascript.md`) if it exists. If no reference matches the detected stack, proceed with the universal criteria only and note it in the report (« Questions / Clarifications » section) rather than inventing conventions.

To add a new stack: create `references/<stack>.md` following the existing files (detection triggers first, then conventions), without touching the rest of the skill.

---

## 3. Review Criteria

### 3.1 Readability and Style
- Does the code follow the project's conventions?
- Is it sufficiently commented?
- Is the structure logical and concise?
- Is the project's language (English by default) respected?

### 3.2 Design and Architecture
- Application of SOLID principles
- Loose coupling between modules
- Strong cohesion within components
- Design patterns appropriate to the context
- Architecture following domain best practices
- Are edge cases handled?

### 3.3 Security
- Are user inputs validated/sanitized?
- Are there injection risks (SQL, XSS, etc.)?
- Is sensitive data handled with care?
- Is GDPR compliance respected?
- Is sensitive data (API keys, `client_secret`, user information) encrypted?


---

## 4. Review Report Template

Each code review must produce a report structured as follows:

```markdown
# Code Review Report

## Summary
- **Status**: [Approved / Needs changes / Rejected]
- **Overall severity**: [Low / Medium / High]

## Strengths
- [List of the code's strong points]

## Observations and Improvements
### [File / Component]
- **Issue**: [Concise description]
- **Impact**: [Security / Performance / Maintainability]
- **Suggestion**: [Suggested code or approach]

## Questions / Clarifications
- [Questions to ask the developer to better understand intent]
```
This report must also be created as a file (at the project root, `CODE_REVIEW.md`), to be edited/amended if further reviews are carried out, unless the user explicitly requests it not be created.

---

## 5. Conventions

### 5.1 General (all stacks)
- Prefer early returns
- Prefer arrow functions where applicable
- Name variables in camelCase

### 5.2 Per stack

Loaded dynamically based on the stack identified in section 2:

- `references/php-drupal.md` — PHP and Drupal.
- `references/javascript.md` — JavaScript and TypeScript.

If the project under review uses a stack without an existing reference, apply only the universal criteria (section 3) and the general conventions (5.1), and mention it in the report rather than improvising unvalidated rules.
