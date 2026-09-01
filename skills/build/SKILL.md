---
name: build
description: Build or implement a single GitHub issue, Linear ticket, current plan, or confirmed contract as small, reviewable, test-first vertical slices. Use when the user invokes /build, /build plan, /build execute, asks to implement an issue/ticket, passes a PRD plus one agent-ready issue, or when /issue-to-pr hands over a confirmed contract for its implementation stage.
---

# Build

Build one agent-ready work item in small, reviewable chunks.

It has two entry modes. Which one applies decides which steps run:

- **Standalone**: the user invokes `/build`, `/build plan`, `/build execute`, or passes an issue, ticket, or PRD. Build owns everything from resolving the work item to the close-out report.
- **Contract mode**: `/issue-to-pr` (or the user) hands over a confirmed contract. Build implements only. The caller has already pinned the base, created the branch, and owns review and publishing.

A contract is present when the invocation includes a block with `base_sha`, `branch`, in-scope paths, acceptance criteria, and a test seam. Anything less is standalone.

## Contract

- Work one issue, ticket, plan, or contract at a time.
- Treat `/build plan` as instructions to act on the plan just made in conversation. Otherwise the issue body, linked PRD, plan, acceptance criteria, or contract is the source of truth.
- If the work item is too broad for one reviewable PR, stop and propose a split instead of silently doing a sprawling implementation. In contract mode, report this as a blocker; the contract is the caller's to change.
- Keep changes scoped to the ticket or the contract's in-scope paths. Preserve unrelated dirty files.
- Implement test-first at the agreed seam, following `/tdd`: one failing test, the minimal code that passes it, then the next vertical slice. Each chunk moves real behavior end-to-end, not one horizontal layer.
- Commit each implementation round as its own conventional-commit commit before starting the next round.
- If a Hunk review is open or in progress, do not stage diffs or commit until the review is closed or the user explicitly says to proceed; staging/committing removes the working-tree review diff from Hunk.
- Never force-push ordinary changes. Force-push only after a rebase requires updating remote history, and use `--force-with-lease`, not `--force`.
- Verify with focused tests first, then broader checks when the blast radius justifies it.
- Standalone mode runs structured autoreview at the end of each round with an engine family different from the implementer. Contract mode runs none; the pipeline's code-review and apr stages review the whole diff once.
- For `/build execute`, delegate to one executor subagent and review its result; read [EXECUTE.md](EXECUTE.md).
- For a Hunk walkthrough, first verify Herdr with `test "${HERDR_ENV:-}" = 1`; when true, use the installed `herdr-hunk-walkthrough` skill and let it own Hunk session discovery, layout, and AI notes. If the skill is unavailable, report that instead of falling back to direct Hunk checks. When not in Herdr, skip the walkthrough and report that it requires a Herdr-managed pane.
- Do not publish, close, or relabel the issue unless the user asks, or they invoke a publish flow such as `/apr`.

## Workflow

Contract mode starts at step 4. Standalone mode runs every step.

### 1. Resolve the work item

For `/build plan`, skip issue/ticket lookup and use the current conversation plan plus any referenced PRD, ADR, handoff, or repo docs; if no plan is present, ask for it. Otherwise read the full issue or ticket: title, body, comments, labels/status, acceptance criteria, linked PRD/plan/ADR/handoff/parent issue, and blockers. For GitHub, use the GitHub app when available, otherwise `gh`; for Linear, use the Linear app when available.

### 2. Establish repo state

Inspect the repository before editing: `git status -sb`, current branch/default branch, remotes, dirty files, and relevant docs such as `CONTEXT.md`, ADRs, plans, or domain glossary. If on `main`, `master`, or the default branch, sync the base branch and create a `{type}/{short-description}` branch, for example `feat/add-billing-export` or `fix/handle-empty-import-rows`, where `{type}` is a conventional commit type. If already on a feature branch, stay there unless the user asked for a new branch. Never revert unrelated user changes.

### 3. Check readiness

