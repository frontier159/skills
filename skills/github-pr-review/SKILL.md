---
name: github-pr-review
description: Review one or more GitHub PRs (including a stacked set) and post the findings as inline review comments, after the operator triages them and every proposed fix is proven on a scratch branch. Use when the user asks to review PRs by number or link and post or propose inline comments, wants findings "as proposals I'll opine on before you post", or asks to add suggestions, code snippets or before/after screenshots to PR comments.
---

# GitHub PR review

Every comment you post makes a claim about someone else's code, so each one
must be **proven** before it leaves the machine:
- the finding traced in the code;
- the fix applied on a scratch branch and passing the repo's gates;
- the visual effect captured, where there is one.

The operator holds the **gate**. Nothing is posted until they give their verdicts and say go.

## 1. Pin the PRs

- Read each PR's head, base, body and size. Use REST (`gh api repos/O/R/pulls/N`): `gh pr view` goes through GraphQL, which rate-limits separately and often runs out first.
- For a stack, order the PRs base-first. Each PR's diff is `git diff <base-branch>...<head>`. Check that each head contains its parent's head (`git merge-base --is-ancestor`): a forked stack tests code without the parent's later fixes, and GitHub's diffs won't show it.
- Work in a fresh worktree (`git worktree add ../<repo>-review origin/<head>`) whenever the operator's checkout has uncommitted changes. Say so in one line.
- Run the repo's typecheck, lint, format and test gates at every PR tip, in the background, and record the results.
- Gather the **house rules**, in this order, reading every hit in full:
  1. Agent guides from the repo root down to every path the PRs touch: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursor/rules/`, `.github/copilot-instructions.md`, plus `CONTRIBUTING.md` and any style guide they link.
  2. Your memory: the index, then every entry marked as feedback, a rule or a convention, and any house-rules file an entry names.
  3. Any `house-rules.md` file or `## House rules` section those sources point to.

  The more specific source wins a conflict. When nothing turns up, say so; the reviewers fall back on the repo's guides alone.
- Check which [recommended companion skills](#recommended-companion-skills) are installed. Ask the operator once about all the missing ones together, and install only what they approve. Continue without the rest.

Done when every PR has a pinned head sha and a baseline gate result, every house-rules source is listed by path (or recorded as "none found"), and the operator has answered for every missing companion skill.

## 2. Collect what's already said

Fetch every inline comment, review body and issue comment on each PR (REST, `--paginate`).
- Turn bot findings (CodeRabbit and the like) into a **dedupe list**: path, line, one-line claim.
- Read human comments as decisions. A finding that contradicts one gets dropped or reframed, not posted.

Done when the dedupe list covers every existing finding.

## 3. Cold reviews

Write one shared brief from [reviewer-brief.md](references/reviewer-brief.md); with `code-review` installed, borrow its standards and spec axes, and with `test-audit` installed, hand it any tests the PR adds. Then launch, in one message, fresh-context agents that never see each other's output:
- **One per PR:** standards (the repo's guides and the operator's house rules) plus an over-engineering pass (what can be deleted or collapsed).
- **One for the whole stack:** does each PR do what its body says, is code placed in the right PR, is it correct against the real system it talks to (service, API, contract), and is anything unsafe for production.

While they run, read the key seams yourself so you can judge findings rather than relay them.

Done when every reviewer has reported.

## 4. Verify and triage

Open the code for every finding you keep.
- A correctness claim gets a trace, not a glance.
- A claim about a dependency or registry gets checked at the source (registry tarball, lockfile re-resolution).
- Reviewers exaggerate, and they sometimes contradict each other; the trace settles it.

Give every finding a **severity** from 1 to 10. It measures how much the finding matters to users and maintainers, not how sure you are:

| Score | Tier | Typical finding |
|---|---|---|
| 9–10 | must fix | wrong money or time reaching users, data loss, a security hole |
| 7–8 | important | a correctness bug users will hit |
| 5–6 | should fix | a latent bug, a rejected request, a broken build or bisect |
| 3–4 | minor | a narrow bug, a rule broken with real cost, a test that can't fail |
| 1–2 | nit | style, naming, comments, duplication |

Write the **tracker**: a markdown file in the repo root (untracked), one table per PR plus one stack-wide table. Columns: ID · Where (`path:line`) · Severity (1–10) · Finding · Proposed fix · Status. Sort each table by severity, highest first.

Then STOP at the **gate**: summarise the tracker in chat and ask for verdicts. As verdicts arrive, the tracker shows **only the latest state**:
- dropped rows are deleted;
- a downgraded row keeps only its new wording;
- themes the operator defers (e.g. "until the service is wired") collapse into one consolidated top-level comment, with its text in the tracker.

Done when every row has the operator's verdict.

## 5. Prove the fixes

Brief one strong-model agent to apply **all** surviving fixes in bulk, as one local commit on a scratch branch off the stack tip. It runs the gates **once** at the end and fixes forward. For each ID it reports one of: works as proposed / works with this change / doesn't work → alternative. For behaviour fixes it also gives concrete input → output cases. It never pushes.

Then check the agent's key hunks yourself, and rewrite each tracker row that changed. If `review-loop` is installed, run its cold-review pass over the proof branch too.

Done when every row is marked proven, or rewritten to the variant that was proven.

## 6. Capture the visual effect

Skip this step when no fix changes rendered UI. Otherwise follow [screenshots.md](references/screenshots.md):
- capture before (stack tip) and after (proof branch) story screenshots, back to back;
- diff them;
- write a bullet for every story that changed, and send the operator a comparison page.

Done when every changed story has a bullet that names its fix, and every unexplained difference has been recaptured or explained.

## 7. Draft and post

Follow [posting.md](references/posting.md). Each comment stands alone for the PR author:
- the finding;
- the fix: a **suggestion** block when the change is self-contained, otherwise "Sketch of the fix:" with code from the proof branch;
- any before/after image.

Decide suggestion line ranges **before** posting. An edit can't change a comment's anchor. Check the heads haven't moved since step 1, then post one review per PR.

Done when every tracker row is `posted`, its link is in the tracker, and each review URL is reported to the operator.

## Tools that may be useful

- `gh api` (REST): reviews, comments, edits, deletes. `gh pr comment --attach` uploads images (recent gh).
- `git worktree`, `git merge-base`, `git diff -U0` to anchor lines in a diff.
- Headless Chrome `--screenshot`, Pillow in a scratch venv for pixel diffs, the repo's story server (Storybook, Ladle).
- Registry tarballs (`npm view <pkg>@<v> dist.tarball`) to check what a dependency version really ships.

## Recommended companion skills

| Skill | Used in | Install |
|---|---|---|
| [`review-loop`](https://github.com/frontier159/skills) | step 5: a cold re-review of the proof branch | `npx skills add frontier159/skills -s review-loop -g` |
| [`code-review`](https://github.com/mattpocock/skills) | step 3: a two-axis (standards + spec) template for the cold reviewers | `npx skills add mattpocock/skills -s code-review -g` |
| [`test-audit`](https://github.com/openclaw/openclaw/tree/main/.agents/skills/test-audit) | step 3: when a PR adds or changes tests | `npx skills add openclaw/openclaw -s test-audit -g` |
| [`stop-slop`](https://github.com/hardikpandya/stop-slop) | step 7: strips AI writing patterns from comment prose | `npx skills add hardikpandya/stop-slop -g` |
