---
name: visual-diff
description: Explain a working-tree diff, staged changes, commit, branch, range, or pull request as a polished self-contained HTML visual story. Use when the user asks for a visual diff, visual PR walkthrough, diagrammed code-change explanation, before/after architecture view, or an interactive explanation of what changed and why.
---

# Visual diff

The HTML rendering of `/explain-diff`. The narrative does the teaching; the visuals make the relationships obvious. This skill adds only what a page needs beyond the prose explanation: a verified fact sheet, a page hierarchy, and render checks.

## Compose the source skills

Load both before gathering data:

1. `/explain-diff` owns getting the changes and the seven-part teaching spine. Follow its step 1 and step 2 exactly; do not restate the spine here.
2. `/visual-explainer` owns the HTML workflow: templates, responsive navigation, Mermaid controls, styling constraints, browser delivery, and quality checks.

Where they differ, explain-diff wins on narrative and visual-explainer wins on rendering.

## 1. Build the fact sheet

After explain-diff step 1, write a private fact sheet before any HTML:

- resolved comparison and file/line counts
- each behavioral claim with its supporting file and line
- new or changed public interfaces, configuration, data shapes, side effects
- the before and after path through the system
- tests and the user-visible guarantee each establishes
- uncertainties, missing context, unverified assumptions

**Complete when:** every claim planned for the page has evidence or an explicit uncertainty label.

## 2. Map the spine to the page

A scrollable self-contained page by default; slides only on request. Sections, in order, each carrying the matching spine part:

1. **Hero**: the problem and outcome, with the resolved comparison stated.
2. **The key trick**: the strongest callout on the page.
3. **Before → after**: only the architecture, state, data shape, or interaction that materially changed. A local edit gets no topology diagram.
4. **Moving parts**: two to four concept cards; filenames are evidence labels, not headings.
5. **Trace a real example**: a Mermaid flow or sequence diagram when connections matter, with the concrete values on the path.
6. **Hidden infrastructure**: the non-obvious dependency made explicit.
7. **What the tests guarantee**: guarantees, evidence, gaps; tests read versus tests run.
8. **Evidence and file map**: compact or collapsible, with clickable local file links where supported.
9. **Zoom in**: the labeled follow-ups from the spine.

Add a review section only when the user asks for review, risks, or quality assessment, and keep it separate from the explanation.

## 3. Render and verify

Use visual-explainer's template and its full Mermaid zoom/pan/expand shell whenever Mermaid is present. Write to `~/.agent/diagrams/<name>-visual-diff.html`.

Before delivery check that:

- every page claim matches the fact sheet
- sections follow the spine order
- the trace shows real or clearly labeled illustrative values
- the hierarchy survives the squint test
- narrow and wide layouts do not overflow
- light and dark themes are both intentional
- diagrams render with working zoom, pan, reset, and expand
- the page opens without console errors

Open it in the product-native browser preview when available, otherwise the OS browser. In chat give the file path, a one-paragraph takeaway, and the labeled zoom-in choices, then ask which part is still fuzziest.
