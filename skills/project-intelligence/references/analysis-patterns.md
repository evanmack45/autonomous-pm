# Analysis Patterns by Tech Stack

Stack-specific detection patterns, check commands, and common issues. Use these during Steps 1-4 of the existing project analysis.

---

## Node.js / TypeScript

### Test Detection

Check for test framework configuration in this order:

1. Look for `vitest.config.ts`, `vitest.config.js`, or a `vitest` key in `package.json`. If found, run: `npx vitest run`.
2. Look for `jest.config.ts`, `jest.config.js`, `jest.config.mjs`, or a `jest` key in `package.json`. If found, run: `npx jest`.
3. Look for a `test` script in `package.json`. If found, run: `npm test` or `pnpm test`.
4. Check for test files matching `**/*.test.{ts,js,tsx,jsx}` or `**/*.spec.{ts,js,tsx,jsx}` or a `__tests__/` directory.

If no framework is configured but test files exist, note the mismatch as a P2 candidate.

### Lint Detection

Check for linter configuration:

1. Look for `oxlint` in `package.json` devDependencies or an `oxlintrc.json` file.
2. Look for `biome.json` or `biome.jsonc`.
3. Look for `.eslintrc.*` files or an `eslintConfig` key in `package.json`.
4. Check for a `lint` script in `package.json`.

Run the detected linter. If it produces warnings or errors, categorize them: fixable automatically (not a candidate -- just fix them) vs. requiring manual decisions (emit a candidate per category).

### Type Safety

Check `tsconfig.json` for strict mode settings:

- `"strict": true` -- baseline requirement
- `"noUncheckedIndexedAccess": true` -- prevents unsafe array/object access
- `"exactOptionalPropertyTypes": true` -- prevents `undefined` assignment to optional props
- `"noImplicitOverride": true` -- requires explicit override keyword
- `"verbatimModuleSyntax": true` -- enforces explicit type imports

If `tsconfig.json` exists but `strict` is false or missing, emit a P2 candidate to enable strict mode. If `strict` is true but additional strictness flags are missing, emit a P4 candidate.

### Common Issues

Search for these patterns during the code quality scan:

- **`any` type usage** -- Search for `: any`, `as any`, `<any>`. Each file with `any` usage is a quality signal. Emit one candidate if the count exceeds 5 occurrences, describing the worst offenders.
- **Missing async error handling** -- Search for `async` functions that lack try/catch or `.catch()` on promise chains. Focus on HTTP handlers, middleware, and entry points.
- **`console.log` in production code** -- Search for `console.log`, `console.warn`, `console.error` outside of test files. Distinguish between intentional logging (structured logger) and debug leftovers.
- **Implicit `any` from untyped imports** -- Check for `@ts-ignore` or `@ts-expect-error` comments that suppress type errors rather than fixing them.
- **Missing `await`** -- Search for async function calls whose return value is not awaited, especially in express/fastify handlers.
- **Non-null assertions** -- Search for `!` postfix operator usage. High density indicates type system gaps.

### Build Detection

1. Check for a `build` script in `package.json`. Run: `npm run build` or `pnpm build`.
2. Look for `tsconfig.build.json` or `tsconfig.json` with `outDir`. Run: `npx tsc --noEmit` for type checking without build.
3. Look for bundler config: `vite.config.ts`, `webpack.config.js`, `rollup.config.js`, `esbuild` in scripts.

---

## Python

### Test Detection

Check for test infrastructure in this order:

1. Look for `pytest` in `pyproject.toml` dependencies or dev-dependencies, `setup.cfg`, or `pytest.ini`. If found, run: `pytest -q`.
2. Look for a `tests/` directory with files matching `test_*.py` or `*_test.py`.
3. Look for `unittest` imports in test files (less common in modern projects).
4. Check for a `test` script in `pyproject.toml` or `Makefile`.

If test files exist but pytest is not in dependencies, note the missing dependency as part of the P2 candidate.

### Lint Detection

