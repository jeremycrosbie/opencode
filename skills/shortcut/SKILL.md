---
name: shortcut
description: Interact with Shortcut stories, epics, and references via the local proxy. Use when any task needs to read or write Shortcut data — fetching stories, creating epics, linking blockers, or resolving workspace metadata.
---

## Bootstrap — run once before any other call

```
curl --max-time 3 -s http://localhost:8000/workspaces
```

Returns a list of workspace slugs (e.g. `["leftlanewheelhouse"]`). Infer the correct slug from the current repo name or directory. If the call fails or times out, report "Shortcut proxy unavailable" and stop — do not proceed with stale or guessed data.

Confirm auth and discover available operations:
```
GET http://localhost:8000/<workspace>/capabilities
```

All subsequent calls use `http://localhost:8000/<workspace>/` as the base URL.

## Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/workspaces` | List configured workspace slugs |
| `GET` | `/<workspace>/capabilities` | Confirm auth, list available operations |
| `GET` | `/<workspace>/epics` | List all epics (id, name) |
| `POST` | `/<workspace>/epics` | Create an epic |
| `GET` | `/<workspace>/epics/<epic_id>/stories` | All stories in an epic with full detail |
| `GET` | `/<workspace>/stories/<story_id>` | Fetch a single story by ID |
| `POST` | `/<workspace>/stories` | Create a story |
| `POST` | `/<workspace>/stories/link` | Create a blocking relationship between two stories |
| `GET` | `/<workspace>/reference/groups` | List groups (teams) |
| `GET` | `/<workspace>/reference/labels` | List labels |

Use `curl` for all calls.

## Story shape (key fields)

| Field | Notes |
|-------|-------|
| `name` | Story title |
| `story_type` | `feature`, `bug`, or `chore` |
| `description` | Full story body — check for `QA (X.Y.Z)` or `Production (X.Y.Z)` prefix to identify bugs reported against a prior release |
| `estimate` | Fibonacci point value |
| `tasks` | Acceptance criteria (`description`, `complete`) |
| `blocked_by_ids` | IDs of blocking stories |
| `app_url` | Link to the story in Shortcut |
| `group_id` | Team assignment |
| `labels` | Array of label objects |

## Blocker link shape

```json
POST /<workspace>/stories/link
{
  "subject_id": <blocker_story_id>,
  "object_id": <blocked_story_id>,
  "verb": "blocks"
}
```

## Rules

- Never modify Shortcut data unless explicitly asked to create or link stories.
- If a story fetch returns no data or a 404, fall back to the commit message for context — do not abort.
- `chore` story type is internal — omit from user-facing output unless the description explicitly describes a user-visible benefit.
