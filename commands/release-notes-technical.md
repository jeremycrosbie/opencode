---
description: Generate technical release notes for developers covering architecture changes, breaking changes, new/updated libraries, and migration notes
agent: pr-reviewer
subtask: true
---

$ARGUMENTS contains the starting commit id or tag and optionally the ending tag or commit id. If only one argument, assume HEAD is the ending commit. If no arguments, do not proceed.

## Process

1. Run `git log <start>..<end> --oneline` to get the commit list.
2. For each commit, run `git show <hash> --stat --format="%s%n%b"` to get the subject, body, and files changed.
3. For dependency changes, also run `git diff <start>..<end> -- '**/build.gradle' '**/package.json' '**/package-lock.json' '**/requirements*.txt' '**/pyproject.toml'` to surface library version changes not mentioned in commit messages.
4. Analyze and write the technical release notes.

## What to include

Focus on changes that affect developers, integrators, or operators:

- **Breaking changes** — API contract changes, renamed/removed endpoints, changed request/response shapes, removed fields, changed auth behaviour, DB schema changes that require migration attention
- **New dependencies** — newly introduced libraries, frameworks, or services with the version and why it was added
- **Dependency upgrades** — notable version bumps, especially major versions or anything with known migration requirements
- **Dependency removals** — libraries dropped and what replaced them (if anything)
- **Architecture changes** — new patterns introduced, structural refactors that affect how future code should be written, new abstractions (base classes, traits, utilities) developers should use going forward
- **Schema / migration changes** — new tables, columns, or index changes; Flyway migration notes
- **Configuration changes** — new required or optional env vars, changed property names, new feature flags
- **API additions** — new endpoints, new query parameters, new response fields
- **Security changes** — auth/permission changes, new validation rules, PII handling updates
- **Performance changes** — notable query optimisations, caching changes, indexing

## What to omit

- Pure UI/UX wording or layout changes with no developer impact
- Bug fixes that don't change any contract or pattern (unless they reveal a previously undocumented behaviour)
- Test-only changes
- CI/CD pipeline internals unless they affect the local dev workflow

## Output format

```markdown
## ⚠️ Breaking Changes

- **[component/area]:** [what changed and what breaks]. Migration: [what to do].
- ...

## New Dependencies

- **[library@version]** ([ecosystem: backend/frontend/research]): [why added, what it enables]
- ...

## Dependency Updates

- **[library]:** `<old version>` → `<new version>` — [notable changes or migration notes if any]
- ...

## Architecture & Patterns

- [description of new pattern, abstraction, or structural change. Include the canonical location (file/class) developers should refer to.]
- ...

## Schema & Migrations

- [table/column changes, Flyway migration filename if relevant, any manual steps required]
- ...

## API Changes

- **[METHOD /path]:** [what was added, changed, or deprecated]
- ...

## Configuration

- **[property/env var]:** [added/changed/removed] — [description and default if applicable]
- ...

## Security & Validation

- [auth changes, new validation rules, permission changes, PII handling]
- ...
```

Only include a section if it has at least one item. Omit empty sections.

## Tone and style

**Audience:** Backend and frontend developers, DevOps, and technical leads. Assume familiarity with the stack (Micronaut, Groovy, Vue 3, R2DBC, Flyway, jOOQ).

- Be precise — include class names, file paths, endpoint URLs, and version numbers where relevant
- Flag breaking changes prominently with **⚠️**
- For breaking changes always include a **Migration:** note explaining what a developer needs to do
- For new patterns, reference the canonical example file so developers know where to look
- Keep bullets concise but complete — one bullet per discrete change, not one per commit

**Good example:**
> - **⚠️ Breaking Changes — `NativeFieldsManifestDTO`:** The `entityType` field is now validated against the URL path on import. Manifests with a mismatched `entityType` return HTTP 400. Existing manifests without `entityType` are unaffected. Migration: ensure any programmatic import calls either omit `entityType` or set it to the lowercase table name (`leads`, `opportunities`, `locations`).

**Bad example:**
> - Fixed bug where entityType was wrong