Check for linter configuration:

1. Look for `[tool.ruff]` in `pyproject.toml` or `ruff.toml`. If found, run: `ruff check .`.
2. Look for `[tool.flake8]` in `setup.cfg` or `.flake8`. If found, note that migration to ruff is a P4 candidate.
3. Look for `[tool.pylint]` or `.pylintrc`. Same migration note.

Run the detected linter and categorize output the same way as Node.js.

### Type Safety

Check for type checking configuration:

1. Look for `[tool.ty]` in `pyproject.toml`. If found, run: `ty check`.
2. Look for `[tool.mypy]` in `pyproject.toml` or `mypy.ini`. If found, run: `mypy .`.
3. Look for `[tool.pyright]` in `pyproject.toml` or `pyrightconfig.json`.
4. Check if `py.typed` marker exists (for library projects).

If the project uses type hints in source but has no type checker configured, emit a P2 candidate.

### Common Issues

Search for these patterns:

- **Bare `except`** -- Search for `except:` without an exception type. This catches `KeyboardInterrupt` and `SystemExit`, which is almost never intended. Also check for `except Exception:` that silently passes.
- **Mutable default arguments** -- Search for function definitions with `def func(items=[])` or `def func(data={})`. These share state across calls.
- **Missing `__init__.py`** -- Check package directories for missing init files. In modern Python (3.3+) namespace packages work without them, but explicit init files are clearer.
- **`import *` usage** -- Search for wildcard imports that pollute the namespace and make dependencies unclear.
- **String formatting inconsistency** -- Check for mixed usage of f-strings, `.format()`, and `%` formatting. The dominant style should win.
- **Missing `if __name__ == "__main__"` guard** -- Check entry point scripts for unguarded top-level execution.
- **`os.path` vs `pathlib`** -- Check for mixed path handling. Modern Python prefers `pathlib.Path`.

### Build Detection

1. Check for `pyproject.toml` with a `[build-system]` section. Run: `uv build` or `python -m build`.
2. Check for `setup.py` or `setup.cfg` (legacy). Note migration to `pyproject.toml` as a P4 candidate.
3. For applications (not libraries), check for a working entry point: `python -m <package>` or a console script.

---

## Rust

### Test Detection

Rust has built-in test support. Check for:

1. Search for `#[test]` annotations in source files. Run: `cargo test`.
2. Check for a `tests/` directory containing integration tests.
3. Check for `#[cfg(test)]` modules within source files (unit tests).
4. Look for `proptest` or `quickcheck` in `Cargo.toml` for property-based tests.

If `cargo test` succeeds but the project has no `#[test]` annotations and no `tests/` directory, the test suite is empty. Emit a P2 candidate.

### Lint Detection

Clippy is the standard Rust linter. Check:

1. Look for `[lints.clippy]` in `Cargo.toml`. If present, run: `cargo clippy --all-targets --all-features -- -D warnings`.
2. If no clippy configuration exists, run clippy with defaults and emit a P2 candidate to configure clippy lints in `Cargo.toml`.
3. Check for `#[allow(...)]` annotations. High density may indicate suppressed warnings rather than fixed issues.

### Type Safety

Rust is statically typed by default. Focus instead on:

- **Strict lints** -- Check whether `Cargo.toml` configures pedantic clippy lints. If not, emit a P4 candidate.
- **Unsafe usage** -- Search for `unsafe` blocks. Each one should have a `// SAFETY:` comment explaining the invariant. Missing safety comments are a P3 candidate.

### Common Issues

Search for these patterns:

