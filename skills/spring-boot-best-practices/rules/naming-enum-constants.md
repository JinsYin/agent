---
title: 枚举与常量命名
impact: MEDIUM
tags: naming, enum, constants, jackson
---

## 枚举与常量命名

- 常量：`UPPER_SNAKE_CASE`，放在 `{Xxx}Constants` final 类里
- 枚举类型：`PascalCase`；枚举值：`UPPER_SNAKE_CASE`

枚举若需序列化成业务字符串，用 `@JsonValue` + `@JsonCreator` 显式控制，不要依赖 `name()`——那会让 Java 侧改名直接破坏 API 契约。

```java
@Getter
@RequiredArgsConstructor
public enum OrderStatus {
    PENDING("pending", "待支付"),
    PAID("paid", "已支付"),
    CANCELLED("cancelled", "已取消");

    private final String value;
    private final String label;

    @JsonValue
    public String getValue() { return value; }
}
```
