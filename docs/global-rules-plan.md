# Plan: one global rule set, plus rules per repository

- **Scope:** where Kade's coding and workflow conventions live, how they reach local and cloud
  sessions, and what each repository keeps for itself.
- **Status:** proposed 2026-10-09, revised the same day after checking the Claude Code docs
  (section 9). Nothing is changed yet; every step below waits for Kade's yes.
- **Related:** `docs/refactor-plan.md` (the current setup), `docs/skills-improvement-plan.md`
  (what the account skills should say).
- **Decision wanted:** whether to adopt option B (below), the pin policy in section 5, and whether
  the clone also carries the hook and the skills (step 4).

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
| Always-on global text | No | Yes, in `~/.claude/CLAUDE.md` and `~/.claude/rules/` | Locally only, through a plugin `SessionStart` hook (plugins cannot ship `CLAUDE.md`). **Cloud sessions do not install plugins a repository enables** | Yes, as a file in each repository |
| Skills and hook | Account skills; hook copied | From the same clone, into `~/.claude/skills/` and `~/.claude/hooks/` (step 4), or as today | Both, but again not in the cloud | Copies in each repository |
| Cloud and local identical | Partly | Yes, same source on PC, Mac and cloud | No | Yes |
| Update path | Upload skills by hand; edit hook copies | Edit once here, bump the pinned tag in the setup script | Edit once here, bump the plugin version | Edit once here, merge one PR per repository |
| Context cost | Low | Higher: the text is in every turn, so keep it short | Low (skills load on demand) | Higher, same as B |
| New moving parts | None | A read-only token as an environment variable; network allowlist; a tag to maintain | Marketplace entry in each repository's `settings.json` | A sync workflow and a token that can open PRs |
| Failure mode | Silent drift | Failed clone leaves sessions without the rules (made loud, step 3); stale tag | Plugin not installed in a session | PR left unmerged, so a repository lags |
| Trust | Uploaded skills and repo files | Everything in this repository is trusted instructions for every session | Same | Same, but each change is reviewed per repository |
| Review of a change | Per repository | One place, but a bad change reaches all repositories at once | Same as B | Per repository |

Rejected:

- A git submodule in each repository: friction on both machines and in cloud checkouts for no gain
  over B or D.
- The global text pasted into the setup script, with no clone: no token needed, but the text then
  lives in the app's settings, where it cannot be diffed or reviewed, and drifts like the uploaded
  skills.
- A `SessionStart` hook committed to each repository that fetches the text: it does run in cloud
  sessions, but it is one copy per repository (the problem of D) and still needs the token.
- Managed policy `CLAUDE.md` or server-managed settings: the only documented route that reaches
  cloud sessions with plugins, but it is for organisations, not one person's account.

## 4. Recommendation

Adopt **B**. Start with the short, always-on text; let the same clone carry the hook and the skills
once step 4 shows the cloud honours them.

- `global/CLAUDE.md` (here): the few rules that must always apply: coding conventions, the order
  plan, test, gate, commit; where state is recorded; the Windows shell traps. Target: under 60
  lines. Long procedures stay in skills, which load on demand. The first line is a version stamp,
  `Global rules: v<tag>`, so a session can say which version it loaded.
- `global/rules/*.md` (here): path-scoped global rules only if one is needed (*unverified*: the
  docs show `paths:` for project rules, not explicitly for user-level ones).
- Each repository's `CLAUDE.md`: repo facts only, plus a short section **"Overrides of global
  rules"** that names each global rule it changes. Claude Code does not enforce "the repository
  wins"; both files are simply in context, so an unstated conflict is a guess.
- Hook and skills: today's setup until step 4. If step 4 passes, the vendored hook copies and the
  hand uploads go away, and with them the drift checks for both.

Why not C: cloud sessions do not install plugins a repository turns on, so C fails the one
constraint that matters (section 1). Why not D: it makes every global edit a merge in every
repository, and the copies still exist.

## 5. Steps

1. **Prove the cloud clone** (was step 3; every other step depends on it). In a throwaway cloud
   environment:
   - add `GH_TOKEN`: a fine-grained token, read-only, for `enjay27/claude-skills` only, with an
     expiry. The setup script runs before the session connects to the GitHub proxy, so the access
     granted to the Claude app is not expected to cover a clone from the script. Every session in
     the environment can read the token, so it must grant nothing else;
   - allow `github.com` in the network settings;
   - run the script below with a test `global/CLAUDE.md` and check `/context` lists
     `/root/.claude/CLAUDE.md`.

   ```bash
   tag=v2026.10.09   # bump on purpose; see the pin policy below
   d="$HOME/.claude-global"
   mkdir -p "$HOME/.claude/rules"
   rm -rf "$d"
   if git clone --depth 1 --branch "$tag" \
        "https://x-access-token:${GH_TOKEN}@github.com/enjay27/claude-skills" "$d"; then
     cp "$d/global/CLAUDE.md" "$HOME/.claude/CLAUDE.md"
     cp -r "$d/global/rules/." "$HOME/.claude/rules/" 2>/dev/null || true
     rm -rf "$d"   # the token is in the clone's .git/config; keep only the copied files
   else
     echo "GLOBAL RULES FAILED TO LOAD ($tag). Tell Kade before doing anything else." \
       > "$HOME/.claude/CLAUDE.md"
   fi
   exit 0
   ```

   The script fails soft on purpose: with `set -e`, a GitHub outage would stop every session in the
   environment from starting. The fallback file makes the failure the first thing the session says.

   **Pin policy:** pin a tag, never `main`. The cloud caches the result of the setup script and
   reruns it only when the script or the allowed hosts change, or after about seven days. A
   moving branch would therefore lag by up to a week without a sign; bumping the tag edits the
   script, which forces the rerun. The tag bump is the update.
