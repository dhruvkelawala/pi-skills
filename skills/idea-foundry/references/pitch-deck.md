# Idea Foundry — Shark Tank pitch deck spec

Build via the `visual-explainer` skill (invoke it; follow its aesthetic
rules — no AI-slop patterns). One self-contained interactive HTML page,
opened in the browser. The user is the shark; the pipeline is pitching.

Required structure, in order:

1. **Hero** — "N ideas walked in, K survived" framing; count-up stats
   (ideas generated, lenses, live searches, kills, survivors).
2. **The Gauntlet** — funnel of the run: generated → finalists → red-team
   kills → survivors, one stage per row with a shrinking bar.
3. **The Tank** — tabbed pitch per surviving finalist. Each pitch panel:
   - one-liner + badges (type, energies, standout fact)
   - "The pitch" written in pitch-night voice ("Sharks — ...") but every
     claim sourced
   - animated 6-bar rubric scorecard + composite (e.g. 27/30)
   - market evidence card with linked sources
   - moat / founder-fit card
   - the magic-moment pull quote (gold-bordered, full width)
   - collapsible **"The sharks attack"** — the red team's actual kill
     shots — followed by the founder's rebuttal and accepted constraint
   - **"The ask"** — the two-week wedge — plus the red-team verdict stamp
4. **The Kill Floor** — gravestone cards for every red-team kill: idea,
   what it was, cause of death (specific evidence, not vibes). Plus a
   collapsible **cut ledger** table for ideas eliminated at scoring
   (idea / what it was / why cut) and a "bench" callout for reserves.
5. **The Deal** — decision matrix table (all rubric scores, winner row
   highlighted) and "the cheque I'd write": ONE recommendation with the
   counter-bet named and the risk profiles contrasted (execution risk vs
   demand risk).

Style notes that worked: dark stage aesthetic, deep navy + gold, Bricolage
Grotesque + Fragment Mono, shark-fin SVG on the attack summaries, verdict
stamps (champion = green, viable = amber), reveal-on-scroll, animated score
bars, reduced-motion respected. Vary the aesthetic between runs per
visual-explainer rules.

Output to `~/.agent/diagrams/<run-name>-pitch.html` and `open` it.
