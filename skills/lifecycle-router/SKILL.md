---
name: lifecycle-router
description: This skill should be used when classifying improvement candidates into lifecycle tiers (small, medium, large) and determining which skill chain to execute. Provides classification criteria and skill chain mappings for the autonomous PM agent.
---

# Lifecycle Router

Classify each improvement candidate into a lifecycle tier (small, medium, or large) and execute the corresponding skill chain. The tier determines how much ceremony the change receives -- more ceremony for riskier or more complex changes, less for mechanical fixes.

Always create a GitHub Issue before starting any chain. Always close the issue after merge.

---

## Classification Criteria

Evaluate each candidate against the tier definitions below. Start from the top: check large first, then medium, then small. When a candidate straddles two tiers, classify one tier higher -- more ceremony is always safer than less.

### Small Tier

ALL of the following must be true for a candidate to qualify as small:

- Touches 1-3 files.
- Introduces no new public interfaces or APIs.
- Requires no architectural decisions.
- The fix is obvious and mechanical -- a reasonable reviewer would approve it without discussion.
- No new tests are needed, or the only test change is a trivial assertion update.

**Examples of small work:**

- Fix a typo in README or docs.
- Add a missing `.gitignore` entry.
- Adjust lint or formatter configuration.
- Remove an unused import or dead variable.
- Bump a non-breaking dependency version.
- Delete commented-out code.
- Fix a broken link in documentation.

### Medium Tier

ANY of the following is true, AND the candidate does not meet any large tier criterion:

- Touches 4 or more files.
- Adds or modifies a test suite (new test file, new describe block, or substantial assertion additions).
- Introduces a new function, endpoint, handler, or module that follows an existing pattern in the codebase.
- Fixes a bug that requires understanding control flow across multiple functions or modules.
- Refactors code that has existing test coverage (internal restructuring with behavioral preservation).
- Adds or modifies input validation logic.

**Examples of medium work:**

- Add test coverage for an untested module.
- Implement a new API endpoint following the project's existing routing and handler patterns.
- Fix a multi-file bug where the root cause and the symptom are in different modules.
- Add input validation and error messages to request handlers.
- Refactor duplicated logic into a shared utility with tests.
- Add error handling to functions that currently swallow exceptions.

### Large Tier

ANY of the following is true:

- Introduces a new architectural pattern or module structure not already present in the codebase.
- Creates a new system boundary (new service, new database table, new external API integration, new message queue consumer).
- Changes the data flow -- how data moves between modules, services, or layers.
- Requires design decisions where multiple valid approaches exist and the choice has lasting consequences.
- Affects a public interface or contract (API response shape, CLI flags, config file format, library exports).

**Examples of large work:**

- Add an authentication or authorization system.
- Restructure the project directory layout or module boundaries.
- Integrate a new external service (payment provider, email service, search engine).
- Implement a multi-module feature that spans data model, business logic, and API layers.
- Replace a core dependency with a different library (e.g., swapping ORMs or HTTP frameworks).
- Add a caching layer or change the caching strategy.

---

## Issue Creation

Before starting any skill chain, create a GitHub Issue to track the work:

```
gh issue create --title "<short description>" --body "<context, acceptance criteria>" --label "<tier>" --label "pm-automated"
```

Include in the issue body:

- What the change does and why it matters.
- Acceptance criteria: specific conditions that must be true when the work is complete.
- The priority tier (P1-P5) and lifecycle tier (small, medium, large).

Capture the returned issue number. Use it in:

- Branch names: `pm/<tier>/<issue-number>-<slug>`
- PR bodies: `Closes #<issue-number>` (triggers auto-close on merge)

---

## Skill Chain: Small

Small changes are direct fixes with minimal ceremony. No plan, no worktree, no brainstorming.

### Steps

1. **Create a feature branch.**
   ```
   git checkout -b pm/small/<issue-number>-<slug>
   ```
   Derive the slug from the issue title in kebab-case. Keep it short (3-5 words max).

2. **Make the fix directly.** Open the relevant files, apply the change. Small items are mechanical -- the fix should be obvious from the candidate description.

3. **Verify the change.** Invoke: `superpowers:verification-before-completion`. This runs the test suite, linter, type checker, and formatter. All must pass. If verification fails, fix the issue before proceeding.

4. **Commit, push, and open a PR.** Invoke: `commit-commands:commit-push-pr`. Include `Closes #<issue-number>` in the PR body. Use conventional commit format with `(pm)` scope.

