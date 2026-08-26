# Field Guide Structure

The document anatomy for research-explainer output. The template implements all CSS classes named here.

## Document skeleton (in order)

| Section | TOC id | Purpose |
|---|---|---|
| Hero | — | doc-id line, serif headline (one italic accent phrase), mono subtitle, lead paragraph stating the single thesis the whole doc builds toward |
| The one idea | `s0` | The framing that makes everything else obvious. Include a 3-step `mini-pipe` (horizontal cards) mapping the blocks to jobs, and a callout stating the governing rule/principle |
| Building blocks | `s1…sN` | 3–7 concept sections, each self-contained (anatomy below) |
| Assembly | — | One Mermaid diagram showing how the blocks compose into the real system, each node labeled with its block number; follow with a callout on why the composition compounds |
| Cheat sheet | — | Glossary table: `term` column (mono, accent color) + one honest plain-language sentence each. Every term of art used anywhere in the doc must appear |
| Sources | — | Per-block source list: papers (arXiv links), repos, AND the repo-relative paths of the Phase 1 research docs and data |
| Closing line | — | One mono footer line: what this doc is, what it feeds into |

## Block anatomy

Each building block, strictly in this order:

1. **Section head** — `sec-head` with a `BLOCK N` chip + serif title (the concept's name, not jargon: "Why models repeat themselves", not "Mode collapse")
2. **The question** — one italic serif line: the concrete question this block answers, phrased from the reader's world ("We asked for creative output at temperature 1. Why did we get the same answer five times?")
3. **Explanation prose** — 2–3 short paragraphs, `prose` class. Build from zero: define every term at first use, bold the term (`<b>` renders accent-colored). Prefer mechanism over assertion: say *why* the thing happens.
4. **Figure** — one `fig` card with an inline SVG (catalog below) + `figcaption` that adds information, never repeats the prose.
5. **Project callout** — `callout callout--ours`, title "In our pipeline" / "In our project": the user's real incident, metrics, thresholds, costs, file paths that this block explains. This is what makes the doc theirs.
6. **Watch-out callout** (only when warranted) — `callout callout--warn`: the sharp edge — what is NOT claimed, calibration caveats, failure modes, what experiment gates it.

**Never add** "say it to a friend" / "repeat after me" / "TL;DR" / "key takeaway" boxes. The prose, figure, and cheat sheet carry the takeaways.

## Figure catalog (inline SVG, themed via CSS classes)

Pick the shape that matches the concept; all classes exist in the template:

| Concept shape | Figure | Classes |
|---|---|---|
| Distribution / where mass concentrates | one or two bell curves, mode marked with dashed drop-line, area filled | `s-curve`, `s-curve--dim`, `s-fill`, `s-fill-gold`, `s-dash` |
| Sampling / selection from a space | curve or scatter with highlighted dots + labels | `s-dot`, `s-dot-gold`, `s-dot-dim` |
| Ceiling / diminishing returns | rising curve flattening under a dashed limit line, annotated regions | `s-curve`, `s-dash-red`, axis via `s-axis` |
| Ranking / band placement | horizontal rungs + candidate dot + arrows with verdicts | `s-rung`, `s-arrow` (needs the `#arr` marker def), `s-text-green`/`s-text-red` |
| Similarity / clustering | 2D scatter, near-duplicates ringed dashed-red, selections ringed green | `s-ring-red`, `s-ring-green` |
| Loop / feedback cycle | 4 rounded boxes (`s-box`) in a cycle with arrows, numbered steps |
| System composition | Mermaid `graph TD` in the zoomable `diagram-shell` — assembly section ONLY |

SVG rules: `viewBox` ~720 wide; text in `s-text*` classes (mono, themed); keep labels short enough to fit the viewBox — check for clipping; every text/element uses the CSS classes, never hardcoded colors (themes break otherwise); add `role="img"` + `aria-label`.

## Aesthetics

- Two bundled template variants; rotate between generations and pick to fit the audience:
  - **editorial** (`templates/explainer-template.html`) — quiet reading document. Warm-editorial default (Instrument Serif + DM Sans + JetBrains Mono; cream/terracotta/ochre/sage). Sidebar TOC, soft-bordered cards, Mermaid re-themed per color scheme. Best for calm deep-reading docs. Re-skins that hold up: deep-navy + gold editorial, paper/ink with sage, a real IDE palette (Nord, Gruvbox).
  - **arcade** (`templates/explainer-template-arcade.html`) — loud game-manual poster. Archivo Black display + Atkinson Hyperlegible body + Space Mono; full-bleed bands alternating cream/coal/periwinkle with coral/yellow/green accents; sticky top tab nav; giant coral section numbers with band-colored text-stroke; chips and cards with 2–3px hard borders + offset shadows; "In our project" / "Watch out" as tilted green/coral ribbons; figures and the Mermaid assembly ALWAYS on a constant dark panel with a yellow frame, so all SVG + Mermaid colors are identical in light and dark schemes (only band backgrounds flip). Best for plans, reviews, and topics that benefit from energy. Keep band rotation strict (paper → coal → blue → …) and never place two same-color bands adjacent.
  - Variant-specific checks for arcade: the intentional horizontal scroller is the top nav only — `document.documentElement.scrollWidth` must still equal `clientWidth`; `<b>` inside `.band--blue` prose renders as a yellow highlight, so keep bolded runs short there.
- Forbidden regardless: Inter/Roboto as body, violet/indigo accents, gradient heading text, glowing shadows, emoji section icons.
- Light AND dark palettes via `prefers-color-scheme`; Mermaid theme colors set from the same palette in both modes (the template shows the `isDark` + `themedSource` pattern).

## Verification (before handing over)

1. Headless-open the file; screenshot hero, EVERY `.fig`, the assembly diagram, the glossary — in both color schemes.
2. `[...document.querySelectorAll('*')].filter(el => el.scrollWidth > document.documentElement.clientWidth + 4).length` must be 0.
3. Assembly diagram must fit at contain zoom on first paint: single-line node labels + `flowchart.wrappingWidth: 440+`; if the zoom label reads "width-priority", shorten labels until it reads "contain".
4. Common SVG bugs to check: paths missing `fill: none` rendering as black blobs; labels clipping at the viewBox edge; overlapping annotations.
