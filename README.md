# Autonomous PM

A Claude Code plugin that runs your project for you.

## What It Does

Install this plugin in any repo, run `/autonomous-pm:run`, and the PM agent takes over. It:

1. Analyzes your codebase to understand what it is and what state it's in
2. Identifies improvements -- from broken tests to missing features
3. Executes each improvement through a strict development lifecycle
4. Gets every PR reviewed by GitHub Copilot (a different LLM)
5. Merges clean PRs and stops when there's nothing meaningful left to do

All work is tracked through GitHub Issues. The agent creates an issue before starting each item, links it in the PR, and closes it on merge.

## Lifecycle Tiers

The PM scales ceremony to the size of the change:

- **Small** (typo, lint fix): branch, fix, verify, PR, review, merge
- **Medium** (new tests, bug fix): branch, plan, implement, verify, PR, review, merge
- **Large** (new feature): brainstorm, design, plan, implement, verify, PR, review, merge

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
# then type: /autonomous-pm:run
```

The PM runs until convergence. When it stops, it prints a summary of what was accomplished.

Check `.pm/runs/` for detailed run reports and `.pm/manifest.md` for the agent's project understanding and history.

## How It Works

The PM agent runs a five-step loop:

1. **Orient** -- Read the codebase, docs, and manifest to build a mental model
2. **Identify** -- Generate ranked improvement candidates (broken things first, new features last)
3. **Classify** -- Assign each candidate a tier (small, medium, large)
4. **Execute** -- Run the skill chain for that tier, including Copilot code review
5. **Evaluate** -- Check if meaningful work remains; if yes, loop back

The agent stops when it can't find anything worthwhile left to do, or when all remaining candidates are speculative.

## Safety

- Never deletes user code without replacement tests passing
- Never changes public interfaces unless fixing a bug
- Never pushes directly to main
- Never commits secrets
- Escalates to human review when stuck instead of forcing through
