# Plugin Sync, Review Patterns, and Scope-Aware Pushback Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Automate plugin file sync (#25), wire review patterns into implementer dispatch (#26), and add diff-aware scope checking before addressing Copilot comments (#27).

**Architecture:** Three independent changes to the autopilot plugin. Task 1 adds a git post-commit hook to the source repo that copies plugin files to the marketplace and cache. Task 2 fills the gap between pattern storage (Phase 5 Step 8) and pattern consumption (Phase 2/3) by adding concrete extraction code and implementer awareness. Task 3 adds a diff-classification step in Phase 5 Step 3 that checks each Copilot comment against the PR's changed hunks before dispatching fixes.

**Tech Stack:** Bash (git hooks), Markdown (skill/agent prompts)

**Status:** All three tasks have been implemented and merged to main. This plan is retained as an archival reference.

---

## Task 1: Automate Plugin Sync (Issue #25)

**Files:**
- Create: `.git/hooks/post-commit`
- Modify: `CLAUDE.md` (update plugin installation section to reference the hook)

### Step 1: Create the post-commit hook

Create `.git/hooks/post-commit` with this content:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Sync plugin files from source repo to local marketplace and cache.
# This hook runs after every commit to keep all three locations in sync.

REPO_ROOT="$(git rev-parse --show-toplevel)"
MARKETPLACE="$HOME/.claude/plugins/local-marketplace/plugins/autopilot"
CACHE="$HOME/.claude/plugins/cache/local-plugins/autopilot/0.1.0"

# The 9 plugin files that Claude Code loads
PLUGIN_FILES=(
  ".claude-plugin/plugin.json"
  "commands/autopilot.md"
  "skills/pm-workflow/SKILL.md"
  "agents/implementer.md"
  "hooks/hooks.json"
  "hooks/run-hook.cmd"
  "hooks/session-start"
  "CLAUDE.md"
  "README.md"
)

for FILE in "${PLUGIN_FILES[@]}"; do
  SRC="$REPO_ROOT/$FILE"
  if [ -f "$SRC" ]; then
    for DEST in "$MARKETPLACE" "$CACHE"; do
      mkdir -p "$DEST/$(dirname "$FILE")"
      cp "$SRC" "$DEST/$FILE"
    done
  fi
done
```

Run: `ls -la .git/hooks/post-commit`
Expected: File exists

### Step 2: Make the hook executable

Run: `chmod +x .git/hooks/post-commit`

### Step 3: Verify the hook works

Run: `git commit --allow-empty -m "test: verify post-commit hook syncs plugin files"`
Then: `diff skills/pm-workflow/SKILL.md ~/.claude/plugins/cache/local-plugins/autopilot/0.1.0/skills/pm-workflow/SKILL.md && echo "MATCH"`
Expected: `MATCH`

### Step 4: Update CLAUDE.md to reference the hook

In `CLAUDE.md`, replace the "Plugin Installation" section with:

```markdown
## Plugin Installation

This plugin is installed in three locations. A post-commit hook (`.git/hooks/post-commit`) automatically syncs all plugin files after every commit:

1. **Source repo** (you edit here) — `~/Projects/autonomous-pm/`
2. **Local marketplace** (synced by hook) — `~/.claude/plugins/local-marketplace/plugins/autopilot/`
3. **Plugin cache** (synced by hook) — `~/.claude/plugins/cache/local-plugins/autopilot/0.1.0/`

If the hook is missing (e.g., after a fresh clone), recreate it manually by following the post-commit hook setup steps in this plan. Git hooks live under `.git/hooks` and are not version-controlled, so this file will not be present after `git clone`.
```

### Step 5: Commit

```bash
git add CLAUDE.md
git commit -m "Add post-commit hook to auto-sync plugin files (#25)"
```

Then verify sync happened: `diff CLAUDE.md ~/.claude/plugins/cache/local-plugins/autopilot/0.1.0/CLAUDE.md && echo "MATCH"`
Expected: `MATCH`

---

## Task 2: Wire Review Patterns into Implementer Dispatch (Issue #26)

**Files:**
- Modify: `skills/pm-workflow/SKILL.md` (Phase 2 "Review pattern constraints" section, ~line 354)
- Modify: `skills/pm-workflow/SKILL.md` (Phase 3 dispatch template, ~line 485)
- Modify: `agents/implementer.md` (add pattern awareness section)

### Step 1: Replace Phase 2 "Review pattern constraints" with concrete instructions

In `skills/pm-workflow/SKILL.md`, find the section starting at line 354 ("### Review pattern constraints"). Replace it with:

````markdown
### Review pattern constraints

Check for `.autopilot/review-patterns.md` in the target repo. If it exists, extract actionable constraints:

```bash
if [ -f ".autopilot/review-patterns.md" ]; then
  echo "Review patterns file found. Extracting top constraints..."
else
  echo "No review patterns file. Skipping."
fi
```

If the file exists, read it and identify categories that appear across 2 or more PR sections (each `## PR #N` heading is one section). Count how many distinct PR sections contain each category. Rank by frequency and take the top 3.

Store the result as `REVIEW_CONSTRAINTS` — a numbered list of plain-language rules. For example:

```
REVIEW_CONSTRAINTS:
1. Always add null checks when accessing properties from external inputs or API responses.
2. Include error context (what operation failed, what input caused it) in every catch block.
3. Use explicit return types on all public functions.
```

If the file does not exist, has fewer than 2 PR sections, or no category appears in 2+ sections, set `REVIEW_CONSTRAINTS` to empty and skip.
````

### Step 2: Update Phase 3 dispatch template to include constraints

In `skills/pm-workflow/SKILL.md`, find the "Provide every implementer with" list (~line 493). Replace it with:

