---
name: explain-diff
description: Explain a diff, staged changes, a branch, or a PR in plain language — what problem it solves, the one key insight, and a concrete end-to-end trace of real data. Use when the user asks to "explain this PR", "explain the diff", "explain the staged/branch changes", "walk me through these changes", "what does this change do", or says they don't understand a change.
---

# Explain a Diff / PR

Explain code changes so a smart reader unfamiliar with this codebase actually
gets it: ground-up, plain language, one concrete example traced end-to-end.
Group by concept, never walk files top-to-bottom.

## 1. Get the changes

Pick the command matching what they mean by "the changes":

- Working tree: `git diff`
- Staged: `git diff --cached`
- Branch vs base: `git diff $(git merge-base main HEAD)..HEAD --stat`, then read files
- A commit: `git show <sha>`
- A PR: `gh pr view <n>` and `gh pr diff <n>`

Then **read the changed files themselves** (not just the diff) and find the
*why*: the linked issue, PR body, ADR, or commit message. Never explain a change
you haven't understood.

## 2. Explain with this structure (always this order)

1. **The problem** — why this change exists at all. 1–2 sentences, no code yet.
2. **The moving parts** — group the change into 2–4 conceptual pieces, named in
   plain terms. A concept list, not a file list.
3. **The one key trick** — the single insight that makes it click (a method, a
   flag, a pattern, an inversion). Call it out explicitly and isolate it.
4. **Trace one concrete example** — take a realistic input and follow it through
   each part, showing what happens to it at each step. Include one made-up but
   believable value to demonstrate the edge behavior (e.g. an unknown field).
5. **The non-obvious infrastructure** — name the part a newcomer would never
   guess (e.g. "we don't build our own store, we reuse X").
6. **What the tests prove** — restate each test as a plain-language *guarantee*,
   not a description of what it does.
7. **Offer to zoom in** — list the 3–4 likeliest sticking points as labelled
   options (a/b/c/d) and offer a live demo if the code is runnable.

## 3. Tone rules

- Plain language. Expand jargon on first use ("a workflow — basically an HTTP endpoint").
- One analogy per hard concept.
- Prefer a traced example over abstract prose.
- Group by what the code *does*, never file-by-file top-to-bottom.
- End by asking which part is fuzziest — don't assume it all landed.

## Worked shape (example output skeleton)

> **Problem:** sources emit different formats; we need one internal shape that never drops fields.
> **Parts:** (1) the shape, (2) the intake endpoint, (3) where it's stored.
> **Key trick:** `.passthrough()` keeps unknown fields instead of deleting them.
> **Trace:** POST `{…, weirdField: 123}` → validated → stored → read back, `weirdField` still there even after restart.
> **Hidden infra:** storage is the framework's built-in run log, not a new table.
> **Tests prove:** (1) unknown fields survive validation; (2) they survive a server restart.
> **Zoom in?** (a) what a workflow is (b) how "logging" counts as saving (c) why passthrough matters
