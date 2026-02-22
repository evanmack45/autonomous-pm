# Autonomous PM

A Claude Code plugin that autonomously manages project development. When installed in a repo and kicked off with `/autonomous-pm:run`, it analyzes the codebase, identifies improvements, and executes the full development lifecycle with cross-LLM review from GitHub Copilot.

## Architecture

Skill-chain pipeline. The plugin provides:
1. The outer loop (what to work on next) via the PM agent
2. Lifecycle routing (which skill chain to run) via the lifecycle-router skill

Existing superpowers skills handle design, planning, implementation, verification, and review.

## Components

- `commands/run.md` -- User entry point. Checks prerequisites, initializes manifest, launches PM agent.
- `agents/pm-agent.md` -- Orchestrator. Runs the orient-identify-classify-execute-evaluate loop.
- `skills/project-intelligence/` -- Codebase analysis and candidate generation.
- `skills/lifecycle-router/` -- Tier classification and skill chain mapping.
- `skills/convergence-checker/` -- Stopping criteria and run reports.

## Dependencies

- GitHub CLI (`gh`) for issue and PR management
- Git worktrees for branch isolation
- Superpowers plugin for development lifecycle skills
- GitHub Copilot for cross-LLM code review

## Project State

The PM writes its state to `.pm/` in the target repository:
- `.pm/manifest.md` -- persistent memory (project understanding, completed/rejected/escalated items)
- `.pm/runs/YYYY-MM-DD.md` -- run reports
