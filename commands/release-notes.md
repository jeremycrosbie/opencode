---
description: Generate user-friendly release notes from a commit range
agent: pr-reviewer
subtask: true
---

$ARGUMENTS contains the starting commit id or tag and optionally the ending tag or commit id. If only one argument, assume HEAD is the ending commit. If no arguments, do not proceed.

## Process

1. Run `git log <start>..<end> --oneline` to get the list of commits in the range.
2. For each commit, run `git show <hash> --stat --format="%s%n%b"` to get the subject, body, and files changed. Do NOT use `git diff` — you need commit intent, not code.
3. Analyze each commit message for user-visible changes. Ignore anything that is purely internal (dependency bumps, refactors with no user impact, CI config, test-only changes, code style).
4. Write the release notes.

## Writing rules

**Audience:** End users of the application — not developers. They do not know what DTOs, R2DBC, EAV, reactive chains, Groovy, Micronaut, or Serde are. Write as if explaining to a non-technical product user.

**Tone:** Clear, direct, friendly. One idea per bullet. Active voice. No jargon.

**Translation guide — never use these; use the plain version instead:**

| Technical term | Plain English |
|---|---|
| `feat(...)` prefix | (drop it — just describe the feature) |
| `fix(...)` prefix | (drop it — just describe what was fixed) |
| `refactor` | (omit unless user-visible — if so, describe the benefit) |
| DTO, entity, model | (omit — describe what the user can now do) |
| EAV, custom field schema | custom fields |
| R2DBC, reactive, Mono, Flux | (omit entirely) |
| GString, Serde, Micronaut | (omit entirely) |
| `addressLine2` | address line 2 / suite / apartment number |
| `entityType`, `typeName` | entity type / field category |
| `manifest` | custom fields file / import file |
| `updatePolicy=force` | force import option |
| URL path parameter | (omit — describe the behaviour) |

## Output format

```markdown
## What's New

- [user-facing description of new capability]
- ...

## Bug Fixes

- [user-facing description of what was broken and is now fixed]
- ...

## Improvements

- [user-facing description of something that works better, is clearer, or is more reliable]
- ...
```

Only include a section if it has at least one item. Omit sections with no relevant changes.

Each bullet should be a single sentence that answers: *"What can the user now do, or what problem did the user experience that is now resolved?"*

**Good example:**
> - Importing a custom fields file now automatically applies it to the correct entity type (Leads, Opportunities, or Locations) regardless of which tab is active when you click Import.

**Bad example:**
> - Fixed GString serialization error in BlObjectAdminController.importFieldsManifest when entityType in NativeFieldsManifestDTO did not match the URL path parameter typeName.
