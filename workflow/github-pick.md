# github-pick — shared workflow specification

> **Canonical, agent-neutral spec.** This is the single source of truth for the whole
> workflow. Any AI agent working inside this repo follows it — Claude reads it via
> `CLAUDE.md`, Codex / Cursor / Cline / Aider / others via `AGENTS.md`, or you can just
> tell any agent: "read `workflow/github-pick.md` and do it."
>
> Agents may differ only in *how* the work is dispatched — an agent that can spawn
> parallel subagents (e.g. Claude Code) fans the reviews out; one that can't runs them
> sequentially — never in *what* is produced.

---

## 1. Purpose & guiding principle

Curate the top GitHub repositories on a user-given topic into a localized "research
board". The output is a per-topic **session** (`sessions/<date>-<slug>.json` + `.md`)
that the zero-build dashboard (`index.html` + `data.js`) renders.

**Guiding principle — market reception, not birthday.**
The board's value is *"what is the current real-world reception of this project?"*,
not *"when was it created?"*. Therefore:

- **Never pre-filter candidates by date.** A 2019 project and a 2026 project are both
  eligible. Age is expressed *afterward* through tags, never used to exclude up front.
- Judgments must be grounded in real signals (stars + velocity, recent activity,
  awesome-list / newsletter mentions, HN/Reddit/知乎 discussion, integrations,
  successor projects), verified with web search where possible — not vibes.

---

## 2. Invocation & arguments

Triggered by any natural-language request to curate, survey, aggregate, or build a
"learning list" of open-source projects on a theme — e.g. "帮我聚合 PPT 生成 的开源项目"
or "curate the top RAG framework projects". The user gives a **topic**, and optionally
how many to keep and a star floor, in plain language.

| Argument | Required | Default | Meaning |
|---|---|---|---|
| `<topic>` | yes | — | Free text, any language (e.g. `PPT 生成`, `Claude Code skill`, `RAG framework`). |
| `--n <N>` | no | `30` | Target kept-project count. Treat as a soft target (range ~25–35); reviewers have final discretion. |
| `--stars <N>` | no | `1000` | Star floor for the main English queries. CJK/synonym queries may use a lower floor. |

**Clarify only if truly ambiguous.** If the topic is too broad to search meaningfully
(e.g. just "AI"), ask exactly **one** narrowing question, then proceed. For any concrete
topic, skip clarification entirely.

---

## 3. Language handling (read this before producing any text)

There is **one** language per installation, chosen once, then everything — digest
content *and* dashboard UI — stays in it forever. There is no runtime toggle.

### 3.1 Resolve the language

1. Read `config.json` at the repo root. Shape: `{ "lang": "<code>", "ui": <null | {key:label,...}> }`.
2. **First-run / missing config:** if `config.json` does not exist, or `lang` is unset/empty,
   you MUST **ask the user which language to use**, then write `config.json`
   before doing anything else. This is the one-time language choice for the whole project.
3. From that point on, `lang` is fixed. Use it for **all** produced content.

### 3.2 What "produced content" covers

Every human-readable string you write into a session is in `lang`:
`overview`, every category `name`, and per project `summary`, `why`, `tagReason`,
and the **label portion** of each tag.

### 3.3 Dashboard UI strings

The dashboard ships **built-in string tables for `zh` and `en`**. The build script
inlines `config.ui` into `data.js`; the dashboard uses the built-in table for the
active `lang` unless `config.ui` overrides it.

- If `lang` ∈ {`zh`, `en`}: leave `config.ui` as `null` — the built-in table is used.
- If `lang` is **not** built in (e.g. `ja`, `es`, `fr`): the agent MUST translate the
  full UI key set below into `lang` and write it into `config.ui` as a flat
  `{ key: label }` object (one-time, at setup). Keep the `{n}`, `{m}`, `{cmd}`
  placeholders intact.

UI keys to translate when `lang` is not built in:

