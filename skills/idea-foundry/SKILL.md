---
name: idea-foundry
description: Run the Idea Foundry pipeline — multi-lens researched idea generation, rubric scoring, adversarial red-teaming, and a Shark Tank pitch deck — to find or pressure-test ambitious project ideas. Use when the user invokes /idea-foundry, asks "what should I build", wants startup/project ideas generated and vetted, or wants a single idea of theirs steelmanned and red-teamed ("run it through the pipeline").
---

# Idea Foundry

A repeatable pipeline for finding projects worth building. The goal is never
"a good idea" — it is an idea at the intersection of three filters: **the
builder would use it weekly**, **other people would love it (not just like
it)**, and **it is newly possible** (a "why now" that wasn't true 18 months
ago). Every claim must be grounded in live web research; every finalist must
survive an adversarial red team before the user ever sees it.

Two modes:
- **Discovery mode** (default): generate → converge → red-team → pitch.
- **Vet mode**: the user brings one idea ("run X through the pipeline") →
  skip to Stage V below.

Read `references/agent-prompts.md` before launching any subagents — it holds
the lens, red-team, and steelman prompt templates with the hard rules baked
in.

## Stage 0 — Context harvest (founder profile)

Ground everything in the builder, not abstract markets:
1. Check persistent memory for a builder-preferences entry (any memory
   describing what the user likes to build, for whom, and their no-go
   spaces). If found, use it and confirm only deltas.
2. Scan their recent work. Ask once for the directory that holds their
   projects if it is not already known, then `ls -lat` it and read README
   headers of the 6–10 most recently touched projects. Extract: stack
   strengths, product taste, lived pains, unfair advantages.
3. Write a 5-line founder profile. This goes verbatim into every agent
   prompt — but mark it **"for taste calibration ONLY"**: new ideas must not
   be extensions of the builder's existing projects unless the user says
   otherwise.

## Stage 1 — Grill the builder

If no saved preferences exist (or the user asks to re-elicit), use
AskUserQuestion — one round, up to 4 questions:
- **Audience**: who do they want to build for (multiSelect)?
- **Energy**: what keeps them building at 11pm — frontier capability, craft &
  delight, hard real-world problems, social & connection (multiSelect)?
- **Ambition shape**: venture-scale / indie / OSS / whatever the idea demands?
- **Exclusions**: hard no-go spaces (crypto, regulated/liability, etc.)?

Save the answers to persistent memory so future runs skip this stage.

## Stage 2 — Divergence (4 parallel lens agents)

Launch 4 `general-purpose` agents **in one message** using the lens template
in `references/agent-prompts.md`. Pick 4 lenses that fit the elicited
profile from the lens library (or invent better ones):

- **Founder pain** — mine the builder's plausible recurring frictions, then
  validate each against live complaints (HN, Reddit, reviews).
- **Why now** — verify 8–12 capability/regulatory shifts from the last 12–18
  months, derive products only newly possible.
- **Love mechanics** — research products with cult followings, distill the
  mechanics of love, apply to the builder's turf.
- **White space** — loud complaint clusters meeting *structurally* weak
  incumbents (business model, legacy architecture, incentives).
- **Frontier × consumer** — under-exploited new capabilities aimed at
  mainstream delight, not productivity.
- **Social & connection** — unmet social mechanics; useful at N=2.
- **Craft wins a tired category** — documented incumbent misery winnable on
  taste alone.
- **[Audience]-as-humans** — the emotional/cultural life of the target
  audience, not their workflow.

When iterating (user says "go broader" / "different direction"), never reuse
the immediately previous round's lenses; carry the reigning champion forward
as the idea to beat.

## Stage 3 — Converge and score

1. Merge near-duplicates across lenses. **Multi-lens convergence is signal**
   — note which clusters were found independently by 3+ lenses.
2. Score every surviving idea 1–5 on: **daily-use** (would the builder reach
   for it this week without willpower?), **love** (a moment people tell
   friends about?), **why-now**, **ambition** (real ceiling?), **wedge**
   (small shippable v1 useful alone?), **moat** (does usage compound?).
3. Kill rules: anything ≤2 on daily-use or love dies regardless of
   cleverness. High variance beats high average (5-5-5-2 beats 4-4-4-4).
4. **Anti-sherlock filter**: kill anything a platform vendor
   (Apple/Google/Meta/OpenAI/Anthropic/GitHub) will obviously ship natively
   within ~6 months. Prefer ideas needing sustained product taste, private
   compounding data, or territory structurally invisible to incumbents.
5. Pick 3–7 finalists for the red team. Park "bench" ideas (good but
   off-brief) explicitly — they are reserves, not kills.

## Stage 4 — Red team

Batch finalists into 1–2 adversarial agents (3–4 ideas each) using the
red-team template. Their only job is to KILL each idea with live evidence:
missed competitors, graveyard analogs, platform/API/legal risk, retention
collapse. Verdicts: **KILL / WOUNDED-BUT-VIABLE / SURVIVES**, plus the
"strongest honest rebuttal" and "the one constraint that matters most if
built anyway". A finalist advances only with a written rebuttal to its
strongest objection. Record kills honestly — they are the pipeline's main
product.

## Stage 5 — Decide and pitch

1. Maintain a run log at `idea-foundry/RESULTS.md` inside the projects
   directory from Stage 0, appending per round: all ideas, clusters,
   scores, red-team verdicts with sources, cuts-with-reasons, bench.
2. **Pitch deck**: if the `visual-explainer` skill is available, invoke it
   and build a Shark Tank-style interactive HTML page (see
   `references/pitch-deck.md` for the required structure). Otherwise render
   the same structure as markdown.
3. Give ONE recommendation with reasoning, then AskUserQuestion for the
   pick: recommended idea / runner-up / iterate again.
4. On a pick: define the two-week wedge — the smallest version the builder
   would genuinely use before anyone else sees it — and start building only
   when the user confirms.
5. Update persistent memory with the run outcome.

## Stage V — Vet mode (single user-supplied idea)

Launch two agents in parallel:
- **Steelman**: research the adjacent landscape, then construct the 2–3
  strongest product shapes of the premise (ownership/incentive/security
  models included), each with evidence.
- **Red team**: attack the core premise and its likely shapes with the full
  Stage-4 checklist.

Then score the strongest surviving shape on the Stage-3 rubric, apply the
kill rules, and deliver a verdict with the same honesty as discovery mode —
including "KILL" if that is what the evidence says, even if the user loves
the idea. Never agree by default; the user invoked this pipeline precisely
to avoid flattery.

## Operating notes

- All divergence/red-team agents MUST do live web research (they load
  WebSearch via ToolSearch — the instruction is in the templates). No
  armchair brainstorming.
- Agents return raw structured data, not prose for humans.
- Respect the user's exclusions absolutely, even for "amazing" ideas.
- Cost: a full discovery round is ~6 background agents (~15–25 min
  wall-clock). Tell the user before launching.