5. **Run the Copilot review loop.** Request review from Copilot, read feedback, address every comment, and re-request review until approved. See the PM agent definition for the full review loop procedure and stuck detection.

6. **Close the issue after merge.** Verify auto-close triggered. If it did not, close manually:
   ```
   gh issue close <issue-number>
   ```

---

## Skill Chain: Medium

Medium changes benefit from isolated work and a written plan. Use a worktree to keep the main working directory clean, and write a plan before executing.

### Steps

1. **Create a worktree.** Invoke: `superpowers:using-git-worktrees`. This creates a fresh worktree on a new branch named `pm/medium/<issue-number>-<slug>`. All subsequent work for this item happens in the worktree.

2. **Write and execute a plan.** Invoke: `superpowers:writing-plans`. This skill chains automatically into `superpowers:executing-plans` after the plan is written. When prompted for an execution strategy, choose **"Subagent-Driven"** so independent plan steps can execute in parallel.

   The plan should include:
   - What files to modify and why.
   - What tests to add or update.
   - The order of operations (what depends on what).
   - Verification criteria for each step.

3. **Verify the change.** Invoke: `superpowers:verification-before-completion`. All tests, lint, type checks, and formatting must pass.

4. **Commit, push, and open a PR.** Invoke: `commit-commands:commit-push-pr`. Include `Closes #<issue-number>` in the PR body. Use conventional commit format with `(pm)` scope.

5. **Run the Copilot review loop.** Follow the same review loop as small tier: request review, address feedback, re-request until approved or escalated.

6. **Close the issue after merge.** Verify auto-close triggered. If it did not, close manually.

---

## Skill Chain: Large

Large changes require design exploration before committing to a plan. Start with brainstorming to explore the design space and narrow down the approach.

### Steps

1. **Brainstorm the approach.** Invoke: `superpowers:brainstorming`. This explores the design space: what are the possible approaches, what are the tradeoffs, what are the risks, what is the recommended path forward.

   Brainstorming chains automatically into `superpowers:writing-plans` and then into `superpowers:executing-plans`. When prompted for an execution strategy, choose **"Subagent-Driven"** so independent plan steps can execute in parallel.

   The brainstorming output should cover:
   - Two or more viable approaches with tradeoffs.
   - The recommended approach with reasoning.
   - Known risks and how to mitigate them.
   - What tests or verification will confirm the change is correct.

2. **Verify the change.** Invoke: `superpowers:verification-before-completion`. All tests, lint, type checks, and formatting must pass. For large changes, also verify that no existing public interfaces changed unintentionally -- compare API surfaces, CLI help output, or config schemas before and after.

3. **Commit, push, and open a PR.** Invoke: `commit-commands:commit-push-pr`. Include `Closes #<issue-number>` in the PR body. Use conventional commit format with `(pm)` scope. For large PRs, include a summary in the PR description explaining the design decision and why this approach was chosen over alternatives.

4. **Run the Copilot review loop.** Follow the same review loop: request review, address feedback, re-request until approved or escalated.

5. **Close the issue after merge.** Verify auto-close triggered. If it did not, close manually.

---

## Decision Rules

These rules handle ambiguous cases during classification and execution.

### Tier ambiguity

- When a candidate could be either small or medium, check whether it introduces any new function, test file, or module. If yes, classify as medium.
- When a candidate could be either medium or large, check whether it introduces a pattern not already present in the codebase. If yes, classify as large.
- When uncertain, classify one tier higher. The cost of extra ceremony is low; the cost of insufficient ceremony is a broken merge.

### Scope creep during execution

- If a small item turns out to touch 4+ files during implementation, stop, re-classify as medium, and switch to the medium chain (create a worktree, write a plan).
- If a medium item reveals an architectural decision during planning, stop, re-classify as large, and switch to the large chain (brainstorm first).
- Update the GitHub Issue label when reclassifying. Add a comment explaining why the tier changed.

### Chain failures

- If `superpowers:verification-before-completion` fails and the fix is not obvious after two attempts, escalate the PR with the `pm-escalated` label and move on.
- If `superpowers:writing-plans` produces a plan that would modify a public interface, re-evaluate whether this item is actually large tier. If so, restart with the large chain.
- If the Copilot review loop gets stuck (same comment recurring after 2 fix attempts), escalate per the stuck detection procedure.

### Multiple candidates at the same priority

- When several candidates share the same priority tier, prefer the one with the smallest scope (fewer files, simpler change). Quick wins build momentum and reduce the candidate list.
- Between two candidates of equal scope, prefer the one that unblocks or simplifies other candidates.
