# Example run: reviewing a five-PR UI stack

A fictionalised transcript of a real run. A team shipped an orders feature
for a trading web app as five stacked PRs: shared UI atoms, a data layer, a
list page, a detail page, and a hero card with a cancel dialogue. A review
bot had already commented. The operator wanted proposals first and the
posting only after their say-so.

## Invocation

```
> review PRs #201 -> #205. Report findings here as proposals for inline
> comments; I'll opine before you post. The bot may already have left
> comments we don't want to duplicate.
```

The operator's checkout had uncommitted work, so the agent opened a
worktree instead of switching branches. `gh pr view` hit the GraphQL rate
limit, so it read the five PRs over REST (`gh api repos/acme/shop/pulls/N`)
and pinned each head sha. Typecheck, lint and format passed at every tip.

The bot had left seven inline comments and one nitpick. They became the
dedupe list in the reviewer brief.

## Cold reviews

Six fresh-context agents ran in parallel: one per PR on standards and
over-engineering, and one on the whole stack against the real order
service. The agent traced each finding before keeping it:

- **Kept:** "the cancel button only closes the dialogue". It traced
  `confirmCancel` to `setCancelOpen(false)`.
- **Corrected:** "the icon library bump is unnecessary". The registry
  tarball for the old version lacks the one icon the stack needs, so the
  bump stays. It just belongs in the PR that first uses the icon.
- **Root-caused:** lockfile churn in #201. The engine package resolved an
  optional peer of the chain library to v4 while every other consumer
  resolved v3, so the package manager injected copies. The fix went to a
  separate PR on the base branch, which then merged.

## Triage

The tracker, `orders-stack-review.md`, had one table per PR. The
operator's rulings, in order:

> the mock orders are intentional until the service is live
→ the "mock ships to prod" blocker was deleted from the tracker.

> 1 and 7 drop, they'll merge as a stack anyway · 11 agree, drop the
> "move to #203" part · 31 drop, an extra check is fine
→ rows deleted or reworded; only the latest state remains.

> unused fetchers / coming soon / story-only states: one consolidated
> comment saying we address them when the service is integrated
→ six findings became one top-level comment on #202.

## Proof

An agent applied all 35 surviving fixes as one local commit on
`review-fix-trial`, ran the gates once, and reported per ID:

```
2  — WORKS as proposed
9  — DOESN'T WORK: resolving tokens once needs a third dependent query
     → drop only the always-ok Result wrapper
15 — WORKS with change: the comparison sign follows direction
     (buy: "1 vETH ≤ 2.6316 USDC", sell: "1 vETH ≥ 2.6000 USDC")
38 — DOESN'T WORK as proposed: the expired state inherits the vault
     hero's white glow → share the geometry only, keep backgrounds apart
```

The tracker took the proven variants.

## Screenshots

The agent ran the story server from both trees on spare ports and captured
35 stories. The first pass showed 22 changes, but most were relative
timestamps that had moved between the two capture runs. Recaptured as
back-to-back pairs, 17 stories really changed, and each got a bullet:

```
orderdetail--ordertimeline--cancelled
  - "Open for fills" now ends at the cancel time instead of valid_to (#27)
pages--orderstable--showcase
  - Limit price shows a unit and quotes per vault share (#15)
  - Amount column header lost its sort icon (#16)
```

The operator asked why the signing dialogue went "from white to grey".
Recapturing it three times from each server showed the white "before" was a
blank first paint, and the proof branch never touches the dialogue. They
also asked why two stories had disappeared. The agent had deleted them as
duplicates, but on a second look they carried the orders-specific empty
copy, so they were kept and the comment edited.

## Posting

The agent checked the heads hadn't moved, resolved every anchor by code
substring, and confirmed each line fell inside its PR's diff. It then posted
one review per PR: 37 inline comments, plus the consolidated note.

Later rounds:

- **Images.** `gh pr comment --attach` posted one before/after summary per
  affected PR. The agent read back the `user-attachments` URLs and embedded
  the same images into five inline comments with PATCH.
- **Suggestions.** Six fixes were self-contained, but their comments were
  anchored on single lines and an edit can't widen an anchor. The agent
  posted each one again on its full range with a `suggestion` block, then
  deleted the original (none had replies). A fix that also needed an import
  got a second suggestion on the import line, to apply as a batch.
- **Sketches.** Seventeen prose-only comments gained "Sketch of the fix:"
  with code copied from the proof branch.

One `gh pr comment` loop printed nothing and posted nothing. The agent
listed the comments before retrying, so no PR got a duplicate.
