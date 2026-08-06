---
name: shortcut
description: Interact with Shortcut stories, epics, and references via the shortcut MCP server. Use when any task needs to read or write Shortcut data — fetching stories, creating epics, linking blockers, or resolving workspace metadata.
---

## Bootstrap — run once before any other call

Call the MCP tool `shortcut_list_workspaces` (no args). It returns a list of workspace slugs (e.g. `["leftlanewheelhouse"]`). Infer the correct slug from the current repo name or directory.

If the MCP server is unavailable (tool call errors or times out), report "Shortcut MCP unavailable" and stop — do not proceed with stale or guessed data. Do not fall back to `curl`/HTTP — the local proxy is no longer the interface; the MCP server is.

Every other tool below takes `workspace` as its first argument (the slug from bootstrap).

## Tools

| Tool | Args | Purpose |
|------|------|---------|
| `shortcut_list_workspaces` | — | List configured workspace slugs |
| `shortcut_list_epics` | `workspace` | List all epics (id, name, app_url) |
| `shortcut_create_epic` | `workspace, name, description?, group_id?` | Create an epic |
| `shortcut_list_epic_stories` | `workspace, epic_id` | All stories in an epic, full detail, resolved owners/iteration/labels |
| `shortcut_list_iteration_stories` | `workspace, iteration_id` | All stories in an iteration, full detail |
| `shortcut_get_story` | `workspace, story_id` | Fetch a single story by ID, resolved owners/iteration/labels |
| `shortcut_create_story` | `workspace, name, story_type?, description?, group_id?, workflow_state_id?, epic_id?, owner_ids?, labels?, estimate?, tasks?, story_links?` | Create a story |
| `shortcut_create_story_link` | `workspace, subject_id, object_id, verb` | Create a relationship (blocks/duplicates/relates to) between two stories |
| `shortcut_list_groups` | `workspace` | List groups (teams) |
| `shortcut_list_labels` | `workspace` | List labels |
| `shortcut_list_members` | `workspace` | List workspace members |
| `shortcut_list_iterations` | `workspace` | List iterations (sprints) |

All tools return JSON as a string — parse before use. Tool names above assume the MCP server is registered as `shortcut` in `opencode.json` (tool names are prefixed with the server name); adjust the prefix if it's registered under a different name.

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

Call `shortcut_create_story_link` with:
```json
{
  "workspace": "<slug>",
  "subject_id": <blocker_story_id>,
  "object_id": <blocked_story_id>,
  "verb": "blocks"
}
```

## Rules

- Never modify Shortcut data unless explicitly asked to create or link stories.
- If a story fetch returns no data or an error, fall back to the commit message for context — do not abort.
- `chore` story type is internal — omit from user-facing output unless the description explicitly describes a user-visible benefit.
