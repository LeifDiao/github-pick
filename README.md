# github-pick

> **Clone it, open any AI agent inside the folder, and just say *"curate the top GitHub
> projects on X."*** You get a localized, double-click-to-open research board — the agent
> searches GitHub broadly, judges each project's real-world reception, tags it, and writes
> a localized digest. Zero build, zero API key, nothing to install. The repo **is** the workspace.

🌏 [中文版](./README_zh.md) · 🖥 [Live page](https://leifdiao.github.io/github-pick/) · ⚖️ [License](./LICENSE)

> **中文简介：** Clone 下来，在项目文件夹里打开任意一个 AI，直接说一句「帮我聚合 X 主题的开源项目」，就能得到一个本地化、双击即开的开源研究看板。AI 广搜 GitHub、判断每个项目的真实反响、打标签、写本地化点评，一个零构建的静态看板负责展示。完整中文文档 → [README_zh.md](./README_zh.md)

---

**github-pick** turns *"I want to learn the open-source landscape of X"* into a clean,
scannable dashboard. You give a topic; the AI searches GitHub broadly, judges each
project's real-world reception, tags it, and writes a localized digest. A zero-build
static dashboard (`index.html` + `data.js`) renders it — double-click, no server, no
build step.

It is **not a plugin and not a skill you install anywhere.** It is just a project folder
with an instruction file (`AGENTS.md` / `CLAUDE.md`) that any modern AI coding agent reads
automatically. The repo **is** the workspace.

---

## How it works in one picture

```
clone repo  ──▶  open an AI agent in the folder  ──▶  "帮我聚合 X 主题的开源项目"
                                                              │
        AI reads AGENTS.md + workflow/github-pick.md ◀────────┘
                                                              │
        searches GitHub · tags · writes sessions/*.json ──────┤
                                                              │
        runs `node scripts/build-data.mjs` → data.js ────────▶  open index.html ✦
```

No installation. No API key. No global config. Everything lives in the cloned folder.

---

## Requirements

- **An AI coding agent** that reads `AGENTS.md` or `CLAUDE.md` — e.g. **Claude Code**,
  **Codex CLI**, Cursor, Cline, Aider.
- **Node.js** (any recent version) — for the tiny zero-dependency build script.
- **`gh` CLI** *(optional)* — if installed and authenticated, GitHub search gets 5000
  req/hr instead of the anonymous 60/hr. Either works for a single run.

## Quick start

```bash
git clone https://github.com/LeifDiao/github-pick.git
cd github-pick
# open your AI agent IN this folder, then just talk:
#   "帮我聚合 RAG 框架 的开源项目"
#   "curate the top Claude Code skill projects"
```

The **first time** you run it, the agent asks which language you want. After that the
choice is locked (see below) and it never asks again. When it finishes, open `index.html`.

> Prefer to set things up by hand first? Just edit `config.json` (`"lang": "en"`) and run
> `node scripts/build-data.mjs`.

## Language — pick once, stay forever

One language per project, chosen on first run, stored in `config.json`. From then on
**everything** — every digest and the dashboard UI — stays in that language. There is no
runtime toggle.

- `zh` and `en` ship with built-in dashboard UI labels.
- Any other language (`ja`, `es`, `fr`, …) works too: the agent translates the UI labels
  into `config.json` the first time.
- To change later: edit `config.json` and re-run `node scripts/build-data.mjs`. (Existing
  sessions keep the language they were written in; new ones use the new language.)

## The dashboard

- Double-click `index.html` — opens via `file://`, no server.
- Sessions are listed in the sidebar; projects are grouped into categories with tags.
- Per-card buttons mark a project **studied** or **hidden** (stored in your browser's
  `localStorage`, so your progress is private and local).

### Tag legend

| Tag | Meaning |
|---|---|
| 🔥 | Trending right now (recent momentum, discussion, integrations) |
| 🆕 | Created recently |
| 🏆 | Top-tier — what leading practitioners actually use |
| 📜 | Still works, but has clearer successors / slowing maintenance |

Tags are language-agnostic: the emoji is the stable key, only the label is translated.

---

## Project structure

```
github-pick/
├── AGENTS.md                 # entry point any agent reads (Codex/Cursor/Cline/Aider…)
├── CLAUDE.md                 # makes Claude Code read AGENTS.md
├── workflow/github-pick.md   # canonical, agent-neutral workflow spec (source of truth)
├── config.json               # { lang, ui } — the one-time language choice
├── index.html                # the dashboard (double-click to open)
├── data.js                   # generated — do not hand-edit
├── scripts/build-data.mjs    # zero-dependency: sessions/*.json + config → data.js
├── sessions/                 # one <date>-<slug>.json (+ .md) per topic run
└── LICENSE
```

## How the data flows

`sessions/*.json` (one per topic, written by the agent) → `node scripts/build-data.mjs`
inlines `config.json` + all sessions into → `data.js` → `index.html` reads it. The build
script is the **only** thing that writes `data.js`; agents never hand-edit it.

## Works with any agent

The "brain" is a plain spec (`workflow/github-pick.md`) that any capable agent can follow.
Agents differ only in *how* they dispatch the work — one that can spawn parallel subagents
(e.g. Claude Code) fans the category reviews out for speed; a single-thread agent (e.g.
Codex) runs them sequentially. The output is identical either way.

## Contributing

Issues and PRs welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md). The workflow logic is
centralized in `workflow/github-pick.md`; change behavior there. Keep `data.js` out of
manual edits; regenerate it with the build script.

## License

[MIT](./LICENSE)
