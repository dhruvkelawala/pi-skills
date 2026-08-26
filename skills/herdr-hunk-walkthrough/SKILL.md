---
name: herdr-hunk-walkthrough
description: Open a branch or pull request’s full changes in Hunk inside a 50/50 Herdr split or a new Herdr tab, then add a numbered, narrative code walkthrough. Use when the user asks to review, explain, or walk through a diff/PR in Hunk on Herdr, including /herdr-hunk-walkthrough tab.
compatibility: Requires a Herdr-managed pane plus git, gh, herdr, and hunk on PATH.
---

# Herdr Hunk Walkthrough

Turn a finished changeset into a guided Hunk review: one Herdr-hosted Hunk surface, the complete diff, and a small set of numbered agent notes that tell the implementation story.

## Boundaries

- This is a changeset walkthrough, not a full-codebase audit. Do not create `REVIEW.md`, a handbook, or a review branch.
- If the user asked to resolve PR feedback first, finish that loop, push the clean result, and only then open Hunk.
- Once the walkthrough starts, leave source code unchanged unless the user flags a fix.
- Preserve panes and comments you did not create.

## Mode

Default to `split` mode: open Hunk in a 50/50 pane split beside the caller. If the invocation includes a standalone `tab` token, as in `/herdr-hunk-walkthrough tab`, use `tab` mode: open Hunk in a new Herdr tab instead of splitting the current tab. A PR number, branch name, or other work item may follow the mode token.

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
herdr tab
herdr pane
herdr tab list --workspace "$HERDR_WORKSPACE_ID"
herdr pane current --current
herdr pane layout --pane "$HERDR_PANE_ID"
```

Look for an existing Hunk pane/session for this repo before creating one:

```bash
hunk session list --json
herdr pane list --workspace "$HERDR_WORKSPACE_ID"
```

- Reuse a live Hunk session for this repo only when it already matches the requested surface; reload it with the pinned range and focus its existing Herdr surface. If `tab` was requested and the live session is in a split, leave it alone and create a tab surface for this run.
- In `split` mode, otherwise split the caller’s wide pane to the right at exactly half width, preserving cwd and focus:

```bash
herdr pane split --pane "$HERDR_PANE_ID" --direction right --ratio 0.5 --cwd "$PWD" --no-focus
```

Parse the new pane ID from `.result.pane.pane_id`; never guess it.

- In `tab` mode, otherwise create a new tab in the caller’s workspace, preserving cwd and keeping focus unchanged:

```bash
herdr tab create --workspace "$HERDR_WORKSPACE_ID" --cwd "$PWD" --label "Hunk: $(basename "$PWD")" --no-focus
```

Parse the Hunk tab ID from `.result.tab.tab_id` and the Hunk pane ID from `.result.root_pane.pane_id`; never guess either.

Done when the requested surface exists: `split` mode has a `ratio: 0.5` split, or `tab` mode has a labeled Hunk tab with a root pane. No unrelated pane or tab was replaced or closed.

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

Navigate to waypoint 1:

```bash
hunk session navigate --repo "$PWD" --file <first-file> --hunk <n>
```

Then focus the requested surface. In `split` mode, focus the Hunk pane from the caller; use the actual neighbor direction if the Hunk pane was reused elsewhere:

```bash
herdr pane focus --direction right --pane "$HERDR_PANE_ID"
```

In `tab` mode, focus the Hunk tab:

```bash
herdr tab focus <hunk-tab-id>
```

Verify Hunk and the requested Herdr surface:

```bash
hunk session context --repo "$PWD" --json
hunk session comment list --repo "$PWD"
```

For `split` mode:

```bash
herdr pane layout --pane "$HERDR_PANE_ID"
```

For `tab` mode:

```bash
herdr tab get <hunk-tab-id>
```

Report only:

- loaded range and file count
- requested Herdr surface: 50/50 split or new tab, and focused Hunk pane/tab
- walkthrough comment count
- instruction to use Hunk’s next-comment navigation

The walkthrough is complete when Hunk shows the full pinned changeset, waypoint 1 is selected, agent notes are visible, and the requested Herdr surface has focus.
