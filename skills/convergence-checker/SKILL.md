---
name: convergence-checker
description: This skill should be used when the autonomous PM agent needs to decide whether to continue working or stop. Provides convergence criteria, stopping conditions, and run report format.
---

# Convergence Checker

Evaluate whether the autonomous PM loop should continue executing or stop. This skill runs after each completed (or escalated) work item, during Step 5 (Evaluate) of the outer loop.

---

## Convergence Evaluation

After completing each item, re-run the Identify phase using the project-intelligence skill against the updated manifest. Generate a fresh candidate list with all completed, rejected, and escalated items filtered out. Then evaluate the candidate list against these criteria.

### Continue Conditions

Continue the loop if any of the following are true:

- **P1 candidates exist.** Broken things (failing tests, build errors, runtime crashes) always take priority. Never stop while something is broken.
- **P2 candidates exist.** Missing foundations (no tests, no linter, no CI, incomplete `.gitignore`) represent gaps that compound over time. Address them before stopping.
- **P3 candidates exist.** Functionality gaps (swallowed exceptions, missing validation, incomplete error handling) affect reliability. Continue working through them.
- **P4 candidates exist AND at least one is clearly impactful.** Quality improvements like dead code removal, dependency updates with security implications, or significant code duplication warrant action. Skip cosmetic-only P4 items -- formatting tweaks, minor naming inconsistencies, or stylistic preferences do not justify another cycle.

When continuing, return to Step 3 (Classify) with the top-ranked candidate and proceed through execution.

### Stop Conditions

Stop the loop if any of the following are true:

- **Zero candidates remain.** After filtering out completed, rejected, and escalated items, no work items exist across any priority tier. The project is in good shape.
- **Only cosmetic P4 candidates remain.** The remaining P4 items are formatting fixes, minor naming adjustments, or style preferences -- nothing that affects correctness, reliability, or maintainability. These do not justify the overhead of another cycle.
- **Only P5 candidates remain and none are natural extensions.** The only work left is new features, and none of them are obvious, low-risk extensions of existing functionality. Do not add features for the sake of having something to do.
- **A full Identify cycle rejected every candidate.** The project-intelligence skill ran, generated candidates, and each one was evaluated and determined to be not worth doing (too risky, too speculative, already handled adequately). Log all rejected candidates with reasoning in the manifest.

When stopping, proceed to the Manifest Update and Run Report sections below.

### Edge Cases

**Repeated failures on the same candidate.** If a candidate has been attempted and failed (implementation broke tests, review could not be resolved, approach turned out to be wrong) across two or more attempts, stop retrying. Add the item to "Considered and Rejected" in the manifest with an explanation of what was tried and why it failed. Do not count repeated failures as progress -- move on to the next candidate or evaluate convergence.

**Greenfield project fully scaffolded.** When a greenfield project has been scaffolded (project structure, package manifest, tooling configured) and initial features have been implemented to match the project description, stop. Do not invent features beyond what the project description calls for. A greenfield project that builds, passes tests, has linting configured, and implements its core purpose is done.

**Escalation cascade.** If three or more consecutive items are escalated rather than completed, pause and evaluate whether the remaining candidates are within the agent's capability. If the pattern suggests the work requires human judgment or domain knowledge the agent lacks, stop and note this in the run report rather than continuing to generate escalated PRs.

**Diminishing returns.** If the last two completed items were both P4 and the remaining candidates are also P4 or P5, evaluate whether the project has reached a natural stopping point. Stopping one cycle earlier is always better than doing marginal work that adds noise.

---

## Manifest Updates on Stop

Before writing the run report, ensure the manifest at `.pm/manifest.md` is fully current. Walk through each section and verify:

**Completed Items table.** Every item completed during this run has an entry with:
- Date of completion
- GitHub Issue link (`#N`)
- Pull Request link (`#N`)
- Short description of what was done

**Considered and Rejected table.** Every item that was evaluated and skipped has an entry with:
- Date of evaluation
- Description of what was considered
- Specific reasoning for why it was not worth doing

**Escalated Items table.** Every item that was escalated has an entry with:
- Date of escalation
- PR link (`#N`)
- Description of what was attempted
- Current status (`open`)

**Run History table.** Add a new row for this run with:
- Date
- Count of items completed
- Count of items escalated
- Count of items skipped
- Approximate duration of the run

**Project Understanding section.** Update to reflect the current state of the project after this run's changes. Include any new tooling configured, architectural changes made, or gaps that remain.

Commit the updated manifest:

```
git add .pm/manifest.md
git commit -m "pm: update manifest for YYYY-MM-DD run"
```

---

## Run Report

When stopping, write a run report to `.pm/runs/YYYY-MM-DD.md`. If a report for today already exists, append a counter: `YYYY-MM-DD-2.md`, `YYYY-MM-DD-3.md`.

### Report Structure

```markdown
# PM Run Report -- YYYY-MM-DD

## Summary

<One paragraph covering: how many items were completed, what priority tiers
they fell into, what the project health looks like now, and why the run
is stopping. Be specific about the stopping reason -- "zero candidates
remain" is different from "only cosmetic P4s remain.">

## Completed

| Issue | PR | Tier | Description |
|-------|----|------|-------------|
| [#N](link) | [#N](link) | P1-P5 | What was done |

## Considered and Skipped

| Description | Tier | Reason |
|-------------|------|--------|
| What was considered | P1-P5 | Why it was not worth doing |

## Escalated

| PR | Tier | Description | What is stuck |
|----|------|-------------|---------------|
| [#N](link) | P1-P5 | What was attempted | Why it could not be resolved |

## Project State

<Narrative assessment of the project after this run. Cover:>

- **What is healthy:** Areas that are solid -- passing tests, clean linting,
  good error handling, adequate documentation.
- **What could use attention in a future run:** Remaining gaps that did not
  meet the bar for this run but are worth revisiting. Be specific enough
  that a future run can pick these up without re-analyzing from scratch.
- **Open escalations:** Any PRs left for human review, with enough context
  for the reviewer to understand what was tried.
```

If a section has no entries (e.g., nothing was escalated), include the heading with "None." beneath it. Do not omit sections.

### Commit the Report

```
git add .pm/runs/
git commit -m "pm: run report for YYYY-MM-DD"
```

---

## Decision Flowchart

For quick reference, the evaluation logic in order:

1. Run Identify with updated manifest. Filter completed, rejected, escalated items.
2. Any P1 candidates? Continue.
3. Any P2 candidates? Continue.
4. Any P3 candidates? Continue.
5. Any impactful P4 candidates (not cosmetic-only)? Continue.
6. Any P5 candidates that are natural extensions? Continue.
7. Otherwise, stop. Update manifest. Write run report.
