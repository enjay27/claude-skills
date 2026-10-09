# claude-skills

Kade's account skills and the tests that check them. Everything that applies to every
repository (global rules, the `context-guard` hook, the setup plans) lives in
[`enjay27/claude-global`](https://github.com/enjay27/claude-global).

| Path | What | How it is used |
|---|---|---|
| `skills/kade-workflow/` | How to work in any of Kade's repositories: plan, test, gates, commit and PR style, state, context budget | Saved as a claude.ai account skill |
| `skills/session-resume/` | Start a session from the recorded state | Account skill |
| `skills/session-handoff/` | Close a session: park the task, record state, print a starter prompt | Account skill |
| `scripts/` | Scenario harness and fixtures for testing the skills | Run by the session changing a skill |
| `docs/skill-*.md`, `docs/skills-improvement-plan.md` | Scenarios, their results, and the improvement plan | Read by the session changing a skill |

Account skills apply in Claude Code (terminal and cloud) and Cowork sessions. Each repository's
`CLAUDE.md` keeps one line, *"Follow the `kade-workflow` skill"*, and its own gates, paths and
branch rules; the skills never hold repository-specific facts.

## Changing a skill

Edit `skills/<name>/SKILL.md` here, commit, then ask Claude to propose the updated skill and
save it from the card. The file here is the source; the account copy follows it.
