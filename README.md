# skills

Agent skills for reviewing work and getting it merged. Each one runs a fixed
process with a human gate, so the agent takes the same steps every run.

| Skill | What it does | Install |
|---|---|---|
| [github-pr-review](docs/github-pr-review) | Reviews GitHub PRs, including stacks, then posts findings you've approved as inline comments, with proven suggestions, code sketches and before/after screenshots. | `npx skills add frontier159/skills -s github-pr-review` |
| [review-loop](docs/review-loop) | Runs a cold, adversarial review loop over code, plans, specs or docs: verify the findings, apply the fixes, update the spec, stop at your gate. | `npx skills add frontier159/skills -s review-loop` |

Try it on your own branch before you push:

```
/review-loop self-review my branch feature-x against main before I push.
Spec: docs/feature-x-plan.md. Loop until clean, then stop.
```

Install everything with `npx skills add frontier159/skills`, or ask your agent
to "install the review-loop skill from https://github.com/frontier159/skills". The skills use
the [Agent Skills](https://agentskills.io) format, so they work in Claude
Code, Codex, Cursor and other agents. You can also copy a folder from
[`skills/`](skills) into your agent's skills directory.

In Claude Code, they're also available as a plugin:

```
/plugin marketplace add frontier159/skills
/plugin install frontier159-skills@frontier159-skills
```

## Layout

```
skills/<name>/SKILL.md        the skill itself, with any references/ beside it
docs/<name>/README.md         how to use it, with examples/
.claude-plugin/               plugin and marketplace manifests
```

## Previously

- `github-pr-review` used to live at frontier159/github-pr-review, which is
  now archived.
- [pr-stack](https://github.com/frontier159/pr-stack) is deprecated:
  frontier models now split a branch into a reviewable stack well without a
  skill.

## License

MIT
