---
name: run
description: Launch the autonomous PM to analyze, improve, and ship code with cross-LLM review
---

# Run Autonomous PM

Launch the autonomous project manager on the current repository. The PM analyzes the codebase, identifies improvements, and executes them through a strict development lifecycle with cross-LLM code review.

## Prerequisites

Before starting, verify the environment:

1. Confirm this is a git repository: `git rev-parse --is-inside-work-tree`
2. Confirm GitHub CLI is available and authenticated: `gh auth status`
3. Confirm the repo has a GitHub remote: `gh repo view --json name`

If any check fails, report the specific issue and stop. Do not proceed without all three.

## Initialize Manifest

Check if `.pm/manifest.md` exists in the repository root.

If it does not exist:

1. Create the directory structure: `mkdir -p .pm/runs`
2. Write `.pm/manifest.md` with the empty manifest template (defined in the PM agent instructions)
3. Ensure the required GitHub labels exist:
   - `gh label create small --description "Small tier: mechanical fix" --color "0E8A16" 2>/dev/null || true`
   - `gh label create medium --description "Medium tier: substantive change" --color "FBCA04" 2>/dev/null || true`
   - `gh label create large --description "Large tier: feature or architectural change" --color "D93F0B" 2>/dev/null || true`
   - `gh label create pm-escalated --description "PM could not resolve review feedback" --color "B60205" 2>/dev/null || true`
4. Commit the manifest: `git add .pm/ && git commit -m "pm: initialize project manifest"`

If it already exists, read it to restore context from previous runs.

## Launch the PM Agent

Hand full control to the PM agent. The agent runs the outer loop autonomously:

1. **Orient** -- understand the project
2. **Identify** -- find candidates for improvement
3. **Classify** -- assign lifecycle tiers (small, medium, large)
4. **Execute** -- run the appropriate skill chain for each item
5. **Evaluate** -- check convergence, loop or stop

The agent manages everything from here: creating GitHub Issues, branching, invoking skills, handling Copilot code review, merging, and updating the manifest.

When the agent reaches convergence and stops, it writes a run report to `.pm/runs/YYYY-MM-DD.md`. Read the report and present a conversational summary of what was accomplished.