- **`unwrap()` in non-test code** -- Search for `.unwrap()` and `.expect()` outside of `#[test]` functions and `#[cfg(test)]` modules. Each occurrence in production code is a panic risk. Emit one candidate per module with this problem.
- **Missing `?` operator** -- Search for manual `match` on `Result` or `Option` that could use `?` for propagation. This is a readability issue, not a bug.
- **Unused `Result`** -- Search for function calls that return `Result` but where the return value is not bound or propagated. The compiler warns about this (`#[must_use]`), but the warning may be suppressed.
- **`clone()` overuse** -- Search for `.clone()` calls. High density may indicate ownership issues being papered over rather than solved.
- **`todo!()` and `unimplemented!()`** -- Search for these macros in non-test code. Each one is a potential panic at runtime.
- **Large functions** -- Check for functions exceeding 100 lines. Rust functions tend to be shorter; long ones often indicate missing abstractions.
- **Missing `#[must_use]`** -- Check public functions that return `Result` or `Option` for the `#[must_use]` attribute.

### Build Detection

1. Run: `cargo build`. Check for compilation errors.
2. Run: `cargo build --all-features` to verify feature flag combinations compile.
3. Check for `cargo-deny` in CI or dev dependencies. If absent, note it as a P4 candidate for supply chain auditing.

---

## Universal Checks

Apply these regardless of tech stack.

### CI Detection

Check for continuous integration configuration:

1. Look for `.github/workflows/` directory with YAML files. Read each workflow to understand what it runs.
2. Look for `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/config.yml`, `.travis.yml`.
3. If CI exists, verify it runs at minimum: build, lint, and test. Missing steps are P2 candidates.
4. If no CI exists at all, emit a P2 candidate: "Add CI pipeline."

Check CI configuration quality:

- Actions pinned to SHA hashes (not tags): `uses: actions/checkout@<sha> # vX.Y.Z`
- `persist-credentials: false` on checkout steps
- No secrets in workflow files

### Security Basics

Check for fundamental security hygiene:

1. **No hardcoded secrets** -- Search for patterns that look like API keys, tokens, passwords, or connection strings in source files. Common patterns:
   - Strings matching `sk-`, `pk-`, `ghp_`, `gho_`, `xoxb-`, `xoxp-` (known API key prefixes)
   - Variables named `password`, `secret`, `token`, `api_key` assigned to string literals
   - Connection strings with embedded credentials
   If found, emit a P1 candidate (this is a security issue, not just quality).

2. **`.env` in `.gitignore`** -- Check that `.env`, `.env.local`, `.env.production`, and similar files are gitignored. If not, emit a P2 candidate.

3. **Pinned dependencies** -- Check that dependencies use exact versions (not ranges) and that a lockfile exists (`package-lock.json`, `pnpm-lock.yaml`, `Cargo.lock`, `uv.lock`, `poetry.lock`). Missing lockfile is a P2 candidate.

4. **No secrets in git history** -- Run `git log --oneline --all --diff-filter=A -- "*.env" ".env*"` to check if environment files were ever committed. If found, note that history contains secrets (this requires a more complex remediation -- escalate rather than fix).

### Documentation

Check for basic project documentation:

1. **README.md** -- Must exist. Check that it covers: project description, setup instructions, usage, and how to run tests. Missing sections are P4 candidates.
2. **LICENSE** -- Check for a license file. If the project is on GitHub and has no license, emit a P4 candidate noting that unlicensed code is "all rights reserved" by default.
3. **CHANGELOG or release notes** -- Not required, but if the project has published releases, check that some form of change documentation exists.

### Git Hygiene

Check the repository state:

1. **`.gitignore` completeness** -- Verify coverage for the detected stack. Common misses:
   - Node: `node_modules/`, `dist/`, `.next/`, `.turbo/`, `*.tsbuildinfo`
   - Python: `__pycache__/`, `*.pyc`, `.venv/`, `*.egg-info/`, `.ruff_cache/`
   - Rust: `target/`
   - Universal: `.env*`, `.DS_Store`, `*.log`, `.idea/`, `.vscode/` (unless shared settings are intentional)

2. **Large files** -- Check for binary files, media assets, or data files committed to the repo that should use Git LFS or be gitignored.

3. **Stale branches** -- Run `git branch --merged` to find branches already merged into the default branch. These are safe to clean up but are a low-priority P4 candidate.
