# skill-kit

存放可复用的 [Cursor Skills](https://docs.cursor.com/) 的仓库。每个 skill 是一个独立目录，内含一个 `SKILL.md`，通过 YAML frontmatter 中的 `name` 和 `description` 被 Cursor 自动发现并按触发词激活。

## 目录结构

```
skill-kit/
├── README.md
├── skills/
│   ├── commit-helper/
│   │   └── SKILL.md
│   ├── extract-system-knowledge/
│   │   ├── SKILL.md
│   │   └── templates.md
│   └── gen-skill/
│       └── SKILL.md
└── knowledges/                      # 由 extract-system-knowledge 按需生成（可选）
    ├── INDEX.md
    └── <name>-knowledge/
        ├── KNOWLEDGE.md
        └── sources.json
```

## 现有 Skills

| Skill | 作用 | 典型触发词 |
| --- | --- | --- |
| [`commit-helper`](skills/commit-helper/SKILL.md) | 分析已暂存的 Git 改动，生成符合规范（如 `FEAT[YYYYMMDD]: ...`）的中文提交信息 | 提交、commit、git commit、`/skill-commit-helper` |
| [`extract-system-knowledge`](skills/extract-system-knowledge/SKILL.md) | 从 Confluence 文档树（含子文档）和项目代码提炼"概念 / 关系 / 逻辑"三层系统知识，文档与代码冲突时以代码为准；产出包写入 `knowledges/<name>-knowledge/` | 提取系统知识、梳理系统、建立项目认知、`/extract-system-knowledge` |
| [`gen-skill`](skills/gen-skill/SKILL.md) | 从最近一次对话中提炼可复用的工作流程，并生成新的 `SKILL.md` | 生成 skill、提取 skill、`/gen-skill` |

## 使用方式

### 1. 安装到个人 skills 目录（全局生效）

将本仓库中的某个 skill 链接或复制到 `~/.cursor/skills/`：

```bash
# 软链（推荐，便于跟随仓库更新）
ln -s "$(pwd)/skills/commit-helper" ~/.cursor/skills/commit-helper

# 或直接复制
cp -R skills/commit-helper ~/.cursor/skills/
```

### 2. 安装到项目 skills 目录（仅当前项目生效）

在目标项目根目录下：

```bash
mkdir -p .cursor/skills
cp -R /path/to/skill-kit/skills/commit-helper .cursor/skills/
```

安装后在 Cursor 中触发对应关键词即可激活，无需重启。

## 新增 Skill

推荐直接使用 `gen-skill`：在完成一段值得沉淀的工作流后，对 Cursor 说 “生成 skill” 或 `/gen-skill`，它会引导你产出符合规范的 `SKILL.md`。

手写时遵循以下约定：

- 目录命名：`skills/<kebab-case-name>/`
- `SKILL.md` 必须包含 YAML frontmatter：
  ```yaml
  ---
  name: <kebab-case-name>
  description: <第三人称、一句话说明 WHAT + WHEN，包含明显触发词>
  ---
  ```
- 正文 ≤ 200 行；超出部分用 `reference.md` / `examples.md` 拆分
- 使用第三人称祈使句，避免“我会 / 你可以”
- 不写时效性内容、不放一次性的项目细节

## Knowledges

由 [`extract-system-knowledge`](skills/extract-system-knowledge/SKILL.md) 产出的"系统知识包"约定如下：

- 一个知识包对应一个目录 `knowledges/<name>-knowledge/`，`<name>` 用 kebab-case
- 目录内固定两个文件：
  - `KNOWLEDGE.md`：YAML frontmatter（`name` / `system` / `description` / `generated_at` / `code_anchors[]` / `status`）+ 7 节正文（系统定位 / 概念表 / 关系 / 流程 / 文档vs代码差异 / 未覆盖 / 出处索引）
  - `sources.json`：原始资料**索引**（`pageId` / `title` / `url` / `confluence_updated` / `depth` / `parent_pageId`），**不落正文**——重读时按 pageId 用 `get_confluence_content` 重抓
- 顶层 `knowledges/INDEX.md` 由 skill 自动 upsert，记录该 root 下所有知识包

输出位置在 skill 运行时由用户三选一：

- **项目内** `<repo>/knowledges/`（推荐，团队共享）
- **个人** `~/.cursor/knowledges/`（跨项目、私人）
- **skill-kit 仓库** `<skill-kit>/knowledges/`（集中管理、可发布）

让模型用上知识包的两种方式：

- 临时引用：`@knowledges/<name>-knowledge/KNOWLEDGE.md`
- 自动注入：在 `<repo>/.cursor/rules/` 下加一个 `<name>-knowledge.mdc`（`alwaysApply: true`）指向该 KNOWLEDGE.md

## 维护

- 提交信息可使用 `commit-helper` 自动生成
- 新增/修改 skill 后，更新本 README 的「现有 Skills」表格
