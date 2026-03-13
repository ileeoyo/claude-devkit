# claude-devkit

个人开发效率工具集，作为 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 插件使用，提供 API 文档生成、Git 改动分析、代码重构、提示词优化等技能。

## 安装

首先添加插件市场，然后安装插件：

```bash
/plugin marketplace add ileeoyo/claude-devkit
/plugin install claude-devkit@claude-devkit
```

安装完成后，使用 `/reload-plugins` 重新加载插件即可使用。

## 技能列表

### `/api-doc` - API 文档生成

根据 Java Controller 接口自动生成面向前端开发的 API 对接文档。深入分析 Service 实现层代码，确保文档反映接口的真实运行行为。

**使用方式：**

```bash
/api-doc                          # 基于 git diff 中的 Controller 变动自动生成
/api-doc /api/user/list           # 指定接口路径
/api-doc UserController.java      # 指定 Controller 文件
/api-doc UserController#list      # 定位到具体方法
```

### `/git-changes` - Git 改动分析

读取当前 Git 项目的未提交代码、暂存区变更等改动信息，支持根据参数智能执行后续操作。

**使用方式：**

```bash
/git-changes                      # 展示改动摘要
/git-changes 生成commit            # 分析改动并生成 commit message
/git-changes 代码审查              # 对改动进行 code review
/git-changes staged               # 仅查看已暂存的改动
```

### `/refactor` - 代码重构

分析代码坏味道，制定重构方案并执行优化。支持识别重复代码、过长方法、过大类、命名不清晰等常见问题，应用提取方法、提取类、策略模式等重构手法。

**使用方式：**

```bash
/refactor OrderService.java       # 重构指定文件
/refactor src/utils/              # 重构指定目录
```

### `/refine-prompt` - 提示词优化

将口语化、模糊或描述不清的提示词整理为结构化、表达清晰的版本，经用户确认后再执行任务，避免因理解偏差导致返工。

**使用方式：**

```bash
/refine-prompt 把那个用户列表的接口改一下，加个按时间筛选的功能，还有翻页好像有点问题也看看
```

## 许可证

MIT
