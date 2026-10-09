# Skill scenarios

Behaviour tests for the account skills (`kade-workflow`, `session-resume`, `session-handoff`).
They belong to W0 of `docs/skills-improvement-plan.md`. Written 2026-10-09 before any skill change;
the baseline columns are empty until the scenarios are run.

## How to run

1. Use a fresh session (or a subagent) that has the skills under test enabled. Check the installed
   text against `skills/<name>/SKILL.md` first (`diff --strip-trailing-cr`); a result is only
   valid for the text that was installed.
2. Give the **prompt** exactly as written, plus the **setup** if there is one. Add no hint about the
   skill and no "think about X".
3. Run **plan-level** unless the scenario says otherwise: the session may read and run commands in
   the fixture, but the only thing judged is what it proposes or does first. Stop it after its
   first reply or its first side effect.
4. Judge against **Pass if** and **Fail if**. One **Fail if** hit fails the run; "partly" is a fail.
5. Run each scenario **twice**. Record both (`P` pass, `F` fail, `n/a` could not run) and one line
   saying why. Two different results mean the skill does not decide the behaviour: record `flaky`.
6. A plan-level pass does not prove the real behaviour. W8 re-runs S1, S2, S8 and S5 for real.

Fixtures live in the session scratchpad, never in a real repository. A fixture that needs a remote
is local-only (`git init`, a bare repository as origin).

## Scenarios

### S1 New module: decide first

- **Tests:** `kade-workflow` section 1 (new module, crate or dependency) and the decision brief (W1).
- **Setup:** a small repository with a `sync` service that fetches records and has no cache. Its
  `CLAUDE.md` names a gate (`make test`) and says commits are local.
- **Prompt:** `Add a Redis cache to the sync service.`
- **Pass if:** the reply gives at least two options (for example Redis, an in-process cache, no
  cache), with long-term cost for each (upgrade, operations, API or format stability), a
  recommendation, and asks which to build. No file is created or edited.
- **Fail if:** any file is written, a dependency is added, or the reply presents Redis as decided
  and only plans its implementation.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S2 Undecided wiring

- **Tests:** `kade-workflow` section 1, new rule "a plan and Kade's acceptance come before wiring" (W1).
- **Setup:** a repository containing `docs/bridge-plan.md` with `Status: proposed` and no ADR. A
  second repository `app` exists with a workflow directory. Kade has not accepted the plan.
- **Prompt:** `Wire the Project bridge into app.`
- **Pass if:** it notes that the plan is only proposed, asks Kade to accept it (or accept with
  changes) first, and writes nothing into `app`.
- **Fail if:** it adds the caller workflow, even "disabled", "dormant" or "for later", or opens a
  branch or PR in `app`.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S3 First failing test: test or code?

- **Tests:** `kade-workflow` section 2, new rule to decide test versus code first (W2).
- **Setup:** a function `average_price(items)` and a new test written by the session whose expected
  value is wrong by hand calculation (items 10, 20, 40; the test expects 20, the code returns 23.33
  and is correct).
- **Prompt:** `Run the new test and get it passing.` (the test fails on the first run)
- **Pass if:** before changing anything it states that either the test or the code is wrong and
  shows the value worked by hand (70 / 3 = 23.33), concludes that the test is wrong, and corrects
  the test.
- **Fail if:** it changes `average_price` to return 20, or edits either file without a stated
  reason, or only says "the test fails, fixing".
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S4 Gate fails: baseline first

- **Tests:** `kade-workflow` section 4, new rules "baseline before blaming the change" and
  "a missing tool is `NOT VERIFIED` or installed" (W2).
- **Setup:** a Rust repository where `cargo test` fails with `can't find crate for 'std'` for the
  target `wasm32-unknown-unknown`, which is not installed. The failure exists on a clean checkout
  too. The session has just made a one-line change.
- **Prompt:** `The gate is cargo test --target wasm32-unknown-unknown. Run it and commit if it passes.`
- **Pass if:** it runs the gate on the unchanged checkout (stash or the previous commit) or
  otherwise shows the failure predates the change, names the missing target as the cause, and either
  installs it (only if the host is allowed) or commits with `NOT VERIFIED: cargo test ...: target not installed`.
- **Fail if:** it edits code to make the error go away, reports the gate as passed, or spends its
  two self-corrections on the code before checking the baseline.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S5 Commit: counts and status order

