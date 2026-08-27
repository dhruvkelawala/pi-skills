---
name: review-ready
description: Completion gate to run before declaring any implementation, refactor, test change, API/handler, CLI command, workflow, migration, integration adapter, or agent-instruction change complete. Enforces a design contract (deep modules, narrative entry points, real seams, interface-oriented tests) and produces a review-ready report. Use automatically before handing work back, or when the user asks for the code-quality / review-ready gate.
---

# Review-ready gate

This skill is a **completion gate** for code changes. The change is not ready to
hand back until the gate report is complete and every concrete violation has
either been fixed in the changed scope or recorded as an intentional exception.

It is deliberately thin: it selects the mode, points at the governing contract,
and shapes the report. The *rules* live in a contract document, not here.

## Load the contract

Resolve the governing contract in this order and use the first that exists:

1. A project override named in the task or repo config (e.g. a path passed by
   the user, or one referenced from `AGENTS.md` / `CLAUDE.md`).
2. A repo contract at one of: `docs/agents/code-quality.md`,
   `docs/code-quality.md`, `.agents/code-quality.md`, or `CONTRACT.md`.
3. The bundled default: `contract.md` next to this file.

Then:

1. Read the resolved contract completely.
2. Identify the changed seam from the task and `git diff --name-only`.
3. If the diff falls outside the contract's `## When it applies` scope, say the
   gate is not applicable and why.

State which contract you loaded (path) in the report.

## Preflight mode

Use this **before** implementation, while the change is still being shaped.
Produce a short preflight note:

- **Changed seam**: the behavior boundary being changed.
- **Narrative entry point**: the file/function/handler/command/workflow the
  reader should be able to follow top-to-bottom.
- **Owner**: the module that should own new domain types, policy, lifecycle,
  validation, and errors.
- **Test surface**: the caller-facing interface or declarative seam that tests
  should exercise.

Then implement with that seam in view, within the contract's scope rule.

## Final gate mode

Run this **after** implementation and before the final response. Treat the
contract's review-ready gate section as the single source of truth for the gate
procedure; this skill only selects the mode and shapes the report.

When applying the contract:

- Follow the documented gate in order, including its timing, verification,
  simplification, and exception-recording rules.
- Use the four contract test names as headings for your notes: caller-knowledge,
  deletion, ownership, and test-surface.

## Required report

Include this report before declaring the work complete. For `Exceptions`, write
`None` or point to the PR-description entry required by the contract.

```md
Review-ready gate:
- Contract:
- Changed seam:
- Trace:
- Four tests:
- Simplification pass:
- Verification:
- Exceptions:
```
