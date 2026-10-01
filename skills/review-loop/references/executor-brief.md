# Executor brief template

Write one brief per branch or document. The executor is a strong-model agent
with fresh context, and everything it needs is in the file. Replace every
`<placeholder>` for this task.

## Header

```
# Review fixes: executor brief for <subject>

Worktree: <absolute path>   (or document: <path>)
Branch: <name> (<n> commits on <base ref> = <sha>). Work ONLY here.
Read <path to the review brief> first: its house rules bind every line you write.

Toolchain: <environment setup, e.g. PATH or version-manager shims>
Build first: <workspace libraries the checks depend on>
Checks: <typecheck command>; <test command>; <lint command>
Also typecheck every dependent package whose interface changes: <list>
Gate on test exit codes. Fix lint findings in the files you touched.
```

For a stacked branch, add **Step 0: rebase**: `git rebase --onto <new parent> <old parent tip> <branch>`, the files expected to conflict, and the shape to resolve toward. Nothing else starts until the rebased branch builds and passes its tests.

## Changes

Number them. Each change names the file or section, the current shape, the target shape, a one-sentence reason, and the test to add, change or delete. Mark judgement calls as such. Name the existing helper to reuse rather than describing one. Prefer deleting over adding.

At this level of precision:
- "`QuoteReq` becomes `{ from; inputs: readonly Input[] }`, with `export interface Input { asset: Asset; amount: Decimal }`. Delete `legacyInputs` everywhere. Test: the retry seeds exactly `amount`."
- "Delete as unreachable: `fooMode`, `barFactor` (list them all), then the two tests that only exercised them."

## Comment sweep

Give exact lines: `path (or the comment's first words) → DELETE` or `→ REWRITE: <text>`. If a named comment doesn't exist on the branch, the executor reports that rather than inventing a placement.

## Landing the fixes

Code:
```
Each fix goes into the commit that introduced the lines it changes (git blame):
git commit --fixup=<sha> (plain git commit; keep the repo's signing), then
GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash <base>.
Afterwards: every commit is signed if the repo signs;
git rebase --exec '<typecheck>' <base> passes at every commit;
the full checks pass at the tip.
```

Document: edit in place, and add one line per decision to a dated "review revisions" list in the document.

## Boundaries and report

```
Do NOT push. Do NOT open or edit a PR. Do NOT touch any other worktree. No stash.
Report: the new tip (or document version), files per commit, test counts
before and after, and anything in this brief you couldn't do, with the reason.
```

## Why this shape

An executor with a precise list does the mechanical work well and stops; one
with a vague list improvises. Every ambiguity in the brief becomes a decision
the executor takes alone. The "couldn't do" clause matters: a sweep item
sometimes names a comment that lives on another branch, and the honest answer
is to say so.
