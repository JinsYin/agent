---
title: Service 与 Mapper 方法名
impact: MEDIUM
tags: naming, service, mapper
---

## Service 与 Mapper 方法名

**Service** —— 与 Controller 不同，这里**带**资源名，因为一个 Service 可能被多处调用，脱离上下文也要可读：

- 写：`create{Name}` / `update{Name}` / `remove{Name}`（软删）/ `delete{Name}`（硬删），业务动作用 `cancel{Name}` / `approve{Name}`
- 读：`getById` / `getBy{Xxx}`（单个）、`listAll{Name}s`（全量）、`list{Name}Options`（下拉）、`list{Name}s`（分页）
- 返回类型必须是 `Response` / `Dto` / 原始类型，**绝不是** Entity

**Mapper** —— 简单 CRUD 由 `MPJBaseMapper` 提供（`selectById`、`insert`…），自定义方法用 `selectBy{Xxx}` / `listBy{Xxx}` / `countBy{Xxx}` / `existsBy{Xxx}`，批量用 `insertBatch` / `updateBatchById`。
