---
name: herdr-hunk-walkthrough
description: Open a branch or pull request’s full changes in Hunk inside a 50/50 Herdr split or a new Herdr tab, then add a numbered, narrative code walkthrough. Use when the user asks to review, explain, or walk through a diff/PR in Hunk on Herdr, including /herdr-hunk-walkthrough tab.
compatibility: Requires a Herdr-managed pane plus git, gh, herdr, hunk, and jq on PATH.
---

# Herdr Hunk Walkthrough

Turn a finished changeset into a guided Hunk review: one Herdr-hosted Hunk surface, the complete diff, and a small set of numbered agent notes that tell the implementation story. Arrange Hunk's file stream in that same story order so next-comment navigation advances monotonically.

## Boundaries

- This is a changeset walkthrough, not a full-codebase audit. Do not create `REVIEW.md`, a handbook, or a review branch.
- If the user asked to resolve PR feedback first, finish that loop, push the clean result, and only then open Hunk.
- Once the walkthrough starts, leave source code unchanged unless the user flags a fix.
- Preserve panes and comments you did not create.

## Mode

Default to `split` mode: open Hunk in a 50/50 pane split beside the caller. If the invocation includes a standalone `tab` token, as in `/herdr-hunk-walkthrough tab`, use `tab` mode: open Hunk in a new Herdr tab instead of splitting the current tab. A PR number, branch name, or other work item may follow the mode token.

If the user changes mode after surface creation, preserve the Hunk process whenever it has started. For split → tab after launch, move the live pane and parse its new pane/tab IDs:

```bash
herdr pane move <hunk-pane-id> --new-tab --workspace "$HERDR_WORKSPACE_ID" --label "Hunk: $(basename "$PWD")" --focus
```

Do not launch a duplicate Hunk. If Hunk has not started, inspect the obsolete split first with `herdr pane process-info --pane <created-split-pane-id>`; close it only when it is still an empty shell and only after the requested tab exists. Close only pane/tab IDs created by this run; never clean up a reused or user-owned surface.

## 1. Pin the changeset

1. Confirm a clean worktree with `git status -sb`. If dirty files are part of unfinished work, finish or clarify them before presenting the diff.
2. For any PR — open, closed, or merged — read immutable endpoint OIDs and the PR number:

```bash
gh pr view <pr> --json number,url,state,baseRefName,baseRefOid,headRefName,headRefOid,mergeStateStatus
```

Do not assume the PR branch still exists or that local `HEAD` is its head. Fetch GitHub's pull ref plus the base history, then pin both exact OIDs under private non-branch refs:

```bash
git fetch origin <base-ref-name> refs/pull/<pr-number>/head
git cat-file -e '<base-ref-oid>^{commit}' || git fetch origin <base-ref-oid>
git cat-file -e '<head-ref-oid>^{commit}' || git fetch origin refs/pull/<pr-number>/head
git update-ref refs/herdr/pr-<pr-number>-base <base-ref-oid>
git update-ref refs/herdr/pr-<pr-number>-head <head-ref-oid>
test "$(git rev-parse refs/herdr/pr-<pr-number>-base)" = "<base-ref-oid>"
test "$(git rev-parse refs/herdr/pr-<pr-number>-head)" = "<head-ref-oid>"
```

Set the review range to `refs/herdr/pr-<pr-number>-base...refs/herdr/pr-<pr-number>-head`. These refs change only Git metadata; never check out or rewrite source files to reconstruct a merged PR.

3. For a non-PR branch/range, resolve both endpoints to immutable commit OIDs using the user's target or the merge-base with the default branch, and use `<base-oid>...<head-oid>`.
4. Record `review_range`, the changed-file list, and diffstat from `git diff "$review_range"`. Done when the immutable range names exactly the intended changes, including a merged PR whose source branch was deleted.

## 2. Establish Herdr

Verify the caller is inside Herdr before controlling layout:

```bash
test "${HERDR_ENV:-}" = 1
```

If false, stop and tell the user this workflow must run from a Herdr-managed pane.

Resolve these bundled files relative to this `SKILL.md` and keep their absolute paths for the run:

- `extensions/walkthrough-order.ts` — passive Hunk transform that applies an exact, unexpired file order
- `scripts/write-order.mjs` — validates and writes that order under the checkout's private Git metadata

Never substitute a repo-controlled extension. Hunk extensions execute with the user's permissions.

Read the installed CLI rather than assuming syntax:

```bash
herdr tab
herdr pane
herdr tab list --workspace "$HERDR_WORKSPACE_ID"
herdr pane current --current
herdr pane layout --pane "$HERDR_PANE_ID"
```

Look for an existing Hunk pane/session for this repo before creating one. Hunk's list JSON can be large, so capture it before rendering a repo-filtered, row-bounded projection:

```bash
repo_root="$(git rev-parse --show-toplevel)"
before_sessions="$(mktemp)"
hunk session list --json > "$before_sessions"
jq -r --arg repo "$repo_root" '
  limit(50; .sessions[]
    | select(.repoRoot == $repo)
    | [.sessionId, .repoRoot, .title, (.pid|tostring), (.fileCount|tostring)]
    | @tsv)
' "$before_sessions"
herdr pane list --workspace "$HERDR_WORKSPACE_ID"
```

Keep the snapshot until a new session is identified. The projection is the only session-list output needed: at most 50 rows containing session ID, repo, title, PID, and file count.

