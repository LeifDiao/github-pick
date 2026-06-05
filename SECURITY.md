# Security

github-pick is almost entirely local: a zero-dependency Node build script
(`scripts/build-data.mjs`) turns `config.json` + `sessions/*.json` into `data.js`, and a
static dashboard (`index.html`) renders it from `file://`. github-pick ships no
credentials and stores none.

## Reporting Issues

If you find a security issue, please open a private report through GitHub security
advisories when available, or contact the maintainer through the GitHub profile linked in
this repository. Please do not file public issues for security-sensitive reports.

## Trust model

The actual GitHub searching and judging is done by **whatever AI agent you run** inside the
folder, using **your own** `gh` CLI / agent web tools and your own GitHub access. github-pick
itself never asks for, reads, or stores a token. The build script makes **no network
calls** at all. The dashboard runs locally and stores per-project "studied" / "hidden"
state in your browser's `localStorage`.

## Security-sensitive areas

- **Dashboard rendering of session content.** Project names, descriptions, and digests in
  `sessions/*.json` originate from external sources (GitHub repositories) and are
  attacker-influenceable text. The dashboard currently renders some of this content with
  `innerHTML`, so a repo field containing markup could inject HTML into the local page.
  Because sessions are produced by your own trusted agent run and the page is opened
  locally, the practical exposure is limited — but **escaping repo-derived text before it
  reaches the DOM is a known hardening area**, and a change that introduces or worsens
  unescaped rendering is in scope.
- **Build-time validation.** `scripts/build-data.mjs` inlines session JSON into `data.js`.
  It must reject malformed input rather than emit a broken or injectable `data.js`.
- **Local path handling** when reading `sessions/*` and writing `data.js` (e.g. path
  traversal via a crafted session filename or slug).
- **External resources in the dashboard.** `index.html` loads one web font from a public
  CDN (jsDelivr). Adding further third-party scripts or trackers to the page is in scope;
  prefer keeping the dashboard self-contained.
- **Credential handling.** github-pick stores no GitHub tokens. The optional `gh` CLI uses
  your own authenticated session; do not commit tokens, `.env` files, or any credentials.

## What github-pick does to limit exposure

It keeps the build path zero-dependency and offline, never persists credentials, confines
user progress to local `localStorage`, and keeps generated data (`data.js`) and any private
sessions inside your own checkout.
