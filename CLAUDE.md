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

This plugin is installed in three locations. A post-commit hook (`.git/hooks/post-commit`) automatically syncs all plugin files after every commit:

1. **Source repo** (you edit here) — `~/Projects/autonomous-pm/`
2. **Local marketplace** (synced by hook) — `~/.claude/plugins/local-marketplace/plugins/autopilot/`
3. **Plugin cache** (synced by hook) — `~/.claude/plugins/cache/local-plugins/autopilot/0.1.0/`

If the hook is missing (e.g., after a fresh clone), recreate it manually by following the post-commit hook setup steps in `docs/plans/2026-02-25-plugin-sync-review-patterns-scope-pushback.md`. Git hooks are not version-controlled and won't be present after `git clone`.

## Dependencies

- superpowers plugin (brainstorming, writing-plans, TDD, systematic-debugging, verification-before-completion, receiving-code-review)
- GitHub CLI (`gh`) for issues, PRs, and Copilot review API
- Git worktrees for isolated implementation
