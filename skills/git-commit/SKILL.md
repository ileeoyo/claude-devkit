---
name: git-commit
description: 分析代码改动并生成规范的 commit message
category: Git
tags: [git, commit, version-control]
---

**Guardrails**
- 如果项目有指定的 commit 规范，遵循项目规范；否则使用通用格式
- commit message 格式：`类型: 描述`
- 禁止添加 `Co-Authored-By` 或 `Generated with Claude Code` 后缀
- 不要自动 push 到远程仓库，除非用户明确要求
- 不要使用 `git add -A` 或 `git add .`，应明确指定要提交的文件
- 如果有敏感文件（.env、credentials 等），警告用户不要提交

**Steps**
1. 运行 `git status` 查看当前工作区状态
2. 运行 `git diff` 和 `git diff --staged` 查看具体改动内容
3. 分析改动，确定：
   - **类型**：feat（新功能）、fix（修复）、refactor（重构）、docs（文档）、style（格式）、test（测试）、chore（杂项）
   - **描述**：简洁描述改动内容（中文）
4. 生成 commit message，格式如下：

   ```
   类型: 总结性描述

   - 改动点1
   - 改动点2
   - 改动点3
   ...
   ```

   **分项描述规则**：
   - 总结性描述应简洁精炼，一行内完成
   - 分项描述主要的改动点，最多不超过 **5 个**，避免大改动时分项过多
   - 每个分项应简洁明了，说明"做了什么"而非"怎么做的"
   - 分项之间避免重复，聚焦不同方面的改动
5. 生成 commit message 并询问用户确认
6. 执行 `git add <files>` 添加指定文件
7. 执行 `git commit -m "message"` 提交

**Commit 类型说明**

| 类型 | 说明 | 示例 |
|-----|------|------|
| feat | 新增功能 | `feat: 新增用户注册功能` |
| fix | 修复 bug | `fix: 修复登录验证重复问题` |
| refactor | 重构代码 | `refactor: 重构工具类方法` |
| docs | 文档变更 | `docs: 更新部署说明` |
| style | 代码格式 | `style: 格式化代码` |
| test | 测试相关 | `test: 新增用户服务单元测试` |
| chore | 构建/工具 | `chore: 升级依赖版本` |

**Reference**
- 运行 `git log --oneline -10` 查看最近的提交风格
