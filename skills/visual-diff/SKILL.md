---
name: visual-diff
description: Explain a working-tree diff, staged changes, commit, branch, range, or pull request as a polished self-contained HTML visual story. Use when the user asks for a visual diff, visual PR walkthrough, diagrammed code-change explanation, before/after architecture view, or an interactive explanation of what changed and why.
---

# Visual Diff

Turn code changes into a verified visual explanation that does not assume prior knowledge of the codebase. Make the narrative do the teaching and the visuals make the relationships obvious.

## Compose the source skills

Load and follow both source skills before gathering data:

1. `$explain-diff` supplies the explanation contract: problem, conceptual moving parts, one key trick, a concrete trace, hidden infrastructure, and test guarantees.
2. `$visual-explainer:visual-explainer` supplies the HTML workflow, templates, responsive navigation, Mermaid controls, styling constraints, browser delivery, and quality checks.

Treat this skill as the integration layer. Where the source skills differ, preserve `explain-diff`'s explanation-first narrative and `visual-explainer`'s rendering and verification standards.

## Communication contract

Give the reader enough context to understand where the change fits before explaining its details. Write all page copy and the chat handoff in ASD-STE100 Simplified Technical English.

Look for `CONTEXT.md` from the repository root before drafting. When it exists, read it and use its ubiquitous language exactly and consistently. When it does not exist, use terms already established in the repository's code and documentation. Do not invent replacement names for established domain concepts.

## 1. Resolve the comparison

Infer the intended scope from the request:

- Working tree: `git diff` plus `git diff --cached`
- Staged changes: `git diff --cached`
- Commit: `git show <sha>`
- Branch or base ref: diff from the merge base to `HEAD`
- Commit range: `git diff <from>..<to>`
- Pull request: read its body and metadata, then its diff with the available GitHub tooling

With no explicit target, prefer uncommitted changes when present. Otherwise compare the current branch with the repository's default base branch. Record the resolved endpoints in the page so the comparison is unambiguous.

Finish when the exact before and after revisions, inclusion of staged/unstaged changes, and changed-file set are known.

## 2. Build a fact sheet

Gather the diff summary, name/status list, full diff, and commit or PR context. Then read every changed file in full plus the surrounding callers, consumers, types, configuration, and tests needed to understand behavior. Find the why in the PR body, linked issue, ADR, plan, changelog, or commit messages. Label rationale inferred only from code as an inference.

Capture a private fact sheet containing:

- resolved comparison and file/line counts;
- each behavioral claim with supporting file and line;
- new or changed public interfaces, configuration, data shapes, and side effects;
- the before and after path through the system;
- tests and the user-visible guarantee each one establishes;
- uncertainty, missing context, and unverified assumptions.

Never derive the explanation from diff hunks alone. Finish when every substantive claim planned for the page has evidence or an explicit uncertainty label.

## 3. Find the teaching spine

Organize by behavior, not by file. Decide these before writing HTML:

1. **Problem** — why the change exists, in one or two plain-language sentences.
2. **Moving parts** — two to four conceptual pieces with plain names.
3. **Key trick** — the one method, flag, pattern, or inversion that makes the change click.
4. **Concrete trace** — one realistic value followed end-to-end through the changed system. Include one believable edge value when it reveals important behavior.
5. **Hidden infrastructure** — reused framework behavior, storage, runtime, generated code, or implicit coupling a newcomer would not guess.
6. **Guarantees** — restate relevant tests as promises the system now keeps.

Expand jargon on first use and use at most one analogy for each hard concept. Finish when a reader could understand the change from this spine without seeing a filename list.

## 4. Map the spine to a visual page

Create a scrollable self-contained HTML page by default. Use slides only when the user explicitly asks for them.

Use this information hierarchy:

1. **Hero: the problem and outcome** — lead with the intuition, scope, and resolved comparison.
2. **The key trick** — isolate the central insight as the strongest callout.
3. **Before → after** — show only the architecture, state, data shape, or interaction that materially changed. Do not manufacture a topology diagram for a local edit.
4. **Moving parts** — two to four concept cards; filenames are evidence labels, not section headings.
5. **Trace a real example** — use a Mermaid flow or sequence diagram when connections matter, and place the concrete values on the path.
6. **Hidden infrastructure** — make the non-obvious dependency or reuse explicit.
7. **What the tests guarantee** — present guarantees, evidence, and gaps; distinguish tests read from tests actually executed.
8. **Evidence and file map** — compact or collapsible reference material with clickable local file links where supported.
9. **Zoom in** — offer three or four labeled follow-up topics and a live demo when the code is runnable.

Include a review section only when the user asks for review, risks, or quality assessment. Keep factual explanation separate from review judgments.

## 5. Render and verify

Follow the visual explainer's relevant template and reference-reading rules. Use its full Mermaid zoom/pan/expand shell whenever Mermaid is present. Write the result to `~/.agent/diagrams/` with a descriptive `*-visual-diff.html` filename.

Verify before delivery:

- every page claim matches the fact sheet;
- the narrative follows the required teaching spine in order;
- the concrete trace contains real or clearly labeled illustrative values;
- visual hierarchy survives the squint test;
- narrow and wide layouts do not overflow;
- light and dark themes remain intentional;
- diagrams render with working zoom, pan, reset, and expand controls;
- the page opens without console errors.

Open the page in the product-native browser preview when available; otherwise use the operating system browser. In chat, give the file path, a one-paragraph takeaway, and the labeled zoom-in choices. End by asking which part is still fuzziest.
