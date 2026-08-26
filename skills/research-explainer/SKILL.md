---
name: research-explainer
description: Research a technical topic against primary sources via background agents, save cited findings as Markdown in the repo, then generate a self-contained HTML "field guide" that teaches the concepts from scratch as building blocks. Use when the user wants a topic researched AND explained as a learning doc, says "research and explain", "field guide", "explain this research to me from scratch", or invokes /research-explainer.
---

# Research Explainer

Two phases, always in this order: **research against primary sources**, then **teach it as a field guide**. The output is (1) one or more cited Markdown research docs saved in the repo, and (2) one self-contained HTML page that builds the topic up from zero, block by block, using the user's own project as the running example wherever one exists.

## Phase 1 — Research (background agents, primary sources)

Spin up **background agents** (read-only researchers; librarian-type if available) so you keep working while they read. Split the topic into 1–3 independent research threads; one agent per thread.

Each agent's contract:

1. Investigate against **primary sources only** — papers (arXiv over blog posts), official docs, and the actual source code of libraries. Follow every claim back to the source that owns it; never cite a summary when the original is reachable.
2. Write findings to a **single Markdown file**, one per thread. Every claim carries a citation (URL; file path + line range for code claims). Structure: answers-first summary → per-question findings with citations → integration sketch for the current repo → open risks.
3. Save where the repo already keeps such notes (`docs/research/` is a common convention); create the directory if none exists and say where.

While agents run, prepare Phase 2: read the template and structure reference below.

## Phase 2 — Field guide (HTML learning doc)

When research lands, generate the explainer. Read both bundled references BEFORE writing HTML:

- [references/structure.md](references/structure.md) — the document anatomy: hero, "the one idea", building blocks, assembly, cheat sheet, sources; block fixtures; figure catalog; hard rules.
- One of the bundled templates (pick per the variant notes in structure.md, and rotate between generations):
  - [templates/explainer-template.html](templates/explainer-template.html) — **editorial** variant: quiet warm-editorial reading document (Instrument Serif + DM Sans; sidebar TOC, soft cards, per-scheme Mermaid theming).
  - [templates/explainer-template-arcade.html](templates/explainer-template-arcade.html) — **arcade** variant: neo-brutalist game-manual poster (Archivo Black + Atkinson Hyperlegible + Space Mono; sticky tab nav, full-bleed alternating color bands, giant numbered section heads, hard-shadow chips/cards, tilted ribbon callouts, constant dark schematic panels so figures/Mermaid are identical in both schemes).

  Either way, copy the chosen template's machinery wholesale; replace the placeholder content.

Core principles (details in structure.md):

- **Building blocks**: decompose the topic into 3–7 self-contained concepts ordered so each builds on the previous. A reader with zero background must be able to read top to bottom without getting lost.
- **Grounded, not generic**: every block ends with an "in our project" callout using the user's real incidents, numbers, and file paths from the Phase 1 research. If there is no project context, use one concrete worked example per block instead.
- **One figure per block**: a hand-drawn inline SVG (distribution, curve, ladder, scatter, loop — see the figure catalog) beats prose. Mermaid only for the final assembly diagram.
- **Honest edges**: blocks with sharp caveats get a "watch out" callout — calibration warnings, known failure modes, what the paper does NOT claim.
- **No rhetorical filler sections**: no "say it to a friend" / "repeat after me" / "TL;DR" blocks. Takeaways live in the prose, the cheat-sheet glossary, and the figures.
- **Aesthetics**: pick a distinctive font pairing and palette per the template's notes; never Inter/Roboto, never violet-gradient defaults. Support light and dark via `prefers-color-scheme`.

## Delivery

1. Write the HTML to the repo next to the research docs (e.g. `docs/research/<topic>-explainer.html`) so it ships with the project.
2. Verify before handing over: open it headless, screenshot the hero, every figure, and the assembly diagram; check both color schemes; confirm zero horizontal overflow and that the Mermaid diagram fits at contain zoom (shorten node labels / raise `wrappingWidth` if it crops).
3. Open it in the user's browser and report: file path, the research doc paths, and the block list.

## Review checklist

- [ ] Every research claim in the guide traces to a cited line in a Phase 1 doc
- [ ] Blocks readable in order by a newcomer; no forward references
- [ ] Each block: question → explanation → figure → project callout (→ watch-out if warranted)
- [ ] Cheat-sheet glossary: every term of art gets one honest sentence
- [ ] Sources section links each block to its papers/repos AND the repo research docs
- [ ] Both themes intentional; no overflow; figures legible at default zoom
