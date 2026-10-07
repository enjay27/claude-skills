# claude-skills

Kade's shared Claude configuration: the account skills, the hooks every repository vendors,
and the plan that moves each repository onto them. Private.

| Path | What | How it is used |
|---|---|---|
| `skills/kade-workflow/` | How to work in any of Kade's repositories: plan, test, gates, commit and PR style, state, context budget | Saved as a claude.ai account skill |
| `skills/session-resume/` | Start a session from the recorded state | Account skill |
| `skills/session-handoff/` | Close a session: park the task, record state, print a starter prompt | Account skill |
| `hooks/context-guard.cjs` | Warns at 200k tokens of context, recommends a handoff at 400k (never later than 40% / 60% of the window), and after a compaction | Copied into each repository's `.claude/hooks/`; merge `hooks/settings-snippet.json` into its `.claude/settings.json` |
| `docs/refactor-plan.md` | Moving `CLAUDE.md`, `MEMORY.md`, skills and rules to this setup, per repository | Read by the session doing the work |

Account skills apply in Claude Code (terminal and cloud) and Cowork sessions. Each repository's
`CLAUDE.md` keeps one line, *"Follow the `kade-workflow` skill"*, and its own gates, paths and
branch rules; the skills never hold repository-specific facts.

## Changing a skill

Edit `skills/<name>/SKILL.md` here, commit, then ask Claude to propose the updated skill and
save it from the card. The file here is the source; the account copy follows it.

## Tests

```bash
node --test hooks/context-guard.test.cjs
```

## Installing the hook in a repository

```bash
mkdir -p .claude/hooks
cp ~/claude-skills/hooks/context-guard.cjs .claude/hooks/
# merge hooks/settings-snippet.json into .claude/settings.json (keep existing hooks such as graft)
```

The hook assumes a 1M window (the default of Opus 4.7+, Sonnet 5+ and the Fable models). Set
`CLAUDE_CONTEXT_WINDOW=200000` for a 200k-window model; the lines then fall to 80k and 120k.
