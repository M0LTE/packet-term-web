# CLAUDE.md

Operating notes for Claude Code (and other agents) working in `m0lte/packet-term-web`.

## What this repo is

A self-contained browser-based AX.25 terminal app — single `index.html` file, no build step, no bundler. Late-80s phosphor-CRT aesthetic, TNC2 command set. Imports `@packet-net/ax25` and `xterm.js` from [esm.sh](https://esm.sh) at runtime.

Extracted from `m0lte/packet.net` on 2026-05-17 (originally `web/ax25/examples/packet-terminal/`) so it can have its own deploy cadence + issue tracker. Live at https://packet-term.m0lte.uk.

## How to work in here

There is **no build step**. Just edit `index.html` and reload in a Chromium browser. The page imports its deps from esm.sh at runtime, so a Chrome refresh is the full feedback loop.

- **Test locally** by serving the file over HTTP (Web Serial requires a secure context — `localhost` qualifies; `file://` does not):
  ```sh
  python3 -m http.server 8000
  # then open http://localhost:8000/
  ```
- **Deploy** to https://packet-term.m0lte.uk via the OARC custom static hosting. See the `oarc-static-upload` Claude skill if you have it; otherwise manual upload through OARC's web UI.

## Hard rules

### Don't bundle, don't add build tooling

The whole point of this demo is "single HTML file, no build, no package.json, no node_modules". If you find yourself reaching for vite / webpack / rollup / npm — stop and ask. The ESM-via-CDN approach is deliberate. It lets non-developers fork the file, change a colour, drop it on any static host.

### Pin npm versions explicitly in the import URL

The page imports `@packet-net/ax25@0.2.1` (or whatever the current pin is) — never `@latest`. Pinning means the deployed page doesn't break silently when an upstream publishes a bad release. When bumping, do it in one PR + verify in a browser before merging.

### Web Serial is Chromium-only

Don't try to feature-detect-and-polyfill Firefox/Safari support. They don't support Web Serial; the only honest UI is the existing "no web serial" modal. If a user wants to use Firefox / Safari, they need a different transport (which this demo doesn't have).

### The deploy is publicly visible

This is a public-facing site. Any change you push and deploy affects whoever's using it right now. Test locally first.

## Things to avoid

- **Don't move runtime logic into the HTML.** Anything that would belong in the library (`@packet-net/ax25`) belongs upstream at the [`ax25-ts` repo (when extracted)](https://github.com/m0lte/ax25-ts) — not here. The demo's JS is UI / TNC2-command-parsing glue only.
- **Don't add a backend.** This is a static site by design.
- **Don't commit secrets** (deploy credentials, API tokens). The `oarc-static-upload` skill keeps OARC creds in `~/.config/oarc/` outside the repo.

## When in doubt

The other side of every interesting decision lives upstream in `@packet-net/ax25`. If a change feels like "the library should expose X so the demo can do Y", file it against the library, not here.
