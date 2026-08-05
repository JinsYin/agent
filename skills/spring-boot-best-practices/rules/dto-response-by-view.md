---
title: Response 按视图命名
impact: MEDIUM
tags: dto, response, openapi
---

## Response 按视图命名

按**使用场景**命名而非按实体：`OrderDetailResponse`、`OrderListItemResponse`、`OrderCreateResponse`。列表页和详情页需要的字段量差异往往很大，共用一个类会让列表接口传输大量无用字段。

```java
@Data
@Schema(description = "订单详情响应")
public class OrderDetailResponse {
    @Schema(description = "订单 ID") private Long id;
    @Schema(description = "订单编号") private String orderNo;
}
```

每个字段都要有 `@Schema(description = ...)`，中文描述——这是 Knife4j 文档的唯一来源。
