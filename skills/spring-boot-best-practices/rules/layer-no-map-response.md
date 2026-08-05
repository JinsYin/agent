---
title: 不用 Map 当响应类型
impact: HIGH
tags: layer, controller, response, openapi
---

## 不用 Map 当响应类型

`Map<String, Object>` 没有编译期约束，也生成不出 OpenAPI schema，调用方只能靠猜或翻源码。改字段时无任何提示。

**错误：**

```java
@GetMapping("/stats")
public R<Map<String, Object>> stats() {
    Map<String, Object> m = new HashMap<>();
    m.put("total", 100);
    m.put("active", 80);
    return R.ok(m); // ❌ 契约不可见
}
```

**正确：**

```java
@Data
@Schema(description = "统计响应")
public class StatsResponse {
    @Schema(description = "总数") private Long total;
    @Schema(description = "活跃数") private Long active;
}
```

仅当 schema 真正动态（如用户自定义表单）时才例外，且仍应包一层带类型的信封。
