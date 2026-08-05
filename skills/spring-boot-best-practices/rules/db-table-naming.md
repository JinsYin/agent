---
title: 表名 t_ 前缀 + snake_case + 单数
impact: HIGH
tags: db, naming, sql
---

## 表名 t_ 前缀 + snake_case + 单数

表名统一 `t_` 前缀、`snake_case`、**单数**：`t_order`、`t_user`、`t_order_item`。前缀把业务表与视图、中间表、框架表区分开；单数与实体类一一对应，避免 `t_users` ↔ `UserEntity` 这种单复数错位。

实体用 `@TableName` 显式映射，不依赖驼峰自动转换：

```java
@Data
@TableName("t_order_item")
public class OrderItemEntity { }
```
