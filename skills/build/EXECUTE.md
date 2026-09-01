# Build Execute

`/build execute` delegates implementation to one executor subagent. The main agent resolves the work item, dispatches the executor, then reviews the result; it does not edit code itself in execute mode.

## Invocation

Syntax:

```text
/build execute [modelProvider/model] [on <thinking> thinking] [work item args]
```

Examples:

```text
/build execute
/build execute openai-codex/gpt-5.5 on xhigh thinking
/build execute anthropic/claude-sonnet-4 on max thinking #123
```

Parse the first token after `execute` containing `/` as the executor model spec. Parse `on <thinking> thinking` as the requested thinking/effort level and remove those words from the work item args. If no thinking level is supplied, default the executor to `high` thinking. Treat remaining args as the issue, ticket, plan, or context pointer. If the host cannot honor the requested model or thinking level, stop and ask; do not silently switch executors.

## Preconditions

- The host must support subagents in isolated git worktrees. If not, say so and ask whether to run normal `/build` instead.
- Resolve the work item using normal `/build` rules before dispatch: issue, ticket, `/build plan`, PRD/handoff, current conversation plan, or a confirmed contract. In contract mode, pass the contract block through to the executor unchanged; do not re-resolve the issue or create a branch.
- Confirm the work is narrow enough for one reviewable PR and that unrelated dirty files are protected.
- Capture repo state: current branch, default branch, remote, dirty files, and intended `{type}/{short-description}` branch.

## Dispatch

Spawn one executor subagent in an isolated worktree. Pass the requested `modelProvider/model` and requested or default `high` thinking level to the host subagent facility when supported.

Inline all needed context in the prompt:

- full issue, ticket, or plan text
- acceptance criteria and chunk plan
- in-scope and out-of-scope files when known
- branch, commit, force-push, and verification rules from `SKILL.md`
- the autoreview engine rules from `SKILL.md`, or in contract mode the instruction to skip autoreview because the caller reviews the whole diff

Executor preamble:

```text
You are the executor for the work item below. Follow it step by step.
Touch only scoped files. Commit each implementation round with a
conventional-commit subject unless a Hunk review is active; if Hunk is
reviewing the diff, stop before staging/committing and report that blocker.
Never publish, merge, close, or relabel anything.
Run focused verification at the end of each implementation round, plus
structured autoreview unless the contract mode instruction says to skip it.
When autoreview is yours to run and it cannot run, report the exact blocker
and fallback used. Before reporting, audit every claim against actual tool
output from this session.
```

Report format:

```text
STATUS: COMPLETE | STOPPED
BRANCH:
COMMITS:
STEPS: per chunk, with verification result
AUTOREVIEW: command, engine/model, result
FILES CHANGED:
STOPPED BECAUSE: only if stopped
NOTES:
```

## Review

Treat the executor diff as untrusted.

The main agent must:

1. Inspect the full diff and verify every hunk maps to the work item.
2. Reject out-of-scope files unless the executor documented a necessary deviation.
3. Re-run focused tests or done criteria when practical.
4. Confirm the executor ran autoreview, handled accepted findings, and reported the command. In contract mode, confirm instead that it skipped autoreview and left review to the caller.
5. Check commits are conventional, round-scoped, and avoid forbidden force-push behavior.

Do not replace a missing executor autoreview with a main-agent autoreview. Send the executor back to run it.

## Verdict

- `APPROVE` — scope is clean, verification passes, commits are sane, and autoreview is clean or consciously rejected findings are acceptable.
- `REVISE` — fixable gaps. Send precise feedback to the same executor. Max two revision rounds.
- `BLOCK` — missing subagent support, stale or unclear work item, scope drift, repeated failed revisions, or executor cannot run autoreview.

Never merge or publish unless the user explicitly asks.
