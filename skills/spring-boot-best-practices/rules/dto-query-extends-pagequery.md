---
title: 分页查询必须 extends PageQuery
impact: HIGH
tags: dto, query, pagination
---

## 分页查询必须 extends PageQuery

`{Name}Query` 用于 GET，通过 `@Valid @ModelAttribute` 绑定查询串。**需要分页就必须 `extends PageQuery`**，拿到统一的 `pageNo` / `pageSize` 及其边界校验（页码 ≥ 1、每页 ≤ 200）。

自己声明分页字段会漏掉上限校验，前端传 `pageSize=999999` 直接拖垮数据库。

**错误：**

```java
public class OrderPageQuery {
    private Long pageNo;    // ❌ 无 @Min
    private Long pageSize;  // ❌ 无上限
}
```

**正确：**

```java
@Data
@Schema(description = "订单分页查询")
public class OrderPageQuery extends PageQuery {
    @Schema(description = "订单编号（模糊查询）")
    private String orderNo;

    @Schema(description = "状态")
    @EnumMatch(enumClass = OrderStatus.class)
    private String status;
}
```

不分页的查询用普通 `{Name}Query`，不要为了统一而硬加分页字段。