- Reuse a live Hunk session for this repo only when it already matches the requested surface **and was launched by this workflow with the bundled walkthrough-order extension**. Extension registration happens only at process startup; when provenance is uncertain, preserve the session and create the requested surface for this run. If `tab` was requested and the live session is in a split, leave it alone and create a tab surface.
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

This workflow is the deliberate exception to the generic Hunk-review rule that asks the user to launch Hunk: start the interactive TUI only through `herdr pane run` in its user-visible surface, never directly in the agent's terminal.

For a new pane, start Hunk through Herdr with the immutable range:

```bash
herdr pane run <hunk-pane-id> "hunk diff '$review_range' --mode auto --extension '<absolute-walkthrough-extension>'"
```

Wait for registration, capture a second session snapshot, and identify exactly one new session for this repo by set difference:

```bash
after_sessions="$(mktemp)"
pane_processes="$(mktemp)"
hunk session list --json > "$after_sessions"
herdr pane process-info --pane <hunk-pane-id> > "$pane_processes"
hunk_session_id="$(jq -r --arg repo "$repo_root" \
  --slurpfile before "$before_sessions" \
  --slurpfile pane "$pane_processes" '
  ($before[0].sessions | map(.sessionId)) as $known
  | ($pane[0].result.process_info.foreground_processes | map(.pid)) as $pane_pids
  | .sessions[]
  | select(.repoRoot == $repo)
  | select((.sessionId as $id | $known | index($id)) == null)
  | select(.pid as $pid | ($pane_pids | index($pid)) != null)
  | .sessionId
' "$after_sessions")"
test "$(printf '%s\n' "$hunk_session_id" | sed '/^$/d' | wc -l | tr -d ' ')" = 1
```

Always require both proofs: the session is new since the first snapshot **and** its PID belongs to the created Herdr pane. If no ID appears yet, repeat both the session and pane-process snapshots after a short delay. Never accept a same-repo candidate by timing or repo alone.

For a reused session, select its exact ID from the bounded discovery projection before reloading it:

```bash
hunk_session_id='<selected-session-id>'
hunk session reload "$hunk_session_id" -- diff "$review_range"
```

From this point forward, use the captured session ID for **every** Hunk command. Never fall back to `--repo`: more than one live session may share a repo root.

Verify the exact session and review structure:

```bash
hunk session get "$hunk_session_id" --json
hunk session review "$hunk_session_id" --json
```

The Hunk file set must equal `git diff --name-only "$review_range"`. Reload the same session ID if it does not. Record Hunk's current file order; unannotated files retain this relative order later. Stop reading the session/process snapshots after the ID is pinned; host temporary-file cleanup owns them.

## 4. Build and order the walkthrough

Read the review structure first. Inspect raw patches or local files only for the architectural waypoints you need.

Create **6–10 draft comments** that trace one end-to-end path through the change. Prefer this order:

1. entry/admission
2. orchestration or state transition
3. core behavior
4. trust or authorization boundary
5. side effects and idempotency
6. tests that prove the contract

Make the narrative realizable as one top-to-bottom Hunk stream:

- Each annotated file occupies one contiguous waypoint block; do not leave a file and return to it later.
- Within one file, targets advance by rendered source position. Use at most one waypoint per hunk; consolidate comments when the conceptual order would otherwise move backward or share an anchor.
- Order annotated file blocks by the implementation story. Append every unannotated file in its original Hunk order.
- Only after this normalization, number summaries `N/total — <one-sentence waypoint>` in stream order.

Each rationale explains what happens, why this seam owns it, and the invariant or risk to notice. Keep comments instructional rather than evaluative. Do not annotate every hunk. Target changed lines when possible; use `hunkNumber` when the key symbol is unchanged context inside a changed hunk.

Before applying comments, write the complete deduplicated file order — annotated blocks first, then unannotated files — through the bundled helper:

```bash
printf '%s\n' '{"files":["first/story/file.ts","second/story/file.ts","remaining/file.ts"]}' \
  | node <absolute-write-order-script> --repo "$PWD"
```

The helper stores a 24-hour sidecar under private Git metadata, so it does not dirty the worktree. Reload the pinned range to run the transform, then verify exact order:

```bash
hunk session reload "$hunk_session_id" -- diff "$review_range"
hunk session review "$hunk_session_id" --json
```

The returned file paths must exactly equal the order sidecar. A mismatch means the session did not load the bundled extension; do not apply comments to a misordered stream. Preserve that session, start a fresh Hunk process with `--extension <absolute-walkthrough-extension>`, and replace `hunk_session_id` using the same before/after set-difference procedure before continuing.

Once order is verified, use `hunk session comment apply "$hunk_session_id" --stdin` once with the validated JSON batch. List comments from that same session ID afterwards and confirm their numbered summaries appear in ascending order under Hunk's next-comment navigation.

If a numbered walkthrough already exists for the loaded range, inspect and reuse it instead of duplicating notes. Still verify that its numbering, file order, and within-file target order agree; reorder the stream when needed without duplicating comments.

Done when every major behavior belongs to one waypoint, Hunk's file stream matches the narrative order, and next-comment navigation advances `1/total` through `total/total` without jumping backward.

## 5. Hand control to the user

Navigate to waypoint 1:

```bash
hunk session navigate "$hunk_session_id" --file <first-file> --hunk <n>
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
hunk session context "$hunk_session_id" --json
hunk session comment list "$hunk_session_id"
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
