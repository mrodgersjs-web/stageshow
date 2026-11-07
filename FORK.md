# Fork notes — mrodgersjs-web/stageshow

Fork of <https://github.com/benjitaylor/skills> (the `stageshow` skill lives at the repo root
upstream). Forked 2026-11-07 at upstream `299140f` ("Stageshow 1.1.1: fix playground shake with
always-on scroll bars"), cloned to `~/Developer/stageshow`.

Where it is wired in on this machine:

- symlink `~/.aside/u/0/skills/user/stageshow` -> `~/Developer/stageshow/stageshow`
- registered as layer-1 meta `/stageshow` in the RIG skill surface
  (`~/.aside/u/0/skill-library/_manifest.json`, `META_INFO` / `OWNERS` in
  `~/Developer/rig-capabilities/bin/skill-surface`, and its `SKILL_SURFACE` menu block in
  `~/.aside/u/0/AGENTS.md`)

`LICENSE` and `NOTICE.md` are untouched: the kit's MIT licence does not cover the app icons in
`kit/brands/` (CC0 / MIT / CC-BY sources, per-file hashes in `kit/brands/SOURCES.md`), and a
`Phone frame="photo"` needs the user's own bezel photo.

## Divergence from upstream

1. `playwright-core` `~1.48.2` -> `^1.65.0`. The old pin launches with `--headless=old`, and
   Chrome removed old headless in 132, so on this machine's Chrome 141 every shot script
   (`scripts/film.mjs`, `shots.mjs`, `layout.mjs`) fails at `browserType.launch` with
   `Target page, context or browser has been closed` — and the script's own message blames a
   missing Chrome. The pin is bumped so the kit can drive a current browser. Not proven end to
   end here (see the sandbox note below): the failure was reproduced with the old pin, the fix
   needs one real film run in a normal terminal.

## Install and run notes (macOS, agent-spawned shell)

- `npm install` fails in esbuild's postinstall (exit 1 / SIGKILL) when the shell is sandboxed or
  the downloaded binary carries `com.apple.quarantine`. Install with `npm install --ignore-scripts`,
  then `xattr -dr com.apple.quarantine node_modules`, then `node node_modules/esbuild/install.js`.
  Verified: esbuild 0.25.12 and vite 6.4.5 both run afterwards; `npm run dev` serves the playground
  (portless proxy on 1355, vite on 4307, `http://stageshow.localhost:1355`).
- Chrome cannot be launched from inside the Aside agent shell at all: `Google Chrome --headless=new`
  exits 134 (SIGABRT) with empty stderr, and playwright reports `kill EPERM`. So the shot scripts
  were not exercised here. Run `scripts/film.mjs` / `shots.mjs` / `layout.mjs` from a normal
  terminal, or set `CHROME_PATH` to a browser that can start in the shell you are in.
- The playground itself was verified in a real browser (Aside): `http://127.0.0.1:4307/?capture=hero-demo`
  mounts the example as an act-able stage — 1 `.ss-stage`, 1 `.ss-fit`, 51 stylesheets, live text.

## Syncing upstream

```sh
git fetch upstream && git merge upstream/main   # origin = fork, upstream = benjitaylor/skills
```

## Running the shot scripts here (resolved 2026-11-07, later the same day)

The earlier note in this file said the shot scripts were not exercised because Chrome aborts
inside the Aside agent shell (`exitCode=null, signal=SIGABRT`, and Playwright cannot even signal
it: `exception while trying to kill process: Error: kill EPERM`). That is the agent sandbox's
native-binary boundary, not a kit defect and not the pin.

It runs after all, with no fork edit, because `scripts/browser.mjs` already honours `CHROME_PATH`:

    export CHROME_PATH="$HOME/Library/Caches/ms-playwright/chromium_headless_shell-1248/chrome-headless-shell-mac-arm64/chrome-headless-shell"
    npm run dev &
    node scripts/film.mjs "http://127.0.0.1:4307/?capture=hero-demo" --out /tmp/ss-film.png --every 600 --count 6 --delay 500

Playwright's own headless shell survives the sandbox where the desktop Chrome binary does not;
`executablePath` bypasses the exact-revision check, so the cached 1248 shell serves playwright
1.65 (if a later playwright refuses, `npx playwright install chromium-headless-shell` and point
`CHROME_PATH` at the new directory).

What that run proved, on this machine, 2026-11-07:

- `npm run typecheck` - clean.
- `npm run check:css` - clean: "class clash: no overlapping class namespaces detected (83 CSS files, 2 explicit bundles)".
- `scripts/film.mjs` - shot a real contact sheet (6 frames at 600ms, 203 KB) and printed its own
  coverage line: `coverage stage-1: 256 frames, 4.25s, largest gap 39.0ms (inconclusive)`.
- `scripts/film.mjs --gate` - the rehearsal gate works and is honest: it failed the run on one
  never-intentional line (`tight control: div.hd-popup-btn "Add" sits 6.0px under ... a button keeps
  8px or more from the field`) and reported the caret's blink as a jump for a human to judge, rather
  than passing it. `rehearsal failed: 1 never-intentional line(s) above`.

So: install path is `npm install --ignore-scripts`, `xattr -dr com.apple.quarantine node_modules`,
and `CHROME_PATH` at a Playwright headless shell for anything that shoots.
