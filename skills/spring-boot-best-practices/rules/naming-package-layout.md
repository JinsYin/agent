---
title: 包结构按类型分一级，按业务分二级
impact: MEDIUM
tags: naming, package, structure
---

## 包结构按类型分一级，按业务分二级

- 一级子包按**类型**：`controller`、`service`、`mapper`、`entity`、`converter`、`config`、`constants`、`enums`、`exception`、`utils`
- 二级子包按**业务名词**：只有 `dto` 需要再按业务分（`dto/order/`、`dto/user/`）

Controller、Service、Mapper 直接放在各自类型包根下，不再按业务嵌套——这类文件本就一个业务一个类，再套一层只增加路径深度。

```
com.example.app
├── controller/OrderController.java
├── service/OrderService.java
├── service/impl/OrderServiceImpl.java
├── mapper/OrderMapper.java
├── entity/OrderEntity.java
└── dto/order/OrderCreateRequest.java
```
