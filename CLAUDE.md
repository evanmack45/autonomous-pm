# Autopilot

Claude Code plugin for autonomous project management with Copilot-driven code review.

## Structure

- `commands/autopilot.md` — Entry point slash command
- `skills/pm-workflow/SKILL.md` — Core workflow: assessment, planning, implementation, quality gate, PR creation, CI verification, Copilot review loop, wrap-up
- `agents/implementer.md` — Subagent dispatched for implementation tasks and review fixes
- `hooks/` — Session-start hook that reminds users about `/autopilot`

## Runtime State

The PM creates a `.autopilot/` directory in target repos to store:

- `runs/` — Timestamped audit logs per run
- `review-patterns.md` — Accumulated Copilot review patterns across PRs
- `checkpoint.json` — Pipeline state for cross-session recovery

## Plugin Installation

This plugin is installed in three locations. ALL THREE must be updated when files change:

1. **Source repo** — `/Users/evanmcmillan/Projects/autonomous-pm/`
2. **Local marketplace** — `~/.claude/plugins/local-marketplace/plugins/autopilot/`
3. **Plugin cache** — `~/.claude/plugins/cache/local-plugins/autopilot/0.1.0/`

After editing any file in the source repo, copy changed files to both other locations. Claude Code loads from the cache, not the source repo.

## Dependencies

- superpowers plugin (brainstorming, writing-plans, TDD, systematic-debugging, verification-before-completion, receiving-code-review)
- GitHub CLI (`gh`) for issues, PRs, and Copilot review API
- Git worktrees for isolated implementation
