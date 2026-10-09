# Plan: one global rule set, plus rules per repository

- **Scope:** where Kade's coding and workflow conventions live, how they reach local and cloud
  sessions, and what each repository keeps for itself.
- **Status:** proposed 2026-10-09. Nothing is changed yet; every step below waits for Kade's yes.
- **Related:** `docs/refactor-plan.md` (the current setup), `docs/skills-improvement-plan.md`
  (what the account skills should say).
- **Decision wanted:** whether to adopt option B (below), and the pin policy in section 5.

## 1. The question

Kade wants one place for conventions that apply to every repository (coding and workflow), and
repository files that hold only what is specific to that repository: which file to open first,
which directory a new kind of file goes in, repo-only hard rules.

The constraint that shapes the answer: **a cloud session cannot read the home folder of Kade's
PC or Mac**, so a user-level `~/.claude/CLAUDE.md` alone reaches only local sessions.

## 2. The current setup

| Piece | Carries | Reaches |
|---|---|---|
| Account skills (`kade-workflow`, `session-resume`, `session-handoff`, `graft-kade`) | Workflow procedures, loaded when their description matches | Local and cloud |
| Each repository's `CLAUDE.md` | Repo facts, gates, one line "follow `kade-workflow`" | That repository |
| `.claude/rules/*.md` per repository | Path-scoped rules, loaded when a matching file is read | That repository |
| `.claude/hooks/context-guard.cjs`, vendored copy | Context warnings | That repository |

Gaps: no always-on global text (a skill that does not trigger is a rule that is missed); the hook
is copied by hand into each repository, so copies can drift; nothing checks that the account copy
of a skill equals the file in this repository.

## 3. The options

| | A. Today | B. Clone this repository at cloud-session start | C. Plugin from a git marketplace | D. CI opens sync PRs into every repository |
|---|---|---|---|---|
| Always-on global text | No | Yes, in `~/.claude/CLAUDE.md` and `~/.claude/rules/` | *Unverified*: believed not to ship `CLAUDE.md` text | Yes, as a file in each repository |
| Skills and hook | Account skills; hook copied | Skills stay in the account; hook copied or read from the clone | Both, installed from the marketplace | Copies in each repository |
| Cloud and local identical | Partly | Yes, same source on PC, Mac and cloud | Yes for skills and hook | Yes |
| Update path | Upload skills by hand; edit hook copies | Edit once here, bump the pinned tag | Edit once here, bump the plugin version | Edit once here, merge one PR per repository |
| Context cost | Low | Higher: the text is in every turn, so keep it short | Low (skills load on demand) | Higher, same as B |
| New moving parts | None | Credential for a private repository in the cloud environment; network allowlist; a tag to maintain | Marketplace entry in each repository's `settings.json` | A sync workflow and a token that can open PRs |
| Failure mode | Silent drift | Failed clone leaves sessions without the rules (use `set -e` so it is loud); stale tag | Plugin not installed in a session | PR left unmerged, so a repository lags |
| Trust | Uploaded skills and repo files | Everything in this repository is trusted instructions for every session | Same | Same, but each change is reviewed per repository |
| Review of a change | Per repository | One place, but a bad change reaches all repositories at once | Same as B | Per repository |

Rejected: a git submodule in each repository (friction on both machines and in cloud checkouts for
no gain over B or D).

## 4. Recommendation

Adopt **B** for the short, always-on text only; keep everything else where it is.

- `global/CLAUDE.md` (here): the few rules that must always apply: coding conventions, the order
  plan, test, gate, commit; where state is recorded; the Windows shell traps. Target: under 60
  lines. Long procedures stay in the account skills, which load on demand.
- `global/rules/*.md` (here): path-scoped global rules only if one is needed.
- Each repository's `CLAUDE.md`: repo facts only. Where it and the global text differ, the
  repository wins (already stated in the current files).
- Hook: keep the vendored copy and add a CI check that fails when it differs from
  `hooks/context-guard.cjs` ignoring line endings. Move it to option C later if the copies hurt.

Why not C first: it does not solve always-on text, which is the actual gap. Why not D first: it
makes every global edit a merge in every repository, and the copies still exist.

## 5. Steps

1. **Draft `global/`** here: `CLAUDE.md`, and a note on what each line replaced. Move rules out of
   repository `CLAUDE.md` files only when they are global; list the candidates first and let Kade
   confirm each.
2. **Install script for the PC and Mac** (`scripts/install-global.*`): copy `global/` to
   `~/.claude/`; on Windows run from PowerShell or Git Bash; never overwrite a user file without
   saying so. Test on the PC, then the Mac.
3. **Cloud setup script**, set in the cloud environment (*unverified*: where the setting sits in
   the app):

   ```bash
   set -e
   d="$HOME/.claude-global"
   rm -rf "$d"
   git clone --depth 1 --branch <tag> https://github.com/enjay27/claude-skills "$d"
   mkdir -p "$HOME/.claude/rules"
   cp "$d/global/CLAUDE.md" "$HOME/.claude/CLAUDE.md"
   cp -r "$d/global/rules/." "$HOME/.claude/rules/"
   ```

   Pin `<tag>`, not `main`, and bump it on purpose.
4. **Drift checks:** a test here that the account skill files and `global/` match what was
   uploaded (by hash in the README), and the hook copy check from section 4 in each repository.
5. **Cut the duplicates** from repository `CLAUDE.md` files once the global text is verified in a
   live cloud session. One pull request per repository.

## 6. Open questions (check before step 3)

| Question | How to check |
|---|---|
| Can a `git clone` of this private repository run inside the cloud setup script? Access was granted to the Claude app, but that may not cover a clone by script | Run the script once in a throwaway environment. If it fails, use a read-only token stored as an environment secret |
| Does the cloud environment allow `github.com` in its network settings? | Same run |
| Is `~/.claude/CLAUDE.md` read in a cloud session started after the script? | Ask the session "which instructions are loaded?" |
| Does a repository file load the same way on Windows when `CLAUDE_PROJECT_DIR` is unset? | The context-guard live check in `stella-rain/app#5` covers this |

## 7. How it is tested

Same bar as the skills: write the checks first.

- A fresh cloud session on `app` quotes one rule from `global/CLAUDE.md` when asked which rules are
  loaded (fails today).
- A fresh local session on the PC and one on the Mac do the same.
- A deliberately broken clone (wrong tag) makes the setup script fail loudly.
- After step 5, the same session still works with the duplicates removed.

## 8. Risks

- **One bad edit reaches every repository.** Mitigation: protect `main` here, require a PR, pin by
  tag.
- **Always-on text costs context every turn.** Mitigation: the 60-line target and a review of each
  line against "would a missed rule cost a failed command or a wrong PR".
- **The cloud clone becomes a hidden dependency.** Mitigation: the loud failure in step 3 and the
  open-questions table above.
