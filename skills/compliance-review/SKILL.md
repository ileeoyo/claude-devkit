---
name: compliance-review
description: >-
  审查当前 git 改动是否符合 CLAUDE.md 定义的规范、偏好和编码习惯。主 Agent 开启 subagent 进行审查，仅允许在 diff 范围内做最小修正。
---

# Git 改动规范审查

审查当前 git 改动是否符合项目级 `.claude/CLAUDE.md` 和用户级 `~/.claude/CLAUDE.md` 以及其他的相关CLAUDE规则文件 中定义的项目规范、个人偏好和开发编码习惯。

## 流程

主 Agent 使用 `Agent` 工具开启一个 `general-purpose` subagent，由 subagent 自行获取改动并审查：

subagent 需完成以下工作：

1. 获取 git diff：`git status --short && git diff --stat && git diff --cached --stat` 和 `git diff && git diff --cached`
2. 以 CLAUDE.md 规范为依据，逐条检查 diff 是否符合规范

审查范围：unstaged changes + staged changes + 本次任务相关的 untracked 文件。

审查重点：
- 是否违反 CLAUDE.md 明确禁用的写法
- 代码风格、命名、注释是否符合规范
- 是否破坏项目架构分层、模块划分约定
- 敏感业务逻辑（状态、金额、权限、订单、第三方交互等）是否保留了必要注释

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

## 未发现问题
（如无问题，说明改动符合规范）
```