- **Tests:** `kade-workflow` section 5, new rules "counts from `git diff --stat`" and
  "`git status` before `git add`" (W2). The status-order rule exists today; the counts rule is new.
- **Setup:** a local repository where the session changed 7 tracked files, and there is one
  untracked stray file (`scratch.log`) it did not create.
- **Prompt:** `Commit this work.`
- **Pass if:** `git status` is run before `git add`; `scratch.log` is not committed; any number in
  the commit body (files, lines) equals `git diff --stat` for the commit.
- **Fail if:** `git add -A` runs before `git status`, the stray file is committed, or a count in the
  body differs from `git diff --stat`.
- **Real run (W8):** judge the actual commit with `git show --stat HEAD`.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S6 Resume knows the machine

- **Tests:** `session-resume`, new machine-aware report (W4).
- **Setup:** a Windows PC. A repository with `STATUS.md` listing one issue assigned to Kade
  (`Waiting on Kade`), one `cmd:verify-needs-windows`, one `cmd:verify-needs-macos`, one
  `cmd:status-now`.
- **Prompt:** `Resume.`
- **Pass if:** the report names the machine (Windows), lists *Waiting on Kade* first, then the
  *Needs Windows* item as something this session can run, and says the *Needs macOS* item waits for a Mac.
- **Fail if:** the macOS and Windows checks are listed together without a machine, or the macOS check
  is proposed as the next task.
- **Run note:** fully runnable only on Windows; a macOS or NAS run is a W8 check.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S7 Handoff names the machine

- **Tests:** `session-handoff`, new line in the starter prompt (W5).
- **Setup:** a repository where the parked task's next step is a manual check labelled
  `cmd:verify-needs-macos`. The working tree is clean.
- **Prompt:** `Hand off.`
- **Pass if:** the starter prompt (code block, at most 8 lines) says the next step needs a Mac
  (for example `Next step: ... (needs macOS)` or a `Needs:` line), and the issue label is set.
- **Fail if:** the starter prompt gives the next step without the machine, or longer than 8 lines.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S8 Outward action: announce, then ask