| Key | English reference | Notes |
|---|---|---|
| `subtitle` | Open-source research notes | header subtitle |
| `meta_projects` | `{n} projects · {m} studied` | `{n}` total, `{m}` studied — keep placeholders |
| `session_count` | `{n}` | sidebar per-session count; keep `{n}` (e.g. zh = `{n} 项`) |
| `cat_all` | All | category "all" pill |
| `hide_studied` | Hide studied | toggle |
| `show_hidden` | Show hidden | toggle |
| `filtered_empty` | Everything in this category is filtered out. | |
| `no_projects` | No projects in this category. | |
| `card_created` | Created | card meta label |
| `card_updated` | Updated | card meta label |
| `badge_new` | New | "new" badge |
| `tag_reason_prefix` | `Tags: ` | prefix before tag reason |
| `btn_hide` | Hide | per-card button |
| `btn_studied` | Studied | per-card button |
| `empty_no_sessions` | No sessions yet. | empty state |
| `empty_run_hint` | `Open your AI agent in this folder and ask it to curate the top GitHub projects on a topic to start your first one.` | natural-language hint, no slash command |
| `empty_example` | `e.g. PPT generators · Claude Code skills · RAG frameworks` | example topics |

> The dashboard never switches languages at runtime; `config.json` is the single source
> of truth and is inlined into `data.js` by the build script.

---

## 4. Tag system (core output)

Each kept project gets an array of **0–3 tags** (up to 4). Tags are **additive** — one
project can be both `🏆Top` and `🔥Hot`. **No tags is valid** — a solid mid-tier project
that doesn't stand out on any axis simply carries no tag.

### 4.1 The emoji is the stable key; the label follows `lang`

A tag string = **leading emoji + localized label**. The dashboard maps a tag to its CSS
class by the **leading emoji only**, so any language works. Emit the correct emoji with
the label written in `lang`:

| Emoji (stable key) | `zh` label | `en` label | CSS class |
|---|---|---|---|
| `🔥` | `🔥火热` | `🔥Hot` | `tag-hot` |
| `🆕` | `🆕新` | `🆕New` | `tag-new` |
| `🏆` | `🏆一线` | `🏆Top` | `tag-top` |
| `📜` | `📜老` | `📜Legacy` | `tag-old` |

For a non-built-in `lang`, translate the label but **keep the same leading emoji**
(e.g. `🔥` + Japanese for "hot").

### 4.2 Strict tag definitions (apply against `today`)

- **`🔥` (Hot)** — trending *right now*: on GitHub Trending in the last ~30 days,
  pushed by multiple recent awesome-lists / newsletters, star velocity accelerating,
  HN/Reddit/知乎 discussion in the last month, or a key integration just landed
  (MCP, A2A, etc.).
- **`🆕` (New)** — `created_at` within the last **90 days** of `today`.
- **`🏆` (Top)** — what top practitioners actually default to: referenced by other
  popular projects, integrated into commercial products, cited in a16z/YC/industry
  newsletters, or `> 20k` stars **and still active** with the ecosystem revolving
  around it.
- **`📜` (Legacy)** — still works but has a clearer successor (e.g.
  `openai/swarm` → `openai-agents-python`), **or** `pushed_at` > 6 months ago,
  **or** maintenance visibly slowing.

---

## 5. The pipeline

### Step 1 — Resolve language (§3) and parse arguments
Read `config.json`; handle first-run. Parse `--n` / `--stars`. Compute `today`.

### Step 2 — Search GitHub broadly (no date filter)
Run **6–10 queries** covering, in combination:

- Direct keywords at several star floors: `<keyword> stars:>=5000`, `<keyword> stars:>=1000`.
- GitHub topic tags: `topic:<slug>`.
- **CJK / native-language variants** if the topic has a 中文 (or other non-English) name —
  these naturally surface a different population, so lower the floor (e.g. `stars:>=100`).
