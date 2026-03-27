# API 文档生成工作流

详细的执行步骤说明，供需要时参考。

---

## 步骤 1：解析参数

分析输入参数，确定接口定位方式：

| 参数格式 | 处理方式 |
|---------|---------|
| 空/`git` | 执行 `git diff --name-only` 找 Controller 变动 |
| `/api/xxx` | 使用 Grep 搜索 `@XxxMapping.*xxx` |
| `XxxController.java` | 直接读取该文件 |
| `XxxController#method` | 读取文件后定位到具体方法 |

---

## 步骤 2：定位接口代码

从 Controller 中提取：

```
类级路径：@RequestMapping("/api/xxx")
方法级路径：@GetMapping("/list")
完整路径：/api/xxx/list
```

---

## 步骤 3：接口清单确认【防遗漏】

读取完 Controller 后，**必须先完成以下核对再进入后续步骤**：

1. 列出 Controller 中所有带 `@XxxMapping` 注解的方法（方法名 + 路径）
2. 统计接口总数
3. 将此清单作为后续生成文档的基准，确保每个方法都有对应的文档章节

```
示例清单：
Controller: OpenApiServiceController (共 14 个接口)
  - list()        → POST /admin/pc/open-api-service/list
  - export()      → POST /admin/pc/open-api-service/export
  - add()         → POST /admin/pc/open-api-service/add
  ...（列出全部）
```

---

## 步骤 4：分析接口定义

| 提取项 | 来源 |
|-------|------|
| 请求方法 | `@GetMapping`/`@PostMapping` 等 |
| 路径参数 | `@PathVariable` |
| 查询参数 | `@RequestParam` + 无注解 POJO |
| 请求头 | `@RequestHeader` |
| 请求体 | `@RequestBody` |
| 文件上传 | `MultipartFile` / `@RequestPart` |
| 响应类型 | 方法返回类型 |

---

## 步骤 5：确定请求信息

### GET/DELETE/HEAD 请求

```markdown
| 项目 | 值 |
|------|-----|
| 响应 Content-Type | `application/json` |
```

### POST/PUT/PATCH 请求

```markdown
| 项目 | 值 |
|------|-----|
| 请求 Content-Type | `application/json` |
| 响应 Content-Type | `application/json` |
```

请求 Content-Type 判断优先级：
1. consumes 属性指定 → 按指定值
2. MultipartFile/@RequestPart → multipart/form-data
3. @RequestBody → application/json
4. POJO 参数 → application/x-www-form-urlencoded

---

## 步骤 6：追溯关联类型

递归读取：
- 请求参数类（Req/DTO）
- 响应结果类（Res/VO）
- 嵌套对象
- 枚举类

**枚举提取强制规则：**
1. 读取完整枚举类代码
2. 提取所有枚举常量，一个不能少
3. 从代码逻辑推断含义，代码优先于注释
4. 无注释时根据枚举名称和命名模式推断

---

## 步骤 7：深入分析 Service 层【关键】

**必须读取 ServiceImpl 代码**，提取：

### 权限校验
```java
// 代码模式
if (!settlement.getInitiatorType().match(SettlementInitiatorType.AGENT)) {
    throw new ServiceException("仅允许删除经纪人发起的结算单");
}
// → 文档：仅允许删除经纪人发起的结算单
```

### 状态校验
```java
// 代码模式
if (!order.getStatus().match(OrderStatus.PENDING)) {
    throw new ServiceException("仅待处理订单可取消");
}
// → 文档：仅状态为「待处理」的订单可取消
```

### 隐式必填
```java
// 字段无 @NotNull，但代码中校验
Assert.isNotEmpty("企业ID不能为空", req.getCustomerId());
// → 文档：customerId 标记为「必填」
```

### 条件必填
```java
// 条件校验
if (req.getConfirm() && StringUtils.isBlank(req.getBankCardNo())) {
    throw new ServiceException("确认收款时银行卡号必填");
}
// → 文档：bankCardNo 标记为「条件必填」，说明条件
```

---

## 步骤 8：提取字段信息

每个字段提取：

| 信息 | 来源 |
|-----|------|
| 字段名 | Java 字段名（考虑 @JsonProperty） |
| 类型 | Java 类型转换（**时间类型必须转为 String 并标注格式**，见 RULES.md 类型转换规范） |
| 必填 | 注解 + 代码逻辑综合判断 |
| 说明 | Javadoc / @ApiModelProperty |
| 约束 | @Size/@Min/@Max 等 |

---

## 对象定义原则

所有请求对象和响应对象一律内联定义在各自接口中，即使多个接口使用完全相同的对象，也各自独立定义，方便接口对接人员直接查看，无需来回跳转。只有枚举统一放入「枚举定义汇总」章节。

---

## 步骤 9：汇总枚举

生成文档前，收集所有接口涉及的枚举类，统一放入「枚举定义汇总」章节。

---

## 步骤 10：生成文档

按 references/TEMPLATE.md 模板生成。

---

## 步骤 11：验证检查

生成后确认：

```
□ 文档接口数量与 Controller 方法数量一致（对比步骤 3 清单，无遗漏、无多余）
□ 所有 Service 层校验已转化为业务规则
□ 必填性反映真实代码行为（代码优先于注解）
□ 所有枚举常量都已提取（对比源代码数量）
□ 枚举描述准确（无注释时已从代码推断）
□ 每个接口的请求/响应对象均内联定义在该接口中（无公共对象引用）
□ 嵌套对象都有定义
□ 锚点链接正确
□ JSON 示例包含对应对象定义的所有字段（无省略）
```
