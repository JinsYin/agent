---
title: Request 按动作命名并强制校验
impact: HIGH
tags: dto, request, validation
---

## Request 按动作命名并强制校验

`{Name}{Action}Request`——`OrderCreateRequest`、`OrderUpdateRequest`、`OrderCancelRequest`。除非资源确实只有一个动作，否则不要用笼统的 `OrderRequest`：创建和更新的必填字段几乎从不相同，共用一个类就只能把校验全放宽。

字段**必须**带 Jakarta Validation，`message` 用中文（会直接透给前端）。

**正确：**

```java
@Data
@Schema(description = "创建订单请求")
public class OrderCreateRequest {
    @Schema(description = "产品 ID", example = "1001")
    @NotNull(message = "产品 ID 不能为空")
    private Long productId;

    @Schema(description = "数量", example = "2")
    @NotNull(message = "数量不能为空")
    @Min(value = 1, message = "数量必须大于等于 1")
    @Max(value = 999, message = "数量必须小于等于 999")
    private Integer quantity;
}
```

校验失败由 `GlobalExceptionHandler` 捕获 `MethodArgumentNotValidException` 统一处理，不在 Controller 里手写判空。