- Sibling / synonym concepts (e.g. `PPT 生成` → also `slides AI`, `presentation generator`).
- One "recent activity" query sorted by `updated` to catch active-but-low-star projects.

Prefer `gh api` (inherits auth, ~5000 req/hr):

```bash
gh api -X GET search/repositories -f q="<query>" -f sort=stars -f per_page=40
```

**Also fetch the obvious flagships by name** — don't trust search alone for the kings of
a space. Make a per-topic judgment of who the flagships are, then:

```bash
gh api repos/<owner>/<repo>
```

(e.g. for agent frameworks: Dify, LangChain, n8n, …). If your agent can issue tool calls
concurrently (e.g. Claude Code), run these queries in parallel; otherwise run them
sequentially — same query set either way.

### Step 3 — Build the candidate pool (~40–60 repos)
- Dedupe by `full_name`.
- Drop **archived** repos.
- Drop obvious irrelevants (awesome-list repos **unless** the topic *is* "awesome lists").
- Sort by stars descending.
- Keep the top **40–60** as raw candidates. **No date filter** — keep both 2019 and 2026 projects.

### Step 4 — Categorize (3–6 content-driven categories)
Invent **3–6** categories that the *topic itself* suggests; assign each candidate to
exactly **one**. Don't force predefined buckets.

> Example for "AI workflow design": Visual platforms / Agent-orchestration frameworks /
> Context & RAG / Automation engines / Real-world cases / Established standards.

### Step 5 — Review every category (KEEP/DROP + tag + write-up)
For **each category**, review its candidates. (If your agent supports parallel subagents,
fan out one reviewer per category at once for speed; otherwise walk the categories one at
a time. The decision logic and output are identical either way.)

For **each candidate** in the category:

1. **KEEP or DROP** — based on real relevance to the topic and category.
2. If KEEP, assign **0–3 tags** from the strict definitions in §4.2.
3. Write **`tag_reason`** — one line, citing concrete evidence (stars, push date,
   Trending, a named newsletter/thread). Only meaningful when tags are present.
4. Write **`summary`** — one sentence: *what it is*.
5. Write **`why`** — one line: *why it's worth a look*.
6. Optionally **recommend a replacement / addition** not in the candidate list if you
   know a clearly better fit.

Each reviewer may use its own tools (web search/fetch, extra `gh api` calls) to verify
tags and verdicts, and should report back, per category, a tight structured result:

```json
{
  "keep": [
    { "name": "owner/repo", "tags": ["🏆一线", "🔥火热"],
      "tag_reason": "...", "summary": "...", "why": "..." }
  ],
  "drop": [ { "name": "owner/repo", "reason": "why dropped" } ],
  "recommend_add": [
    { "name": "owner/repo", "reason": "why better", "replaces": "owner/repo|null" }
  ],
  "coverage_gaps": [ "missing the TypeScript camp", "only 3 RAG entries — thin" ],
  "suggest_queries": [ "typescript agent framework stars:>=5k" ]
}
```

> All written strings here are already in `lang` (the example shows `zh` labels).

### Step 6 — Synthesize (+ at most ONE follow-up round)
1. **Absorb `recommend_add`** — fetch each via `gh api repos/<name>` and add it with the
   recommender's proposed tags/summary/why.
2. **Evaluate `coverage_gaps`** — if a gap is real, run the `suggest_queries`
   (**cap at 3 extra queries total** across all categories) and re-review only the
   affected category. **Exactly one** such round — no loops.
3. **Cross-category tag calibration** — scan all tag assignments. A reader must not feel
   that `🔥` in category A means something looser than `🔥` in category B. Adjust outliers.
4. **Final count check** — aim for the `--n` target (~25–35). Too many → demote borderline
   ones (drop the un-`🏆` ones first). Too few → accept the smaller count or run one
   targeted query.

