---
name: project-intelligence
description: This skill should be used when analyzing a codebase to identify improvement candidates. Provides structured analysis patterns for code quality, test coverage, architecture, dependency health, and feature completeness. Used by the autonomous PM agent during Orient and Identify phases.
---

# Project Intelligence

Analyze any codebase systematically to produce a ranked list of improvement candidates. Work through each analysis step in order. Higher-priority findings always take precedence over lower-priority ones.

The output of this skill is a numbered candidate list that feeds into the PM agent's Classify step.

---

## Existing Project Analysis

For projects with source code already in place, run these six analysis steps sequentially. Each step has a priority tier -- if a step produces findings, tag every candidate with that tier.

### Step 1: Build and Test Health (P1)

Determine whether the project compiles, builds, and passes its test suite. Broken builds and failing tests block all other work.

1. Detect the tech stack from package manifests (`package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `Gemfile`, `pom.xml`).
2. Identify the build command. Check `scripts` in `package.json`, `Makefile`, `justfile`, `build.rs`, or framework conventions.
3. Run the build. Record whether it succeeds, fails, or has no build step.
4. Identify the test command. Check for `test` scripts, `pytest.ini`, `vitest.config`, `.cargo/config.toml`, or test directory conventions.
5. Run the test suite. Record pass count, fail count, and skip count.
6. If the build fails, emit a P1 candidate: "Fix build failure" with the error summary.
7. If any tests fail, emit a P1 candidate per failing test file (or per failing test if the count is small): "Fix failing test in `<module>`" with the failure message.

Do not proceed to lower-priority steps if P1 candidates exist. The PM agent will address these first.

### Step 2: Foundation Check (P2)

Check whether the project has the baseline infrastructure that every maintained project needs. Missing foundations are P2 candidates.

1. **Test infrastructure** -- Check for a test framework configuration and at least one test file. If no tests exist at all, emit: "Add test infrastructure and initial test suite."
2. **Linter** -- Check for linter configuration (`.eslintrc`, `oxlint`, `biome.json`, `ruff.toml`, `clippy` in `Cargo.toml`, etc.). If absent, emit: "Configure linter."
3. **Formatter** -- Check for formatter configuration (`prettier`, `oxfmt`, `ruff format`, `rustfmt`, etc.). If absent, emit: "Configure formatter."
4. **CI pipeline** -- Check for `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, or equivalent. If absent, emit: "Add CI pipeline."
5. **Type checking** -- For languages with static type support (TypeScript, Python with type hints, Rust), check for type checker configuration (`tsconfig.json` with `strict`, `ty` or `mypy` config, etc.). If the language supports types but they are not configured, emit: "Configure type checking."
6. **`.gitignore`** -- Check for a `.gitignore` file. Verify it covers build artifacts, dependency directories, OS files, and environment files for the detected stack. If missing or incomplete, emit: "Fix `.gitignore` coverage."

### Step 3: Code Quality Scan (P3)

Scan the source code for common quality problems. These are not broken -- the project works -- but they represent gaps in the existing code.

1. **Dead code** -- Search for unused imports, unreachable branches, commented-out code blocks, and unused variables or functions. Emit one candidate per cluster of dead code in the same module.
2. **Inconsistent patterns** -- Identify where the same concern is handled differently across the codebase (e.g., error handling done three different ways, mixed async patterns, inconsistent naming). Emit one candidate per pattern inconsistency, describing both the inconsistency and the dominant pattern to converge on.
3. **Swallowed exceptions** -- Find catch blocks, `except` clauses, or error handlers that discard the error without logging or re-throwing. Emit one candidate per occurrence.
4. **Missing input validation** -- Check functions that accept external input (HTTP handlers, CLI argument parsers, file readers, user-facing APIs) for missing validation. Emit one candidate per unvalidated entry point.
5. **Hardcoded values** -- Find hardcoded URLs, ports, file paths, credentials, magic numbers, and configuration that should be externalized. Emit one candidate per category of hardcoded value (not per occurrence).
6. **Missing error context** -- Find error messages that provide no actionable information (bare `throw new Error()`, `raise Exception()`, `panic!()` without a message). Emit one candidate per module with this problem.

### Step 4: Dependency Health (P4)

Examine the project's dependency tree for problems. Security vulnerabilities are P3 (upgrade to P3 if found); everything else stays P4.

1. **Outdated dependencies** -- Check for available major and minor version updates. Use `npm outdated`, `cargo outdated`, `uv pip list --outdated`, or equivalent. Emit one candidate summarizing the outdated dependency count and highlighting any with breaking changes.
2. **Vulnerable dependencies** -- Run `npm audit`, `cargo audit`, `pip-audit`, or equivalent. If vulnerabilities exist, emit one P3 candidate per severity level (critical, high, moderate), describing the affected packages.
3. **Unused dependencies** -- Check for dependencies declared in the manifest but not imported anywhere in the source. Emit one candidate listing unused dependencies for removal.
4. **Unpinned dependencies** -- Check for loose version specifiers (`^`, `~`, `>=`, `*`) that could cause non-reproducible builds. Emit one candidate if pinning is inconsistent or absent.

### Step 5: Documentation Accuracy (P4)

Compare what the documentation claims with what the code actually does. Outdated docs are worse than no docs.

1. Read `README.md`. Extract every factual claim: setup steps, available commands, API endpoints, configuration options, feature descriptions.
2. Verify each claim against the actual codebase. Check that commands exist, endpoints are implemented, config options are read, and described features work.
3. For each discrepancy, emit one candidate: "Fix README: `<claim>` does not match implementation." Describe what the README says and what the code actually does.
4. Check for documented features that no longer exist (removed but not updated in docs). Emit one candidate per phantom feature.
5. If no README exists, emit one P4 candidate: "Add README with project description and setup instructions."

### Step 6: Feature Completeness (P5)

Identify natural, obvious gaps in the project's functionality. Apply strict criteria -- only suggest features that are clearly missing, not speculatively useful.

1. Look for asymmetries: CRUD operations missing one verb, handlers that process some formats but not an obviously adjacent one, modules that handle the happy path but have no error recovery.
2. Look for TODO/FIXME comments that describe clearly scoped, implementable work.
3. For each gap, emit one candidate describing what exists, what is missing, and why the gap is obvious rather than speculative.

Do not emit P5 candidates for:
- Features the project has never claimed to support.
- Nice-to-haves that require new external interfaces.
- Anything that changes how the project is used or deployed.
- Plugin systems, extension points, or configuration frameworks.

---

## Greenfield Project Analysis

For projects with no source files (only a README, a manifest, or an empty directory), skip the six-step existing project analysis. Instead, generate scaffolding candidates.

### Understand intent

1. Read every file that exists: README, docs, manifest files, any configuration.
2. Identify the intended tech stack from any available signals: language mentioned in README, manifest file type, framework references.
3. If intent is ambiguous, note the ambiguity in the candidate list and pick the most likely interpretation based on available evidence.

### Generate scaffolding candidates

Emit each scaffolding step as a separate P2 candidate:

1. **Project structure** -- Create the directory layout appropriate for the detected stack (e.g., `src/`, `tests/`, `docs/` for a library; `app/`, `tests/`, `public/` for a web app).
2. **Package manifest** -- Initialize the package manager and create the manifest file (`package.json`, `Cargo.toml`, `pyproject.toml`, etc.) with project metadata.
3. **Linter configuration** -- Set up the standard linter for the stack with recommended rules.
4. **Formatter configuration** -- Set up the standard formatter for the stack.
5. **Test framework** -- Configure the test runner and create a minimal passing test to verify the setup works.
6. **CI pipeline** -- Create a GitHub Actions workflow (or equivalent) that runs build, lint, and test on push.
7. **Type checking** -- Configure static type checking if the language supports it.
8. **Initial architecture** -- Create the entry point and core module structure based on what the README or docs describe.
9. **Core functionality** -- Implement the most basic described behavior so the project does something when run.

Omit any scaffolding step that already exists (e.g., if a `package.json` is already present, do not emit a manifest candidate).

---

## Candidate Output Format

Produce a numbered list of all candidates from the analysis. Sort by priority tier (P1 first), then by estimated impact within the same tier. Each candidate follows this format:

```
1. [P1] Fix failing test in auth module -- tests/auth.test.ts fails with "Cannot read property 'id' of undefined". 1-2 files.
2. [P1] Fix build error in parser module -- `cargo build` fails with lifetime mismatch in parser/mod.rs. 1 file.
3. [P2] Add test infrastructure -- no test framework configured, no test files exist. 3-4 files.
4. [P2] Configure linter -- no linter present. Using oxlint with typescript and import plugins recommended. 2 files.
5. [P3] Fix swallowed exception in error handler -- src/api/handler.ts catches all errors and returns 200. 1 file.
6. [P4] Remove dead code in utils module -- 3 unused functions and 12 unused imports across src/utils/. 4 files.
7. [P4] Update outdated dependencies -- 8 packages outdated, 2 with major version bumps. 1 file.
8. [P5] Add YAML parsing to config loader -- JSON parsing exists, YAML is referenced in README but not implemented. 2 files.
```

Include:
- Priority tag in brackets
- One-sentence description of the problem
- Brief context (error message, file location, or scope)
- Estimated file count

Exclude:
- Items already completed in the manifest
- Items previously considered and rejected
- Items covered by an open escalated PR

---

## Tech-Stack-Specific Patterns

For detailed detection patterns, check commands, and common issues organized by language and framework, see `references/analysis-patterns.md`.
