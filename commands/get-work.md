---
description: Get work from Shortcut, through a proxy, plan, and implement
agent: orchestrator
subtask: true
---

You are retrieving epics and stories from Shortcut via the local story proxy so the user can plan or implement work.

Before doing anything, you need:
1. **The proxy base URL** (e.g., `http://localhost:8000`)

Check if $ARGUMENTS contains a URL. If not, ask the user for it.

## Steps

1. `GET <base_url>/capabilities` will describe all supported operations.
2. **List epics** by calling `GET <base_url>/epics`. Present the list (id, name) and ask the user which epic to look at. If there's only one, use it.

3. **Fetch stories** by calling `GET <base_url>/epics/<epic_id>/stories`.

4. **Determine work order.** Sort stories so unblocked stories come first:
   - A story is "unblocked" if its `blocked_by_ids` list is empty
   - A story is "ready" if all stories in its `blocked_by_ids` have already appeared earlier in the sorted order
   - If circular dependencies exist, flag them and list those stories at the end

5. **Present the results.** For each story (in dependency order), show:
   - Name, type, estimate, story ID, and link (`app_url`)
   - Description (the main content — this is the spec)
   - Acceptance criteria (`tasks` array — description and completion status)
   - Blocked by (list the *names* of blocking stories, not just IDs — look them up from the same response)
   - Workflow state ID

Use this as the working context for whatever the user wants to do next — planning, implementation, or review.

Use `curl` to make the API calls.

Do not modify any data in Shortcut.
