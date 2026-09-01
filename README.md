# skills

## Available skills

| Skill | Description |
|-------|-------------|
| [apr](skills/apr/SKILL.md) | Run autoreview, commit, push, and open or update a ready-for-review PR. |
| [build](skills/build/SKILL.md) | Build or delegate a single GitHub issue, Linear ticket, or current plan as a small, reviewable vertical slice. |
| [bro](skills/bro/SKILL.md) | Restate the last message in plain human language, with no jargon. |
| [code-walkthrough](skills/code-walkthrough/SKILL.md) | Guided codebase review for engineers inheriting or auditing agent-written or unfamiliar codebases. |
| [explain-diff](skills/explain-diff/SKILL.md) | Explain a diff, staged changes, a branch, or a PR in plain language with a concrete end-to-end trace. |
| [herdr-hunk-walkthrough](skills/herdr-hunk-walkthrough/SKILL.md) | Open a full PR diff in a Herdr Hunk split or tab with a numbered code walkthrough. |
| [idea-foundry](skills/idea-foundry/SKILL.md) | Generate or pressure-test ambitious project ideas through research, scoring, red-teaming, and pitching. |
| [issue-to-pr](skills/issue-to-pr/SKILL.md) | Take one GitHub issue end to end into a merge-ready PR by chaining /implement, /code-review until clean, /verify, /apr, and /pr-watch, standalone or stacked. |
| [product-description](skills/product-description/SKILL.md) | Build a prose "product description" repo describing what the user sees and exactly what happens when they act, drafted from code and tests, then verified and triaged into a bug list. |
| [pr-watch](skills/pr-watch/SKILL.md) | Poll a PR until CI is green, review threads are resolved, and each configured review agent has covered the current HEAD, repairing findings in between. |
| [research-explainer](skills/research-explainer/SKILL.md) | Research a topic against primary sources via background agents, then generate a self-contained HTML field guide that teaches it from scratch. |
| [review-ready](skills/review-ready/SKILL.md) | Completion gate that enforces a design contract (deep modules, narrative entry points, real seams, interface-oriented tests) and produces a review-ready report before handing work back. |
| [use-clawpatch](skills/use-clawpatch/SKILL.md) | Run clawpatch automated code review: map features, review for findings, fix issues, revalidate, and track progress. |
| [verify](skills/verify/SKILL.md) | Discover and run every verification a project defines and report each as passed, failed, or could-not-run. |
| [visual-diff](skills/visual-diff/SKILL.md) | Explain a diff, commit, branch, range, or PR as a polished self-contained HTML visual story with diagrams and a before/after narrative. |

## Installation

Add to your Pi `settings.json`:

```json
{
  "packages": [
    "git:github.com/dhruvkelawala/skills"
  ]
}
```

Or install a single skill:

```json
{
  "skills": [
    "git:github.com/dhruvkelawala/skills/skills/use-clawpatch"
  ]
}
```

## License

MIT
