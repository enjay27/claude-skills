---
name: session-resume
description: Start a session in one of Kade's repositories by reconstructing where the work stands - read the state (STATUS.md on the status branch, or MEMORY.md), open and recently merged PRs, and the current branch, then summarize and propose the next task. Use when Kade opens a session with "resume", "continue", "where were we", or pastes a starter prompt from a handoff.
---

# Session resume

Goal: in one short reply, Kade sees where things stand and what you propose to do next.
Do not edit anything until he confirms (kade-workflow, plan first).

## 1. Read the state

Use the first source that exists:

1. **Org Project repositories** (Stella Rain): the snapshot on the `status` branch of `app`.
   ```bash
   git fetch origin status && git show origin/status:STATUS.md
   ```
   Check its *Generated* time. If it is older than the work needs, run the snapshot workflow
   (`project-snapshot.yml`, `workflow_dispatch`, a repository-scoped call), wait for it, and read again.
2. **`MEMORY.md` repositories**: `MEMORY.md` (often already loaded through `CLAUDE.md`).
   If its *Now* names a handoff or session file, read that one file, not the whole folder.

If Kade pasted a starter prompt from `session-handoff`, it names the branch, PR and next step:
trust it, then confirm against the sources above.

## 2. Read the recent work

- Open PRs from `claude/*` branches: their status and CI result.
- The last 3 to 5 merged PRs: read their *NOT VERIFIED* and *Wrong turns* sections.
- `git status` and the current branch. Uncommitted changes are Kade's unless the starter
  prompt says they are a parked task.

## 3. Report (at most about 15 lines)

- **Where things stand**: one or two lines.
- **Now**: the items in progress.
- **Needs Kade / NOT VERIFIED**: anything waiting on him or unproven.
- **Proposed next task**: one task, with the gate it needs and whether this session can run it.

Then ask him to confirm or redirect.
