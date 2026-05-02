---
name: extract-system-knowledge
description: 从 Confluence 文档树（含子文档）和当前项目代码中提取系统知识，聚焦"概念 / 核心关系 / 核心逻辑"三层；文档与代码冲突时以代码为准并标注差异。当用户提供 Confluence URL 并要求"提取系统知识 / 梳理系统 / 建立项目认知 / 理解这个系统 / 理解登录系统"等时使用。
---

# Extract System Knowledge：系统知识提取

把"一组 Confluence 文档 + 一个项目代码库"提炼成可被未来复用的系统知识包：**概念**（这个系统在说什么）、**关系**（谁和谁怎么联系）、**逻辑**（核心流程怎么跑）。文档会过时，代码不会撒谎——必要时以代码为准。

---

## 触发场景

- 用户给出一个或多个 `confluence.zhenguanyu.com` 页面，说"帮我理解这个系统 / 梳理一下 / 提取知识"
- 用户在某个项目目录下，说"建立对 `<模块名>` 的认知"或"我要接手这块业务，先帮我整理"
- 用户说"提取系统知识"、"系统梳理"、"系统认知"、"系统知识包"、`/extract-system-knowledge`

## 执行流程

### 步骤 1：对齐目标边界

先用一段话向用户复述要提取什么，避免漫无目的爬文档。

```
我将提取的"系统"是：<系统/模块名，例：Focus 工作台>
- 主入口文档：<URL 列表>（仅向下抓子页，不向上抓父页）
- 项目代码范围：<repo 路径或 "无（仅文档）">
- 关注点：概念 / 关系 / 逻辑（默认全部）
- 子文档递归深度：默认 2 层
```

如有 `AskQuestion` 工具，对"代码范围"和"递归深度"做结构化询问；否则文本中直接问。用户已明确指定的字段直接采用，跳过询问。

### 步骤 2：抓取 Confluence 文档（仅当前页 + 子页）

**只抓两个范围：用户给的入口页本身、它的直接/递归子页。绝不向上反查 ancestors/父级，避免越抓越散。**

抓取首选 `tutor-wb-mcp` 提供的 MCP 工具（直接返回结构化 Markdown，无需截图、不消耗浏览器）：

| 工具 | 何时用 | 入参 |
| --- | --- | --- |
| `get_confluence_content` | URL 含 `pageId=` 参数，或已知数字 ID | `{"pageId": "..."}` 或 `{"page": "<url>"}` |
| `get_confluence_content_by_title` | URL 是 `/display/<SpaceKey>/<Title>` 格式 | `{"page": "<url>"}` 或 `{"spaceKey": "...", "title": "..."}` |

返回包含 `元数据`（`id` / `title` / `author` / `updated` / `url`）、`Markdown 内容`、`HTML 内容` 三段；**优先读 Markdown**，只有 Markdown 为空（如目录页）时才看 HTML。

**列子页**：MCP 工具不返回子页清单，需要走 Confluence REST API（已实测可用）：

```
GET https://confluence.zhenguanyu.com/rest/api/content/{pageId}/child/page?limit=50
```

关注 `size` 和 `results[].{id,title,_links.webui}`；`size > 50` 时分页 `?start=50&limit=50`。未登录会被重定向到 `login.zhenguanyu.com`，此时提示用户完成 SSO 后再继续。

**抓取流程**：

1. 对每个入口 URL 调用合适的 MCP 工具，取入口页正文与元数据
2. 调 `child/page` REST API 列直接子页：
   - `size == 0`：当前是叶子页，**到此为止**，只用入口页内容即可
   - `size > 0`：对每个子页递归回到第 1 步（默认深度 2 层；**单次会话子文档总数 ≤ 30**，超出主动询问用户是否继续）
3. 每篇文档收集：`pageId` / `title` / `updated` / Markdown 要点 / 嵌入图片（图片用 vision 看一遍写要点）

> 输出：内部维护一份"文档清单"，含 `pageId`、`title`、URL、要点摘要、出处链。

### 步骤 3：扫描项目代码（如果用户在项目下）

只在用户提供了项目路径或当前 workspace 是相关 repo 时执行。

1. 用 `SemanticSearch` 搜文档中的核心名词（实体名、流程名、接口名）落到了哪些文件
2. 用 `Grep` 验证关键字面量（接口路径、错误码、配置 key、SQL 表名）
3. 对每条文档要点标注：
   - `[文档=代码]`：一致
   - `[文档过时]`：代码已变（标注新实现位置）
   - `[文档独有]`：代码未实现或已删除
   - `[代码独有]`：文档未提及但代码存在的关键逻辑

> **冲突原则**：文档与代码不一致时，**以代码为准**，但保留"文档说 X，代码实现 Y"的差异说明，给未来更新文档的人留线索。

### 步骤 4：提炼三层知识

按"概念 → 关系 → 逻辑"顺序产出，**不要复制大段原文**，每条都要简短、可被独立引用。

- **概念层**：术语表，只收录系统内反复出现 / 影响接口和数据模型的核心术语
- **关系层**：默认两张 mermaid 图——模块/角色调用图 + 数据/实体关系图，节点 ≤ 12
- **逻辑层**：3–7 条核心流程，每条带触发 / 前置 / 步骤 / 关键分支 / 后置 / 失败模式

