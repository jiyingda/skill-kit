# Extract System Knowledge：模板集

本文件配套 [SKILL.md](SKILL.md) 使用，提供"概念表 / 关系图 / 流程描述 / 知识包正文 / sources.json / INDEX.md"五种模板的完整范式。SKILL.md 只保留指引，模板细节都放在这里。

---

## 1. 概念表（术语层）

只收录"系统内反复出现 / 跨多篇文档 / 影响接口和数据模型"的核心术语。一次性的描述性词汇不收。

```markdown
| 术语 | 定义（≤ 30 字） | 别名 / 缩写 | 出处 |
| --- | --- | --- | --- |
| `<术语>` | `<定义>` | `<别名>` | `Confluence:pageId` 或 `path/to/file.ts:L12` |
```

## 2. 关系图（关系层）

默认产出两张 mermaid 图：

- **角色 / 模块图**：谁调用谁、谁依赖谁
- **数据 / 实体图**：核心实体之间的关联（1:N、N:N）

```mermaid
graph LR
  User -->|登录| AuthService
  AuthService -->|签发 token| Gateway
  Gateway -->|鉴权后转发| BizService
```

节点不超过 12 个；超出说明系统范围划得过大，回到 SKILL.md 的步骤 1 收窄。

## 3. 流程描述（逻辑层）

每条核心流程用"步骤清单 + 关键分支"描述，必要时配 mermaid 时序图。模板：

```
流程：<流程名，例：手机号验证码登录>
触发：<谁、在什么场景触发>
前置：<已登录态、配置项、权限>
步骤：
  1. <动作>（<出处>）
  2. <动作>（<出处>）
  3. <动作 + 关键分支：成功 / 失败 / 限流 / 验证码错>
后置：<状态变化、副作用、异步任务>
失败模式：<已知的容易出错点>
```

每个系统通常 3–7 条核心流程；先列出列表再展开，避免一次性陷入细节。

---

## 4. KNOWLEDGE.md 正文模板

写入 `<output-root>/<name>-knowledge/KNOWLEDGE.md`，frontmatter 在前，正文 7 节固定结构在后。

### 4.1 frontmatter

```yaml
---
name: focus-workbench
system: Focus 工作台
description: Focus 工作台是面向辅导老师的带班工作台，覆盖学生档案、沟通记录、任务系统、学习数据查看与续报等场景
generated_at: 2026-05-02
generator: extract-system-knowledge@v1
code_anchors:
  - path: backend/api/focus
    note: focus 列表页接口
status: draft   # draft | reviewed | stale
---
```

字段说明：

- `name`：kebab-case，与目录名前缀一致
- `system`：人类可读的系统名
- `description`：一句话说明这个系统是什么（≤ 80 字），让模型/用户不打开正文也能判断是否相关
- `generated_at`：本次生成日期（YYYY-MM-DD）
- `generator`：固定 `extract-system-knowledge@v1`，方便未来工具识别
- `code_anchors[]`：相关代码路径与说明（无项目代码时为空数组 `[]`）
- `status`：`draft`（首次生成）/ `reviewed`（人工核对过）/ `stale`（已知过期，待刷新）

> 抓过的页面清单只放在同目录下的 `sources.json`（见第 5 节），不再在 frontmatter 里重复，避免双份维护。

### 4.2 正文 7 节

```markdown
# <系统名> 知识包（YYYY-MM-DD）

## 1. 系统定位
<2–4 句：这个系统解决什么问题、对谁负责、在大图里处于什么位置>

## 2. 概念表
<术语表，模板见本文件第 1 节>

## 3. 系统关系
<模块/角色 mermaid 图 + 实体 mermaid 图，每张图后写 1–2 句解读>

## 4. 核心流程
### 4.1 <流程名>
<流程模板，见本文件第 3 节>
### 4.2 ...

## 5. 文档 vs 代码差异
| 主题 | 文档说法 | 代码实现 | 建议 |
| --- | --- | --- | --- |

## 6. 未覆盖与后续问题
- <还没读完的子文档 / 还没验证的代码模块 / 需要找谁确认的开放问题>

## 7. 出处索引
- Confluence: pageId=... <title>
- Code: <file:line>
```

---

## 5. sources.json schema

写入 `<output-root>/<name>-knowledge/sources.json`。**只索引、不存正文**，要重读时用 `get_confluence_content` 按 pageId 重抓。

```json
{
  "schema_version": 1,
  "system": "Focus 工作台",
  "captured_at": "2026-05-02T17:21:00+08:00",
  "captured_via": ["get_confluence_content", "get_confluence_content_by_title", "rest:/child/page"],
  "pages": [
    {
      "pageId": "827675663",
      "title": "Focus 工作台",
      "url": "https://confluence.zhenguanyu.com/pages/viewpage.action?pageId=827675663",
      "confluence_updated": "2025-03-13T15:14:48+08:00",
      "depth": 0,
      "parent_pageId": null,
      "note": "入口页，正文为空（目录型父页）"
    }
  ]
}
```

字段说明：

- `schema_version`：固定 `1`
- `system` / `captured_at`：与 KNOWLEDGE.md frontmatter 保持一致
- `captured_via[]`：实际用到的抓取工具/端点，方便排查（如 `get_confluence_content` / `get_confluence_content_by_title` / `rest:/child/page`）
- `pages[].depth`：相对入口页的递归深度，入口页 `0`，直接子页 `1`
- `pages[].parent_pageId`：入口页为 `null`，其余指向父 pageId
- `pages[].note`（可选）：自由文本，标注特殊情况，常见值：
  - `"入口页，正文为空（目录型父页）"` —— 该页只用作子页索引，不参与概念/流程提取
  - `"由 get_confluence_content_by_title 抓取"` —— URL 是 `/display/<SpaceKey>/<Title>` 形式
  - `"权限受限，部分内容缺失"` —— 抓到了但内容不完整

**多入口模式**：用户给一组兄弟 URL（不是单根递归）时，每个入口页都是 `depth: 0`、`parent_pageId: null`；递归出来的子页 `depth >= 1`、`parent_pageId` 指向其入口或上一层。

## 6. INDEX.md 模板

`<output-root>/INDEX.md` 列出该 root 下的所有知识包，每次生成 / 更新时由 skill upsert 对应行（按 `name` 唯一）。

```markdown
# Knowledges Index

| 知识包 | 系统 | 生成时间 | 来源页数 | 状态 |
| --- | --- | --- | --- | --- |
| [focus-workbench](focus-workbench-knowledge/KNOWLEDGE.md) | Focus 工作台 | 2026-05-02 | 5 | draft |
```

upsert 规则：

- 文件不存在 → 创建并写入表头
- 已存在同名行 → 替换
- 不存在同名行 → 追加到末尾

---

## 7. 让模型用上知识包

生成完毕后，在聊天中告诉用户两种加载方式（任选其一）：

**临时引用**：

```
@knowledges/<name>-knowledge/KNOWLEDGE.md 帮我...
```

**项目内自动注入**（推荐给团队共享场景）：在 `<repo>/.cursor/rules/` 下新建 `<name>-knowledge.mdc`：

```mdc
---
description: <系统名> 知识包，理解该系统时自动加载
globs:
alwaysApply: true
---

请在涉及"<系统名>"的对话中参考 `knowledges/<name>-knowledge/KNOWLEDGE.md` 的内容。
```
