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
5. 用户只要求生成提交文案时，直接返回提交信息；用户明确要求执行提交时，再使用 heredoc 方式提交

## Examples

- `FEAT[20260430]: 新增课程列表筛选条件`
- `FIX[20260430]: 修复登录超时后页面空白问题`
- `MOD[20260430]: 调整首页卡片文案`

## When NOT to use

- 工作区没有 staged 改动时，先提示用户 `git add`
- 用户明确要求 amend 已有提交时，走另一条路径