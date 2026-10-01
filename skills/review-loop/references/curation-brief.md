# Curation brief template

Re-carve a branch's history so a human can read it top to bottom. This is
history only: the final tree must be byte-identical to the current tip.

## Header

```
# Curation brief: re-carve <branches> for human review

Each branch is correct at its tip. The job is history only.
Worktrees: <path>, branch <name>, tip <sha>, base <ref>=<sha> (one line each;
a stacked child's base is its parent's tip).
Toolchain: <as in the executor brief>. Typecheck per commit: <command>.
```

## Rules

```
- First: git branch curate-backup-<x>-<date> <tip>. Never delete it.
- Do the parent first, then rebase the child onto the new parent tip
  (git rebase --onto <parent> <old parent tip> <child>) and re-carve it.
- Final trees must match: git diff curate-backup-<x>-<date> <branch> prints nothing.
- Every commit typechecks alone: git rebase --exec '<typecheck>' <base>.
  Tests pass at each tip.
- Plain git commit (keep the repo's signing). No stash. If interactive staging
  is unavailable, build commits forward from the base with
  git checkout <backup> -- <paths>, and hand-edit intermediate states where
  one file lands in two commits.
- Commit messages: `<area>: <one present-tense sentence>` and a body of one to
  three short paragraphs: what changes, and the fact that motivates it.
  No plan numbers, PR numbers, review references or dates.
- Change no code. If a split can't typecheck alone without a code change,
  restructure the split and say so in the report.
```

## Target structure

A numbered list per branch: each commit's subject, then the files it owns (and, where a file splits, which hunks). Order by dependency: interfaces before their consumers, libraries before applications, code before guides. Give a fallback ("if 3 and 4 can't be separated, merge them into …") so the executor doesn't stall.

Heuristics that produce a readable history:
- one interface change per commit, with every caller it forces;
- a large feature split along module boundaries: a pure module and its tests alone, then the wiring and deletions;
- guides last, as one commit;
- a hand-edited intermediate state is a strict subset of the tip's text, which the empty final diff proves.

## Report

```
Per branch: the new tip, the commit list with files, the empty diff against
the backup, the per-commit typecheck result, the test counts at the tip, and
any place the target structure changed, with the reason.
Do NOT push, do NOT open or edit a PR, do NOT touch any other worktree.
```
