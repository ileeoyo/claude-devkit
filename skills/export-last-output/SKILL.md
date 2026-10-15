---
name: export-last-output
description: 将当前 Claude Code 会话中上一条 AI 文本输出导出为 Markdown 文件，并用系统默认 Markdown 程序打开。当用户要求“把刚才的输出导出成 md”“上一条回答存成文件打开”时使用。
disable-model-invocation: true
category: Workflow
tags: [output, export, markdown, session, temp]
---

# 上一条输出导出

把当前会话中最后一条 AI 文本输出抽出来，写成 Markdown 文件，再交给系统默认的 Markdown 程序打开。

## 执行命令

无需参数，用 Bash 执行：

```bash
d="$HOME/.claude/projects/$(pwd -W | sed 's|[:/]|-|g')"; if [ -n "$CLAUDE_CODE_SESSION_ID" ]; then f="$d/$CLAUDE_CODE_SESSION_ID.jsonl"; o="/tmp/llm-last-output-${CLAUDE_CODE_SESSION_ID%%-*}.md"; else f=$(ls -t "$d"/*.jsonl 2>/dev/null | head -1); o="/tmp/llm-last-output.md"; fi; jq -s -r '[.[]|select(.message.role=="assistant")|.message.content[]?|select(.type=="text")|.text]|last' "$f" > "$o" && cmd //c start "" "$(cygpath -w "$o")"
```

## 要点

- 会话文件必须由 `CLAUDE_CODE_SESSION_ID` 拼出，不能用 `ls -t` 取最新修改的文件：同一项目同时开多个窗口时各窗口 sessionId 不同，取最新会导出到别的会话
- 项目目录名由当前工作目录推出，`C:/Work/demo` 对应 `C--Work-demo`
- 导出文件按会话固定命名，反复导出只覆盖同一个文件，不随执行次数累积；`/tmp` 在 Git Bash 下即 `%TEMP%`
- `jq -s … | last` 取的是最后一条完整助手消息，不是文件最后一行
- 用户指定了文件名或目录时，替换 `o` 的取值，其余不变

## 例外

`CLAUDE_CODE_SESSION_ID` 缺失时命令回退到取最新文件，此时多窗口下可能串会话，需向用户说明。
