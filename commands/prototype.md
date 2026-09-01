---
description: Build a throwaway prototype (logic or UI) to answer a design question
agent: prototyper
subtask: true
---

Build a throwaway prototype that answers a design question, following the `prototype` skill.

First decide the branch:
- **LOGIC** (single double-clickable HTML file, pure liftable module) when the question is about business logic, a state machine, or data shape.
- **UI** (variations on one route in the project's real stack) when the question is about what something should look like.

If the branch is ambiguous from the request below and the user is reachable, ask which question they want answered before building. Otherwise pick per the skill's default and state the assumption at the top of the artifact.

Build it, verify the logic actually works (don't assert on vibes), then report where it lives, how to run it, and the design questions the human should judge while driving it.

The thing to prototype: $ARGUMENTS
