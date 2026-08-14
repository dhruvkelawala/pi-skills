# skills

## Available skills

| Skill | Description |
|-------|-------------|
| [apr](skills/apr/SKILL.md) | Run autoreview, commit, push, and open or update a ready-for-review PR. |
| [code-walkthrough](skills/code-walkthrough/SKILL.md) | Guided codebase review for engineers inheriting or auditing agent-written or unfamiliar codebases. |
| [explain-diff](skills/explain-diff/SKILL.md) | Explain a diff, staged changes, a branch, or a PR in plain language with a concrete end-to-end trace. |
| [herdr-hunk-walkthrough](skills/herdr-hunk-walkthrough/SKILL.md) | Open a full PR diff in a 50/50 Herdr Hunk split with a numbered code walkthrough. |
| [idea-foundry](skills/idea-foundry/SKILL.md) | Generate or pressure-test ambitious project ideas through research, scoring, red-teaming, and pitching. |
| [implement](skills/implement/SKILL.md) | Implement or delegate a single GitHub issue, Linear ticket, or current plan as a small, reviewable vertical slice. |
| [research-explainer](skills/research-explainer/SKILL.md) | Research a topic against primary sources via background agents, then generate a self-contained HTML field guide that teaches it from scratch. |
| [use-clawpatch](skills/use-clawpatch/SKILL.md) | Run clawpatch automated code review: map features, review for findings, fix issues, revalidate, and track progress. |

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
