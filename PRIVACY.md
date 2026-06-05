# Privacy

github-pick runs on your machine. Its build script and dashboard work entirely locally; the
only thing that touches the network is the AI agent **you** run when it searches GitHub and
the web to curate a topic — using **your own** access.

## Data Read

- The **topic** you ask for (e.g. "RAG frameworks", "帮我聚合 PPT 生成的开源项目"), plus any
  optional `--n` / `--stars` you pass.
- `config.json` — your one-time language choice (and translated UI labels for non-`zh`/`en`
  languages).
- `sessions/*.json` — the curated sessions already in your checkout.

## Network & External Services

- **github-pick's own code makes no network calls.** `scripts/build-data.mjs` is offline
  Node; it only reads local files and writes `data.js`.
- **The AI agent you run does the searching.** To curate a topic, the agent sends your
  topic / search terms to **GitHub** (search and repository APIs) and, where it verifies
  reception, to **web search / the pages it fetches**. This uses your own `gh` CLI or your
  agent's web tools and your own GitHub access — github-pick supplies no key. GitHub and any
  sites visited are operated by third parties and governed by **their own privacy policies
  and terms of service**; the terms you search for are subject to those policies.
- **The dashboard loads one web font** (LXGW WenKai) from a public CDN (jsDelivr) when you
  open `index.html` with a network connection. That request goes to the CDN, not to
  github-pick. Remove the font `<link>` in `index.html` if you want the page fully offline.
- github-pick uploads **no telemetry or analytics** of its own.

## Data Written

- `sessions/<date>-<slug>.json` (+ `.md`) — one per topic run, written into your checkout.
- `data.js` — generated locally by the build script from your config + sessions.
- **Browser `localStorage`** — the dashboard stores your per-project progress and view
  preferences (keys like `studied:<name>`, `hidden:<name>`, `ghpick:*`). This stays in your
  browser, is private to your machine, and never leaves it.

## Sharing

The dashboard is self-contained: `index.html` + `data.js` render without a server. If you
send those files to someone, they contain the curated digests (project names, descriptions,
and your notes) — share them only when you mean to. Your `localStorage` progress is **not**
included; it stays in your own browser.
