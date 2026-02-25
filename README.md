# Autopilot

A Claude Code plugin that autonomously manages your project. Run `/autopilot` in any repo and the PM takes over — it assesses the codebase, picks the highest-priority work, plans it, implements via subagents, creates a PR, and iterates through GitHub Copilot code reviews until the PR is clean. In pipeline mode, it continues to the next issue automatically.

## What It Does

1. **Assesses** the repo — reads docs, checks issues, runs tests, discovers CI workflows, understands the current state
2. **Picks** the highest-priority task and creates GitHub issues to track it
3. **Plans** the work using brainstorming and planning skills scaled to complexity, informed by patterns from past reviews
4. **Implements** by dispatching subagents in isolated git worktrees — independent subtasks run in parallel
5. **Validates** against acceptance criteria, runs tests, checks scope, updates docs
6. **Opens a PR** linked to the issue with full context, adds a basic CI workflow if the repo has none
7. **Iterates** through Copilot code reviews and CI checks — fixes feedback, pushes back when warranted, resolves threads, records review patterns for future runs
8. **Merges** the PR when Copilot review is clean and CI is green
9. **Continues** to the next issue (pipeline mode) or stops after the configured limit

All work is tracked through GitHub Issues. Subtasks get their own issues linked to the parent. PRs auto-close issues on merge.

## Requirements

- [Claude Code](https://claude.com/claude-code) with the [superpowers plugin](https://github.com/anthropics/claude-code-plugins)
- [GitHub CLI](https://cli.github.com/) (`gh`), authenticated
- [GitHub Copilot](https://github.com/features/copilot) enabled on the repo for code review
- A GitHub remote on the repository

## Install

```bash
claude plugins add /path/to/autonomous-pm
```

## Usage

```bash
cd your-project
claude
# then type: /autopilot
```

## Configuration

The PM writes an `## Autopilot PM` section to the target repo's CLAUDE.md on first run. This section includes optional configuration:

- **Approval Gates** — pause at specific points (after assessment, after planning, before merge) to require human approval. Default: fully autonomous.
- **Pipeline** — set `max-issues` to control how many issues the PM resolves per session. Default: 3.

## Plugin Structure

```
.claude-plugin/plugin.json   # Plugin metadata
commands/autopilot.md         # /autopilot slash command
skills/pm-workflow/SKILL.md   # Core PM workflow (7 phases)
agents/implementer.md         # Subagent for coding tasks and review fixes
hooks/                        # Session-start hook (reminder to run /autopilot)
```

## Runtime State

The PM stores runtime state in `.autopilot/` in the target repo:

- `runs/` — Timestamped audit trail for each run (structured log of decisions, API results, and outcomes per phase)
- `review-patterns.md` — Accumulated review feedback patterns across PRs (used to inform future implementations)
- `checkpoint.json` — Pipeline checkpoint for recovery across sessions

## Safety

- Never pushes directly to main — always uses feature branches
- Never force pushes
- Never skips tests
- Stops the review loop and escalates when a fix can't be resolved automatically
- Resolves pushback deadlocks instead of arguing indefinitely
- Optional approval gates for human oversight at key decision points
