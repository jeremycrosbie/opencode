---
name: commit-message-writer
description: Generate Conventional Commit messages by inspecting staged and unstaged Git changes in a repository. Use when the user asks to write, draft, improve, or validate a commit message based on current repository work.
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

If a story number is provided, it is an issue tracker reference (e.g. `sc-12345`). Append it to the very end of the commit message on its own line, wrapped in square brackets, with a blank line before it:

```
<type>(<scope>): <description>

<body>

<footer(s)>

[sc-12345]
```

If a story number is not provided or is empty, omit this line entirely.

## Process

1. Run `git status` to see all changed files
2. Run `git diff --cached` to inspect staged changes
3. Run `git diff` to inspect unstaged changes
4. Run `git log --oneline -5` to learn local commit style
5. Subtask: summarize the work performed
   - Identify the primary intent of the changes
   - Separate staged vs unstaged changes
   - Note whether changes are feature, fix, refactor, docs, tests, etc.
   - Detect breaking changes, issue references, migrations, or risky behavior changes
6. Use the summary to draft a Conventional Commit message
7. Validate that the message matches the actual diff and does not overclaim
8. Present the message in a code block
9. Ask if the user wants to commit with this message
