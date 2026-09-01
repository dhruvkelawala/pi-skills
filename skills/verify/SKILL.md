---
name: verify
description: Discover and run every verification a project defines (tests, typecheck, lint, build, CI steps, commands named in agent instructions) and report each as passed, failed, or could-not-run. Use when the user invokes /verify, asks to run the project's checks, or a pipeline needs proof the tree is green before publishing.
---

# Verify

Run what the project itself says proves a change is good. This skill invents nothing: it finds the commands the repository already defines and runs them with fixed, non-interactive invocations.

Invoke as `/verify [--focused <path or pattern>]`. Focused mode runs only the tests that touch the given paths, for fast inner-loop feedback. Full mode is the default.

## 1. Discover

Collect candidate commands from every source that exists, in this order, and keep them all:

1. Commands named in `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, or a `docs/agents/` file under a heading such as "verification", "tests", or "checks". These are authoritative over anything inferred below.
2. CI workflow steps under `.github/workflows/`, `.gitlab-ci.yml`, or `.circleci/`: take the `run:` lines from jobs named test, lint, typecheck, check, or build.
3. Package-manager scripts: `test`, `lint`, `typecheck`, `check`, `build` from `package.json` (respect the lockfile's package manager), `Makefile` or `justfile` targets with those names, `cargo test` and `cargo clippy` when `Cargo.toml` exists, `pytest`, `ruff`, or `mypy` when `pyproject.toml` configures them, `go test ./...` and `go vet ./...` when `go.mod` exists.
4. A test runner with no script wrapper, when a test directory or test file pattern exists and nothing above covered it.

Dedupe by effect, keeping the most specific source. List the plan before running: one line per command with its source.

## 2. Run

- Run commands from the repository root unless the source names a working directory.
- Use non-interactive flags: `CI=1`, `--no-watch`, `--run`, `--ci`, or the runner's equivalent.
- Run focused tests first when `--focused` is given, then stop. In full mode run lint and typecheck before tests, and build last.
- Capture the exit status and the last relevant lines of output for each command.
- A command that cannot run because of a missing tool, service, environment variable, or network access is **could-not-run**, never a pass. Say what was missing.
- Fix nothing here. Verification reports; repair belongs to the caller.

**Complete when:** every discovered command has a recorded result, and no command was skipped without a stated reason.

## 3. Report

```md
Verification:
- <command> (<source>): passed | failed | could-not-run — <one line>
...
Result: green | red | incomplete
```

`green` means every command passed. `red` means at least one failed. `incomplete` means none failed but at least one could not run.
