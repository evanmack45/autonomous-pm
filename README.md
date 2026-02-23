# Autopilot

A Claude Code plugin that autonomously manages your project. Run `/autopilot` in any repo and the PM takes over — it assesses the codebase, picks the highest-priority work, plans it, implements via subagents, creates a PR, and iterates through GitHub Copilot code reviews until the PR is clean.

## What It Does

1. **Assesses** the repo — reads docs, checks issues, runs tests, discovers CI workflows, understands the current state
2. **Picks** the highest-priority task and creates GitHub issues to track it
3. **Plans** the work using brainstorming and planning skills scaled to complexity
4. **Implements** by dispatching subagents in isolated git worktrees
5. **Validates** against acceptance criteria, runs tests, checks scope, updates docs
6. **Opens a PR** linked to the issue with full context
7. **Iterates** through Copilot code reviews and CI checks — fixes feedback, pushes back when warranted, resolves threads
8. **Merges** the PR when Copilot review is clean and CI is green

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

## Plugin Structure

```
.claude-plugin/plugin.json   # Plugin metadata
commands/autopilot.md         # /autopilot slash command
skills/pm-workflow/SKILL.md   # Core PM workflow (6 phases)
agents/implementer.md         # Subagent for coding tasks and review fixes
hooks/                        # Session-start hook (reminder to run /autopilot)
```

## Safety

- Never pushes directly to main — always uses feature branches
- Never force pushes
- Never skips tests
- Stops the review loop and escalates when a fix can't be resolved automatically
- Resolves pushback deadlocks instead of arguing indefinitely
