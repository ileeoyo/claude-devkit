# API 文档规则参考

详细规则供需要时查阅。

---

## 目录

- [参数过滤规则](#参数过滤规则)
- [必填判断规则](#必填判断规则)
- [字段约束提取](#字段约束提取)
- [模块显示规则](#模块显示规则)
- [术语规范](#术语规范)
- [枚举处理](#枚举处理)
- [分页规范](#分页规范)
- [特殊场景](#特殊场景)
- [锚点规范](#锚点规范)
- [Java 类型转换规范](#java-类型转换规范)

---

## 参数过滤规则

### 需过滤的参数类型

以下类型**不应出现在文档中**：

- `HttpServletRequest`/`HttpServletResponse`
- `HttpSession`
- `Principal`/`Authentication`
- `BindingResult`/`Errors`
- `Model`/`ModelMap`/`ModelAndView`
- `RedirectAttributes`
- `InputStream`/`OutputStream`/`Reader`/`Writer`
- `@CurrentUser`/`@LoginUser` 等自定义注解参数

### 需忽略的字段

- `@JsonIgnore`
- `@JsonProperty(access = READ_ONLY)`（请求中忽略）
- `@JsonProperty(access = WRITE_ONLY)`（响应中忽略）
- `static`/`transient` 字段
- `serialVersionUID`

### 被覆盖的参数

Controller 方法内被赋值的参数不应出现在文档中：

```java
public R<?> method(@RequestBody Req req) {
    req.setUserId(acl.getUserId());  // userId 不应在文档中
}
```

---

## 必填判断规则

**核心原则：Service代码逻辑 > Controller代码校验 > 注解声明**

必填性判断优先级：
1. Service层代码中的实际校验（`Assert.isNotEmpty()`, `if (x == null) throw`）
2. Controller代码中的throw逻辑
3. 注解声明（`@NotNull`, `@NotBlank`等）
4. 注解与代码冲突时，以代码为准

### @RequestParam

| 情况 | 必填 |
|------|------|
| `@RequestParam Long id` | **是** |
| `@RequestParam(required = false)` | 否 |
| `@RequestParam(defaultValue = "1")` | 否 |
| `@RequestParam Optional<Long>` | 否 |

### @PathVariable

| 情况 | 必填 |
|------|------|
| `@PathVariable Long id` | **是** |
| `@PathVariable(required = false)` | 否 |

### POJO 字段

**前提**：Controller 需有 `@Valid` 或 `@Validated` 注解。

| 注解 | 必填 |
|------|------|
| `@NotNull`/`@NotBlank`/`@NotEmpty` | **是** |
| 基本类型 `int`/`long` | **是** |
| 包装类型无校验注解 | 否 |

### 分组校验

- `@Validated(CreateGroup.class)` → 仅该分组注解生效
- `@Validated` 无分组 → Default.class 注解生效

---

## 字段约束提取

| 注解 | 文档输出 |
|------|---------|
| `@Size(min=1, max=50)` | 长度: 1-50 |
| `@Min(0) @Max(100)` | 范围: 0-100 |
| `@Email` | 格式: 邮箱 |
| `@Pattern(regexp="^1[3-9]\\d{9}$")` | 格式: 手机号 |

---

## 模块显示规则

**核心原则**：无内容则不显示标题。

| 模块 | 显示条件 |
|------|---------|
| 公共请求头 | 项目有公共请求头时 |
| 请求头 | 有 @RequestHeader 时 |
| 路径参数 | 有 @PathVariable 时 |
| 查询参数 | 有 @RequestParam 或 POJO 时 |
| 请求体 | Body 类型非 none 时 |
| 响应结构 | 始终存在 |

---

## 术语规范

| 标准术语 | 禁止使用 |
|---------|----------|
| 请求体 | Body 参数、请求数据 |
| 响应结构 | 返回结果、返回值 |
| 查询参数 | URL 参数、Query 参数 |
| 路径参数 | Path 参数 |
| 必填 | 必传、required |
| 枚举值 | 可选值、取值范围 |

---

## 枚举处理

### 提取规则

**强制要求：必须提取所有枚举常量，一个不能少。**

描述获取规则：

1. 读取完整枚举类代码，提取所有枚举常量（包括无注释的）
2. 优先从 `@EasyExcelEnumConvertDescription` 标注字段获取描述
3. 其次从 Javadoc 注释获取描述
4. 无注释时，根据枚举名称、代码逻辑、命名模式推断含义
5. 注释与代码逻辑冲突时，以代码逻辑为准

**代码逻辑优先于注释。**

### 引用格式

```markdown
| status | String | 是 | 状态 | 见 [StatusEnum 枚举](#statusenum-枚举) |
```

### 传值说明

前端传递枚举常量名（区分大小写）：
- 正确: `?status=ENABLE`
- 错误: `?status=enable`

---

## 分页规范

### 请求参数

| 参数名 | 类型 | 必填 | 默认值 |
|--------|------|------|--------|
| pageNum | Integer | 否 | 1 |
| pageSize | Integer | 否 | 10 |
| orderBy | String | 否 | - |

### 响应结构

```
data (PageInfo):
├── total: Long - 总记录数
├── list: Array<T> - 数据列表
├── pageNum: Integer
├── pageSize: Integer
└── pages: Integer - 总页数
```

---

## 特殊场景

### 文件上传

- 单文件：`file | File | 是`
- 多文件：`files | Array<File> | 是`
- 混合（@RequestPart）：标注 `multipart/form-data`

### 文件下载

```markdown
返回二进制文件流。

| 响应头 | 说明 |
|--------|------|
| Content-Type | 文件 MIME 类型 |
| Content-Disposition | attachment; filename="文件名" |
```

### SSE 流式响应

```markdown
返回 SSE 流式数据。
Content-Type: text/event-stream

事件格式：data: {"field": "value"}
```

---

## 锚点规范

| 类型 | 格式 | 示例 |
|------|------|------|
| 接口 | `#编号-接口名称` | `#11-创建订单` |
| 子模块 | `#三级编号-名称` | `#111-请求信息` |
| 内联对象 | `#对象名-对象` | `#orderitem-对象` |
| 枚举 | `#枚举名-枚举` | `#orderstatus-枚举` |

---

## Java 类型转换规范

文档面向前端开发，字段类型必须转换为前端友好的类型。

### 时间类型转换（强制）

| Java 类型 | 文档类型 | 说明列追加格式标注 |
|-----------|---------|------------------|
| `LocalDateTime` | `String` | 追加 `格式: yyyy-MM-dd HH:mm:ss` |
| `LocalDate` | `String` | 追加 `格式: yyyy-MM-dd` |
| `LocalTime` | `String` | 追加 `格式: HH:mm:ss` |
| `Date` | `String` | 根据项目配置标注格式 |

格式推断优先级：
1. 字段上的 `@JsonFormat(pattern = "...")` → 使用指定格式
2. 项目全局 Jackson 配置 → 使用全局格式
3. 无法确定时 → 使用上表中的默认格式

**示例**：

```markdown
| createTime | String | 创建时间，格式: yyyy-MM-dd HH:mm:ss | - |
| establishTime | String | 成立时间，格式: yyyy-MM-dd | - |
```

### 其他类型保持不变

`Long`、`Integer`、`String`、`Boolean`、`BigDecimal`、`Array<T>` 等类型直接使用，无需转换。