```markdown
Provide every implementer with:
- The specific task from the plan
- The GitHub issue number for this subtask
- Relevant file paths
- Which superpowers skills to use (TDD, verification-before-completion)
- The repo's conventions from CLAUDE.md
- Review pattern constraints from Phase 2 (if any). Include verbatim:
  "Constraints from past Copilot reviews — follow these to avoid known review feedback:
  [paste REVIEW_CONSTRAINTS here]"
  If REVIEW_CONSTRAINTS is empty, omit this section from the prompt.
```

### Step 3: Add pattern awareness to implementer agent

In `agents/implementer.md`, add a new section between "## Operating Modes" (after Review-Fix Mode) and "## Git Lock Recovery". Insert:

```markdown
## Review Pattern Constraints

The PM may include constraints from past Copilot reviews in your dispatch prompt. These are patterns that Copilot has repeatedly flagged across previous PRs. When provided, treat them as mandatory requirements — the same priority as the repo's CLAUDE.md conventions.

Example constraints you may receive:
- "Always add null checks when accessing properties from external inputs"
- "Include error context in every catch block"

If no constraints are provided, ignore this section.
```

### Step 4: Commit

```bash
git add skills/pm-workflow/SKILL.md agents/implementer.md
git commit -m "Wire review patterns from storage to implementer dispatch (#26)"
```

---

## Task 3: Add Scope-Aware Pushback for Out-of-Diff Comments (Issue #27)

**Files:**
- Modify: `skills/pm-workflow/SKILL.md` (Phase 5, between Step 2 and current Step 3, ~line 886)

### Step 1: Add a new Step 3 for diff classification

In `skills/pm-workflow/SKILL.md`, find the line "### Step 3: Invoke receiving-code-review" (~line 888). Insert a new step BEFORE it. This means the current Step 3 becomes Step 4, Step 4 becomes Step 5, and so on through Step 8. Renumber all subsequent steps accordingly.

Insert this new Step 3:

````markdown
### Step 3: Classify comments by PR scope

Before evaluating comments, determine which ones target code the PR actually changed versus pre-existing code.

**Get the PR's changed file/line ranges:**
```bash
gh pr diff $PR_NUMBER --name-only
```

For a more precise check, get the full diff with line numbers:
```bash
gh pr diff $PR_NUMBER > /tmp/pr_diff.txt
```

**For each unresolved Copilot comment from Step 2, classify it:**

- **In-scope** — The comment's `path` is in the PR's changed files AND the comment's `line` falls within a changed hunk (added/modified lines in the diff). These get processed normally in Step 4.
- **Out-of-scope** — The comment's `path` is not in the PR's changed files, OR the `line` is outside any changed hunk. These are pre-existing issues that the PR did not introduce.

**Handle out-of-scope comments:**

For each out-of-scope comment, reply in the thread explaining this is pre-existing code:
```bash
gh api repos/$OWNER/$REPO/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies \
  -f body="This comment targets code not modified by this PR (pre-existing). Filing as a separate issue for future cleanup."
```

Optionally batch-create a single issue for all out-of-scope findings:
```bash
# Only if there are out-of-scope comments worth tracking
gh issue create --title "Address pre-existing code issues flagged on PR #$PR_NUMBER" --body "$(cat <<'ISSUE_EOF'
## Pre-existing issues flagged by Copilot

These were flagged during review of PR #$PR_NUMBER but target code not modified by that PR.

$(for each out-of-scope comment: "- **$PATH:$LINE** — $BODY")

Filed automatically by Autopilot PM.
ISSUE_EOF
)"
```

Add the out-of-scope thread IDs to the resolution list in Step 6 (they should be resolved after replying).

**Proceed to Step 4 with only the in-scope comments.**
````

### Step 2: Renumber Steps 3-8 to Steps 4-9

Find and replace in `skills/pm-workflow/SKILL.md` within the Phase 5 section only:

- Current "### Step 3: Invoke receiving-code-review" becomes "### Step 4: Invoke receiving-code-review"
- Current "### Step 4: Address comments" becomes "### Step 5: Address comments"
- Current "### Step 5: Resolve all addressed threads" becomes "### Step 6: Resolve all addressed threads"
- Current "### Step 6: Commit, push, request review, and verify CI" becomes "### Step 7: Commit, push, request review, and verify CI"
- Current "### Step 7: Check for completion" becomes "### Step 8: Check for completion"
- Current "### Step 8: Extract review patterns" becomes "### Step 9: Extract review patterns"

Also update all internal references to step numbers within Phase 5:
- "Step 1b" references remain unchanged (still Step 1)
- "Step 5" references in HARD-GATE about thread resolution now point to "Step 6"
- "Steps 1-7" in the cycle budget description becomes "Steps 1-8"
- "go back to Step 1" in Step 7 (now Step 8) remains correct

### Step 3: Update the out-of-scope thread handling in Step 6 (resolve threads)

In the renamed Step 6 (formerly Step 5, "Resolve all addressed threads"), update the description to include out-of-scope threads:

Find: "Resolve all addressed threads (both fixed and pushed-back)"
Replace with: "Resolve all addressed threads (fixed, pushed-back, and out-of-scope)"

### Step 4: Commit

```bash
git add skills/pm-workflow/SKILL.md
git commit -m "Add diff-aware scope check before addressing Copilot comments (#27)"
```

---

## Execution Notes

- Tasks 1, 2, and 3 are fully independent — they can be implemented in any order or in parallel
- Task 1 modifies non-plugin infrastructure (git hook + CLAUDE.md)
- Tasks 2 and 3 both modify `skills/pm-workflow/SKILL.md` but in different sections (Phase 2/3 vs Phase 5), so they shouldn't conflict
- After all tasks, the post-commit hook from Task 1 will automatically sync changes from Tasks 2 and 3 to the marketplace and cache
