---
description: Generate user-friendly release notes from a commit range and append them to CHANGELOG.md
agent: orchestrator
subtask: true
disable-model-invocation: true
---

$ARGUMENTS contains the starting commit/tag and optionally the ending commit/tag. If only one argument, HEAD is the end. If no arguments, stop and ask.

## Step 1 — Resolve the version spans

Find all `rel-*` tags reachable within the range using:
```
git log <start>..<end> --format="%H %D" | grep "tag: rel-"
```

This produces a list of intermediate tags. Build a span list:
- First span: `<start>..<first_rel_tag>`
- Middle spans: each consecutive tag pair
- Final span: `<last_rel_tag>..<end>` (if end is not itself a `rel-*` tag, determine the version from a "Release version X.Y.Z" commit in that span)

Each span becomes one `## vX.Y.Z` section. If there are no intermediate tags, treat the whole range as one span.

Process spans in order from oldest to newest — output will be prepended to CHANGELOG.md so newest ends up at top.

## Step 2 — Collect commits

Run:
```
git log <start>..<end> --format="%H %s"
```

For each commit hash, run:
```
git show <hash> --format="%s%n%b" --no-patch
```

Extract from each commit:
- **Subject** — the first line
- **Body** — everything after the subject
- **Story IDs** — all `[sc-NNNNN]` tokens in the body (a commit may reference multiple)
- **Type** — from the Conventional Commit prefix: `feat` → feature, `fix` → bug fix, `chore`/`build`/`ci`/`refactor`/`test`/`docs`/`style`/`perf` → internal

Discard commits where type is internal AND the body contains no user-visible benefit. Discard merge commits, release bump commits ("Release version X.Y.Z", "RC version"), and debug commits (`debug(...)` prefix).

Completion criterion: every commit in the range has been inspected; a typed, story-annotated list exists.

## Step 3 — Enrich from Shortcut (optional, skip if proxy unavailable)

Use the **shortcut skill** to connect and resolve the workspace. If the proxy is unavailable, skip this step and note "Shortcut unavailable — commit messages only" in your working notes.

For each unique story ID collected in Step 2, fetch `GET /<workspace>/stories/<story_id>` and extract `name`, `story_type`, and `description`.

Use the story name and description as the **primary source** for what this change does. The commit body is secondary context. `chore` stories are internal — omit unless the description explicitly calls out a user-visible benefit.

Completion criterion: every story ID has been fetched or marked failed; failed fetches fall back to the commit body.

## Step 4 — Consolidate

Group commits by story ID. Commits sharing a story ID produce **one bullet**, not many. Commits with no story ID each produce at most one bullet (use the commit subject as the source).

Assign each bullet to a section:
- `feat` type or `feature` story → **What's New**
- `fix` type or `bug` story → **Bug Fixes**, but only if the bug was discovered in a **prior shipped release**. The signal is a `QA (X.Y.Z)` or `Production (X.Y.Z)` prefix in the Shortcut story description, or a commit message that references a prior version. Bugs caught during development of the feature in the same release are omitted.
- Everything else → omit

Completion criterion: no story ID appears in more than one bullet; bullet count is less than or equal to the number of distinct stories plus orphan commits.

## Step 5 — Write the bullets

**Audience:** End users of the CRM — salespeople and managers. They do not know what DTOs, migrations, reactive chains, jOOQ, EAV, Micronaut, R2DBC, or Groovy are.

**Voice:** Clear, direct, one idea per bullet, active voice, present tense. Each bullet answers: *"What can the user now do, or what problem did the user experience that is now fixed?"*

**Translation table — render the left as the right, never use the left:**

| Source term | Plain English |
|---|---|
| `feat(...)` / `fix(...)` prefix | drop — just describe the thing |
| EAV, custom field schema, manifest | custom fields |
| lead, opportunity, location | lead / deal / location (match the UI label) |
| import job, spreadsheet import wizard | import tool / import wizard |
| override / overwrite flag | option to update existing records |
| R2DBC, Mono, Flux, reactive | omit |
| DTO, entity, model, repository | omit |
| migration, Flyway | omit |
| URL path parameter, endpoint | omit |
| `addressLine2` | address line 2 |
| `uuidv7`, `gen_random_uuid` | omit |
| nuclear delete | omit (internal tooling) |

**Good bullet:**
> You can now choose to update existing leads when importing a spreadsheet, instead of skipping rows that already exist.

**Bad bullet:**
> Fixed GString serialization in BlObjectAdminController when entityType in NativeFieldsManifestDTO mismatched the URL path parameter.

Completion criterion: every bullet passes the "would a salesperson understand this?" test; no source term from the translation table appears in any bullet.

## Step 6 — Assemble and append

Format each section (newest first in the assembled block):
```markdown
## v{VERSION} — {Month D, YYYY}

### What's New

- [bullet]

### Bug Fixes

- [bullet]
```

Use today's date for all sections. Omit a section heading if it has no bullets.

Then:
1. Assemble all sections into a single block, newest version at top.
2. Check whether `CHANGELOG.md` exists in the repo root.
   - If it does, **prepend** the new block above the existing content.
   - If it doesn't, create it with the new block.
3. Write the file.
4. Report to the user: each version name, bullet count per section, and whether Shortcut was used.
