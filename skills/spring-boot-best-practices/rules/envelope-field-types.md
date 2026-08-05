---
title: 字段类型约定
impact: HIGH
tags: envelope, response, jackson, types, money
---

## 字段类型约定

| 用途 | 用 | 不用 |
|---|---|---|
| 日期时间 | `LocalDate` / `LocalDateTime` / `OffsetDateTime` | `java.util.Date` |
| 金额 | `BigDecimal`，标度固定 2 位 | `double` / `float` |
| 主键与 ID | `Long`（雪花）**或** `String`（UUID） | 同一实体上两者混用 |
| 枚举 | `@JsonValue` 显式指定序列化值 | 依赖 `name()` |

三条理由：

- `Date` 同时携带日期与时区语义且可变，跨时区序列化行为不确定；`LocalDateTime` 由 commons 里的 Jackson 序列化器统一格式化为 `yyyy-MM-dd HH:mm:ss`。
- 浮点数无法精确表示十进制小数，金额累加必然产生偏差——这类 bug 在对账时才暴露，且难以追溯。
- 同一实体上 ID 类型混用（有的字段 `Long`、有的 `String`）会让调用方无法统一处理，序列化后前端还可能因 JS 数字精度截断长整型。

**错误：**

```java
private Date createdAt;      // ❌
private double amount;       // ❌ 精度丢失
private Long id;
private String parentId;     // ❌ 与 id 类型不一致
```

**正确：**

```java
private LocalDateTime createdAt;
private BigDecimal amount;   // 标度 2
private Long id;
private Long parentId;
```
