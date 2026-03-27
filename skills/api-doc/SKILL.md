---
name: api-doc
description: 根据 Java Controller 接口生成前端 API 对接文档。当用户要求生成 API 文档、接口文档，或提到前端对接、接口说明时使用。支持基于 git 改动、接口路径、文件路径等多种方式定位接口。
---

# 前端 API 对接文档生成器

根据 Java Controller 接口生成面向前端开发的 API 对接文档。

---

## 文件路径说明

**当前 Skill 目录**：`${CLAUDE_SKILL_DIR}`

**目录结构**：
```
${CLAUDE_SKILL_DIR}/
├── SKILL.md (本文件)
└── references/
    ├── WORKFLOW.md
    ├── RULES.md
    └── TEMPLATE.md
```

---

## ⚠️ 核心原则（强制遵守）

**代码绝对优先原则：**

1. **代码 > 注释** - 代码实际行为优先于任何文字注释
2. **代码 > 注解** - 代码逻辑优先于注解声明（如@RequestParam(required = false)）
3. **代码 > 命名** - 代码实际行为优先于方法名、类名、参数名、包名
4. **Service实现 > Controller注解** - Service层的真实逻辑优先于Controller的表面注解

**执行要求：**
- 必须读取ServiceImpl代码，分析实际业务逻辑
- 注解与代码逻辑冲突时，以代码逻辑为准
- 所有枚举必须全部提取，从代码推断含义
- 文档必须反映接口的真实运行行为

---

## 开始前必读

**立即读取以下两个文档** - 这两个文档贯穿整个执行过程：

```
□ 已读取 references/WORKFLOW.md（必须！包含完整的11步执行流程）
□ 已读取 references/RULES.md（必须！包含必填性判断、业务规则提取规则）
```

**注意：** references/TEMPLATE.md在步骤10生成文档时才读取

---

## 快速开始

### 参数格式

| 参数 | 说明 | 示例 |
|------|------|------|
| 空/`git` | 基于 git diff 中的 Controller 变动 | `/api-doc` |
| 接口路径 | 搜索匹配该路径的 Controller | `/api-doc /api/user/list` |
| 文件路径 | 读取整个 Controller 文件 | `/api-doc UserController.java` |
| `类名#方法名` | 定位到具体方法 | `/api-doc UserController#list` |

**输出路径**：默认 `tmp/{模块名称}-api.md`，可在参数中指定 `.md` 结尾的路径。

---

## 执行流程

**流程概览：** 定位代码 → 分析提取 → 生成文档 → 验证

**详细执行步骤见 references/WORKFLOW.md**，整体分为以下阶段：

| 阶段 | 步骤 | 关键动作 |
|------|------|---------|
| 定位 | 1-3 | 解析参数 → 定位 Controller → 确认接口清单（防遗漏） |
| 分析 | 4-8 | 分析接口定义 → 确定请求信息 → 追溯类型 → **深入 Service 层** → 提取字段 |
| 汇总 | 9 | 汇总所有枚举，收集到枚举定义汇总 |
| 生成 | 10 | **此时读取 references/TEMPLATE.md**，按模板生成文档并保存 |
| 验证 | 11 | 对照检查清单验证文档完整性和准确性 |

**⚠️ 分析阶段（步骤4-8）严格遵循代码优先原则，参考 references/RULES.md**

---

## 关键规则速查

### 代码优先判断原则

**任何时候都遵循：代码实际行为 > 文字描述 > 注解 > 命名**

- 必填性：Service代码校验 > 代码throw校验 > 注解校验 > 无校验
- 枚举描述：代码逻辑 > Javadoc注释 > 注解描述 > 枚举名推断
- 业务规则：Service实现代码 > Controller注释 > 参数命名
- 字段含义：代码使用方式 > Javadoc注释 > 字段名

📖 **完整规则**: 见 references/RULES.md

### 业务规则提取模式

从ServiceImpl代码中识别（不是从注释）：
- `hasPermission()` → 权限范围说明
- `status.match()` → 状态限制
- `initiatorType.match()` → 角色限制
- `Assert.isTrue()` → 业务约束

📖 **更多模式**: 见 references/RULES.md

---

## 禁止事项

- ❌ 仅依赖注解/注释生成文档，不读取实际代码逻辑
- ❌ 遗漏任何枚举常量（必须全部提取）
- ❌ 未读取 Service 实现代码就生成文档
- ❌ 注解与代码逻辑冲突时采用注解（必须采用代码逻辑）
- ❌ 前端调用示例代码（JS/axios 等）
- ❌ curl 示例
- ❌ 后端实现细节（缓存、MQ 等）
- ❌ 提前读取TEMPLATE.md（必须在步骤10才读取）

---

现在开始执行任务。
