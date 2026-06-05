# Contributing

Contributions are welcome.

## What this project is

github-pick is **not** a compiled app or a published package. It is a project folder an AI
agent reads: an agent-neutral spec (`workflow/github-pick.md`), entry-point instructions
(`AGENTS.md` / `CLAUDE.md`), a zero-dependency Node build script
(`scripts/build-data.mjs`), and a static dashboard (`index.html`). There is nothing to
install and no server.

## Where to make changes

- **Behavior / workflow** (search strategy, tag system, categories, output schema) lives
  in `workflow/github-pick.md` — it is the **single source of truth**. Change it there.
  `AGENTS.md` only points to it; keep the two consistent if you touch the entry points.
- **Build / data shape** lives in `scripts/build-data.mjs` (reads `config.json` +
  `sessions/*.json`, validates, regenerates `data.js`).
- **Dashboard** lives in `index.html` (renders `data.js`).
- **`data.js` is generated** — never hand-edit it; regenerate with the build script.

## Development checks

The build script is plain Node with **zero dependencies** — nothing to install.

```bash
# Regenerate data.js from config.json + sessions/*.json (run after any change)
node scripts/build-data.mjs

# Then open the dashboard (no server needed — it loads via file://)
open index.html        # macOS;  use xdg-open / start on Linux / Windows
```

To test a change end to end, drop a small sample session into `sessions/` (a
`<date>-<slug>.json`), run the build, and confirm it renders.

## Session data rules

`sessions/<date>-<slug>.json` files are the **data of record**. The build only inlines
them; it never invents data.

- **Never mutate old sessions** — only add new ones. Each topic run is one session.
- In JSON string values, **CJK emphasis uses 「」, never ASCII double-quotes** — ASCII
  quotes break JSON parsing.
- **Tags keep their leading emoji** (🔥 / 🆕 / 🏆 / 📜); the dashboard styles a tag by its
  emoji, so only the label is translated.

## Design principles

- **Market reception, not birthday.** Judge a project by its current real-world reception
  (stars + velocity, recent activity, awesome-list / newsletter mentions, HN/Reddit/知乎
  discussion, integrations, successors) — never pre-filter candidates by creation date.
  Age is expressed *afterward* through tags, never used to exclude up front.
- **Ground judgments in real signals**, verified with web search where possible — not
  vibes, and never invented numbers.
- **Stay zero-build and zero-dependency.** The whole point is double-click-to-open. Don't
  add a bundler, a framework, or a runtime dependency unless it removes meaningful
  complexity.
- **One language per project, chosen once.** Everything — digests and dashboard UI — stays
  in `config.lang`. Don't add a runtime language toggle.
- **Finish every run with `node scripts/build-data.mjs`.** Never hand-write `data.js`.
