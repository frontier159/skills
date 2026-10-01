# Story screenshots, before and after

## Serve both trees

- A second worktree at the stack tip gives you the **before** tree; the proof branch is the **after** tree. Install dependencies in each.
- Serve each tree's story server from its own directory, on a free high port: check with `lsof -i :<port>`. Leave the developer's usual ports alone, and stop only servers you started.
- Start servers with a plain background command. A preview launcher config can resolve to the primary checkout and silently serve the wrong tree.
- List story ids from the server's index (Ladle: `curl localhost:<port>/meta.json`; Storybook: `/index.json`). Keep the stories that touch changed components, plus any that share changed CSS.

## Capture back to back

Capture each before/after **pair** in immediate succession. Stories that render relative times or `Date.now()` otherwise differ by minutes and show as false changes.

```bash
C="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"   # or chromium
"$C" --headless=new --disable-gpu --hide-scrollbars --window-size=1200,1100 \
  --virtual-time-budget=8000 --screenshot=out/<id>.png \
  "http://localhost:<port>/?story=<id>&mode=preview"
```

Ladle needs `mode=preview` so the page itself scrolls. Storybook uses `/iframe.html?id=<id>`.

## Diff

Install Pillow in a scratch venv (`python3 -m venv v && v/bin/pip install pillow`). Call a pair changed when its difference bounding box is non-empty after thresholding (> ~24 of 255) away antialiasing noise.

A changed pair whose "before" or "after" is blank, or lacks an overlay or dialog, is usually a **flake**: an animation or portal that hadn't rendered yet. Recapture it 3× from both servers before believing it. Check the proof diff too: if the branch never touches that component, the difference is a flake.

## Comparison page

One self-contained HTML file, with images embedded as base64 data URIs. For each story it shows:
- id and tag (changed / identical / timestamp only);
- a bullet list of what changed and which fix ID caused it, for every changed story;
- the before and after images side by side.

Add toggles to hide identical and timestamp-only stories. Send it with the rendered file view, then answer the operator's per-story questions from the images, not from memory.

For PR comments, crop the changed region from the same screenshots into one labelled image per finding ("Before" over "After"), scaled up 2× for small widgets. Reuse these captures instead of regenerating them.
