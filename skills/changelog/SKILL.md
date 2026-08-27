---
name: changelog
description: Generate an end-user-facing CHANGELOG.md entry summarizing everything between two git tags. Use when the user wants to update the changelog, write release notes, or asks to document a tagged release.
---

Produces one changelog entry — What's New, Bug Fixes, and (if any) Breaking Changes — for the range between two git tags. Written for end users: no file names, class names, ticket IDs, or implementation detail in the output copy. Match the existing `CHANGELOG.md` format on `main` exactly — check it before running this if you haven't recently, formats drift.

This skill takes a **range**, not a single tag. Given `rel-0.12.1`/`rel-0.12.2` it emits one entry for that release. Given `rel-0.1.0`/`rel-0.12.2` it emits **one combined entry** summarizing the whole span — it does not walk and split by every intervening tag. If you want per-release entries for a backfill, invoke it once per adjacent tag pair.

Requires the `shortcut` skill's proxy (bootstrap per that skill before calling any Shortcut endpoint here).

## 1. Pin the range

Get `start_tag` and `end_tag` from the user. Confirm both resolve:

```bash
git rev-parse "$start_tag" "$end_tag"
```

If either fails, stop and ask — do not guess a tag name.

## 2. Determine the entry header

- **Version**: read `VERSION` at `end_tag` — `git show {end_tag}:VERSION`. Fall back to `end_tag` with any `rel-` prefix stripped if the file doesn't exist at that ref.
- **Date**: `end_tag`'s commit date — `git log -1 --format=%ad --date=short {end_tag}`, reformatted to `Month D, YYYY` (e.g. `2026-08-03` → `August 3, 2026`).

Header format: `## v{version} — {Month D, YYYY}` (em dash, not hyphen — matches every existing entry in `CHANGELOG.md`).

## 3. Collect the range

```bash
git log {start_tag}..{end_tag} --format='%H%x00%s%x00%b%x00---COMMIT---'
```

Keep subjects and bodies together per-commit — both breaking-change detection and Shortcut ID extraction need the full message, not just the subject line.

## 4. Breaking changes — from git, not Shortcut

Scan each commit **subject** for conventional-commit `!` before the colon (`feat!:`, `fix(scope)!:`, etc.). For each match, note the raw subject — you'll rephrase it for end users in step 7. If a matched commit also references an `sc-\d+` ID, prefer the linked story's `name` as the basis for the rewrite; it's usually closer to user-facing language than a commit subject.

No match across the whole range → the Breaking Changes section is omitted entirely (not emitted thin).

## 5. Find candidate Shortcut stories

Regex `sc-\d+` across the **full message** (subject + body) of every commit in range — not just a structured footer. Real commit history here references story IDs loosely in prose (e.g. "fixed in sc-24415"), so a footer-only match misses most of them. Dedupe IDs: multiple commits touching the same story collapse to one bullet later.

Commits with no `sc-\d+` anywhere are invisible to this process — there's no story to classify against, so they're silently excluded, not flagged. This is expected for chores, jOOQ regen, version-bump commits, etc.

## 6. Classify each story

For each deduped ID, `GET /{workspace}/stories/{id}` (see `shortcut` skill for proxy bootstrap). A 404 or fetch failure: drop the ID into an "unresolved" list to report back at the end — do not guess its classification or silently omit it from your final report.

**Shipped check, before classification.** A commit mentioning `sc-XXXX` does not mean that story shipped in this release — commits reference related or future work by ID too (e.g. "tracked separately as a fast-follow, sc-24401"). Before classifying, check the story's `tasks` array: if it has one or more tasks and *any* task has `complete: false`, the story is not done — exclude it and add it to the unresolved list, regardless of `story_type`. If `tasks` is empty, there's nothing to verify mechanically — proceed to classification but note it in the unresolved list as low-confidence, since there's no completion signal to check.

