---
name: compliance-review
description: >-
  审查当前 git 改动是否符合 CLAUDE.md 定义的规范、偏好和编码习惯。主 Agent 开启 subagent 进行审查，仅允许在 diff 范围内做最小修正。
---

# Git 改动规范审查

审查当前 git 改动是否符合主 Agent 指定规范文件中定义的项目规范、个人偏好和开发编码习惯。

## 流程

主 Agent 直接使用 `Agent` 工具开启一个 `general-purpose` subagent 审查，**不自行查询 git diff 或文件内容**，所有审查工作由 subagent 完成。

规范文件路径由主 Agent 明确传给 subagent，通常包括项目级 `.claude/CLAUDE.md`、仓库级 `CLAUDE.md`、用户级 `~/.claude/CLAUDE.md`，以及当前会话上下文明确给出的其他 CLAUDE 规则文件。subagent 不得自行猜测或扩展规范文件路径。

传给 subagent 的 prompt 包含：审查目标、审查范围、规范文件路径列表、必须执行的审查步骤、输出格式要求。

subagent 必须按顺序完成：

- 读取改动：`git status --short && git diff --stat && git diff --cached --stat` 和 `git diff && git diff --cached`，并读取相关 untracked 文件
- 读取主 Agent 指定的每一个规范文件
- 逐条展开审查，对每条适用规则判断符合、不符合、不适用或无法判断；只输出问题和结论，不逐条列出全部规范
- 审查业务逻辑时需读取实际实现代码，不能仅凭函数名或注释判断

审查范围：unstaged changes + staged changes + 本次任务相关的 untracked 文件。

### 执行修复

subagent 审查完成后，主 Agent 根据审查结果修复违规项，遵循最小修复原则：

- 只改违规的新增/变更行
- 保持原文件格式风格
- 不改与 diff 无关的旧代码、不新增抽象

禁止：格式化未改动代码、重构、修改历史问题、git 版本控制操作。

### 输出报告

```markdown
老板，规范审查完成。

## 已修正
- `路径:行号`：修正内容与依据

## 未修改但需注意
- `路径:行号`：原因

## 规范审查结论
（概述整体是否符合规范；不逐条列出全部规范）

## 未发现问题
（如无问题，说明改动符合规范）
```
