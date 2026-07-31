---
description: create work to be done in Shortcut based on a Markdown file
agent: orchestrator
subtask: true
---

Use the **shortcut skill** to connect to Shortcut and resolve the workspace.

$ARGUMENTS should contain a path to a plan markdown file and an optional team name. If the file path is missing, ask for it.

## Steps

1. **Resolve team** (if a team name was provided):
   - `GET /<workspace>/reference/groups` — find the group whose `name` matches (case-insensitive)
   - If no match, stop and list valid group names
   - Store the `group_id` for use in all create calls

2. **Parse the markdown file.** Look for content in this format:

```
# Epic: <epic name>

<epic description>

## Stories

### <Feature|Bug|Chore>: <story name>
**Estimate:** <number from the scale: 0, 1, 2, 3, 5, 8>
**Labels:** <comma-separated label names, optional>
**Blockers:** <pipe-separated list of story names>

<story description>

#### Acceptance Criteria
- [ ] <criterion>
```

3. **Find or create the epic** — `GET /<workspace>/epics` to check for an existing match before creating. Create via `POST /<workspace>/epics` with name, description, and `group_id` if provided.

4. **Create each story** — `POST /<workspace>/stories` sequentially. Track a `story name → story id` mapping as you go (name without the type prefix as the key). Field mapping:
   - `story_type`: lowercase type prefix (`feature`, `bug`, `chore`)
   - `description`: text between metadata and Acceptance Criteria
   - `estimate`: from `**Estimate:**` line (omit if absent)
   - `labels`: from `**Labels:**` line (omit if absent)
   - `tasks`: each `- [ ]` item under `#### Acceptance Criteria`
   - `epic_id`: from step 3
   - `group_id`: from step 1 if provided
   - **Do not include blockers in the create request** — handled in step 5

   Stop and report if any story creation fails.

5. **Create blocker links** — for each story with a `**Blockers:**` line, call `POST /<workspace>/stories/link` for each blocker:
   ```json
   { "subject_id": <blocker_id>, "object_id": <this_story_id>, "verb": "blocks" }
   ```
   Warn but continue if a blocker name doesn't match a created story.

6. **Validate** — `GET /<workspace>/epics/<epic_id>/stories` and verify each story: name, non-empty description, task count, estimate, and blocker IDs. Report mismatches as warnings.

7. **Summarize** — list everything created with `app_url` links and blocker relationships established.