Before coding, confirm the issue is implementable:

- acceptance criteria are clear enough
- blockers are not still open
- the requested behavior is not already implemented
- the codebase has an obvious place for the change
- the public seam tests will exercise is named, or can be named now and confirmed with the user

If it is not ready, say why and ask the smallest necessary question. If the issue is raw or under-specified, recommend `/triage` or `/grill-with-docs` rather than guessing.

### 4. Make a chunk plan

In contract mode, first confirm the checkout: `git rev-parse HEAD` must be the contract's `base_sha` or descend from it, the current branch must be the contract's `branch`, and `git status` must be clean apart from files the contract names. A mismatch is a blocker, not something to fix here.

Create a short checklist of reviewable chunks, one per acceptance criterion where possible. Each chunk should be small enough to inspect, but complete enough to compile and test. Prefer this shape:

1. Write the narrow failing test at the agreed seam.
2. Implement the smallest code path that makes it pass.
3. Wire the behavior through the real integration point.
4. Run focused verification.
5. Standalone only: run structured autoreview, fix accepted findings, rerun focused verification.
6. Commit the round with a conventional-commit subject unless a Hunk review is active.
7. Repeat for the next acceptance criterion.

Do not over-plan. Once the chunks are clear, start implementing.

### 5. Implement

Work through the chunks in order. While editing:

- follow existing repo patterns
- keep abstractions local unless the codebase already has a shared pattern
- keep naming aligned with the domain model
- update docs or plan trackers only when the repo convention or issue asks for it
- avoid opportunistic refactors outside the ticket; refactoring belongs to review, not the red-green loop
- in contract mode, touch only in-scope paths. If a change genuinely needs a path outside them, stop and report it as a scope blocker with the reason

If live code has drifted from the issue or plan, compare against current behavior and adapt narrowly. If the drift changes the product decision, pause and explain the tradeoff.

### 6. Verify and review

Run the smallest meaningful checks first: focused unit/integration tests, typecheck/lint/format when relevant, app or browser-visible verification for user-facing UI changes, and regression commands named in the issue, PRD, contract, or repo docs. If a command cannot run because of missing env, dependencies, or external services, report that as a verification limit and still run any deterministic local checks available.

**Standalone only.** After each implementation round, run `/autoreview` with an engine family different from the model that wrote the code: `--engine claude` when the implementer is an OpenAI model, `--engine codex` when it is an Anthropic model, and `--engine claude` when unknown. Pass the model `/apr` uses for that engine. If the selected engine cannot run because it is missing or exits with a tooling error before producing a review, rerun with the other engine and report the fallback in the closeout. Do not switch engines merely because findings are inconvenient.

Treat review findings as advisory: verify each accepted finding against the real code, fix accepted actionable issues, rerun focused tests, and rerun autoreview until it is clean or a remaining finding is consciously rejected.

**Contract mode.** Skip autoreview. The caller reviews the whole diff from `base_sha` once implementation is complete.

When a Hunk walkthrough is part of the round or closeout, use `herdr-hunk-walkthrough` only after confirming `HERDR_ENV=1`. Do not run `hunk skill path` or ad-hoc Hunk polling from `/build`; the walkthrough skill owns those checks. If the skill is not installed for the current agent, report that and leave Hunk untouched.

### 7. Close out

End with a concise implementation report:

- branch name, HEAD SHA, changed behavior, and important files touched
- acceptance criteria, one line each: the commit and test that prove it, or the blocker that prevents it
- tests/checks run, plus the autoreview command, engine, and result in standalone mode
- Herdr Hunk walkthrough status when running in Herdr
- skipped checks, remaining risks, follow-up work, and whether the work item appears fully satisfied

In standalone mode, if the user wants review and publish, hand off naturally to `/apr`. In contract mode, return the report to the caller and stop; `git status` must be clean.

**Complete when:** every acceptance criterion in the report has a commit and a test or an explicit blocker, focused verification passed on the final HEAD, and nothing is left uncommitted.
