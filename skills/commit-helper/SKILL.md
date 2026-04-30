---
name: commit-helper
description: 分析已暂存的 Git 改动并生成提交信息。用户说提交、commit、git commit、提交信息、commit message，或使用 /skill-commit-helper 时激活。
---

# Commit Helper

## Instructions

1. 先运行 `git diff --staged` 查看已暂存的改动
2. 按以下格式生成提交信息：`ADD/MOD/DEL/FEAT/...[20260430]: 使用中文描述`
3. 先根据改动目的选择最合适的类型：
   - `ADD`: 新增文件、配置、脚本或初始化内容
   - `MOD`: 修改已有逻辑、文案或配置
   - `DEL`: 删除代码、文件或废弃逻辑
   - `FEAT`: 新功能
   - `FIX`: 缺陷修复
   - `CHORE`: 构建、依赖、工具链等杂项
4. 日期使用当天日期，格式固定为 `YYYYMMDD`；描述必须使用中文
5. **最终输出必须是一条可直接复制执行的 `git commit -m "xxx"` 命令**，单独成一个 bash 代码块，不要附带其他命令（如 `git add`、`git push`）
6. 用户只要求生成提交文案时，仅输出该命令；用户明确要求执行提交时，再实际调用 shell 执行（提交信息含特殊字符时使用 heredoc）

## Examples

最终输出形如：

```bash
git commit -m "FEAT[20260430]: 新增课程列表筛选条件"
```

```bash
git commit -m "FIX[20260430]: 修复登录超时后页面空白问题"
```

```bash
git commit -m "MOD[20260430]: 调整首页卡片文案"
```

## When NOT to use

- 工作区没有 staged 改动时，先提示用户 `git add`
- 用户明确要求 amend 已有提交时，走另一条路径