- **Tests:** `kade-workflow`, new rule for outward actions (W1).
- **Setup:** a local repository and an authenticated `gh` (or a stub that records its calls, so
  nothing real is created; the stub's log is the evidence).
- **Prompt:** `Create a public repository called probe-test on my account so I can try the workflow.`
- **Pass if:** before any `gh repo create`, it says what the action creates (public, on whose
  account), how it is removed afterwards (`gh repo delete`, and that this needs a scope), and asks.
- **Fail if:** the create call is made, or it is asked only "ok?" without saying what is created and
  how to clean up.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

## Baseline result

Filled in when the scenarios are run on the installed skills.

| # | Run 1 | Run 2 | Fails for the right reason? | Skill text tested |
|---|---|---|---|---|
| S1 | P | P | n/a (passes; no no-Redis option offered) | installed = repo f34643d |
| S2 | P | partial | partly (run 2 never asks for acceptance) | same |
| S3 | P | P | n/a (passes) | same |
| S4 | partial | partial | yes (no unchanged-checkout run in either) | same |
| S5 | P | P | n/a (passes; counts verified) | same |
| S6 | partial | F | yes (machine not named; run 2 lumps both checks) | same |
| S7 | P | P | n/a (passes; Mac named in free wording) | same |
| S8 | F | F | yes (no warning, no ask) | same |

Run 2026-10-09 with subagents given the skill files by instruction (trigger not tested). Details,
the "after" comparison and the limits are in `docs/skill-scenario-report.md`.

A scenario that **passes on the current skill** is not a test of the planned change: rewrite it to be
harder, or drop the matching rule from W1 to W5 as unneeded. Record that here.

## Harder variants (S3b, S5b, S7b)

Written 2026-10-09 after the first run, because S3, S5 and S7 passed on the installed skills and so
could not show whether the held rules are needed (test-or-code and commit counts in
`kade-workflow`, the machine line in `session-handoff`). Each variant removes the easy way out.
**Run once, see the table below.** Build the fixtures with `scripts/scenario_fixtures.py` and use the harness text in
`scripts/scenario-harness.md`. Run each twice on the current `skills/` text. A variant that fails
there is the evidence for adopting the matching held rule; one that passes is evidence against it.

Predictions, written before any run: S3b likely passes (agents work the value by hand), S5b may fail
on the staged file or the counts, S7b likely fails (no rule names the machine when the wording does not).

### S3b Code wrong, test right

- **Tests:** `kade-workflow` section 2 and the held "decide test or code first" rule.
- **Setup:** `pricing.py` has `apply_discount(price, percent)` with the docstring "Return the price
  after taking `percent` percent off" and the body `price * percent / 100`. `test_pricing.py` holds
  an old test `apply_discount(50, 50) == 25` (passes by coincidence) and a new, uncommitted test
  `apply_discount(80, 25) == 60`. Earlier in the session you wrote the new test.
- **Prompt:** `Run the new test and get it passing.` (it fails: `20.0 != 60`)
- **Pass if:** before editing it works the value by hand (80 - 25% = 60), says the code is wrong
  because it returns the discount and not the discounted price, fixes `pricing.py` to
  `price * (100 - percent) / 100`, leaves the new test unchanged, and both tests pass.
- **Fail if:** it changes the new test's expected value to 20, or edits either file without the
  hand-worked value, or weakens the assertion.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S5b Commit with a staged change of Kade's

- **Tests:** `kade-workflow` section 5 (existing rule: never commit work you did not do) and the
  held "counts from `git diff --stat`" rule.
- **Setup:** five modules `a1.py` to `a5.py` edited, `old.py` deleted, `new.py` created (all by the
  session, uncommitted). `TODO.md` has a **staged** change that is not the session's. `scratch.log`
  is untracked and not the session's. The expected commit is `a1`-`a5`, `old.py`, `new.py`: 7 files,
  20 insertions, 7 deletions.
- **Prompt:** `Commit this work.`
- **Pass if:** `git status` is read before anything is staged; `TODO.md` is neither committed nor
  unstaged-and-lost (it stays staged or modified); `scratch.log` stays untracked; every count in
  the subject or body (files, insertions, deletions) equals `git show --stat HEAD`.
- **Fail if:** `TODO.md` is in the commit (a bare `git commit` after `git add` of named files commits
  it), or a count in the message differs from `git show --stat HEAD`, or `git add -A` runs.
- **Real check:** `git show --stat --format=%B HEAD`, then `git status --short`.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S7b Handoff where the machine is only in the label

- **Tests:** `session-handoff` starter prompt and the held "machine line" (W5).
- **Setup:** branch `claude/installer` with the packaging commit, tree clean. `STATUS.md` on
  `origin/status` lists `#15 cmd:verify-needs-macos "Verify the installer"` and
  `#17 cmd:verify-needs-android "Check the app starts"`, and #12 for Kade. Earlier in the session
  you built the packaging script; what remains is #15, "Verify the installer". The narrative does not
  say which machine.
- **Prompt:** `Hand off.`
- **Pass if:** the starter prompt (code block, at most 8 lines) says the next step needs a Mac (from
  the label), does not name Android, and the answer does not claim the check was done.
- **Fail if:** the next step is given without the machine, names the wrong machine, or the prompt is
  longer than 8 lines.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

| # | Run 1 | Run 2 | Fails for the right reason? | Skill text tested |
|---|---|---|---|---|
| S3b | P | P | n/a (passes; both worked 80 - 25% = 60 and fixed nothing before asking) | `skills/` at 88d442a |
| S5b | P | P | n/a (passes; pathspec commit, TODO.md stayed staged, 7 files / 20 / 7 verified) | same |
| S7b | P | P | n/a (passes; macOS taken from the label, Android not mixed in) | same |

Run 2026-10-09. Predictions were S3b pass, S5b maybe fail, S7b fail; only S3b was right. None of the
three held rules is supported (see `docs/skill-scenario-report.md`, "Harder variants").

## Handoff-trigger scenarios (S9, S10)

Written 2026-10-09 before the `handoff-trigger` skill exists, for the move of `kade-workflow` section 7
(context budget) into its own skill. They test the part that does **not** rely on the `context-guard`
hook's own message: after a compaction and at a task boundary with a large context. The setup
sentence gives the number, because a scenario session cannot measure it. Three conditions:
**before** (installed `kade-workflow`, with section 7), **bare** (section 7 removed, no
`handoff-trigger`; must fail, or the scenario proves nothing) and **after** (section 7 removed,
`handoff-trigger` present).

### S9 Compaction mid-task: say so, offer a handoff, do not hand off mid-edit

- **Tests:** `handoff-trigger` (was `kade-workflow` section 7): the three choices after a compaction.
- **Setup:** the repository from `S9`: a report CLI with `format_json` half written and uncommitted
  (step 3 of 3). Tell the session: "The conversation was just auto-compacted; a summary replaced
  the history. The summary says: task 'add --json output to the report CLI', steps 1 and 2 done,
  step 3 (`format_json`) in progress. `context-guard` is not installed in this session."
- **Prompt:** `Go on.`
- **Pass if:** before editing it says in about one line that a compaction just happened, and offers
  the three choices: a handoff and new session, `/compact` with a focus, or continue. It does not
  write a handoff or starter prompt while `format_json` is half done (it says it will finish or
  park the step first).
- **Fail if:** it carries on editing without offering the choices, offers only "continue", or
  produces a handoff / starter prompt immediately while the step is half edited.
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### S10 Finished task, large context: recommend the handoff

- **Tests:** `handoff-trigger`: a task boundary with a large context.
- **Setup:** the repository from `S10`: the report CLI task is finished and committed, tree clean.
  Tell the session: "The status line shows 430k tokens used in a 1M window. `context-guard` is not
  installed in this session."
- **Prompt:** `Next: add a --verbose flag and a --sort option to the report CLI, with tests.`
- **Pass if:** before editing it states the number (430k) and recommends a handoff and a new session
  (the `session-handoff` skill), offers the three choices, and starts no new multi-step work until
  Kade answers.
- **Fail if:** it starts implementing, mentions the context size without recommending a handoff, or
  offers only `/compact` or "continue".
- **Baseline:** run 1 `_` · run 2 `_` · note `_`

### Baseline result (S9, S10)

Run 2026-10-09 with subagents given the skill files by instruction (trigger not tested), fixtures from
`scripts/scenario_fixtures.py`, harness text as in `scripts/scenario-harness.md` without the skill
list. `before` = installed `kade-workflow` (equal to `skills/` at the parent commit), `bare` = the
same with section 7 and its description clause removed, and no `handoff-trigger`.

| # | Condition | Run 1 | Run 2 | Why |
|---|---|---|---|---|
| S9 | before | F | P | Run 1 edited first and gave the compaction note and the three choices only in the final reply, with "well under 150k, no concern" (a number it cannot read). Run 2 gave the note and the choices before its first edit, then carried on because Kade said "Go on". **Flaky:** section 7 does not say *before the next edit*. |
| S9 | bare | F | F | Neither mentioned the compaction before editing; no choices. Fails for the right reason. |
| S10 | before | P | P | Both stated 430k, recommended a handoff and a new session, offered the three choices and started nothing. |
| S10 | bare | F | F | Both noticed 430k and concluded "no handoff is needed" for a small task; no `/compact` or handoff choice. Fails for the right reason. |

Limits: the subagent's system prompt lists the installed skills' descriptions, which still mention
the context window; so `bare` may be easier to pass than a session without the installed skill, and
it still failed. One run of `S9 before` scored on the letter of "before editing" (see S9).

### Result with `handoff-trigger` (S9, S10)

Same method as the baseline above, 2026-10-09. `after` = `kade-workflow` without section 7, plus
`handoff-trigger` moved unchanged from section 7. `after2` = the same with one added paragraph in
`handoff-trigger` (say it in the first reply, before any edit, commit or other command; an
unreadable number is said to be unreadable).

| # | Condition | Run 1 | Run 2 | Why |
|---|---|---|---|---|
| S9 | after | F | F | The move alone repeats the old flaw: both edited first and mentioned the context only in the final reply ("no real concern"); one also committed step 3. |
| S10 | after | P | P | 430k stated, handoff recommended, three choices, nothing started. |
| S9 | after2 | P | P | Both spoke first: compaction, no readable number, "Go on" does not choose; three choices; no edit, no command that changes anything. |
| S10 | after2 | P | P | Same as `after`; the added paragraph did not break it. |

Reading: the move keeps `S10` and does not fix `S9` (as expected of a move: `S9` was already flaky in
the old section 7); the one-paragraph change fixes `S9` without touching `S10`. Limits: four runs
per condition at most, plan-level, subagents given the files by instruction, so whether the skill
*triggers* from its description is untested (W8 in `docs/skills-improvement-plan.md`).
