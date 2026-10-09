---
name: handoff-trigger
description: Decide when to recommend a handoff and a new session, from the size of the context. Use when a `context-guard` message appears, right after a compaction, when a finished task is followed by an unrelated one while the context is large (about 200k tokens or more), or when you notice the session degrading (repeating a fix Kade rejected, forgetting a guardrail). It recommends the handoff and offers the choices; the `session-handoff` skill does the handoff.
---

# Handoff trigger

This is about the session, not the task. The `context-guard` hook (in `enjay27/claude-global`) warns
by size; this skill says what to do when it does, and what to do in a session without the hook.

Quality drops gradually as the context fills ("context rot"), well before the window is full, and
auto-compaction replaces history with a summary. **Task boundaries matter more than the number.**

- **Offer a handoff and a new session** (the `session-handoff` skill) when a task is finished and
  the next one is unrelated, and **switch early** when you notice any of these in yourself: you
  repeat a fix Kade already rejected, you have been corrected twice on the same issue, you forget a
  guardrail, or you re-read files you already saw.
- **Numbers, for a 1M window:** under about 150k, no concern. At **200k** (`context-guard` warns),
  finish the current unit of work, then offer the handoff. At **400k** (`context-guard` recommends
  the handoff), recommend it and do not start new multi-step work until he answers. A smaller
  window: 40% and 60% of it.
- When a `context-guard` message appears, or right after a compaction, tell Kade the number in one
  line and offer three choices:
  1. **handoff and a new session** (the `session-handoff` skill),
  2. `/compact` with a focus he names,
  3. continue.
- In sessions without the hook, offer the same choices after a compaction, after a finished task,
  or when the session has run many large tool outputs.
- Finish or park the current step before a handoff; never hand off mid-edit. Update the state
  (*Now* or the Project) and commit first, so the new session starts from that summary.
