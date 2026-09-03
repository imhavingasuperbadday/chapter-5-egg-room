# Agent instructions

Rules for any AI agent working in this repo. Read this before touching anything.

## Branches: don't proliferate them

**Do not create a new branch for your work.** Commit to the branch that is
already checked out and push to that same branch on `origin`.

This repo already has too many one-off `agent/*` branches from past sessions.
Every extra branch means another merge back into `main` later, and a bad merge
is exactly how the game got broken once already (see "History" below).

- Continuing existing work? Commit on the current branch.
- The current branch is `main`? Ask the user before committing there; if they
  say to branch, use the branch name *they* give you.
- Never `git checkout -b` on your own initiative.
- Never push a branch the user did not ask for.

## Merging

If you must merge or rebase, **verify the result parses before committing**:

```sh
sed -n '18,/<\/script>/p' index.html > /tmp/check.js && node --check /tmp/check.js
```

The whole game is one `<script>` block in `index.html`. A single bad merge
hunk makes the file unparseable, so nothing runs and the page renders as a
plain black canvas — with no error message anywhere on screen. Duplicated
function bodies and stray leftovers from a replaced code path are the usual
symptom of a botched resolution; read the full diff of a merge before you
commit it.

## Verifying a change actually works

Static checks are not enough — render the page and look at it:

```sh
python -m http.server 8765          # from the repo root
"/c/Program Files/Google/Chrome/Application/chrome.exe" \
  --headless=new --disable-gpu --window-size=854,480 \
  --virtual-time-budget=8000 --screenshot=shot.png \
  http://localhost:8765/index.html
```

A working boot shows the dark room, the block tree, and Kris. An all-black
image means the script failed to parse or `boot()` rejected.

Serve over HTTP, not `file://` — the canvas gets tainted by `file://` images
and `getImageData` (used by the font tinting and tree collision maps) throws.

## Project shape

- `index.html` — the entire game: assets table, room logic, dialogue system,
  render loop, title/settings menu. Single file on purpose.
- `assets/` — sprites, font atlas, audio. Small images are inlined as data
  URIs in `index.html`; only larger ones live on disk.

## History

`06527c7` merged `main` into `agent/egg-room-polish` and left a duplicate,
unterminated `drawOfficialChoices()` plus fragments of the old sequential
asset loader inside `boot()`. The script stopped parsing and the game showed
only a black screen. Fixed in `522b67b` by restoring the pre-merge file.
