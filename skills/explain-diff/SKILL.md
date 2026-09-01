---
name: explain-diff
description: Explain a diff, staged changes, a commit, a branch, or a PR in plain language as a fixed teaching spine — the problem, the moving parts, the one key trick, a concrete end-to-end trace, hidden infrastructure, and what the tests guarantee. Use when the user asks to "explain this PR", "explain the diff", "walk me through these changes", "what does this change do", or says they don't understand a change. Also the spine source for visual-diff.
---

# Explain a diff

Explain code changes so a smart reader unfamiliar with this codebase actually gets it: ground-up, plain language, one concrete example traced end-to-end. Group by concept, never walk files top-to-bottom.

## 1. Get the changes

Pick the command matching what they mean by "the changes":

- Working tree: `git diff` plus `git diff --cached`
- Staged: `git diff --cached`
- Commit: `git show <sha>`
- Branch: `git diff $(git merge-base <default-branch> HEAD)...HEAD`
- Range: `git diff <from>..<to>`
- PR: `gh pr view <n>` then `gh pr diff <n>`

With no explicit target, prefer uncommitted changes when present, otherwise the branch against the default branch. Record the resolved endpoints so the comparison is unambiguous.

Then read the changed files themselves, not just the hunks, plus the callers and tests around them, and find the why: the linked issue, PR body, ADR, plan, or commit message. Label rationale inferred only from code as an inference. Never explain a change you have not understood.

**Complete when:** the exact before and after revisions and the changed-file set are known, and every claim you plan to make has a file and line behind it.

## 2. The teaching spine

Always this order. It is the single source for every explain-shaped skill in this repo.

1. **The problem.** Why this change exists at all. One or two sentences, no code yet.
2. **The moving parts.** Two to four conceptual pieces with plain names. A concept list, not a file list.
3. **The one key trick.** The single insight that makes it click: a method, a flag, a pattern, an inversion. Isolate it.
4. **Trace one concrete example.** A realistic input followed through each part, showing what happens to it at each step. Include one believable edge value when it reveals important behavior.
5. **The hidden infrastructure.** The part a newcomer would never guess: reused framework behavior, storage, runtime, generated code, implicit coupling.
6. **What the tests guarantee.** Each relevant test restated as a promise the system now keeps, not a description of what the test does. Distinguish tests read from tests actually run.
7. **Zoom in.** Three or four labeled sticking points the reader can pick from, and a live demo when the code is runnable.

## 3. Tone

- Plain language. Expand jargon on first use.
- One analogy per hard concept, at most.
- A traced example beats abstract prose.
- Use the repository's own names for domain concepts; read `CONTEXT.md` first when it exists. Invent no replacements.
- End by asking which part is still fuzziest.

## Worked shape

> **Problem:** sources emit different formats; we need one internal shape that never drops fields.
> **Parts:** (1) the shape, (2) the intake endpoint, (3) where it is stored.
> **Key trick:** `.passthrough()` keeps unknown fields instead of deleting them.
> **Trace:** POST `{..., weirdField: 123}` → validated → stored → read back; `weirdField` still there after a restart.
> **Hidden infra:** storage is the framework's built-in run log, not a new table.
> **Tests guarantee:** (1) unknown fields survive validation; (2) they survive a server restart.
> **Zoom in?** (a) what a workflow is (b) how logging counts as saving (c) why passthrough matters
