---
description: Get work from Shortcut, through a proxy, plan, and implement
agent: orchestrator
subtask: true
---

Use the **shortcut skill** to connect to Shortcut and resolve the workspace.

## Steps

1. **List epics** — `GET /<workspace>/epics`. Present the list (id, name) and ask the user which epic to work on. If there's only one, use it.

2. **Fetch stories** — `GET /<workspace>/epics/<epic_id>/stories`.

3. **Determine work order.** Sort stories so unblocked stories come first:
   - A story is "unblocked" if its `blocked_by_ids` list is empty
   - A story is "ready" if all stories in its `blocked_by_ids` have already appeared earlier in the sorted order
   - If circular dependencies exist, flag them and list those stories at the end

4. **Present the results.** For each story (in dependency order), show:
   - Name, type, estimate, story ID, and link (`app_url`)
   - Description (the main content — this is the spec)
   - Acceptance criteria (`tasks` array — description and completion status)
   - Blocked by (list the *names* of blocking stories, not just IDs)

Use this as the working context for whatever the user wants to do next — planning, implementation, or review.
