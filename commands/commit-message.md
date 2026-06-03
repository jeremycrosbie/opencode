---
description: Generate a Conventional Commits formatted commit message
agent: pr-reviewer
subtask: true
---

Analyze all staged and unstaged changes and generate a commit message following the Conventional Commits v1.0.0 specification (https://www.conventionalcommits.org/en/v1.0.0/).

## Format

```
<type>(<scope>): <description>

<body>

<footer(s)>
```

## Rules

1. **type** (required): One of `feat`, `fix`, `refactor`, `test`, `docs`, `style`, `perf`, `build`, `ci`, `chore`, `revert`
   - `feat` = new feature
   - `fix` = bug fix
   - `refactor` = code change that neither fixes a bug nor adds a feature
   - `test` = adding or updating tests only
   - `docs` = documentation only
   - Use the most specific type that applies
2. **scope** (recommended): A noun describing the section of the codebase in parentheses, e.g. `(dropbox)`, `(auth)`, `(frontend)`
3. **description** (required): Imperative, lowercase, no period at the end. Short summary of the change.
4. The type, scope, and description should be no longer than 50 characters where possible, but absolutely no longer than 72 characters.
5. **body** (recommended): Explain the *why*, not the *what*. Separated from description by a blank line. Wrap at 72 characters.
6. **footer** (when applicable): `BREAKING CHANGE: <description>` for breaking changes. `Refs: #<issue>` for related issues.
7. Append `!` after type/scope for breaking changes: `feat(api)!: ...`

## Issue Tracker Reference

If `$1` is provided, it is an issue tracker reference (e.g. `sc-12345`). Append it to the very end of the commit message on its own line, wrapped in square brackets, with a blank line before it:

```
<type>(<scope>): <description>

<body>

<footer(s)>

[sc-12345]
```

If `$1` is not provided or is empty, omit this line entirely.

## Process

1. Run `git status` to see all changed files
2. Run `git diff --cached` to see staged changes (prioritize these)
3. Run `git diff` to see unstaged changes
4. Run `git log --oneline -5` to see recent commit style for context
5. Analyze all changes and draft a commit message
6. Present the message formatted in a code block so it's easy to copy
7. Ask if the user wants to commit with this message

$ARGUMENTS
