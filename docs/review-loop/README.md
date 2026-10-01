# review-loop

An agent skill that runs a cold, adversarial review loop over anything with
a spec or a standard to meet: code branches, PRs, plans, specs, design docs.
Fresh-context reviewers find the issues, the agent verifies each one, and an
executor applies the fixes. Then it stops for your go-ahead.

```mermaid
flowchart LR
    you([you]) -- "work to review" --> rl[["/review-loop"]]
    rl -- "parallel, cold" --> rv["reviewers<br>standards · spec · soundness · goal"]
    rv --> v{verified?}
    v -- "HARD / judgement" --> ex["executor<br>fixes folded or edited in place"]
    v -- dropped --> rl
    ex --> doc["spec updated<br>to what shipped"]
    doc --> stop([STOP: your gate])
```

## Install

```bash
npx skills add frontier159/skills -s review-loop
```

Or copy [`skills/review-loop/`](../../skills/review-loop) into your agent's
skills directory by hand.

## Usage

```
deep review of feature-a and feature-b against plan.md, implement the feedback then stop
adversarial review of docs/spec.md: does it achieve its goal?
second review loop
curate the branch for human review
```

## How it works

**Pin.** Names the subject (branches with their tips and bases, or a document
and its version), the spec it answers to, and the standards: the repo's
guides plus your house rules. For code it records a baseline of build,
typecheck and tests.

**Review cold.** Launches parallel reviewers with fresh context, which never
see each other's output. Axes depend on the subject:
- **standards** and **spec** for code;
- **soundness** for documents;
- **remaining work** for plans with unbuilt phases;
- **goal** when you ask whether the whole thing gets where it's meant to.

**Verify.** Traces every finding before acting. Each one ends up HARD (a rule
or a correctness bug), judgement (fixed when small and clearly better), or
dropped with a reason. Where the spec and the rules conflict, the rules win,
and the report says so.

**Fix.** A strong-model executor follows a precise brief.
- Code: fixes fold into the commits that own them, and every commit
  typechecks alone.
- Documents: the executor edits in place and keeps a dated list of review
  revisions.

**Document.** Updates the spec to what actually shipped and lists the open
questions.

**Curate (optional).** Re-carves the history into a readable story. The final
tree stays byte-identical, and every commit stays green. Then it loops again.

**Stop.** Reports, per subject: what changed and why, what it declined to
change, and the decisions left for you. Nothing is pushed until you say go.

## Hard rules

- Reviewers are cold and read-only.
- Never acts on a finding it hasn't verified.
- Never pushes, opens a PR or starts the next phase without your go-ahead.
- Never stashes; folds fixes rather than appending fix commits.
