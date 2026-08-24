---
name: herdr-hunk-walkthrough
description: Open a branch or pull request’s full changes in Hunk inside a 50/50 Herdr split and add a numbered, narrative code walkthrough. Use when the user asks to review, explain, or walk through a diff/PR in Hunk on Herdr.
compatibility: Requires a Herdr-managed pane plus git, gh, herdr, and hunk on PATH.
---

# Herdr Hunk Walkthrough

Turn a finished changeset into a guided Hunk review: one 50/50 Herdr split, the complete diff, and a small set of numbered agent notes that tell the implementation story.

## Boundaries

- This is a changeset walkthrough, not a full-codebase audit. Do not create `REVIEW.md`, a handbook, or a review branch.
- If the user asked to resolve PR feedback first, finish that loop, push the clean result, and only then open Hunk.
- Once the walkthrough starts, leave source code unchanged unless the user flags a fix.
- Preserve panes and comments you did not create.

## 1. Pin the changeset

1. Confirm a clean worktree with `git status -sb`. If dirty files are part of unfinished work, finish or clarify them before presenting the diff.
2. For a PR branch, read its actual base and head:

```bash
gh pr view --json url,baseRefName,headRefName,headRefOid,mergeStateStatus
```

3. Fetch the base and resolve the review range as `origin/<base>...HEAD`. For a non-PR branch, use the user’s target or the merge-base with the default branch.
4. Record the changed-file list and diffstat. Done when the range names exactly the changes the user intends to review.

## 2. Establish Herdr

Verify the caller is inside Herdr before controlling layout:

```bash
test "${HERDR_ENV:-}" = 1
```

If false, stop and tell the user this workflow must run from a Herdr-managed pane.

Read the installed CLI rather than assuming syntax:

```bash
herdr pane
herdr pane current --current
herdr pane layout --pane "$HERDR_PANE_ID"
```

Look for an existing Hunk pane/session for this repo before creating one:

```bash
hunk session list --json
herdr pane list --workspace "$HERDR_WORKSPACE_ID"
```

- Reuse a live Hunk session for this repo when present; reload it with the pinned range.
- Otherwise split the caller’s wide pane to the right at exactly half width, preserving cwd and focus:

```bash
herdr pane split --pane "$HERDR_PANE_ID" --direction right --ratio 0.5 --cwd "$PWD" --no-focus
```

Parse the new pane ID from `.result.pane.pane_id`; never guess it.

Done when `herdr pane layout` shows a `ratio: 0.5` split and no unrelated pane was replaced or closed.

## 3. Load the complete diff

For a new pane, start Hunk through Herdr:

```bash
herdr pane run <hunk-pane-id> "hunk diff origin/<base>...HEAD --mode auto"
```

For an existing current-repo session:

```bash
hunk session reload --repo "$PWD" -- diff origin/<base>...HEAD --mode auto
```

Wait for registration, then verify the source and file count:

```bash
hunk session get --repo "$PWD" --json
hunk session review --repo "$PWD" --json
```

The Hunk file list must equal `git diff --name-only origin/<base>...HEAD`. Reload if it does not.

## 4. Build the walkthrough

Read the review structure first. Inspect raw patches or local files only for the architectural waypoints you need.

Create **6–10 numbered comments** that trace one end-to-end path through the change. Prefer this order:

1. entry/admission
2. orchestration or state transition
3. core behavior
4. trust or authorization boundary
5. side effects and idempotency
6. tests that prove the contract

Each comment has:

- summary: `N/total — <one-sentence waypoint>`
- rationale: what happens, why this seam owns it, and the invariant or risk to notice

Keep comments instructional rather than evaluative. Do not annotate every hunk. Use `hunk session comment apply --repo "$PWD" --stdin` once with a validated JSON batch. Target changed lines when possible; use `hunkNumber` when the key symbol is unchanged context inside a changed hunk.

If a numbered walkthrough already exists for the loaded range, inspect and reuse it instead of duplicating notes.

Done when every major behavior in the PR belongs to one waypoint and the comments form a coherent trace without requiring chat context.

## 5. Hand control to the user

Navigate to waypoint 1 and focus Hunk:

```bash
hunk session navigate --repo "$PWD" --file <first-file> --hunk <n>
herdr pane focus --direction right --pane "$HERDR_PANE_ID"
```

Use the actual neighbor direction if the Hunk pane was reused elsewhere.

Verify:

```bash
hunk session context --repo "$PWD" --json
hunk session comment list --repo "$PWD"
herdr pane layout --pane "$HERDR_PANE_ID"
```

Report only:

- loaded range and file count
- 50/50 split and focused Hunk pane
- walkthrough comment count
- instruction to use Hunk’s next-comment navigation

The walkthrough is complete when Hunk shows the full pinned changeset, waypoint 1 is selected, agent notes are visible, and the Hunk pane has focus.