2. **Draft `global/`** here: `CLAUDE.md`, and a note on what each line replaced. Move rules out of
   repository `CLAUDE.md` files only when they are global; list the candidates first and let Kade
   confirm each.
3. **Local setup on the PC and Mac:** no copy. `~/.claude/CLAUDE.md` holds one import line,
   `@~/<path to this clone>/global/CLAUDE.md` (the docs confirm `~` in imports), plus any personal
   lines. `git pull` here updates it; nothing is overwritten. Same for `~/.claude/rules/`
   (symlinks, or one import per file on Windows). Test on the PC, then the Mac.
4. **Hook and skills from the clone** (decision for Kade). Extend the setup script, and the local
   setup, to install `hooks/context-guard.cjs` into `~/.claude/hooks/`, register it in
   `~/.claude/settings.json`, and copy `skills/*` into `~/.claude/skills/`. Check first, in the
   same throwaway environment:
   - that a user-level `settings.json` written by the setup script is honoured in a cloud session
     (*unverified*);
   - that user-level skills load in the cloud, and how a name clash with the account skills of the
     same name resolves; rename or remove the account copies if needed.

   If both pass, delete the vendored hooks (one PR per repository) and stop uploading skills for
   Claude Code; the account copies remain only for claude.ai chat and Cowork, if those still use
   them. If either fails, keep today's hook and skills and do step 5.
5. **Drift checks, only for what step 4 did not remove:** a test here that the account skill files
   match what was uploaded (by hash in the README); for the hook, each repository stores the
   expected hash next to its copy and a test here prints the current one. A cross-repository CI
   comparison would need a token to read this private repository in every repository's CI.
6. **Cut the duplicates** from repository `CLAUDE.md` files once the global text is verified in a
   live cloud session, and add the "Overrides of global rules" section where needed. One pull
   request per repository.

## 6. Open questions

| Question | How to check | Status |
|---|---|---|
| Can the setup script clone this private repository? | Step 1 | Docs: the GitHub proxy is connected only after the script; use `GH_TOKEN` |
| Does the cloud environment allow `github.com` in its network settings? | Step 1 | Open |
| Is `~/.claude/CLAUDE.md` read in a cloud session started after the script? | `/context` in step 1 | Docs: yes, an example in the cloud environment docs does exactly this |
| Are user-level `settings.json` hooks and `~/.claude/skills/` honoured in the cloud? | Step 4 | Open |
| What happens when an imported file (step 3) is missing? | Rename the target and ask for the version stamp | Open; the docs do not say |
| Do `paths:` rules work at user level? | One test rule in `global/rules/` | Open |
| Does a repository file load the same way on Windows when `CLAUDE_PROJECT_DIR` is unset? | The context-guard live check in `stella-rain/app#5` | Open |

## 7. How it is tested

Same bar as the skills: write the checks first. Each check asks the session to quote the version
stamp, `Global rules: v<tag>`, so the answer is exact and also shows a stale cache.

- A fresh cloud session on `app` quotes the stamp of the current tag (fails today).
- A fresh local session on the PC and one on the Mac do the same.
- A deliberately broken clone (wrong tag) starts the session and makes it report the
  "GLOBAL RULES FAILED TO LOAD" line first.
- A local session with the import target renamed shows what a missing import does (section 6).
- After step 6, the same session still works with the duplicates removed.

## 8. Risks

- **One bad edit reaches every repository.** Mitigation: protect `main` here, require a PR, pin by
  tag.
- **Always-on text costs context every turn.** Mitigation: the 60-line target and a review of each
  line against "would a missed rule cost a failed command or a wrong PR".
- **The cloud clone becomes a hidden dependency.** Mitigation: the fail-soft fallback in step 1 and
  the open-questions table above.
- **The token is readable in every session of the environment.** Mitigation: fine-grained,
  read-only, this repository only, with an expiry; a calendar note to renew it before it lapses,
  since an expired token shows up as the fallback line, not as an error.
- **A stale cache serves old rules.** Mitigation: the tag pin (a bump reruns the script) and the
  version stamp, which makes the loaded version visible.

## 9. Sources (Claude Code docs, read 2026-10-09)

| Fact used above | Page |
|---|---|
| Plugins cannot ship `CLAUDE.md`; a plugin `SessionStart` hook can add text to context and refires on `compact` | `code.claude.com/docs/en/plugins/components`, `/hooks` |
| Cloud sessions do not install plugins a repository enables, nor its extra marketplaces | `/cloud-environments` ("What carries over"), `/plugins/loading` |
| Setup script runs before Claude Code launches; writing `~/.claude/CLAUDE.md` there loads it; check with `/context` | `/cloud-environments` ("Setup scripts") |
| Setup script is cached; reruns when the script or allowed hosts change, or after about seven days; must exit 0 | `/cloud-environments` |
| The agent proxy connects after the setup script; `GH_TOKEN` or `GITHUB_TOKEN` as an environment variable for a private clone | `/cloud-environments` |
| `~/.claude/rules/*.md` exists and loads before project rules; `@~/…` imports are allowed | `/memory` |