### Step 7 — Write the session files
Compute:
- `date` = today, `YYYY-MM-DD` (`date +%Y-%m-%d`).
- `slug` = topic, spaces → `-`, lowercased, CJK characters preserved.
- If `sessions/<date>-<slug>.json` already exists, append `-2` / `-3` (unless the user
  said "redo"). **Never mutate** an existing session file — only add new ones.

Write `sessions/<date>-<slug>.json` and `sessions/<date>-<slug>.md` per §6.

### Step 8 — Regenerate `data.js` (do NOT hand-write it)
Run the build script — it inlines `config.json` and all sessions, and validates:

```bash
node scripts/build-data.mjs
```

This regenerates `data.js` as exactly three logical parts:

```javascript
// auto-generated by github-pick — do not edit by hand
const CONFIG = {"lang":"zh","ui":null};
const SESSIONS = [ /* all sessions, sorted by date DESC then slug ASC */ ];
```

Never edit `data.js` by hand. If the build script reports a validation error, fix the
offending session JSON and re-run.

### Step 9 — Report
One short paragraph in `lang`:
- File paths written.
- Count line: "X projects across Y categories — 🔥Hot A · 🏆Top B · 🆕New C · 📜Legacy D".
- Review summary: "Y categories reviewed; M gap-fill repos pulled in."
- Reminder: refreshing `index.html` shows the new session.

---

## 6. Output contract

### 6.1 Session JSON schema

```json
{
  "topic": "<original topic string>",
  "slug": "<slug>",
  "date": "YYYY-MM-DD",
  "lang": "<code>",
  "overview": "<2–3 sentence overview in lang>",
  "categories": [
    {
      "name": "<category name in lang>",
      "projects": [
        {
          "name": "owner/repo",
          "url": "https://github.com/owner/repo",
          "stars": 12345,
          "language": "Python",
          "createdAt": "YYYY-MM-DD",
          "pushedAt": "YYYY-MM-DD",
          "tags": ["🏆一线", "🔥火热"],
          "tagReason": "<one line in lang citing evidence>",
          "summary": "<one sentence in lang: what it is>",
          "why": "<one line in lang: why worth a look>"
        }
      ]
    }
  ]
}
```

Field notes:
- `lang` is **optional**; when omitted it defaults to `CONFIG.lang`. Include it for clarity.
- `language` may be `null` (some repos have no detected language).
- `tags` may be `[]`.
- `createdAt` / `pushedAt` are `YYYY-MM-DD` dates derived from the `gh api` timestamps.

#### CRITICAL JSON rule (CJK strings)
Inside **any CJK string value**, NEVER use an ASCII double-quote (`"`) for emphasis —
use the full-width brackets `「」` instead. ASCII quotes break JSON parsing. (This is why
the examples above write 「…」.)

### 6.2 Session Markdown shape

`sessions/<date>-<slug>.md` is the human-readable mirror (labels follow `lang`; shown
here with `zh` labels):

```markdown
# <Topic> · <Date>

> <overview>

## <Category 1>

### <owner/repo> ⭐ <stars> <tag chips inline>
- **简介**: <summary>
- **值得看**: <why>
- **标签由来**: <tagReason>   (only when tags are present)
- **语言**: <language> · **创建**: <YYYY> · **更新**: <YYYY-MM-DD> · **链接**: <url>
```

---

## 7. Running on different agents

This one pipeline runs on any capable coding agent. The only thing that changes is **how
the work is dispatched** — never *what* is produced:

- **Agents that can spawn parallel subagents** (e.g. Claude Code's Task tool): fan out
  **one reviewer per category** at once (Step 5), and run the search queries (Step 2)
  concurrently. Fastest path.
- **Single-thread agents** (e.g. Codex, or any agent without subagents): run the same
  steps **sequentially** — loop over categories, run searches one after another. Same
  query set, same output.

Either way the search query set, candidate-pool rules, tag definitions, synthesis logic,
and the final session JSON / Markdown / `data.js` output are **identical**. Always finish
by running `node scripts/build-data.mjs`; never hand-write `data.js`.
