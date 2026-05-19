---
name: compliance-review
description: >-
  审查当前 git 改动是否符合 CLAUDE.md 定义的规范、偏好和编码习惯。主 Agent 开启 subagent 进行审查，仅允许在 diff 范围内做最小修正。
---

# Git 改动规范审查

审查当前 git 改动是否符合主 Agent 指定规范文件中定义的项目规范、个人偏好和开发编码习惯。

## 流程

主 Agent 先确认本次审查使用的规范文件路径，再使用 `Agent` 工具开启一个 `general-purpose` subagent 审查。

规范文件路径由主 Agent 明确传给 subagent，不能要求 subagent 自行猜测或自行扩展。通常包括已存在且可读取的项目级 `.claude/CLAUDE.md`、仓库级 `CLAUDE.md`、用户级 `~/.claude/CLAUDE.md`，以及当前会话上下文明确给出的其他 CLAUDE 规则文件。

传给 subagent 的 prompt 保持简略，只包含：审查目标、审查范围、规范文件路径列表、必须执行的审查步骤、输出格式要求。

subagent 必须按顺序完成以下工作：

- 先读取文件改动：`git status --short && git diff --stat && git diff --cached --stat` 和 `git diff && git diff --cached`，并读取本次任务相关的 untracked 文件内容
- 再读取主 Agent 指定的每一个规范文件
- 将规范文件中的规则逐条展开审查，对每条适用规则都判断符合、不符合、不适用或无法判断
- 审查过程必须完整对比，不能只审查主要、重要或简略规范；最终报告只输出问题和结论，不逐条列出全部规范

审查范围：unstaged changes + staged changes + 本次任务相关的 untracked 文件。

审查业务逻辑时需读取实际实现代码，不能仅凭函数名或注释判断。

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
