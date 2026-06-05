# github-pick

> **Clone 下来，在项目文件夹里打开任意一个 AI，直接说一句「帮我聚合 X 主题的开源项目」，**
> 就能得到一个本地化、双击即开的开源研究看板。AI 广搜 GitHub、判断每个项目的真实市场反响、
> 打标签、写本地化点评，一个零构建的静态看板（`index.html` + `data.js`）负责展示。零构建、
> 不要 API key、不用安装——这个仓库本身就是工作区。

🌏 [English](./README.md) · 🖥 [在线页面](https://leifdiao.github.io/github-pick/) · ⚖️ [许可](./LICENSE)

---

**github-pick** 把「我想了解 X 领域的开源生态」变成一块干净、易扫读的看板。你给一个主题，
AI 广搜 GitHub、判断每个项目的真实市场反响、打标签、写中文（或你选的语言）点评。一个零构建的
静态看板（`index.html` + `data.js`）负责展示——双击打开，不用起服务器、不用构建。

它**不是插件，也不是你要安装到某处的 skill**。它就是一个**带说明书的项目文件夹**：`AGENTS.md` /
`CLAUDE.md` 是任何现代 AI 编程 agent 打开文件夹时会自动读的说明。**这个仓库本身就是工作区。**

---

## 用起来就一句话

```
clone 仓库  ──▶  在文件夹里打开 AI  ──▶  「帮我聚合 X 主题的开源项目」
                                              │
      AI 读 AGENTS.md + workflow/github-pick.md ◀┘
                                              │
      搜 GitHub · 打标签 · 写 sessions/*.json ──┤
                                              │
      跑 `node scripts/build-data.mjs` → data.js ──▶  打开 index.html ✦
```

**不用安装、不用 API key、不往任何全局目录塞东西。** 所有东西都在 clone 下来的文件夹里。

## 前置条件

- **一个会读 `AGENTS.md` / `CLAUDE.md` 的 AI 编程 agent**——比如 **Claude Code**、
  **Codex CLI**、Cursor、Cline、Aider。
- **Node.js**（任意较新版本）——给那个零依赖的构建脚本用。
- **`gh` CLI**（可选）——装了并登录的话，GitHub 搜索额度是 5000 次/小时，否则匿名 60 次/小时；
  跑单期都够。

## 快速开始

```bash
git clone https://github.com/LeifDiao/github-pick.git
cd github-pick
# 在这个文件夹里打开 AI，然后直接说人话：
#   「帮我聚合 RAG 框架 的开源项目」
#   「curate the top Claude Code skill projects」
```

**第一次**运行时，AI 会问你想用哪种语言。选完就锁定（见下），之后不再问。跑完打开 `index.html`。

> 想先手动配？改 `config.json`（`"lang": "zh"`）再跑 `node scripts/build-data.mjs` 即可。

## 语言——选一次，终身锁定

每个项目一种语言，第一次运行时选定，存进 `config.json`。此后**所有东西**——每一期 digest 和
看板 UI——都用这个语言。没有运行时切换。

- `zh` 和 `en` 自带看板 UI 文案。
- 其他语言（`ja`、`es`、`fr`…）也行：AI 第一次会把 UI 文案翻译进 `config.json`。
- 以后想换：改 `config.json` 再跑 `node scripts/build-data.mjs`。（已有 session 保留当初写的
  语言，新 session 用新语言。）

## 看板

- 双击 `index.html`——通过 `file://` 打开，不用起服务器。
- 左侧栏列出各期 session；项目按分类分组、带标签。
- 每张卡片上的按钮可把项目标为**已学**或**隐藏**（存在浏览器 `localStorage` 里，进度私有、本地）。

### 标签含义

| 标签 | 含义 |
|---|---|
| 🔥火热 | 当下正热（近期热度、讨论、集成） |
| 🆕新 | 最近创建 |
| 🏆一线 | 顶级——一线从业者真在用的 |
| 📜老 | 还能用，但有更明确的继任者 / 维护放缓 |

标签与语言无关：emoji 是稳定的 key，只有文字部分会翻译。

---

## 目录结构

```
github-pick/
├── AGENTS.md                 # 任意 agent 都会读的入口（Codex/Cursor/Cline/Aider…）
├── CLAUDE.md                 # 让 Claude Code 读 AGENTS.md
├── workflow/github-pick.md   # 平台中立的工作流规范（唯一事实源）
├── config.json               # { lang, ui } —— 一次性的语言选择
├── index.html                # 看板（双击即开）
├── data.js                   # 生成物，别手改
├── scripts/build-data.mjs    # 零依赖：sessions/*.json + config → data.js
├── sessions/                 # 每期一个 <date>-<slug>.json（+ .md）
└── LICENSE
```

## 数据怎么流动

`sessions/*.json`（每个主题一个，由 agent 写）→ `node scripts/build-data.mjs` 把
`config.json` + 所有 session 内联进 → `data.js` → `index.html` 读它。构建脚本是**唯一**会写
`data.js` 的东西；agent 从不手改它。

## 跟任何 agent 配合

「大脑」是一份朴素的规范（`workflow/github-pick.md`），任何有能力的 agent 都能照做。区别只在于
*怎么派活*——能开并行子 agent 的（如 Claude Code）会把分类评审并行铺开提速；单线程的（如 Codex）
顺序跑。产出完全一样。

## 参与贡献

欢迎提 Issue 和 PR——见 [CONTRIBUTING.md](./CONTRIBUTING.md)。工作流逻辑集中在
`workflow/github-pick.md`，改行为就改那里。别手改 `data.js`，用构建脚本重新生成。

## 许可

[MIT](./LICENSE)
