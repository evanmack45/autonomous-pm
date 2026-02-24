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

## Dependencies

- superpowers plugin (brainstorming, writing-plans, TDD, systematic-debugging, verification-before-completion, receiving-code-review)
- GitHub CLI (`gh`) for issues, PRs, and Copilot review API
- Git worktrees for isolated implementation
