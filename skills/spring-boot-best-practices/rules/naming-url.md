---
title: URL 全小写 kebab-case 复数
impact: MEDIUM
tags: naming, url, rest, api
---

## URL 全小写 kebab-case 复数

- 全小写，多词用 **kebab-case**：`/order-items`
- 资源名用**复数**：`/orders`、`/users`、`/roles`
- 单个资源 `GET /orders/{id}`，集合 `GET /orders`
- 非 CRUD 动作：`POST /orders/{id}/cancel`，**不要** `/cancelOrder` 这种把动词塞进路径的写法

分页、全量、下拉三种查询的路径约定：

| 用途 | 路径 | 说明 |
|---|---|---|
| 分页 | `GET /roles` | 不要 `/roles/page` |
| 全量 | `GET /roles/all` | 必须限制最大返回条数防 OOM |
| 下拉选项 | `GET /roles/options` | 只返回 `{id, name}` |

注意 URL 用复数，而类名和实体用单数——这不矛盾：URL 指的是资源集合，类指的是单个对象。