具体表格、mermaid、流程描述模板见 [templates.md 第 1–3 节](templates.md)。

### 步骤 5：产出知识包目录

不再作为单文件输出，而是写入一个标准化的"知识包"目录，与 skill-kit 的 `skills/` 风格对齐。

#### 5.1 与用户确认输出位置

用 `AskQuestion` 让用户三选一（默认推荐 A）：

- **A. 项目内**：`<repo>/knowledges/<name>-knowledge/`（团队共享、可入 git）
- **B. 个人**：`~/.cursor/knowledges/<name>-knowledge/`（跨项目、私人）
- **C. skill-kit 仓库**：`<skill-kit>/knowledges/<name>-knowledge/`（集中管理、可发布）

`<name>` 用 kebab-case，目录后缀固定 `-knowledge`（例：`focus-workbench-knowledge`）。下文用 `<output-root>` 指代选定的根目录。

#### 5.2 写入两个文件

```
<output-root>/<name>-knowledge/
├── KNOWLEDGE.md      # YAML frontmatter + 正文 7 节
└── sources.json      # 原始资料索引（schema_version=1，不落正文）
```

- **KNOWLEDGE.md**：frontmatter 字段 `name / system / generated_at / generator / sources[] / code_anchors[] / status`；正文按 7 节顺序「系统定位 → 概念表 → 系统关系 → 核心流程 → 文档vs代码差异 → 未覆盖 → 出处索引」。完整模板见 [templates.md 第 4 节](templates.md)
- **sources.json**：只索引 pageId / title / url / confluence_updated / depth / parent_pageId，**不存正文**。要重读时按 pageId 用 `get_confluence_content` 重抓。schema 见 [templates.md 第 5 节](templates.md)

#### 5.3 upsert 顶层 INDEX.md

更新 `<output-root>/INDEX.md` 表格（按 `name` 唯一 upsert，文件不存在则新建）：

| 知识包 | 系统 | 生成时间 | 来源页数 | 状态 |
| --- | --- | --- | --- | --- |

模板与 upsert 规则见 [templates.md 第 6 节](templates.md)。

#### 5.4 向用户报告

输出三个绝对路径（KNOWLEDGE.md / sources.json / INDEX.md）+ 两种模型加载方式提示（`@` 引用 或 `.cursor/rules/<name>-knowledge.mdc`），完整提示文案见 [templates.md 第 7 节](templates.md)。

---

## 关键规则

- **三层优先级**：概念 > 关系 > 逻辑。先把名词定清楚，再画关系，最后写流程；顺序乱了会越写越糊
- **代码胜于文档**：冲突一律以代码为准，但必须在第 5 节留下差异记录
- **每条知识带出处**：术语、流程、图节点都要可追溯到 `pageId` 或 `file:line`
- **递归收口**：子文档默认 2 层、单次 ≤ 30 篇；继续深挖前先问用户
- **不向上爬**：只抓用户给的入口页 + 其子树；禁止用 ancestors / 父页扩散范围（用户没要求范围就别扩）
- **优先 MCP 工具**：能用 `get_confluence_content` / `get_confluence_content_by_title` 拿 Markdown 就别用浏览器截图；只有列子页和未登录时才需要走浏览器 + REST API
- **原始资料只索引、不落正文**：`sources.json` 仅记录 `pageId / url / confluence_updated`；要重读时按 pageId 重抓，避免本地副本和源不一致
- **图片必看**：Confluence 嵌入图常含架构/时序图，跳过会丢核心信息；用 vision 提取要点后写入摘要
- **避免复述原文**：每个要点压缩到 1–2 句，长内容放出处链让人自己点过去
- **不输出"未来计划"和"已废弃"**：除非用户特别要求；这些极易过时

## 反模式

- ❌ 把 Confluence 正文整段复制到报告里（应提炼，不是搬运）
- ❌ 文档与代码不一致时只信文档（结果是误导未来读者）
- ❌ 一次性递归整个文档空间（爆上下文 + 引入大量无关页面）
- ❌ 跳过 mermaid 直接写一坨自然语言关系（关系图远比文字易读）
- ❌ 输出无出处的"事实陈述"（无法验证 = 不可信）
- ❌ 把这个 skill 当成"读一篇 Confluence 并总结"的轻量任务（那是 `read-confluence` 的活）

## 验收清单

提交报告前自检：

- [ ] 第 1 节用 ≤ 4 句话能让一个新人知道这个系统是干什么的
- [ ] 概念表每个术语都有定义和出处
- [ ] 至少 1 张关系图 + 1 张实体图（或合并为 1 张但说明完整）
- [ ] 至少 3 条核心流程，每条带触发条件和失败分支
- [ ] 第 5 节"文档 vs 代码差异"非空（若真无差异，写"已逐项核对，无差异"）
- [ ] 每条结论可顺着出处索引追回原始 Confluence 或代码位置
- [ ] 第 6 节列出尚未覆盖的部分，便于下一轮迭代
- [ ] KNOWLEDGE.md frontmatter 字段齐全（`name` / `system` / `generated_at` / `sources[]` / `status`）
- [ ] `<output-root>/INDEX.md` 已 upsert 当前知识包行
