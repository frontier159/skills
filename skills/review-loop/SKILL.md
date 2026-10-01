---
name: review-loop
description: A cold, adversarial review loop for anything with a spec or a standard to meet — code branches, PRs, plans, specs, design docs. Fresh-context reviewers in parallel, every finding verified, fixes applied by an executor (folded into the owning commits for code, edited in place for documents), the spec updated to match, optionally a second loop, then a STOP at the operator's gate. Use when the user asks for a "deep review", "adversarial review", "review and fix", "implement the feedback then stop", "second review loop", "curate the branch", or to check that a plan or spec achieves its goal.
---

# Review loop

Someone, often a cheaper model, has produced work: code on a branch, a plan, a spec, a design doc. This loop is where quality gets enforced. It reviews the work **cold**, fixes what survives verification, and **stops** at the operator's **gate**.

1. Pin the inputs.
2. Spawn cold reviewers in parallel.
3. Verify every finding yourself.
4. Brief an executor to apply the fixes.
5. Verify the executor's work and update the documents.
6. Optionally curate the history and loop once more.
7. Report and STOP.

Each step works for both **subjects**: **code** (one or more branches) and a **document** (a plan, spec or design doc). The difference shows up in what the reviewers read and how fixes land.

## 1. Pin the inputs

- **The subject.**
  - Code: each branch, its worktree, its tip sha and its base. A stacked branch reviews against its parent: `git diff parent...child`.
  - Document: the file path and its current version (commit sha, or a saved copy).
- **The spec.** What the work is meant to achieve: a plan, issue, PRD, ticket, or the document's own stated goal. Note any design docs it names.
- **The house rules.** Gather them in this order, and read every hit in full:
  1. Agent guides from the repo root down to every path under review: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursor/rules/`, `.github/copilot-instructions.md`, plus `CONTRIBUTING.md` and any style or conventions doc they link.
  2. Your memory: the index, then every entry marked as feedback, a rule or a convention, and any house-rules file an entry names.
  3. Any `house-rules.md` file or `## House rules` section a guide or memory entry points to.

  Where sources conflict, the more specific one wins: the operator's memory over the repo, and a package guide over the root. When nothing turns up, use the defaults in the review brief and say so in the report.

For code, confirm each fixed point resolves and each diff is non-empty. Run the baseline build, typecheck and tests at every tip, in the background, and record the counts; reviewers' claims about tests get checked against them. Note the toolchain quirks the executor will need: environment setup, workspace libraries to build first, which gate's exit code counts.

Done when every subject and spec source is named, every house-rules source is listed by path (or recorded as "none found, defaults apply"), and code has a baseline result.

## 2. Spawn the reviewers

Write one shared brief from [review-brief.md](references/review-brief.md). It is a template: resolve every path and ref for this task, and fill its house-rules slot from step 1. Then launch fresh-context subagents in one message. They never see each other's output. Pick the axes that fit the subject:

- **Standards** (code, per branch): the repo's guides and house rules, API quality, over-engineering, and a comment sweep.
- **Spec** (code, per branch): missing or partial items, scope creep, implemented-but-wrong (trace the code, don't trust names), contradicted decisions, test gaps. Each finding quotes the spec line.
- **Soundness** (document): internal contradictions, stale references to code or other docs, claims that can't be verified, steps that can't be executed as written, speculative scope.
- **Remaining work** (a plan with unbuilt phases): stale references, design of what's proposed, speculative scope, day-one blockers, unverifiable test items.
- **Goal** (when asked whether the work reaches its goal): trace the end-to-end flow as it will be when finished, then list gaps, ordering hazards and risks on money or safety paths. End with a verdict and the minimum additions it needs.

Keep the axes separate in the report: a change can pass one and fail another. While the reviewers run, read the key seams or sections yourself, so you can judge findings rather than relay them.

Done when every reviewer has reported.

## 3. Verify before acting

Open the subject for every finding you'd act on. A correctness claim gets a trace, not a glance. Sort the survivors:

- **HARD:** a documented rule, a house rule, or a correctness bug. Fix it.
- **Judgement:** a smell or a design preference. Fix it when the change is small and clearly better; otherwise record it as a follow-up with the reason.
- **Spec vs rules:** when the spec asks for something the rules forbid, the rules win. Say so prominently, so the operator can overrule.

Duplication or naming that predates the diff and that the diff only touched is a note, not a fix.

Done when every finding is HARD, Judgement, or dropped with a reason.

## 4. Brief the executor

Write a brief from [executor-brief.md](references/executor-brief.md). Give every change as an instruction: where it goes, the current shape, the target shape, and the test to add or delete. Give comment changes as exact `DELETE` / `REWRITE: <text>` lines. Spawn one strong-model agent per branch, one after another when branches stack. Always spawn a fresh agent; never resume a finished one for more file work.

- **Code:** each fix becomes `git commit --fixup=<owning sha>`, then `GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash <base>`. Every commit passes the typecheck alone (`git rebase --exec '<typecheck>' <base>`), tests pass at the tip, and commit signing stays intact. No stash, no push.
- **Document:** the executor edits the document in place and keeps a dated "review revisions" list in it, one line per decision, each one the operator can reject.

Done when the executor reports every instruction applied, or says which it couldn't apply and why.

## 5. Verify and document

Read the executor's key changes yourself: correctness fixes, new seams, deleted branches, added comments (`git diff base...tip | grep '^+ *//'`). Then bring the paperwork in line:

- **The spec:** update it to the shapes that shipped.
  - Add a dated "review revisions" list to its status block, with one line per decision, each one the operator can reject.
  - Fix stale names and references, and record the new tip and test counts.
  - Rewrite the unbuilt phases to match the remaining-work review.
  - List the open questions the operator must answer.
- **Any index of plans or specs:** update its row.
- **Handover or status notes,** if the project keeps them: state, next steps in order, open decisions.
- **Memory:** only what a future session can't derive from the repo.

When the code's name predates the doc's, rename in the doc rather than the code.

Done when the spec describes what exists, and every open question is listed.

## 6. Curate and loop (when asked)

Human review wants a story, not a fixup trail. Brief a strong-model agent from [curation-brief.md](references/curation-brief.md): history only, with byte-identical final trees and every commit typechecking alone. Then run steps 2–5 again on the result. A second loop typically finds prose and placement issues, not correctness bugs.

## 7. Report and STOP

For each subject, report:
- the tip or version, and the commit list;
- what changed and why;
- test counts before and after;
- what you declined to change, and why;
- the decisions the operator must take.

Then stop. Pushing, opening PRs and starting the next phase wait for the operator's explicit go-ahead.

## Why these rules

- Cold contexts, because a reviewer who watched the work being made defends it.
- Verify before acting, because a confident wrong finding costs more than a missed one.
- Fold into owning commits, because the branch is what a human reviews, not the fix trail.
- Every commit typechecks alone, because bisect and review both walk the history.
- STOP, because the operator wants a gate between phases.