There's no reliable way to resolve `workflow_state_id` to a human name through the Shortcut proxy (no endpoint exposes it) — don't try to hardcode or infer meaning from that field's numeric value.

Classification is **entirely Shortcut's**, not commit-message interpretation. Default assumption: a `bug` story with no prefix describes behavior that was already live, i.e. a real shipped bug — include it. The `QA (X.Y.Z)` / `UAT (X.Y.Z)` prefixes are the exception: they mean the bug was caught internally before release, so only exclude when that prefix's version matches the release currently being cut (never reached a user). `Production (X.Y.Z)` is just an explicit confirmation of the default — treat it the same as no prefix.

| `story_type` | Rule | Bucket |
|---|---|---|
| `feature` | always | What's New |
| `bug` | no prefix, or `Production (X.Y.Z)` | Bug Fixes |
| `bug` | `QA (X.Y.Z)` or `UAT (X.Y.Z)`, X.Y.Z = the version being released | excluded — caught before it ever shipped to a user |
| `bug` | `QA (X.Y.Z)` or `UAT (X.Y.Z)`, X.Y.Z = an earlier version | Bug Fixes — reopened/stale catch, but it still reached users in that earlier release |
| `chore` | `description` explicitly states a user-visible benefit | whichever of the two buckets above fits the benefit |
| `chore` | otherwise | excluded |

## 7. Write end-user copy

One bullet per story (not per commit). Base the bullet on the story's `name`/`description`, not the commit message — commit messages carry implementation detail (file names, DTOs, migration IDs) that doesn't belong in front of a user. Rephrase in plain language: what changed for the person using the product, not how it was built.

Match the existing style in `CHANGELOG.md`: headline-worthy items under **What's New** lead with a **bolded** short phrase naming the feature, then an em dash and a plain-language description (e.g. `- **Location ID** — a short code...`). Minor tweaks don't need the bold lead-in — a plain sentence is fine (e.g. `- Dates now default to 12-hour format...`). **Bug Fixes** bullets are always plain sentences, no bold.

For breaking-change commits with no linked story, rephrase the commit subject the same way — strip the conventional-commit prefix and any internal terms before it goes in the entry.

## 8. Assemble and write the entry

```
## v{version} — {Month D, YYYY}

### Breaking Changes   <!-- omit entirely if step 4 found nothing -->
- ...

### What's New          <!-- omit entirely if nothing classified here -->
- ...

### Bug Fixes           <!-- omit entirely if nothing classified here -->
- ...
```

**Omit empty sections — never emit a header with no bullets under it.** This matches every existing entry: `v0.1.7` has only `### What's New`, `v0.1.6` has only `### Bug Fixes`, `v0.1.0` has no section headers at all. If a whole entry has nothing in any bucket (common for pre-`sc-`-era or pre-labeling tags), write a single plain-prose line instead of any headers — follow `v0.1.0`'s `Initial release.` pattern: something like `Internal changes only — see git history for this range.` Never fabricate content, and never leave an empty header for the user to "fill in" — omit it instead and say so in your report (step 9).

Prepend the new entry to the top of `CHANGELOG.md` at the repo root, directly under the `# Changelog` title and its `---` rule (create the file with that title if it doesn't exist yet). Newest entry on top. Insert a blank line, `---`, blank line **after** your new entry to separate it from the next (previously topmost) entry — every entry boundary in the file follows this pattern except end-of-file, which has no trailing rule. Never rewrite or reorder existing entries below it.

## 9. Report back

After writing, tell the user:
- The entry that was added (or a diff/preview).
- Any unresolved story IDs from step 6 — 404s/fetch failures, stories excluded for incomplete tasks, and stories with no tasks to verify completion against — these need manual follow-up in Shortcut or the changelog.
- Any breaking-change commits that got rephrased — worth a human glance since the rewrite is a judgment call, not mechanical.
- Any section omitted for lack of content, or a plain-prose fallback used in place of a fully thin entry.
