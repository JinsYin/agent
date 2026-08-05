---
title: 列名与审计字段约定
impact: HIGH
tags: db, naming, audit, soft-delete
---

## 列名与审计字段约定

- 列名 `snake_case`（`order_no`）
- 主键 `id`，`BIGINT` 或 `VARCHAR(32)`，与主键策略一致
- 外键列 `fk_{ref}_id`
- 布尔用语义名（`enabled`、`is_default`、`deleted`），类型 `TINYINT(1)`，`0=false / 1=true`
- 审计四件套：`created_at`、`updated_at`、`created_by`、`updated_by`
- 软删：`deleted TINYINT(1) NOT NULL DEFAULT 0`，`1` 为已删除

布尔列不要用 `flag`、`type` 这类无语义名——半年后没人知道 `flag=1` 是启用还是禁用。
