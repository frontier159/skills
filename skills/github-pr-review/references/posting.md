# Posting to GitHub

## Comment shape

Write for the PR author, who hasn't seen the review:
1. The finding: what is wrong and why, in one to three sentences. Include concrete input → output where behaviour changes.
2. The fix, in one of two forms:
   - A **suggestion** block when the replacement is self-contained: committing it alone keeps the build green.
   - Otherwise "Sketch of the fix:" and a fenced snippet copied from the proof branch, with `// …` for elided lines. A sketch still beats prose even when imports or callers also change.
3. Optional: **Before/after** bullets and an image.

Run `stop-slop` over the comment text when it's installed.

## Anchoring

- A line comment must land on a line inside the PR's diff, on the RIGHT side. Resolve each anchor by a code substring at the PR head, not a hand-typed number. Then check the line falls inside a hunk of `git diff -U0 $(git merge-base <base> <head>) <head> -- <path>`.
- For a stacked PR, the base is the parent branch.
- Anchor on the line the fix changes, not an import above it.

## Suggestions

- A suggestion replaces exactly the anchored lines: `line`, plus `start_line` for a range. Create range comments with `start_line`, `start_side: RIGHT`, `line`, `side: RIGHT`.
- **Editing a comment can't change its anchor.** Decide each suggestion's range before posting. If a single-line comment later needs a multi-line suggestion, post the new range comment first, then delete the original (only when it has no replies).
- An empty suggestion block deletes the anchored lines.
- A fix that also needs an import gets a second, small suggestion on the import line. Tell the author to apply both with "Add suggestion to batch".

## Posting

- Check each PR head sha against the one you reviewed. If a head moved, re-resolve the anchors.
- Post one review per PR: `gh api repos/O/R/pulls/N/reviews -X POST --input payload.json`, with `{commit_id, event: "COMMENT", body?, comments: [{path, line, side, start_line?, start_side?, body}]}`. Put a consolidated top-level note in `body`.
- Edit: `PATCH repos/O/R/pulls/comments/<id>` with `-f body=…`. Find ids by body prefix and the operator's login.
- After any post that printed nothing, list the comments before retrying. Silent failures happen, and so do duplicates.

## Images

- `gh pr comment <N> --attach ./img.png` (recent gh) uploads the file, and rewrites `![alt](./img.png)` in the body to a `user-attachments` URL. Review comments have no attach flag.
- To get an image onto inline comments: post one top-level "Before/after" summary per affected PR with the images and bullets. Read that comment's body back, extract the `user-attachments` URLs, and append `**Before/after** … ![alt](url)` to each inline comment with PATCH.
- A browser extension's file upload can also attach straight into an inline comment's edit box. First confirm which browser is connected and that it is signed in as the right GitHub user.

## Tracker

After posting, set each row to `posted`, link the reviews and summary comments, and record which comments carry suggestions and which carry sketches.
