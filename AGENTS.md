# github-pick — instructions for AI agents

You are working inside **github-pick**, a project that turns a topic into a curated,
localized "research board" of the top open-source GitHub projects on that topic.

**This file is read automatically** by Codex, Cursor, Cline, Aider, and most coding
agents when they open this repo. (Claude Code reads it via `CLAUDE.md`.) It tells you
*when* to act and the hard rules; the full step-by-step procedure lives in
**`workflow/github-pick.md`** — read that file before doing the task.

There is nothing to install. The user clones this repo into a normal folder, opens an AI
agent **inside the folder**, and just talks. You read the project and do the work; results
are written back into this same folder.

## When to act

When the user asks you — in **any language** — to **curate / aggregate / survey / pick /
build a learning list of** open-source GitHub projects on a topic (e.g. "帮我聚合 PPT 生成
的开源项目", "find the best RAG frameworks", "选一批 Claude skill 的项目"):

1. **Read `workflow/github-pick.md`** — it is the canonical spec (search strategy, tag
   system, categories, output schema). Follow it exactly.
2. The user gives a **topic**, optionally how many to keep (`--n`, default 30) and a star
   floor (`--stars`, default 1000), in plain words.

## Language — chosen once, then locked

- Read `config.json`: `{ "lang": "<code>", "ui": <null | {key:label}> }`.
- **If `lang` is empty/missing, ASK the user which language to use, then write
  `config.json`** before producing anything. This is a one-time choice for the project.
- After that, produce **everything** in `config.lang` — digest content *and* dashboard UI
  labels alike. There is no per-run switching; the next time you run, you read the same
  `config.json` and keep the same language without asking again.
- `zh` and `en` have built-in dashboard UI tables (leave `config.ui` = `null`). For any
  other language, also translate the UI key set (listed in `workflow/github-pick.md`)
  into `config.ui`.

## Where things live

- `sessions/<date>-<slug>.json` (+ `.md`) — one per topic run; the JSON files are the data
  of record. **Never mutate old ones — only add new ones.**
- `scripts/build-data.mjs` — zero-dependency Node script: reads `config.json` +
  `sessions/*.json` → regenerates `data.js` (with validation).
- `data.js` — **generated only** by that script. **Never hand-edit it.**
- `index.html` — the dashboard; double-click to open (loads `data.js` via `<script>`, so
  config is inlined, not fetched — no server needed).

## Hard rules

- **Finish every run with `node scripts/build-data.mjs`** to regenerate `data.js`. Never
  hand-write `data.js`.
- In JSON session files, **CJK string values use 「」, never ASCII double-quotes** for
  emphasis — ASCII quotes break JSON parsing.
- **Tags keep their leading emoji** (🔥 / 🆕 / 🏆 / 📜); the dashboard styles a tag by its
  emoji, so only the label is translated.
- When done, tell the user to open / refresh `index.html`.
