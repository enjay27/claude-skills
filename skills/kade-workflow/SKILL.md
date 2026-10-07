---
name: kade-workflow
description: Kade's standing way of working in every one of his repositories (Stella Rain, resonance-stream, lakehouse-k8s and others). Use at the start of any task that edits code, config, CI or docs in a repository - plan first, test first, run the gates, commit and PR style, recording state, and what to do when the context window fills up.
---

# Kade's workflow

The repository's `CLAUDE.md` says **what** applies there: its parts, gates, branches and where
state lives. This skill says **how** to work in any of them. Where they differ, `CLAUDE.md` wins.

## 1. Plan before editing

- Locate the code first (graft if the repo has it, otherwise search), then present an impact
  analysis: which files change, what logic changes, what does not, which gate applies, and
  whether this session can run that gate.
- **Wait for Kade's explicit approval.** Approval covers the steps in the plan, nothing more.
  A new step that widens the scope goes back to him.
- "Follow your recommendation" from Kade means: do what you recommended.

## 2. Test first

New behaviour and bug fixes start with a failing test that pins the wanted behaviour; run it
and see it fail for the right reason, then write the code. If a change cannot be unit-tested
(platform glue, UI, a cluster), say so in the commit body.

## 3. Execute

- One task at a time. A move or rename changes no behaviour; change behaviour in a separate commit.
- Edit files in place. Never retype a file from earlier tool output, which may be truncated.
- At most **2** self-corrections on a failing gate, then stop and report.

## 4. Verify honestly

- Run the gate for **every part touched**, as the repository's `CLAUDE.md` defines it.
- **Never report a gate as passed when it could not run.** Name it in the last commit body:
  `NOT VERIFIED: <gate>: <reason>` (for example, no Windows toolchain, no cluster reach).
- A blocked network host is **asked for**, never worked around: name the host and what needs it.
  No mirrors, stubs or patched dependencies.

## 5. Commit and pull request

- `git status` **before** `git add -A`, never after. Never commit work you did not do;
  pre-existing changes stay untouched unless the task is about them.
- Follow the repository's flow: if it uses branches and PRs (`claude/<task>` → PR → CI
  auto-merge), never commit to `main`, push only a green local gate, and wait for the merge
  before starting the next task. If it commits locally, commit each finished task
  automatically, without being asked. `git push` to a shared branch is Kade's unless the
  repository's flow says otherwise.
- **Subject: the finding or the point of the change**, not the files touched.
  Good: `A shared log file lost 1780 of 4000 lines under rotation; one file per pod lost none`.
  Bad: `update values.yaml`, `fix bug`.
- **Body:** what changed, with numbers; why, and which earlier belief it corrects; what is
  verified and what is still open.
- **PR description:** *What changed and why*, *Verified*, *NOT VERIFIED*, *Wrong turns*
  (approaches tried and abandoned). The PR is the session record; there are no separate
  session notes unless the repository still keeps them.

## 6. Record state where the repository keeps it

- **Org Project repositories** (Stella Rain): issues plus `cmd:` labels
  (`cmd:status-now`, `cmd:verify-needs-kade`, `cmd:verify-not-verified`), never a state file.
- **`MEMORY.md` repositories**: update its *Now* section, one line per item, decisions by ID
  only. Detail goes where that repository's `CLAUDE.md` says.
- State changes go in the same branch or commit as the work they describe.

## 7. Context budget

Quality drops as the context window fills, and auto-compaction replaces history with a summary.

- When a `context-guard` message appears (repositories that vendor the hook), or right after a
  compaction, tell Kade the number in one line and offer three choices:
  1. **handoff and a new session** (the `session-handoff` skill),
  2. `/compact` with a focus he names,
  3. continue.
- At **80%** or more, recommend the handoff and do not start new multi-step work until he answers.
- In sessions without the hook, offer the same choices after a compaction, or when the session
  has run many large tool outputs or several completed tasks.
- Finish or park the current step before a handoff; never hand off mid-edit.
